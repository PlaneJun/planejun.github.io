---
title: "攻防世界-逆向题笔记{csaw2013reversing2}"
description: "攻防世界-逆向题笔记{csaw2013reversing2}"
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
分析对象：csaw2013reversing2

工具：Ida、od、vs

步骤：1）、运行查看效果

2）、PE工具查壳

3）、查看文件平台

4）、如果有壳就脱壳，无壳进行第四步

5）、文件拖入ida进行分析

6）、解密

**一、运行查看效果**

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId4.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId4.png)

直接提示了flag，但是是乱码。我们查一下壳。

**二、PE工具查壳**

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId5.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId5.png)

莫得壳

**三、查看文件平台**

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId6.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId6.png)

32位，我们直接打开32位ida分析

**四、脱壳**

无

**五、Ida分析**

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId7.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId7.png)

日常字符串搜索，但是除了就发现一个MessageBoxW的函数。我们直接去main函数看看。

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId8.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId8.png)

看到memcpy，我们看看他复制了什么。

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId9.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId9.png)

然后我们继续往下看代码可以看到一个反调试函数**IsDebuggerPresent()**，如果这个程序被调试，那么函数返回**1**，否则返回**0**。然后大概我们去看一下这个**sub_40102A()。**

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId10.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId10.png)

看到他这个函数默认返回0，而且我们直接运行程序后，**IsDebuggerPresent()**也是0(**因为我i们没有调试**)，所以if里面的函数没有被执行。而if中的这个函数**sub_401000(v3 + 4, (int)lpMem)，**对lpMem做了手脚，所以猜测他是给flag运算。

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId11.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId11.png)

进来之后，看到算法。这里说一下两种方法，第一种依旧是用到调试器OD，第二种是直接跑脚本。

**六、解密**

**第一种方法：**

找到if判断的地方，把跳转给nop或者改jmp，直接让其执行加密函数。大家可以在ida记录main函数的偏移，然后在od通过 **镜像地址+偏移** 得到main函数，也可以手动单步跟。这里我就直接单步跟。

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId12.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId12.png)

找到关键代码

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId13.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId13.png)

修改后，直接在加密函数的位置下断（**这里有一个坑，需要把int3给nop掉，因为这个int3本身是一个中断指令，导致程序直接崩掉**）

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId14.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId14.png)

运行后直接断下，我们F7进去。

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId15.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId15.png)

这里对应着ida这里

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId16.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId16.png)

说明flag存在edx。我们在函数末尾下断后F9运行，直接查看edx的值。

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId17.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId17.png)

**第二种方法：**

lpMenm的值上面已经找到了，直接写脚本。

![/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId18.png](/posts/CTF/攻防世界-逆向题笔记{csaw2013reversing2}/document_image_rId18.png)

这里就上一份百度来的代码。mlgb，别问为什么是C++，因为我python写不出，C++我也写不出。

**总结：**

无