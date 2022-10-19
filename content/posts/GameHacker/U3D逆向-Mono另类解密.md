---
title: "U3D逆向-Mono另类解密"
description: "U3D逆向-Mono另类解密"
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

# U3D逆向-Mono另类解密

- [PlaneJun](https://bbs.pediy.com/user-home-826671.htm) ・2021-5-26 17:24
- 2021-06-04 14:11 登上看雪公众号头条
- [https://mp.weixin.qq.com/s/nXZ2qfD4Qmt65aq0_0F6ag](https://mp.weixin.qq.com/s/nXZ2qfD4Qmt65aq0_0F6ag)
    
![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId6.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId6.png)
    

很久不来看雪了，这段时间过得真的是一言难尽。许久的发呆后决定干点有意义的事，于是乎就顺手来写一篇技术文章，算是本年的打卡。

如题所示，U3D的Mono解密，这个玩意我昨晚在我的技术群里貌似看到说U3D是要弃用了Mono这个机制，也不知道是真是假，趁着还没弃用，我也就来说说这个机制的一个另类解密。

关于Mono这东西我就不啰嗦了，我就直接进入正题。众所周知，Mono的加密主要是针对 **Assembly-CSharp.dll**，此DLL包含了游戏的所有功能性函数，并且可以通过工具dnSpy.exe加载后进行查看。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId7.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId7.png)

此DLL公开等于说源码公开，可以通过C#工程引入该DLL自写一个GameObject注入到游戏里调用游戏自带函数实现作弊。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId8.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId8.png)

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId9.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId9.png)

大多的加密手段就是对这DLL进行二进制处理，也就是把文件字节给进行处理。对于这种加密的处理方式很简单，Mono.dll是U3D用来初始化并加载dll的一个模块，他里面有一个函数`mono_image_open_from_data_with_name`，这里放一下他的函数代码。

```cpp
MonoImage *
mono_image_open_from_data_with_name (char *data, guint32 data_len, gboolean need_copy, MonoImageOpenStatus *status, gboolean refonly, const char *name)
{
    return mono_image_open_from_data_internal (data, data_len, need_copy, status, refonly, FALSE, name);
}
```

```cpp
MonoImage *
mono_image_open_from_data_internal (char *data, guint32 data_len, gboolean need_copy, MonoImageOpenStatus *status, gboolean refonly, gboolean metadata_only, 
                const char *name)
{
    MonoCLIImageInfo *iinfo;
    MonoImage *image;
    char *datac;
 
    if (!data || !data_len) {
        if (status)
            *status = MONO_IMAGE_IMAGE_INVALID;
        return NULL;
    }
    datac = data;
    if (need_copy) {
        datac = (char *)g_try_malloc (data_len);
        if (!datac) {
            if (status)
                *status = MONO_IMAGE_ERROR_ERRNO;
            return NULL;
        }
        memcpy (datac, data, data_len);
    }
 
    image = g_new0 (MonoImage, 1);
    image->raw_data = datac;
    image->raw_data_len = data_len;
    image->raw_data_allocated = need_copy;
    image->name = (name == NULL) ? g_strdup_printf ("data-%p", datac) : g_strdup(name);
    iinfo = g_new0 (MonoCLIImageInfo, 1);
    image->image_info = iinfo;
    image->ref_only = refonly;
    image->metadata_only = metadata_only;
    image->ref_count = 1;
 
    image = do_mono_image_load (image, status, TRUE, TRUE);
    if (image == NULL)
        return NULL;
 
    return register_image (image);
}
```

看不懂没关系，我们只需要关注他的**data,data_len,name**这三个参数，这三个分别表示当前被加载模块的二进制内容，二进制长度，模块名。大部分游戏厂商都会在这里判断模块名是否为Assembly-CSharp，然后进行二进制内容解密。那么只需要用调试工具在这个函数下段，然后在这里分析结束的位置，然后直接dump即可。这里放一下大概代码

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId10.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId10.png)

直接在解密完毕的位置下断，然后把rdi给dump出来就行，得到的就是解密后的dll。也可以自写脚本。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId11.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId11.png)

