# 数字和字符串之间的转换

# atoi
相关函数：atoi，atol，strtod，strtol, stroul
头文件： #include <stdlib.h>





# itoa
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
![avatar](https://github.com/JMWY/MyBlog/blob/master/OtherSkills/images/itoa_benchmark.png "benchmark")


