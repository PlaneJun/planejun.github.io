---
title: "二、UE4三件套获取"
description: "二、UE4三件套获取"
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
# 一、UWorld

- **字符串**：`SeamlessTravel FlushLevelStreaming`源代码如下：

```cpp
GWorld = LoadedWorld;    //特征
if (!LoadedWorld->bIsWorldInitialized)
{
    LoadedWorld->InitWorld();
}
bWorldChanged = true;
// Track session change on seamless travel.
NETWORK_PROFILER(GNetworkProfiler.TrackSessionChange(true, LoadedWorld->URL));
checkSlow((LoadedWorld->GetNetMode() == NM_Client) == bIsClient);
if (bCreateNewGameMode)
{
    LoadedWorld->SetGameMode(PendingTravelURL);
}
// if we've already switched to entry before and this is the transition to the new map, re-create the gameinfo
if (bSwitchedToDefaultMap && !bIsClient)
{
    if (FAudioDevice* AudioDevice = LoadedWorld->GetAudioDeviceRaw())
    {
        AudioDevice->SetDefaultBaseSoundMix(LoadedWorld->GetWorldSettings()->DefaultBaseSoundMix);
    }
    // Copy cheat flags if the game info is present
    // @todo UE4 FIXMELH - see if this exists, it should not since it's created in GameMode or it's garbage info
    if (LoadedWorld->NetworkManager != nullptr)
    {
        LoadedWorld->NetworkManager->bHasStandbyCheatTriggered = bHasStandbyCheatTriggered;
    }
}
// Make sure "always loaded" sub-levels are fully loaded
{
    SCOPE_LOG_TIME_IN_SECONDS(TEXT("    SeamlessTravel FlushLevelStreaming "), nullptr)  //特征
    LoadedWorld->FlushLevelStreaming(EFlushLevelStreamingType::Visibility);    
}
```

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId4.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId4.png)

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId5.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId5.png)

- **Count**：

> Uworld -> ULevel -> Count
> 
- **调试符号**：`Gworld`

跳过去直接拿

# 二、GName

- **字符串**：`ByteProperty`

源代码如下：

```cpp
static FNamePool& GetNamePool()
{
    if (bNamePoolInitialized)
    {
        return *(FNamePool*)NamePoolData;
    }
    FNamePool* Singleton = new (NamePoolData) FNamePool;
    bNamePoolInitialized = true;
    return *Singleton;
}
```

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId6.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId6.png)

- **算法逆推**：

```cpp
//
//   Version:4.22
//
string NameStore::GetName(int i)
{
    int Id = (int)(i  / (int)GameInst::ChunkSize);  //0 旧版本中未加密游戏的ChunkSize固定为16384
    int Idtemp = (int)(i % (int)GameInst::ChunkSize);//1
    auto NamePtr = Process::Read<uint64_t>(Names + Id * 8);  //Names已经空读了一层
    auto Name = Process::Read<uint64_t>(NamePtr + 8 * Idtemp);
    char name[0x100] = {0};
    if (Process::ReadMemory(PVOID(Name + 0x10), name, 0x100)) // 0xC需要手动测
    {
        return name;
    }
    return string();
}
```

- UE中，每个ID都会对应一个属于自己的字符串。比如下表

| Id | String |
| --- | --- |
| 0 | None |
| 1 | ByteProperty |
| 2 | IntProperty |
| 3 | BoolProperty |
| 4 | FloatProperty |
| ..... | ..... |

那么，可以利用算法进行反推出Gname。那么假设`ID=1`，则`Id = 1 / 16384 = 0`、`Idtemp = 1 % 16384 = 1`。**ChunkSize固定为0x4000（16384）**。
搜索`ByteProperty`。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId7.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId7.png)

将结果逐个`Ctrl+B`进行查看，直到找得到如下图内容。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId8.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId8.png)

然后将字符串`None`的地址减掉`??`后的地址（**offset = ②地址 - ①地址 = 0xC**）。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId9.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId9.png)

然后将`ByteProperty`地址剪掉`offset`后进行搜索。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId10.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId10.png)

排除掉最后两个，然后将第一个地址减掉`Idtemp`后进行搜索。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId11.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId11.png)

只有一个结果，由于`id=0`，所以在搜索一次，得到GName。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId12.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId12.png)

- **地址遍历**：

```cpp
uint64_t Base = GetModuleHandleA("xxxxx.exe");
int i = 0;
do
  uint64_t gname = Mem::Read<uint64_t>(Base + 0x8 * i);
  if(gname != NULL)
  {
    char* byteProperty = GetName(gname,1);
    if(!strcmp(byteProperty,"ByteProeprty"))
      break;
  }
while(true);
printf("offset_gname=%x\n",i*8);
```

