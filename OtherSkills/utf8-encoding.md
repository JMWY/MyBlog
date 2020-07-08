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

参考资料：
http://userguide.icu-project.org/ (icu： International Components for Unicode)
https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/（字符集基本知识）
https://www.ietf.org/rfc/rfc3629.txt
 http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html  (unicode 和 utf-8)

