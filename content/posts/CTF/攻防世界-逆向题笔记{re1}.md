---
title: "攻防世界-逆向题笔记{re1}"
description: "攻防世界-逆向题笔记{re1}"
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
分析对象：re1

工具：ida、exeinfope

步骤：1）、运行查看效果

2）、PE工具查壳

3）、查看文件平台

4）、如果有壳就脱壳，无壳进行第四步

5）、文件拖入ida进行分析

**一、运行查看效果**

下载完成后鼠标双击**运行查看运行效果**。

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId4.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId4.png)

程序会提示我们输入flag，我们随便输入一个试试。

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId5.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId5.png)

发现直接提示flag不太对。效果查看完毕后。我们开始分析。

**二、PE文件查壳**

**首先先进行查壳**

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId6.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId6.png)

很明显，该文件无壳。点击PE按钮

**三、查看文件平台**

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId7.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId7.png)

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId8.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId8.png)

观察Magic的值，**如果位0x10B,即为32位应用程序。如果为0x20B,即为64应用程序。很明显该文件位32位程序，所以我们用32位Ida打开。**

**四、无**

**五、Ida分析**

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId9.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId9.png)

ida成功加载之后，在左边寻找main函数。**小技巧：鼠标随便单击一个函数后，按下Ctrl+F快捷键打开搜索栏，输入main后回车即可找到main函数。**

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId10.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId10.png)

来到main函数后，直接F5，将汇编代码转换为伪代码。

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId11.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId11.png)

我们先Shift+F12进行字符串搜索，看看有没有明文flag。

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId12.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId12.png)

可以看到有相关字符串，但是依旧没有出现类似明文的flag。我们返回到伪代码。

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId13.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId13.png)

我们简单分析之后得到上图的注释内容，很明显，v5就是我们所要找的flag。

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId14.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId14.png)

这里是对v5的初始化，我们双击后面的

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId15.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId15.png)

可以来到这个位置

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId16.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId16.png)

发现他是一串**十六进制(因为结尾有h)**显示，而我们从代码分析，知道他是以**字符串**的方式进行比较，所以该处应该是一组字符串数据。接下来我们尝试把他转换为字符串。

我们先单击该处数据，确保被我们选中，然后按下**A(转换字符串快捷键)**

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId17.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId17.png)

点击**是**

![/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId18.png](/posts/CTF/攻防世界-逆向题笔记{re1}/document_image_rId18.png)

即可得到我们所需要的flag。

**总结：**

能Shift+F12解决的问题，坚决不去调试。