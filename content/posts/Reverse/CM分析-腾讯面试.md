---
title: "CM分析-腾讯面试"
description: "CM分析-腾讯面试"
tags: ["Reverse"]
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

# CM分析-腾讯面试

# 一、反调试

## 1、分析

程序启动时会对Zw函数进行拷贝到自身内存中，并将内存加密。实现过掉常规API检测。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId4.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId4.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId5.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId5.png)

然后对`BeginDebug`和`ForceFlags`标志位进行检测，如果根据是否进行调试来设置自定义值。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId6.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId6.png)

执行到MFC入口后会首先判断Wow64Transition是否被hook。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId7.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId7.png)

然后对`PEB->ProcessHeap->Flags`、`PEB->ProcessHeap->ForceFlag`、`PEB->BeginDebug`、`PEB->NtGlobalFlag`多个标志位进行检查是否存在调试。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId8.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId8.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId9.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId9.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId10.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId10.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId11.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId11.png)

除此之外，程序会通过解密自拷贝的Zw函数来进行反调试。

- 通过设置ThreadInformation为`ThreadHideFromDebugger`，将调试器分离实现反调试。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId12.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId12.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId13.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId13.png)

- 通过查询调试对象来判断是否进行调试。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId14.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId14.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId15.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId15.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId16.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId16.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId17.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId17.png)

- 调试时默认会有一个调试对象，程序尝试创建一个调试对象后获取，如果获取不到则表明处于调试状态。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId18.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId18.png)

- 通过传递非法指针地址引发0xC0000005 异常，如果执行后返回的异常码不正确则表明调试器接管了异常处理，即存在调试。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId19.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId19.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId20.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId20.png)

- 常规API检测反调试。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId21.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId21.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId22.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId22.png)

- 查询系统信息

![/posts/Reverse/CM分析-腾讯面试/document_image_rId23.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId23.png)

- 引发异常检测调试

![/posts/Reverse/CM分析-腾讯面试/document_image_rId24.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId24.png)

其中，反调试检测大多在窗口消息中。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId25.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId25.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId26.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId26.png)

包括文字的渲染。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId27.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId27.png)

## 2、反反调试

### 2.1、Patch

通过nop 反调试函数方式或者修改返回值的方式的patch实现过掉反调试。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId28.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId28.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId29.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId29.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId30.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId30.png)

### 2.2、Hook Zw

### 2.2.1 EAT

在程序启动初期，快速将ntdll.dll的导出函数进行替换后自写分发过滤。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId31.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId31.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId32.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId32.png)

### 2.2.2 替换wow64transition

通过ce查看32位下的syscall。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId33.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId33.png)

可以将mov edx,xxxxxx的硬编码20 8f 87 77 修改为自己的函数地址，然后通过eax的值判断函数后进行自定义处理。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId34.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId34.png)

由于CM会检测wow64transition头部是否为0xE9(默认 FF 25 xxxxxxx)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId35.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId35.png)

因此自定义函数的头字节不能为0xe9。但在实现后测试的环节中发现部分Zw函数调用时崩溃，原因也暂未查找，因此该方法只作为一个额外的思路。

# 二、注册机

## 1、算法分析

![/posts/Reverse/CM分析-腾讯面试/document_image_rId36.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId36.png)

Win32获取EditBox函数大多是GetWindowText，但是在ida中交叉引用发现并没有什么有价值的东西。但是在导入表看到`OutputDebugStringA`,然后尝试打开了DbgView看看能不能捕获到内容。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId37.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId37.png)

发现当用户名和注册码输入长度较长时，点击注册有输出，但该输出并非正确的注册码。猜测上层有对注册逻辑处理，交叉引用后发现上层代码无法进行伪代码生成。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId38.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId38.png)

简单分析后发现该段代码存在异常处理和花指令。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId39.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId39.png)

在异常地址0x401B86和异常函数0x0415890下断点后Shift+F9跳转到自定义异常跟踪分析后，发现该处最终的跳转结果为0x401BA1。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId40.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId40.png)

