---
title: "攻防世界-逆向题笔记{open-source}"
description: "攻防世界-逆向题笔记{open-source}"
tags: ["CTF"]
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
分析对象：open-source

工具：python

步骤：1）、阅读源代码

2）、分析源代码

3）、编写脚本运行

**一、阅读源代码**

![/posts/CTF/攻防世界-逆向题笔记{open-source}/document_image_rId4.png](/posts/CTF/攻防世界-逆向题笔记{open-source}/document_image_rId4.png)

用到了命令参数。并且每个参数都有规定，flag是根据参数分析出来的。

**二、分析源代码**

最总表达式为：

unsigned int hash = first * 31337 + (second % 17) * 11 + strlen(argv[3]) - 1615810207;

那么拆分来看看：

first：明显为0xcafe

second:这个就稍微有点小麻烦，要求 (对 5 求 等于 3) 或者 (对 17 求余不能为 8)，那么我们实现他的反证，即 (对 5 求余 不等于3) 且 (对17求余 等于8)，这个写一个遍历器就可以了。

strlen(argv[3]):很明显长度直接可以看出来，为 7

**三、编写脚本运行**

![/posts/CTF/攻防世界-逆向题笔记{open-source}/document_image_rId5.png](/posts/CTF/攻防世界-逆向题笔记{open-source}/document_image_rId5.png)

本人只测试了second为第一个值的情况，是正确的。其他值并未测试，估计不正确。

**总结：**

这题太简单，没什么好总结 ，就是这么膨胀！ =。=