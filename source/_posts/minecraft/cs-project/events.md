---
title: CustomStuff3事件概述
date: 2017-11-30 10:46:56
categories: [minecraft, cs-project]
tags:
    - Minecraft
    - 模组开发
    - CustomStuff3
---

![635589679383149678.png](https://s2.loli.net/2023/01/10/k8xFHuVPMw9pe14.png)

**<font size=6 face="微软雅黑">简单事件</font>**
-
**<font size=5 face="微软雅黑">序言</font>**
-
 <font size=3 face="微软雅黑">
 这章我来介绍一下CS3事件部分的基础内容，比较高级的实例及全部API的说明会放在下一章进行

**<font size=5 face="微软雅黑">添加事件</font>**
-
 你可以进入你创建的方块/物品的编辑界面，你可以看到一些on开头的属性，在这里，我把它叫做触发动作(我自己的命名)

    所有的触发动作所对应的意思:
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

这些就是全部的触发动作，我们可以在触发动作里面编写一些代码，这些代码叫做这个触发动作的事件，那么事件都可以写些什么呢，我会在下一章的API帮助文档中详细介绍到，这也是CS3的重要组成部分

那么我们如何给一个触发动作添加我们想要的事件呢？我们只需点击对应的触发事件的按键就可以了(如果你找不到，你可以去方块/物品里面看看，有一些触发动作是方块/物品独有的)，比如说这里我们点击**onRightClick**，也就是右击的时候，如图

![](https://miao.su/images/2017/08/16/TIM20170816134415cea3c.png)

进入编辑界面，我们可以看到一个黑框，我们可以在这里编辑事件的代码，左下角是Save(保存), 右下角有Info(API新建)以及Cancel(不保存退出)，当我们编辑好了事件的代码，我们就可以直接保存了，如果有语法错误，CS3也会随即提示你，如果你有不知道怎么解决的问题，也可以直接在MCBBS或百度贴吧问我

![](https://miao.su/images/2017/08/16/TIM201708161347584c049.png)

你们可能很想知道，我里面输入的这句语句是什么意思，会产生什么样的效果，你们可以先照着我的输入一遍，我在下面会讲解这个语句的作用。当你输入完成后，点击Save，并拿出你创建的物品，点击鼠标右键

![](https://miao.su/images/2017/08/16/TIM20170816135024d5a67.png)

我们刚刚的语句就发生了作用，在你输入的过程中，要注意双引号，如果你看了前面的JavaScript那课，你可能对双引号并不陌生，对，双引号引起的部分叫做字符串(string)，也就是一段文本，当然你也可以把HelloWorld!改成任何中文/日文/英文等文字，如果你安装了Inputfix，就可以直接输入中文了，同样也支持颜色代码§的使用，具体的颜色可以前往第一章第二课查看

![](https://miao.su/images/2017/08/16/TIM201708161355594374c.png)

![](https://miao.su/images/2017/08/16/TIM20170816135611fe258.png)

    有些人可能想问了，这个语句到底是什么意思，他的官方解释是什么呢，你可以打开Info界面，点击player, 并且找到sendMessage条目，左键点击，你就可以找到你的答案
    什么！你不懂英语？
    不懂我来给你翻译下啦
    
![](https://miao.su/images/2017/08/16/TIM2017081614005091e98.png)
	
    这个命令很简单，就是给玩家发送一条信息，括号中要填写的是message(信息)
    你这时可能会想问，我们没学过message这个类型啊？看到下面
    - message (string) ······
    这里了吗，这里就详细说明了message需要填写一个字符串，而且后面还给予了提示，十分人性化，且易理解
    关于所有的语句的意思及使用方法，都会放在下一章来介绍


----------
你们学会了吗，这就是本章的全部内容，如果你们想知道更多语句，就给我金粒人气吧，这样我可能会更新的快一点233 谢谢大家的支持！

----------
顺便我在这里说一下下一章节可能会使用到的一些关于编程的语言

    我们以 player.sendMessage() 为例
    sendMessage是player的一个方法
    player.sendMessage()是一个函数，我习惯直接称作语句
    ()括号里面内容叫做参数
    比如player.sendMessage("你好")
    "你好"是player.sendMessage的一个参数
    有些语句可能有多个函数，中间使用英文逗号, 隔开
    比如world.sendMessageToPlayer("TROU2004", "你是TROU2004")
    这个语句就有2个参数
    第一个参数是"TROU2004"
    第二个参数是"你是TROU2004"
    这个语句需要2个参数才能正常使用
    有些语句不需要参数，但是()括号是不能省略的
    有些语句有返回值，有些没有
    比如 1+2
    他的返回值是3
    在下一章API文档中我也会具体告诉你们所有语句的返回值类型
    另外在JS中也可以使用小括号来更改运算顺序
    var a = 1+2*8
    这时变量a是整数型(int)，值是17
    var b = (1+2)*8
    这时变量a是整数型(int)，值是24

----------
大家看到这里辛苦了，递杯JAVA给你们 233
