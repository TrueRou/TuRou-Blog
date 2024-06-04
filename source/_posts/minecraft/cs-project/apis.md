---
title: CustomStuff3 API概述
date: 2017-11-31 10:46:56
categories: [minecraft, cs-project]
tags:
    - Minecraft
    - 模组开发
    - CustomStuff3
---

![635589679383149678.png](https://s2.loli.net/2023/01/10/k8xFHuVPMw9pe14.png)

-
<font size=3 face="微软雅黑">
 **<font size=3 face="微软雅黑">声明:本API文档是帮助理解CS3的JS内容，编写:TROU</font>**   
-
 **<font size=3 face="微软雅黑">序言：</font>**   
 距离上次发教程也过了很久了，我考虑了一下，我决定先放出API及事件部分，可能还是需要这部分的人比较多，官方的游戏内帮助写的也不是很清楚，我也顺便完善以下，给第一次使用JS的朋友们加一些注释和举例，以便于大家理解，这章会比较长，我会都写在这一个文件里面，以便于查看，有些难以理解的或者太过复杂的内容我将会使用gif或视频来讲解，最后说一句话，不要局限在我的例子里面，灵感是属于你自己的，我只是帮助你理解而已，废话我就不说了，直接开始API部分

 **<font size=3 face="微软雅黑">格式：</font>**   
> 这是我的注释
> 关于语法的易错点也会写在里面

    这是代码
    关于代码的注释也会使用{}写在里面
 普通文字

{这是代码注释}

 **重要的文字**
 

----------

 **<font size=5 face="微软雅黑">Part1：World</font>**   
> 对于当前存档中的世界的操作及信息

可使用world操作的触发动作:

    onBlockDestroyed 当使用这个物品破坏方块的时候
    onBlockStartBreak 当方块被破坏的时候(破坏前一瞬间)
    onCreated 当你制作这个东西的时候(合成/冶炼)
    onDroppedByPlayer 当玩家捡起这个东西的时候
    onEaten 被吃掉的时候
    onHitEntity 击打实体的时候
    onLeftClickLiving 左击生物的时候
    onLeftClickPlayer 左击玩家的时候
    onRightClick 右击的时候
    onStoppedUsing 停止使用的时候
	onUpdate 更新的时候(只要你身上有这个东西就会循环执行)
    onUse 被使用的时候
	onUseOnEntity 在实体上使用的时候
	onUseOnPlayer 在玩家上使用的时候
	onUsing 正在使用的时候
	onActivated 活跃的时候(空手右击)
    onAdded 增加的时候
	onBonemeal 使用骨粉的时候
	onBreak 被破坏的时候
    onClicked 右击的时候
    onCollided 生物穿过的时候(循环执行)
    onFallenUpon 在上面跳跃的时候
    onNeighborChange 周围方块改变的时候(检测方块更新)
    onPlaced 被放置的时候
    onPlaceBy 依靠于什么方块放置的时候
    onPlaceByPlayer 被玩家放置的时候
    onRandomDisplayTick 随机发生的事件(通常用于粒子效果的显示)
	onRedstoneSignal 接收到红石信号的的时候
    onUpdate 更新的时候(只要放下就会循环执行)
	onWalking 在上面走的时候

 **<font size=4 face="微软雅黑">World的所有方法:</font>**   
### countEntities (取实体数量)
**调用方式**:world.countEntities(x, y, z, radius, entities)   
**返回值类型**:int(范围内实体的数量)

> 返回在指定位置(x, y, z)的指定范围(radius)内的指定实体(entities)的数量

**参数规则**:

x(int) 欲取出范围中心点的X轴   
y(int) 欲取出范围中心点的Y轴   
z(int) 欲取出范围中心点的Z轴   
> x, y, z这三个参数也可以使用某些触发动作自带的position来定义   

radius(float) 欲检测中心点的范围，注意使用浮点型   
entities(string) 欲检测实体的名称，多个名称使用英文逗号“,”来隔开
> 既可以输入实体ID，也可以输入实体名称(建议使用CS3自带的实体名字获取来确定，因为Minecraft Wiki关于实体部分的内容已经过期，无法使用，且Minecraft实体种类过多，这里不做介绍，如果感兴趣可以在后面找到实体部分的取实体名称)，这里也可以填写实体的种类，我列举一下实体的种类，注意要使用""来括起来，因为这里要求填写的是字符串

    hostile 敌对生物
    animal 动物
    mob 生物
    player 玩家
    item 掉落的物品 
    all 全部

**注意这里的一个实体使用2来表示，比如这个范围内有1个实体，那么数值就是2，有2个实体，数值为4，同样的物品每64个算为一个实体**

**例子**:
> 我在onRightClick中写的代码
>
> 这里会用到一个另外的比较简单的代码，我也会简单介绍，后文详细介绍
>
> 关于代码的注释会使用{}括起来

    带注释的代码:
    var 实体
    {声明实体这个变量}
    实体 = world.countEntities(-209, 4, 48, 5.0, "items, player")
    {取-209, 4, 48这个地方5格以内物品和玩家的数量并储存在“实体”这个变量里面}
    world.sendMessageToAllPlayers(实体.toString())
    {这个语句我们没有接触过，我简单讲解一下，这个语句的意思是把实体里面的数值先转换为字符串，在给全体玩家发送过去(显示在聊天栏)，比如我输入world.sendMessageToAllPlayers("啊啊啊啊")，这样全体玩家的聊天栏都会出现"啦啦啦啦"这四个字，会在以后详细介绍}
    
    不带注释的代码:
    var 实体
    实体 = world.countEntities(-209, 4, 48, 5.0, "items, player")
    world.sendMessageToAllPlayers(实体.toString())

    效果:
    {因为我输入的是检测玩家和物品，一个玩家+两种物品，也就是2+2+2=6}
![](https://miao.su/images/2017/08/11/TIM2017081109465522843.png)

###createExplosion (创建爆炸)
**调用方式**:world.createExplosion(x, y, z, strength, flaming)   
**返回值类型**：没有返回值
> 在你指定的位置创建一个指定强度的爆炸

**参数规则**:

x(float) 欲爆炸位置的X轴   
y(float) 欲爆炸位置的Y轴   
z(float) 欲爆炸位置的Z轴   
> x, y, z这三个参数也可以使用某些触发动作自带的position来定义   

strength(float) 爆炸的强度

    TNT的爆炸强度是4.0

flaming(bool) 是否让爆炸的附近着火(是填true，否填false)

**例子**:

    带注释的代码:
    world.createExplosion(-207, 4, 48, 8.0, true)
    {在-207, 4, 48的位置创建一个会着火的8.0级别的爆炸}

    不带注释的代码:
    world.createExplosion(-207, 4, 48, 8.0, true)

    效果:
    {因为我是超平坦，爆炸效果可能不是特别明显}

![](https://miao.su/images/2017/08/11/TIM20170811095913308de.png)

###createThunderbolt 创建一道雷电
**调用方式**:world.createThunderbolt(x, y, z)   
**返回值类型**:没有返回值
> 在指定的位置创建一道闪电

**参数规则**:

x(float) 欲雷击位置的X轴   
y(float) 欲雷击位置的Y轴   
z(float) 欲雷击位置的Z轴   
> x, y, z这三个参数也可以使用某些触发动作自带的position来定义

**例子**:

    带注释的代码:
    world.createThunderbolt(-226, 4, 70)
    {在-226, 4, 70的位置召唤一道闪电}

    不带注释的代码:
    world.createThunderbolt(-226, 4, 70)

    效果:
    
![](https://miao.su/images/2017/08/11/TIM2017081110062590867.png)

###enumEntities (枚举实体)
**调用方式**:world.enumEntities(x, y, z, radius, entities)   
**返回值类型**:entity(枚举的结果)
> 枚举一个范围内的所有实体

**参数规则**:

x(int) 欲枚举实体范围中心点的X轴   
y(int) 欲枚举实体范围中心点的Y轴   
z(int) 欲枚举实体范围中心点的Z轴   
> x, y, z这三个参数也可以使用某些触发动作自带的position来定义   

radius(float) 欲枚举实体中心点的范围，注意使用浮点型   
entities(string) 欲枚举实体的名称，多个名称使用英文逗号“,”来隔开
> 既可以输入实体ID，也可以输入实体名称(建议使用CS3自带的实体名字获取来确定，因为Minecraft Wiki关于实体部分的内容已经过期，无法使用，且Minecraft实体种类过多，这里不做介绍，如果感兴趣可以在后面找到实体部分的取实体名称)，这里也可以填写实体的种类，我列举一下实体的种类，注意要使用""来括起来，因为这里要求填写的是字符串

    hostile 敌对生物
    animal 动物
    mob 生物
    player 玩家
    item 掉落的物品 
    all 全部

**例子**:

    带注释的代码:
    world.enumEntities(-226, 4, 70, 8.0, "animal")
    {枚举-226, 4, 70为中心点8格范围内的动物}

    不带注释的代码:
    world.enumEntities(-226, 4, 70, 8.0, "animal")

    效果:
    这个命令比较特殊，他的返回值是entity实体，但是我无法对这个实体进行操作，而且这个命令不常用，尽量不要使用

###getBiome (获取生物群系)
**调用方式**:world.getBiome(x, y, z)   
**返回值类型**:string(生物群系的英文名称)
> 获取指定位置的生物群系并返回

**参数规则**:

x(float) 欲取得生物群系的方块的位置的X轴   
y(float) 欲取得生物群系的方块的位置的Y轴  
z(float) 欲取得生物群系的方块的位置的Z轴  
> x, y, z这三个参数也可以使用某些触发动作自带的position来定义

**例子**:

    带注释的代码:
    world.sendMessageToAllPlayers(world.getBiome(-226, 4, 70))
    {先获取-226, 4, 70这个位置的生物群系，在显示在所有玩家的聊天栏}

    不带注释的代码:
    world.sendMessageToAllPlayers(world.getBiome(-226, 4, 70))
    效果:
    {因为-226, 4, 70这个位置是草原，也就是Plains}
{关于生物群系的对应文本，可以在[这里]找到}

[这里]:https://minecraft-zh.gamepedia.com/%E7%94%9F%E7%89%A9%E7%BE%A4%E7%B3%BB#.E7.94.9F.E7.89.A9.E7.BE.A4.E7.B3.BBID
![](https://miao.su/images/2017/08/11/TIM20170811113255fec1b.png)

###getBlockLightLevel (获取方块的光照等级)
**调用方式**:world.getBlockLightLevel(x, y, z)   
**返回值类型**:int(指定方块的光照等级)
> 获取指定位置的方块的光照等级并返回

**参数规则**:

x(int) 欲取得光照等级的方块的位置的X轴   
y(int) 欲取得光照等级的方块的位置的Y轴  
z(int) 欲取得光照等级的方块的位置的Z轴  
> x, y, z这三个参数也可以使用某些触发动作自带的position来定义

**例子**:

    带注释的代码:
    var 光照 = world.getBlockLightLevel(-232, 4, 170)
    {先初始化“光照”这个变量，并同时给“光照赋值”为取-232, 4, 170的光照等级的结果}
    world.sendMessageToAllPlayers(光照)
    {把获取到的光照等级转换为字符串显示在所有玩家的聊天栏}

    不带注释的代码:
    var 光照 = world.getBlockLightLevel(-232, 4, 170)
    world.sendMessageToAllPlayers(光照.toString())
    效果:

![](https://miao.su/images/2017/08/11/TIM20170811114209fe48d.png)