上面介绍了Assembly-CSharp的一种加密和解密方式，虽说是加密，但是非常下饭，基本有手就行。今天就来说说另一种加密方式。小白鼠是国内某款游戏，现在应该应该不开放了，但是游戏依旧躺在我硬盘里。该游戏同样是加密Assembly-CSharp，但是有个不同点就是该文件只有1kb，这个就很可疑，因为这等于说这文件是空的，用十六进制打开文件也能看出里面无内容，那么游戏很可能是联网获取新的，或者是内存释放。那么除了去分析后解密，就没有办法拿到解密后的文件吗，答案是否定的。这个时候就需要上点硬技术了。

回到Mono的源码，我们需要去分析一下他的一个流程。这里我就不去分析了，有一个帖子已经说得很详细。[[原创]关于Unity游戏Mono注入型外挂的一个检测思路-编程技术-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-264444.htm)

大家可以看看这个帖子，主要是去熟悉一下mono的一个流程。我也直接请出主角。

```cpp
static MonoImage *
register_image (MonoImage *image)
{
    MonoImage *image2;
    GHashTable *loaded_images = get_loaded_images_hash (image->ref_only);           //       重点关注对象
 
    mono_images_lock ();
    image2 = (MonoImage *)g_hash_table_lookup (loaded_images, image->name);
 
    if (image2) {
        /* Somebody else beat us to it */
        mono_image_addref (image2);
        mono_images_unlock ();
        mono_image_close (image);
        return image2;
    }
 
    GHashTable *loaded_images_by_name = get_loaded_images_by_name_hash (image->ref_only);
    g_hash_table_insert (loaded_images, image->name, image);                       //       重点关注对象
    if (image->assembly_name && (g_hash_table_lookup (loaded_images_by_name, image->assembly_name) == NULL))
        g_hash_table_insert (loaded_images_by_name, (char *) image->assembly_name, image);
    mono_images_unlock ();
 
    return image;
}
```

这里可以清楚的看到，U3D直接把需要的加载的模块插入了一个**HashTable**，然后完成模块的一个装载。这里我们先不管这个模块进行了如何加密和解密，既然你要扔给U3D托管，那么你不可能扔了一个加密模块的给U3D托管，除非你的U3D引擎进行了一个大规模的魔改。所以说，我们只需要找到这个HashTable，然后去遍历一下里面的模块，我们是不是就能拿到解密后的Assembly-CSharp，答案是肯定的。那么现在重点对象从 如何解密Assembly-CSharp变成了 如何使用**loaded_images**。

我们应该如何去找到这个loaded_images？没办法，上分析。调试器打开游戏跳到**mono_image_open_from_data_with_name**这个函数然后对照源码进行分析。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId13.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId13.png)

从源码得知，register_image是最后一个调用的函数，我们直接找到最后一个函数，进去后分析得知。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId14.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId14.png)

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId15.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId15.png)

拿到loaded_images后，我们下一步就是要去遍历这个HashTabel,两个办法，一个是自己去搭建**GHashTable**库，然后自己去跑一遍。第二个办法，自己去分析g_hash_table_insert函数的流程。我选择后者 =。=

Ida打开mono.dll后搜索**g_hash_table_insert**，有一个

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId16.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId16.png)

进去F5后无脑暴力分析。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId17.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId17.png)

分析了一个大概流程后，打开CE，用数据结构工具分析验证看看

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId18.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId18.png)

很明显，这里应该就是模块列表，我们再随便进去一个去看看。**通过刚刚IDA的分析，0x0是模块名，0x8是image。**

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId19.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId19.png)

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId20.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId20.png)

和我们刚刚分析的一致，现在我们还差一个东西，那就是image的结构，我们在源码查找一下。

```cpp
struct _MonoImage {
           ………
           guint32 raw_data_len;
           char *raw_data;                   //模块二进制
           char *name;              //模块名
           …………
}
```

这里就贴出重要的。问题来了，我们怎么知道他的偏移。别忘了，这里有

