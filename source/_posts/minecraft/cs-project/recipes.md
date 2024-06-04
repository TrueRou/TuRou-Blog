---
title: 给CustomStuff3物品添加配方
date: 2017-10-23 10:46:56
categories: [minecraft, cs-project]
tags:
    - Minecraft
    - 模组开发
    - CustomStuff3
---

![635589679383149678.png](https://s2.loli.net/2023/01/10/k8xFHuVPMw9pe14.png)

在上个篇章，我们详细的介绍了物品/方块的创建与编辑，这章，我们将讲解如何给一个方块或物品添加合成或冶炼的配方，这次的内容比较简单，在制作之前，我先把我们需要用到的物品制作出来，方便我们一会加入合成冶炼配方，我制作了一把魔法镐和一个魔法矿石。

![](https://miao.su/images/2017/08/02/TIM2017080208525497276.png)
![](https://miao.su/images/2017/08/02/TIM2017080208531242343.png)

 然后我给他们设置属性和贴图，方便查看辨认，工具的贴图也放在items才能识别

![](https://miao.su/images/2017/08/02/TIM20170802090259c0ba1.png)
![](https://miao.su/images/2017/08/02/TIM20170802092837521c1.png)
![](https://miao.su/images/2017/08/02/TIM2017080209421693824.png)

**<font size=5 face="微软雅黑">给这个物品添加合成配方</font>**
-
 <font size=3 face="微软雅黑">
 一个正常的MOD是必须要有合成配方的，CS3的MOD工程也是一样，所以现在我们来学习一下如何添加合成配方

 我们先回到CS3主界面，选择下方的ShapedRecipes，也就是创建有序合成，然后点击New来新建一个合成

![](https://miao.su/images/2017/08/02/TIM20170802111031a2134.png)

 点击空白部分选择物品，右边的两个箭头增减产物的物品，下面的Width和Height是当前合成表的长宽，没什么用，点击空白部分后

 ![](https://miao.su/images/2017/08/02/TIM201708021113131de1b.png)

  <font size=3 face="微软雅黑">
 可以在Items里面选择普通的物品，两个物品交换说明这两个物品在合成时是可以互相交换的，部分工具的耐久条会变化，这样的物品在合成时会忽略掉耐久值，在这里可以找到原版、CS3工程和其他MOD的方块与物品，如果你无法找到对应的物品，你可以去OreClasses里面找找，选择Ore Classes会匹配物品的矿物词典，你自己创建的矿物词典也会显示在这里，，如果你无法编辑这个页面，请确定你使用CS3的版本是不是0.7.9，如果你使用的0.7.10，请回退版本，具体请查看索引页，如果想要更多关于合成的更改与设定，可以使用MineTweaker与这个MOD联动，你可以在[这里]找到MineTweaker的教程，注意这个页面会自动播放音乐
 [这里]:http://www.mcbbs.net/thread-304800-1-3.html

 这是我创建的合成

 ![](https://miao.su/images/2017/08/02/TIM20170802112219e582b.png)

 ![](https://miao.su/images/2017/08/02/TIM2017080211271006876.png)

 创建好的合成会在这里直观的显示出来

 ![](https://miao.su/images/2017/08/02/TIM20170802112536f3f2d.png)

 ShapelessRecipe(无序合成)与有序合成的设置方法相似，这里就不再次叙述了，在这里设置的合成可以不用按照顺序来摆放

**<font size=5 face="微软雅黑">给这个物品添加冶炼配方</font>**
-
 <font size=3 face="微软雅黑">

 冶炼配方的添加也十分简单，我们先回到以你MOD命名的主菜单，选择Smelting Recipes来创建冶炼合成，点击空白来编辑物品，是与合成一样的，RecipeList目前无法更改，不需要更改，保持Vanilla就好了

 ![](https://miao.su/images/2017/08/02/TIM201708021136205720d.png)
 
 <font size=3 face="微软雅黑">
 既然我们说道冶炼，那么我们顺便也介绍下如何添加一个燃料吧，我先添加一个新的物品“魔法燃料”

 ![](https://miao.su/images/2017/08/02/TIM20170802114913f0576.png)

 然后我们回到以你MOD命名的主菜单，选择Fuels

    Duration 可以提供的烧制时间
    单位为tick，一秒有20tick
    木棍 100
    木板 300
    煤炭 1600

 因为要做出是煤炭的23倍，所以1600*23=36800

 ![](https://miao.su/images/2017/08/02/TIM2017080211552406ee6.png)

 ![](https://miao.su/images/2017/08/02/TIM20170802115708c6974.png)
 ![](https://miao.su/images/2017/08/02/TIM20170802115729b17ff.png)
