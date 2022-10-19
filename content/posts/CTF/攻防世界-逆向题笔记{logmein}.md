---
title: "攻防世界-逆向题笔记{logmein}"
description: "攻防世界-逆向题笔记{logmein}"
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

工具：Kali-Linux、Ida、Python

步骤：1）、运行查看效果

2）、PE工具查壳

3）、查看文件平台

4）、如果有壳就脱壳，无壳进行第四步

5）、文件拖入ida进行分析

6）、解密

**一、运行查看效果**

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId4.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId4.png)

**二、PE工具查壳**

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId5.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId5.png)

无壳

**三、查看文件平台**

从第二步可以知道该文件为64位，所以我们需要64位的ida进行分析。

**四、无**

**五、Ida分析**

加载完毕后，首先搜索字符串

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId6.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId6.png)

通过字符串，没发现类似明文的flag，但是可以看到输出的信息。

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId7.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId7.png)

我们在左边的函数窗口找到main函数，然后F5查看伪代码。

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId8.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId8.png)

看到第二句代码很可疑，把一串内容复制到v8。和我们之前练的一题很像，初步猜测用于加密的密钥。往下又看到一个long long类型的变量也很可疑(**后缀为LL，long long缩写**)

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId9.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId9.png)

往下继续看，可以看到一个比较长度和一个循环，再往下就为输入是否成功内容。到这里可以肯定，上面的for循环为加密运算，但是不同的是，他并不是把加密完成后的内容和输入内容比较，而是加密一位就比较一位。

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId10.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId10.png)

然后在这里我们发现一个猫腻，可以看到**(char)(*((_BYTE *)&v7 + i % v6) ^ v8[i])**，最后强制转换为了**char**，而且中间的**&v7+ i % v6**很像一个数组寻址，但是我们在上面看到**v7**却是一串数字，那么我们猜测他会不会是一串字符，只是ida并未正确的识别。我们到**v7**赋值的地方然下**R键**看看。

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId11.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId11.png)

神奇的发现，变成了字符串，那么我们猜测是正确的。(**虽然通过ida的快捷键把这里的数字转换为了字符，但是有一个地方是要考虑的，就是字节的排放顺序。大部分的linux都是小尾排放顺序，所以需要把字符串给反过来读，得到harambe，以后遇到类似 变量=‘xxxxxxxxxx’ 的字符串赋值都需要反过来读，实在实在不明白可以通过linux下的edb调试器去查看此处的内存**)

接下来直接根据伪代码写一个python脚本把整个算法跑一边。以下是大概分析：

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId12.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId12.png)

**六、解密**

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId13.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId13.png)

![/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId14.png](/posts/CTF/攻防世界-逆向题笔记{logmein}/document_image_rId14.png)

**总结：**

看到类似数组寻址的表达式(a1 + xx)，但是a1又不是一个数组表示方式，那么可以怀疑a1就是一个数组，只是可能ida未能正确表示。然后就是字节排序问题。小尾：低位放低位，高位放高位。大尾：高位放低位，低位放高位。红字均指的是数据的高低位。