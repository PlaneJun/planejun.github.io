---
title: "QBot-Outlook窃密"
description: "QBot-Outlook窃密"
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

# QBot-Outlook窃密

# ⼀、加载器分析

## **1**、样本标签

| 病毒名称 | c8810d5eaaea95b36bbb529a2b9be5c5e6dda10f95992e7c35ac8bbf9f3a8f71 |
| --- | --- |
| 原始⽂件名 | ⽆ |
| MD5 | 93d6d599c37d1858cc86c0d8fe8fb8d4 |
| ⽂件⼤⼩ | 222.50 KB (227840 bytes) |
| ⽂件格式 | Win32 DLL |
| 时间戳 | 2020-04-20 23:33:29 |
| 加壳类型 | ⽆ |
| 编译语⾔ | C/C++ |
| SHA1 | f85a63cb462b8fd60da35807c63cd13226907901 |
| CRC32 | F0A18B09 |

## 2、详细分析

样本(93D6D599C37D1858CC86C0D8FE8FB8D4)⾃⾝为DLL⽂件,同时也是⼀个加载器。样本⼀旦被加载⾸先获取远线程注⼊所需要的API函数地址。

![/posts/Virus/QBot-Outlook窃密/image1.jpeg](/posts/Virus/QBot-Outlook窃密/image1.jpeg)

其中样本中的所有字符串都是加密的，通过传⼊字符串对应的id解密出字符串。

![/posts/Virus/QBot-Outlook窃密/image2.png](/posts/Virus/QBot-Outlook窃密/image2.png)

![/posts/Virus/QBot-Outlook窃密/image3.jpeg](/posts/Virus/QBot-Outlook窃密/image3.jpeg)

获取函数之后会通过token设置sid的⽅式进⾏提权。

![/posts/Virus/QBot-Outlook窃密/image4.png](/posts/Virus/QBot-Outlook窃密/image4.png)

根据当前运⾏的环境是否为64位来获取Windows⽬录。接着在⽂件夹**ProgramW6432ProgramFiles(x86)**、**ProgramFiles**中搜索**outlook.exe**⽂件。

![/posts/Virus/QBot-Outlook窃密/image5.jpeg](/posts/Virus/QBot-Outlook窃密/image5.jpeg)

![/posts/Virus/QBot-Outlook窃密/image6.jpeg](/posts/Virus/QBot-Outlook窃密/image6.jpeg)

如果存在outlook.exe⽂件后会根据⾃⾝进程位数来选择32位还是64位模式注⼊。

![/posts/Virus/QBot-Outlook窃密/image7.png](/posts/Virus/QBot-Outlook窃密/image7.png)

注⼊⾸先会创建⼀个**ping.exe**进程作为傀儡，并且运⾏参数为**-t 127.0.0.1**，保证傀儡程序能持续运⾏。

![/posts/Virus/QBot-Outlook窃密/image8.png](/posts/Virus/QBot-Outlook窃密/image8.png)

然后加载名为RES_DATA_1(如果是64位注⼊则为RES_DATA_2)的资源⽂件。

![/posts/Virus/QBot-Outlook窃密/image9.png](/posts/Virus/QBot-Outlook窃密/image9.png)

# ⼆、邮箱窃密模块分析

由于32位和64位功能相同，这⾥只分析32位。

## **1**、样本标签

| 病毒名称 | RES_DATA_1 |
| --- | --- |
| 原始⽂件名 | emailcollector.dll |
| MD5 | 9B0A4E5E5227B766EDE92340D7064545 |
| ⽂件⼤⼩ | 84.00 KB (86016 bytes) |
| ⽂件格式 | Win32 DLL |
| 时间戳 | 2020-04-20 23:33:01 |
| 加壳类型 | ⽆ |
| 编译语⾔ | C/C++ |
| SHA1 | 369EE5FF9960BB9E73503E0B0A4B3A57A96EC549 |
| CRC32 | 1654A136 |

## 2、详细分析

样本(9B0A4E5E5227B766EDE92340D7064545)为核⼼模块，负责窃取outlook的邮箱数据。当样本⾃⾝被成功加载时，与加载器相同。通过GetProAddress获取⼀些API函 数，其中字符串也是被加密状态，使⽤的算法也与加载器相同。

![/posts/Virus/QBot-Outlook窃密/image10.jpeg](/posts/Virus/QBot-Outlook窃密/image10.jpeg)

初始化完毕后会通过**CreateThread**函数创建⼀个线程，开始进⾏窃密。

![/posts/Virus/QBot-Outlook窃密/image12.png](/posts/Virus/QBot-Outlook窃密/image12.png)

### 2.1、窃密邮箱数据

样本⾸先会将当前⼯作⽬录设置为“C:\User\USERNAME\”（该字符串在初始化时获取）