对代码进行修复后且nop无效代码后，ida已经可以进行伪代码生成。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId41.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId41.png)

![/posts/Reverse/CM分析-腾讯面试/document_image_rId42.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId42.png)

注册算法为从地址为0x401BA9处循环执行0x960次，每次以四字节大小读取地址数据后与初始值的Key(0x19820714)，进行异或运算得到最后一个校验值。然后生成三个固定常量表。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId43.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId43.png)

紧接着获取用户名和注册码对应EditBox的内容。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId44.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId44.png)

其中会判断用户名长度是否为8，注册码长度是否为24。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId45.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId45.png)

最后将用户名和上边生成的三个表、Key进行计算注册码。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId46.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId46.png)

计算完毕后会执行函数OutputDebugStringA将计算出来的注册码输出，但输出的方式为将计算出的注册码的头和尾字符各上升一个字符，及"0"->"1"。

![/posts/Reverse/CM分析-腾讯面试/document_image_rId47.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId47.png)

## 2、注册机实现

```cpp
#include <stdio.h>
#include <Windows.h>

namespace MemTools {

	HANDLE	m_hProcess;
	DWORD	m_pid = NULL;

	void Attach(DWORD pid)
	{
		m_pid = pid;
		if (m_hProcess)
			CloseHandle(m_hProcess);
		m_hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
	}

	template<typename T>
	T ReadMem(DWORD addr)
	{
		if (!m_hProcess)
			return T();
		T value{};
		ReadProcessMemory(m_hProcess, (LPVOID)addr, &value, sizeof(T), NULL);
		return value;
	}
}

int main() {

	int key = 0x19820714;
	HWND hWnd = FindWindowA(NULL,"CrackMeDemo");
	DWORD pid{};
	GetWindowThreadProcessId(hWnd, &pid);
	MemTools::Attach(pid);

	for (int i = 0; i < 0x960; i++)
		key ^= MemTools::ReadMem<DWORD>(0x401ba9 + i*4);

	char v20[27] ="0";
	char v26[27] = "A";
	char v18[27] = "a";

	for (int j = 1; j < 26; ++j)                  // 生成表
	{
		v20[j] = v20[j - 1] + 1;
		v26[j] = v26[j - 1] + 1;
		v18[j] = v18[j - 1] + 1;
	}

	char user[] = "test1234";
	char pwd[1024]{};
	if (strlen(user) != 8)
		return 0;

	for (int k = 0; k < 8; k++)
	{
		user[k] ^= k;
		user[k] ^= MemTools::ReadMem<BYTE>(0x401BB9 +k);
		user[k] ^= MemTools::ReadMem<BYTE>(0x401BC9 +k);
	}
	*(DWORD*)(user) ^= key;
	*(DWORD*)&user[4] ^= key;

	for (int m = 0; m < 8; m++)
	{
		
		unsigned __int8 v8 = (unsigned __int8)(user[m] & 0xE0) / 32;
		unsigned __int8 v6 = (user[m] & 0x1c) / 4;
		unsigned __int8 v7 = user[m] & 3;
		if (m % 3 == 2)
		{
			pwd[m * 3] = v20[v7];
			pwd[m * 3 + 1] = v26[v8 + 8];
			pwd[m * 3 + 2] = v18[v6 + 16];
		}
		if (m % 3 == 1)
		{
			pwd[m * 3] = v26[v8 +16];
			pwd[m * 3 + 1] = v18[v6 + 8];
			pwd[m * 3 + 2] = v20[v7];
		}
		if (!(m % 3))
		{
			pwd[m * 3] = v26[v6 +16];
			pwd[m * 3 + 1] = v18[v7 + 8];
			pwd[m * 3 + 2] = v20[v8];
		}
	}
	printf("%s\n",pwd);
	return(0);
}
```

![/posts/Reverse/CM分析-腾讯面试/document_image_rId48.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId48.png)

DbgView捕获到的内容

![/posts/Reverse/CM分析-腾讯面试/document_image_rId49.png](/posts/Reverse/CM分析-腾讯面试/document_image_rId49.png)

将头尾各降一个字符得到的内容与注册机一致。

Uk4Xm30PtWj6Vl13LtRk2Xp1 -> Tk4Xm30PtWj6Vl13LtRk2Xp0