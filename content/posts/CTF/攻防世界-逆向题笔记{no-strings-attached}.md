---
title: "攻防世界-逆向题笔记{no-strings-attached}"
description: "攻防世界-逆向题笔记{no-strings-attached}"
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
分析对象：no-strings-attached

工具：Kali-Linux、Ida、python、edb

步骤：1）、运行查看效果

2）、PE工具查壳

3）、查看文件平台

4）、如果有壳就脱壳，无壳进行第四步

5）、文件拖入ida进行分析

6）、解密

6.1、手撕加密

6.2、edb调试

**一、运行查看效果**

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId4.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId4.png)

**二、PE工具查壳**

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId5.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId5.png)

无壳

**三、查看文件平台**

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId6.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId6.png)

32位文件，故使用32位ida

**四、脱壳**

无

**五、Ida分析**

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId7.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId7.png)

老方法，先搜索字符串

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId8.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId8.png)

emmm，空空如也。那只能去**main函数**了。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId9.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId9.png)

F5即可得到伪代码。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId10.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId10.png)

经过简单的分析呢，发现最后一个函数为关键函数。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId11.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId11.png)

双击进入该函数进行分析

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId12.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId12.png)

这里呢，两种方法，获取flag，也就是s2。一种是手动撕加密算法；另一种是让程序自身把所有加密运算完成后，直接获取加密的结果。由于s2的值来源于**decrypt**函数的返回值，所以我们直接在**decrypt**函数尾部，等待断下后看**eax**的值，因为**eax**的值通常为函数返回值。

**六、解密**

**第一种：**

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId13.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId13.png)

双击进入函数，发现该函数操作的是宽字节，也就是unicode(**一个字符两字节保存**)。然后返回外层双击两个参数查看值。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId14.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId14.png)

首先看s的数据

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId15.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId15.png)

发现他是 **0x?? 0x14 0x00 0x00** 这种格式去显示，为了显示更清楚，记录左边的地址，去十六进制数去看看

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId16.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId16.png)

会发现他所有的数据都是这样的。刚刚说过unicode是一个字符用两个字节保存，但是这个仅限与window下，**在linux系统中，unicode都是一个字符以四个字节保存**，所以会看到每个数据都是以这种格式存放 **0x?? 0x14 0x00 0x00** ，其中**0x14**是无效字符，我们不用管，直接按照小尾方式读取。那么**s**的数据中第一个字符为**0x3A(:)**，其他字符也是一样。那么我们**s**的数据看完了，我们去看看**byte_8048A90**的数据。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId17.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId17.png)

发现也是一样，都是 **0x?? 0x14 0x00 0x00**格式。知道这两个字符串保存什么之后，我们可以开始写脚本了。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId18.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId18.png)

**第二种：**

这种方法非常实用，也很快速得到flag。我们首先在linux运行edb。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId19.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId19.png)

然后把我们的实验分析对象拖进去，或者也可以通过菜单 **File->open** 打开

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId20.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId20.png)

这个时候我们的程序已经在调试了，我们**返回ida去看看加密运算的函数地址，**

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId21.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId21.png)

红框的位置是函数头，我们在edb中跳到这个地址。按下快捷键**Ctrl+G**，然后输入地址，记住一定要有**前缀0x**。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId22.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId22.png)

点击OK，直接可以跳转到关键代码。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId23.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId23.png)

这几个函数和ida里的一样，所以我们是跳转到了正确地方。接下来按照前面的分析我们直接在这个函数尾部下断，然后观察**eax**的值，就可以拿到**flag**。我们往下拉会看到ret指令，我们选中这点指令按下**F2**下断，也可以单击**左边红色的地址**进行下断。下断成功会看到左边红色地址有一个**小红点。**

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId24.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId24.png)

接下来我们按下F9或者点击工具栏的

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId25.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId25.png)

让程序跑起来。运行了两次后，发现程序断在了我们下断的地方，我们看看eax

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId26.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId26.png)

记录这个值，然后在左下方的**DataDump中按下Ctrl+G**，跳转到这个地址。

![/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId27.png](/posts/CTF/攻防世界-逆向题笔记{no-strings-attached}/document_image_rId27.png)

依旧能看到flag，因为是unicode型的字符串，所以**一个字符4个字节大小(Linux)。**

**总结：**

大概熟悉一下edb操作，如果本身会OD，那么edb操作基本不是问题。另外就是要知道一个东西，Window平台下的unicode是一个字符2个字节，Linux下是一个字符4个字节。