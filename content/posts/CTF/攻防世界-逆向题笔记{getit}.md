---
title: "攻防世界-逆向题笔记{getit}"
description: "攻防世界-逆向题笔记{getit}"
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
分析对象：getit

工具：Ida、kali、edb、python

步骤：1）、运行查看效果

2）、PE工具查壳

3）、查看文件平台

4）、如果有壳就脱壳，无壳进行第四步

5）、文件拖入ida进行分析

6）、解密

**一、运行查看效果**

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId4.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId4.png)

emmm。好像没什么效果。

**二、PE工具查壳**

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId5.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId5.png)

无壳

**三、查看文件平台**

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId6.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId6.png)

64位的文件

**四、脱壳**

无

**五、Ida分析**

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId7.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId7.png)

ida加载完后直接搜索字符串，看到类似flag的东西，但是被加密了，直接去main函数看看。

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId8.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId8.png)

t和s的内容双击变量然后按下**A(数据->字符串的快捷键),** 代码很清晰，那么获得flag的方法依然有两种，一种是脚本，一种是调试器下断看数据窗口。

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId9.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId9.png)

解密之前需要注意的是这个10(图中是十进制),它所表达的含义就是t字符串的前10个字符，因为程序是从**harifCTF{**这段之后开始加密的

**六、解密**

**第一种方法：**

**直接上python脚本**

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId10.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId10.png)

**第二种方法：**

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId11.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId11.png)

关键语句，此处正是加密flag出，记录偏移地址，直接在调试器跳转到此处后，在下一条代码下断。

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId12.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId12.png)

运行

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId13.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId13.png)

断下之后，由于知道t是一个常量，所以很容易判断出红框中的硬编码就是t的地址，我们在数据窗口跳转到该地址即可看到flag。

![/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId14.png](/posts/CTF/攻防世界-逆向题笔记{getit}/document_image_rId14.png)

**总结：**

**无**