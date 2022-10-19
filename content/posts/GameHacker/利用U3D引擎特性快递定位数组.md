---
title: "利用U3D引擎特性快递定位数组"
description: "利用U3D引擎特性快递定位数组"
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

# 利用U3D引擎特性快递定位数组

- [PlaneJun](https://bbs.pediy.com/user-home-826671.htm) ・2021-8-18 15:11

作为当代不争气的青年代表，在放弃挣扎前奋笔写下最后一篇文章，之后不问江湖事。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId5.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId5.png)

IL2CPP我就不介绍了，显得水文字。搞过U3D的男人都懂，IL2CPP模式打包的游戏，游戏下面会存在一个名为**global-metadata.dat**的文件，这个东西解包之后会得到一个dump.cs和Mono模式下的所有DLL。那么综合网上大部分的教程来说，他们只会用dump.cs来看偏移，虽然直接用dnspy打开dll看也是可以的，但是dump.cs更偏向于sdk文件，可以直接把它的内容当作头文件来用，更加方便。接下来我将给大家带来一套组合拳，教大家如何正确的使用这个玩意来快速找到一个游戏的数组。这里插个题外话，标题虽然这次讲解的是针对IL2CPP,但是相对来说，Mono也是可以的，只不过在偏移和一些结构上面略显差异。

老早时期，精力旺盛，在一次乱翻内存之中发现了u3d的一个惊天大秘密，然后经过一系列的操作和研究后就总结出了一套自己独有的使用方式。是这样的，u3d里的对象，假设对象地址是0x1000000000,那么在这个对象的某一个地方是会存着这个对象的类名，并且寻址的偏移还是固定的，之前查了一会，这玩意好像叫做反射。那么，这个有什么用捏？我这里先模拟一下传统的找数组方法：**首先CE搜索人物的某项数据，比如坐标，然后一层一层往上跟到本人地址，然后通过访问本人地址或者扫指针，最后找出数组。**这些看起来都是非常常规的方法，但是如果放到u3d游戏上来说是非常恶心的。众所周知，u3d有用的偏移大部分都是10层起步，像这种偏移量，先不说你有没有那个耐心用CE跟出来，就算你扫指针，那也是吃不消的，况且你还要去做数据对比。所以说整个流程下来，即使你找到数组，那也是累得半死的。这个时候，类名就派上用场了。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId6.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId6.png)

我们知道凡是U3D打包的游戏，源代码基本都在**Assembly-CSharp.dll**这个DLL里，那么我们可不可以大胆猜测一下，**游戏有一个主管理器类，里面存放着很多数据，其中就包括玩家数组，模型数组之类的，然后我们通过用DnSpy这个工具来找到这个类在游戏的实际名字，最后通过搜索类名的方法，反推到对象名**。上面我提到过了，对象到类名的偏移是固定的，那么找到类的实例对象就相当于做小学加减法。存在既合理，下面开干。这里用的是**枪火重生**这个游戏，我们用il2cppDumper把GM文件解包之后用DnSpy打开DLL。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId7.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId7.png)

既然是找有玩家数组的类，我们第一步就是去找玩家类，玩家类可以通过玩家的属性去找，常见的是血量，对应的关键词大部分是**HP,Health**，这个游戏用的是HP。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId8.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId8.png)

**左边表示成员，右边表示所在的类**。放眼看去，只有**PlayerProp**符合我们的要求，我们双击后，左边的列表将会定位到该类，我们点击展开按钮查看内部的内容。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId9.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId9.png)

发现里面都是一些属性，从类名也可以判断出这个是存放玩家属性的类而不是玩家类，并且应该是属于玩家类的一个成员，所以我们只需要查找那个类引用了**PlayerProp**这个类。对这个类右键点分析，在右下角点开被使用。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId10.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId10.png)

根据类名可以发现有两个可能是玩家类，经过两者内容对比，最终确认**NewPlayerObject**为玩家类。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId11.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId11.png)

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId12.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId12.png)

既然已经找到了玩家类，那么数找组就很容易了。同样的操作，对玩家类右键点分析，然后点击被使用，通过根据类名进行了一系列筛选后，判定**NewPlayerManager**为**主管理类，而且为静态类。**

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId13.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId13.png)

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId14.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId14.png)

