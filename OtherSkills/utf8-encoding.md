# utf8  编码

## 概要
* unicode 字符编码表：http://www.ssec.wisc.edu/~tomw/java/unicode.html
* UTF-8 是 Unicode 的实现方式之一。
* UTF-8 是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度。
* UTF-8 的编码规则：<br/>
1）单字节的符号，第一位设为 0，后面 7 位为 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的。<br/>
2）n 字节的符号（n > 1），第一个字节的前 n 位都设为 1，第 n + 1 位设为 0，后面字节的前两位一律设为 10。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码。<br/>

|   Unicode符号范围                     |       UTF-8编码方式                             |
|---------------------------           |--------------------------------               |
| 0000 0000-0000 007F （0 - 127）       |       0xxxxxxx                                |
| 0000 0080-0000 07FF （128 - 2047）    |       110xxxxx   10xxxxxx                     |
| 0000 0800-0000 FFFF （2048 - 65535）  |     1110xxxx   10xxxxxx   10xxxxxx            |
| 0001 0000-0010 FFFF （65536 以上）     |     11110xxx   10xxxxxx   10xxxxxx   10xxxxxx |


例：严的 Unicode 是 4E25（100111000100101），根据上表，可以发现 4E25处在第三行的范围内（0000 0800 - 0000 FFFF），因此严的 UTF-8 编码需要三个字节，
即格式是 1110xxxx 10xxxxxx 10xxxxxx。然后，从严的最后一个二进制位开始，依次从后向前填入格式中的x，多出的位补0。这样就得到了，严的 UTF-8 编码是
11100100 10111000 10100101，转换成十六进制就是 E4B8A5。

