---
title: "一个易语言SouGou蠕虫样本"
description: "一个易语言SouGou蠕虫样本"
tags: ["Virus"]
ShowReadingTime: true
ShowShareButtons: false
ShowPostNavLinks: true
ShowBreadCrumbs: true
ShowCodeCopyButtons: false
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
disableSpecial1stPost: false
disableScrollToTop: false
comments: false
hidemeta: false
hideSummary: false
showtoc: true
tocopen: true
---

# 一个易语言SouGou蠕虫样本

- [PlaneJun](https://bbs.pediy.com/user-home-826671.htm) ・2022-06-22 13:29

# 一、概述

该样本为一个被感染了蠕虫的样本，原程序为**Sougou.exe**，且有正规产商签名。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId5.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId5.png)

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId6.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId6.png)

样本被运行后首先会执行蠕虫病毒，最后再执行原程序。蠕虫病毒会遍历电脑磁盘，并且感染除C盘外且后缀名为.docx和.exe的文件，其中感染.docx的方式为在文件头添加字符串`HELLO！`，感染.exe的方式为在程序头添加蠕虫病毒。

# 二、样本信息

| 样本名 | SouGou.exe|
| - | -|
| MD5| 233440021438C6B4D97AD822EB434842| 
| SHA-1| 051A9C3F8DA2EC70C6DD46CBA3F550D56F14D30D| 
| File size| 1.02 MB (1,075,954 字节)| 


通过`Resource Hacker`工具查看资源发现为易语言的黑月编译器编译。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId7.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId7.png)

# 三、详细分析

## 1、SouGou详细分析

| 样本名 | SouGou.exe|
| - | -|
| MD5| 233440021438C6B4D97AD822EB434842| 
| SHA-1| 051A9C3F8DA2EC70C6DD46CBA3F550D56F14D30D| 
| File size| 1.02 MB (1,075,954 字节)| 

由于是黑月编译器，直接通过对函数nullsub_1交叉引用后来到真实的程序入口点。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId8.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId8.png)

样本首先是将自身读入内存，然后以字符串"**>>>>><-----<>>>>>**"为目标进行数据切割。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId9.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId9.png)

紧接着会判断同级路径下是否存在名为**下划线(_)+样本名的.exe**的可执行文件，如存在则运行。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId10.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId10.png)

不存在则将刚刚切割得到的数据以该文件名写出后运行。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId11.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId11.png)

此处真正目的为运行出**原程序**。运行原程序后随即写出主模块并运行。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId12.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId12.png)

## 2、msimg32详细分析

| 样本名 | msimg32.exe|
| - | -|
| MD5| 8B16E470E4324FDB74EDAAA9C95450AF| 
| SHA-1| 993FE67ED998BB7F3F7B9F38C721701ECE766997| 
| File size| 676 KB (692,224 字节)

该模块同样为黑月编译器编译，直接跳转到入口。发现为创建线程方式执行。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId13.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId13.png)

首先判断是否存在移动硬盘，判断方式也很简单，直接通过固定路径判断。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId14.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId14.png)

如果存在移动硬盘，则将主模块msimg32.exe拷贝到移动硬盘中，并运行主模块msimg32.exe。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId15.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId15.png)

如果不存在移动硬盘，则创建一个时间周期为100毫秒的始终开始执行事件。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId16.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId16.png)

事件开始会获取当前日期天数，如果天数为奇数则释放感染模块rundllx.exe并运行。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId17.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId17.png)

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId18.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId18.png)

反之，如果天数为偶数，则释放检测模块wsock32.exe，并运行。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId19.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId19.png)

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId20.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId20.png)

释放完毕后，主模块会进行写注册表项方式实现自启动。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId21.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId21.png)

并且禁止进入安全模式、隐藏文件扩展名、隐藏文件夹操作。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId22.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId22.png)

## 3、rundllx详细分析

| 样本名 | rundllx.exe|
| - | -|
| MD5| C6C3DD31B15753FEF32ABC053A04138C| 
| SHA-1| B495326942038B891387B71B9F0E41C4E755F003| 
| File size| 288 KB (294,912 字节)

该模块为最终进行感染操作的模块，同样为黑月编译器编译。代码入口获取了磁盘信息。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId23.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId23.png)

并且开始扫描除C盘外的磁盘。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId24.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId24.png)

使用了递归的方法对磁盘下的目录进行扫描，扫描的文件格式为.docx和.exe。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId25.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId25.png)

扫描完毕后开始进行文件感染。

### ①EXE感染

首先是判断文件后缀名是否为.exe文件。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId26.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId26.png)

然后判断文件属性是否为**非系统文件**和**非隐藏文件**。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId27.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId27.png)

在同级目录下写出T.exe文件并读入内存。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId28.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId28.png)

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId29.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId29.png)

同样操作Z.exe。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId30.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId30.png)

最后将原程序数据与T.exe、Z.exe合并后写出，完成对原程序的感染。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId31.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId31.png)

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId32.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId32.png)

带加密完毕后，删除T.exe和Z.exe。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId33.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId33.png)

### ②DOCX感染

判断是否为.docx文件。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId34.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId34.png)

加密方式为在文件头插入字符串"**HELLO！**"数据。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId35.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId35.png)

## 4、wsock32详细分析

| 样本名 | wsock32.exe|
| - | -|
| MD5| A29DA8DDBB40BEA8EB65E8101E41786D| 
| SHA-1| 81D7DC554212CC9B931F87E378720293F9B50D39| 
| File size| 188 KB (192,512 字节)

该模块为主要负责检测msimg32是否在运行，若不运行则运行msimg32，同样为黑月编译器编译。入口为创建了一个时间周期为100毫秒的时钟。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId36.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId36.png)

事件入口判断msimg32进程是否存在，不存在则运行。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId37.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId37.png)

然后将自身添加到注册表启动项。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId38.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId38.png)

# 四、还原文件

## 1、EXE文件

直接以字符串"**>>>>><-----<>>>>>MZ**"为目标进行数据切割。将头部的蠕虫程序去除即可。

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId39.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId39.png)

## 2、DOCX文件

直接删除文件头的字符串"**HELLO！**"

![/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId40.png](/posts/Virus/一个易语言SouGou蠕虫样本/document_image_rId40.png)