---
title: "某Pubg Lite辅助逆向分析过程及心得"
description: "某Pubg Lite辅助逆向分析过程及心得"
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
- [PlaneJun](https://bbs.pediy.com/user-home-826671.htm) ・2020-1-25 21:36

新的一年，先祝看雪各位成员新年快乐！另外也要多注意身体和卫生，今年与往年大大不同，这次瘟疫大爆发且气象异常搞得人心不安，不要在这样一个欢喜的节日里把自己交代了。在次也希望武汉能挺过这次大劫，武汉加油！

# 正文：

昨天得到一款PubgLite辅助，然后看到一些好玩的功能就尝试着去分析一下，看看能不能为我所用。分析的第一步，当然是先运行查看效果。由于该辅助会被游戏检测，所以只放一张菜单图。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled.png)

从左上方的菜单我们可以看到有很多功能，首先透视功能基本就是通用公式算出来的，本文主要注重分析其他功能的一个实现。**Tip:该游戏位64位游戏，在启动辅助的时候，会加载一个用于读写的驱动，但是火绒报毒说该驱动存在后门，至于真实性不做考察，大家如果想下载来运行一定要注意。**

首先PE工具查壳，显示为无壳且32位程序**(本次只采用ida原地手撕，至于为什么不用动态调试有两点：1、该辅助实在太lj，开了直接检测，然后秒封号，根本没法调试。2、有后门)。**

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%201.png)

知道大概信息后，我们直接打开32位的ida，扔进去加载。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%202.png)

老规矩，直接搜索字符串，因为该程序没有加壳，所以搜索到的东西都是明文。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%203.png)

此时我们可以看到字符串中有VMP保护壳的相关字符串，既然前面查壳的时候是无壳，那么这个保护壳，我估计是给驱动文件加的，并不影响我们的分析。我们看看其他内容。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%204.png)

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%205.png)

可以看到一些辅助相关的内容。第一张图片可以看看到游戏进程名和类名，还有一些类似特征码和地址、偏移的东西；第二张图就可以看到辅助菜单的内容。那么我们如何通过这些字符串去分析呢？在这里，我给大家普及一下目前市场上辅助大致的绘制流程。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%206.png)

Emmm,有点丑。。。。这张流程图中间的三个模块是在一个死循环里面的。

**1）、第一个模块是用来获取游戏数据的，用来对后面功能作为一个初始化。**

**2）、第二个模块该项功能是整个辅助的核心部分，里面存在多种算法和数据处理，比如透视常用的矩阵算法或者三角函数算法、自瞄用的角度算法、雷达透视用的朝向算法等等，但是这些算法大多都是已经另外封装，这里面只是对其调用。**

**3）、第三个模块辅助菜单功能模块，就是用来显示和开关用的，里面通常会通过读取控制某个功能开关的变量来判断是否要打开功能。**

上面三个模块执行完后就开始一个新的循环。

以上就是目前市场上辅助的大致绘制流程。我们简单了解之后，思考一下怎么去分析，怎么去下手。在上面字符串中我们已经搜索到了菜单的字符串，那么我们就可以直接通过这个字符串来作为突破口。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%207.png)

我们通过ida的交叉引用和F5功能来到了这里，我们大概看了一下，这里面都是菜单内容，而且最让人注意的是它每个功能都会有一个变量来判断开和关(和我上面所说的流程一致)。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%208.png)

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%209.png)

这里的标签是我在分析的是注释的，我们现在只需要注意这个地方。我们把这个段代码拉到顶部，也就是函数头的位置，按下**X**，看看什么地方有调用这个函数。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2010.png)

发现只有一个地方在调用，那么这个地方只可能在绘制流程内。
我们点击确定进行跳转。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2011.png)

我们可以看到我们来到了一坨函数的调用。那么这个地方，就和我说的流程一样了，只不过该辅助呢分的模块比较多。这里我已经分析完了，所以标签和注释什么的都写的一清二楚。接下来将带领大家来分析，看看如何得到这些东西。

我们既然来到了这绘制流程，那么所有的核心代码肯定都在这个部分，我们可以一个一个函数去看，当然，这个只属于有从事这方面开发和有经验的人员，不然还是很难看出来的。那么就代表我们小白脸没法去分析了吗？当然是No。还记得上面菜单模块里面的那些个**if**吗？

