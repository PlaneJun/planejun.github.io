---
title: "天堂M傳說-吃药call的逆向分析"
description: "天堂M傳說-吃药call的逆向分析"
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

# 天堂M傳說-吃药call的逆向分析

• [PlaneJun](https://bbs.pediy.com/user-home-826671.htm) ・2019-10-7 13:10

**目录**

一、基础理论知识

二、实践是检验真理的唯一标准

1、恢复钩子

2、发包的灵活处理以及分析

3、调用

三、总结

分析对象：**天堂M 傳說**

编程工具：**Visual Studio 2019**

工具：**CheatEngine7.0** （该游戏有调试器检测，OD附加后电脑直接进入卡死状态）、**火绒剑**

# 一、基础理论知识

既然是吃药call，那么我们需要先思考一下这个call的一些信息，比如说他的参数个数，触发时机等等，这样的话才能方便我们去逆向找出这个call。

首先我们明确两点：1、这个游戏的是一个网络游戏，那么就说明他和服务器存在着数据的交互，用来保证数据的正确性**(封包的检测)**，举个例子：我们要做某个动作，那么此时客户端**(就是我们正在玩的游戏)**就会向服务器发送一个封包数据，然后服务器对这个封包进行一个验证，该动作是否合法或者可以操作，然后服务器再发送一个结果封包到客户端，客户端接收到封包后进行处理。如果合法，那么进行这个操作，如果不合法就阻止或者非法提示**(封号)**。2、我们既然是要找吃药call，那么肯定是在我们吃药的时候才会触发的游戏内部call，因为游戏不可能说在我们做其他动作的时候游戏会调用的吃药call(该情况可以存在，但是没有必要)。明确了上面两点后，我们呢就有了一些找call的思路。

**思路一**：通过发包。因为在我们吃药的时候，客户端肯定会发送一个封包到服务器，用来验证这个动作是否可行，然后服务器反馈到客户端。我们就可以直接在相关发包的函数头下一个断点，然后吃药，断下的时候，我们就可以通过不断的返回上一层来定位到吃药call。

**思路二**：通过物品的属性去逆向查找。因为药品是背包的物品，那么我们吃药这个动作也相当于一个使用背包物品的动作，此时，游戏需要知道我们使用了背包的哪个物品。然后再根据这个物品去调用他的相关函数。所以说，在我们找吃药call的时候需要注意一些的参数**(也就是push的值)**,因为他极大的可能会存着药品的某个属性**(比如：地址、ID、数量)**。因此我们就可以通过这个特性直接在药品的相关属性下一个访问断点，通过调试器给我们断下的地方去定位到一些关键代码段，然后再进行逆向反推，这样的方法也是可以找到吃药call的。

笔者在本篇文章中采用的是思路一，原因是思路二看起来虽然道理很简单，但是真要去逆向反推的话其过程是相当繁琐和困难的，因为游戏公司不可能给你轻轻松松的找到call，他们肯定会进行一些代码处理和加密，让逆向的难度大大增加。其困难的程度由该call的火爆性决定**(如果一个call被众多外挂所利用，那么游戏公司会对其进行一些处理，例如:加密，检测)**。话不多说，咱们开始搞事！

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled.png)

# 二、实践是检验真理的唯一标准

咱们既然是通过发包下手，那么简单来介绍一下常见的发包函数: send,sendto,WSASend，这些呢基本都是现在比较常见的发包函数，游戏也基本上也是通过这三个发包函数中的其中一个去与服务器交互的。Emmm，既然我们知道，那么游戏公司也知道，所以咱们相对于某些游戏来说，直接在这些函数头部下断是会吃瘪的，需要去瞎搞更深底层的WSPSend。因为本篇文章所采用的游戏对其的处理并不怎么高手，这里呢我就不过多讨论。

# 1、恢复钩子

这里我用的是火绒剑**(官方认可工具,强大又安全)**。咱们打开游戏后先来看一下这游戏干了什么事。打开火绒剑扫描钩子。

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%201.png)

通过扫描结果我们会发现这个游戏下了很多的钩子，其中有一个就是发包函数send的钩子，我们双击看一下。

![QQ图片20221016013535.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/QQ%25E5%259B%25BE%25E7%2589%258720221016013535.png)