并且在该类下面也发现了很多数组。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId15.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId15.png)

到这里，我们基本工作就已经完成了。到时候我们直接到游戏找到这个类然后通过DnSpy显示的偏移或Dump.cs显示的偏移拿到对应数组即可。

上面我提到过，U3D的对象都会在自身下的某的地方存着该对象的类型，而且该偏移是固定的。那么这个偏移怎么找？我也不知道。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId16.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId16.png)

因为是当时偶然间在CE内存翻到的，然后去对比了其他游戏，发现偏移是固定的。目前只知道这个玩意叫做**runtimeType.**

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId17.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId17.png)

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId18.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId18.png)

由于万物继承object，所以直接在头部翻就能找到了。

```cpp
Il2cpp：
        obj+0x10]=className
        obj+0x18]=spaceName
  Mono：
        obj+0x0]+0x48]=className
        obj+0x0]+0x50]=spaceName
```

这里说一个细节，在il2cpp中，className的内存属性是只读，Mono中是可读可写，然后就是className的后面紧接的字符串是该类的成员函数名和成员变量名，可以通过这个细节来筛选其他数据。

进入游戏**(这里建议一定要进入游戏，因为进入游戏后才会有对象，有了对象才能方便我们查看数组对象)**，打开CE搜索我们找到的类NewPlayerManager。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId19.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId19.png)

这里只有一个结果，那么通过查看内存数据再对比DnSpy,发现该地址是正确的，然后开始做加减法。将该地址放到CE搜索**（记得恢复查找的内存属性）。**

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId20.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId20.png)

发现这里有两个，这个时候不要慌，都拿下来减0x10.

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId21.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId21.png)

第二个为0，u3d的对象头部不可能为0，这是经验之谈。这个时候0x 2E99B5E6060，实际上已经是NewPlayerManager的实例了，现在可以通过跟地址或者扫指针去找基地址，但是因为这个类是静态类所以这里直接搜索一下看看有没有基地址存着这个地址。有一个，那么这个就可以用了。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId22.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId22.png)

数据都找好了，现在就来看看数组吧，我们打开dump.cs，然后讲这个地址添加到ce的数据结构工具。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId23.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId23.png)

此时会存在一个问题，dump中显示0x10是一个字典，但是当先的0x10却是类名，那这就很尴尬。。这里在说一个点，也算是经验之谈。当前我们这个地址**0x 2E99B5E6060，**只能算是一个假体，真正实体在**0xB8**的位置，至于为什么，我也不懂，分析多了，见多了就默认了。我们进入**0xB8**的位置.

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId24.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId24.png)

发现现在数据相似度已经很高了。我们直接去看看怪物的数组**（0x48）**

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId25.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId25.png)

**进去后0x10是数组指针，0x18为数量，这是List固定的，然后进入0x10。**

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId26.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId26.png)

同样数组是从0x20开始的，也是固定的。这些其实不是说我在瞎说，因为U3D的List他内存结构就是这样的。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId27.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId27.png)

当前敌人也只有一个，然后0x20的指针，类名也是NewPlayerObject。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId28.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId28.png)

然后剩下的，就是查看dump.cs去拿到对应偏移即可。

### 补充一下

在NewPlayerManager类中由于该类直接是继承object，所以该类的0x10直接是类名。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId29.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId29.png)

但是像NewPlayerObject类基类并不是object，所以需要在0x0的地方读一层之后0x10的位置才是类名。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId30.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId30.png)

因为是文字叙述，无法将所有细节给描述出来。所以大部分东西直接一笔带过，至于为什么，多去实践就知道了。这次针对的是il2cpp，所以在其他也是il2cpp的游戏中可能操作起来的相似度较高，Mono上可能有一些差异，但并不碍事，我只是想说有那么个方法可以快速拿到数据而已，至于能不能做到看自己的造化。文章中难免出现先一些错误，各位就留点情面吧。

这大概也是我最后一次写这种类型的技术文了。。留个纪念吧。

![/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId31.png](/posts/GameHacker/利用U3D引擎特性快递定位数组/document_image_rId31.png)