先贴出绘制菜单模块的代码：

```cpp
void DrawMenu()
{
  int v0; // eax
  LPVOID v1; // eax
  char *v2; // eax
  LPVOID v3; // eax
  LPVOID v4; // ST0C_4
  int v5; // ST08_4
  int v6; // eax
  LPVOID v7; // eax
  char *v8; // eax
  LPVOID v9; // eax
  LPVOID v10; // ST0C_4
  int v11; // ST08_4
  int v12; // eax
  LPVOID v13; // eax
  char *v14; // eax
  LPVOID v15; // eax
  LPVOID v16; // ST0C_4
  int v17; // ST08_4
  int v18; // eax
  LPVOID v19; // eax
  char *v20; // eax
  LPVOID v21; // eax
  LPVOID v22; // ST0C_4
  int v23; // ST08_4
  int v24; // eax
  LPVOID v25; // eax
  char *v26; // eax
  LPVOID v27; // eax
  LPVOID v28; // ST0C_4
  int v29; // ST08_4
  int v30; // eax
  LPVOID v31; // eax
  char *v32; // eax
  LPVOID v33; // eax
  LPVOID v34; // ST0C_4
  int v35; // ST08_4
  int v36; // eax
  LPVOID v37; // eax
  char *v38; // eax
  int v39; // eax
  int v40; // ST0C_4
  int v41; // ST08_4
  int v42; // eax
  LPVOID v43; // eax
  char *v44; // eax
  LPVOID v45; // eax
  LPVOID v46; // ST0C_4
  int v47; // ST08_4
  int v48; // eax
  LPVOID v49; // eax
  char *v50; // eax
  LPVOID v51; // eax
  LPVOID v52; // ST0C_4
  int v53; // ST08_4
  int v54; // eax
  LPVOID v55; // eax
  char *v56; // eax
  LPVOID v57; // eax
  LPVOID v58; // ST0C_4
  int v59; // ST08_4
  int v60; // eax
  int v61; // [esp+4h] [ebp-34h]
  int v62; // [esp+Ch] [ebp-2Ch]
  int v63; // [esp+Ch] [ebp-2Ch]
  int v64; // [esp+Ch] [ebp-2Ch]
  int v65; // [esp+Ch] [ebp-2Ch]
  int v66; // [esp+Ch] [ebp-2Ch]
  int v67; // [esp+Ch] [ebp-2Ch]
  int v68; // [esp+Ch] [ebp-2Ch]
  int v69; // [esp+Ch] [ebp-2Ch]
  int v70; // [esp+Ch] [ebp-2Ch]
  int v71; // [esp+Ch] [ebp-2Ch]
  LPVOID v72[2]; // [esp+14h] [ebp-24h]
  double v73; // [esp+1Ch] [ebp-1Ch]
  LPVOID lpMem[2]; // [esp+24h] [ebp-14h]
  LPVOID StrSwitch; // [esp+2Ch] [ebp-Ch]
  int v76; // [esp+30h] [ebp-8h]
  int v77; // [esp+34h] [ebp-4h]
 
  v77 = 10;
  v76 = 20;
  StrSwitch = 0;
  v0 = sub_4065DD(255, 0, 0);
  lpMem[0] = "    T s";
  DrawText(lpMem, 10, 20, v0);
  if ( lpMem[0] )
    FreeMem(lpMem[0]);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_Box )
    v1 = lpMem[1];
  else
    v1 = (LPVOID)HIDWORD(v73);
  v72[1] = "显示方框F1";
  DrawText(&v72[1], v77, v76, v1);              // 这里是直接把文字显示
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_Box )                             // 判断控制该功能的变量是否为真
    v2 = "开";
  else
    v2 = "关";
  StrSwitch = v2;
  *(double *)lpMem = (double)v77;
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v62 = sub_4065DD(255, 0, 0);
  if ( Switch_Box )
    v3 = v72[0];
  else
    v3 = (LPVOID)v62;
  v4 = v3;
  v5 = v76;
  v6 = toInt64(v73);
  DrawText(&StrSwitch, v6, v5, v4);             // 把结果显示出去
  if ( StrSwitch )
    FreeMem(StrSwitch);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_Hp )
    v7 = lpMem[1];
  else
    v7 = (LPVOID)HIDWORD(v73);
  v72[1] = "显示血量F2";
  DrawText(&v72[1], v77, v76, v7);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_Hp )
    v8 = "开";
  else
    v8 = "关";
  StrSwitch = v8;
  *(double *)lpMem = (double)v77;
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v63 = sub_4065DD(255, 0, 0);
  if ( Switch_Hp )
    v9 = v72[0];
  else
    v9 = (LPVOID)v63;
  v10 = v9;
  v11 = v76;
  v12 = toInt64(v73);
  DrawText(&StrSwitch, v12, v11, v10);
  if ( StrSwitch )
    FreeMem(StrSwitch);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_Line )
    v13 = lpMem[1];
  else
    v13 = (LPVOID)HIDWORD(v73);
  v72[1] = "显示射线F3";
  DrawText(&v72[1], v77, v76, v13);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_Line )
    v14 = "开";
  else
    v14 = "关";
  StrSwitch = v14;
  *(double *)lpMem = (double)v77;
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v64 = sub_4065DD(255, 0, 0);
  if ( Switch_Line )
    v15 = v72[0];
  else
    v15 = (LPVOID)v64;
  v16 = v15;
  v17 = v76;
  v18 = toInt64(v73);
  DrawText(&StrSwitch, v18, v17, v16);
  if ( StrSwitch )
    FreeMem(StrSwitch);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_Aim )
    v19 = lpMem[1];
  else
    v19 = (LPVOID)HIDWORD(v73);
  v72[1] = "右键自瞄F4";
  DrawText(&v72[1], v77, v76, v19);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_Aim )
    v20 = "开";
  else
    v20 = "关";
  StrSwitch = v20;
  *(double *)lpMem = (double)v77;
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v65 = sub_4065DD(255, 0, 0);
  if ( Switch_Aim )
    v21 = v72[0];
  else
    v21 = (LPVOID)v65;
  v22 = v21;
  v23 = v76;
  v24 = toInt64(v73);
  DrawText(&StrSwitch, v24, v23, v22);
  if ( StrSwitch )
    FreeMem(StrSwitch);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_Breath )
    v25 = lpMem[1];
  else
    v25 = (LPVOID)HIDWORD(v73);
  v72[1] = "开启屏息F5";
  DrawText(&v72[1], v77, v76, v25);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_Breath )
    v26 = "开";
  else
    v26 = "关";
  StrSwitch = v26;
  *(double *)lpMem = (double)v77;               // v78=10.0f
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v66 = sub_4065DD(255, 0, 0);
  if ( Switch_Breath )
    v27 = v72[0];
  else
    v27 = (LPVOID)v66;
  v28 = v27;
  v29 = v76;
  v30 = toInt64(v73);
  DrawText(&StrSwitch, v30, v29, v28);
  if ( StrSwitch )
    FreeMem(StrSwitch);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_HightJump )
    v31 = lpMem[1];
  else
    v31 = (LPVOID)HIDWORD(v73);
  v72[1] = "人物高跳F6";
  DrawText(&v72[1], v77, v76, v31);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_HightJump )
    v32 = "开";
  else
    v32 = "关";
  StrSwitch = v32;
  *(double *)lpMem = (double)v77;
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v67 = sub_4065DD(255, 0, 0);
  if ( Switch_HightJump )
    v33 = v72[0];
  else
    v33 = (LPVOID)v67;
  v34 = v33;
  v35 = v76;
  v36 = toInt64(v73);
  DrawText(&StrSwitch, v36, v35, v34);
  if ( StrSwitch )
    FreeMem(StrSwitch);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_CarFly )
    v37 = lpMem[1];
  else
    v37 = (LPVOID)HIDWORD(v73);
  v72[1] = "车辆飞行F7";
  DrawText(&v72[1], v77, v76, v37);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_CarFly )
    v38 = "开";
  else
    v38 = "关";
  StrSwitch = v38;
  lpMem[1] = (LPVOID)sub_40D2AF(1, dword_4DF3E8);
  lpMem[0] = (LPVOID)sub_40204B((char)StrSwitch);
  if ( StrSwitch )
    FreeMem(StrSwitch);
  if ( lpMem[1] )
    FreeMem(lpMem[1]);
  v73 = (double)v77;
  *(double *)v72 = v73 + 75.0;
  v68 = sub_4065DD(255, 0, 0);
  v61 = sub_4065DD(255, 0, 0);
  if ( Switch_CarFly )
    v39 = v68;
  else
    v39 = v61;
  v40 = v39;
  v41 = v76;
  v42 = toInt64(*(double *)v72);
  DrawText(lpMem, v42, v41, v40);
  if ( lpMem[0] )
    FreeMem(lpMem[0]);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_SuperSpeed )
    v43 = lpMem[1];
  else
    v43 = (LPVOID)HIDWORD(v73);
  v72[1] = "人物加速F8";
  DrawText(&v72[1], v77, v76, v43);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_SuperSpeed )
    v44 = "开";
  else
    v44 = "关";
  StrSwitch = v44;
  *(double *)lpMem = (double)v77;
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v69 = sub_4065DD(255, 0, 0);
  if ( Switch_SuperSpeed )
    v45 = v72[0];
  else
    v45 = (LPVOID)v69;
  v46 = v45;
  v47 = v76;
  v48 = toInt64(v73);
  DrawText(&StrSwitch, v48, v47, v46);
  if ( StrSwitch )
    FreeMem(StrSwitch);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_NoRecoil )
    v49 = lpMem[1];
  else
    v49 = (LPVOID)HIDWORD(v73);
  v72[1] = "腰射午后F9";
  DrawText(&v72[1], v77, v76, v49);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_NoRecoil )
    v50 = "开";
  else
    v50 = "关";
  StrSwitch = v50;
  *(double *)lpMem = (double)v77;
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v70 = sub_4065DD(255, 0, 0);
  if ( Switch_NoRecoil )
    v51 = v72[0];
  else
    v51 = (LPVOID)v70;
  v52 = v51;
  v53 = v76;
  v54 = toInt64(v73);
  DrawText(&StrSwitch, v54, v53, v52);
  if ( StrSwitch )
    FreeMem(StrSwitch);
  v76 += 25;
  StrSwitch = 0;
  lpMem[1] = (LPVOID)sub_4065DD(255, 0, 0);
  lpMem[0] = 0;
  HIDWORD(v73) = sub_4065DD(255, 0, 0);
  if ( Switch_AbsorbObject )
    v55 = lpMem[1];
  else
    v55 = (LPVOID)HIDWORD(v73);
  v72[1] = "超级吸人F10";
  DrawText(&v72[1], v77, v76, v55);
  if ( v72[1] )
    FreeMem(v72[1]);
  if ( Switch_AbsorbObject )
    v56 = "开";
  else
    v56 = "关";
  StrSwitch = v56;
  *(double *)lpMem = (double)v77;
  v73 = *(double *)lpMem + 75.0;
  v72[1] = 0;
  v72[0] = (LPVOID)sub_4065DD(255, 0, 0);
  v71 = sub_4065DD(255, 0, 0);
  if ( Switch_AbsorbObject )
    v57 = v72[0];
  else
    v57 = (LPVOID)v71;
  v58 = v57;
  v59 = v76;
  v60 = toInt64(v73);
  DrawText(&StrSwitch, v60, v59, v58);
  if ( StrSwitch )
    FreeMem(StrSwitch);
}
```

