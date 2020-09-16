# LeetCode 梳理

## 字符串类
<details>
<summary> 1. 验证回文串 (easy)（https://leetcode-cn.com/problems/valid-palindrome/） </summary> 
    
```cpp
class Solution {
public:
    bool isPalindrome(const string& s) {
        int left = 0;
        int right = s.length() - 1;
        while (left < right) {
            while (left < right && !isalnum(s[left])) { ++left; }
            while (left < right && !isalnum(s[right])) { --right; }
            if (tolower(s[left]) != tolower(s[right])) { return false; }
            ++left, --right;
        }
        return true;
    }
};
``` 
</details>

<details>
<summary> 2. 分割回文串 (medium)（https://leetcode-cn.com/problems/palindrome-partitioning/） </summary> 
</details>

<details>
<summary> 3. 单词拆分 (medium)（https://leetcode-cn.com/problems/word-break/） </summary> 
</details>

<details>
<summary> 4. 单词拆分 II (hard)（https://leetcode-cn.com/problems/word-break-ii/） </summary> 
</details>

<details>
<summary> 5. 单词搜索 II (hard)（https://leetcode-cn.com/problems/word-search-ii/） </summary> 
</details>

<details>
<summary> 6. 实现 Trie (前缀树) (medium)（https://leetcode-cn.com/problems/implement-trie-prefix-tree/） </summary> 
</details>

<details>
<summary> 6. 字符串转换整数 (atoi) (medium)（https://leetcode-cn.com/problems/string-to-integer-atoi/） </summary> 
</details>

<details>
<summary> 7. 翻转单词顺序 (easy)（https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/） </summary>
    
```cpp
// 状态转移
class Solution {
public:
    string reverseWords(string s) {
        string res("");
        bool is_blank_status = true;
        size_t end = s.size();
        for (int cur = s.size()-1; cur >= 0; --cur) {
            if (is_blank_status && !isBlank(s[cur])) {
                end = cur;
                is_blank_status = false;
            }
            if (!is_blank_status && isBlank(s[cur])) {
                if (!res.empty()) {
                    res.push_back(' ');
                }
                res.append(s, cur+1, end - cur);
                is_blank_status = true;
            }
        }
        // not end with blank
        if (!is_blank_status) {
            if (!res.empty()) {
                res.push_back(' ');
            }
            res.append(s, 0, end+1);
        }
        return res;
    }
    
private:
    bool isBlank(char c) const {
        return ' ' == c;
    }
};
```

</details>


