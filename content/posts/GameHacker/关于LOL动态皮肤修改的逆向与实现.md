---
title: "关于LOL动态皮肤修改的逆向与实现"
description: "关于LOL动态皮肤修改的逆向与实现"
tags: ["GameHacker"]
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

# 关于LOL动态皮肤修改的逆向与实现

• [PlaneJun](https://bbs.pediy.com/user-home-826671.htm) ・2018-12-3 09:18

第一次在看雪这种地方发这种帖子，心里很动荡不安。因为我本身很菜，所以有哪里不足的请大佬多多包涵并指出。

最近莫名其妙又染上LOL这个游戏了，然后突然想到之前网上的动态换肤辅助，所以就打算下载一个然后搞搞事，最后虽然是弄出来了，但是因为技术比较差。在实现方面会因为和游戏主线程冲突，导致游戏崩掉（原因看文章结尾）。下面我开始上分析。**Ps：本人技术以及表达能力都很差，请大家见谅。(因为在写这篇文章的时候，我是一边分析一边写这篇文章所以游戏可能是已经结束的状态。但是如果有需要调试游戏的话，我会重新打开游戏进行调试，所以会发现有一些图片和数据是不一样的，重要的是分析)**

在开始分析的时候，因为我以前搞过这游戏，所以我知道这个实现动态换皮肤是需要一个皮肤ID的。所以我就直接打开下载好的辅助，来进行找这个ID。

辅助图片如下：

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled.png)

在这里先说一下如何去找一个`英雄皮肤ID`，所谓的英雄皮肤ID指的是某个英雄他的皮肤数从0开始算，比如【暗裔剑魔】有4个皮肤，那么他的皮肤ID就是0(默认皮肤)、1、2、3。如下图：

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%201.png)

于是我在寻找这个ID的时候就是按照这种方法来寻找，因为默认皮肤的ID为0，所以我通过辅助来换到下一个皮肤的时候，我搜索1，这样慢慢的搜索，最后就会剩下一个我们需要的地址。下面我以【剑圣】为例(搜索好了的)。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%202.png)

然后我们右键访问这个地址，然后去自杀。因为在人物复活的时候皮肤CALL会访问这个皮肤ID来重新对皮肤进行初始化，然后我们就可以通过这个来找出皮肤CALL。下图是该地址在人物复活的时候被访问。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%203.png)

现在出现了下面两条：

> 00581B6B - 89 46 40  - mov [esi+0x40],eax        -1
00582CB7 - 8B 5F 40  - mov ebx,[edi+0x40]        -2
> 

我们选择第一条点击详细信息

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%204.png)

我们会发现eax=0，esi=7DE7D2E8。回到游戏我们会发现我们原本换好的皮肤现在变成默认的皮肤了，也就是皮肤ID变成了0。然后还会发现esi+0x40=7DE7D328，而7DE7D328这个地址刚刚好就是我们搜索出来的皮肤ID地址，就是说我们在复活的时候游戏会把0赋值给我们的皮肤ID地址，让我们的皮肤变成默认的皮肤。那么这个地方很可能就是一个皮肤CALL的传参关键。。。。。然后我们看第二条，一样查看详细信息。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%205.png)

发现edi=7DE7D2E8，然后把edi+0x40的值传进ebx，而这个edi+0x40刚好是我们搜索到的这个皮肤ID地址。可是我们来想一下，此时这个时候皮肤ID地址的值已经为0，说明游戏在执行到这个语句之前，皮肤ID的值已经被改变，就是说在执行到这条语句之前皮肤已经被修改过，所以这条语句不太可能是调用皮肤CALL之前的语句，而是在调用皮肤CALL之后的语句。
以上分析后，发现只有第一条。

> 00581B6B - 89 46 40  - mov [esi+0x40],eax
> 

符合我们的要求，我们记录他的地址00581B6B，然后扔到OD里查看。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%206.png)

我们打开OD，Ctrl+G输入我们记录的地址然后回车跳到这一行代码。有意思的地方到了，在反汇编窗口中可以看到在eax传进皮肤ID地址后，立即去执行了一个CALL，这个就引起了我们的注意，因为我上面说过，调用皮肤CALL需要的一个参数就是皮肤ID值。现在我们在这个CALL下一个断点，然后去送自杀，等待复活。**注意：这个CALL需要在即将复活的那一瞬间下断，不然会被其他地方调用的时候断下，这个时候所返回的数据是不正确的。**

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%207.png)

当人物即将重生的时候游戏断了下来，看右边，EAX的值依然为0。这个时候往上看一下EAX的来源：

> mov eax,dword ptr ss:[esp+0x10]
> 

然后在我开始研究的时候，第一想法是去追这个esp的来源然后想直接通过找到的最终地址加上各种偏移来修改皮肤的，然后发现追不到。结果重新思考一下发现是没有必要的去追这个esp的，因为我直接可以在调用这段的时候给eax一个皮肤ID的值或者直接把皮肤ID的值传进皮肤ID地址(esi+0x40)里，然后直接调用下面的那个CALL就ojbk了。于是我就打算尝试直接调用这个CALL看看效果。为了弄懂调用这个CALL需要哪些参数的需要，我们继续分析。
这里先贴上这个部分的反汇编代码：

