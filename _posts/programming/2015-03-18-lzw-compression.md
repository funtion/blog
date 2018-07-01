---
layout: post
date: 2015-03-18 13:20
status: public
tags: [algorithm, LZW, compression]
title: 'LZW 算法之压缩'
categories: [Programming]
---

LZW (Lempel–Ziv–Welch) 压缩是一种基于表的无损压缩。在压缩时，会将反复出现的部分放入表格进行编号，而最终输出的结果就是编号的序列。在解压的时候通过查表就可以还原出原始的数据。

通常基于表的压缩方法需要将表格附在压缩的数据上以便解压，然而 LZW 采用了一种巧妙的方法，不需记录表格信息，而是在解压的时候，一遍解压一边进行表的构造，完成整个算法。
压缩的核心伪代码表示如下
```
1. 初始化字典dict，只包含长度为1的元素
2. w = "", k = “”
3. 从输出中读取一个字母到 k, w = w + k
4. 如果w出现在字典中，跳转到 3 
5. 否则，输出 dict[ w[:-1] ]
6. 将 w 加入到字典中
7. w = k
8. 跳转到 3
```

也就是说，在压缩的时候我们先构造出一个基础的表，然后贪心的从输入里面读取信息直到它不出现在表中。那么这个没出现过字符串w的去掉最后一个字母子串一定出现过，我们把它的编号输出，然后将w加入到字典中。
比如说要压缩这样一个字符串
```
 abaaabba
 ```
首先初始化dict
```
dict={a=0, b=1}
```
然后从头开始扫描这个字符串
第一次读到最长的 w 后(下面 `[]`中表示的是当前的 w)
```
[ab]aaabba
```

所以输出`0`即`dict[a]`，同时dict变为
```
dict={a=0, b=1, ab=2}
```
第二次读到最长的 w 后
```
a[ba]aabba
```
所以输出`1`即`dict[b]`，同时dict变为
```
dict={a=0 ,b=1, ab=2. ba=3}
```
第三次读到最长的w后
```
ab[aa]abba
```
所以输出`0`即`dict[a]`，同时dict变为
```
dict={a=0, b=1, ab=2. ba=3, aa=4}
```
第四次读到最长的w后
```
aba[aab]ba
```
所以输出`4`即`dict[aa]`，同时dict变为
```
dict={a=0, b=1, ab=2, ba=3, aa=4, aab=5}
```
第五次读到最长的w后
```
abaaa[bb]a
```
所以输出`1`即`dict[b]`，同时dict变为
```
dict={a=0, b=1, ab=2, ba=3, aa=4, aab=5, bb=6}
```
第六次读到最长的w后
```
abaaab[ba]
```
所以输出`3`即`dict[ba]`，因为已经结束且ba出现过，所以dict不变
```
dict={a=0, b=1, ab=2, ba=3, aa=4, aab=5, bb=6}
```
最终的输出为`0,1,0,4,1,3`

关于解压的算法，可以看[这里](http://www.functor.me/post/programming/lzw-decompression)