## 数组类
<details>
<summary> 1. 最大子段和 (easy &hearts;)（https://leetcode-cn.com/problems/maximum-subarray/） </summary> 

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxSum = INT_MIN;
        int curSum = 0;
        for (size_t id = 0; id < nums.size(); ++id) {
            if (curSum <= 0) {
                curSum = 0;
            }
            curSum += nums[id];
            if (curSum > maxSum) {
                maxSum = curSum;
            }
        }
        return maxSum;
    }
};
```

</details>

<details>
<summary> 2. 最大子矩阵 (hard &hearts;)（https://leetcode-cn.com/problems/max-submatrix-lcci/） </summary> 
    
```cpp
class Solution {
public:
    vector<int> getMaxMatrix(vector<vector<int>>& matrix) {
        vector<int> res;
        int maxSum = INT_MIN;
        for (size_t row1 = 0; row1 < matrix.size(); ++row1) {
            // 记录各列的和
            vector<int> sum(matrix[0].size(), 0);
            for (size_t row2 = row1; row2 < matrix.size(); ++row2) {
                int curSum = -1;
                int col1 = -1;
                for (size_t col2 = 0; col2 < matrix[0].size(); ++col2) {
                    sum[col2] += matrix[row2][col2];
                    if (curSum <= 0) {
                        // 重置左边列号
                        curSum = 0;
                        col1 = col2;
                    }
                    curSum += sum[col2];
                    if (curSum > maxSum) {
                        // 记录右边列号
                        maxSum = curSum;
                        if (res.empty()) {
                            res.resize(4);
                        }
                        res[0] = row1, res[1] = col1;
                        res[2] = row2, res[3] = col2;
                    }
                }
            }
        }
        return res;
    }
};
```
    
</details>

<details>
<summary> 3. 乘积最大子数组 (medium)（https://leetcode-cn.com/problems/maximum-product-subarray/） </summary> 
</details>

<details>
<summary> 4. 多数元素 (easy)（https://leetcode-cn.com/problems/majority-element/） </summary> 
</details>

<details>
<summary> 5. 旋转数组 (easy)（https://leetcode-cn.com/problems/rotate-array/） </summary> 
</details>

<details>
<summary> 6. 存在重复元素 II (easy)（https://leetcode-cn.com/problems/contains-duplicate-ii/） </summary> 
</details>

<details>
<summary> 7. 存在重复元素 III (medium)（https://leetcode-cn.com/problems/contains-duplicate-iii/） </summary> 
</details>

<details>
<summary> 8. 两个数组的交集 II (easy)（https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/） </summary> 
</details>

<details>
<summary> 9. 移动零 (easy)（https://leetcode-cn.com/problems/move-zeroes/） </summary> 
</details>

<details>
<summary> 10. 递增的三元子序列 (medium)（https://leetcode-cn.com/problems/increasing-triplet-subsequence/） </summary> 
</details>

<details>
<summary> 11. 搜索二维矩阵 II (medium)（https://leetcode-cn.com/problems/search-a-2d-matrix-ii/） </summary> 
</details>

<details>
<summary> 12. 除自身以外数组的乘积 (medium)（https://leetcode-cn.com/problems/product-of-array-except-self/） </summary> 
</details>


## 链表类

<details>
<summary> 1. 反转链表 （easy &hearts;）（https://leetcode-cn.com/problems/reverse-linked-list/） </summary>
    
```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head) {
            return head;
        }
        ListNode *cur = head;
        ListNode *pre = NULL;
        ListNode *post = NULL;
        while (cur != NULL) {
            post = cur->next;
            cur->next = pre;
            pre = cur;
            cur = post;
        }
        return pre;
    }
};
```

</details>

<details>
<summary> 2. 链表中倒数第k个节点 (easy &hearts;)（https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/） </summary> 
    
```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        ListNode *fast = head;
        for (int id = 0; id < k; ++id) {
            if (!fast) {
                return NULL;
            }
            fast = fast->next;
        }
        ListNode *slow = head;
        while (fast != NULL) {
            fast = fast->next;
            slow = slow->next;
        }
        return slow;
    }
};
```

</details>

<details>
<summary> 3. 环形链表 (easy &hearts;)（https://leetcode-cn.com/problems/linked-list-cycle/） </summary> 
    
```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if (NULL == head || NULL == head->next) {
            return false;
        }
        ListNode *fast = head->next->next;
        ListNode *slow = head->next;
        while (fast && fast->next) {
            if (fast == slow) {
                return true;
            }
            slow = slow->next;
            fast = fast->next->next;
        }
        return false;
    }
};
```
    
</details>

<details>
<summary> 4. 相交链表 (easy &hearts;)（https://leetcode-cn.com/problems/intersection-of-two-linked-lists/） </summary> 

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        //  链表 L1 长度为 a+c, 链表 L2 长度为 b+c, 其中 c >= 0.
        //  由 (a+c)+b == (b+c)+a, 得出两个指针在公共点相遇
        //  时间复杂度 O(a+b+c)
        ListNode *p1 = headA;
        ListNode *p2 = headB;
        while (p1 != p2) {
            p1 = (NULL == p1) ? headB : p1->next;
            p2 = (NULL == p2) ? headA : p2->next;
        }
        return p1;
    }
};
```
 
</details>

<details>
<summary> 5. 回文链表 (easy)（https://leetcode-cn.com/problems/palindrome-linked-list/） </summary> 
</details>

<details>
<summary> 6. 链表排序 (medium)（https://leetcode-cn.com/problems/sort-list/） </summary> 
</details>

<details>
<summary> 7. 删除链表中的节点 (easy)（https://leetcode-cn.com/problems/delete-node-in-a-linked-list/） </summary> 
</details>

<details>
<summary> 8. 奇偶链表 (medium)（https://leetcode-cn.com/problems/odd-even-linked-list/） </summary> 
</details>

