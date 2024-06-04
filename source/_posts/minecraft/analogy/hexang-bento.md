---
title: 举二反三深入模组开发 息壤 + 午餐盒 = ?
date: 2020-04-13 11:19:59
categories: [minecraft, analogy]
tags:
    - Minecraft
    - 模组开发
---

> 本节将带领你通过息壤和午餐盒的代码实现来制作一把可变换贴图万能工具
>
> 知识速览:
>
> - ToolHandler.isRightMaterial
> - ItemMeshDefinition

------

### 列一元二次方程 - 神秘工匠的息壤实现

息壤是在神秘工匠灵道篇目颇为冷门的物品之一, 这里先简单介绍一下, 他会在你使用错误的觉醒灵宝系列工具挖方块时自动替换成背包中正确的工具, 引用mcmod里面举的例子大概就是这样的: 一旦你试图使用觉醒灵宝镐挖土，而背包中有觉醒灵宝铲时会自动替换两工具的位置，以便让你更快挖掘脚下泥土。

站在开发的角度我们可以理解, 息壤的实现主要有两部分, 一是**判定交互的方块应该用何种工具挖掘**, 二是**在物品栏寻找对应的工具替换手中的错误工具**, 我们先抛开第二个问题不谈, 来看看如何实现第一个部分, 我们可以在TT的开源项目关于息壤的代码中找到如下的部分

```
if (block != null) {
	Material mat = block.getMaterial();
	if (ToolHandler.isRightMaterial(mat, ToolHandler.materialsPick))
		typeToFind = "pick";
	else if (ToolHandler.isRightMaterial(mat, ToolHandler.materialsShovel))
		typeToFind = "shovel";
	else if (ToolHandler.isRightMaterial(mat, ToolHandler.materialsAxe))
		typeToFind = "axe";
}
神秘工匠开源地址: https://github.com/Thaumic-Tinkerer/ThaumicTinkerer/
节选自(1.7.10分支): /common/item/kami/ItemProtoclay.java
```

不难看出TT的作者自己实现了一个实用的方法**"isRightMaterial"**, 来判断方块A是否应该用工具B来采掘, 继续深入, 我们来看看**ToolHandler.isRightMaterial()**方法的相关细节

```
public static Material[] materialsPick = new Material[]{ Material.rock, Material.iron, Material.ice, Material.glass, Material.piston, Material.anvil };

public static Material[] materialsShovel = new Material[]{ Material.grass, Material.ground, Material.sand, Material.snow, Material.craftedSnow, Material.clay };
	
public static Material[] materialsAxe = new Material[]{ Material.coral, Material.leaves, Material.plants, Material.wood };
	
public static boolean isRightMaterial(Material material, Material[] materialsListing) {
	for (Material mat : materialsListing) {
		if (material == mat)
		return true;
	}
	return false;
}
神秘工匠开源地址: https://github.com/Thaumic-Tinkerer/ThaumicTinkerer/
节选自(1.7.10分支): /common/item/kami/tool/ToolHandler.java
```

看到这里, 相信读者已经略知一二了, 既isRightMaterial实则遍历了一遍**你提供的Material[]**, 如果你提供的Material是预先填充好的Material[]中的成员, 则返回了true. 如果作为读者的你觉得这有点暴力, 那不妨去看看ItemPickaxe等对应工具类的源码(雾), 这里就不展示了

------

### 列一元二次方程 - 生活调味料的午餐盒实现

接下来我们看看生活调味料中难得可见的两个物品中的其中一个 —— 午餐盒. 午餐盒的投食逻辑实现这里就不多说了, 这并不是我们要讨论的重点, 我们着重来看一看午餐盒的贴图(Models)的变化, 如果读者使用过午餐盒的话大概会有印象, 午餐盒可以打开和关上, 如果里面存有食物的话会有果蔬的贴图, 像下面这样: 

