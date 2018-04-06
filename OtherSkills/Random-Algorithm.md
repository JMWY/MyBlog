# 随机算法

## A. 随机选择

### 1. 随机选择一个数（数据总量未知，数据被选中的概率相等）
* 分析：当数据量达到 n 时，需要满足出现的所有数的概率都为 1/n.  
n = 1 时， 1/n = 1, 选择第一个数的概率为 1.  
n = 2 时， 1/n = 1/2，选择第二个数的概率为 1/2，则选择第一个数的概率为 1*(1-1/2) = 1/2.  
...  
假设当 n = k 时， 选择第 k 个数的概率为 1/k.  
那么当 n = k+1 时， 选择第 k+1 个数的概率需为 1/(k+1)，那么选择第 k 个数的概率为 (1/k)*(1-1/(k+1)) = 1/(k+1), 由此知第 k,  k+1 的数等概率选中。  
`因此得出，只要保证以 1/n 的概率选择第 n 个数，那么之前出现的 n-1 个数被选中的概率也是 1/n，所有 n 个数等概率被选中。`

```
#include <stdlib.h>

char randomChoseOneData(const char *s) {
    srand(time(NULL));
    char chose_data = '\0';
    int n = 0;
    for (;*s != '\0'; s++) {
        n++;
        if (rand() % n == 0) {
            chose_data = *s;
        }
    }
    return chose_data;
}
```
* 应用场景：数量量未知的数据流、有效内容不清楚的信息、蓄水池抽样等

### 2. 随机选择 k 个数（数据总量为 n，数据被选中的概率相等）
* 分析：<br/>
第 1 个数的概率为 k/n, 不被选中的概率是 (n-k)/n.  
第 2 个数的概率为 (k/n)*((k-1)/(n-1)) + ((n-k)/n) * (k/(n-1)) = k/n  
...
```
#include <stdlib.h>
#include <vector>

template<class T>
int randomChoseKData(const std::vector<T>& data_set, int k, std::vector<T>& chose_data_set)
{
    size_t n = data_set.size();
    if (k < 0) return -1;
    if (n < k) return -2;
    srand(time(NULL));
    chose_data_set.clear();
    chose_data_set.reserve(k);
    for (size_t i = 0; i < data_set.size(); ++i) {
        if(rand() % n < k) {
            chose_data_set.push_back(data_set[i]);
            k--;
        }
        n--;
    }
    return 0;
}
```
* 应用场景：蓄水池抽样等


### 3. 随机选择 k 个数（数据总量未知，数据被选中的概率相等）
* 方法:<br/>
前 k 个直接放入，之后的数，比如第 i（i>k） 个数， 会按概率 k/i 替换掉之前已选的 k 个数之一（即，每个数被替换掉的概率为 1/k）。
* 分析:<br/>
读取到第 k+1 个元素时，被选中的概率为 k/(k+1)，同时对前 k 个元素来说，任一个元素会被替换掉的概率为 1/k ，则保留下来的概率为 1-(k/(k+1))*(1/k) = k/(k+1)。由此知道，所有元素会被等概率选中。
假设读取到第 i(i>k) 个元素时，被选中的概率为 k/i.  
读取到第 i+1 个元素时，被选中的概率为 k/(i+1), 则对前 i 个元素来说，任一个元素被选中，且不替换掉的概率为 (k/i)*(1-(k/(i+1))*(1/k))=k/(i+1)。得证。
```
#include <stdlib.h>
#include <vector>

template<typename T>
int randomChoseKData(const std::vector<T>& data_set, int k, std::vector<T>& chose_data_set)
{
    if (k < 0) return -1;
    srand(time(NULL));
    chose_data_set.clear();
    chose_data_set.reserve(k);
    int n = 0;
    typename std::vector<T>::const_iterator iter = data_set.begin();
    for (; iter != data_set.end(); iter++, ++n) {
        if (n < k) {
            chose_data_set.push_back(*iter);
            continue;
        }
        int mod = rand() % n;
        if(mod < k) {
            chose_data_set[mod] = *iter; 
        }
    }
    return 0;
}
```


## B. 随机排列

### 1. 数组完全随机排列（数组大小为 n）

* Knuth-Durstenfeld Shuffle (时间: O(n), 空间: O(1))

```
RANDOMIZE-IN-PLACE ( A , n ) for i ← 1 to n do swap A[i] ↔ A[RANDOM(i , n )]
```
* 证明：<br/>
1. 对任一个元素，放在位置 1 的位置的概率为 1/n ， 放在位置 2 的概率为 ((n-1)/n)*（1/(n-1)）=1/n, 放在位置 3 的概率 ..., 得出任一元素放在任一位置的概率都为 1/n.
2. 位置 1 从 n 个元素中选，有 n 种可能，位置 2 从剩下的 n-1 个元素中选，有 n-1 种可能，位置 3 ....,得出总共可能得到的排列数为: n*(n-1)*(n-2)*...*1 = n! 种 。正好为 n 个数的全排列数。
由上两条得证。

```
template <typename T>
void shuffle(std::vector<T>& data) {
    srand(time(NULL));
    size_t n = data.size();
    while (n > 0) {
        size_t i = rand() % n;
        if (--n != i) {
            std::swap(data[n], data[i]);
        }
    }
}
```

* 应用场景： 知道总牌数的洗牌算法等。

### 2. 数组完全随机排列（数组大小未知）

* Inside-Out Algorithm（时间：O(k), 空间: O(1)， k 为出现的数据量）

* 方法:<br/>
用新数组保存每个出现的数据。当有新数据出现时，保存新数据后，再从全部历史数据中随机抽一个和新数据互换位置。

* 证明:<br/>
当全部 k 个元素出现完毕时，
1. 第 i 个元素放在前 i 个位置的概率为：(1/i)*(i/(i+1))*((i+1)/(i+2))*...*((k-1)/k) = 1/k.  
2. 第 i 个元素放在第 j(j>i) 位置的概率为：(1/j)*(j/(j+1))*...*((k-1)/k) = 1/k.  

```
#include <stdlib.h>
#include <vector>
#include <algorithm>

template <typename T>
void shuffle(std::vector<T>& data, size_t k, std::vector<T>& result) {
    srand(time(NULL));
    for (size_t i = 0; i < std::min(k, data.size()); ++i) {
        result.push_back(data[i]);
        int j = rand() % (i + 1);
        if (j != i) {
            result[i] = result[j];
            result[j] = data[i];
        }
    }
} 
```

* 应用场景： 数量未知的洗牌 (即：来一张，洗一张)。