![/posts/Virus/QBot-Outlook窃密/image13.jpeg](/posts/Virus/QBot-Outlook窃密/image13.jpeg)

然后在当前⽬录下创建⼀个⽂件夹，格式为“**EmailStorage_计算机名_当前时间戳**”。

![/posts/Virus/QBot-Outlook窃密/image14.jpeg](/posts/Virus/QBot-Outlook窃密/image14.jpeg)

![/posts/Virus/QBot-Outlook窃密/image15.png](/posts/Virus/QBot-Outlook窃密/image15.png)

接着拼接⼀个名为collector_log.txt的⽂件名，⽤于输出执⾏的⽇志。

![/posts/Virus/QBot-Outlook窃密/image16.jpeg](/posts/Virus/QBot-Outlook窃密/image16.jpeg)

样本通过MAPIXXXXXXX函数访问outlook的数据，获取当前outlook所存在的邮箱账号。

![/posts/Virus/QBot-Outlook窃密/image17.png](/posts/Virus/QBot-Outlook窃密/image17.png)

然后开始遍历每⼀个邮箱账号，获取详细信息。

![/posts/Virus/QBot-Outlook窃密/image18.png](/posts/Virus/QBot-Outlook窃密/image18.png)

为每个邮箱账号创建名为**”当前索引”**的⽂件夹⽤来保存数据，接着创建线程sub_10002af8执⾏。

![/posts/Virus/QBot-Outlook窃密/image19.jpeg](/posts/Virus/QBot-Outlook窃密/image19.jpeg)

获取当前邮箱账号所存在的⽂件夹，⽂件夹的名字从数字0开始递增。

![/posts/Virus/QBot-Outlook窃密/image20.png](/posts/Virus/QBot-Outlook窃密/image20.png)

然后获取⽂件夹中的所有邮件，每个邮件的命名从数字**0**开始递增。

![/posts/Virus/QBot-Outlook窃密/image22.png](/posts/Virus/QBot-Outlook窃密/image22.png)

![/posts/Virus/QBot-Outlook窃密/image23.png](/posts/Virus/QBot-Outlook窃密/image23.png)

![/posts/Virus/QBot-Outlook窃密/image24.png](/posts/Virus/QBot-Outlook窃密/image24.png)

每获取完⼀个⽂件夹中的邮件时，会⽣成⼀个名为”folder.txt”的⽂件⽤来存储当前⽂件夹的名字。

![/posts/Virus/QBot-Outlook窃密/image25.png](/posts/Virus/QBot-Outlook窃密/image25.png)

![/posts/Virus/QBot-Outlook窃密/image26.png](/posts/Virus/QBot-Outlook窃密/image26.png)

当邮箱中所有⽂件夹都获取完毕后，⽣成⼀个名为“email.txt”的⽂件⽤来存储当前邮箱账号。

![/posts/Virus/QBot-Outlook窃密/image27.png](/posts/Virus/QBot-Outlook窃密/image27.png)

![/posts/Virus/QBot-Outlook窃密/image28.png](/posts/Virus/QBot-Outlook窃密/image28.png)

### 2.2、数据回传

如果能窃取到outlook邮箱数据，则进⾏数据回传。

![/posts/Virus/QBot-Outlook窃密/image29.png](/posts/Virus/QBot-Outlook窃密/image29.png)

- 上传Log

样本将log⽂件读取到内存后，以zip压缩、base64编码的⽅式对内容进⾏加密。

![/posts/Virus/QBot-Outlook窃密/image30.png](/posts/Virus/QBot-Outlook窃密/image30.png)

然后⽣成⼀个json⽂件。

![/posts/Virus/QBot-Outlook窃密/image31.png](/posts/Virus/QBot-Outlook窃密/image31.png)

然后⽤**https+post**的⽅式回传⾄攻击者的服务器**hxxps://82.118.22.125/bgate**

![/posts/Virus/QBot-Outlook窃密/image33.png](/posts/Virus/QBot-Outlook窃密/image33.png)

- 上传邮箱内容

遍历获取窃取到的邮件⽂件。

![/posts/Virus/QBot-Outlook窃密/image34.png](/posts/Virus/QBot-Outlook窃密/image34.png)

![/posts/Virus/QBot-Outlook窃密/image35.png](/posts/Virus/QBot-Outlook窃密/image35.png)

将遍历到的邮件⽂件读取到内存。

![/posts/Virus/QBot-Outlook窃密/image37.jpeg](/posts/Virus/QBot-Outlook窃密/image37.jpeg)

同样先将读取到的数据进⾏zip压缩，然后使⽤base64编码加密。

![/posts/Virus/QBot-Outlook窃密/image38.png](/posts/Virus/QBot-Outlook窃密/image38.png)

