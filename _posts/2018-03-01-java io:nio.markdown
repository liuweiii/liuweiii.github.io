---
layout: post
title:  "java io/nio"
date:   2018-03-01 21:24:20 +0800
categories: java
tags:
- learn
- java
- io
- nio
---
## IO
### stream/reader
- Input/Output stream 读写字节（byte），Reader 读写字符（character）。

``` 
 byte：8bit
 char：2 * 8bit
 int：4 * 8bit 
```
- InputStream的long skip(long bytesToSkip)方法可以跳过bytesToSkip个字节，这个方法比“读取后再忽略”更有效率，因为skip只是把读取的指针往前移。

- 操作系统和硬件的flush方法一般不会清空buffer，java的OutputStream的flush方法会清空buffer

- 通常java的io操作相关的函数都会抛出IOException，但System.out是个特殊的PrintStream，它的写方法不会抛出IOException，这样设计上位了调用简单。

- 为什么stream用完了需要close？因为操作系统上的很多资源都是有限的，比如一般操作系统不能同时打开几百个文件。close方法会告诉操作系统，可以释放与这个stream关联的资源了。但是System.in不需要close。

- mark和reset
 一些input stream支持mark(int readLimit)和reset（如BufferedInputStream、ByteArrayInputStream）。调用mark会在当前位置打上一个标签,当读取不超过readLimit字节的时候调用reset可以回到当前位置。mark只能打一个标签，打多个标签会覆盖以前的标签。

```
try {
URL u = new URL("http://www.digitalthink.com/");
URLConnection uc = u.openConnection();
uc.connect();
InputStream in = uc.getInputStream();
//...
}
catch (IOException e)
```

## NIO

### buffer
![java-nio-1](/public/img/2018-03-17-java-nio-buffer1.png)
#### 属性：
##### capacity 容量
buffer里能存储元素的最大个数，buffer创建后就不能修改。
##### limit 上限
最后一个有效元素的下一个位置。比如往capacity=100的buffer中，从头开始存入10个元素，则limit=10，limit的作用在于channel从buffer读数据的时候知道读到limit这个位置就读完了。
##### position 位置
下一个可以写入元素的位置。get和put（读写）能改变它。
##### mark 标记
标记一个位置，可以用于其他作用，如一般channel读取的时候是从position开始读到limit位置。如果想从中间位置开始读，可以先mark现在的位置，把position设置为中间，读完后再把position设置为刚才mark的位置。
#### 方法
##### get读/put写
两种get方法： 

1. 无参数的get/put。一次读/写一个元素，postion会加1
2. 带参数（index）的get/put。读取/写入一个位于index位置的元素，postion不会变。

##### limit上限
设置上限值。

##### flip翻转
使limit=position，position=0，mark=-1，目的是写完毕了送入channel的时候重置position和limit使适合读（从position读到limit，即从0读到原来写入完毕读位置）。

##### compact压缩
为了从缓冲区中释放一部分数据，然后重新填充，需要把释放后剩下的数据全部往左边移动。
![java-nio-1](/public/img/2018-03-17-java-nio-buffer2.png)

##### mark标记
设置mark值。

##### reset重置
使position=mark。

##### clear清空
使positon=0，limit=capacity，mark=-1

#### rewind 倒带
使positon=0，mark=-1

##### equals相等
只要[position，limit)之间的元素一一相等（类型、值都一样）即可（如buffer1的position=2，limit=5，buffer2的position=10，limit=13，它们两个的[position，limit)间的元素一一相等，它们两equals就是true）。

##### compareTo比较
也是比较两个buffer的[position，limit)之间的元素，返回0、1、-1，代表相等、大于、小于。

### channel

### selector