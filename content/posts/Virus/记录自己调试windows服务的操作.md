---
title: "记录自己调试windows服务的操作"
description: "记录自己调试windows服务的操作"
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

# 记录自己调试windows服务的操作

- [PlaneJun](https://bbs.pediy.com/user-home-826671.htm) ・2022-1-9 23:25

如题，近日分析了一个样本，发现需要调试服务，然后自己也没调试过服务，就在国内查了一些资料，基本能用的也就看雪一个大哥发的教程。[[原创]使用Windbg&OllyDbg从头调试windows服务-软件逆向-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-229643.htm)

但是跟着操作弄了一遍，虽然是弄好了，但是会出现一个情况就是调试器会非常的卡，而且过了一会就消失了，整个虚拟机就直接是卡死状态，研究了许久还是解决不了就放弃了。然后就自己办法弄，考虑过写驱动给拦截了，但是VS2022没有WDK，于是想到全局API Hook，然后发现svchost.exe不过消息队列。在觉得无望的时候，我在火绒剑发现了这么一个玩意。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId6.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId6.png)

因为自己是没去研究过服务启动的一个流程，在分析样本的时候用的是sc命令启动，就以为服务是从sc.exe去加载的，然后去分析了半天，发现不行。于是乎发现了这么一个流程，服务的一个启动过程如下：

service.exe -----[CreateProcessW]------->svchost.exe-------[LoadLibrary]-------->服务启动

然后就有了一个想法，直接附加service.exe然后给CreateProcessW下断点，再启动服务，拦截到svchost.exe的创建后，把参数修改为PROCESS_SUSPEND_RESUME，最后再用调试器附加svchost.exe下断LoadLibrary,手动恢复线程后等待目标服务DLL加载。理论存在开始实践。

## 法一：

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId7.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId7.png)

启动服务后直接断下，然后直接查看参数，发现是包含了线程暂停。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId8.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId8.png)

猜到后面会存在恢复线程操作，就稍微跟了一下。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId9.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId9.png)

F8走几步发现了恢复线程的函数，这里直接Nop掉不让他执行。走完之后直接恢复这个地方的Call

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId10.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId10.png)

这个时候已经可以把调试器F9跑起来了，然后会突然断在异常上。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId11.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId11.png)

这里的这个异常让我想到了另一种调试方法，在法二里。

这个时候另一个调试器附加创建出来的进程。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId12.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId12.png)

恢复线程，然后F9让调试器跑起来。这个时候这里是不会断下的，需要到第一个调试器F9，一直F9到第二个调试器断下。（**这里猜测是因为services.exe的操作没跑完，然后svchost.exe单独跑就出现问题。）**

第二个调试器断下后，就可以直接单次F9等到目标DLL加载了。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId13.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId13.png)

加载完毕，直接跳到目标DLL的函数下断即可。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId14.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId14.png)

接下来配合IDA食用即可。

## 法二：

这个方法是我在调试的时候发现的一个特征，不知道适不适用，就是直接暴力断网，因为恶意样本以服务形式启动，肯定不是为了好玩，其中肯定会有网络行为。那么断网会导致样本在通讯的时候会产生一个等待行为，那么这个等待行为就可以拿来用。举个例子，样本存在下面的代码。

```c
a = URLDownloadToFileW()
if(a)
{
......
}
else
{
return
}
```

那么，在服务调用URLDownloadToFileW这个函数时，由于你网络断开，那么他就会有一个网络等待行为，通常这个等待行为不会很长也不会很久，在这个等待时间里，直接用火绒剑查看服务所存在的进程PID，然后调试器附加后到if(a)这个位置下断，然后把网连上即可，函数返回断下后就可以开始分析了。

## 注意事项：

1）、服务加载的超时时间，因为调试原因，服务加载的时候超过了规定时间，就会直接结束调试，所以需要设置时间。设置方法，这里直接截图开头那位大哥帖子里的内容。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId15.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId15.png)

2）、调试需要提前开启好两个，因为一但调试进行，电脑就会卡死，无法打开东西。（貌似可以通过cmd命令打开）

3）、关闭系统异常时自动关机的选项，不然会突然直接提示一分钟后自动关机。（计算机右键-属性-高级系统设置）

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId16.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId16.png)

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId17.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId17.png)

虽然我这两个方法确实有点乱七八槽，但确实是可以调试服务的。如果各位大哥还有啥好法子，也可以支支招。

![/posts/Virus/记录自己调试windows服务的操作/document_image_rId18.png](/posts/Virus/记录自己调试windows服务的操作/document_image_rId18.png)