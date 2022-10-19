---
title: "攻防世界-逆向题笔记{Hello,"
description: "攻防世界-逆向题笔记{Hello,"
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
分析对象：Hello,CTF

工具：ida、exeinfope

步骤：1）、运行查看效果

2）、PE工具查壳

3）、查看文件平台

4）、如果有壳就脱壳，无壳进行第四步

5）、文件拖入ida进行分析

6）、解密

**一、运行查看效果**

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId4.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId4.png)

可以发现输入错误的注册码后，又从新开始要求输入，可知他是一个循环。

**二、PE工具查壳**

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId5.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId5.png)

无壳。

**三、查看文件平台**

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId6.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId6.png)

依旧为**32位程序**，直接32位ida打开

**四、无**

**五、文件拖入ida进行分析**

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId7.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId7.png)

依旧先**搜索字符串**，看看有没有**可用内容**或者**明文flag**。

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId8.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId8.png)

发现这里有可用信息，**success! ,**双击后交叉引用到此处。

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId9.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId9.png)

拉到头，发现该函数是**main**函数。

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId10.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId10.png)

并发现该处有可疑之处，猜测是加密密钥。

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId11.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId11.png)

简单进行分析后，发现输入的内容长度不能>17，但是在最后比较的时候却和v13的比较。通过手算v13长度可只为34，而17的两倍就是为34。猜测程序将输入的内容进行了一个类似翻倍的操作。那么看看他是怎么翻倍操作。

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId12.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId12.png)

很明显，他把输入内容的字符转换为十六进制后保存。放一张ASCLL码表

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId13.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId13.png)

转换过程非常明显。比如输入的内容为ABC，那么经过程序转换就为 0x41 0x42 0x43,存入变量中就是414243。

**六、解密**

明白了转换过程，那么我们就可以解密了。

可以写脚本，也可以手动转换，这里就直接脚本转换

将密钥两辆分解得到 43 72 61 63 6b 4d 65 4a 75 73 74 46 6f 72 46 75 6e

因为这些都是十六进制的Ascll码，直接对应表转换为字符即可。

![%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId14.png](%E6%94%BB%E9%98%B2%E4%B8%96%E7%95%8C-%E9%80%86%E5%90%91%E9%A2%98%E7%AC%94%E8%AE%B0%7BHello,%20CTF%7D%2014ff0db5f95c45b48734ba56c22d3c72/document_image_rId14.png)

**总结：**

这题太简单，没什么好总结 ，就是这么膨胀！           =。=