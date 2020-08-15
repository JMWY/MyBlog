# LeedCode 梳理

## 字符串类


## 数组类


## 链表类


## 树类


### 二叉树类


## 图类


## 数学类



## 数据结构类

<details>
<summary> 1. 稀疏相似度 (倒排索引) （https://leetcode-cn.com/problems/sparse-similarity-lcci/） </summary> 

题解：
```bash
class Solution {
public:
    vector<string> computeSimilarities(vector<vector<int>>& docs) {
        vector<string> res;
        unordered_map<int, vector<int> > elem2doc;
        for (size_t doc_id = 0; doc_id < docs.size(); ++doc_id) {
            for (size_t elem_id = 0; elem_id < docs[doc_id].size(); ++elem_id) {
                elem2doc[docs[doc_id][elem_id]].push_back(doc_id);
            }
        }
        
        unordered_map<int, unordered_map<int, size_t> > doc2doc2freq;
        for (auto iter = elem2doc.begin(); iter != elem2doc.end(); ++iter) {
            for (size_t id = 0; id < iter->second.size(); ++id) {
                for (size_t k = id+1; k < iter->second.size(); ++k) {
                    doc2doc2freq[iter->second[id]][iter->second[k]]++;
                }
            }
        }

        for (auto iter = doc2doc2freq.begin(); iter != doc2doc2freq.end(); ++iter) {
            for (auto iter2 = iter->second.begin(); iter2 != iter->second.end(); ++iter2) {
                double similarity = double(iter2->second) / double(docs[iter->first].size() + docs[iter2->first].size() - iter2->second);
                if (similarity >= 0.000005f) {
                    char buffer[256];
                    int n = snprintf(buffer, 256, "%lu,%lu: %.4f", iter->first, iter2->first, similarity + 1e-9);
                    if (0 < n && n < 256) {
                        buffer[n] = '\0';
                        res.push_back(buffer);
                    }
                }
            }
        }
        return res;
    }
};
``` 

超时题解（O(n^2*m)）：
```c++
class Solution {
public:
    string compare2DocSimilarity(vector<int>& short_doc, size_t short_id, vector<int>& long_doc, size_t long_id) {
        if (short_doc.size() > long_doc.size()) return compare2DocSimilarity(long_doc, long_id, short_doc, short_id);
        string res("");
        if (short_doc.empty()) return res;
        unordered_set<int> short_set;
        for (size_t id = 0; id < short_doc.size(); ++id) short_set.insert(short_doc[id]);
        size_t intersection_num = 0;
        size_t unionsection_num = short_set.size();
        for (size_t id = 0; id < long_doc.size(); ++id) {
            if (short_set.find(long_doc[id]) != short_set.end()) {
                ++intersection_num;
            } else {
                ++unionsection_num;
            }
        }
        double similarity = double(intersection_num) / double(unionsection_num);
        if (similarity < 0.00005f) return res;
        char buffer[256];
        size_t min_id = min(short_id, long_id);
        size_t max_id = max(short_id, long_id);
        int n = snprintf(buffer, 256, "%lu,%lu: %.4f", min_id, max_id, similarity+1e-9);
        if (0 < n && n < 256) {
            buffer[n] = '\0';
        }
        return buffer;
    }

    vector<string> computeSimilarities(vector<vector<int>>& docs) {
        vector<string> res;
        for (size_t m = 0; m < docs.size(); ++m) {
            for (size_t n = m+1; n < docs.size(); ++n) {
                string cmp_str = compare2DocSimilarity(docs[m], m, docs[n], n);
                if (!cmp_str.empty()) {
                    res.push_back(cmp_str);
                }
            }
        }
        return res;

    }
};
```

</details>



### 栈


### 队列


### 堆




-----------
# 附录
-----------
## 算法解题思想 
算法思想：枚举，模拟，递推，递归，分治，动归，贪心和回溯（试探，万能解，一般需要挖掘约束条件做剪枝来提升效率）。
### 分治
### 动态规划

### 贪心
### 回溯
### 分支界限法
