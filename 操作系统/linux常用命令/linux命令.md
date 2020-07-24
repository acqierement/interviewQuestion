# top、free、df、du、tail、ps、grep命令

## top

Linux系统可以通过top命令查看系统的CPU、内存、运行时间、交换分区、执行的线程等信息。通过top命令可以有效的发现系统的缺陷出在哪里。是内存不够、CPU处理能力不够、IO读写过高….

参考[Linux中top命令参数详解](https://blog.csdn.net/yjclsx/article/details/81508455)

```shell
top - 15:56:14 up 59 days,  1:15,  1 user,  load average: 0.00, 0.00, 0.00
Tasks: 103 total,   1 running, 102 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.6%us,  0.9%sy,  0.0%ni, 98.2%id,  0.1%wa,  0.0%hi,  0.0%si,  0.3%st
Mem:   3859356k total,  3071820k used,   787536k free,   162552k buffers
Swap:        0k total,        0k used,        0k free,  1270632k cached
```

先看第一行：top - 15:56:14 up 59 days,  1:15,  1 user,  load average: 0.00, 0.00, 0.00

| 内容                           | 含义                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| 15:56:14                       | 当前时间                                                     |
| up 59 days,  1:15              | 系统运行时间 格式为时：分                                    |
| 1 user                         | 当前一个用户                                                 |
| load average: 0.00, 0.00, 0.00 | 系统负载，即任务队列的平均长度。 三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。 |

第二行：Tasks: 103 total,   1 running, 102 sleeping,   0 stopped,   0 zombie

这个很好理解，进程总数，运行、睡眠、停止、僵尸进程的数量

## free

free 命令显示系统内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存。

## df

df（disk filesystem 的简称）用于显示 Linux 系统的磁盘利用率。（LCTT 译注：`df` 可能应该是 disk free 的简称。）

linux中df命令的功能是用来检查linux服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

## du

`du`（disk usage 的简称）是用于查找文件和目录的磁盘使用情况的命令。

```
du /home 该命令的输出将显示 /home 中的所有文件和目录以及显示块大小。
du -h /home  以人类可读格式也就是 kb、mb 等显示文件/目录大小
du -sh 查看当前文件夹大小
```

## tail

linux tail命令用途是依照要求将指定的文件的最后部分输出到标准设备，通常是终端，通俗讲来，就是把某个档案文件的最后几行显示到终端上，假设该档案有更新，tail会自己主动刷新，确保你看到最新的档案内容。

## ps

Linux ps命令用于显示当前进程 (process) 的状态。

## grep

grep (global search regular expression(RE) and print out the line,全面搜索正则表达式并把行打印出来)是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

将/etc/passwd，有出现 root 的行取出来

```
# grep root /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
或
# cat /etc/passwd | grep root 
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
```

##  awk命令的应用