# python & C++ 混合编程

## 1. python 调用 C 库

```
// example.h
#pragma once
int plus(int v1, int v2);
int sum(int A[], size_t n);
```
```
// example.c
#include <stdio.h>

#define DEBUG                   \
    printf("debug\t%s:%s:%d\n", \
           __FILE__, __func__, __LINE__)

int plus(int v1, int v2)
{
    DEBUG;
    return v1 + v2;
}

int sum(int A[], size_t n) {
    DEBUG;
    int sum = n > 0 ? A[0] : 0x80000000;
    size_t i = 1;
    for (; i < n; ++i) {
        sum += A[i];
    }
    return sum;
}
```

* compile <br/>
`$ gcc example.c -fPIC -shared -o example.so`

* run <br/>

```
>>> from ctypes import cdll
>>> e = cdll.LoadLibrary('./example.so')
>>> e.plus(10, 10)
debug	example.c:plus:10
20
>>> import ctypes
>>> py_A = [1, 2, 3, 4, 5]
>>> c_A = (ctypes.c_int * len(py_A))(*py_A)
>>> e.sum(c_A, len(c_A))
debug	example.c:sum:15
15
```

* [ctypes](https://docs.python.org/2/library/ctypes.html) mapping

 |  ctypes | C type | Python type  |
 | :-----: | :------: | :------:  | 
 |  c_bool | _Bool | bool (1)  |
 |  c_char | char | 1-character string  | 
 |  c_wchar | wchar_t | 1-character unicode string  | 
 |  c_byte | char | int/long  | 
 |  c_ubyte | unsigned | char | int/long  | 
 |  c_short | short | int/long  | 
 |  c_ushort | unsigned | short | int/long  | 
 |  c_int | int | int/long  | 
 |  c_uint | unsigned | int | int/long  | 
 |  c_long | long | int/long  | 
 |  c_ulong | unsigned | long | int/long  | 
 |  c_longlong | __int64 | or | long | long | int/long  | 
 |  c_ulonglong | unsigned __int64 or unsigned long long | int/long  | 
 |  c_float | float | float  | 
 |  c_double | double | float  | 
 |  c_longdouble | long double | float  | 
 |  c_char_p | char * (NUL | terminated) | string or None  | 
 |  c_wchar_p | wchar_t * (NUL terminated) | unicode or None  | 
 |  c_void_p | void * | int/long or None++  |



#### Alternative：using [swig](http://www.swig.org/Doc3.0/Introduction.html#Introduction_nn10) skill <br/>
* [example](http://www.swig.org/tutorial.html)

<br/>


## 2. python 调用 C++ 库

```
// kmodel.hpp
#ifndef KMODEL_H
#define KMODEL_H

#include <string>
#include <iostream>
#include <unordered_map>
#include <stdint.h>

class KModel{
public:
    KModel(){
        umap["北京"] = 1024;
        umap["上海"] = 2048;
    }
    std::unordered_map<std::string,int> umap; 
    void say(std::string v);
};
#endif
```
```
// kmodel.cpp
#include "kmodel.hpp"
using namespace std;

void KModel::say(string v){
    cout<<umap[v]<<endl;
}
```
* using swig, edit .i file <br/>
```
// kmodel.i
%module kmodel 
%include "std_string.i"
%{
    #include "kmodel.hpp"
%}
%include "kmodel.hpp"
```

* edit file setup.py <br/>
```
"""
setup.py
"""
from distutils.core import setup, Extension
setup(
    name = "kmodel",
    version = "1.0", 
    py_modules=['kmodel'],
    ext_modules = [
        Extension("_kmodel",
        ["kmodel_wrap.cxx", "kmodel.cpp"],
        extra_compile_args = ['-std=c++11'])]
)
```

* compile <br/>
`$ swig -c++ -python kmodel.i`
`python setup.py build` // or python setup.py install

* run (cd ./build/lib.linux-x86_64-2.7/) <br/>
```
>>> from kmodel import *
>>> k=KModel()
>>> k.say("北京")
1024
>>> k.say("上海")
2048
```

* c++ head file mapping <br/>

| C++ head file  | SWIG Interface head file |
| :------------: | :---------------------:  | 
|  deque         | std_deque.i              | 
|  list          | std_list.i               | 
|  map           | std_map.i                | 
|  utility       | std_pair.i               | 
|  set           | std_set.i                | 
|  string        | std_string.i             | 
|  vector        | std_vector.i             |


#### more about [SWIG and C++11](http://www.swig.org/Doc3.0/CPlusPlus11.html#CPlusPlus11)


## [C/C++ 调用 Python](https://www.zhihu.com/question/23003213)