我们可以看到，这部分只有绘制菜单的代码，而且每一个子菜单的代码部分都是基本相同。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2012.png)

这部分的**if**只是用来判断和显示功能是否打开，里面并未存在任何其他代码，说明真正的功能开关在别处，而且在功能开启时菜单的内容也有所改变，所以一定会修改到控制变量。我们直接在控制变量上按下**X**，看看哪里有对他赋值或者判断。**（我这用内存屏息作为例子，因为涉及到透视的功能存在算法会稍微复杂）**

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2013.png)

我们看到该处有多个交叉引用，我们发现后面三个都是在我们这一个函数内，所以不用看。主要就看前三个，第一个时进行判断，第二第三个存在于同一个函数内，而且时相邻代码，可以知道他这里是修改了该控制变量，我们直接跳过去看看。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2014.png)

我们发现这里面都是对所有功能的控制变量进行一个取反操作。那我们就知道这个函数只用于对控制变量的一个赋值，我们在函数头交叉引用看看。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2015.png)

发现还是来到这个地方，那么我们就已经分析出两个函数了。现在我们返回对控制变量的交叉引用，我们这次看cmp的那个语句。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2016.png)

我们直接跳转到了这里，很显然这里基本上就是对功能的一个开关。他这里判断如果控制变量为=1就执行。其实这里如果对易语言熟悉的人，脑海中基本可以浮现出这一段所对应易语言代码。

