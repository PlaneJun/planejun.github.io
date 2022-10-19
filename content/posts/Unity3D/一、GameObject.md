---
title: "一、GameObject"
description: "一、GameObject"
tags: ["Unity3D"]
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

# 一、GameObject

# **1**、吹⽜⼤王

Unity 3D简称u3d，是由 Unity Technologies 公司开发的⼀个让玩家轻松创建诸如三维视频游戏、建筑可视化、实时三维动画等类型互动内容的**多平台的综合型游戏开发⼯具**。类似的开发⼯具还有UnrealEngine(ue4)、Source(起源)、寒霜 等等。

有很多由Unity 引擎提供⽀持的游戏。国内例如原批、永劫⽆间、枪⽕重⽣、王者荣耀、⽣死狙击等；国外例如Rust、逃离塔克夫、ECO、糖⾖⼈、动物派对等。

# **2**、开发环境

由于u3d主打的开发环境是C#(其实还有js、B啥的语⾔)，加上C#的环境机制，导致u3d的游戏逆向难度降低，在⾮加密的情况下，游戏极易容易遭到破坏。

![/posts/Unity3D/一、GameObject/image1.jpeg](/posts/Unity3D/一、GameObject/image1.jpeg)

在游戏未对⾃⾝运⾏环境的检查下，甚⾄可以⾃写插件的形式让游戏加载(类似win32下的劫持注⼊和远线程注⼊)。除了编程语⾔上的选择，u3d的开发的游戏存在两种环境：Mono和il2cpp。参考资料:[https://zhuanlan.zhihu.com/p/352463394](https://zhuanlan.zhihu.com/p/352463394)

- Mono

Mono是⼀个由Xamarin公司所主持的⾃由开放源码项⽬。⽬标是在尽可能多的平台上使.net标准的东⻄能正常运⾏的⼀套⼯具，核⼼在于“跨平台的让.net代码能运⾏起来。

C#的执⾏与通常的WIN32程序不同，C#在编译初期会被翻译成⼀种被称为IL的中间语⾔，然后将IL执⾏在.net的虚拟机中。⽽Mono的作⽤就是将IL编译成对应平台的原⽣码，实现跨平台。但mono编译IL的过程较为复杂切繁琐，因此在⼀些⼤型游戏上移植后的运⾏效率并不是那么出⾊。因此IL2CPP出⽣了!!!

![/posts/Unity3D/一、GameObject/image2.jpeg](/posts/Unity3D/一、GameObject/image2.jpeg)

- IL2CPP

IL2CPP (Intermediate Language To C++)是新开发的脚本后端。简单来说就是把IL代码转换为 C++代码,并且C++的⾼移植性和⾼效率的优点也⼀并继承过来。⼤脚本被编译成IL，在游戏运⾏的时候，IL和项⽬⾥其他第三⽅兼容的DLL⼀起，放⼊Mono VM虚拟机，由虚拟机解析成机器码，并且执⾏IL2CPP做的改变由下图红⾊部分标明：

![/posts/Unity3D/一、GameObject/image3.jpeg](/posts/Unity3D/一、GameObject/image3.jpeg)

Mono和IL2CPP的编译⽅法如下：

![/posts/Unity3D/一、GameObject/image4.jpeg](/posts/Unity3D/一、GameObject/image4.jpeg)

![/posts/Unity3D/一、GameObject/image5.jpeg](/posts/Unity3D/一、GameObject/image5.jpeg)

![/posts/Unity3D/一、GameObject/image6.jpeg](/posts/Unity3D/一、GameObject/image6.jpeg)

选择想要编译的平台后Build即可。

---

# **3**、引擎结构

![这个图我不知道烂⼤街了没，反正有⼀些地⽅是不对的，但是我懒得改了，凑合看吧。](/posts/Unity3D/一、GameObject/Untitled.png)

这个图我不知道烂⼤街了没，反正有⼀些地⽅是不对的，但是我懒得改了，凑合看吧。

## 3.1、**GOM**

u3d的每⼀个元素都是⼀个对象(**GameObject**)，并且有⼀个全局的对象管理器管理(**Game Object Manager**，简称**GOM**)，GOM为双链表结构。

```cpp
class GameObjectManager
{
public:
	BaseObject* m_lastTaggedObject; //0x0000 
	BaseObject* m_taggedObjects; //0x0008 
	BaseObject* m_lastActiveObject; //0x0010 
	BaseObject* m_activeObjects; //0x0018
};

class BaseObject
{
public:
	BaseObject* m_lastObject; //0x0000 
	BaseObject* m_nextObjects; //0x0008 
	GameObject* m_objectEntity; //0x0010
};
```

在GOM中你可以找到在游戏世界中存在的对象。你可以使⽤C++代码进⾏遍历。

```cpp
//通过tag名字来查找对象
ULONG64 GameData::GetObjectFromList(ULONG64 listPtr, ULONG64 lastObjectPtr, const char* objectName)
{
	BaseObject activeObject = ReadBase(listPtr); 
	ULONG64 classNamePtr = 0;
	char buf[40];
	if (activeObject.Object != 0)
	{
		int i = 0; 
		while (1)
		{
			i++;
			if (i >= 1000)
				break;
			classNamePtr = ReadMem<ULONG64>(activeObject.Object + 0x60); 
			ReadVirtual((LPVOID)classNamePtr, &buf, sizeof(buf));
			if (strcmp(buf, objectName) == 0) 
				break;
			activeObject = ReadBase(activeObject.nextObjectLink);
		}
		classNamePtr = ReadMem<ULONG64>(activeObject.Object + 0x60); 
		ReadVirtual((LPVOID)classNamePtr, &buf, sizeof(buf));
		if (strcmp(buf, objectName) == 0) 
			return activeObject.Object;
	}
}
```

但我不建议你那么操作，因为游戏通常会存在⼤量对象，遍历起来会⾮常慢也⾮常低效。

![/posts/Unity3D/一、GameObject/image8.jpeg](/posts/Unity3D/一、GameObject/image8.jpeg)

GOM的查找⽅法，可以根据u3d的固定字符串 `Tag: tag name is null or empty.` .

```cpp
。text:00007FFF81DC9FF0 											sub_7FFF81DC9FF0 proc near 	; CODE XREF: sub_7FFF819C1D80+144↑p
.text:00007FFF81DC9FF0 																										; sub_7FFF819C2560+170↑p
.text:00007FFF81DC9FF0 																										; DATA XREF: ...
.text:00007FFF81DC9FF0
.text:00007FFF81DC9FF0 											var_18 	= byte ptr -18h
.text:00007FFF81DC9FF0 											arg_0 	= qword ptr 8
.text:00007FFF81DC9FF0
.text:00007FFF81DC9FF0 48 89 5C 24 08 							mov [rsp+arg_0], rbx
.text:00007FFF81DC9FF5 57 													  push rdi
.text:00007FFF81DC9FF6 48 83 EC 30 									sub rsp, 30h
.text:00007FFF81DC9FFA 80 79 20 01 									cmp byte ptr [rcx+20h], 1
.text:00007FFF81DC9FFE 48 8B FA 										mov rdi, rdx
.text:00007FFF81DCA001 48 8B D9 										mov rbx, rcx
.text:00007FFF81DCA004 75 0F 												jnz short loc_7FFF81DCA015
.text:00007FFF81DCA006 48 0F BE 41 18 							movsx rax, byte ptr [rcx+18h]
.text:00007FFF81DCA00B B9 18 00 00 00 							mov ecx, 18h
.text:00007FFF81DCA010 48 2B C8 										sub rcx, rax
.text:00007FFF81DCA013 EB 04 												jmp short loc_7FFF81DCA019
.text:00007FFF81DCA015 ; ---------------------------------------------------------------------------
.text:00007FFF81DCA015
.text:00007FFF81DCA015 												loc_7FFF81DCA015: 				; CODE XREF: sub_7FFF81DC9FF0+14↑j
.text:00007FFF81DCA015 48 8B 49 10 									mov rcx, [rcx+10h]
.text:00007FFF81DCA019
.text:00007FFF81DCA019 												loc_7FFF81DCA019: 				; CODE XREF: sub_7FFF81DC9FF0+23↑j
.text:00007FFF81DCA019 48 85 C9 										test rcx, rcx
.text:00007FFF81DCA01C 75 13 												jnz short loc_7FFF81DCA031
.text:00007FFF81DCA01E 48 8D 15 3B 51 27 01 				lea rdx, aTagTagNameIsNu ; "Tag: tag name is null or empty."
.text:00007FFF81DCA025 48 8D 4C 24 20 							lea rcx, [rsp+38h+var_18]
.text:00007FFF81DCA02A E8 21 20 1A 00 							call sub_7FFF81F6C050
.text:00007FFF81DCA02F EB 32 												jmp short loc_7FFF81DCA063
.text:00007FFF81DCA031 ; ---------------------------------------------------------------------------
.text:00007FFF81DCA031
.text:00007FFF81DCA031 												loc_7FFF81DCA031: 			; CODE XREF: sub_7FFF81DC9FF0+2C↑j
.text:00007FFF81DCA031 E8 4A 7E DF FF 							call sub_7FFF81BC1E80
.text:00007FFF81DCA036 48 8B D3 										mov rdx, rbx
.text:00007FFF81DCA039 48 8B C8 										mov rcx, rax
.text:00007FFF81DCA03C E8 6F AA DF FF 							call sub_7FFF81BC4AB0
.text:00007FFF81DCA041 83 F8 FF 										cmp eax, 0FFFFFFFFh
.text:00007FFF81DCA044 75 3B 												jnz short loc_7FFF81DCA081
.text:00007FFF81DCA046 80 7B 20 01 									cmp byte ptr [rbx+20h], 1
.text:00007FFF81DCA04A 74 03 												jz short loc_7FFF81DCA04F
.text:00007FFF81DCA04C 48 8B 1B 										mov rbx, [rbx]
.text:00007FFF81DCA04F
.text:00007FFF81DCA04F 												loc_7FFF81DCA04F: 			; CODE XREF: sub_7FFF81DC9FF0+5A↑j
.text:00007FFF81DCA04F 4C 8B C3 										mov r8, rbx
.text:00007FFF81DCA052 48 8D 15 27 51 27 01 				lea rdx, aTagSIsNotDefin ; "Tag: %s is not defined."
.text:00007FFF81DCA059 48 8D 4C 24 20 							lea rcx, [rsp+38h+var_18]
.text:00007FFF81DCA05E E8 9D 25 1A 00 							call sub_7FFF81F6C600
.text:00007FFF81DCA063
.text:00007FFF81DCA063 												loc_7FFF81DCA063: 			; CODE XREF: sub_7FFF81DC9FF0+3F↑j
.text:00007FFF81DCA063 4C 8B 00 										mov r8, [rax]
.text:00007FFF81DCA066 48 8B D7 										mov rdx, rdi
.text:00007FFF81DCA069 33 C9 												xor ecx, ecx
.text:00007FFF81DCA06B 48 8B D8 										mov rbx, rax
.text:00007FFF81DCA06E FF 15 24 09 65 01 						call cs:qword_7FFF8341A998
.text:00007FFF81DCA074 48 8B 43 08 									mov rax, [rbx+8]
.text:00007FFF81DCA078 48 89 47 08 									mov [rdi+8], rax
.text:00007FFF81DCA07C B8 FF FF FF FF 							mov eax, 0FFFFFFFFh
.text:00007FFF81DCA081
.text:00007FFF81DCA081 												loc_7FFF81DCA081: 			; CODE XREF: sub_7FFF81DC9FF0+54↑j
.text:00007FFF81DCA081 48 8B 5C 24 40 							mov rbx, [rsp+38h+arg_0]
.text:00007FFF81DCA086 48 83 C4 30 									add rsp, 30h
.text:00007FFF81DCA08A 5F 													  pop rdi
.text:00007FFF81DCA08B C3 													  retn
```

对该函数头交叉引⽤。

![/posts/Unity3D/一、GameObject/image9.png](/posts/Unity3D/一、GameObject/image9.png)

进⼊下⼀个函数。

![/posts/Unity3D/一、GameObject/image10.png](/posts/Unity3D/一、GameObject/image10.png)

其中⼀个是直接返回GOM。

![/posts/Unity3D/一、GameObject/image11.png](/posts/Unity3D/一、GameObject/image11.png)

另⼀个是在头部返回GOM。

![/posts/Unity3D/一、GameObject/image12.png](/posts/Unity3D/一、GameObject/image12.png)

## 3.2、**GameObject**

u3d所有的元素都是⼀个GameObject对象，并且以绑定脚本的形式为对象赋能。

![/posts/Unity3D/一、GameObject/image13.jpeg](/posts/Unity3D/一、GameObject/image13.jpeg)

GameObject的结构并不复杂，其内存成员的属性和内容与u3d编辑器⾯板所表⽰的基本⼀致，但仍需要了解⼀些固定的结构。

```cpp
class GameObject
{
public:
	void* m_vtable;//0x0000
	Class<T>* m_instance;
	Array<Component>* m_components; //0x0030
	uint32_t m_componentCount; //0x0040
	uint16_t m_tag; //0x0054
	char* m_name; //0x0060
};
```

上⾯列出了在GameObject中较为重要的结构成员。

- **vtable**为对象的虚表，记录着操作的函数。
- m_instance是⼀个⾮常**重要**的成员，主要⽤于组件，此处先略过，下⼀⼩节会说。
- m_components记录着所有绑定在该对象上的组件实例。需要注意的是，每个对象的第⼀个组件**默认**绑定⼀个名为**Transform**的组件**(+0x0008)**⽤来表⽰该对象的位移数据。其次才是开发者⾃添加的脚本。

![/posts/Unity3D/一、GameObject/image14.jpeg](/posts/Unity3D/一、GameObject/image14.jpeg)

![/posts/Unity3D/一、GameObject/image15.png](/posts/Unity3D/一、GameObject/image15.png)

- m_componentCount表⽰该对象被绑定的组件数。
    
    ![/posts/Unity3D/一、GameObject/image16.jpeg](/posts/Unity3D/一、GameObject/image16.jpeg)
    
- m_tag为⼀个常数，也等同于Group ID。可以通过设置这个tag来标识该对象为哪个组，tag可以为⾃定义值。⽐如u3d的Camera相机通常为常数5。

![/posts/Unity3D/一、GameObject/image17.jpeg](/posts/Unity3D/一、GameObject/image17.jpeg)

- m_name表⽰对象的名字，也就是tag上⽅的那⼀栏。

![/posts/Unity3D/一、GameObject/image18.jpeg](/posts/Unity3D/一、GameObject/image18.jpeg)

GameObject成员属性这⾥也就介绍”完了”，使⽤双引号是因为这⾥列举出的GameObject并⾮完整的结构，只是列举的这些 成成员相对来说⽐较重要。在这些成员之后其实还有⼀些属性，⽐如isActivity、isStatic等。这⾥⼤家可以⾃⾏去查看， 也算是⼀个作业。

![/posts/Unity3D/一、GameObject/image19.jpeg](/posts/Unity3D/一、GameObject/image19.jpeg)

## 3.3、组件

⼀个存粹的对象是毫⽆意义的，因此u3d以组件的形式绑定在对象上，使得对象百花⻬放。这⾥随便打开⼀个绑定在对象上的组件。

![/posts/Unity3D/一、GameObject/image20.jpeg](/posts/Unity3D/一、GameObject/image20.jpeg)

![/posts/Unity3D/一、GameObject/image21.jpeg](/posts/Unity3D/一、GameObject/image21.jpeg)

可以看到只要是组件都会继承⼀个叫做`MonoBehaviour`的基类。这个⾮常重要，因为会涉及到之后编写mono插件。然后对⽐⼀下组件⾯板的成员和打开的源码。

![/posts/Unity3D/一、GameObject/image22.jpeg](/posts/Unity3D/一、GameObject/image22.jpeg)

可以看到类中使⽤了public修饰词的变量都会出现在⾯板上，这就意味着这些变量修改后会存在⼀些效果，上⼀⼩节，我们有⼀个属性m_instance没讲解是因为这个成员只有当对象为组件时才存在意义。

我们拿到`Enemy_HoverBot`的对象地址`0x25C002CE990`,然后跳转到+0x30的位置查看⼀下内容。

![/posts/Unity3D/一、GameObject/image23.png](/posts/Unity3D/一、GameObject/image23.png)

由于0x0008为Transform组件，这⾥我们⽤第⼆个组件(0x0018)来讲解。展开后得到

![/posts/Unity3D/一、GameObject/image24.png](/posts/Unity3D/一、GameObject/image24.png)

先来看⼀下m_instance的泛型类型Class。

```cpp
template<typename T>
class Class
{
public:
	T* m_runtime; //0x0000
	GameObject* m_cachePtr; //0x0010 上⼀层指针
};
```

其中只关注的第⼀个成员，也就是m_runtime,它的类型为泛型T,即开发者⾃定义的类。通⽤结构如下：

```cpp
class T
{
public:
#ifdef __MONO__
	Info* m_info; //0x0000
#else
	Info m_info; //0x0010
#endif
//这⾥开始为类的数据
};

struct Info
{
#ifdef __MONO__
	char* className; //0x0048
	char* namespace; //0x0050
#else
	char* className; //0x0000
	char* namespace; //0x0008
#endif
}
```

可以看到模板类中既可以通过Info结构获取到类的**名字**和所属的**命名空间**，也可以获取到类的**成员数据**。下⾯是Mono的效果。**(这是个⾮常重要的东⻄对于U3D的逆向,拿到⼀个对象，可以知道他下边绑定了什么组件，组件叫啥)**

![/posts/Unity3D/一、GameObject/image25.jpeg](/posts/Unity3D/一、GameObject/image25.jpeg)

![Untitled](/posts/Unity3D/一、GameObject/Untitled%201.png)

u3d⼏乎所有的组件都遵循这样的规律，但Transform组件除外。Transform的数据在0x38的位置，该处被称为AccessReadOnly。

![/posts/Unity3D/一、GameObject/image27.png](/posts/Unity3D/一、GameObject/image27.png)

<aside>
💡 ⽣活⼩妙招1：U3D的矩阵，直接暴⼒遍历GOM，判断tag是否为相机，然后进⼊相机的Transform]+0x0038]下看
⼀下就能发现矩阵哦!