```cpp
MonoImage *
mono_image_open_from_data_internal (char *data, guint32 data_len, gboolean need_copy, MonoImageOpenStatus *status, gboolean refonly, gboolean metadata_only, 
                const char *name)
{
    MonoCLIImageInfo *iinfo;
    MonoImage *image;
    char *datac;
 
    if (!data || !data_len) {
        if (status)
            *status = MONO_IMAGE_IMAGE_INVALID;
        return NULL;
    }
    datac = data;
    if (need_copy) {
        datac = (char *)g_try_malloc (data_len);
        if (!datac) {
            if (status)
                *status = MONO_IMAGE_ERROR_ERRNO;
            return NULL;
        }
        memcpy (datac, data, data_len);
    }
 
    image = g_new0 (MonoImage, 1);
    image->raw_data = datac;                                                                       <<<<<<<<<<<<<<<
    image->raw_data_len = data_len;                                                                <<<<<<<<<<<<<<<
    image->raw_data_allocated = need_copy;
    image->name = (name == NULL) ? g_strdup_printf ("data-%p", datac) : g_strdup(name);            <<<<<<<<<<<<<<<
    iinfo = g_new0 (MonoCLIImageInfo, 1);
    image->image_info = iinfo;
    image->ref_only = refonly;
    image->metadata_only = metadata_only;
    image->ref_count = 1;
 
    image = do_mono_image_load (image, status, TRUE, TRUE);
    if (image == NULL)
        return NULL;
 
    return register_image (image);
}
```

我们ida返回去分析

```cpp
__int64 __fastcall sub_18006CA20(__int64 data, unsigned int data_len, int need_cpy, _DWORD *status, __int64 refonly, char metadata_only, __int64 name)
{
  __int64 len; // r14
  _DWORD *v8; // rdi
  char v9; // si
  __int64 data_; // rbx
  __int64 data__; // rbp
  __int64 v12; // rax
  __int64 result; // rax
  __int64 v14; // rbx
  __int64 m_name; // rax
  signed __int64 v16; // rax
  __int64 v18; // rax
  __int64 v19; // rax
 
  len = data_len;
  v8 = status;
  v9 = need_cpy;
  data_ = data;
  if ( data && data_len )
  {
    data__ = data;
    if ( need_cpy )
    {
      v12 = sub_180004B80(data_len);
      data__ = v12;
      if ( !v12 )
      {
        if ( v8 )
          *v8 = 1;
        return 0i64;
      }
      sub_180314D40(v12, data_, len);
    }
    v14 = sub_180004AE0(1856i64);
    *(_BYTE *)(v14 + 0x1C) &= 0xFDu;
    *(_BYTE *)(v14 + 0x1C) |= 2 * (v9 & 1);
    *(_QWORD *)(v14 + 0x10) = data__;   //data
    *(_DWORD *)(v14 + 0x18) = len;      //data_len
    if ( name )
    {
      v16 = -1i64;
      while ( *(_BYTE *)(name + v16++ + 1) != 0 )
        ;
      m_name = sub_180004A10(name, (unsigned int)(v16 + 1));
    }
    else
    {
      m_name = sub_180006230("data-%p", data__);
    }
    *(_QWORD *)(v14 + 0x20) = m_name;    //name
    v18 = sub_180004AE0(408i64);
    *(_BYTE *)(v14 + 0x1C) &= 0xBFu;
    *(_BYTE *)(v14 + 0x1D) &= 0xFEu;
    *(_QWORD *)(v14 + 0x50) = v18;
    *(_DWORD *)v14 = 1;
    *(_BYTE *)(v14 + 0x1C) |= (refonly & 1) << 6;
    *(_BYTE *)(v14 + 0x1D) |= metadata_only & 1;
    v19 = sub_1800699B0(v14, v8, 1i64);
    if ( !v19 )
      return 0i64;
    result = sub_18006D6D0(v19);
  }
  else
  {
    if ( status )
      *status = 3;
    result = 0i64;
  }
  return result;
}
```

可知

```cpp
0x10 raw_data
0x18 raw_data_len
0x20 name
```

我们再用CE看看

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId21.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId21.png)

可以看到，直接是一个标准的PE文件，说明这里存的是二进制内容。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId22.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId22.png)

到这里，我们就已经分析完了loaded_images的一个内存结构。接下来就是写一个遍历工具，然后打开游戏，运行工具，等待DLL生成。这里我用易语言写了一个。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId23.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId23.png)

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId24.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId24.png)

**运行。。。**

原文件

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId25.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId25.png)

dump文件

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId26.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId26.png)

**Dnspy打开**

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId27.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId27.png)

到这里，就已经完美提取了。这里的话，个人觉得，也可以提取一些第三方挂钩mono的DLL，但是本人没去试过。有兴趣的可以去试试。输入法和键盘同时出了问题，打字很难受，如果文章中存在错别字，，，见谅。。。。

![/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId28.png](/posts/GameHacker/U3D逆向-Mono另类解密/document_image_rId28.png)