他首先是对14D8这样一个临时进行一个十六进制到十进制的转换，然后对其临时字符串释放(这个也是我在本次逆向分析的时候学到的)。其实他这一句，可能就对应着易语言的**进制_十六到十()**的这个命令。继续转换之后就开始地址之间的加法，这里也要注意的是，由于程序位32位，游戏为64位，所以指针大小也不同**(32位4字节，64位8字节)**。Ida这里直接用double计算(一个double 8字节)，然后再转为int64也是我这次学到的。最后进行读写操作。这里他是驱动读写，并且在分析的时候发现如果是**+44(十进制)**那么就为**读操作**，如果是**+64(十进制)**就为**写操作**。大概了解之后，这个函数的代码基本就可以阅读完了。都是对地址+偏移后读，然后再内容。这里就顺便看一下他的屏息如何实现的。

```cpp
if ( Switch_Breath == 1 )                     // 开启屏息
  {
    lpMem = "14D8";
    v22 = HexToDec(&lpMem);
    if ( lpMem )
      FreeMem(lpMem);
    v21 = (double)MySelftAddr;
    v20 = (double)v22;
    v19 = v21 + v20;                            // 本人地址+0x14D8
    v0 = toInt64(v19);                          // 转成int64
    LODWORD(LayerOne) = (*(int (__stdcall **)(LPVOID *, int, _DWORD, _DWORD))(*(_DWORD *)dword_4DF300 + 44))(
                          &dword_4DF300,
                          dword_4DF2F4,
                          v0,
                          HIDWORD(v0));         // 读一次
    v26 = LayerOne;
    lpMem = "458";
    v22 = HexToDec(&lpMem);
    if ( lpMem )
      FreeMem(lpMem);
    v21 = (double)v26;
    v20 = (double)v22;
    v19 = v21 + v20;                            // 把读出来的内容再加上0x458,此时得到的这个地址为目标地址
    v26 = toInt64(v19);
    lpMem = &v18;
    v2 = sub_40D2C1(28, 2);                     // 不晓得干嘛
    if ( lpMem != &v18 )
      sub_40D2BB((LPCSTR)6);
    HIDWORD(v21) = v2;
    if ( v2 )
      (*(void (__stdcall **)(LPVOID *, int, _DWORD, _DWORD, _DWORD))(*(_DWORD *)dword_4DF300 + 64))(
        &dword_4DF300,
        dword_4DF2F4,
        v26,
        HIDWORD(v26),
        0);
    lpMem = &v18;                               // 对目标地址写0
    v3 = sub_40D2C1(28, 2);
    if ( lpMem != &v18 )
      sub_40D2BB((LPCSTR)6);
    HIDWORD(v21) = v3;
    if ( !v3 )
      (*(void (__stdcall **)(LPVOID *, int, _DWORD, _DWORD, signed int))(*(_DWORD *)dword_4DF300 + 64))(
        &dword_4DF300,
        dword_4DF2F4,
        v26,
        HIDWORD(v26),
        0x3F000000);                            // 对目标地址写0.5
  }
```

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2017.png)