![](https://i.loli.net/2020/04/06/MokSeIBa3Qgq5Ks.png)

关于贴图更换的逻辑, 我们不难猜想, 实际更换贴图的思路伪代码应该是这样的

```
if (找找NBT里面午餐盒是开着的吗) {
	if (那午餐盒里面有东西吗) {
		渲染带马铃薯面包苹果的贴图()
	} else {
		渲染开着的空盒子的贴图()
	}
} else {
	渲染关上的贴图()
}
```

思而不学则殆, 下面我们来看看午餐盒改变贴图的代码

```
@SideOnly(Side.CLIENT)
public void registerModels()
{
    final ModelResourceLocation closed = new ModelResourceLocation(getRegistryName(), "inventory");
    final ModelResourceLocation openEmpty = new ModelResourceLocation(getRegistryName() + "_open_empty", "inventory");
    final ModelResourceLocation openFull = new ModelResourceLocation(getRegistryName() + "_open_full", "inventory");

    ModelLoader.registerItemVariants(this, closed, openEmpty, openFull);

    ModelLoader.setCustomMeshDefinition(this, new ItemMeshDefinition()
    {
        @Override
        public ModelResourceLocation getModelLocation(ItemStack itemStack)
        {
            if (isOpen(itemStack))
            {
                return isEmpty(itemStack) ? openEmpty : openFull;
            }
            return closed;
        }
    });
}
生活调味料开源地址: https://github.com/squeek502/SpiceOfLife/
节选自(1.12分支): /items/ItemFoodContainer.java
```

前三行代码很普通的声明了三个ModelResourceLocation, 其地址很普通的对应了三个Model的json文件, 这与创建三个物品为他们分配贴图的时候的ModelResourceLocation没什么大区别, 读者像普通给物品放贴图一样的方式来声明就可以了

接下来, 我们要让Minecraft知道, 我们准备要给物品使用的贴图分别都有什么, 以便于让Minecraft事先为我们准备, 我们需要用ModelLoader.registerItemVariants(Item, ModelResourceLocation**s......**)来提供, 我们可以看到上文代码中把刚刚声明的三个资源位置传了进去

最后, 我们要给物品添加贴图, 为了让Minecraft根据我们的需要自由的变换材质, 我们应该给物品添加一个**CustomMeshDefinition**, 何为**MeshDefinition**呢, 我们引用Harbinger中的定义: 

> `ItemMeshDefinition` 是一个原版类，允许某个物品只根据 `ItemStack` 提供的信息返回不同的 `ModelResourceLocation`。一个 `ItemStack` 里有什么？物品类型、数量、metadata（或损害值）甚至是 NBT 数据。理论上应该足够满足各种奇怪的需求了。
>
> 地址: https://harbinger.covertdragon.team/chapter-11/baked/custom-mesh.html

相信读到这里, 读者已经明白使用CustomMeshDefinition的意图了, 它可以直接访问ItemStack对象, 符合我们此时根据午餐盒这个ItemStack是否装满的状态来决定贴图的需求, 其后对ItemMeshDefinition接口的实现的具体代码结合我们上文的伪代码来看, 含义也就不言而喻了

------

### 联立一元二次方程组求解 - 动态渲染的"万能工具"

#### 造物品

读者不妨想一想结合上文两个例子的分析我们能开发出什么来...........

我认为你可能已经猜出来答案了, 因为他就写在标题上(雾), 所以我们要做万能工具, 超凡脱俗的万能工具, 举世闻名的万能工具, 是会动的长方体......哦扯远了......

传统的万能工具只是一个贴图, 我们这次通过ItemMeshDefinition来让贴图随着挖掘方块的改变而改变, 而决定应该用什么工具的方式, 就是使用ToolHandler.isRightMaterial()类似的用法了, 我们可以直接套用原版工具的材质, 这样~~不花一分钱~~就能搞到好看的贴图了

俗话说万事开头难, 但写一个工具的开头还是很简单的, 我们可以简单创建一个继承自ItemTool的工具并且使其包含所有方块, 不过这个解决方案不够浪漫, 所以我们直接继承自ItemPickaxe, 然后为其添加斧和铲的支持, 对于原版来说, 我们直接允许我们的万能工具挖掘所有的方块都用efficiency的速度就万事大吉了

```
public class UniversalIronTool extends ItemPickaxe {
    public UniversalIronTool() {
        super(ToolMaterial.IRON);
        this.setCreativeTab(CreativeTabs.TOOLS);
        this.setRegistryName("universal_irontool");
        this.setTranslationKey("forgedev.universalIronTool");
    }

    @Override
    public float getDestroySpeed(ItemStack stack, IBlockState state) {
        return this.efficiency;
    }
}
```

之后我们要监听注册事件并且调用registry来注册这个物品, 笔者这里就不多赘述了, 不要忘记就可以了, 如果此时进入游戏我们已经能得到一个紫黑块的"万能工具了"

#### 填材质

之后我们就要着手给我们的工具添加材质了, 也是本章的重中之重, 首先我们一如既往的监听**ModelRegistryEvent**

```
@SubscribeEvent
    @SideOnly(Side.CLIENT)
    public static void onModelRegistry(ModelRegistryEvent event) {
    
    }
```

有了午餐盒的经验, 我们已经对自定义模型的添加有了初步认识, 我们这里先小小的复习一下: 首先我们应该告诉Minecraft去准备我们需要的模型. 像下面这样, 我们声明需要的模型, 并且拜托Minecraft先为我们加载着

```
@SubscribeEvent
    @SideOnly(Side.CLIENT)
    public static void onModelRegistry(ModelRegistryEvent event) {
    	//我们这里使用了原版的木棍, 铁镐, 铁斧, 铁铲的材质, 读者也可以更换
    	final ModelResourceLocation stick = new ModelResourceLocation("stick");
        final ModelResourceLocation pickaxe = new ModelResourceLocation("iron_pickaxe");
        final ModelResourceLocation axe = new ModelResourceLocation("iron_axe");
        final ModelResourceLocation shovel = new ModelResourceLocation("iron_shovel");
        //这里universalIronTool是刚刚创建的继承自ItemPickaxe类的实例
        ModelLoader.registerItemVariants(universalIronTool, pickaxe, axe, shovel, stick);
    }

```

之后我们就要基于ItemStack能搞到的所有信息来为我们选择正确的材质了, 毫无疑问我们要基于ItemStack里面存储的NBT来判断, 所以我们的**初步想法**是: 

- 步骤一、在玩家开始挖掘方块的一刹那把对应工具类型的字符串**存到ItemStack的NBT**里面
- 步骤二、在自定义渲染模型中再次读取NBT, 根据**字符串的内容**决定该返回**什么工具的ModelResourceLocation**

##### 践行想法步骤二(午餐盒的应用)

有了想法我们就可以开始实践了, 我们先把渲染趁热打铁写完, 我们在上文的代码结束处追加

```
//这里universalIronTool是刚刚创建的继承自ItemPickaxe类的实例
ModelLoader.setCustomMeshDefinition(universalIronTool, itemStack -> {
    if (itemStack.hasTagCompound()){
        NBTTagCompound nbtTag = itemStack.getTagCompound();
        switch (nbtTag.getString("toolType")) { //我们打算往NBT里面存一个名为"toolType"的字符串值
            case "pickaxe":return pickaxe; //如果这个值为pickaxe, 则返回铁镐的ModelResourceLocation
            case "axe":return axe;
            case "shovel":return shovel;
            default:return stick; //如果值为none或者什么其他的不合法的值, 则让我们的万能工具变成木棍的样子
        }
    }
    return stick; //如果干脆物品就没有NBT, 那也是渲染木棍的贴图
});
```

现在Minecraft已经会根据NBT的内容不同来为我们正确挑选材质了, 不过接下来也是重点, 我们要完成想法的步骤一(不要问我为什么先二后一)

##### 践行想法步骤一(息壤的应用)

玩家挖掘方块的一刹那, 相信读者头脑中自然就产生了监听**PlayerInteractEvent**事件的念头, 那么恭喜你, 猜对了, 我们正是要监听玩家与方块交互的这个事件, 我们创建EventHandler类并监听它

```
@Mod.EventBusSubscriber(modid = "forgedev")
public class EventHandler {
    @SubscribeEvent
    public static void onInteract(PlayerInteractEvent event) {
        if (event.getSide().isClient()) return;
        if (event.getHand().equals(EnumHand.OFF_HAND)) return;
        if (event.getItemStack().getItem() instanceof UniversalIronTool) {
            Material curMaterial = event.getWorld().getBlockState(event.getPos()).getMaterial();
            NBTTagCompound nbtTag = event.getItemStack().hasTagCompound() ? event.getItemStack().getTagCompound() : new NBTTagCompound();
            nbtTag.setString("toolType", ToolUtils.getRightTool(curMaterial));
            event.getItemStack().setTagCompound(nbtTag);
        }
    }
}

```

相信代码的含义读者也一目了然, 我们给ItemStack创建了一个NBT, 并且调用**ToolUtils.getRightTool()**方法来获取指定方块应该用什么工具来采掘..........Wait a second.......别着急, 我们现在来创建ToolUtils类

```
public class ToolUtils {
    private static Material[] materialsPickaxe = new Material[]{ Material.ROCK, Material.IRON, Material.ICE, Material.GLASS, Material.PISTON, Material.ANVIL };
    private static Material[] materialsShovel = new Material[]{ Material.GRASS, Material.GROUND, Material.SAND, Material.SNOW, Material.CRAFTED_SNOW, Material.CLAY };
    private static Material[] materialsAxe = new Material[]{ Material.CORAL, Material.LEAVES, Material.PLANTS, Material.WOOD };

    public static String getRightTool(Material material) {
        for (Material mat : materialsAxe) {
            if (material == mat) return "axe";
        }
        for (Material mat : materialsPickaxe) {
            if (material == mat) return "pickaxe";
        }
        for (Material mat : materialsShovel) {
            if (material == mat) return "shovel";
        }
        return "none";
    }
}
```

类似于神秘工匠中息壤的写法, 我们也创建了三个对应工具应挖掘的Material的集合, 并且在getRightTool中遍历集合, 找到其正确对应的集合并且返回"axe", "pickaxe"或者是"shovel", 如果该material不属于任何工具将会返回"none"

#### 看效果

至此, 我们已经创建了我们会动的万能工具了, 你可以检查一遍自己是否已经监听了所有应该监听的事件, 注册了所有应该注册的事件处理器, 之后我们就可以进入游戏查看效果了, 不出意外的话你会得到下面这样拔群的效果

![](https://ftp.bmp.ovh/imgs/2020/04/0f18c95f3ec36699.gif)

------

### 小结

本节内容的小结: 

1.  了解了神秘工匠息壤工具决策的内部逻辑
2. 明白了如何使用**MeshDefinition**来自定义更改物品模型
3.  在1和2的基础上举"万能工具"例子, 使读者对1, 2有**更深了解**
4.  明确了**举二反三深入模组开发**的基本形式 

课后小思考:

- 我们能否通过多态和封装让我们的万能工具直接支持原版所有的材料呢?
- 思考能否通过**MeshDefinition**来**动态生成**材质呢

基于本文的实例制作的Mod: https://www.mcbbs.net/thread-1010246-1-1.html (其源码中实现了课后思考1, 读者感兴趣可以阅读)