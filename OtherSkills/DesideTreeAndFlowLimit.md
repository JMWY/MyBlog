# 决策树用于精细化限流
 
 ```cpp
 /*================================================================
*   Copyright (C) alibaba.com 2021-10 All rights reserved.
*   文件名称：TrafficLimitTree.h
*   创 建 者：chenfeng.lyg
*   创建日期：2021年10月26日
*   描    述：
================================================================*/
#ifndef BASIC_UTIL_TRAFFICLIMITTREE_H_
#define BASIC_UTIL_TRAFFICLIMITTREE_H_

#include <pthread.h>
#include <jsoncpp/json/json.h>
// #include <basic/include/Head.h>
// #include <wings_worker/common/FileUtil.h>
#include "TrafficLimitMechanism.h"
#include <string>
#include <vector>
#include <unordered_map>
using namespace std;
typedef std::unordered_map<std::string, std::string> KeyValueHashMap;
namespace basic_util {

/********** 限流树配置示例&&说明 *****************************************************************
[  // 中括号，说明接下来是: 【TCM】 + 【n个子树】
    "total_max_qps:1000/1", // 【流控参数：名称:上限阈值/ 窗口（单位:s） 】可选项
    {  // 花括号，无 key，说明接下来是一个新的子树
        "page_id":{  // 花括号，有 key，说明接下来是多个决策
            "9":["navtab_page_flow_limit:1000/1"],
            "48":["article_page_flow_limit:100/1"]
        }
    },
    {
        "query_type":{
            "mainbubble":["mainbubble_flow_limit:900/1"],
            "amap_shanpin":[
                {
                    "scene":{
                        "106":["shanping_scene_106_flow_limit:100/1"]
                    }
                },
                {
                    "os":{
                        "ios":["shanping_ios_flow_limit:100/1"]
                    }
                }
            ]
        }
    }
]

* 其中, 流控参数为可选配置，如果未配或者配置不合法，则相关节点不限流;
* 时间窗口 <=0 时，不限流; 否则,
* 时间窗口 > 0 时，如果上限阈值 <= 0，则表示流量全部过滤。
********************************************************************************************/

class TrafficLimitTree {
 private:
  // 决策树共有三种有效节点类型
  enum NodeType {
      LEAF_NODE,
      SELECT_KEY_NODE,
      SELECT_VAL_NODE,
      UNKNOWN_NODE,
  };

  // 决策树Node
  class Node;
  typedef std::shared_ptr<Node> NodePtr;
  class Node {
   public:
    Node();
    // 节点类型
    NodeType node_type_;
    // 决策的key; 叶子结点用来记录路径
    string key_;
    // 限制流量的最小单元, 处理登记流量，返回 true 和 false
    TrafficLimitMechanismPtr tlm_;
    // 当前可决策值列表
    std::unordered_map<std::string, NodePtr> children_;
    // 下游决策子树列表
    vector<NodePtr> next_;
  };

 public:
  TrafficLimitTree();

  // 支持用 json 配置更新树
  int parseJson(const Json::Value &jva);
  // 支持用 string 配置更新树
  int parseString(const std::string& content);
  // 支持用本地文件配置更新树
  int parseFile(const std::string& filePath);

  // 决策树提供的流量登记入口
  bool registering(const KeyValueHashMap& params);

 private:
  // 叶子节点用，根据配置信息生成最小流量管理单元
  TrafficLimitMechanismPtr makeTLM(const std::string& cfg_str);

  // 执行建树核心代码
  int parseJson(const Json::Value &jval,
                const std::string& cur_path,
                Node &cur_node,
                bool has_no_key = false);
  // 执行流量登记
  bool registering(const KeyValueHashMap& params,
                   Node& cur_node,
                   const time_t& timstamp);

 private:
  NodePtr decide_tree_;
  pthread_rwlock_t  rwlock_;

// private:
//  LOG_DECLARE();
};

typedef std::shared_ptr<TrafficLimitTree> TrafficLimitTreePtr;

}

#endif  // BASIC_UTIL_TRAFFICLIMITTREE_H_
 ```
 
 
 ```cpp
 /*================================================================
*   Copyright (C) alibaba.com 2021-10 All rights reserved.
*   文件名称：TrafficLimitTree.cpp
*   创 建 者：chenfeng.lyg
*   创建日期：2021年10月27日
*   描    述：
================================================================*/
#include "TrafficLimitTree.h"
#include <string.h>
#include <cassert>
#include <memory>
#include <algorithm>
using namespace std;
namespace basic_util {
struct StringUtil {
    static void Split(const std::string& text,
               const char* sepStr,
               std::vector<std::string> *vec) {
        assert(vec != NULL);
        size_t sepLen = strlen(sepStr);
        size_t e = 0, s = 0;
        while (e != std::string::npos) {
            e = text.find(sepStr, e, sepLen);
            if (e != std::string::npos) {
                if (e != s) {
                    vec->push_back(text.substr(s, e-s));
                }
                e += sepLen;
                s = e;
            }
        }
        vec->push_back(text.substr(s, text.length() - s));
    }
};

TrafficLimitTree::Node::Node() : node_type_(UNKNOWN_NODE) {
}

TrafficLimitTree::TrafficLimitTree()
    : decide_tree_(new TrafficLimitTree::Node()) {
        pthread_rwlockattr_t attr;
        pthread_rwlockattr_setkind_np(&attr, PTHREAD_RWLOCK_PREFER_READER_NP);
        pthread_rwlock_init(&rwlock_, &attr);
}

// int TrafficLimitTree::parseFile(const std::string& filePath) {
//     string content;
//     if (!WN_COMMON::FileUtil::readFileContent(filePath, content)) {
//         LOG(ERROR, "read %s failed", filePath.c_str());
//         return -1;
//     }
//     int ret = parseString(content);
//     if (ret < 0) {
//         LOG(ERROR, "parse %s failed, error_code:%d", filePath.c_str(), ret);
//         return -2;
//     }
//     return ret;
// }

int TrafficLimitTree::parseString(const std::string& content) {
    Json::Reader reader;
    Json::Value jval;
    if (!reader.parse(content, jval)) {
        LOG(ERROR,  "parse content failed, for it cannot be translated to json");
        return -1;
    }
    return parseJson(jval);
}

/** 
 * private
 * 构建叶子节点内的流量限制机制
 * @ param cfg_str 限流配置
     ** 配置格式：name:limit/window, 例如 scene1_max_qps:10000/1
 * @ return TrafficLimitMechanismPtr 流量限制机制(流控最小单元)
*/
TrafficLimitMechanismPtr TrafficLimitTree::makeTLM(const std::string& cfg_str) {
    TrafficLimitMechanismPtr tlm;
    std::vector<std::string> vec;
    // 配置格式：name:limit/window, 例如 scene1_max_qps:10000/1
    StringUtil::Split(cfg_str, ":", &vec);
    if (vec.size() != 2) {
        LOG(ERROR, "parse tlm config[%s] invalid failed", cfg_str.c_str());
        return tlm;
    }
    const std::string& name = vec[0];
    std::vector<std::string> vec2;
    StringUtil::Split(vec[1], "/", &vec2);
    if (vec2.size() != 2) {
        LOG(ERROR, "parse tlm config[%s] invalid failed", cfg_str.c_str());
        return tlm;
    }
    const int limit_traffic = atoi(vec2[0].c_str());
    const int window = atoi(vec2[1].c_str());

    tlm = std::make_shared<TrafficLimitMechanism>(name,
                                                  limit_traffic,
                                                  window);
    if (nullptr == tlm) {
        LOG(ERROR, "parse tlm config[%s] invalid failed", cfg_str.c_str());
    }
    return tlm;
}


/**
 * public
 * 解析限流树的配置,生成限流树
 * @ param jval 来自 diamond 的限流树配置
 * @ return 0, 解析成功; 其他，解析出错
 */
int TrafficLimitTree::parseJson(const Json::Value &jval) {
    if (!jval.isArray()) {
        LOG(ERROR, "parse %s failed", jval.toStyledString().c_str());
        return -1;
    }
    TrafficLimitTree::NodePtr new_tree(new TrafficLimitTree::Node());
    const std::string cur_path("root:");
    int ret = parseJson(jval, cur_path, *new_tree);
    if (ret < 0) {
        LOG(ERROR, "parse %s failed, %d", jval.toStyledString().c_str(), ret);
        return -2;
    }
    pthread_rwlock_wrlock(&rwlock_);
    decide_tree_ = new_tree;
    pthread_rwlock_unlock(&rwlock_);
    return 0;
}

/** 
 * private
 * 递归解析限流树的配置,生成限流树
 * @ param jval 目前节点下的限流配置
     *** 配置示例&&说明,如下 ***
     [  // 中括号，说明接下来是: 【TCM】 + 【n个子树】
         "total_max_qps:1000/1", // 【流控参数：名称:上限阈值/ 窗口（单位:s） 】可选项
         {  // 花括号，无 key，说明接下来是一个新的子树
             "query_type":{  // 花括号，有 key，说明接下来是多个决策
                 "mainbubble":["mainbubble_flow_limit:900/1"],
                 "amap_shanpin":[
                     {
                         "scene":{
                             "106":["shanping_scene_106_flow_limit:100/1"]
                         }
                     },
                     {
                         "os":{
                             "ios":["shanping_ios_flow_limit:100/1"]
                         }
                     }
                 ]
             }
         }
     ]
     ** 其中, 流控参数为可选配置，如果未配或者配置不合法，则相关节点不限流;
     ** 时间窗口 <=0 时，不限流; 否则,
     ** 时间窗口 > 0 时，如果上限阈值 <= 0，则表示流量全部过滤。
 * @ param cur_path 从根到目前节点的路径
 * @ param cur_node 目前正在解析的节点
 * @ param has_no_key 如果是花括号(参考上面配置示例)，记录其前头有无key
 * @ return 0, 解析成功; 其他，解析出错
 */
int TrafficLimitTree::parseJson(const Json::Value &jval,
                                const std::string& cur_path,
                                TrafficLimitTree::Node &cur_node,
                                bool has_no_key) {
    const Json::ValueType type = jval.type();
    int ret = 0;
    switch (type) {
        case Json::arrayValue:
            {
                cur_node.node_type_ = SELECT_KEY_NODE;
                for (Json::ArrayIndex id = 0; id < jval.size(); ++id) {
                    TrafficLimitTree::NodePtr node(new TrafficLimitTree::Node());
                    cur_node.next_.emplace_back(node);
                    ret = parseJson(jval[id], cur_path, *node, true);
                    if (ret < 0) {
                        return ret;
                    }
                }
            }
            break;
        case Json::objectValue:
            {
                cur_node.node_type_ = SELECT_VAL_NODE;
                Json::Value::Members keys = jval.getMemberNames();
                if (has_no_key) {
                    if (keys.size() != 1) {
                        for (const auto& key : keys) {
                            printf ("key=%s\n", key.c_str());
                        }
                        return -2;
                    }
                    cur_node.key_ = keys[0u];
                    std::string tmp_path = cur_path + keys[0u] + "=";
                    return parseJson(jval[keys[0u]], tmp_path, cur_node);
                }
                for (size_t idx = 0; idx < keys.size(); ++idx) {
                    TrafficLimitTree::NodePtr node(new TrafficLimitTree::Node());
                    cur_node.children_[keys[idx]] = node;
                    std::string tmp_path = cur_path + keys[idx] + "|";
                    ret = parseJson(jval[keys[idx]], tmp_path, *node);
                    if (ret < 0) {
                        return ret;
                    }
                }
            }
            break;
        case Json::stringValue:
            {
                cur_node.node_type_ = LEAF_NODE;
                cur_node.key_ = cur_path;
                // 若配置不当,make 失败时,该节点默认免登记,不控流
                cur_node.tlm_ = makeTLM(jval.asString());
                LOG(INFO, "path:[%s], tlm config[%s] parse finished", cur_path.c_str(), jval.asCString());
                ret = 0;
            }
            break;
        default:
            ret = -1;
    }

    return ret;
}

/** 
 * public
 * 限流树对外可访问登记的函数接口
 * @ param params 当前请求信息
 * @ return true, 登记成功,不限流; false，登记失败,限流
 */
bool TrafficLimitTree::registering(const KeyValueHashMap& params) {
    // 读写锁是为了防止登记时候有配置更新, 采用双 buffer 机制,
    // 并确保配置更新频度时，可以不用锁
    const int32_t timestamp = time(NULL);
    pthread_rwlock_rdlock(&rwlock_);
    // 记录登记时的时间戳
    bool status = registering(params, *decide_tree_, timestamp);
    pthread_rwlock_unlock(&rwlock_);
    return status;
}

/** 
 * private
 * 限流树登记：内部实际执行函数
 * @ param params 当前请求信息
 * @ param cur_node 当前登记节点
 * @ param timestamp 登记时的时间戳
 * @ return true, 登记成功,不限流; false，登记失败,限流
 */
bool TrafficLimitTree::registering(const KeyValueHashMap& params, TrafficLimitTree::Node& cur_node, const time_t& timestamp) {
    switch (cur_node.node_type_) {
        case LEAF_NODE:
            {
                // 查看当前叶子节点路径
                LOG(INFO, "peek tlm, path:%s", cur_node.key_.c_str());
                // 当前叶子节点没有配流量限制, 不限流
                if (cur_node.tlm_ == nullptr) {
                    // 没配置 tlm, 表示无需登记, 不限流
                    LOG(INFO, "leaf node:%s, has no tlm data, not need to be registered",
                               cur_node.key_.c_str());
                    return true;
                }
                // 当前叶子节点配了流量限制, 进行登记, 判断是否限流
                return cur_node.tlm_->doRegister(timestamp);
            }
            break;
        case SELECT_KEY_NODE:
            {
                // 多棵决策子树, 分别登记, 若有一个限流，则限流
                bool is_permitted = true;
                for (auto it : cur_node.next_) {
                    is_permitted = registering(params, *it, timestamp) && is_permitted;
                }
                return is_permitted;
            }
            break;
        case SELECT_VAL_NODE:
            {
                // 从请求中获取当前决策节点的 val 值
                auto iter = params.find(cur_node.key_);
                // 请求无此key，表示不用走此路径的限流 
                if (iter == params.end()) {
                    return true;
                }
                string val = iter->second;
                auto iter2 = cur_node.children_.find(val);
                // 请求中此key对应 val 值是否需要限流
                if (iter2 != cur_node.children_.end() && iter2->second != nullptr) {
                    // 用此 val 对应的限流节点进行登记，判断是否限流
                    return registering(params, *(iter2->second), timestamp);
                }
            }
            break;
        default:
            LOG(ERROR, "unknow node type:%d", cur_node.node_type_);
            break;
    }
    return true;
}

}
 
 ```
 
 
 
 
 ```cpp
 g++ -std=c++14 *.cpp -ljsoncpp -lpthread -o testBinary
 
 ```