首先是获取一个对象地址，然后让该地址+0x14d8第一层，再加上0x458得到目标地址，最后对该地址写入0就实现了屏息。其他功能都是一样的，现在我们返回函数头部按下X跳出去看看。
我们发现跳出来了还是这个地方。现在我们又分析完了一个，那么现在怎么去分析其他函数呢？

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2018.png)

还记得这里吗，它这个数据怎么来我们还不知道，但是我们知道他肯定也是绘制循环里的，而且肯定是本人对象**(屏息就是我们自身的呼吸，所以肯定是在本人对象下的)**，因为他要时时更新游戏数据。我们看看交叉引用，找什么地方给他赋值的。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2019.png)

中间的是我们这个功能函数里的，并不是我们要找的。我们就看第一个

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2020.png)

我们来到这，发现他来源另一个值，我们看看另一个值怎么来。
我们发现它来源于V5，而v5又是读一个值得到的，那么再往上看，就会发现这里其实就是再遍历对象数组。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2021.png)

64位一个地址相差刚刚好是0x8。知道这个后，这一段的代码其实已经很清晰，首先去遍历对象数组，然后去判断血量是否>0和<=100，成立的话就是继续判断是否为本人或者敌人，再成立的话，就进行一个本人对象的赋值和读取对象坐标。这里贴分析后注释的代码。

