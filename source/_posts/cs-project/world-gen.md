---
title: 给CustomStuff3方块添加世界生成
date: 2017-11-10 10:46:56
categories: [cs-project]
tags:
    - Minecraft
    - 模组开发
    - CustomStuff3
---

![635589679383149678.png](https://s2.loli.net/2023/01/10/k8xFHuVPMw9pe14.png)

-
 <font size=3 face="微软雅黑">
 我们打开我们工程的主菜单，选择World Generators添加一个世界生成，然后点击New填写信息，我这里就以我们制作的魔法矿石为例，Name可以随便填写，最好不使用中文字符

    Type 要生成的类型
    ore 矿物类型
    flower 花的类型
    因为我们要创建矿物的世界生成，所以我们选择ore

![](https://miao.su/images/2017/08/02/TIM20170802130129e75f2.png)

然后点击Create创建，创建完成后，我们点击我们刚刚创建的项目，点击Edit来编辑属性

###allowedBiomes
>矿物允许生成的生物群系，默认是all，也就是全部，可以在这里添加多个生物群系，这样就只会在指定的生物群系生成，这里是生物群系对应的意思，可以在MinecraftWiki的[这页]找到，内容获取至MinecraftWIKI,可能有多出的部分，请自行查看

    all 全部生物群系
    Ocean	海洋
	Plains	草原
	Desert	沙漠
	Extreme Hills	峭壁
	Forest	森林
	Taiga	针叶林
	Swampland	沼泽
	River	河流
	Hell	地狱（下界）
	The End	末路之地（空岛）
	FrozenOcean	冻洋
	FrozenRiver	冻河
	Ice Plains	冰原
	Ice Mountains	雪山
	MushroomIsland	蘑菇岛
	MushroomIslandShore	蘑菇岛岸
	Beach	沙滩
	DesertHills	沙漠山丘
	ForestHills	森林山丘
	TaigaHills	针叶林山丘
	Extreme Hills Edge	悬崖
	Jungle	丛林
	JungleHills	丛林山丘
	JungleEdge	丛林边缘
	Deep Ocean	深海
	Stone Beach	石滩
	Cold Beach	寒冷沙滩
	Birch Forest	桦木森林
	Birch Forest Hills	桦木森林山丘
	Roofed Forest	黑森林
	Cold Taiga	冷针叶林
	Cold Taiga Hills	冷针叶林山丘
	Mega Taiga	大型针叶林
	Mega Taiga Hills	大型针叶林山丘
	Extreme Hills+	峭壁+
	Savanna	热带草原
	Savanna Plateau	热带高原
	Mesa	平顶山
	Mesa Plateau F	平顶山高原 F
	Mesa Plateau	平顶山高原
	Sunflower Plains	向日葵草原
	Desert M	沙漠 M
	Extreme Hills M	峭壁 M
	Flower Forest	繁花森林
	Taiga M	针叶林 M
	Swampland M	沼泽 M
	Ice Plains Spikes	冰刺平原
	Jungle M	丛林 M
	JungleEdge M	丛林边缘 M
	Birch Forest M	桦木森林 M
	Birch Forest Hills M	桦木森林山丘 M
	Roofed Forest M	黑森林 M
	Cold Taiga M	冷针叶林 M
	N/A	N/A
	Mega Spruce Taiga	红木森林
	Redwood Taiga Hills M	红木山丘
	Extreme Hills+ M	峭壁+ M
	Savanna M	热带草原 M
	Savanna Plateau M	热带高原 M
	Mesa (Bryce)	平顶山（岩柱）
	Mesa Plateau F M	平顶山高原 F M
	Mesa Plateau M	平顶山高原 M

![](https://miao.su/images/2017/08/02/TIM201708021319395db3b.png)

###amount

>每次生成的矿脉大小，只有编辑了generatedBlock(一会说)才能被编辑，下方的格子中将会按照你填写的数值随机生成一些矿脉，以便于你参考

![](https://miao.su/images/2017/08/02/TIM20170802132138a5e0c.png)

###dimensions
>要生成这个方块的世界

    Overworld 主世界
    Nether 地狱
    End 末地

![](https://miao.su/images/2017/08/02/TIM20170802132318fad21.png)

###generatedBlock
>要生成的方块，点击中间的槽就可以修改，右键清除，只有选好了要生成的方块，才能更改矿脉的大小 

###generationsPerChunk
>一个区块允许出现几次生成的计算，也就是最多可以生成几撮矿石

###height
>矿石的允许生成高度

![](https://miao.su/images/2017/08/02/TIM20170802132635a10d1.png)

###replacedBlocks
>说这个之前我先说一下Minecraft的矿物生成机制：他会先正常生成地形，然后根据算法替换指定方块为特定矿石，一般不用修改。他会自动把主世界的石头，地狱的地狱岩，末地的末地石计算后替换为矿石，如果有特定需求，也可以修改

![](https://miao.su/images/2017/08/02/TIM201708021329324c3fa.png)

###原版参数
>这里稍微提一下与原版有关的参数，因为矿物生成本身有随机性，数据仅供参考

    煤矿石: amount:5~64
           dimensions:Overworld
           generatedBlock:<minecraft:coal_ore>
           generationsPerChunk:142.6
           height:1~255
    铁矿石: amount:4~10
           dimensions:Overworld
           generatedBlock:<minecraft:iron_ore>
           generationsPerChunk:77
           height:1~63
    金矿石: amount:1~16
           dimensions:Overworld
           generatedBlock:<minecraft:gold_ore>
           generationsPerChunk:8.2
           height:1~32
    钻石矿石: amount:3~8
           dimensions:Overworld
           generatedBlock:<minecraft:diamond_ore>
           generationsPerChunk:1
           height:1~16
    钻石矿石: amount:4~10
           dimensions:Nether
           generatedBlock:<minecraft:quartz_ore>
           generationsPerChunk:79
           height:7-117
    青金石矿石: amount:4~8
           dimensions:Overworld
           generatedBlock:<minecraft:lapis_ore>
           generationsPerChunk:3.43
           height:1~31
    红石矿石: amount:4~5
           dimensions:Overworld
           generatedBlock:<minecraft:redstone_ore>
           generationsPerChunk:24.8
           height:1~16

**<font size=5 face="微软雅黑">给方块添加自然生成</font>**
-

自然生成我这里就不细说了，实际上和矿物生成一样，如果有什么问题可以联系我，就是有一个blockRate不一样

###blockRate
> 每次生成的区域方块的数量，也就是一撮方块的大小

----------
自然生成这部分也完成了，下一部分是其他的获取方式，今天也会更新出来，希望能顺利进行，第二章的内容就开始麻烦起来了，我的更新速度也不会像现在这么快了，因为我要自己去测试功能，好了就说这么多，下章见