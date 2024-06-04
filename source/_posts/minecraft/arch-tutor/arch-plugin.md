---
title: 深入了解Architectury Plugin
date: 2022-06-11 11:35:12
categories: [minecraft, arch-tutor]
tags:
    - Minecraft
    - 模组开发
    - ArchitecturyAPI
---

# ExpectPlatform

### ExpectPlatform

读者应该已经对ExpectPlatform有了一个基本的认识, ExpectPlatform是Architectury Plugin的一个重要注解, 它可以根据平台的不同将**一个方法**映射到**不同的实现**上去.

Architectury对ExpectPlatform修饰的方法有一些规范和要求:&#x20;

* 被修饰的方法必须是**公开**且**静态**的
* 方法的实现, 命名必须是**原方法名+Impl**
* 方法的实现, 应该存放到forge模块, 相同包下名为forge的包中
* 方法的实现, 应该存放到fabric模块, 相同包下名为fabric的包中

在common模块中创建类Storyteller, 并创建一个**公开静态**方法, 添加ExpectPlatform注解

```java
位于common: trou/arch/Storyteller.java
@ExpectPlatform
public static void tellStory() {
    // 方法体为空, Architectury会根据不同的模组加载器自动填充
}
```

在fabric和forge模块中**相同位置**创建类Storyteller**Impl**, 为它添加实现

```java
位于fabric: trou/arch/fabric/StorytellerImpl.java
public class StorytellerImpl {
    public static void tellStory() {
        System.out.println("I'm fabric, your new friend.");
    }
}
```

```java
位于forge: trou/arch/forge/StorytellerImpl.java
public class StorytellerImpl {
    public static void tellStory() {
        System.out.println("I'm forge, your old friend.");
    }
}
```

在common包中, 我们就可以按需调用tellStory方法, 编译时Architectury会根据平台的不同补齐方法

```java
位于common: trou/arch/Arch.java
public static void init() {
    System.out.println("Hello Architectury!");
    Storyteller.tellStory();
}
```

此时分别运行Forge和Fabric客户端, 输出的信息应该有所不同.


我们的文件结构应该是这样的:&#x20;

ExpectPlatform方法: common\src\main\java\trou\arch\Storyteller.java

Forge平台的实现: forge\src\main\java\trou\arch**\forge\\**StorytellerImpl.java

Fabric平台的实现: fabric\src\main\java\trou\arch**\fabric\\**StorytellerImpl.java


![Forge Side](https://s2.loli.net/2022/04/02/TQ4f6vJWEqDRglF.png)

![Fabric Side](https://s2.loli.net/2022/04/02/Xc6QYMF8Es3hIai.png)

### IDEA的Architectury插件

Architectury提供了IDEA的插件来帮助我们补全ExpectPlatform的实现

![Code Hint](https://s2.loli.net/2022/04/02/JBgFSlKcz12d7iI.png)

创建ExpectPlatform方法后, 可以自动在正确的位置创建forge和fabric的实现

# ArchitecturyTarget

### ArchitecturyTarget <a href="#architecturytarget" id="architecturytarget"></a>

利用**ArchitecturyTarget**类的**getCurrentTarget**方法, 可以直接获取当前运行的模组加载器平台.


**ArchitecturyTarget.getCurrentTarget()需要在common包中被调用**

* 当forge环境执行到该语句时, 方法的返回值为forge
* 当fabric环境执行到该语句时, 方法的返回值为fabric


下面我们来用ArchitecturyTarget修改我们上一章节编写的Storyteller代码.

修改StoryTeller类, 去掉ExpectPlatform注解, 直接基于当前运行的加载器环境进行判断

```java
public class Storyteller {
    public static final String FORGE_STORY = "I'm forge, your old friend.";
    public static final String FABRIC_STORY = "I'm fabric, your new friend.";
    public static void tellStory() {
        final String platform = ArchitecturyTarget.getCurrentTarget();
        System.out.println(platform.equals("forge") ? FORGE_STORY : FABRIC_STORY);
    }
}
```

现在应该能得到与之前相同的效果. 读者可以根据需要取舍使用ExpectPlatform和ArchitecturyTarget

# PlatformOnly

### PlatformOnly

PlatformOnly是一个用于限定某一方法只能在某一平台执行的注解.

* @PlatformOnly("forge")修饰的方法**只能**在Forge模块调用(编译后不会出现在Fabric代码中)
* @PlatformOnly("fabric")修饰的方法**只能**在Fabric模块调用(编译后不会出现在Forge代码中)

被PlatformOnly修饰的方法如果在common包直接调用会产生警告 (需要安装插件)

![Warning](https://s2.loli.net/2022/04/02/Hp85GEtrSfOKueB.png)

如果我们尝试在Forge模块中调用被@PlatformOnly("fabric")修饰的方法, 游戏会抛出反射异常

![InvocationTargetException](https://s2.loli.net/2022/04/02/1DZfJ9xFXowS5mn.png)

# Environment & OnlyIn

### Transform to Environment

在Forge中, 为了标识某一方法只在逻辑服务端执行, 我们通常会添加@OnlyIn(Dists.DEDICATED\_SERVER)

在Fabric中, 为了标识某一方法只在逻辑服务端执行, 我们通常会添加@Environment(EnvType.SERVER)

为了统一这两个相同的需求, Architectury为我们添加了自动处理的映射关系.

在实际代码中, 我们只需要添加Environment注解即可, Architrctury会对Forge平台进行映射处理

![LightOverlay](https://s2.loli.net/2022/04/02/rU8y6HdeiV4oaXP.png)