```cpp
while ( 1 )
  {
    HIDWORD(v57) = v1 + 1;
    LODWORD(v57) = v2;
    *v2 = v1 + 1;
    v56 = v3;
    if ( v1 + 1 > v3 )
      break;
    if ( dword_4DF39C >= v69 )
    {
      *(double *)lpMem = (double)v69;           // 遍历
      v4 = toInt64((double)ActorArray + (*(double *)lpMem - 1.0) * 8.0);// actor+（i-1）*8
      LODWORD(v5) = (*(int (__stdcall **)(LPVOID *, int, _DWORD, _DWORD, int, int *, _DWORD))(*(_DWORD *)dword_4DF300
                                                                                            + 44))(
                      &dword_4DF300,
                      dword_4DF2F4,
                      v4,
                      HIDWORD(v4),
                      v56,
                      v2,
                      HIDWORD(v57));            // 读取对象
      Object = v5;
      lpMem[1] = "A90";                         // 0xA90 血量偏移
      lpMem[0] = (LPVOID)HexToDec(&lpMem[1]);
      if ( lpMem[1] )
        FreeMem(lpMem[1]);
      v57 = toInt64((double)Object + (double)(signed int)lpMem[0]);// 得到血量地址
      v56 = dword_4DF2F4;
      v67 = COERCE_FLOAT((*(int (__stdcall **)(LPVOID *))(*(_DWORD *)dword_4DF300 + 0x3C))(&dword_4DF300));// 读取血量
      *(double *)lpMem = v67;
      v6 = v67 > 0.0000001 && v67 - 100.0 <= 0.0000001;
      if ( v6 )                                 // 血量>0 且 血量<=100
      {
        *(double *)lpMem = (double)Object;
        v7 = toInt64(*(double *)lpMem + 792.0); // 0x318
        LODWORD(v8) = (*(int (__stdcall **)(LPVOID *, int, _DWORD, _DWORD, int, _DWORD, _DWORD))(*(_DWORD *)dword_4DF300
                                                                                               + 44))(
                        &dword_4DF300,
                        dword_4DF2F4,
                        v7,
                        HIDWORD(v7),
                        v56,
                        v57,
                        HIDWORD(v57));
        v9 = toInt64((double)v8 + 32.0);        // 0x20
        LODWORD(v10) = (*(int (__stdcall **)(LPVOID *, int, _DWORD, _DWORD))(*(_DWORD *)dword_4DF300 + 44))(
                         &dword_4DF300,
                         dword_4DF2F4,
                         v9,
                         HIDWORD(v9));
        v60 = (double)v10 + 448.0;              // 0x1C0
        v66 = toInt64(v60);
        lpMem[1] = "5E8";                       // 5E8
        lpMem[0] = (LPVOID)HexToDec(&lpMem[1]);
        if ( lpMem[1] )
          FreeMem(lpMem[1]);
        v57 = toInt64((double)v66 + (double)(signed int)lpMem[0]);
        v56 = dword_4DF2F4;
        v65 = (*(int (__stdcall **)(LPVOID *))(*(_DWORD *)dword_4DF300 + 0x1C))(&dword_4DF300);// 这里通过CE查看发现读取的是v64+0x5E8位置
        if ( v65 == 1 )                         // 判断是否为本人
        {
          MySelftAddr = Object;
          qword_4DF3B0 = v66;                   // 读取坐标需要的地址
          lpMem[1] = dword_4DF3B8;              // 开始读坐标
          v11 = (*(int (__stdcall **)(LPVOID *, int, _DWORD, _DWORD))(*(_DWORD *)dword_4DF300 + 60))(
                  &dword_4DF300,
                  dword_4DF2F4,
                  v66,
                  HIDWORD(v66));   //X
          *(_DWORD *)lpMem[1] = v11;
          lpMem[1] = (char *)dword_4DF3B8 + 4;
          v12 = toInt64((double)qword_4DF3B0 + 4.0);
          v13 = (*(int (__stdcall **)(LPVOID *, int, _DWORD, _DWORD))(*(_DWORD *)dword_4DF300 + 60))(
                  &dword_4DF300,
                  dword_4DF2F4,
                  v12,
                  HIDWORD(v12));  //Y
          *(_DWORD *)lpMem[1] = v13;
          lpMem[1] = (char *)dword_4DF3B8 + 8;
          v14 = toInt64((double)qword_4DF3B0 + 8.0);
          v15 = (*(int (__stdcall **)(LPVOID *, int, _DWORD, _DWORD))(*(_DWORD *)dword_4DF300 + 60))(
                  &dword_4DF300,
                  dword_4DF2F4,
                  v14,
                  HIDWORD(v14));   //Z
          *(_DWORD *)lpMem[1] = v15;
        }
       
    }
    v3 = v56;
    v1 = HIDWORD(v57);
    v2 = (int *)v57;
  }
  v59 = v64;
  v44 = *(_DWORD *)v64;
  v45 = (LPVOID *)((char *)v64 + 4);
  v46 = *(_DWORD *)v64 == 0;
  if ( *(_DWORD *)v64 )
  {
    for ( j = (int)*v45; ; j *= (_DWORD)*v45 )
    {
      ++v45;
      if ( !--v44 )
        break;
    }
    v44 = j;
    v46 = j == 0;
  }
  if ( !v46 )
  {
    do
    {
      v58 = v44;
      if ( *v45 )
        FreeMem(*v45);
      ++v45;
      v44 = v58 - 1;
    }
    while ( v58 != 1 );
  }
  FreeMem(v59);
}
```

