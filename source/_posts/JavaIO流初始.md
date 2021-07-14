---
title: JavaIO流初始
date: 2021-07-08 21:48:48
tags: 学习
---

### IO流概要

#### 1. 什么是IO流?

​	I: input O: output

​	通过IO可以完成对硬盘文件的读和写

#### 2. IO流的分类

​	按照方向区分 输入流 , 输出流

​	按照读取数据方式(单位)区分: 字节流,字符流(只能读取纯文本文件,可以一个字符一个字符的读取).

#### 3.java IO流类别

​	在java IO流中,以Stream结尾的就是字节流,以Reader/Writer结尾的就是字符流.

- InputStream 字节输入流
- OutputStream 字节输出流
- Reader 字符输入流
- Writer 字符输出流

<!--more-->

---以上的流都是最为开始的流,全是抽象类

接下来会有16个所包括的流:

文件流:

- FileInputStream
- FIleOutputStream
- InputSteamReader
- OutputSteamWriter

转换流:

- FileReader
- FIleWriter

缓冲流:

- BufferedReader
- BufferedWriter
- BufferedInputStream
- BufferedOutputStream

数据流:

- DataInputSteam
- DataOutputSteam

对象流:

- ObjectInputStream
- ObjectOutputStream

标准输出流:

- PrintWriter
- PrintSteam

​	

​	