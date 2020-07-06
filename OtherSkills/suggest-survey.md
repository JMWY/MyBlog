# 输入自动提示实现方式调研

2019 综述文章：https://zepworks.com/posts/you-autocomplete-me/

## Trie
实现简单，但浪费空间。
Double-Array Trie （indexlib 内的实现）
数据压缩在 base, check 两个数据中:

> base[t] + (code of 'r') = base[t->r] <br/>
> check[t->r] = base[t]

可以看出：base 主要用于查找子节点，check 向上回溯父节点。
结论：难以凭前缀实现子节点遍历。一般用于查找禁词等
* 参考文档：https://linux.thai.net/~thep/datrie/datrie.html

## Ternary Search Trees(TST)
```python
class TSTree(object): 
    def __init__(self):
        self.root = None
        self.resultSet = None
    def _get(self, node, keys):
        key = keys[0]
        if key < node.key:
            return self._get(node.low, keys)
        elif key == node.key:
            if len(keys) > 1:
                return self._get(node.eq, keys[1:])
            else:
                return node.value
        else:
            return self._get(node.high, keys)
    def get(self, key):
        keys = [x for x in key]
        return self._get(self.root, keys)
    def _set(self, node, keys, value):
        next_key = keys[0]
        if not node:
            node = TSTreeNode(next_key, None, None, None, None)
        if next_key < node.key:
            node.low = self._set(node.low, keys, value)
        elif next_key == node.key:
            if len(keys) > 1:
                node.eq = self._set(node.eq, keys[1:], value)
            else:
                # leaf node
                node.value = value
        else:
            node.high = self._set(node.high, keys, value)
        return node
    def set(self, key, value):
        keys = [x for x in key]
        self.root = self._set(self.root, keys, value)
    def _find(self, prefix):
        if len(prefix) == 0:
            return self.root
        idx = 0
        node = self.root
        while idx < len(prefix):
            if not node:
                return None
            key = prefix[idx]
            if key < node.key:
                node = node.low
            elif key > node.key:
                node = node.high
            else:
                if idx < len(prefix) - 1:
                    idx += 1
                    node = node.eq
                else:
                    if node.value:
                        self.resultSet[prefix] = node.value
                    return node.eq
    def _traverse_find(self, prefix, node):
        if node:
            if node.value is not None:
                self.resultSet[prefix+node.key] = node.value
            self._traverse_find(prefix, node.low)
            self._traverse_find(prefix+node.key, node.eq)
            self._traverse_find(prefix, node.high)
    def find(self, prefix):
        self.resultSet = {}
        subRoot = self._find(prefix)
        self._traverse_find(prefix, subRoot)
        return self.resultSet
if __name__ == '__main__':
    tree = TSTree()
    tree.set('AB', 1)
    tree.set('BCD', 2)
    tree.set('BCCD', 3)
    tree.set('A', 4)
    tree.set('ABCD', 5)
    tree.set('ABBA', 6)
    print tree.find('AB')
    print tree.find('A')
    print tree.find('B')    
        
$ python TSTree.py
{'ABCD': 5, 'AB': 1, 'ABBA': 6}
{'A': 4, 'ABCD': 5, 'AB': 1, 'ABBA': 6}
{'BCD': 2, 'BCCD': 3}
```

* trie 树演进，节省内存，能实时更新
* 树的构建和输入顺序有关
* 参考文档：

（作者）http://igoro.com/archive/efficient-auto-complete-with-a-ternary-search-tree/ 
（应用）https://www.cnblogs.com/lexus/archive/2011/10/21/2219976.html

## finite state machine（FST）
* 结构复杂，消耗内存少
参考文档：
(lucene 方案)http://blog.mikemccandless.com/2010/12/using-finite-state-transducers-in.html 
(C++, paper) https://xlinux.nist.gov/dads/HTML/finiteStateMachine.html
(python) https://www.python-course.eu/finite_state_machine.php
A segment tree based Top-k RMQ algorithm
原理：
* 建库
1. 将权重，query 存入一个 vector
2. vector 按 query 排序
* 查找
1. 根据 prefix 二分查找，找出 vector 中前缀匹配的区间集合
2. 利用 heap+ seqment_tree 按如下逻辑，查找出权重 top K 的 query
     
    
    
paper: http://dhruvbird.com/autocomplete.pdf
code: https://github.com/dsindex/autocomplete_libface
参考文档：https://pdfs.semanticscholar.org/a817/ffa96660370bdf9d18fe01810dc0bb2967a2.pdf

## Lucene — suggest module
http://lucene.apache.org/core/4_4_0/suggest/index.html
数据结构：
* TST ：
* FST ：http://blog.mikemccandless.com/2010/12/using-finite-state-transducers-in.html
基于 FST 的分析器：
* AnalyzingSuggester ：  单元测试代码   ： 相关介绍 & QA
* FuzzySuggester 
## Solr—— Suggester
https://wiki.apache.org/solr/Suggester
* JaspellLookup
* TSTLookup
* FSTLookup
* WFSTLookup
## ES Suggester 接口情况
https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-completion.html
Google 自动补全策略（https://support.google.com/websearch/answer/7368877）