这里只分析了本人，敌人的也一样而且敌人还额外去读取了一些值估计是其他的属性，这里就不列出来分析。现在按照同样的操作，函数头交叉引用返回出去。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2022.png)

依旧是熟悉的地方。那么接下来我们就来分析一下，它的对象数组怎么来的。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2023.png)

这个位置交叉引用。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2024.png)

直接看第一个。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2025.png)

返现这里来到它一个赋值的地方。它的值来源于**V4**，我们看看**V4**怎么来的。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2026.png)

这一段基本看完，大家可以下头部返回出去，还是可以到达那个绘制循环的。这里我就不跳了。这里直接分析**Count_Addr**的来源，因为对象数组他是来源于这个地址**-8**的。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2027.png)

同样直接看第一个。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2028.png)

看到**count_addr**被赋值后下面就开始特征码搜索，具体干嘛用后面会说。我们先分析**count_addr**的来源

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2029.png)

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2030.png)

其实没有最后一句没有**+0x8**的情况下，**Count_Addr**存的就是对象指针。你只分析到这里就以为完事了，那不可能。其实这里有一个猫腻，这个辅助很机灵，如果你只运行游戏然后去看**XINPUT1_3.dll+0x500**这个地方，你会发现这个地方一直为**0(下图)**。

他这一段的骚操作。其实就是读取**XINPUT1_3.dll**的句柄后加上**0x400**再加上**0x100**（**我也不知道他为什么不直接+0x500**）。然后读取这里面的值。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2031.png)

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2032.png)

但是如果你打开了辅助，你会发现，他这里的值被改变了。

所以说他这里的值并不是本身就存在的，而是辅助给予赋值的。那么它里面的值怎么来的，咱们看下面。

