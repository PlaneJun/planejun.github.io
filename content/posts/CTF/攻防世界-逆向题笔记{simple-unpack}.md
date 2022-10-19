---
title: "攻防世界-逆向题笔记{simple-unpack}"
description: "攻防世界-逆向题笔记{simple-unpack}"
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
分析对象：simple-unpack

工具：Kali-Linux、Ida

步骤：1）、运行查看效果

2）、PE工具查壳

3）、查看文件平台

4）、如果有壳就脱壳，无壳进行第四步

5）、文件拖入ida进行分析

**前言：**

进行第一步之前，我们首先说一下这个这次题目的文件。这次的文件并非windows平台下的文件，而是属于linux文件。如何知道呢？把文件拖进PE工具就可以看到提示。

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId4.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId4.png)

不是winEXE-.o-ELF 可执行程序 64位 obj，百度看一下ELF

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId5.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId5.png)

所以我们需要虚拟机安装一个linux系统，由于咱们是搞科技人士，所以我推荐安装Kali版本。至于Kali的操作并不会多说，大家自己去学一下。这里直接上手。

**一、运行查看效果**

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId6.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId6.png)

第一句 chom 777 simple-unpack 是为了提升权限，否则可能无法运行。之后我们./simple-unpack运行之后随便输入一串内容然后回车，发现提示了Try again!

**二、PE工具查壳**

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId7.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId7.png)

很明显是UPX，这里说一下，虽然ELF并非正统的PE文件，但是通常能用PE工具查壳。还有一种查壳方式就是十六进制软件打开，查看区段，一般加壳程序都会存在一个对应壳的区段。

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId8.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId8.png)

**三、查看文件平台**

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId9.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId9.png)

这里已经很明先给出是64位。

**四、脱壳**

我们已经知道当前文件为UPX保护壳，那么我们首先得脱壳。window平台下脱UPX的壳方法很多，但是linux文件怎么脱呢？

很简单！Linux下，运行upx -d 文件名即可。注意必须是小写。

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId10.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId10.png)

linux会自动覆盖源程序，现在桌面上的程序已经被脱壳了，我们放到Window下重新查壳看看。

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId11.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId11.png)

此时已经无壳，我们直接用64为Ida打开。

**五、Ida分析**

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId12.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId12.png)

由于分析对象文件并非window平台下的可执行文件，所以这里需要选择**所有文件。**加载完毕后，首先搜索字符串，看看有没有明文之类的。

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId13.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId13.png)

本题直接可以看到Flag。

![/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId14.png](/posts/CTF/攻防世界-逆向题笔记{simple-unpack}/document_image_rId14.png)

**总结：**

此次题目主要是考查我们对一些简单脱壳和linux的基本操作。