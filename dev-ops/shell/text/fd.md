## 文件描述符


### 定义

####1 什么是文件


**linux操作系统下一切可读写的对象都是文件：**

    * 标准输入输出
    * 磁盘文件
    * socket套接字
    * 管道

具体来说，可以用read/write方法来读写数据

####2 什么是文件描述符

文件描述符是系统打开文件表（open file table）的索引，索引条目是文件句柄，记录了打开文件的相关信息

* 当前文件偏移量
* 打开文件所使用的状态标识
* 文件访问模式(只读/只写/读写模式)
* 信号量驱动相关
* 对文件inode的引用
* 文件类型(socket, FIFO等)和访问权限
* 文件的各种属性等

####3 进程与文件描述符

* 一个进程通常会创建多个文件描述符，用于操作各种文件资源，例如socket连接

* 任何一个进程都会创建3个标准的文件描述符
    * 标准输入 -> 0
    * 标准输出 -> 1
    * 标准错误 -> 2

* 文件和fd是一对多的关系
    * 一个文件可以被多个文件描述符指向
    * 不同的文件描述符可能来自一个或多个进程
    * 不同进程的文件描述符可能相同


### 文件描述符相关查询命令


##### 1 fd 打开数限制查询

```
//查看系统最大文件描述符限制
cat /proc/sys/fs/file-max

//查看进程最大文件描述符(软/硬)限制
ulimit -Sn
ulimit -Hn

//查看其它用户资源限制
ulimit -a

//修改,针对用户进行资源限制
vim /etc/security/limits.conf

```

##### 2 查看当前系统已创建的fd

```

//查看进程的fd, 包含fd类型，权限等信息
lsof -p [pid]

```

### 文件描述符相关代码实现

在一个进程内打开不同类型的文件并打印对应的fd

```

#include <unistd.h>
#include <fcntl.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main()
{
    /*标准输入、输出*/
    printf("stdin = %d\n", STDIN_FILENO);
    printf("stdout = %d\n", STDOUT_FILENO);
    printf("stderr = %d\n\n", STDERR_FILENO);

    /*打开两个磁盘文件*/
    int fd1 = open("./a1.txt", O_RDONLY);
    int fd2 = open("./a2.txt", O_RDONLY);
    printf("fd1 = %d\n", fd1);
    printf("fd2 = %d\n\n", fd2);

    /*创建两个套接字*/
    int sfd1 = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    int sfd2 = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    printf("sfd1 = %d\n", sfd1);
    printf("sfd2 = %d\n\n", sfd2);

    /*关闭之前打开的两个磁盘文件*/
    printf("close fd1 fd2\n");
    close(fd1);
    close(fd2);

    /*创建一个管道*/
    int pipefd[2];
    pipe(pipefd);
    printf("pipefd1 = %d\n", pipefd[0]);
    printf("pipefd2 = %d\n\n", pipefd[1]);

    /*再次打开刚才那两个磁盘文件*/
    int fd3 = open("./a1.txt", O_RDONLY);
    int fd4 = open("./a2.txt", O_RDONLY);
    printf("fd3 = %d\n", fd3);
    printf("fd4 = %d\n\n", fd4);

    getchar();
    return 0;
}


//启动进程后运行结果：

stdin = 0
stdout = 1
stderr = 2

fd1 = 3
fd2 = 4

sfd1 = 5
sfd2 = 6

close fd1 fd2
pipefd1 = 3
pipefd2 = 4

fd3 = 7
fd4 = 8

```


### 使用fd以不同操作模式打开文件

```

#!/bin/bash

#创建fd=3以只读模式打开文件test
echo "test" > test.txt
exec 3< test.txt
cat <&3

#创建fd=4以(截断写入模式)打开文件test2
exec 4> test2.txt
echo "test2" >&4
cat test2.txt

#创建fd=5以追加写入模式打开文件test3
exec 5>> test3.txt
echo "test3" >&5
echo "test3 again" >&5
cat test3.txt

```

### java中的资源

高级语言中，关于文件的打开操作，本质上都是通过系统调用，创建fd，并通过fd操作打开文件资源的

例如java的FileInputStream

```

public class FileInputStream extends InputStream{

    //文件描述符引用
    private final FileDescriptor fd;

    //构造FileInputStream时会构造fd
    public FileInputStream(File file) throws FileNotFoundException {
        String name = (file != null ? file.getPath() : null);
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            security.checkRead(name);
        }
        if (name == null) {
            throw new NullPointerException();
        }
        if (file.isInvalid()) {
            throw new FileNotFoundException("Invalid file path");
        }
        //打开文件时创建fd并计算使用数量
        fd = new FileDescriptor();
        fd.incrementAndGetUseCount();
        this.path = name;
        open(name);
    }   
}

```

在java中，gc不负责系统资源的回收，需要主动关闭资源，否则会造成资源浪费或导致资源不足

例如 too manay open files


### 关于标准输出和错误文件描述符

```
//后台方式启动服务，将标准输出以及错误输出重定向到指定日志文件
nohup zkServer start 2>&1 >/tmp/zk.log &

如果想丢弃日志，可以使用nohup zkServer start 2>&1 >/dev/null &

```

* 2>&1 将错误输出重定向到标准输出
* > /dev/null 将标准输出重定向到/dev/null，这李相当于丢弃
* & 以dameon方式启动
