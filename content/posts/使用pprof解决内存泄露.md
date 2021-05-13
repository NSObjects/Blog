---
title: "使用pprof解决内存泄露"
date: 2021-05-13T20:21:21+08:00
draft: false
---

## 前言
在公司项目中使用go-micro作为微服务框架，但是随着服务越来越多线上监控发现内存在不断上涨，后来使用pprof定位到具体原因并修复，本文介绍我是如何通过pprof一步一步找出这个问题
![](/static/images/pleak.png)
![Example image](/images/pleak.png)
## pprof是什么？
> pprof is a tool for visualization and analysis of profiling data.

> pprof reads a collection of  samples in profile.proto format and generates reports to visualize and help analyze the data. It can generate both text and graphical reports (through the use of the dot visualization package).

pprof是用于可视化和分析性能分析数据的工具，可以通过收集到的采样数据生成文本或者图形报告
![]c
## 开启pprof
在main包导入pprof就可以开启pprof，如果使用gin,echo可以到github查找对应的插件
```
_ "net/http/pprof"
```

