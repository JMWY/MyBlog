# LeedCode 梳理

| 编号  | 题目      |题解             |  地址 |  
|:--:|:--:|:--:|:--:|
|a||b|
| 1|A|B|C|

## 1533. 找最大数的索引
- 题目： 数组中有个最大数，其它数都一样，找出这个最大数。要求只能掉 API:<br/>
a. int compareSub(int l, int r, int x, int y). <br/>
1 if arr[l]+arr[l+1]+...+arr[r] > arr[x]+arr[x+1]+...+arr[y].<br/>
0 if arr[l]+arr[l+1]+...+arr[r] == arr[x]+arr[x+1]+...+arr[y].<br/>
-1 if arr[l]+arr[l+1]+...+arr[r] < arr[x]+arr[x+1]+...+arr[y].<br/>

b. int length(): Returns the size of the array.<br/>

- 举例
- 题解
