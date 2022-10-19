---
title: "一、了解UE4引擎及数据查找"
description: "一、了解UE4引擎及数据查找"
tags: ["UnrealEngine"]
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

# 一、了解UE4引擎及数据查找

# 一、获取UE4源代码

github搜索`UnrealEngine`，或者打开地址[https://github.com/EpicGames/UnrealEngine/branches/all](https://github.com/EpicGames/UnrealEngine/branches/all)

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId5.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId5.png)

找到一份你想要的源码后，点击左边版本号，进入对应版本的页面。然后点击下载就行了。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId6.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId6.png)

Tip：UE引擎源码不会公开，需要给EpicGames进行账号关联。[https://zhuanlan.zhihu.com/p/107516361](https://zhuanlan.zhihu.com/p/107516361)

# 二、了解UE4引擎名词

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId8.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId8.png)

- **UObject**：引擎对象的最小单位。只有继承了这个类，才会被UE认为是一个对象。
- **Actor**：在UE里其实不是某种具象化的3D世界里的对象，而是世界里的种种元素，用更泛化抽象的概念来看，小到一个个地上的石头，大到整个世界的运行规则，其实更像是一个容器。
- **RootComponent**：对象的根组件，也就是对象的位置信息。
- **ULevel**：UE的场景的划分模式是基于子关卡级来做的，UE4和其它支持大世界的引擎一样支持游戏场景中的物体动态加载和卸载。UE4中动态加载卸载的子关卡叫做流关卡(StreamingLevel ,ULevelStreaming类)，一开始就加载子关卡叫做持久关卡(PersistentLevel)。场景中的具体物件都是放置在关卡或流关卡中。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId9.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId9.png)

- **UWorld**：游戏主场景。在游戏过程中，一般只存在一个UWorld实例，类似于总导演。
- **GameInstance**：顾名思义，游戏实例。会保存着当前的World和其他整个游戏的信息。官方解释是一个正在运行的游戏的高级别的管理对象，在游戏创建时生成，游戏关闭时销毁，一个游戏中可以有多个GameInstance。在游戏中切换关卡，GameInstance不会销毁（切换关卡时可用GameInstance携带信息）。
- **GameMode**：负责制定游戏的规则，也就是应该如何玩游戏，遵守哪些规则。
- **GameState**：游戏状态。记录游戏的数据，比如当前游戏的进度，世界任务的完成状态等，会自动同步到各个客户端。
- **UEgine**：UE4编辑器本身就是一个游戏。Engine分化出两个子类：UEditorEngine和UGameEngine。UGameEngine用来创建出唯一的一个GameWorld，因为也只有一个，所以为了方便起见，就直接保存了GameInstance指针。EditorWorld其实只是用来预览，所以并不拥有OwningGameInstance，而PlayWorld里的OwningGameInstance才是间接保存了GameInstance。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId10.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId10.png)

- **GObject**：记录着UE4所有类对象，比如，玩家类，引擎类，物品类。
- **GName**：由于C++没有反射机制，故UE自己实现了反射。其中所有的类、字段等字符串信息都存在GName中。

参考链接：

[https://blog.csdn.net/boonti/article/details/82909884](https://blog.csdn.net/boonti/article/details/82909884)

[http://www.zeroyx.com/index.php?r=site/art&id=41&title_id=175](http://www.zeroyx.com/index.php?r=site/art&id=41&title_id=175)

[https://blog.csdn.net/qq_33500238/article/details/104555612](https://blog.csdn.net/qq_33500238/article/details/104555612)

[https://blog.csdn.net/you_lan_hai/article/details/71980058](https://blog.csdn.net/you_lan_hai/article/details/71980058)

[https://zhuanlan.zhihu.com/p/24005952/](https://zhuanlan.zhihu.com/p/24005952/)

# 三、通过源代码寻找数据

首先要确定要找数据的类，然后在类中找有没有一些可用的数据。（单例类之类的）

- **GWorld**：`SeamlessTravel FlushLevelStreaming`

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId16.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId16.png)

- **GName**：`ByteProperty`

找到UObject，从FName中跳到GetName

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId17.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId17.png)

然后转到FName，查看是哪个函数出的字符串

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId18.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId18.png)

然后转到ToString

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId19.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId19.png)

很明显使用Entry算出来的，那么直接进GetDisplayNameEntry

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId20.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId20.png)

进入GetNamePool

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId21.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId21.png)

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId22.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId22.png)

查找哪个地方调用了这个函数。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId23.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId23.png)

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId24.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId24.png)

- **GObject**：`UObjectArray`

可以看到有GObjectArray

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId25.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId25.png)

点过去看到代码注释提示说这是一个全局变量。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId26.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId26.png)

然后直接找这个名字在哪里用过。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId27.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId27.png)

搜索整个字符串，到ID，找到FOR头部。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId28.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId28.png)

由于GObjectArray是全局，然后C++编译器直接把GObjectArray的Num成员以地址形式输出。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId29.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId29.png)

查看NumElements偏移然后计算即可。

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId30.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId30.png)

- **GameInstance**：InWorld->GetGameInstance() is null

![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId31.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId31.png)

- **GEngine**：GenericConnectionFailed
    
    ![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId32.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId32.png)
    
    ![/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId33.png](/posts/UnrealEngine/一、了解UE4引擎及数据查找/document_image_rId33.png)