</aside>

![Untitled](/posts/Unity3D/一、GameObject/Untitled%202.png)

<aside>
💡 ⽣活⼩妙招2：万物皆对象，包括游戏中的⾓⾊，⼀般游戏都会默认绑定⼀个Transform，通过这个就可以获取对象
坐标哦!

</aside>

## 3.4、通⽤回调

u3d为每个组件设置了默认的回调函数，如下：

- void Start() is called is the GameObject is enabled
- void Awake() is called prior to Start()
- void Update() is called every frame, can be skipped for several frames to keep FPS up
- void FixedUpdate() is called every frame, will not be skipped
- void LateUpdate() is called after all other Update-methods
- void OnEnable()/OnDiable() is called when the GameObject is enabled/disabled
- void OnDestroy() is called when the GameObject is destroyed (via GameObject.Destroy)
- void OnGUI() is called when drawing and allows the script to use the GUI-API (described
later on)

⼝头描述可能不太好理解，这⾥直接举个实际例⼦。

现在我要做⼀个⽜逼的盖，我对我的盖呢，我对我的盖有下列要求：

- 创建⼀个线程⽤来实时获取数据。
- 创建⼀个线程⽤来实时处理数据。
- 创建⼀个线程⽤来绘制。

然后碰巧，这个B游戏咧，他不给你创建线程，他会检测你的线程。当然你可以过掉，但是还有⼀个⽅法，那就是hook/替换⼀
个持久化存在的对象的回调函数。hook你可以vmt也可以inline，这样把这个对象变成你⾃⼰的对象，那是不是就更⾼级⼀点
了捏。

Start()→我⽤来初始化我的盖数据
Update()→我⽤来获取数据
LateUpdate()→我⽤来处理获取到的数据
OnGUI()→我拿来绘制
就是这个意思。⾄于怎么⽤，以及怎么去获取这些函数，后⾯再说。