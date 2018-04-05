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
* 应用场景：数量量未知的数据流、有效内容不清楚的信息等

### 2. 随机选择 k 个数（数据总量为 n，数据被选中的概率相等）
* 分析：
第 1 个数的概率为 m/n, 不被选中的概率是 (n-m)/n.
第 2 个数的概率为 (m/n)*((m-1)/(n-1)) + ((n-m)/n) * (m/(n-1)) = m/n
...
```
#include <stdlib.h>
#include <vector>

template<class T>
int randomChoseKData(const std::vector<T>& data_set, int k, std::vector<T>& chose_data_set)
{
    srand(time(NULL));
    size_t n = data_set.size();
    if (k < 0) return -1;
    if (n < k) return -2;
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
