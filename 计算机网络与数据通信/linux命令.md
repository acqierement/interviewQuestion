# top、free、df、du、tail、ps、grep命令

## top

[Linux系统](https://www.baidu.com/s?wd=Linux%E7%B3%BB%E7%BB%9F&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)可以通过top命令查看系统的CPU、内存、运行时间、交换分区、执行的线程等信息。通过top命令可以有效的发现系统的缺陷出在哪里。是内存不够、CPU处理能力不够、IO读写过高….

## free

free 命令显示系统内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存。

## df

df（disk filesystem 的简称）用于显示 Linux 系统的磁盘利用率。（LCTT 译注：`df` 可能应该是 disk free 的简称。）

linux中df命令的功能是用来检查linux服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

## du

`du`（disk usage 的简称）是用于查找文件和目录的磁盘使用情况的命令。

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