<details>
<summary> 9. 复杂链表的复制 (medium)（https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/） </summary> 
</details>

<details>
<summary> 10. 设计链表 (medium)（https://leetcode-cn.com/problems/design-linked-list/） </summary> 
</details>


<details>
<summary> 11. 分隔链表 (medium)（https://leetcode-cn.com/problems/split-linked-list-in-parts/） </summary> 
</details>

<details>
<summary> 12. 分隔链表 (medium)（https://leetcode-cn.com/problems/partition-list/） </summary> 
</details>

<details>
<summary> 13. 链表求和 (medium)（https://leetcode-cn.com/problems/sum-lists-lcci/） </summary> 
    
```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode *l3 = NULL;
        ListNode *l3tail = NULL;
        int sum = 0;
        while (l1 != NULL || l2 !=NULL) {
            if (l1 != NULL && l2 != NULL) {
                sum += l1->val + l2->val;
                l1 = l1->next, l2 = l2->next;
            } else if (l1 != NULL) {
                sum += l1->val;
                l1 = l1->next;
            } else {
                sum += l2->val;
                l2 = l2->next;
            }    
            if (l3 == NULL) {
                l3tail = l3 = new ListNode(sum >= 10 ? (sum - 10) : sum);
            } else {
                l3tail->next = new ListNode(sum >= 10 ? (sum - 10) : sum);
                l3tail = l3tail->next;
            }
            sum = sum >= 10 ? 1 : 0;
        }
        if (sum > 0) {
            l3tail->next = new ListNode(sum);
        }
        return l3;
    }
};
```
* 如果链表非逆序数字，需要在计算前后分别执行下反转链表。

</details>

<details>
<summary> 14. 重排链表 (medium)（https://leetcode-cn.com/problems/reorder-list/） </summary> 
</details>

<details>
<summary> 15. 旋转链表 (medium)（https://leetcode-cn.com/problems/rotate-list/） </summary> 
</details>

<details>
<summary> 16. 环形链表 II (medium)（https://leetcode-cn.com/problems/linked-list-cycle-ii/） </summary> 
</details>

<details>
<summary> 17. 反转链表 II (medium)（https://leetcode-cn.com/problems/reverse-linked-list-ii/） </summary> 
</details>

<details>
<summary> 18. 特定深度节点链表 (medium)（https://leetcode-cn.com/problems/list-of-depth-lcci/） </summary> 
</details>

