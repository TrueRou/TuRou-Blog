---
title: 从零开始认识Architectury
date: 2022-06-10 11:35:12
categories: [arch-tutor]
tags:
    - Minecraft
    - 模组开发
    - Architectury
---

### 本文的想法来源

前些天在Curseforge的Library区闲逛，发现了一直在前排榜单上的**Architectury API**，笔者稍加了解以后决定起笔这篇文章。类似于**Architectury**这种的跨平台思路最近还是挺流行的，比如flutter，uno什么的，但是在Minecraft开发领域并不多见，查了一下Curseforge上依赖**Architectury API**的Mod，发现还挺多的，可能也算比较流行了。

### 本文的受众对象

* **有跨加载器开发模组的需求 (如果没有这个需求的话就没必要阅读了)**
* 对Minecraft及其生态链有一个较为进阶的了解
* **有利用Fabric或Forge开发Mod的经验 (本文不会讲解模组开发的基础部分)**
* 有独立思维解决问题的能力

### 本文的组织方式

本文不会详细记述如何开发一个完整的Forge或Fabric模组，只会抽出开发中的几个关键点进行诠释，并加以笔者感兴趣的实例进行补充讲解。

在Architectury Plugin部分中，注重于介绍**Architectury**的特性和使用方式。所以，这部分**不是**思路引领性的，**而是**知识引领性的。

而Architectury API部分中，注重于介绍**Architectury**的实际使用和开发经验。主要以实例的角度，这部分是思路引领性的，熟悉笔者写作习惯的读者应该更乐于去接受这种方式。

对于希望快速上手的读者, 可以阅读完前言并搭建完环境后, 直接阅读Architectury API部分.

### 本文基于

* Minecraft: 1.18.2
* Architectury: 4.1.32
* Fabric Loader: 0.13.3
* Fabric API: 0.48.0+1.18.2
* Forge: 1.18.2-40.0.17
* (Parchment: 1.18.2-2022.03.13)

### 正文开始

Architectury 是一套跨加载器的模组开发工具链，依靠这套工具链，读者可以快速且舒适的**一次性**开发出模组的**Forge**和**Fabric**版本，而无需构建多套开发环境。

Architectury分为**Architectury Plugin**和**Architectury API**。

Architectury Plugin是一个Gradle插件，意在快速而方便的帮助开发者构建出一个包含Forge模块、Fabric模块和Common模块的项目。

在开发过程中，开发者只需把重心放在Common模块的实现，而为了适配两个不同加载器的**额外代码**则需要编写在forge和fabric模块中，最后构建时Architectury会帮助你进行混合。

开发者的编码组织方式类似于下图:

![](https://s2.loli.net/2022/03/28/3KqQpujYFETZin2.png)

起初开发者们认为这种开发方式的确带来了很大的便利。但因为Fabric和Forge本身存在较大的差异，开发整仍需花费大量时间在Forge模块和Fabric模块的开发上，而且开发者们仿佛都在**重复地**做同一类事情​：

* 注册物品: Common调用注册的方法。Forge里面实现一遍, Fabric里面实现一遍
* 监听事件: Common调用监听事件的方法。Forge里面实现一遍, Fabric里面实现一遍
* 网络发包: Common调用发包的方法。Forge里面实现一遍, Fabric里面实现一遍

所以Architectury API横空出世。

Architectury API在Forge和Fabric上封装了一个抽象层，开发者实际开发中可以跟这些API层打交道，组织方式类似于下图:

![](https://s2.loli.net/2022/03/28/JMs24kZCXvUaAKh.png)

所以实际上我们的开发实际上主要是围绕Architectury API进行的。

在Architectury的工具链中，还有一个重要角色就是Architectury Loom，相信之前尝试过利用Fabric开发的读者应该会想到Fabric Loom, 负责构建开发环境时反编译Minecraft源码并且反混淆的工具。

这里Architectury创造性的把这套东西搬到了Forge, 替代了Forge Gradle，感兴趣的读者可以自行了解。

可以想到的，为了便于管理，在我们的开发环境搭建中也使用Architectury Loom，构建环境时Architectury会自动为我们进行必要的操作，不需要我们手动干涉。

Architectury Loom也可以作为Forge Loom独立使用，如果看不惯Forge Gradle的读者也可以尝试独立使用它构建Forge环境，但是需要注意的是，这套框架还处于测试阶段，可能出现无法构建或者杂七杂八的问题，具体的信息可以在这里找到: [https://github.com/architectury/architectury-loom](https://github.com/architectury/architectury-loom)

相信读者已经基本了解了Architectury，下面我们开始尝试基于Architectury开发模组。请注意，构建过程中需要**通畅**的网络，请读者自行解决，不过鉴于forge-cdn和fabric最近的情况，读者应该不必担心。