直接从进程.exe开始+8或者+4进行遍历，每一次遍历就将遍历到的地址假设为GName，然后配合算法计算1返回的字符串，若返回的字符串为ByteProperty，则表明本次遍历到的地址为GName。

- 调试符号：`FName::GetNames`
    
    ![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId13.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId13.png)
    

# 三、GObjectArray

- **字符串**：`GObject: CanvasObject`

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId14.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId14.png)

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId15.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId15.png)

由于GObjectArray是全局，然后C++编译器直接把GObjectArray的Num成员以地址形式输出。查看NumElements偏移然后计算即可。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId16.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId16.png)

- **构造特征**：

首先简单认识一下UObject的结构和一些属性。

```cpp
class UObject
{
  /** virtual function table */
  void* vtf; 
/** Flags used to track and report various object states. This needs to be 8 byte aligned on 32-bit
	    platforms to reduce memory waste */
	EObjectFlags					ObjectFlags;
	/** Index into GObjectArray...very private. */
	int32							InternalIndex;                //表明该对象在GObjectArray中的第几个
	/** Class the object belongs to. */
	UClass*							ClassPrivate;
	/** Name of this object */
	FName							NamePrivate;
	/** Object this object resides in. */
	UObject*						OuterPrivate;
}
```

还有GObjectArray结构。

```cpp
class FUObjectItem
{
  	// Pointer to the allocated object
	class UObjectBase* Object;
	// Internal flags
	int32 Flags;
	// UObject Owner Cluster Index
	int32 ClusterRootIndex;	
	// Weak Object Pointer Serial number associated with the object
	int32 SerialNumber;
}
class TUObjectArray
{
  enum
	{
		NumElementsPerChunk = 64 * 1024,
	};
	/** Master table to chunks of pointers **/
	FUObjectItem** Objects;
	/** If requested, a contiguous memory where all objects are allocated **/
	FUObjectItem* PreAllocatedObjects;
	/** Maximum number of elements **/
	int32 MaxElements;
	/** Number of elements we currently have **/
	int32 NumElements;
	/** Maximum number of chunks **/
	int32 MaxChunks;
	/** Number of chunks we currently have **/
	int32 NumChunks;
}
class FUObjectArray
{
  int32 ObjFirstGCIndex;
	/** Index pointing to last object created in range disregarded for GC.					*/
	int32 ObjLastNonGCIndex;
	/** Maximum number of objects in the disregard for GC Pool */
	int32 MaxObjectsNotConsideredByGC;
	/** If true this is the intial load and we should load objects int the disregarded for GC range.	*/
	bool OpenForDisregardForGC;
	/** Array of all live objects.											*/
	TUObjectArray ObjObjects;
}
```

手动查看GObject的第一个对象，**Index = 0** 。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId17.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId17.png)

第二个对象，**Index = 1** 。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId18.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId18.png)

那么可以构造一个临时特征码。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId19.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId19.png)

特征码进行模糊处理后得到：`?? ?? ?? ?? ?? 7F 00 00 ?? ?? ?? ?? 00 00 00 00`

这个时候就不断地尝试假设Index的值，然后到CE中搜索这个特征码，看看Index为几的时候结果是最少的。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId20.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId20.png)

经过测试得到Index = 0xAC的结果最少，然后排除掉绿的地址后将剩下四个拉下来后逐个搜索，经过测试只有地址0x25CC2DC8D40能搜索到结果。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId21.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId21.png)

将地址拿下来后逐个进行结构分析，查看FObjectItem的大小，结构如下图。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId22.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId22.png)

经过排查最终只有下列地址符合我们的需求。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId23.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId23.png)

现在假设`sizeof(FObjectItem)=0x10`。然后`Index=0xAC`，那么该Object在GObjectArray的偏移就等于`0xAC * 0x10 = 0xAC0`，我们将第一个地址减掉0xAC0得到0x25CC47CFF00，然后搜索这个地址。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId24.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId24.png)

发现没有结果，说明这个地址不是来自GObject，换下一个搜索。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId25.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId25.png)

同样为0，说明`sizeof(FObjectItem) != 0x10`。那么同样的操作假设`sizeof(FObjectItem)=0x20`，再重复一遍刚刚的操作，后搜索。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId26.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId26.png)

发现有结果，那么**0x25**开头的地址拿下来搜索。

![/posts/UnrealEngine/二、UE4三件套获取/document_image_rId27.png](/posts/UnrealEngine/二、UE4三件套获取/document_image_rId27.png)

GObject到手。