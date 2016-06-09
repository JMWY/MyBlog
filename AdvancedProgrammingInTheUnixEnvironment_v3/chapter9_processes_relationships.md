# 进程关系
## 1. 终端登录
### BSD 终端登录
![login_invoked](https://github.com/JMWY/MyBlog/blob/master/AdvancedProgrammingInTheUnixEnvironment_v3/images/chapter9/login_invoked.PNG)

* `init` 进程 read `/etc/ttys`，对每个允许登录的终端设备(tty), 调用一次 `fork`，自进程执行 `(exec) getty`。
* `getty` 调用 `open`，读写方式打开终端设备,设置文件描述符 0、1、2，然后输出 **login: **, 等待用户输入用户名。用户键入用户名后，调用：       
　　　　`execle("/bin/login", "login", "-p", username, (char*)0, envp);`
* 进程ID 不会因 `exec` 改变，图中底部三个进程：ID相同，父进程ID 为 1。最初的 init 父进程ID 0。
* `login` 调用 `getpwnam` 取得该用户的**口令文件登录项**，然后调 `getpass`  输出 **Password: **，调用 `crypt` 加密口令，然后与所得的**口令文件登录项**的 `pw_passwd` 字段比较。若几次都无效，则调用 `exit(1)`，父进程（`init`）再次调用 `fork` 执行 `getty`。
* `login` 在登陆后，执行如下 Change：       

        dirctory: Change to our home directory (chdir)    
        ownership: chown user:user our terminal device   
        access permissions: can read && write can our terminal device     
        group IDs:  调用 setgid and initgroups    
        environment: our home directory (HOME), shell (SHELL), user name (USER and LOGNAME), and a default path (PATH)      
        user ID: 调用 setuid 设自身(`login`)为 user ID，Then invoke our login shell, as in
            execl("/bin/sh", "-sh", (char *)0);

------------------------------------------

![终端登录后的进程安排](https://github.com/JMWY/MyBlog/blob/master/AdvancedProgrammingInTheUnixEnvironment_v3/images/chapter9/process-arrage_after_login.PNG)

- `login shell` is running now， 父进程为 `init`(ID： 1) 。所以登录 `shell` 终端时，`init` 接收到信号 `SIGCHLD`，将 `login shell` 的文件描述符 0、1 和 2 设为（`set to`）终端设备。
- `login shell` reads its `start-up files` now.                  
(`.profile` for `Bourne shell` and `Korn SHell`)                        
(`.bash_profile`, `.bash_login`, or `.profile` for `GNU Bourne-again shell`)                    
(`.cshrc` and `.login` for the `C shell`)                       

#### (Complement of JMWY) shell 类型：
###### B shell: sh 
* burne shell (sh) 
* burne again shell (bash) 

###### C shell: csh 
* c shell (csh) 
* tc shell (tcsh) 
* korn shell (ksh)              

###### 查看 shell 方法
1. echo $0
2. ps | grep $$ 
3. echo $SHELL


## 2. 网络登录

## 3. 进程组

## 4. 会话

## 5. 控制终端

## 6. tcgetpgrp、tcsetpgrp 和 tcgetsid 函数

## 7. 作业控制

## 8. shell 执行程序

## 9. 孤儿进程组

## 10. FreeBSD 实现

## 11. 习题



