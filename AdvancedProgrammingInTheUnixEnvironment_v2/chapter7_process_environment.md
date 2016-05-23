# 进程环境

## 1. C 程序启动（main 函数）
main 函数是 C 程序进入点。
### 函数原型
(ISO C 和 POSIX.1 支持的)原型有两个:  

    int main(void) { /* ... */ };
    int main(int argc, char *argv[]) { /* ... */ };
    
>历史上，多数 UNIX 系统支持:
    
    int main(int argc, char *argv[], char *envp[]) { /* ... */ };    

>第三个参数是环境表地址。现在通常用 getenv 和 putenv 函数来访问特定环境变量。

### 调用 main 函数
- 内核（通过一个 exec 函数）执行 C 程序时，首先调用一个特殊的启动例程（start-up routine），再调用 main 函数。  
- **C 编译器**调用**链接编辑器**，然后**链接编辑器**设置**执行程序文件**，将**启动例程**指定为程序的**起始地址**。
- **启动例程**（常用汇编语言编写）如果用 C 代码形式表示，大概为：  

        exit(main(argc, argv))

### 命令行参数
执行程序时，**调用 exec 的进程**将命令行参数传递给新程序。*Note:* echo 程序通常不回送第 0 个参数。
``` c
#include <stdio.h>
int main (int argc, char *argv[])
{   
    int i;
    for (int i = 0; i < argc; ++i) {
        printf("argv[%d]: %s\n", i, argv[i]);
    }
    /*note: use exit replace return maybe generate warnings in some C complier or UNIX lint(1) programs*/
    exit(0);
}
```

----------------------------------------------------------------
## 2. 进程终止
进程终止（termination）方式有 8 种：正常终止 5 种方式，异常终止 3 种:
> (1) 从 main 返回      
> (2) exit      
> (3) _exit     
> (4) 最后一个线程从启动例程返回        
> (5) 最后一个线程调用 pthread_exit     
> **以下为异常终止方式**        
> (6) abort     
> (7) 收到（终止）信号      
> (8) 最后一个线程响应取消请求     

### exit 函数 
在头文件 [stdlib.h](http://pubs.opengroup.org/onlinepubs/7908799/xsh/stdlib.h.html) 中声明的原型为：

        void exit(int status); // (ISO C)
        void _Exit(int status); // Note: not found in file stdlib.h

在头文件 [unistd.h](http://pubs.opengroup.org/onlinepubs/7908799/xsh/unistd.h.html) 中声明的原型为：

        void _exit(int status); //(POSIX.1)

- 参数 satus：终止状态（或退出状态，exit status）
- **exit 函数**先调用**终止处理程序**，并关闭所有 I/O 流（为所有打开流调用 fclose 函数），然后调用 _exit(或_Exit) 进入内核

### atexit 函数 - 登记**终止处理程序**
在头文件 [stdlib.h](http://pubs.opengroup.org/onlinepubs/7908799/xsh/stdlib.h.html) 中声明的原型为：

        int atexit(void (*func)(void));

- 参数 func : 所登记函数的地址
- 调用顺序与登记顺序相反
- ISO C 规定，一个进程至少支持登记 32 个函数，即**终止处理程序(exit handler)**。为了确定平台支持的最大**终止处理程序数**，可查看 [sysconf 函数](http://pubs.opengroup.org/onlinepubs/009695399/functions/sysconf.html)中变量 ATEXIT_MAX。

``` c
/*@ test_atexit.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void my_exit1(void);
static void my_exit2(void);

int main (void) 
{
	if (atexit(my_exit1) != 0) {
		perror("can't register my_exit1");
	}
	if (atexit(my_exit2) != 0) {
		perror("can't register my_exit1");
	}

	printf("main is done\n");
	return 0;
}

static void my_exit1(void) 
{
	printf("1st exit handler\n");	
}

static void my_exit2(void) 
{
	printf("2nd exit handler\n");	
}
```

![运行结果](https://raw.githubusercontent.com/JMWY/MyBlog/master/AdvancedProgrammingInTheUnixEnvironment_v2/images/chapter7/test_atexit.png)




------------------------------------------------------------------
## 3. 环境变量

每个程序都有一张**环境表（environment list）**，地址为全局变量 environ:
	
	extern char **environ; 
	
可用于查看整个环境。其大致形式如下：

·图·
<br />

头文件 stdlib.h 中，定义了环境变量的访问函数：

	char *getenv(const char *name);  // return: SUCESS, 对应 value 的指针; FAIL: NULL
	int putenv(char *str);
	int setenv(const char *name, const char *value, int rewrite);
	int unsetenv(const char *name); // return: SUCESS, 0; FAIL: 非0 

**解释**： 		
`putenv`: 字符串形式为 name=value，放入环境表。若 name 已存在，则先删除原定义。		
`setenv`: name 存在时，(a) rewrite 非0，删除原定义 (b) rewirte 为0，不删除原定义，返回0 。		
`unsetenv`: 删除 name 定义。不存在也返回0 。		
<br />
**环境表修改策略:**

------------------------------------------------------------------
## 4. 共享库

--------------------------------------------------------------------
## 5. C 程序内存空间

--------------------------------------------------------------------
## 6. 跳转函数

--------------------------------------------------------------------
## 7. 进程资源限制

-------------------------------------------------------------------
## 8. 习题


``` c
#include <stdio.h>
int main (int argc, char *argv[]) 
{
}
```