## 单字切分
```c
/*================================================================
*   Copyright (C) 2019-07 All rights reserved.
*   文件名称：utf8_util_test.cpp
*   创 建 者：chenfeng
*   创建日期：2019年07月26日
*   描    述：
================================================================*/
#include <fstream>
#include <vector>
#include <iostream>
#include <cstring>
#include <utility>
#include <map>
#include <gtest/gtest.h>
#include "utf8_tokenizer.h"
TEST(Utf8_UTIL, getFirstToken) {
    std::map<std::string, std::string> test_input;
	test_input.insert(std::pair<const char *, std::string>("ผผูผู้", "ผ"));
	test_input.insert(std::pair<const char *, std::string>("ผู้ผผู", "ผู้"));
	test_input.insert(std::pair<const char *, std::string>("北京市朝阳区", "北"));
	test_input.insert(std::pair<const char *, std::string>("카니발", "카"));
	std::map<std::string, std::string>::iterator it;
	for (it = test_input.begin(); it != test_input.end(); ++it) {
		std::string prefix = getFirstToken(it->first);
		std::cout << "it->second:" << it->second << "\tprefix:" << prefix << std::endl; 
		ASSERT_TRUE(it->second == prefix);
	} 
}
TEST(Utf8_UTIL, getTokenList) {
    std::map<const char *, std::string> test_input;
    test_input.insert(std::pair<const char *, std::string>("北京市朝阳区", "北/京/市/朝/阳/区/"));
    test_input.insert(std::pair<const char *, std::string>("카니발", "카/니/발/"));
    test_input.insert(std::pair<const char *, std::string>("ผผู", "ผ/ผู/"));
    test_input.insert(std::pair<const char *, std::string>("ผผูผู้", "ผ/ผู/ผู้/"));
    test_input.insert(std::pair<const char *, std::string>("카니발กลุ่มความงาม", "카/니/발/ก/ลุ่/ม/ค/ว/า/ม/ง/า/ม/"));
    test_input.insert(std::pair<const char *, std::string>("", ""));
    test_input.insert(std::pair<const char *, std::string>("카.니*발", "카/./니/*/발/"));
    test_input.insert(std::pair<const char *, std::string>("카.니*abc발", "카/./니/*/a/b/c/발/"));
    test_input.insert(std::pair<const char *, std::string>("카6.7a*c니발", "카/6/./7/a/*/c/니/발/"));
    test_input.insert(std::pair<const char *, std::string>("카。 니발", "카/。/ /니/발/"));
    test_input.insert(std::pair<const char *, std::string>("카.67ac니발abc*", "카/./6/7/a/c/니/발/a/b/c/*/"));
    test_input.insert(std::pair<const char *, std::string>("严 ่ ่严", "严/ /่/ /่/严/"));
    test_input.insert(std::pair<const char *, std::string>("ผู严ผผผู้", "ผู/严/ผ/ผ/ผู้/"));
    test_input.insert(std::pair<const char *, std::string>("严 ่严", "严/ /่/严/"));
    test_input.insert(std::pair<const char *, std::string>("abc ่abc", "a/b/c/ /่/a/b/c/"));
    test_input.insert(std::pair<const char *, std::string>("ผูผผผู้ ่", "ผู/ผ/ผ/ผู้/ /่/"));
    test_input.insert(std::pair<const char *, std::string>("ผู ่ผ ่ผ ่ผู้ ่", "ผู/ /่/ผ/ /่/ผ/ /่/ผู้/ /่/"));
    vector<string> res;
	std::map<const char *, std::string>::iterator it;
    for (it = test_input.begin(); it != test_input.end(); ++it) {
        res.clear();
        EXPECT_TRUE(getTokenlist(it->first, strlen(it->first), res));
		std::string tokenizer_result("");
        for (std::vector<std::string>::iterator iter = res.begin(); iter != res.end(); ++iter) {
            tokenizer_result += *iter + "/";
        }
        ASSERT_TRUE(it->second == tokenizer_result);
    }
}
int main(int argc, char *argv[])
{
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

```c
/*================================================================
*   Copyright (C) 2019-07 All rights reserved.
*   文件名称：utf8_util_test.cpp
*   创 建 者：chenfeng
*   创建日期：2019年07月26日
*   描    述：
================================================================*/
#include <fstream>
#include <vector>
#include <iostream>
#include <cstring>
#include <utility>
#include <map>
#include <gtest/gtest.h>
#include "utf8_tokenizer.h"
TEST(Utf8_UTIL, getFirstToken) {
    std::map<std::string, std::string> test_input;
	test_input.insert(std::pair<const char *, std::string>("ผผูผู้", "ผ"));
	test_input.insert(std::pair<const char *, std::string>("ผู้ผผู", "ผู้"));
	test_input.insert(std::pair<const char *, std::string>("北京市朝阳区", "北"));
	test_input.insert(std::pair<const char *, std::string>("카니발", "카"));
	std::map<std::string, std::string>::iterator it;
	for (it = test_input.begin(); it != test_input.end(); ++it) {
		std::string prefix = getFirstToken(it->first);
		std::cout << "it->second:" << it->second << "\tprefix:" << prefix << std::endl; 
		ASSERT_TRUE(it->second == prefix);
	} 
}
TEST(Utf8_UTIL, getTokenList) {
    std::map<const char *, std::string> test_input;
    test_input.insert(std::pair<const char *, std::string>("北京市朝阳区", "北/京/市/朝/阳/区/"));
    test_input.insert(std::pair<const char *, std::string>("카니발", "카/니/발/"));
    test_input.insert(std::pair<const char *, std::string>("ผผู", "ผ/ผู/"));
    test_input.insert(std::pair<const char *, std::string>("ผผูผู้", "ผ/ผู/ผู้/"));
    test_input.insert(std::pair<const char *, std::string>("카니발กลุ่มความงาม", "카/니/발/ก/ลุ่/ม/ค/ว/า/ม/ง/า/ม/"));
    test_input.insert(std::pair<const char *, std::string>("", ""));
    test_input.insert(std::pair<const char *, std::string>("카.니*발", "카/./니/*/발/"));
    test_input.insert(std::pair<const char *, std::string>("카.니*abc발", "카/./니/*/a/b/c/발/"));
    test_input.insert(std::pair<const char *, std::string>("카6.7a*c니발", "카/6/./7/a/*/c/니/발/"));
    test_input.insert(std::pair<const char *, std::string>("카。 니발", "카/。/ /니/발/"));
    test_input.insert(std::pair<const char *, std::string>("카.67ac니발abc*", "카/./6/7/a/c/니/발/a/b/c/*/"));
    test_input.insert(std::pair<const char *, std::string>("严 ่ ่严", "严/ /่/ /่/严/"));
    test_input.insert(std::pair<const char *, std::string>("ผู严ผผผู้", "ผู/严/ผ/ผ/ผู้/"));
    test_input.insert(std::pair<const char *, std::string>("严 ่严", "严/ /่/严/"));
    test_input.insert(std::pair<const char *, std::string>("abc ่abc", "a/b/c/ /่/a/b/c/"));
    test_input.insert(std::pair<const char *, std::string>("ผูผผผู้ ่", "ผู/ผ/ผ/ผู้/ /่/"));
    test_input.insert(std::pair<const char *, std::string>("ผู ่ผ ่ผ ่ผู้ ่", "ผู/ /่/ผ/ /่/ผ/ /่/ผู้/ /่/"));
    vector<string> res;
	std::map<const char *, std::string>::iterator it;
    for (it = test_input.begin(); it != test_input.end(); ++it) {
        res.clear();
        EXPECT_TRUE(getTokenlist(it->first, strlen(it->first), res));
		std::string tokenizer_result("");
        for (std::vector<std::string>::iterator iter = res.begin(); iter != res.end(); ++iter) {
            tokenizer_result += *iter + "/";
        }
        ASSERT_TRUE(it->second == tokenizer_result);
    }
}
int main(int argc, char *argv[])
{
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

## 语言识别
```c
/*================================================================
*   Copyright (C)  2019-07 All rights reserved.
*   文件名称：unicode.h
*   创 建 者：chenfeng                                                                                                                            
*   创建日期：2019年07月17日
*   描    述：
================================================================*/

#pragma once

#include <stdint.h>
#include <string>
#include <boost/operators.hpp>

/**
 * Is this code unit (byte) a UTF-8 lead byte?
 */
#define U8_IS_LEAD(c) ((uint8_t)((c)-0xc0)<0x3e)

/**
 * Is this code unit (byte) a UTF-8 trail byte?
 */
#define U8_IS_TRAIL(c) (((c)&0xc0)==0x80)

/**
 * Counts the trail bytes for a UTF-8 lead byte.
 */
#define U8_COUNT_TRAIL_BYTES(leadByte) \
  ((uint8_t)(leadByte)<0xf0 ? \
    ((uint8_t)(leadByte)>=0xc0)+((uint8_t)(leadByte)>=0xe0) : \
     (uint8_t)(leadByte)<0xfe ? \
      3+((uint8_t)(leadByte)>=0xf8)+((uint8_t)(leadByte)>=0xfc) : 0)

/**
 * Counts the trail bytes for a UTF-8 lead byte of a valid UTF-8 sequence.
 */
#define U8_COUNT_TRAIL_BYTES_UNSAFE(leadByte) \
  (((leadByte)>=0xc0)+((leadByte)>=0xe0)+((leadByte)>=0xf0))


template <class Iter>
Iter utf8ForwardCodePoint(Iter p) {
    return p + 1 + U8_COUNT_TRAIL_BYTES_UNSAFE((uint8_t)*p);
}

template <class Iter>
Iter utf8ForwardCodePointSafe(Iter p, Iter end) {
    uint8_t c = (uint8_t) *p++;
    if (U8_IS_LEAD(c)) {
        ssize_t count = U8_COUNT_TRAIL_BYTES(c);
        if (p + count - end > 0) {
            return NULL;
        }
        while (count > 0 && U8_IS_TRAIL(*p)) {
            ++p;
            --count;
        }
        if (count > 0) {
            return NULL;
        }
    }
    return p;
}

inline bool utf8StringIsValid(const std::string& str) {
    auto p = str.data();
    auto e = str.data() + str.size();
    while (p != e) {
        p = utf8ForwardCodePointSafe(p, e);
        if (!p) {
            return false;
        }
    }
    return true;
}

char32_t utf8ToCodePointUnsafe(const unsigned char* p) {
    char32_t cp = *p++;
    if (cp >= 0x80) {
        U8_IS_LEAD(cp);
        if (cp < 0xe0) {
            cp = ((cp & 0x1f) << 6) | (*p & 0x3f);
        } else if (cp < 0xf0) {
            /* no need for (c&0xf) because the upper bits are truncated
             * after <<12 in the cast to (char16_t) */
            cp = static_cast<char16_t>(((cp           ) << 12) |
                                       ((*p     & 0x3f) << 6) |
                                       ((*(p+1) & 0x3f)));
        } else {
            cp = (((cp     & 7   ) << 18) |
                  ((*p     & 0x3f) << 12) |
                  ((*(p+1) & 0x3f) << 6) |
                  ((*(p+2) & 0x3f)));
        }
    }
    return cp;
}


/*
 * Assumes well-formed UTF-8.
 */
template <class Iter>
class Utf8Iterator
    : public std::iterator<std::bidirectional_iterator_tag, char32_t>,
      private boost::totally_ordered<Utf8Iterator<Iter>> {
public:
    Utf8Iterator() : p_() {}
    Utf8Iterator(const Iter& p) : p_(p) {}
    Utf8Iterator(const Utf8Iterator& iter) : p_(iter.p_) {}

    Utf8Iterator& operator++() {
        p_ = utf8ForwardCodePoint(p_);
        return *this;
    }
    Utf8Iterator operator++(int) {
        Utf8Iterator it(*this);
        operator++();
        return it;
    }

    Utf8Iterator& operator--() {
        p_ = utf8BackCodePoint(p_);
        return *this;
    }
    Utf8Iterator operator--(int) {
        Utf8Iterator it(*this);
        operator--();
        return it;
    }

    Utf8Iterator operator+(ssize_t n) const {
        Utf8Iterator it(*this);
        for (; n > 0; --n)
            ++it;
        for (; n < 0; ++n)
            --it;
        return it;
    }
    Utf8Iterator operator-(ssize_t n) const { return *this + -n; }

    ssize_t operator-(const Utf8Iterator& rhs) const {
        ssize_t n = 0;
        Utf8Iterator it(rhs);
        for (; it != *this; ++it)
            ++n;
        return n;
    }

    char32_t& operator*() {
        cp_ = utf8ToCodePointUnsafe((const unsigned char*)p_);
        return cp_;
    }
    const char32_t& operator*() const {
        cp_ = utf8ToCodePointUnsafe((const unsigned char*)p_);
        return cp_;
    }

    Iter operator&() const { return p_; }

    size_t length() const { return utf8CodePointLength(p_); }

private:
    Iter p_;
    mutable char32_t cp_;
};

template <class Iter>
inline bool operator==(const Utf8Iterator<Iter>& lhs,
                       const Utf8Iterator<Iter>& rhs) {
    return &lhs == &rhs;
}

template <class Iter>
inline bool operator<(const Utf8Iterator<Iter>& lhs,
                      const Utf8Iterator<Iter>& rhs) {
    return &lhs < &rhs;
}

template <class Iter>
Utf8Iterator<Iter> makeUtf8Iterator(const Iter& p) {
    return Utf8Iterator<Iter>(p);
}

typedef Utf8Iterator<const char*> Utf8CharIterator;

inline bool checkUtf8String(const std::string& str) {
    if (!utf8StringIsValid(str)) {
        return false;
    }
    Utf8CharIterator p = str.data();
    Utf8CharIterator e = str.data() + str.size();
    while (p != e) {
        char32_t cp = *p;
        //printf("%x ", uint32_t(cp));
        // TODO use icu
        // a rough map, see https://en.wikipedia.org/wiki/UTF-8
        if (!((cp < 0x80 && isprint(cp)) ||
              (cp >= 0xa0 && cp <= 0xff) ||       // https://en.wikipedia.org/wiki/Latin-1_Supplement_(Unicode_block)
              (cp >= 0x100 && cp <= 0x2af) ||     // http://www.ssec.wisc.edu/~tomw/java/unicode.html
              (cp >= 0x370 && cp <= 0x3ff) ||     // https://en.wikipedia.org/wiki/Greek_alphabet#Greek_in_Unicode
              (cp >= 0x600 && cp <= 0x6ff) ||     // https://en.wikipedia.org/wiki/Arabic_(Unicode_block)
              (cp >= 0x1f00 && cp <= 0x1fff) ||   // https://en.wikipedia.org/wiki/Greek_alphabet#Greek_in_Unicode
              (cp >= 0x3000 && cp <= 0x30ff) ||   // http://www.fileformat.info/info/unicode/block/cjk_symbols_and_punctuation/index.htm, https://en.wikipedia.org/wiki/Kana
              (cp >= 0x3400 && cp <= 0x4dbf) ||   // https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
              (cp >= 0x4e00 && cp <= 0x9fd5) ||   // https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
              (cp >= 0xac00 && cp <= 0xd7af) ||   // http://www.ch2ko.com/hanguoyu/hanwen-unicode/
              (cp >= 0xf900 && cp <= 0xfaff) ||   // https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
              (cp >= 0xff01 && cp <= 0xffee) ||   // http://www.fileformat.info/info/unicode/block/halfwidth_and_fullwidth_forms/list.htm 
              (cp >= 0x20000 && cp <= 0x2a6df) || // https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
              (cp >= 0x2a700 && cp <= 0x2b73f) || // https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
              (cp >= 0x2b740 && cp <= 0x2b81f) || // https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
              (cp >= 0x2b820 && cp <= 0x2ceaf) || // https://en.wikipedia.org/wiki/CJK_Unified_Ideographs
              (cp >= 0x2190 && cp <= 0x2194) ||
              (cp >= 0x2460 && cp <= 0x249b) ||
              (cp >= 0x24ea && cp <= 0x24ff) ||
              (cp >= 0x2776 && cp <= 0x2793) ||
              (cp >= 0x3192 && cp <= 0x3195) ||
              (cp >= 0x3220 && cp <= 0x3229) ||
              (cp >= 0x3251 && cp <= 0x325f) ||
              (cp >= 0x3280 && cp <= 0x3289) ||
              (cp >= 0x32b1 && cp <= 0x32bf) ||
              (cp >= 0x0400 && cp <= 0x04ff) || // Cyrillic and Cyrillic Supplement (俄文)
              (cp >= 0x0e00 && cp <= 0x0e7f) || // Thai（泰文)
              (cp == 0x2022 ||
               cp == 0x2013 || cp == 0x2014 ||
               cp == 0x2018 || cp == 0x2019 ||
               cp == 0x201c || cp == 0x201d ||
               cp == 0x2026 ||
               cp == 0x2103 || cp == 0x2109 ||
               cp == 0x25aa ||
               cp == 0x25cf ||
               cp == 0x2605 || cp == 0x2606)))
            return false;
        ++p;
    }
    return true;
}

```

```c
/*================================================================
*   Copyright (C) 2019-07 All rights reserved.
*   文件名称：unicode_test.cpp
*   创 建 者：chenfeng
*   创建日期：2019年07月17日
*   描    述：
================================================================*/
#include "unicode.h"
#include <gtest/gtest.h>

TEST(utf8, utf8StringIsValid) {
    EXPECT_TRUE(utf8StringIsValid(" "));
    EXPECT_TRUE(utf8StringIsValid("0123"));
    EXPECT_TRUE(utf8StringIsValid("abcd"));
    EXPECT_TRUE(utf8StringIsValid("中国"));
    EXPECT_TRUE(utf8StringIsValid("\n"));
    EXPECT_TRUE(utf8StringIsValid("\xFF"));
    EXPECT_TRUE(utf8StringIsValid("���"));
    EXPECT_TRUE(utf8StringIsValid("พระบรมมหาราชวัง"));
    EXPECT_TRUE(utf8StringIsValid("한라산국립공원"));
    EXPECT_TRUE(utf8StringIsValid("北海道の桜"));
    EXPECT_TRUE(utf8StringIsValid("кремль"));
}

TEST(utf8, checkUtf8String) {
    EXPECT_TRUE(checkUtf8String("พระบรมมหาราชวัง"));
    EXPECT_TRUE(checkUtf8String("한라산국립공원"));
    EXPECT_TRUE(checkUtf8String("北海道の桜"));
    EXPECT_TRUE(checkUtf8String("кремль"));
    EXPECT_TRUE(checkUtf8String("zēzēcupcake"));
    EXPECT_TRUE(checkUtf8String(" "));
    EXPECT_TRUE(checkUtf8String("0123"));
    EXPECT_TRUE(checkUtf8String("abcd"));
    EXPECT_TRUE(checkUtf8String("中国"));

    EXPECT_TRUE(checkUtf8String("(|)【2847219839+q扣】★1i7jc9"));
    EXPECT_TRUE(checkUtf8String("(|)【2860260008+扣扣】★n98ecb"));
    EXPECT_TRUE(checkUtf8String("(|)(2890007886+扣扣)★97rg92"));
    EXPECT_TRUE(checkUtf8String("(|qq±2８６０２６０００８】"));
    EXPECT_TRUE(checkUtf8String("(|)【206444680+扣扣】★qfxj0i"));
    EXPECT_TRUE(checkUtf8String("(|)【2056999200+扣q】★ifcc8z"));
    EXPECT_TRUE(checkUtf8String("0℃冷饮批发"));
    EXPECT_TRUE(checkUtf8String("100℃年代火锅"));
    EXPECT_TRUE(checkUtf8String("100℃比萨"));
    EXPECT_TRUE(checkUtf8String("100℃沸瓦罐营养快餐双人餐"));
    EXPECT_TRUE(checkUtf8String("176℃炸鸡排"));
    EXPECT_TRUE(checkUtf8String("260℃快时尚餐厅"));
    EXPECT_TRUE(checkUtf8String("38.6℃"));
    EXPECT_TRUE(checkUtf8String("38.6℃烘焙坊"));
    EXPECT_TRUE(checkUtf8String("39℃交友会所"));
    EXPECT_TRUE(checkUtf8String("42℃汗蒸美痧养生会所"));
    EXPECT_TRUE(checkUtf8String("52℃香纸包鱼"));
    EXPECT_TRUE(checkUtf8String("57℃湘"));
    EXPECT_TRUE(checkUtf8String("67℃芦花鸡主题餐厅"));
    EXPECT_TRUE(checkUtf8String("7＋7"));
    EXPECT_TRUE(checkUtf8String("7＋日式料理"));
    EXPECT_TRUE(checkUtf8String("85℃ 1912"));
    EXPECT_TRUE(checkUtf8String("85℃"));
    EXPECT_TRUE(checkUtf8String("85℃生日蛋糕"));
    EXPECT_TRUE(checkUtf8String("85℃蛋糕"));
    EXPECT_TRUE(checkUtf8String("85℃蛋糕房"));
    EXPECT_TRUE(checkUtf8String("85℃面包"));
    EXPECT_TRUE(checkUtf8String("88℃烘焙"));
    EXPECT_TRUE(checkUtf8String("92℃噜串铺子"));
    EXPECT_TRUE(checkUtf8String("95℃西餐"));
    EXPECT_TRUE(checkUtf8String("????"));
    EXPECT_TRUE(checkUtf8String("??????"));
    EXPECT_TRUE(checkUtf8String("G＋"));
    EXPECT_TRUE(checkUtf8String("S•Y"));
    EXPECT_TRUE(checkUtf8String("T●BEST 德国啤酒城"));
    EXPECT_TRUE(checkUtf8String("hi 爆＋重庆江湖菜"));
    EXPECT_TRUE(checkUtf8String("theFrypan韩国炸鸡＆啤酒"));
    EXPECT_TRUE(checkUtf8String("wuli大叔雪花冰•甜品饮料"));
    EXPECT_TRUE(checkUtf8String("|临汾|±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("|余姚|±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("|尚志|±扣扣４８０.４４４.１５８"));
    EXPECT_TRUE(checkUtf8String("|汝州|±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("|荔波|±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("|酒泉|±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("|锦州|±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("|高邮|QQ±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("|龙胜|±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("|龙胜|±ＱＱ４８０上４４４门１５８"));
    EXPECT_TRUE(checkUtf8String("【】"));
    EXPECT_TRUE(checkUtf8String("【健康东路】 42℃纳米汗蒸养生馆"));
    EXPECT_TRUE(checkUtf8String("东莞旗峰山铂尔曼酒店－樱香阁日本料理餐厅"));
    EXPECT_TRUE(checkUtf8String("中影国际影城"));
    EXPECT_TRUE(checkUtf8String("丽源阳光养生馆、"));
    EXPECT_TRUE(checkUtf8String("九味•众品大虾火锅"));
    EXPECT_TRUE(checkUtf8String("九里•烤肉"));
    EXPECT_TRUE(checkUtf8String("仲记•豫满楼"));
    EXPECT_TRUE(checkUtf8String("佰音ＫＴＶ"));
    EXPECT_TRUE(checkUtf8String("兴合记•时尚海鲜"));
    EXPECT_TRUE(checkUtf8String("再清椿美容"));
    EXPECT_TRUE(checkUtf8String("冬季烤肉推荐－锦州的热门烤肉地"));
    EXPECT_TRUE(checkUtf8String("刮痧＋拔罐"));
    EXPECT_TRUE(checkUtf8String("吉布鲁牛排•海鲜自助(泸州店)"));
    EXPECT_TRUE(checkUtf8String("吉祥馄饨▪面"));
    EXPECT_TRUE(checkUtf8String("名指•汇摩丽会所"));
    EXPECT_TRUE(checkUtf8String("品麦道●鸡翅包饭"));
    EXPECT_TRUE(checkUtf8String("喜满客"));
    EXPECT_TRUE(checkUtf8String("国王＊摩点"));
    EXPECT_TRUE(checkUtf8String("土豆粉 姐弟俩"));
    EXPECT_TRUE(checkUtf8String("天贵国际酒店－自助餐"));
    EXPECT_TRUE(checkUtf8String("婚纱摄影"));
    EXPECT_TRUE(checkUtf8String("宁波开发票QQ2538945497→致电13662221480"));
    EXPECT_TRUE(checkUtf8String("尕记锅魁•酸辣粉"));
    EXPECT_TRUE(checkUtf8String("年底去哪儿八卦？尾牙聚会餐厅~拿好，不谢！"));
    EXPECT_TRUE(checkUtf8String("张亮麻辣烫"));
    EXPECT_TRUE(checkUtf8String("春起三月 悦＋"));
    EXPECT_TRUE(checkUtf8String("晶采轩自助烤肉▪火锅"));
    EXPECT_TRUE(checkUtf8String("木37℃咖啡甜品简餐"));
    EXPECT_TRUE(checkUtf8String("柚子＆Candy"));
    EXPECT_TRUE(checkUtf8String("正新鸡排"));
    EXPECT_TRUE(checkUtf8String("水无沙湖南人●食力派餐厅"));
    EXPECT_TRUE(checkUtf8String("沐府88℃"));
    EXPECT_TRUE(checkUtf8String("泰华车港 ②号店"));
    EXPECT_TRUE(checkUtf8String("渔珺传奇▪云南石锅鱼"));
    EXPECT_TRUE(checkUtf8String("湘遇22＃菜馆"));
    EXPECT_TRUE(checkUtf8String("烤肉村 고기굽는마을"));
    EXPECT_TRUE(checkUtf8String("爱都●欧麦莱"));
    EXPECT_TRUE(checkUtf8String("福口居（淄城路店）"));
    EXPECT_TRUE(checkUtf8String("禧御贡茶"));
    EXPECT_TRUE(checkUtf8String("老红旗▪鲁津饭庄  天津"));
    EXPECT_TRUE(checkUtf8String("艾一诺蛋糕 新华路店－"));
    EXPECT_TRUE(checkUtf8String("莎莎Salsa•川西坝子自助火锅"));
    EXPECT_TRUE(checkUtf8String("衣柜"));
    EXPECT_TRUE(checkUtf8String("西旺冰室•港式茶餐厅"));
    EXPECT_TRUE(checkUtf8String("许小树の小铺"));
    EXPECT_TRUE(checkUtf8String("豪门纯K"));
    EXPECT_TRUE(checkUtf8String("辣皇尚●麻辣诱惑香锅"));
    EXPECT_TRUE(checkUtf8String("邻水吉布鲁牛排•海鲜自助(宏帆广场店)"));
    EXPECT_TRUE(checkUtf8String("酒店"));
    EXPECT_TRUE(checkUtf8String("重庆1＋1"));
    EXPECT_TRUE(checkUtf8String("金粒餐"));
    EXPECT_TRUE(checkUtf8String("镇江自助餐。"));
    EXPECT_TRUE(checkUtf8String("陈家牛出没时尚～～～"));
    EXPECT_TRUE(checkUtf8String("饭思特"));
    EXPECT_TRUE(checkUtf8String("麻辣烫＋肉夹馍"));
    EXPECT_TRUE(checkUtf8String("鼎香面馆•中式快餐"));

    EXPECT_FALSE(checkUtf8String("????ʿ"));
    EXPECT_FALSE(checkUtf8String("\n"));
    EXPECT_FALSE(checkUtf8String("\xFF"));
    EXPECT_FALSE(checkUtf8String("hao wang"));
    EXPECT_FALSE(checkUtf8String("ä¸Šè†³é£Ÿåºœ"));
    EXPECT_FALSE(checkUtf8String("乳酪生日蛋糕🎂"));
    EXPECT_FALSE(checkUtf8String("刨冰🍧"));
    EXPECT_FALSE(checkUtf8String("大树D a su蛋糕"));
    EXPECT_FALSE(checkUtf8String("天津摩天轮🎡"));
    EXPECT_FALSE(checkUtf8String("爱在春天&健康自�"));
    EXPECT_FALSE(checkUtf8String("猫铺❤️虾饼"));
    EXPECT_FALSE(checkUtf8String("生日蛋糕🎂"));
    EXPECT_FALSE(checkUtf8String("空中花园鱼🐠"));
    EXPECT_FALSE(checkUtf8String("自助烧烤➕火锅"));
    EXPECT_FALSE(checkUtf8String("蛋糕房🍰"));
    EXPECT_FALSE(checkUtf8String("���"));
    EXPECT_FALSE(checkUtf8String("����� "));
    EXPECT_FALSE(checkUtf8String("🍌"));
    EXPECT_FALSE(checkUtf8String("😷😥😝"));
}

int main(int argc, char** argv) {
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}

```



## 参考资料：
* http://userguide.icu-project.org/ (icu： International Components for Unicode)
* https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/（字符集基本知识）
* https://www.ietf.org/rfc/rfc3629.txt
* http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html  (unicode 和 utf-8)

