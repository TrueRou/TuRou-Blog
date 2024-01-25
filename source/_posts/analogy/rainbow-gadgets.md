---
title: 举二反三深入模组开发 彩虹桥法杖 + 建筑小助手 = ?
date: 2020-07-31 11:19:59
categories: [analogy]
tags:
    - Minecraft
    - 模组开发
    - 举二反三深入模组开发
---

> 本节将分析彩虹桥方块的动画效果, 建筑小助手是如何触及极远的区域的, 最后我们会造一个便利的建筑方块
>
> 知识速览:
>
> - ITickable TileEntity
> - BlockRayTraceResult

------

### 天之苍苍，其正色耶？

说起Botania的彩虹桥法杖, 相信读者并不陌生, 其美丽的特效出自浪漫的Vazkii之手, 观察彩虹桥方块我们可以发现, 其模型应该是一个不断变色的方块, 并且不时会散发出有植物气息的粒子效果, 彩虹桥方块会在其产生后一段时间内自动消失, 不难想象应该是实现了ITickable接口的TileEntity.

![](https://i.loli.net/2020/04/13/4Nu2kcxbGsMnEPz.png)

从彩虹桥方块中我们可以发现如下三个特性, 我们一个一个来看

- 透明方块的动态材质
- 在方块周围生成指定的粒子效果
- 过一段时间后自我消失

#### 透明方块的动态材质

关于动态材质相信读者都不会太陌生, 因为模组的模型加载说到底还是跟随了原版的机制, 所以我们固然可以想到动态资源包中常用的mcmeta格式以及瀑布般的长条材质, 其实Minecraft中重复简单的方块动画都可以用mcmeta配合一个包含动画过程关键帧的图片格式轻松实现, 其实彩虹桥方块的动画也是这样的, 在Botania的resources目录中我们可以找到bifrost.json, 这就是彩虹桥方块的模型

```
{
  "parent": "minecraft:block/cube_all",
  "textures": {
    "all": "botania:blocks/bifrost"
  }
}
Botania开源地址: https://github.com/Vazkii/Botania
节选自(1.12-final分支): /resources/assets/botania/models/block/bifrost.json
```

显然彩虹桥方块的模型与普通方块是相同的, 换句话说我们只需要关注动画材质就可以了, 根据Model里面声明的路径我们去找对应的贴图以及mcmeta文件.

![](https://i.loli.net/2020/04/13/9Tpkw1CDHErVZJe.png)

这里就不多说什么了, 如果我们也想要一个动态模型只需如图一样创建贴图和mcmeta标记文件就可以了

> 注: mcmeta的文件名应为贴图名+.mcmeta
>
> frametime: 动画播放的帧率
>
> interpolate: 是否开启差值补帧(使动画渐变更流畅)

之后我们来看透明的实现, 在Bifrost的方块声明中我们可以找到如下代码

```
public class BlockBifrostPerm extends BlockMod implements ILexiconable {

	public BlockBifrostPerm(String name) {
		super(Material.GLASS, name);
		setLightOpacity(0);
		setLightLevel(1F);
		setSoundType(SoundType.GLASS);
	}

	@Override
	public boolean isOpaqueCube(IBlockState state) {
		return false;
	}

	@Override
	public boolean isFullCube(IBlockState state) {
		return false;
	}

	@Override
	public boolean shouldSideBeRendered(IBlockState state, @Nonnull IBlockAccess world, @Nonnull BlockPos pos, EnumFacing side) {
		if (world.getBlockState(pos.offset(side)).getBlock() == this) {
			return false;
		}
		return super.shouldSideBeRendered(state, world, pos, side);
	}

	@SideOnly(Side.CLIENT)
	@Nonnull
	@Override
	public BlockRenderLayer getRenderLayer() {
		return BlockRenderLayer.TRANSLUCENT;
		
	}
}
Botania开源地址: https://github.com/Vazkii/Botania
节选自(1.12-final分支): /common/block/BlockBifrostPerm.java
```

显然我们的彩虹桥方块需要像玻璃一样可以透光, 在Minecraft1.15.2中我们可以让方块直接继承自AbstractGlassBlock从而直接得到类似玻璃的透光能力, 但在1.12.2我们还不能这样做, 于是我们可以覆盖isOpaqueCube isFullCube getRenderLayer等一系列方法, 使方块可以透光, 为了拥有更好的显示效果, Vazkii还覆盖了shouldSideBeRendered方法, 重叠的面将不会渲染, 这里笔者就不多说了, 读者可以自行阅读上文该方法的代码

#### 在方块周围生成指定的粒子效果

有心的读者如果去Github阅读了BlockBifrostPerm的源码可以发现我们列举的代码中缺少了如下部分

```
@Override
public void randomDisplayTick(IBlockState state, World world, BlockPos pos, Random rand) {
    if(rand.nextBoolean())
        Botania.proxy.sparkleFX(pos.getX() + Math.random(), pos.getY() + Math.random(), pos.getZ() + Math.random(), (float) Math.random(), (float) Math.random(), (float) Math.random(), 0.45F + 0.2F * (float) Math.random(), 6);
}
Botania开源地址: https://github.com/Vazkii/Botania
节选自(1.12-final分支): /common/block/BlockBifrostPerm.java
```

显然这段代码的含义便是粒子效果的渲染, randomDisplayTick会在贴图刷新的随机游戏刻执行, 可以看到Vazkii在代码的开头使用if (rand.nextBoolean())使下面代码执行的概率变为50%, 防止生成太多粒子造成卡顿. Vazkii自己实现了一套proxy来运行sparkleFX()来保持客户端渲染, 并且让粒子可以运动, 其中大多使用GL直接渲染, 这里的实现略微有点复杂我们就不展开了, 读者如果希望粒子效果包含运动的效果, 可以自行阅读Botania源码, 其代码大多位于fx包中

若要渲染普通的粒子效果, 使用world.spawnParticle(), 并且保证位于ClientSide执行

#### 过一段时间后自我消失

在BlockBifrost类中, 我们可以发现彩虹桥方块绑定了TileEntity, 其类为TileBifrost, 用于储存数据, 我们来看一下TileBifrost类

```
public class TileBifrost extends TileMod implements ITickable {

	private static final String TAG_TICKS = "ticks";

	public int ticks = 0;

	@Override
	public void update() {
		if(!world.isRemote) {
			if(ticks <= 0) {
				world.setBlockToAir(pos);
			} else ticks--;
		}
	}

	@Nonnull
	@Override
	public NBTTagCompound writeToNBT(NBTTagCompound par1nbtTagCompound) {
		NBTTagCompound ret = super.writeToNBT(par1nbtTagCompound);
		ret.setInteger(TAG_TICKS, ticks);
		return ret;
	}

	@Override
	public void readFromNBT(NBTTagCompound par1nbtTagCompound) {
		super.readFromNBT(par1nbtTagCompound);
		ticks = par1nbtTagCompound.getInteger(TAG_TICKS);
	}

}
Botania开源地址: https://github.com/Vazkii/Botania
节选自(1.12-final分支): /common/block/tile/TileBifrost.java
```

显然TileBifrost实现了ITickable, 这意味着这个方块具备了刷新数据的能力, 阅读update方法我们发现, 其实质就是自减内部储存的tick值, 如果减没了, 那么就使方块消失, writeToNBT和readFromNBT的用途不言而喻, 常写TileEntity的读者一定对其有所了解, Minecraft会在合适的时候(一般是保存世界和读取世界的时候)调用这两个方法, 以便于在进入和退出存档之前读写方块里面的数据

善于思考的读者此时可能发问了, 这个TileEntity的tick初始值应该是什么呢, 在上述对彩虹桥方块的分析过程中我们并没有看到对于tick初始值的声明....不妨再来想想彩虹桥方块是怎么被放置在世界中的呢....对了, 就是通过彩虹桥法杖,  所以不难想象设置tick的初始值的代码应该写在法杖中, 读者可自行验证猜想, 代码位于`/common/item/rod/ItemRainbowRod.java`的onItemRightClick方法中

------

### 其远而无所至极邪？

建筑小助手是direwolf20作为模组开发者的处女作, 其中各种小助手可以隔数十格远放置方块, 手中握着小助手就可以追踪到目光所及之处的方块, 通过查阅建筑小助手GadgetBuilding类的代码, 我们可以找到如下方法

```
private void build(ServerPlayerEntity player, ItemStack stack) {
	······
    BlockRayTraceResult lookingAt = VectorHelper.getLookingAt(player, stack);
    if (world.isAirBlock(lookingAt.getPos())) //If we aren't looking at anything, exit
    return;
	Direction sideHit = lookingAt.getFace();
	······
	placeBlock(world, player, index, builder, coordinate, blockData);
	······
}
BuildingGadgets开源地址: https://github.com/Direwolf20-MC/BuildingGadgets
节选自(master分支): /common/items/gadgets/GadgetBuilding.java
```

可以猜想, 在GadgetBuilding类的build方法中, 实现了建筑小助手追踪视线并放置方块的特性, 细心的读者可能发现, build方法里面还具有针对Collection和Undo操作功能实现, 的这里由于篇幅所限, 不能为读者介绍到更为详细的部分, 感兴趣的读者可以自行阅读查看

根据节选的代码片段, 相信读者可以发现, 追踪方块的实现依赖与VectorHelper.getLookingAt()这个方法, 他的返回值为一个BlockRayTraceResult, 对这个光追结果进行getPos(), 就可以获取到目光所及之处了, 所以我们更进一步, 来看看VectorHelper的具体实现

```
public class VectorHelper {
    public static BlockRayTraceResult getLookingAt(PlayerEntity player, RayTraceContext.FluidMode rayTraceFluid) {
        double rayTraceRange = Config.GENERAL.rayTraceRange.get();
        RayTraceResult result = player.pick(rayTraceRange, 0f, rayTraceFluid != RayTraceContext.FluidMode.NONE);

        return (BlockRayTraceResult) result;
    }
}
BuildingGadgets开源地址: https://github.com/Direwolf20-MC/BuildingGadgets
节选自(master分支): /common/util/helpers/VectorHelper.java
```

VectorHelper中包含了对于getLookingAt()的多个重写, 显然, 我们的目光重点应该放在player.pick()方法上, 相信聪明的读者已经意识到了什么, player.pick()就是原版为我们提供好的RayTrace的方法, 我们只需要提供Range(第一个参数), 和流体追踪模式(第三个参数)就可以了

------

### 海运则将徙于南冥

之后我们就要着手来将这两个点子融合, 我们再来回顾一下我们的想法, 既利用彩虹桥方块的发光效果, 以及建筑小助手getLookingAt()方法来制作一个便利的建筑方块, 