# LeedCode 梳理

| 编号  | 题目      |题解             |  地址 |  
|:--:|:--:|:--:|:--:|
|a||b|
| 1|A|B|C|

# leetcode

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
地址：https://leetcode-cn.com/problems/sparse-similarity-lcci/ 

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

</details>



### 栈


### 队列


### 堆