然后将数据以json格式进⾏打包。

![/posts/Virus/QBot-Outlook窃密/image40.png](/posts/Virus/QBot-Outlook窃密/image40.png)

![/posts/Virus/QBot-Outlook窃密/image41.jpeg](/posts/Virus/QBot-Outlook窃密/image41.jpeg)

最后回传到攻击者服务器。

![/posts/Virus/QBot-Outlook窃密/image42.png](/posts/Virus/QBot-Outlook窃密/image42.png)

# 三、样本运⾏环境搭建

![/posts/Virus/QBot-Outlook窃密/image43.png](/posts/Virus/QBot-Outlook窃密/image43.png)

![/posts/Virus/QBot-Outlook窃密/image44.png](/posts/Virus/QBot-Outlook窃密/image44.png)

> 添加测试的QQ邮箱，添加途中需要到QQ邮箱开通IMAP权限，获取应⽤密码。
> 
> 
> ![/posts/Virus/QBot-Outlook窃密/image45.png](/posts/Virus/QBot-Outlook窃密/image45.png)
> 

![/posts/Virus/QBot-Outlook窃密/image46.jpeg](/posts/Virus/QBot-Outlook窃密/image46.jpeg)

![/posts/Virus/QBot-Outlook窃密/image47.jpeg](/posts/Virus/QBot-Outlook窃密/image47.jpeg)

然后设置Outlook为默认使⽤的邮件软件。Win+i打开设置→应⽤→默认设置。

![/posts/Virus/QBot-Outlook窃密/image48.jpeg](/posts/Virus/QBot-Outlook窃密/image48.jpeg)

使⽤编写代码⽤来加载样本(93d6d599c37d1858cc86c0d8fe8fb8d4)

![/posts/Virus/QBot-Outlook窃密/image49.png](/posts/Virus/QBot-Outlook窃密/image49.png)

打开fakenet和⽕绒剑后将编写好的程序拖⼊⽕绒剑进⾏监控。（因为样本使⽤的是HTTPS加密协议，需要过滤⽹络数据才能看到明⽂数据，所以使⽤fakenet）。

![/posts/Virus/QBot-Outlook窃密/image50.jpeg](/posts/Virus/QBot-Outlook窃密/image50.jpeg)

可以看到样本执⾏过程中在遍历⽬录寻找Outlook.exe。

![/posts/Virus/QBot-Outlook窃密/image51.jpeg](/posts/Virus/QBot-Outlook窃密/image51.jpeg)

且傀儡程序被加载。

![/posts/Virus/QBot-Outlook窃密/image52.jpeg](/posts/Virus/QBot-Outlook窃密/image52.jpeg)

可以看到⽂件已经被窃取到指定⽬录下

![/posts/Virus/QBot-Outlook窃密/image53.png](/posts/Virus/QBot-Outlook窃密/image53.png)

fakenet也能捕获到回传的数据。

![/posts/Virus/QBot-Outlook窃密/image54.jpeg](/posts/Virus/QBot-Outlook窃密/image54.jpeg)

由于样本发送后会将⽂件删除，所以重复上⾯的操作，利⽤⽕绒剑⾃定义防护，禁⽌⽂件删除。

![/posts/Virus/QBot-Outlook窃密/image55.png](/posts/Virus/QBot-Outlook窃密/image55.png)

拿到窃取的⽂件

![/posts/Virus/QBot-Outlook窃密/image56.png](/posts/Virus/QBot-Outlook窃密/image56.png)

![/posts/Virus/QBot-Outlook窃密/image57.jpeg](/posts/Virus/QBot-Outlook窃密/image57.jpeg)

![/posts/Virus/QBot-Outlook窃密/image58.png](/posts/Virus/QBot-Outlook窃密/image58.png)

Outlook的⽂件夹数为14，刚好对应上窃取到的⽂件夹编号。

![/posts/Virus/QBot-Outlook窃密/image59.png](/posts/Virus/QBot-Outlook窃密/image59.png)

且每个⽂件夹下都保存着实际⽂件夹名。

![/posts/Virus/QBot-Outlook窃密/image60.jpeg](/posts/Virus/QBot-Outlook窃密/image60.jpeg)

# 四、IoC

**URL**：

hxxps://82.118.22.125/bgate

**MD5**：

93D6D599C37D1858CC86C0D8FE8FB8D4

9B0A4E5E5227B766EDE92340D7064545

18152190D3637C3A24BA40046324B79C

**开源报告：**

(An Old Bot’s Nasty New Tricks: Exploring Qbot’s Latest Attack Methods)[https://research.checkpoint.com/2020/exploring-qbots-latest-attack-methods/](https://research.checkpoint.com/2020/exploring-qbots-latest-attack-methods/)