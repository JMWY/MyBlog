# 将 kv 数据自动填入 protobuf 数据结构

```cpp
/*================================================================
*   Copyright (C) alibaba.com 2022-10 All rights reserved.
*   文件名称：parsePB.cc
*   创 建 者：chenfeng.lyg
*   创建日期：2022年10月31日
*   描    述：
================================================================*/
#include <unordered_map>
#include <string>
#include <iostream>
#include "Requests.pb.h"
using std::string;

typedef std::unordered_map<std::string, std::string> KeyValueHashMap;
// #define LOG(level, format, args...) do { printf(format, ##args); printf("\n");} while (0)
#define LOG(level, format, args...) do { ;} while (0)

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

template <typename T>
inline void String2Value(const std::string& s, T& val) {
    val = static_cast<T>(atof(s.c_str()));
}
template <>
inline void String2Value(const std::string& s, std::string& val) {
    val = s;
}

#ifndef _SET_PB_VALUE
#define _SET_PB_VALUE(PBType, CType, value_str)                                 \
    do {                                                                        \
        if (field_descriptor->is_repeated()) {                                  \
            std::vector<std::string> vec;                                       \
            StringUtil::Split(value_str, /*AND_DELIMITER*/ "&", &vec);          \
            for (size_t id = 0; id < vec.size(); ++id) {                        \
                CType val;                                                      \
                String2Value(vec[id], val);                                     \
                reflection->Add##PBType(message, field_descriptor, val);        \
            }                                                                   \
        } else {                                                                \
            CType val;                                                          \
            String2Value(value_str, val);                                       \
            reflection->Set##PBType(message, field_descriptor, val);            \
        }                                                                       \
    } while (0)
// 
//   函数功能：将 kv 数据自动填入 protobuf 数据结构中的通用函数
//       --* 使用示例 *--
//            --* protobuf 文件: Requsts.proto *--
//            syntax = "proto2";
//            message  MetaRequest {
//                required int32 pid = 1;
//                optional string ip = 2;
//                repeated string category = 3;
//            }
//            message  TanxRequest {
//                required int32 pid = 1;
//                optional float lat = 2;
//                optional float lon = 3;
//            }
//            message Requests {
//                optional MetaRequest meta_req = 1;
//                optional TanxRequest tanx_req = 2;
//            }
//            --* EOF *--
//  
//            --* 函数调用 *--
//            #include <unordered_map>
//            #include <string>
//            #include <iostream>
//            #include "Requests.pb.h"
//
//            int main() {
//                const std::unordered_map data_map = {
//                    {"Requests.meta_req.pid", "10001"},
//                    {"Requests.meta_req.ip","127.0.0.1"},
//                    {"Requests.meta_req.category","美食&火锅&川菜"},
//                    {"Requests.tanx_req.pid","17"},
//                    {"lat","36.789659"},
//                    {"log","106.197318"},
//                };
//                google::protobuf::Requests requests;
//                const std::string message_path("Requests");
//                int num = parseKVToPB(data_map, message_path, &requests);
//                std::cout << num << std::endl;
//                std::cout << requests.Utf8DebugString() << std::endl;
//                return 0;
//            }
//            -- 打印结果:
//            6
//            meta_req {
//              pid: 10001
//              ip: "127.0.0.1"
//              category: "美食"
//              category: "火锅"
//              category: "川菜"
//              height: 2400
//              slot_num: 1
//            }
//            tanx_req {
//              pid: 17
//              lat: 36.789659
//              lon: 106.197318
//            }
//   @ param data_map 候选数据集合, 其中,key 是路径,val 是待填入值
//   @ param message_path 当前 message 对象的路径
//   @ param message 指向当前 message 对象的指针
//   @ return (message 内的内置类型变量)成功赋值次数
int parseKVToPB(const KeyValueHashMap& data_map,
                const std::string& message_path,
                google::protobuf::Message* message) {
    LOG(DEBUG, "message_path = %s", message_path.c_str());
    if (message == nullptr) {
        return -1;
    }
    int get_num = 0;
    const google::protobuf::Reflection* reflection = message->GetReflection();
    const google::protobuf::Descriptor* descriptor = message->GetDescriptor();
    for (int idx = 0; idx < descriptor->field_count(); ++idx) {
        const google::protobuf::FieldDescriptor* field_descriptor  = descriptor->field(idx);
        const std::string& field_name = field_descriptor->name();
        const std::string cur_path = message_path.empty() ? field_name : (message_path + "." + field_name);
        LOG(DEBUG, "start assigning value to protobuf message field[%s]", cur_path.c_str());
        // 变量类型是 Message 数据结构
        if (field_descriptor->cpp_type() == google::protobuf::FieldDescriptor::CPPTYPE_MESSAGE) {
            int hit_num = 0;
            if (field_descriptor->is_repeated()) {
                hit_num = parseKVToPB(data_map, cur_path, reflection->AddMessage(message, field_descriptor));
                if (hit_num == 0) {
                    LOG(DEBUG, "repeated protobuf message field[%s] has no value", cur_path.c_str());
                    reflection->RemoveLast(message, field_descriptor);
                }
            } else {
                hit_num = parseKVToPB(data_map, cur_path, reflection->MutableMessage(message, field_descriptor));
                if (hit_num == 0) {
                    LOG(DEBUG, "protobuf message field[%s] has no value", cur_path.c_str());
                    reflection->ReleaseMessage(message, field_descriptor);
                }
            }
            get_num += hit_num;
            continue;
        }
        // 变量类型是内置数据类型
        KeyValueHashMap::const_iterator iter;
        // 1. 优先查绝对路径下有无数据值，2.再直接查元素名取数据.若都无值，则结束
        if (((iter = data_map.find(cur_path)) == data_map.end()) && ((iter = data_map.find(field_name)) == data_map.end())) {
            LOG(DEBUG, "protobuf field[%s] has no value", field_name.c_str());
            if (field_descriptor->is_required()) {
                LOG(WARN, "protobuf required field[%s] has no value", field_name.c_str());
            }
            continue;
        }
        ++get_num;
        int cpp_type = field_descriptor->cpp_type();
        switch (cpp_type) {
            case google::protobuf::FieldDescriptor::CPPTYPE_INT32:
                _SET_PB_VALUE(Int32, int32_t, iter->second);
                break;
            case google::protobuf::FieldDescriptor::CPPTYPE_INT64:
                _SET_PB_VALUE(Int64, int64_t, iter->second);
                break;
            case google::protobuf::FieldDescriptor::CPPTYPE_UINT32:
                _SET_PB_VALUE(UInt32, uint32_t, iter->second);
                break;
            case google::protobuf::FieldDescriptor::CPPTYPE_UINT64:
                _SET_PB_VALUE(UInt64, uint64_t, iter->second);
                break;
            case google::protobuf::FieldDescriptor::CPPTYPE_DOUBLE:
                _SET_PB_VALUE(Double, double, iter->second);
                break;
            case google::protobuf::FieldDescriptor::CPPTYPE_FLOAT:
                _SET_PB_VALUE(Float, float, iter->second);
                break;
            case google::protobuf::FieldDescriptor::CPPTYPE_BOOL:
                _SET_PB_VALUE(Bool, bool, iter->second);
                break;
            case google::protobuf::FieldDescriptor::CPPTYPE_ENUM:
                _SET_PB_VALUE(EnumValue, int32_t, iter->second);
                break;
            case google::protobuf::FieldDescriptor::CPPTYPE_STRING:
                _SET_PB_VALUE(String, string, iter->second);
                break;
            default:
                --get_num;
                LOG(ERROR, "unknown protobuf field type [%d] on field[%s]",
                           cpp_type, field_name.c_str());
                break;
        }
    }
    return get_num;
}
#undef _SET_PB_VALUE
#endif


int main() {
    const std::unordered_map<std::string, std::string> data_map = {
        {"Requests.meta_req.pid", "10001"},
        {"Requests.meta_req.ip","127.0.0.1"},
        {"Requests.meta_req.category","美食&火锅&川菜"},
        {"Requests.tanx_req.pid","17"},
        {"lat","36.789659"},
        {"lon","106.197318"},
    };
    ::Requests requests;
    const std::string message_path("Requests");
    int num = parseKVToPB(data_map, message_path, &requests);
    std::cout << num << std::endl;
    std::cout << requests.Utf8DebugString() << std::endl;
    return 0;
}

```