<details>
<summary> 19. 合并两个有序链表 (easy)（https://leetcode-cn.com/problems/merge-two-sorted-lists/） </summary> 

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
//  more effective
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode *head = nullptr;
        ListNode *tail = nullptr;
        while (l1 && l2) {
            ListNode *tmpnode = nullptr;
            if (l1->val <= l2->val) {
                tmpnode = l1;
                l1 = l1->next;
            } else {
                tmpnode = l2;
                l2 = l2->next;
            }
            if (nullptr == head) {
                tail = head = tmpnode;
            } else {
                tail->next = tmpnode;
                tail = tail->next;
            }
        }
        if (nullptr == tail) {
            return (nullptr == l1) ? l2 : l1;
        }
        tail->next = (nullptr == l1) ? l2 : l1;
        return head;
    }
};
```

```cpp
// more concise
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode *head = nullptr;
        ListNode *tail = nullptr;
        while (l1 || l2) {
            ListNode *tmpnode = nullptr;
            if (!l2 || (l1 && l1->val <= l2->val)) {
                tmpnode = l1;
                l1 = l1->next;
            } else {
                tmpnode = l2;
                l2 = l2->next;
            }
            if (nullptr == head) {
                tail = head = tmpnode;
            } else {
                tail->next = tmpnode;
                tail = tail->next;
            }
        }
        return head;
    }
};
```

</details>

<details>
<summary> 20. 合并K个升序链表 (hard)（https://leetcode-cn.com/problems/merge-k-sorted-lists/） </summary> 
</details>

<details>
<summary> 21. 对链表进行插入排序 (medium)（https://leetcode-cn.com/problems/insertion-sort-list/） </summary> 
</details>

<details>
<summary> 22. 链表的中间结点 (easy)（https://leetcode-cn.com/problems/middle-of-the-linked-list/） </summary> 
</details>

<details>
<summary> 23. K 个一组翻转链表 (hard)（https://leetcode-cn.com/problems/reverse-nodes-in-k-group/） </summary> 
</details>

<details>
<summary> 24. 扁平化多级双向链表 (medium)（https://leetcode-cn.com/problems/flatten-a-multilevel-doubly-linked-list/） </summary> 
</details>

<details>
<summary> 25. 从尾到头打印链表 (easy)（https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/） </summary> 
</details>

## 树类

<details>
<summary> 1. 天际线问题 (hard)（https://leetcode-cn.com/problems/the-skyline-problem/） </summary> 
</details>


### 二叉树类
<details>
<summary> 1. 二叉树的最近公共祖先 (medium &hearts;)（https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/） </summary> 
    
```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (NULL == root || root == p || root == q) {
            return root;
        }
        TreeNode *left = lowestCommonAncestor(root->left, p, q);
        TreeNode *right = lowestCommonAncestor(root->right, p, q);
        if (left && right) {
            return root;
        }
        return  (NULL == left) ? right : left;
    }
};
```
    
</details>

<details>
<summary> 2. 平衡二叉树 (easy &hearts;)（https://leetcode-cn.com/problems/balanced-binary-tree/） </summary> 
    
```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:

    bool isBalanced(TreeNode* root, int& depth) {
        if (NULL == root) {
            depth = 0;
            return true;
        }
        int left_depth = 0;
        int right_depth = 0;
        if (isBalanced(root->left, left_depth) && isBalanced(root->right, right_depth)) {
            if ((-1 <= left_depth - right_depth) && (left_depth - right_depth <= 1)) {
                depth = max(left_depth, right_depth) + 1;
                return true;
            }
        }
        depth = max(left_depth, right_depth) + 1;
        return false;
    }

    bool isBalanced(TreeNode* root) {
        int depth = 0;
        return isBalanced(root, depth);        
    }
};
```
    
</details>

<details>
<summary> 3. 二叉搜索树的第k大节点  (easy)（https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/） </summary> 
</details>

<details>
<summary> 4. 重建二叉树 (medium)（https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/） </summary> 
</details>

<details>
<summary> 5. 二叉树的深度  (easy)（https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/） </summary> 
</details>

<details>
<summary> 6. 二叉树的序列化与反序列化  (hard)（https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/） </summary> 
</details>

<details>
<summary> 7. 二叉搜索树与双向链表 (medium)（https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/） </summary> 
</details>

<details>
<!--summary> x.  ()（） </summary--> 
</details>

## 图类
<details>
<summary> 1. 岛屿数量 (medium)（https://leetcode-cn.com/problems/number-of-islands/） </summary> 
</details>

<details>
<summary> 2. 课程表 (medium)（https://leetcode-cn.com/problems/course-schedule/） </summary> 
</details>

<details>
<summary> 3. 课程表 II (medium)（https://leetcode-cn.com/problems/course-schedule-ii/） </summary> 
</details>

<details>
<summary> 4. 单词接龙 (medium)（https://leetcode-cn.com/problems/word-ladder/） </summary> 
</details>

<details>
<summary> 5. 判断二分图 (medium)（https://leetcode-cn.com/problems/is-graph-bipartite/） </summary> 
</details>

<details>
<summary> 6. 项目管理 (hard)（https://leetcode-cn.com/problems/sort-items-by-groups-respecting-dependencies/） </summary> 
</details>

<details>
<summary> 7. 正方形数组的数目 (hard)（https://leetcode-cn.com/problems/number-of-squareful-arrays/） </summary> 
</details>

<details>
<summary> 8. 找到最终的安全状态 (medium)（https://leetcode-cn.com/problems/find-eventual-safe-states/ </summary> 
</details>

<details>
<summary> 9. 概率最大的路径 (medium)（https://leetcode-cn.com/problems/path-with-maximum-probability/） </summary> 
</details>

<details>
<summary> 10. 不邻接植花 (easy)（https://leetcode-cn.com/problems/flower-planting-with-no-adjacent/） </summary> 
</details>


## 数学类
<details>
<summary> 1. x 的平方根 (easy &hearts;) （https://leetcode-cn.com/problems/sqrtx/）</summary> 
    
```cpp
// 题解一（按位从大到小试）
class Solution {
public:
    int mySqrt(int x) {
        int ret = 0;
        // (2^16)^2 = 2^32
        // int cur_bit = 1 << ((sizeof(int) * 8) / 2);
        int cur_bit = (1 << ((sizeof(int) << 2)));
        while (cur_bit > 0) {
            ret ^= cur_bit;
            if (ret > x / ret) {
                ret ^= cur_bit;
            }
            cur_bit >>= 1; 
        }
        return ret;
    }
};
```

```cpp
// 题解二（二分查找 - fastest）
class Solution {
public:
    int mySqrt(int x) {
        int left = 1;
        int right = x;
        while (left <= right) {
            int cur = left + ((right - left) >> 1);
            int tmp_val = x / cur;
            if (cur == tmp_val) {
                return cur;
            } else if (cur > tmp_val) {
                right = cur - 1;
            } else {
                left = cur + 1;
            }
        }
        return left - 1;
        
    }
};