这里只截图了部分代码，但是我们可以看到他这里是在搜索特征码，我们如果再往下分析很难，因为交叉引用显示下面并没有对我们**count_addr**的地址进行写操作。我们先不管它在干嘛，我们去调试，从IDA返汇编代码记录它的地址，然后我们下断F8单步跟，查看在什么地方他的值被改变了。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2033.png)

通过CE看了一下，当我们执行完这个call后**XINPUT1_3.dll+0x500**的位置，里面由0变成了一个地址。说明这个地方，存在着对**XINPUT1_3.dll+0x500**这个地址的写操作，至于它这一段执行后得到什么值，写了什么值我不带大家去分析，因为这部分代码里的特征码，在我去搜索和对比后，发现其实搜索出来的就是一段对象的解密代码(**PUBG lite数据都是加密了的**)。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2034.png)

那么他解了什么密？那就和他函数头的操作有关了

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2035.png)

他这里对游戏模块+偏移，可能就是游戏密文。通过读取游戏密文，然后特征码搜索出解密代码，按照解密代码去解密，就能得到明文地址，最后把明文地址写入**XINPUT1_3.dll+0x500**，这样，如果之后要遍历对象就直接读取这个位置的值，就无需解密了。所以这就是为什么只打开游戏，**XINPUT1_3.dll+0x500**的位置是0，而打开辅助之后就变了。我们在头部返回出去。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2036.png)

发现来到这里了。，然后我们再返回出去

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2037.png)

又来到这里，那么这个函数就可以知道他就是去获取游戏的数据，然后给下面模块进行一个加工处理操作。这里基本上就分析完所有模块了，另外几个函数没去分析，因为他里面都是算法，没必要去分析，算法都是通用，网上一大堆。

**DrawOther：**估计是透视用，因为里面出现了矩阵算法，怎么知道的？看特征

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2038.png)

这里判断视角，就是说算出来的人在不在屏幕上，再的话就继续操作，不在的话不进行操作，本部分由V16变量控制。

**Sub_407549:**自瞄算法，很明显

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2039.png)

# 总结：

这次是我第一次对程序整个流程进行逆向和分析，所以想通过文章的形式记录下来，也可以顺便看看自己哪里写得不足，好让各位对我提出意见。我也通过这次去分析程序认识到了很多东西，其中就是字符串释放的和强制转换类型的一个认识。字符串释放使得不占用空间、32位程序通过double类型巧妙的对8字节地址进行操作，另外就是重新温习了一下浮点数操作的指令，不然就忘记了  =。=

# 心得：

接下来是一些心里话，如果有哪里说错，或者得罪的多多原谅。

到这里基本上就把程序流程分析完了，可能这里会有人质疑，你就说了这么点就分析完了？其实我的确是分析完了，但是你没去分析，你听别人分析和讲解，觉得自己也行。但是如果给你一个陌生的程序，让你去分析他，可能短短几句功能性的代码都能耗费你几小时时间。逆向这个东西从来不是看谁了解的多，听得多，而是看谁做得多。逆向的确很迷人，能够让你撕开用户和底层那一层隔膜，但是他学习过程和付出的代价却很大，逆向是我目前学习到到现在最能吃经验和积累的一门技术。前段看雪的周年大会，钱老师(**钱林松**)上台说过最让我震撼的一句话就是：**你给我一段反汇编代码，随便给，我一看就马上能知道他的C/C++代码是怎么写的**。其实这一句话不假，只要你逆向搞得多，看得多，多多少少也有这种能力，但是说随便给你一段反汇编代码，试问谁能看一下就能直接实现他C/C++代码。这个就是靠积累和经验攒来了。我这篇文章写出来是为了记录我对这个程序的分析过程，写出来大家能不能看懂取决于我的文采行不行，但是在我分析完这个程序后，我反正是对这个程序的整个流程一清二楚，另外我之所以只分析部分是因为其他部分也是一样的，而且我也不可能把整个代码流程全部分析出来，一方面我懒，另一方面啰嗦。最后转一张图片，最近再看雪看到的，也让我很有感触。

![Untitled](/posts/GameHacker/某Pubg%20Lite辅助逆向分析过程及心得/Untitled%2040.png)