发现他的头变成了jmp，而且目的地址为游戏地址。为了确保调试正常，我们待会把他下的所有钩子全部恢复。**这里注意：钩子的恢复必须要在游戏内，如果恢复钩子过早那么游戏会提示一些错误。简单说就是你得在游戏界面为以下时才能恢复。**

![1648093052946-09bc4e8c-4939-49bf-a7da-f263ded07320.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/1648093052946-09bc4e8c-4939-49bf-a7da-f263ded07320.png)

# 2、发包的灵活处理以及分析

这个游戏用的是send发包，因为三大发包函数里只有这个下断后会断下。我们用CE附加游戏后到send头部尝试下断后做动作看看。

![aaaaa.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/aaaaa.png)

我们会发现，一旦我们下断点后，切回游戏的一瞬间，断点就被断下，无论我们重复多少次都是如此。而我们当前想要是，只有在游戏角色做某出些动作时才断下。这个就与我们想做的事产生了一些矛盾，那怎么办呢？

![1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422.png)

我们先思考一下为什么在我们切回游戏的时候，断点会断下？断点一旦被断下就说明了有封包发送到服务器，可是客户端(游戏)为什么在我们切回游戏的时候要发送封包呢？其实在切回游戏后，游戏会瞬间发送一个封包到服务器是有一定作用的，这个作用类似于游戏拉回操作。用FPS来说的话就是防止瞬移，有些FPS能实现网截瞬移就是因为游戏并不会对服务器发送一个校验的封包**(有的直接通过第三方工具拦截封包实现瞬移)**，如果说游戏不断的向服务器发送一个请求正确位置的封包，那么服务器接收到后就会返回一个正确位置的封包，然后游戏会进行一个判断，如果当前位置不正确就直接拉回。大致原理就是这个意思。明白这个之后我们来看一下怎么去解决这个问题。稍微学过socket编程的都知道，send函数他有一个参数是包长，我们贴上MSDN的定义：

![1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422%201.png)

在游戏中，不同的动作他的包长也会发生变化的，因为动作不同，包数据也不同。**(当然不排除一些包长一样的动作)**，那么我们就可以通过这个包长去下一个条件断点，只有当包长是吃药时的包长才让断点断下。那么问题来了，我们怎么知道他这个包长是多少，上面也说了，我们一下断点切回游戏，游戏就会发送一个类似校验的封包，使游戏断下，这样的话我们也没法观察吃药时的封包包长。但是只要思想不滑坡，办法总比困难多。咱们直接来一个HOOK获取包长就行了。我们在send头部下断让其断下，然后返回到外层。

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%202.png)

**(为什么不在send头部下断点，因为CE好像不支持[esp+xx] == xx 这样类型的断点，反正我没下成功)**，通过send的定义，包长是第三个参数，那么对应的就是eax是我们的包长，我们可以直接在push eax 下面一句进行一个hook。我们首先分配一个空白地址用来保存我们的eax，我申请的地址为0x03830000,我们直接用CE自带的代码注入HOOK，

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%203.png)

HOOK后，咱们把刚刚申请到的地址放到列表看看。

![1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422%202.png)

我这里以16进制显示，我们发现现在我们可以看到包长了。

![1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422%203.png)

那么咱们去吃药，发现吃药的时候包长也是0xE，那好办，下一个条件断点。

![1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422%204.png)

咱们回去吃药，注意：确保你是吃药后才断下的。因为我说了，不排除其他动作包长也是0xE。

吃药断下后，我们记录一下跟踪到的CALL。

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%204.png)

红框是我们要看的，因为红框以外的call，通过观察参数，基本上就不是了。

咱们从上往下看。

**第一个call：GameClient80.bin+672b0c**

![1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422.png](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/1648093081482-4f9c5bcd-5d04-4459-ac22-48e4acd97422%205.png)

看到他是call 寄存器的形式，基本上也不用看。

**第二个call：GameClient80.bin+66CABB**

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%205.png)

**第三个 call：GameClient80.bin+ 66CF3A**

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%206.png)

**第四个 call：GameClient80.bin+ 603F1D**

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%207.png)

**第五个 call：GameClient80.bin+ 603AB5**

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%208.png)

**第六个 call：GameClient80.bin+ 617B76**

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%209.png)

