---
title: 利用CustomStuff3创建第一个方块/物品
date: 2017-09-12 10:46:56
categories: [cs-project]
tags:
    - Minecraft
    - 模组开发
    - CustomStuff3
---

![635589679383149678.png](https://s2.loli.net/2023/01/10/k8xFHuVPMw9pe14.png)

我们创建好了CS3工程，便迫不及待的想要创建我们的第一个物品，我们点击我们刚刚创建的工程，会打开一个界面，在这里选择要创建的东西，比如我们现在想要创建一个物品，那么我们就可以点击 Items 进入创建物品的界面，因为我们要新建一个物品，所以我们点击New来进入创建物品的界面

![](https://miao.su/images/2017/07/31/TIM20170731171439cc521.png)

    Name 在这里输入要创建物品的名字
    不要输入中文、不要输入中文、不要输入中文
    这里输入的名字可以为大小写字母和数字，不会显示出来
    只是内部识别(/give时候用的)
    
    Type 类型 这里指你要创建物品的类型，我们这里选择Normal
    我也顺便介绍一下其他的类型，在你选择特定类型的时候，CS3会自动帮你调整有关数据

    Normal 普通类型 
    axe 斧头类型
    boots 靴子类型
    bucket 桶类型
    food 食物类型
    helmat 头盔类型
    legs 护腿类型
    pickaxe 镐类型
    plate 胸甲类型
    shovel 铲类型
    sword 剑类型

 <font size=3 face="微软雅黑">
> 另外关于护甲值相关问题，将会在以后说道    
> 在这其中有些类型有特定的内容，我会在遇到的时候说道

 
 选择好后，我们就可以点击Create来创建我们的第一个物品了

![](https://miao.su/images/2017/07/31/TIM2017073119412368e1d.png)

 <font size=3 face="微软雅黑">

 创建好以后，你会发现Edit按钮是黑色的，我们无法对这个物品进行编辑，因为你需要重启游戏，重启过后，我们就可以点击“Edit”进入物品的编辑界面了

![](https://miao.su/images/2017/08/01/TIM2017080107374243e37.png)

 然后你就可以对这个物品进行编辑了，我会在这里介绍到所有可供调整的内容，在介绍每一个内容时都会使用图片来举一个例子，以便大家理解

### anvilMaterial
> 在铁砧被修复时需要用到的材料；如果你制作的东西是个工具或者有耐久值的东西，可以编辑此项，点击中间的槽就可以选择物品了，右键点击可以清除物品，选择完成后点击Edit来保存    

![](https://miao.su/images/2017/08/01/TIM20170801074124c390d.png)
![](https://miao.su/images/2017/08/01/TIM2017080107442705c2d.png)

### containerItem
> 对不起，关于这项我无法理解是什么意思，经过一段时间的测试也没有发现设置containerItem有什么作用，我去google了一下也没有找到相关信息，如果你知道，可以联系我补全这段信息

### creativeTab
> 给你的物品选择一个创造模式物品栏，这样他就可以显示在创造模式中了，你也可以自己创建一个创造模式物品栏并把你的物品放进去，这会在以后说道，选择完成后点击Edit来保存，这是每项对应的内容

    buildingBlocks 建筑方块
    decorations 装饰性方块
    redstone 红石
    transportation 交通运输
    misc 杂项
    search 搜索物品 (他会直接显示在里面，你可以搜索到，但是你放在其他的项目里面也可以搜索到)
    food 食物
    tools 工具
    combat 战斗用品
    brewing 酿造
    materials 材料
    inventory 生存模式物品栏 (请不要放置在这里，它会出现问题，无法正常使用)
    
![](https://miao.su/images/2017/08/01/TIM20170801081436c39b5.png)
![](https://miao.su/images/2017/08/01/TIM201708010814574f798.png)

### damage
> 这个比较简单，就是这个物品的攻击伤害，最大值为2147483647，如果你输入的数字大于这个数字，会自动降低为这个数值

![](https://miao.su/images/2017/08/01/TIM2017080108543661f27.png)
![](https://miao.su/images/2017/08/01/TIM201708010830573b881.png)

### displayName
> 物品的显示名称，默认为Unnamed，为空的话也会变成Unnamed，可以填写Windows支持的任何文字(中文日文英文大小写阿拉伯数字罗马数字)，不过如果你要输入中文请安装 [中文修复补丁(Inputfix)] (这个是1.7.10的版本，如果你需要其他版本，请自行下载)如果你不安装，在外面输入好在复制进来也是可以的
[中文修复补丁(Inputfix)]:https://github.com/zlainsama/InputFix/releases/download/1.7.10-v5/InputFix-1.7.10-v5.jar

![](https://miao.su/images/2017/08/01/TIM20170801085836960bc.png)
![](https://miao.su/images/2017/08/01/TIM201708010837342a3c0.png)

### efficiency
> 挖掘速度，只在类型(上文有介绍)为工具类(比如axe pickaxe等等)的时候有效果，要注意的是：就算你把速度调成100，如果采掘等级(下面会说)没有达到钻石级别，挖黑曜石也是不会掉落的，这里使用gif来演示

efficiency设置1.0的速度

![](https://miao.su/images/2017/08/01/bandicam2017-08-0109-03-19-765_14a503.gif)

efficiency设置10.0的速度

![](https://miao.su/images/2017/08/01/bandicam2017-08-0109-03-45-679308d2.gif)

### enchantability
> 附魔能力：这个就比较复杂了，这个跟原版的附魔机制有关，首先我先上公式

    修改后的附魔等级 = 附魔等级 + 随机值(0, 附魔能力 / 4 * 2) + 1
    （每步计算完毕后四舍五入）
[这个页面]:https://minecraft-zh.gamepedia.com/%E9%99%84%E9%AD%94%E6%9C%BA%E5%88%B6#.E7.AC.AC.E4.B8.80.E6.AD.A5_-_.E5.AF.B9.E9.99.84.E9.AD.94.E7.AD.89.E7.BA.A7.E5.8A.A0.E5.85.A5.E8.B0.83.E8.8A.82.E5.80.BC
> 如果你需要的话，可以在Minecraft中文Wiki的[这个页面]找到
> 这里有一个表格可供参考

![](https://miao.su/images/2017/08/01/TIM2017080109262790e35.png)

### full3D
> 让你的物品可以在手中3D显示出来，具体效果可以在游戏中查看，工具会自动开启full3D效果

> Draw Full 3D:是否使用Full3D的效果

没有开启full3D的显示效果

![](https://miao.su/images/2017/08/01/TIM20170801093106d8c40.png)

开启了full3D的显示效果

![](https://miao.su/images/2017/08/01/TIM20170801093117cc95b.png)

### hasEffect
> 是否开启光效，光效就像附魔的物品那样，会一直闪烁，下面上图，可能看的不是很清楚，可以自己测试一下
> 
> Has Effect:是否开启光效

关闭

![](https://miao.su/images/2017/08/01/TIM20170801093428577e7.png)

开启

![](https://miao.su/images/2017/08/01/TIM20170801093441ca794.png)

[这个网站]:http://www.aigei.com/bgremover/
### icon
> 虽然比较麻烦，但是很重要，给你的物品添加图标
> 
> 在开始之前，首先你要准备一张正方形透明png图片,大小可以是16x16、32x32、64x64、128x128等等，但不建议太大，以免影响显示效果，而且必须背景透明，个人推荐使用paint.net来处理图片，也可以使用Photoshop等软件，但是Windows自带画图不可以，因为它不支持alpha(透明)通道，如果你发现你的材质在游戏中显示背景是白色的，那么就说明你的图片不是透明的，简单的图片你可以在[这个网站]进行转换
> 
> 我不会画画，所以就随便画了一个贴图
>
![](https://miao.su/images/2017/08/01/b01db672c8a3dbf25edd7.png)
>
>然后我们给他命名，可以为大小写字母和数字，中文也可以，不过这里用什么名字是不会影响到在游戏中的查看的，我这里就命名为了“魔法锭.png”
>
>然后我们把这个文件放入.minecraft/mods/你的工程ID/assets/你的工程ID/textures/items（因为我们要给物品加贴图，所以放到items里面）
>
>![](https://miao.su/images/2017/08/01/TIM2017080109532713839.png)
> 
> 然后我们进入icon，点击这里的“...”
> ![](https://miao.su/images/2017/08/01/TIM20170801101220e7e77.png)
> 
> 然后我们就能在“Pack”里面找到自己的材质，并设置了
> ![](https://miao.su/images/2017/08/01/TIM20170801101335a90ba.png)
> 
> 左键点击要设置的材质，然后点击Select就可以了
> 
> 最后效果(我开了“hasEffect”)
> 
> ![](https://miao.su/images/2017/08/01/TIM2017080110143864d04.png)

### information
> 给你的物品添加一个描述(可以使用任何文字，空格，还有颜色符号§，如果你不会打这个符号，可以直接复制到编辑框里面)鼠标指向左下角的你的物品图标可以预览现在的设定

    §4 红色              
    §c 粉红色
    §6 土黄色
    §e 黄色
    §2 绿色
    §a 亮绿色
    §b 靛蓝色
    §3 青色
    §1 蓝色
    §9 亮蓝色
    §d 粉色
    §5 紫色
    §f 白色
    §7 灰色
    §8 深灰色
    §0 黑色

![](https://miao.su/images/2017/08/01/TIM20170801102717d7059.png)

### maxDamage
> 最大耐久值，这个我就不多说了，自己测试下就可以了，默认值100

### maxStack
> 最大堆叠数，默认是64，如果你创建的类型是工具类的，默认是1

![](https://miao.su/images/2017/08/01/TIM2017080110301175ffd.png)

![](https://miao.su/images/2017/08/01/TIM20170801103020378f1.png)

### maxUsingDuration
> 这项也没有找到相关信息，大致意思是“最大持续时间”如果你知道它确切的作用，可以与我联系，补全这个部分

### onBlockDestroyed
> 这部分比较复杂，主要涉及到关于CustomStuff3的事件，会在以后进行讲解，如果你点击这些会闪退，请检查你是否把js.jar放进.minecraft/mods文件夹内。我会暂时跳过这部分，以便统一讲解

### renderScale
> 这个物品拿在手中的显示大小，默认为1.0，如果调整太大可能会不显示(我试过了)下面上图

5.0的效果

![](https://miao.su/images/2017/08/01/TIM2017080110443803e52.png)

### toolClasses
> 主要用于工具类型的物品，可以在这里指定挖掘等级与可挖掘的方块
> 
> 如果你类型选择的镐，这里默认会出现pickaxe - 0，以此类推
> 
> 你可以选择删除，暂时删掉这个，并点击New建立一个新的设定

    Tool class(工具的类型)可以填写:
    pickaxe 挖掘石头，矿物等
    axe 挖掘木头
    shovel 挖掘泥土，沙子等
    all 挖掘全部方块(同时具有pickaxe, axe, shovel的属性)
    noHarvest 什么都挖不了(根本挖不动，像挖基岩一样)
    如果你想同时具有镐和铲的特性，可以用英文逗号来隔开，比如:
    pickaxe,shovel
    
    Harvest level(工具的挖掘等级)可以填写:
    0 木头等级
    1 石头等级
    2 铁等级
    3 钻石等级
    详细的可以挖哪些方块可以进入游戏测试，填写完毕就可以点击Create了
![](https://miao.su/images/2017/08/01/TIM20170801105343ebec8.png)

### usingAction
> 使用的特殊效果，可以自行测试，会有特殊的声音和特效，我没测试过

    none 无
    eat 吃东西
    drink 喝水
    block 放置方块
    bow 拉弓

----------
到这里物品部分就全部说完了，一共6000字，有点太啰嗦了，下面方块可能有与物品相似相同的条目，我就不会讲的这么详细了，如果有不懂的部分，可以翻上来看看，或者问我，如果我知道也会直接告诉你的，然后我说下下面方块的问题，方块种类一共23种，因为我非常敬业，所以我打算全部介绍，不知道会有多长，也没办法啦233

----------
**<font size=5 face="微软雅黑">创建一个方块</font>**
-
   <font size=3 face="微软雅黑">
   &ensp;&ensp;&ensp;&ensp;请做好心理准备，方块条目的方块类型特别多(一共22种)项也特别多，可能不会讲的特别详细，简单的就一笔带过了，不会放太多图片，而且内容可能会特别枯燥，我会全部介绍到，你们可以挑着看，以免丧失信心
   &ensp;&ensp;&ensp;&ensp;在我们CS3工程的界面里，选择Blocks，然后点击New新建一个方块，方块类型有很多种，我先介绍一下

    Normal 普通类型
    flat 平面类型(就是平铺在地上的一个方块)
    fluid 流体类型
    gravity 有重力类型(像沙子一样掉落)
    ladder 梯子类型
    pane 玻璃板
    post 木桩类型(比较粗的不会互相连接的栅栏)
    pressurePlate 压力板类型
    slab 台阶类型
    stairs 楼梯类型
    button 按钮类型
    torch 火把类型
    trapDoor 活板门类型
    wall 墙类型(像圆石墙那样的)
    carpet 地毯
    crossTexture 可以连接的材质
    crossTexturePost 可以连接的木桩
    door 门类型
    facing 贴面(?)
    fence 栅栏
    fenceGate 栅栏门
    CS3教程作者的吐槽：CS3作者你知道有个东西叫木匠方块吗

   &ensp;&ensp;&ensp;&ensp;那么我们就先创造一个normal方块，然后我来介绍方块的项

### blocksPiston
> 是否可以被活塞推动

### containerItem
> 如果有了解的请联系我

### creativeTab
> 创造栏，可以使用自己创建的或者原版的，与原版对应的文字可以去上面介绍物品的那里查看

### displayName
> 在物品栏显示名称，可以是任何文字

### drop
> 挖掘这个方块可以得到的物品
> 
> Amont的意思是数量，也就是挖掉后会掉落一个这个方块本身
>
>我们可以点击New新建一个掉落，并且可以点击Edit进行更改，Min是掉落的最小值，Max是掉落的最大值，CS3会取这个区间的一个值进行掉落，如果你填写相同的数值，将会100%掉落

### expDrop
> 挖掘这个方块能得到的经验
> 
> Min是最小值，指的是能获得经验的最小值,Max是最大值，指的是能获得经验的最大值，CS3会取这个区间的一个值进行掉落，如果你填写相同的数值，将会100%掉落，注意这个数字指的不是等级，实际掉落的多少你可以自行测试，因为MC每个等级所需要的经验是不同的

### fireSpreadSpeed
> 火的传播速度，默认为10，这个可以自行测试，不过感觉没什么用呢

### flammability
> 可燃性，就是可以燃烧的时间长短吧

### fortuneModifier 
> 幸运值，也就是时运附魔对方块挖掘的影响，要想使用需要打开界面底部的useFortune才可以，几率你们可以自己测试一下

### hardness 
> 方块的硬度，数值越大，挖掘的越慢，感觉黑曜石应该是8~10左右，正常采掘的话还是1左右比较合适

### harvesting 
>在这里设置挖掘应该用的工具和挖掘等级，如果挖掘等级设置0(木头)所有工具都可以采集，只是采集的快慢问题，关于工具和挖掘等级的设置可以在上面讲解物品时的toolClasses找到
>
>allow sick harvest 指的就是是否允许精准采集这个方块

### hasCollision 
> 是否开启方块的碰撞箱，如果关闭碰撞箱，就可以穿过这个方块
>
>重要：发现碰撞箱Bug：如果让头部穿过没有碰撞箱的方块，那么游戏会Crash，且无法进入存档；解决方法：通过编辑器等软件调整玩家的位置从方块里出来就可以了

### information
>给方块添加描述，可以使用颜色字符和所有文字空格等

### isBeaconBase
>是否作为信标的基座

### isBurning
>接触到是否会受到火焰伤害，但是我自己测试没有效果

### isWood
>是否是木头，如果打开了这个方块会阻止相邻树叶的消失

### light
>方块发光的强度

    萤石为15
    火把为12
    南瓜灯为15
    红石火把为7

### material
>材料，会影响到挖掘后的粒子效果和一些特殊的效果，更改后必须重启才能应用

    ground 地面(放置在地面上的正常方块)
    ice 冰
    vine 藤蔓
    craftedSnow 雪块
    wood 木头
    glass 玻璃
    redstoneLight 红石灯
    plants 植物
    cactus 仙人掌
    iron 铁块
    fire 火
    grass 草方块
    leaves 树叶
    sponge 海绵
    clay 粘土块
    pumpkin 南瓜
    water 水
    tnt TNT
    sand 沙子
    cloth 羊毛布料
    lava 岩浆
    circuits 红石电路
    rock 岩石
    snow 雪

### maxStack
>最大堆叠，最大是64

### on系列
>在介绍事件时介绍

### opacity
>方块能放置的最大高度，最大为255

### pick
>捡起的意思，不过我测试没什么用，不知道是什么问题

### placement
>放置位置

    Can place on floor 是否可以在地上放
    Can place on ceiling 是否可以再天花板上放
    Can place on wall 是否可以在墙上放

### resistance
>抗爆炸能力，自行测试数值，越高越抗爆炸

### slipperiness
>湿滑值，默认为0.6，冰的值可能会高一些

### stepSound
>发出的声音

    Play Place 播放放置这种方块的声音
    Play Break 播放破坏这种方块的声音
    Play Step 播放在上面行走的声音

    stone 石头
    metal 材料
    ladder 梯子
    wood 木头
    glass 玻璃
    sand 沙子
    cloth 羊毛布料
    anvil 铁砧
    grass 草方块
    gravel 砂砾
    
### textures
>因为上面也已经介绍过了物品的材质，这里会简单的介绍一下
>
>我们可以看到右边有四个紫黑方块，这里是你材质设置的及时预览(你设置在底部的材质在这里看不到，你可以设置好后在游戏中查看)

    bottom 底部材质
    top 顶部材质
    north 北边材质
    south 南边材质
    east 东边材质
    west 西边材质

    Transparent 透明材质(建议开启) 实际效果可以在预览看到，这个会让你在底部设置的透明材质得以显示
    Semi Transparent 半透明 透光度比全透明低一点，根据你的需求来打开火关闭这项
    Tile Transparent 边框显示 如果这项打开，两个透明方块的连接边的材质会显现出来，为了效果，建议关闭，如果有特殊需求可以打开

>必须使用png的图片格式，将你的png文件放置在.minecraft/mods/你的工程ID/assets/你的工程ID/textures/blocks，便会在游戏中显示出来

### tileEntity
> 也就是带数据的方块，这个内容比较复杂且不经常用到(挺经常的)，这个会在扩展篇(如果我研究明白的话)进行讲解

### useFortune
> 是否允许时运，这个可以与上文的fortuneModifier配合使用，具体数据请自行测试

----------
方块部分更新完了，不过还有很多其他的类型没有讲解，你们可以自行测试，听听名字大概就知道是什么东西了，主要是材质的区别，关于材质问题，因为我对材质特别没有信心，所以我就不演示(实际上是懒)了，如果对某种类型有疑问，可以联系我，如果我知道的话，会进行讲解

----------
这小节终于讲完了，我从7:00开始码字，一直码到晚上19点，这期间大部分时间一直在测试功能，有很多功能我介绍的比较牵强，其实我自己也没搞明白，这部分就交给脑补能力很大的观众们来自行补全了啦，就说到这里，然后我可能在这章完成后将本教程上传至MCBBS，希望大家多多支持，我也是第一次做教程！
