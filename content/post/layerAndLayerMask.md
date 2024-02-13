---
title: "[Unity]Layer & LayerMask 简述"
date: 2022-02-19T22:36:34+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Unity"]
---

# Layer
编辑器里的layer（总共32层）：

就像PS里的图层，帮助你通过分层，层与层不会互相干扰。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55767aeb68e748e1b167dbdd731a7637~tplv-k3u1fbpfcp-watermark.image?)

# LayerMask
layer之所以是32层，是因为他是用int32表示，但是表示方法是：
```csharp
0000 0000 0000 0000 0000 0000 0000 0000 
```
假如是第0层, 则表示为：
```csharp
0000 0000 0000 0000 0000 0000 0000 0001 
```
假如是第5层, 则表示为：
```csharp
0000 0000 0000 0000 0000 0000 0010 0000 
```
所以，是按位看的，从左往右第一位是1，就表示第0层，以此类推。

LayerMask就坑在这。

```csharp
LayerMask mask = ~(1 << 9) ;// 打开除了第9之外的层。
```
所以LayerMask 的值，可以用位运算进行赋值。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e051fe7fb7a5412cb69a3a409974b2b2~tplv-k3u1fbpfcp-watermark.image?)

比如 LayerMask mask = 1<<3|0<<5 表示开启Layer3并且同时关闭Layer5。

# gameObject.layer
gameObject.layer 接收的是第几层，而不是位掩码的值。gameObject.layer = 6；将层设为6。

而LayerMask的值，会给你换算成掩码的int32值。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c17008d1dbdc4a0ca43d59ee2c40c701~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/991605f3d9ba4ae49e82bfab2833841b~tplv-k3u1fbpfcp-watermark.image?)

# Physics.Raycast 等方法
```csharp
Physics.Raycast(ray, out hit, 1000, 1<<LayerMask.NameToLayer("TestLayer"))
```
像 Physics.Raycast 等方法，接收的 LayerMask 是位掩码，假如我想检测第6层，填6，会变成检测 0110 ，第1层和第2层。所以要填 1 << 6 或者 64 或者对应的 LayerMask 对象。