到这里呢我们基本上是获取到有关吃药的call了，上面的六个call里也是包含了真实的吃药call，但是呢，我这里就不继续分析了，直接摊牌。

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%2010.png)

我用的是第五个call。其他的call大家可以去尝试，除了第一个call,因为第一个call进去后直接是send发包，就意味着第一个call只是负责给服务器发送一个封包来检测这个动作是否合法，他直接跳过了动作的封装。

# 3、调用

调用非常的简单，因为我用的是第五个call，我也分析过了，我就直接贴上结果。

```cpp
push 2DCFA3B8              //血瓶地址
mov ecx,[0142e368]           //背包基地址
call 00A03C60
```

这个call，如果说从push和call内部的ret来看他只有一个参数，但是其实他有两个参数，其中一个是push，另一个是通过给ecx，然后call内部进行调用。 调用这个call只要传参正确就直接实现了吃药，具体效果我就不发了。这里我简单的说一下他这个背包地址和血瓶地址怎么来。

首先是背包基地址，直接通过血瓶的数量反推就可以直接得到。反推的过程可以算出一个表达式：

**背包物品=0142E368]+0x54]+i*4]**

然后去查找一些偏移，发现+0x20偏移存的是数量   +0x8的位置存的是物品ID。每个物品的ID是唯一的，我们就可以去记录这个药品的ID,然后写一个程序去遍历物品在判断该物品的ID是否为血瓶，这样就可以得到了血瓶的地址，然后就可以去调用了。这里我简单贴一下我的C++代码：

```cpp
void __declspec(naked) Call_Blood()
         
        {
         
                 _asm
         
                 {
         
                          pushad
         
                          push g_dwBloodAddr   //对象地址
         
                          mov ecx, [Base_Pack]
         
                          mov eax, 0x00A03C60
         
                          call eax
         
                          popad
         
                          ret
         
                 }
         
        }
         
        void TraversingObject()
         
        {
         
                 DWORD actorArray = 0, count = 0, id = 0;
         
                 for (int i = 0; i < 180; i++)
         
                 {
         
                          actorArray = *(DWORD*)Base_Pack + 0x54;
         
                          actorArray = *(DWORD*)actorArray + i * 0x4;
         
                          actorArray = *(DWORD*)actorArray;
         
                          /*读取物品数量和ID*/
         
                          ReadProcessMemory(GetCurrentProcess(), (LPVOID)(actorArray + 0x20), &count, sizeof(DWORD), NULL);  
         
                          ReadProcessMemory(GetCurrentProcess(), (LPVOID)(actorArray + 0x8), &id, sizeof(DWORD), NULL);
         
                          if (0x00213D7C == id)  //血
         
                                  g_dwBloodAddr = actorArray;
         
                 }
         
        }
         
        void CDEBUG::OnBnClickedButton2()
         
        {
         
                 // TODO: 在此添加控件通知处理程序代码
         
                 if (NULL == g_dwBloodAddr)
         
                          TraversingObject();
         
                 Call_Blood();
         
        }
```

我这里是编写为带有MFC窗口的DLL，注入游戏后会弹出一个窗口，然后点击一下Button2就可以实现吃药了。

# 三、总结

在分析这个游戏时，深刻认识到了什么叫做吃瘪，笔者刚刚开始是通过血瓶数量、ID、地址去逆向反推血瓶call，然后开始遇到各种奇葩，因为如果你用CE去访问什么写入了血瓶数量，你会发现，无论什么时候，他传进去的总是使用后的血瓶数量**(当前数量-1)**。就是说他血瓶数量减少的代码段并不在当前访问的代码段，而是在其他地方。然后笔者就继续分析数量企图找出被减少的代码段，发现代码有加密运算，经过一顿骚操作后，我直接尝试nop大法，发现。。。。nop后还能吃药，就说明我找错了。。

![Untitled](/posts/GameHacker/天堂M傳說-吃药call的逆向分析/Untitled%2011.png)

然后尝试其他方法，反正就各种吃瘪，最后还找到了播放粒子特效的call。。。然后被迫用send发包，简单的处理之后直接找到吃药call。虽然说找call的整个过程稍微有点心累，但是总的来说收获还是挺多的，因为在找背包数组的时候还是稍微吃点逻辑，另外就是send发包的一个灵活处理。