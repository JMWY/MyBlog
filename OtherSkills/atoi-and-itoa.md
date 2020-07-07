# 数字和字符串之间的转换

## atoi
相关函数：atoi，atol，strtod，strtol, stroul
头文件： #include <stdlib.h>





## itoa
相关函数：gcvt，ecvt，fcvt，sprintf，std::to_string, std::stringstream(类)

```c
#include <sstream>
#include <string>
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <iostream>
#include <sys/time.h>
#include <math.h>

#define nullptr NULL

uint32_t get_number_length(uint64_t value) {
    // method 1
    // #define ALIGNED(size) __attribute__((__aligned__(size)))
    // #define LIKELY(x)   (__builtin_expect((x), 1))
    // #define UNLIKELY(x) (__builtin_expect((x), 0))
    // #define LIKELY(x)   (x)
    // #define UNLIKELY(x) (x)
    //     static const uint64_t powersOf10[20] ALIGNED(64) = {1, 10, 100, 1000, 10000, 100000, 1000000, 10000000, 100000000, 1000000000, 10000000000, 100000000000, 1000000000000, 10000000000000, 100000000000000, 1000000000000000, 10000000000000000, 100000000000000000, 1000000000000000000, 10000000000000000000UL};
    // 
    //     if UNLIKELY(!value) { return 1; }
    //     const uint8_t leadingZeroes = __builtin_clzll(value);
    //     const uint32_t bits = 63 - leadingZeroes;
    //     const uint32_t minLength = 1 + ((bits * 77) >> 8);
    //     return minLength + (uint32_t) (UNLIKELY(value >= powersOf10[minLength]));
    // method 2
    uint32_t result = 1;
    do {
        if (value < 10)  return result;
        if (value < 100)  return result + 1;
        if (value < 1000)  return result + 2;
        if (value < 10000)  return result + 3;
        value /= 10000;
        result += 4;
    } while (true);
    return result;
    // method 3 (bad performance)
    // return log10(value) + 1;
}

uint32_t uint64ToBuffer(uint64_t value, char* const buffer, uint32_t len) {
    uint32_t val_len = get_number_length(value);
    if (val_len >= len) {
        return 0;
    }
    buffer[val_len] = '\0';
    uint32_t pos = val_len - 1;
    while (value >= 10) {
        buffer[pos--] = value % 10 + '0';
        value /= 10; 
    }   
    buffer[pos] = value + '0';
    return val_len;
}

std::string use_myfunc(int a) {
    char buf[64];
    uint64ToBuffer((uint64_t)a, buf, sizeof(buf));
    // std::cout << buf << std::endl;
    return buf;
}

std::string use_std_to_string(int a) {
    return std::to_string(a);
}
std::string use_snprintf(int a) {
    char buf[64];
    snprintf(buf, sizeof(buf), "%d", a);
    return buf;
}

std::string use_stringstream(int a) {
    std::ostringstream oss;
    oss << a;
    return oss.str();
}

const int LOOPS = 1000000;

void *thread(void *p) {
    std::string (*foo)(int) = (std::string (*)(int))p;
    for (int i = 1; i < LOOPS; ++i)
        foo(i + 1);
    return p;
}

double run_with_threads(int threads, std::string (*foo)(int)) {
    timeval start, end;
    gettimeofday(&start, nullptr);

    pthread_t *tids = new pthread_t[threads];
    for (int i = 0; i < threads; ++i)
        pthread_create(&tids[i], nullptr, thread, (void *)foo);
    for (int i = 0; i < threads; ++i)
        pthread_join(tids[i], nullptr);
    delete[] tids;

    gettimeofday(&end, nullptr);

    return (end.tv_sec - start.tv_sec) + (end.tv_usec - start.tv_usec) * 1e-6;
}

void test_with_threads(int threads) {
    printf("%d threads:\n", threads);
    double time_snprintf = run_with_threads(threads, use_snprintf);
    double time_stringstream = run_with_threads(threads, use_stringstream);
    double time_myfunc = run_with_threads(threads, use_myfunc);
    double time_std_to_string = run_with_threads(threads, use_std_to_string);
    printf("snprintf:        %fs\n", time_snprintf);
    printf("stringstream:    %fs\n", time_stringstream);
    printf("std::to_string:  %fs\n", time_std_to_string);
    printf("myfunc:          %fs\n", time_myfunc);
    printf("\n");
}

int main(int argc, char **argv) {
    if (argc > 1) {
        test_with_threads(atoi(argv[1]));
    } else {
        test_with_threads(1);
        test_with_threads(2);
        test_with_threads(4);
        test_with_threads(10);
        test_with_threads(20);
    }
}
```
![](https://github.com/JMWY/MyBlog/blob/master/OtherSkills/images/itoa_benchmark.png "benchmark")

```c
liyangguang@bogon Test]$ g++ -std=c++11 -O2 -o test_itoa test_itoa.cpp -lpthread
liyangguang@bogon Test]$ ./test_itoa 
1 threads:
snprintf:        0.085412s
stringstream:    0.239162s
std::to_string:  0.086575s
myfunc:          0.021520s

2 threads:
snprintf:        0.075452s
stringstream:    0.445572s
std::to_string:  0.087610s
myfunc:          0.019894s

4 threads:
snprintf:        0.076779s
stringstream:    0.820015s
std::to_string:  0.087140s
myfunc:          0.020007s

10 threads:
snprintf:        0.136183s
stringstream:    1.751497s
std::to_string:  0.155591s
myfunc:          0.029591s

20 threads:
snprintf:        0.270698s
stringstream:    3.447055s
std::to_string:  0.315181s
myfunc:          0.054659s
```

## 开发工具库

头文件：
```c
/*================================================================ 
*   Copyright (C) 2020-07 All rights reserved.
*   文件名称：number_util.h
*   创 建 者：chenfeng
*   创建日期：2020年07月07日
*   描    述：对数字的处理的函数集
================================================================*/

#ifndef __NUMBER_UTIL_H__
#define __NUMBER_UTIL_H__

#include <stdint.h>
#include <assert.h>
#include <string>
#include <exception>

namespace myutil {

class NumberUtil
{
public:
	template < typename T>
    static bool strToNumber(const char* str, T & value);

	template < typename T>
    static T strToNumberWithDef(const char* str, T  defaultValue);

	template < typename T>
    static T deserialize(const std::string& str);

	template < typename T>
    static void serialize(T value, std::string& str);

    template <typename T>
    static std::string numberToString(T val);

    template <typename T>
    static void appendNumberToString(T val, std::string& s);
    
public:

    static bool hexStrToUint64(const char* str, uint64_t& value);
    static void uint64ToHexStr(uint64_t value, char* hexStr, int len);
    /**
     * 统计size_t类型的整数编码中有多少个1
     * add by youtai.xcf 20190529
     * @param n  1的个数
     * @return
     */
    static inline int number_of_one(size_t n) {
        int resu = 0;
        while(n != 0){
            resu++;
            n = (n-1)&n;
        }
        return resu;
    }

    /**
     * 计算数据位数
     * @param value 要判断的正整数
     * @return  value 是几位数
     */
    static inline uint32_t get_number_length(uint64_t value) {
        uint32_t result = 1;
        do {
            if (value < 10)  return result;
            if (value < 100)  return result + 1;
            if (value < 1000)  return result + 2;
            if (value < 10000)  return result + 3;
            value /= 10000;
            result += 4;
        } while (true);
        return result;
    }

    /**
     * 正整数转换为字符串
     * @param value 要转换的正整数
     * @param buffer 字符串写入的起始地址
     * @param len buffer 可用字节长度
     * @return 转换字符串长度
     */
    static inline uint32_t uint64ToBuffer(uint64_t value, char* const buffer, uint32_t len) {
        uint32_t val_len = get_number_length(value);
        if (val_len >= len) {
            return 0;
        }
        buffer[val_len] = '\0';
        uint32_t pos = val_len - 1;
        while (value >= 10) {
            buffer[pos--] = value % 10 + '0';
            value /= 10;
        }
        buffer[pos] = value + '0';
        return val_len;
    }


private:
    static inline bool strToInt32(const char* str, int32_t& value);
    static inline bool strToUInt32(const char* str, uint32_t& value);
};

template < typename T>
bool NumberUtil::strToNumber(const char* str, T & value)
{
	return false;
}

template < typename T>
T strToNumberWithDef(const char* str, T  defaultValue)
{
	return defaultValue;
}

template < typename T>
T NumberUtil::deserialize(const std::string& str)
{
    assert(str.length() == sizeof(T));

    T value= 0;
    for (size_t i = 0; i < str.length(); ++i)
    {
        value <<= 8;
        value |= (unsigned char)str[i];
    }
    return value;
}

template < typename T>
void NumberUtil::serialize(T value, std::string& str)
{
    char key[sizeof(T)];
    for (int i = (int)sizeof(T) - 1; i >= 0; --i)
    {
        key[i] = (char)(value & 0xFF);
        value >>= 8;
    }
    str.assign(key, sizeof(T));
}

template <typename T>
std::string NumberUtil::numberToString(T val) {
	// throw std::exception("unsupport type");
    return std::to_string(val);
}

template <typename T>
void NumberUtil::appendNumberToString(T val, std::string& s) {
    s.append(numberToString(val));
}

template < >
std::string NumberUtil::numberToString(uint64_t value);

template < >
std::string NumberUtil::numberToString(int64_t value);

#define NUMBER_TO_STRING_DECLARE(type)              \
template < >                                        \
std::string NumberUtil::numberToString(type value);

NUMBER_TO_STRING_DECLARE(uint64_t)
NUMBER_TO_STRING_DECLARE(int64_t)
NUMBER_TO_STRING_DECLARE(int32_t)
NUMBER_TO_STRING_DECLARE(uint32_t)
NUMBER_TO_STRING_DECLARE(int16_t)
NUMBER_TO_STRING_DECLARE(uint16_t)
NUMBER_TO_STRING_DECLARE(int8_t)
NUMBER_TO_STRING_DECLARE(uint8_t)
NUMBER_TO_STRING_DECLARE(double)
NUMBER_TO_STRING_DECLARE(float)

#define APPEND_NUMBER_TO_STRING_DECLARE(type)                               \
template < >                                                                \
void NumberUtil::appendNumberToString(type value, std::string& s); 

// double, float 不特化
APPEND_NUMBER_TO_STRING_DECLARE(uint64_t)
APPEND_NUMBER_TO_STRING_DECLARE(int64_t)
APPEND_NUMBER_TO_STRING_DECLARE(int32_t)
APPEND_NUMBER_TO_STRING_DECLARE(uint32_t)
APPEND_NUMBER_TO_STRING_DECLARE(int16_t)
APPEND_NUMBER_TO_STRING_DECLARE(uint16_t)
APPEND_NUMBER_TO_STRING_DECLARE(int8_t)
APPEND_NUMBER_TO_STRING_DECLARE(uint8_t)

template < > bool NumberUtil::strToNumber(const char* str, int8_t & value);
template < > bool NumberUtil::strToNumber(const char* str, uint8_t & value);
template < > bool NumberUtil::strToNumber(const char* str, int16_t & value);
template < > bool NumberUtil::strToNumber(const char* str, uint16_t & value);
template < > bool NumberUtil::strToNumber(const char* str, int32_t & value);
template < > bool NumberUtil::strToNumber(const char* str, uint32_t & value);

template < > bool NumberUtil::strToNumber(const char* str, int64_t & value);
template < > bool NumberUtil::strToNumber(const char* str, uint64_t & value);

template < > bool NumberUtil::strToNumber(const char* str, float & value);
template < > bool NumberUtil::strToNumber(const char* str, double & value);

template < > int8_t NumberUtil::strToNumberWithDef(const char* str, int8_t defValue);
template < > uint8_t NumberUtil::strToNumberWithDef(const char* str, uint8_t defValue);
template < > int16_t NumberUtil::strToNumberWithDef(const char* str, int16_t defValue);
template < > uint16_t NumberUtil::strToNumberWithDef(const char* str, uint16_t defValue);
template < > int32_t NumberUtil::strToNumberWithDef(const char* str, int32_t defValue);

template < > uint32_t NumberUtil::strToNumberWithDef(const char* str, uint32_t defValue);
template < > int64_t NumberUtil::strToNumberWithDef(const char* str, int64_t defValue);

template < > uint64_t NumberUtil::strToNumberWithDef(const char* str, uint64_t defValue);
template < > float NumberUtil::strToNumberWithDef(const char* str, float defValue);
template < > double NumberUtil::strToNumberWithDef(const char* str, double defValue);

}


#endif 
```
cpp 文件：
```
#include <errno.h>
#include <stdint.h>
#include <assert.h>
#include <stdlib.h>
#include <stdio.h>
#include "number_util.h"

#define LIKELY(x)   (x)
#define UNLIKELY(x) (x)

namespace myutil
{

inline bool NumberUtil::strToInt32(const char* str, int32_t& value)
{
    if (UNLIKELY(NULL == str || *str == 0))
    {
        return false;
    }
    char* endPtr = NULL;
    errno = 0; // global variable, may be not's thread_safe

# if __WORDSIZE == 64
    int64_t value64 = strtol(str, &endPtr, 10);
    if (UNLIKELY(value64 < INT32_MIN || value64 > INT32_MAX))
    {
        return false;
    }
    value = (int32_t)value64;
# else
    value = (int32_t)strtol(str, &endPtr, 10);
# endif

    if (LIKELY(errno == 0 && endPtr && *endPtr == 0))
    {
        return true;
    }
    return false;
}

inline bool NumberUtil::strToUInt32(const char* str, uint32_t& value)
{
    if (UNLIKELY(NULL == str || *str == 0 || *str == '-'))
    {
        return false;
    }
    char* endPtr = NULL;
    errno = 0;

# if __WORDSIZE == 64
    uint64_t value64 = strtoul(str, &endPtr, 10);
    if (UNLIKELY(value64 > UINT32_MAX))
    {
        return false;
    }
    value = (int32_t)value64;
# else
    value = (uint32_t)strtoul(str, &endPtr, 10);
# endif

    if (LIKELY(errno == 0 && endPtr && *endPtr == 0))
    {
        return true;
    }
    return false;
}

template < > bool NumberUtil::strToNumber(const char* str, int8_t & value)
{
	int32_t v32=0;
	bool ret = strToInt32(str, v32);
	if(UNLIKELY(!ret))
		return false;

	if (LIKELY(v32 >= INT8_MIN && v32 <= INT8_MAX))
	{
		value = (int8_t)v32;
		return true;
	}
	return false;
}
template < > bool NumberUtil::strToNumber(const char* str, uint8_t & value)
{
	uint32_t v32=0;
	bool ret = strToUInt32(str, v32);
	if(UNLIKELY(!ret))
		return false;
	if (LIKELY(v32 <= UINT8_MAX))
	{
		value = (uint8_t)v32;
		return true;
	}
	return false;
}
template < > bool NumberUtil::strToNumber(const char* str, int16_t & value)
{
	int32_t v32=0;
	bool ret = strToInt32(str, v32);
	if(UNLIKELY(!ret))
		return false;
	if (LIKELY(v32 >= INT16_MIN && v32 <= INT16_MAX))
	{
		value = (int16_t)v32;
		return true;
	}
	return false;
}
template < > bool NumberUtil::strToNumber(const char* str, uint16_t & value)
{
    uint32_t v32=0;
    bool ret = strToUInt32(str, v32);
	if(UNLIKELY(!ret))
		return false;
	if (LIKELY(v32 <= UINT16_MAX))
	{
		value = (uint16_t)v32;
		return true;
	}
	return false;
}
template < > bool NumberUtil::strToNumber(const char* str, int32_t & value)
{
	return strToInt32(str, value);
}
template < > bool NumberUtil::strToNumber(const char* str, uint32_t & value)
{
	return strToUInt32(str, value);
}


template < > bool NumberUtil::strToNumber(const char* str, int64_t & value)
{
    if (UNLIKELY(NULL == str || *str == 0))
    {
        return false;
    }
    char* endPtr = NULL;
    errno = 0;
    value = (int64_t)strtoll(str, &endPtr, 10);
    if (LIKELY(errno == 0 && endPtr && *endPtr == 0))
    {
        return true;
    }
    return false;
}

template < > bool NumberUtil::strToNumber(const char* str, uint64_t & value)
{
    if (UNLIKELY(NULL == str || *str == 0 || *str == '-'))
    {
        return false;
    }
    char* endPtr = NULL;
    errno = 0;
    value = (uint64_t)strtoull(str, &endPtr, 10);
    if (LIKELY(errno == 0 && endPtr && *endPtr == 0))
    {
        return true;
    }
    return false;
}


template < > bool NumberUtil::strToNumber(const char* str, float & value)
{
    if (UNLIKELY(NULL == str || *str == 0))
    {
        return false;
    }
    errno = 0;
    char* endPtr = NULL;
    value = strtof(str, &endPtr);
    if (LIKELY(errno == 0 && endPtr && *endPtr == 0))
    {
        return true;
    }
    return false;
}

template < > bool NumberUtil::strToNumber(const char* str, double & value)
{
	if (UNLIKELY(NULL == str || *str == 0))
    {
        return false;
    }
    errno = 0;
    char* endPtr = NULL;
    value = strtod(str, &endPtr);
    if (LIKELY(errno == 0 && endPtr && *endPtr == 0))
    {
        return true;
    }
    return false;
}


template < > int8_t NumberUtil::strToNumberWithDef(const char* str, int8_t defValue)
{
    int8_t tmp;
    if(LIKELY(strToNumber(str, tmp)))
    {
        return tmp;
    }
    return defValue;
}

template < > uint8_t NumberUtil::strToNumberWithDef(const char* str, uint8_t defValue)
{
    uint8_t tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}

template < > int16_t NumberUtil::strToNumberWithDef(const char* str, int16_t defValue)
{
    int16_t tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}

template < > uint16_t NumberUtil::strToNumberWithDef(const char* str, uint16_t defValue)
{
    uint16_t tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}

template < > int32_t NumberUtil::strToNumberWithDef(const char* str, int32_t defValue)
{
    int32_t tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}

template < > uint32_t NumberUtil::strToNumberWithDef(const char* str, uint32_t defValue)
{
    uint32_t tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}


template < > int64_t NumberUtil::strToNumberWithDef(const char* str, int64_t defValue)
{
    int64_t tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}

template < > uint64_t NumberUtil::strToNumberWithDef(const char* str, uint64_t defValue)
{
    uint64_t tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}

template < > float NumberUtil::strToNumberWithDef(const char* str, float defValue)
{
    float tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}

template < > double NumberUtil::strToNumberWithDef(const char* str, double defValue)
{
    double tmp;
    if(strToNumber(str, tmp))
    {
        return tmp;
    }
    return defValue;
}


bool NumberUtil::hexStrToUint64(const char* str, uint64_t& value)
{
    if (UNLIKELY(NULL == str || *str == 0))
     {
         return false;
     }
     char* endPtr = NULL;
     errno = 0;
     value = (uint64_t)strtoull(str, &endPtr, 16);
     if (LIKELY(errno == 0 && endPtr && *endPtr == 0))
     {
         return true;
     }
     return false;
}


void NumberUtil::uint64ToHexStr(uint64_t value, char* hexStr, int len)
{
    assert(len > 16);
    snprintf(hexStr, len, "%016llx", value);
}

template < >
std::string NumberUtil::numberToString(uint64_t value) {
    char buff[128];
    uint32_t len = uint64ToBuffer(value, buff, sizeof(buff));
    return std::string(buff, len);
}

template < >
std::string NumberUtil::numberToString(int64_t value) {
    if (value < 0) {
        if ((value ^ 0x8000000000000000) == 0) {
            return std::string("-9223372036854775808");
        } else {
            value *= -1;
            return "-" + numberToString(static_cast<uint64_t>(value));
        }
    } else {
        return numberToString(static_cast<uint64_t>(value));
    }
}

#define NUMBER_TO_STRING_DEFINE(type)                        \
template < >                                                 \
std::string NumberUtil::numberToString(type value) {         \
    return numberToString(static_cast<int64_t>(value));      \
}
NUMBER_TO_STRING_DEFINE(int32_t)
NUMBER_TO_STRING_DEFINE(uint32_t)
NUMBER_TO_STRING_DEFINE(int16_t)
NUMBER_TO_STRING_DEFINE(uint16_t)
NUMBER_TO_STRING_DEFINE(int8_t)
NUMBER_TO_STRING_DEFINE(uint8_t)

template < >
std::string NumberUtil::numberToString(double dvalue) {
    char buff[128];
    int len = snprintf(buff, sizeof(buff), "%.6f", dvalue);
    
    if (0 < len && len < (int)sizeof(buff)) {
        buff[len] = '\0';
        return std::string(buff, len);
    }
    return std::string("");
}

template < >
std::string NumberUtil::numberToString(float dvalue) {
    return numberToString(static_cast<double>(dvalue));
}

template < >
void NumberUtil::appendNumberToString(uint64_t value, std::string& s) {
    uint32_t val_len = get_number_length(value);
    s.resize(s.size()+val_len, '0');
    uint32_t pos = s.size() - 1;
    while (value >= 10) { s[pos--] += value % 10;
        value /= 10;
    }
    s[pos] += value;
}

template < >
void NumberUtil::appendNumberToString(int64_t value, std::string& s) {
    if (value < 0) {
        if ((value ^ 0x8000000000000000) == 0) {
            s.append("-9223372036854775808");
            return;
        }
        s.push_back('-');
        value *= -1;
    }
    uint32_t val_len = get_number_length((uint64_t)value);
    s.resize(s.size()+val_len, '0');
    uint32_t pos = s.size() - 1;
    while (value >= 10) {
        s[pos--] += value % 10;
        value /= 10;
    }
    s[pos] += value;
}

#define APPEND_NUMBER_TO_STRING_DEFINE(type)                                \
template < >                                                                \
void NumberUtil::appendNumberToString(type value, std::string& s) {         \
    appendNumberToString(static_cast<int64_t>(value), s);                   \
}
APPEND_NUMBER_TO_STRING_DEFINE(int32_t)
APPEND_NUMBER_TO_STRING_DEFINE(uint32_t)
APPEND_NUMBER_TO_STRING_DEFINE(int16_t)
APPEND_NUMBER_TO_STRING_DEFINE(uint16_t)
APPEND_NUMBER_TO_STRING_DEFINE(int8_t)
APPEND_NUMBER_TO_STRING_DEFINE(uint8_t)

}
```

测试：
```
#include "number_util.h"

std::string use_myfunc(int a) {
	std::string buf = myutil::NumberUtil::numberToString(a);
    // std::cout << buf << std::endl;
    return buf;
}
```
```
liyangguang@bogon MyBlog(master*)]$ g++ -std=c++11 -O2 -o test_itoa test_itoa.cpp number_util.cpp -lpthread
liyangguang@bogon MyBlog(master*)]$ ./test_itoa 
1 threads:
snprintf:        0.085493s
stringstream:    0.229670s
std::to_string:  0.085335s
myfunc:          0.019190s

2 threads:
snprintf:        0.076141s
stringstream:    0.444985s
std::to_string:  0.085224s
myfunc:          0.018168s

4 threads:
snprintf:        0.076703s
stringstream:    0.826637s
std::to_string:  0.086076s
myfunc:          0.018535s

10 threads:
snprintf:        0.137665s
stringstream:    1.681481s
std::to_string:  0.155340s
myfunc:          0.031392s

20 threads:
snprintf:        0.276279s
stringstream:    3.258638s
std::to_string:  0.309448s
myfunc:          0.060043s
```