```cpp
00581B67    8BCE            mov ecx,esi       
00581B69    6A 01           push 0x1
00581B6B    8946 40         mov dword ptr ds:[esi+0x40],eax 
00581B6E    E8 1D110000     call League_o.00582C90
```

首先：把esi传进了ecx，然后压入一个1，之后把eax传进[esi+0x40]，最后就直接去调用CALL。到发现这里没看到esi的值是哪里来的，所以要么esi的传值操作在上面，要么就在上一层。我们往上找看看有没有对esi赋值的语句。哟西,果然有

```cpp
00581B48    8BF1            mov esi,ecx
```

把ecx的值传进esi，然后又发现上面没有对ecx的传值操作，那只能去上一层找。我们在程序头下断点，然后去自杀，等复活。等我们即将复活的时候断点断了下来，我们直接快捷键返回上一层，这个时候会看到有对ecx的传值操作。

```cpp
0061E0B0 - 8D 8F B8320000        - lea ecx,[edi+0x32B8]
```

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%208.png)

我们在这里下一个断然后去自杀等复活，看看断下来的时候edi的值。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%209.png)

edi=5D395010，我们把这个值扔进CE去搜索一下 。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%2010.png)

会发现出现一个绿色的值，我们把它拉下来查看地址。

**032A6DCC ("League of Legends.exe"+2EA6DCC)**

那么**032A6DCC**这个估计就是人物地址了，所以

**ecx=[0x032A6DC]+0x32B8C   // lea ecx,[edi+0x32B8]**

**因为esi=ecx，由此可知皮肤ID的地址=[0x032A6DC]+0x32B8C+0x40**

我们现在找出ecx的值了，我们返回到刚刚的这个位置。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%2011.png)

现在我们回车进入到这个CALL里。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%2012.png)

这个时候会发现在这个CALL里面有一条语句就是上面我们用CE调试的时候访问我们皮肤ID地址的语句。

```cpp
00582CB7 - 8B 5F 40 - mov ebx,[edi+0x40]
```

而这一条语句是在皮肤ID已经传入皮肤ID地址这个操作后才执行的，这个时候我就估计这个是皮肤CALL的内部。刚刚那个外层就是调用这个CALL来实现更换皮肤的，并且EAX是一个皮肤ID。现在我们从程序头开始分析。

```cpp
00582C90    83EC 2C         sub esp,0x2C
00582C93    A1 D0D49E01     mov eax,dword ptr ds:[0x19ED4D0]
00582C98    33C4            xor eax,esp
00582C9A    894424 28       mov dword ptr ss:[esp+0x28],eax
00582C9E    53              push ebx                                 
00582C9F    55              push ebp
00582CA0    56              push esi
00582CA1    57              push edi
```

从开头到这为止都没有问题，然后到

```cpp
00582CA2 8BF9 mov edi,ecx
```

看一下这一句，把ecx的值传给edi，因为从程序头到这一行都没有为ecx传值操作，所以可知ecx是上一层传进来的，而我们已经分析出ecx的来源，所以ecx是一个参数。然后在往下看，发现都在不断的从edi+偏移里读取东西，而edi来源于ecx，那么可以确定之前找到那个绿色的地址就是人物地址，而[人物地址]+0x32B8应该是关于人物模型之类的，然后再加一个偏移就是对应的属性。然后继续往下分析，直到这个CALL结束(一路顺畅)。

```cpp
00582E8F    C2 0400         retn 0x4
```

发现retn 04 ，说明这个call就push了一个，就是之前那个1，到这里大部分都已经分析完了，现在我们来整理一下，这里贴上游戏调用这个CALL的原型：

```cpp
00581B63    8B4424 10     mov eax,dword ptr ss:[esp+0x10]    //从某个地方获取ID
00581B67    8BCE          mov ecx,esi   
00581B69    6A 01         push 0x1
00581B6B    8946 40       mov dword ptr ds:[esi+0x40],eax   //传入皮肤ID
00581B6E    E8 1D110000   call League_o.00582C90   //调用CALL
```

**人物基址=0x032A6DC**

**ecx=esi=[人物基址]+0x32B8**

**皮肤ID的地址=[人物基址]+0x32B8C+0x40**

**调用CALL所需要的参数：**

**ecx      //通过mov把人物基址+0x32B8传入ecx**

**1          //通过push压入**

**皮肤ID  //通过mov传进皮肤ID地址([人物基址]+0x32B8C+0x40=ecx+0x40)**

最后来测试一下，其中这一句我们就不用加入到代码了。

```cpp
00581B63 8B4424 10 mov eax,dword ptr ss:[esp+0x10]
```

我们只直接给皮肤ID地址传进一个ID值。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%2013.png)

在这里我换第二个皮肤，注入远程代码。

![Untitled](/posts/GameHacker/关于LOL动态皮肤修改的逆向与实现/Untitled%2014.png)

调用成功，那么这个CALL应该就是调用皮肤的CALL。

**============================================================================================================**

**崩溃原因：如果你们跟着我这个文章一步一步操作的话，会发现在下断点的时候，我们人物即使在没死或者没有复活的情况下也会断下，为什么？因为游戏的主线程也在调用这一段代码，但是具体干什么用我不懂，所以如果你在调用这段换肤CALL的时候主线程刚好也在调用这段换肤CALL，那么这个时候就会撞车，主线程就会不知所措，游戏就会崩溃。如何解决这个，就你们自己弄了，这里就不讨论了。**

**============================================================================================================**