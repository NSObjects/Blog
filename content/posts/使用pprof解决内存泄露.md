---
title: "使用pprof解决内存泄露"
date: 2021-05-13T20:21:21+08:00
draft: false
---

## 前言
在公司项目中使用go-micro作为微服务框架，但是随着服务越来越多线上监控发现内存在不断上涨，后来使用pprof定位到具体原因并修复，本文介绍我是如何通过pprof一步一步找出这个问题，至于pprof是什么和如何使用可以直接看文章末尾的Reference

![](/images/pleak.png)

## 开启pprof
在main包导入pprof就可以开启pprof，如果使用gin,echo可以到github查找对应的插件
```
_ "net/http/pprof"
```

## 通过web查看内存分配情况
执行这条命令后将会在浏览器打开一个可视化界面 `go tool pprof -http :9090 https://xxxx.com/debug/pprof/heap`
通过查看堆栈信息我们发现从go-micro/uitl/kubernetes/client.Update()代码开始内存就开始泄露。
找到这部分代码，我们发现在go-micro封装了一套http请求的方法，但是在每次调用完之后都没有调用Body.Close()方法。众所周知在使用`http.Client`发完请求后是需要调用resp.Body.Clouse()方法的，不然会发生内存泄露。看到这里我们只需要fork一份2.3.0的go-micro代码，然后在每次发完请求后调用下Close()方法，服务的内存就全部都正常了

![](/images/leak_ui.png)



### Reference

https://segmentfault.com/a/1190000016412013

https://colobu.com/2019/08/20/use-pprof-to-compare-go-memory-usage/