```

```cpp
// 题解三（牛顿迭代法）
// 参考地址：https://www.cnblogs.com/liyangguang1988/p/3617926.html
class Solution {
public:
    int mySqrt(int x) {
        /* 用牛顿迭代法求浮点数的平方根 */ 
        double g0 = 0, g1 = x;  
        while(fabs(g1 - g0) > 0.9)  
        {  
            g0 = g1;  
            g1 = (g0 + (x / g0)) / 2;
        }  
        return floor(g1); // (int)g1
    }
};
```

</details>


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

<details>
<summary> 2. LRU缓存机制 (medium &hearts;)（https://leetcode-cn.com/problems/lru-cache/） </summary> 
   
```cpp
class LRUCache {
private:
    struct DListNode {
        int key;
        int value;
        DListNode *next;
        DListNode *prev;
        DListNode(int k, int v): key(k), value(v), next(NULL), prev(NULL) {}
    };

public:
    LRUCache(int capacity)
        :capacity_(capacity)
        ,head_(new DListNode(0, 0))
        ,tail_(new DListNode(0, 0)) {
            head_->next = tail_;
            tail_->prev = head_;
        }
    ~LRUCache() {
        DListNode *node = head_;
        DListNode *post = NULL;
        while (node != NULL) {
            post = node->next;
            delete node;
            node = post;
        }
    }
    
    int get(int key) {
        auto iter = key2node_.find(key);
        if (iter == key2node_.end()) {
            return -1;
        }
        DListNode *node = iter->second;
        if (!isHead(node)) {
            removeNode(node);
            addToHead(node);
        }
        return node->value;
    }
    
    void put(int key, int value) {
        auto iter = key2node_.find(key);
        if (iter == key2node_.end()) {
            DListNode *new_node = NULL;
            if (key2node_.size() < capacity_) {
                new_node = new DListNode(key, value);
            } else {
                new_node = tail_->prev;
                key2node_.erase(tail_->prev->key);
                removeNode(tail_->prev);
                new_node->key = key;
                new_node->value = value;
            }
            addToHead(new_node);
            key2node_.insert(make_pair(key, new_node));
        } else {
            iter->second->value = value;
            if (!isHead(iter->second)) {
                removeNode(iter->second);
                addToHead(iter->second);
            }
        }
    }

private:
    bool isHead(DListNode *node) {
        return head_->next == node;
    }

    void removeNode(DListNode* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void addToHead(DListNode* node) {
        node->next = head_->next;
        head_->next->prev = node;
        node->prev = head_;
        head_->next = node;
    }
    
private:
    int capacity_;
    DListNode *head_;
    DListNode *tail_;
    unordered_map<int, DListNode*> key2node_;
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```
    
</details>



### 栈
<details>
<summary> 1. 基本计算器 II (medium)（https://leetcode-cn.com/problems/basic-calculator-ii/） </summary> 
</details>

<details>
<summary> 2. 逆波兰表达式求值 (medium)（https://leetcode-cn.com/problems/evaluate-reverse-polish-notation/） </summary> 
</details>

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
