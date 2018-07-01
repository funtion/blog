---
layout: post
date: 2015-03-12 13:49
status: public
tags: [linux, kernal, sys-call, 操作系统, 系统调用]
title: '向 Linux 增加系统调用'
categories: [Programming]
---

## 0x00 环境准备
下面的操作都是在Ubuntu下进行的，其他发行版在原理上市一样的。
首先，我们需要去 [这里](https://www.kernel.org/)下载最新的linux内核，我当时用的版本是3.19.1。把下载到的文件解压到 /usr/src/linux-3.19.1/ 这个目录下.
在进行修改和编译内核之前，需要先安装 ccache，这是一个编译的加速软件，可以将编译的结果缓存起来，在反复编译内核的时候就会加快速度。
如果是Ubuntu系统的话可以用下面的命令安装
```
sudo apt-get install ccache
```
然后建立软连接
```
sudo ln -s /usr/bin/ccache /usr/local/bin/gcc
sudo ln -s /usr/bin/ccache /usr/local/bin/g++
sudo ln -s /usr/bin/ccache /usr/local/bin/cc
sudo ln -s /usr/bin/ccache /usr/local/bin/c++
```
这样在make的时候就会直接调用ccache了
如果想要看ccache的运行情况，可以用
```
ccache -s
```
同时我们还要安装 ncurses
```
  sudo apt-get install libncurses5-dev libncursesw5-dev
```
##  0x01添加代码
在 /usr/src/linux-3.19.1/这个目录下新建一个文件夹hello,里面hello.c 
```
#include <linux/kernel.h>
asmlinkage long sys_hello(void)
{
        printk(“hello kernel ”);
        return 0;
}
```
只是用printk在内核里输出一个 “hello kernel“。
同时在/usr/src/linux-3.19.1/inclue/linux/syscalls.h最后增加
```
asmlinkage long sys_hello(void);
```
为了使这个文件能够编译进内核，需要修改Makefile。Linux内核编译使用的是kbuild，具体可以参照 [这里](http://lwn.net/Articles/21835/).
首先在 /usr/src/linux-3.19.1/hello下面新建一个Makefile 内容为
```
obj-y := hello.o
```
然后打开/usr/src/linux-3.19.1/Makefile,找到
```
core-y += kernel/ mm/ fs/ ipc/ security/ crypto/ block/ 
```
改为
```
core-y += kernel/ mm/ fs/ ipc/ security/ crypto/ block/ hello/
```
接下来，需要在系统调用的表里对其进行配置。
由于我用的是64位系统，打开/usr/src/linux-3.19.1/arch/x86/syscalls/syscall_64.tbl，在
```
#
# x32-specific system call numbers start at 512 to avoid cache impact
# for native 64-bit operation.
#
```
之前增加
```
323 common hello sys_hello
```
其中`323`是比前面已经定义好的系统调用不同的数字,以免和其它的调用冲突。
## 0x02 编译内核
1.先把之前的配置复制到/usr/src/linux-3.19.1/下来
```
sudo cp /usr/src/linux-headers-3.13.0-44-generic/.config .config
```
其中3.13.0-44是当前的版本号，如果不知道，可以通过`uname -a `来查看
2.运行
```
sudo make menuconfig
```
会显示一个配置窗口，直接load 然后save就可以
3.执行
```
sudo make bzImage
sudo make
sudo make modules_install install
```
进行编译，如果是第一次会进行很长时间，以后再编译就会比较快了。一直等到它编译完成，重启虚拟机。
## 0x03 测试
测试一下我们的系统调用有没有添加成功，在任意一个地方新建一个c++文件，代码如下，编译并运行
```
#include <iostream>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>
using namespace std;
int main(){
  long x = syscall(323);
  cout<"kernel test";
  cout<<x<<endl;
  return 0;
}
```
其中 `323` 是刚添加的系统调用号。
输出
```
kernel test
0
```
运行
```
dmesg
```
可以在最后看到printk的输出
```
[ 2871.163857] hello kernel
```
证明系统调用已经添加成功。