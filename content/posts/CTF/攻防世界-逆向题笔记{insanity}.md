---
title: "攻防世界-逆向题笔记{insanity}"
description: "攻防世界-逆向题笔记{insanity}"
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
分析对象：insanity

工具：Kali-Linux、Ida

步骤：1）、运行查看效果

2）、PE工具查壳

3）、查看文件平台

4）、如果有壳就脱壳，无壳进行第四步

5）、文件拖入ida进行分析

**一、运行查看效果**

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId4.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId4.png)

**二、PE工具查壳**

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId5.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId5.png)

无壳

**三、查看文件平台**

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId6.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId6.png)

清楚看到是32bit obj，所以该文件为32位。我们直接打开32位ida。

**四、脱壳**

无

**五、Ida分析**

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId7.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId7.png)

依旧先搜索字符串

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId8.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId8.png)

发现了类似明文的flag。我们去看看**main函数**的伪代码。

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId9.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId9.png)

我们双击**strs**看看内容

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId10.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId10.png)

发现这其中存着着许多字符串并且还包含了刚刚看到的flag，返回到**main**函数分析一下最后一个put

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId11.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId11.png)

发现输出的内容由**v4**决定，而**v4**又是随机数。所以知道该程序每次运行都会随机**strs**里的内容，可能偶然会输出flag。

![/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId12.png](/posts/CTF/攻防世界-逆向题笔记{insanity}/document_image_rId12.png)

**总结：**

无总结