---
title: 由实例上手快速开发多方块结构
date: 2022-01-26 11:19:59
categories: [minecraft]
tags:
    - Minecraft
    - 模组开发
---

## 前言 - 真的很快很简单，真的

许多开发者对Minecraft中如何实现多方块结构感兴趣，鉴于国内相关知识的缺失，开发者们很难将自己的想法付诸实践，这里笔者将带领读者力求简单快速的构建自己的多方块结构。

鉴于本篇教程的基础性，我们并不想探究多方块结构中复杂的位置变换和数学计算，而仅仅是想用最快的方式上手并使用它们。所以在撰写过程中我们并不会深究某些方法的具体实现，而是将重点放在如何使用上，降低读者的阅读门槛。

本文编写使用环境: Minecraft 1.16.5 Forge 36.2.23 ZeroCore2 2.1.9

## 明确思路 - 你的下一个蓄水器何必是蓄水器

审视我们之前见到过的多方块结构，笔者这里想把他们分成两类。首先，例如RailCraft的多方块蓄水器，这种多方块结构往往需要玩家搭建一个立方体结构(这个结构的长宽高往往可以变化)，然后在立方体结构上面添加自己需要的交互方块(例如蓄水器阀门)用来与外界交互，笔者希望把这种多方块结构称为立体型多方块结构。

我们似乎还能想到另外一些例子，例如OpenBlocks的蓄水器，可以单个放置使用，但玩家可以通过在旁边摆更多的蓄水器来扩展蓄水器的空间，这种结构往往具有一种扩展性，即围绕一个中心方块(我们第一次放的蓄水器就可以看做中心方块)进行扩展，以扩增原有的功能，笔者希望把这种多方块结构称为扩展性多方块结构。对于扩展型多方块结构，由[Ph-苯](https://www.mcbbs.net/home.php?mod=space&uid=588878)开发的[容积储存](https://www.mcbbs.net/forum.php?mod=viewthread&tid=1186201)可以成为很好的学习例子。

![Tanks](https://s2.loli.net/2022/01/22/mSKjHT1RJvf46WA.png)

这里笔者将着重介绍立体型多方块结构，当然，有的读者除上述的两种多方块结构外，还想到了例如Immersive Engineering的各类多方块结构，他们的形状并不是非常规则的立方体，这里笔者尚且不介绍这种结构，感兴趣的读者可以尝试阅读IE的源码。

对于立体型多方块结构，我们使用ZeroCore2中提供的Multiblock API进行实现

## 立体型多方块结构 - Powerful Tank

本节将带领读者开发一个多方块储罐，~~这个储罐利用Forge Energy构建一个多维异次元空间，使用能量立场对流体的分子结构进行压缩~~ 这个多方块储罐可以储存无限桶流体，但需要消耗FE能量, 当能量不足时内部储存的所有流体将消失。

首先我们创建项目并添加所需的依赖，为了使用ZeroCore中Multiblock API，首先我们要引入ZeroCore2。利用ForgeGradle，我们无需手动下载。在build.gradle中的repositories和dependencies中加入以下条目，并且在mods.toml中添加依赖，之后点击IDE中的加载Gradle变更即可。

```java
位于: build.gradle
repositories {
    maven {
        name = "zerono"
        url = "https://maven.zerono.it/"
    }
}

dependencies {
    //替换成当前的Minecraft版本和ZeroCore2版本
    implementation fg.deobf("it.zerono.mods.zerocore:ZeroCore2:1.16.5-2.1.9") 
}
```

```toml
位于: resources\META-INF\mods.toml
[[dependencies.powerful_tank]]
    modId="zerocore"
    mandatory=true
    versionRange="[1.16.5-2.1.9,)"
    ordering="AFTER"
    side="BOTH"
```

ZeroCore是由ZeroNoRyouki开发的一套API，其中包含了网络包、GUI、连接材质、矿物生成等功能，这里我们着重使用ZeroCore的多方块结构API。需要注意的是，本文面向高版本的ZeroCore2开发。

读者需要注意，本篇教程并不适用于Minecraft 1.12.2及以前的旧版ZeroCore，有需要的读者可以阅读原作者发到自己博客的开发教程: [Zero CORE multiblock API tutorial](http://zerono.it/zerocore-multiblock-api-tutorial/)，该教程已严重过时，仅适用于1.12.2及更早版本的ZeroCore。
> 由于原作者尚未针对ZeroCore2编写相适应的WIKI，本文是笔者根据ZeroNoRyouki自己的模组极限反应堆([Extreme Reactors](https://www.curseforge.com/minecraft/mc-mods/extreme-reactors))中相关内容总结理解而来，可能存在错误，欢迎批评指正。

想要创建一个多方块结构，在代码层面我们需要有如下考量，笔者把它们分为虚实两部分：

- 实：什么方块构成了这个多方块结构，这些方块需要提供什么功能。
- 虚：这个多方块结构整体有何功能，如何编写逻辑。

比如Powerful Tank这个例子中，对于实的部分，我们需要设计对应的能量和流体的接口方块，这些方块必然的需要创建对应的TileEntity(或称BlockEntity)来提供数据处理的支持。对于虚的部分，我们可能需要设计一个主控类负责管理能源和流体，这个类所描述的事物是多方块结构的**整体效果**，我们的设计中心应该在这个类上。

- 虚：这个多方块结构整体有何功能，如何编写逻辑。比如Powerful Tank这个例子中，我们可能需要设计一个MultiBlockTank类，负责管理能源和流体(储存多少流体，能量不足时清空流体)，这个类所描述的事物是多方块结构的**整体效果**，我们的**设计中心**应该在这个类上。

在ZeroCore2中，主控类实际上被称作MultiblockController, 而组成多方块结构的部件，被称为MultiblockPart，需要注意的是，MultiblockPart继承自TileEntity，换言之，我们不应该在功能方块的XxxBlock类上做文章，而是在XxxTileEntity上做文章。

可能有读者对MultiblockController的说法有误解，这里的MultiblockController与我们在游戏中常见的XXX控制器是没有关联的，对于ZeroCore来说XXX控制器也算是多方块的部件，如果读者阅读极限反应堆的源码可以发现，反应堆控制器的TileEntity也继承自MultiblockPart。**反应堆控制器中是不包含反应堆工作的任何逻辑的**，相信理解了笔者写的虚和实关系的读者已经明白了。由此也可以看出，我们创建的多方块结构也是完全可以不包含XXX控制器的，在我们的例子中，笔者就并没有设计储罐控制器方块。

---

综合我们的考量，我们的代码结构应该是这样的

![Structure](https://s2.loli.net/2022/01/25/xc1C8IbWTQaFkDr.png)

首先，我们先创建代表多方块结构的主控类，在multiblock包中创建类MultiblockTank, 这个类需要继承AbstractCuboidMultiblockController。

因为继承自抽象类，我们需要补全他的所有抽象方法。

```java
位于: powerful_tank/multiblock/MultiblockTank.java
public class MultiblockTank extends AbstractCuboidMultiblockController<MultiblockTank> {

}
```

> 读者可能发现, 除了Cuboid外，我们还可以选择Rectangular的主控。对于立方体结构，我们选择Cuboid，而平面的结构一般选择Rectangular，感兴趣的读者可以自己研究Rectangular版本。需要注意的是，在1.12.2版本Rectangular和Cuboid的概念并不是很明确，如果阅读过1.12.2文档的读者在这里可能感到疑惑，请以笔者的版本为准。

使用IDE自动填充所有抽象方法后，我们可以先关注这样几个方法

```java
位于: powerful_tank/multiblock/MultiblockTank.java
@Override
    // 这里定义了多方块结构中Part的最少数量，我们把储罐外壳也作为Part，所以3*3中空结构正好需要26个Part
    protected int getMinimumNumberOfPartsForAssembledMachine() {
        return 26; 
    }

    // 多方块结构在X,Y,Z方向上的尺寸，ZeroCore允许创建可变大小的结构，这里我们想使用3*3的固定大小的结构，所以所有方向的尺寸均为3
    @Override
    protected int getMaximumXSize() {
        return 3; 
    }

    @Override
    protected int getMinimumXSize() {
        return 3;
    }

    @Override
    protected int getMaximumZSize() {
        return 3;
    }

    @Override
    protected int getMinimumZSize() {
        return 3;
    }

    @Override
    protected int getMaximumYSize() {
        return 3;
    }

    @Override
    protected int getMinimumYSize() {
        return 3;
    }

    // 这里我们需要判断某个非Part方块方块是否合法。
    @Override
    protected boolean isBlockGoodForInterior(@Nonnull World world, int i, int i1, int i2, @Nonnull IMultiblockValidator iMultiblockValidator) {
        return world.getBlockState(new BlockPos(i, i1, i2)).getMaterial() == Material.AIR;
    }
```

与isBlockGoodForInterior相似的一系列方法参与了多方块结构的判断，ZeroCore会在合适的时候调用他们来判断多方块结构是否成立。在后文我们还会看到关于多方块结构判断的更多细节。

这里的isBlockGoodForXXX的一系列方法，会在ZeroCore发现**非Part方块**以后执行，一般情况下我们不希望多方块结构中混入其他无关方块，所以所有的方法我们均返回false。但是这里我们希望多方块结构**中空**，所以我们重写了isBlockGoodForInterior，并且要求多方块结构内部为AIR。

之后我们来创建对应的Part，这里我们以外壳GlassWallBlock为例，读者可以采用相似的方法创建GlassEnergyPortBlock，GlassFluidPortBlock几个类。

在block包中创建类GlassWallBlock

```java
位于: powerful_tank/block/GlassWallBlock.java
// 为了实现透明效果，我们继承自AbstractGlassBlock
public class GlassWallBlock extends AbstractGlassBlock {
    public GlassWallBlock() {
        // 为了实现透明效果，必须设置noOcclusion
        super(Properties.of(Material.GLASS, MaterialColor.NONE).sound(SoundType.GLASS).noOcclusion());
        this.setRegistryName("powerful_tank", "wall_block");
    }

    @Override
    public boolean hasTileEntity(BlockState state) {
        return true;
    }

    @Nullable
    @Override
    public TileEntity createTileEntity(BlockState state, IBlockReader world) {
        return new GlassWallTileEntity();
    }
}
```

之后我们来创建对应的GlassWallTileEntity，在包tileEntity中创建类GlassWallTileEntity，他需要继承自AbstractCuboidMultiblockPart，同时泛型类型为我们先前创建的MultiblockTank

```java
位于: powerful_tank/tileentity/GlassWallTileEntity.java
public class GlassWallTileEntity extends AbstractCuboidMultiblockPart<MultiblockTank> {
    public GlassWallTileEntity() {
        // 待会我们要注册的TILE_ENTITY_TYPE
        super(PowerfulTank.RegistryEvents.TILE_ENTITY_TYPE_WALL);
    }

    // 不允许Part被放置在多方块内部
    @Override
    public boolean isGoodForPosition(@Nonnull PartPosition partPosition, @Nonnull IMultiblockValidator iMultiblockValidator) {
        return partPosition != PartPosition.Interior;
    }

    // 这里需要返回我们MultiblockTank类的实例
    @Nonnull
    @Override
    public MultiblockTank createController() {
        return new MultiblockTank(getCurrentWorld());
    }

    // 这里需要返回我们MultiblockTank类的Class
    @Nonnull
    @Override
    public Class<MultiblockTank> getControllerType() {
        return MultiblockTank.class;
    }
}
```

上文我们提到了非Part方块的成立条件判断，对于Part的判断则是在这里的isGoodForPosition方法中。PartPosition是一个枚举类，读者可以自行根据自己需要的逻辑进行判断，这里我们只是不希望Part出现在结构内部，所以针对Interior进行了拒绝。

我们在主类中添加对应的注册，注意这里我们需要对三个方块的RenderType进行特殊的注册，因为我们希望这三者的渲染方式为CutoutMipped，感兴趣的读者可以阅读[Boson的相关内容](https://boson.v2mcdev.com/block/rendertype.html)

```java
位于: powerful_tank/PowerfulTank.java
@Mod("powerful_tank")
public class PowerfulTank {
    // 这里我们让注册类作为主类的内部类
    @Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.MOD)
    public static class RegistryEvents {
        public static final GlassWallBlock BLOCK_WALL = new GlassWallBlock();
        public static final TileEntityType<GlassWallTileEntity> TILE_ENTITY_TYPE_WALL = TileEntityType.Builder.of(GlassWallTileEntity::new, BLOCK_WALL).build(DSL.remainderType());

        @SubscribeEvent
        public static void onBlocksRegistry(RegistryEvent.Register<Block> event) {
            event.getRegistry().registerAll(BLOCK_WALL);
        }

        @SubscribeEvent
        public static void onItemsRegistry(RegistryEvent.Register<Item> event) {
            event.getRegistry().register(new BlockItem(BLOCK_WALL, new Item.Properties().tab(ItemGroup.TAB_BUILDING_BLOCKS)).setRegistryName("powerful_tank", "wall_block"));
        }

        @SubscribeEvent
        public static void onTileEntitiesRegistry(RegistryEvent.Register<TileEntityType<?>> event) {
            event.getRegistry().register(TILE_ENTITY_TYPE_WALL.setRegistryName("powerful_tank:wall"));
        }

        // 注意对于玻璃样式的材质，需要注册RenderType
        @SubscribeEvent
        public static void onRenderTypeRegistry(FMLClientSetupEvent event) {
            event.enqueueWork(() -> {
                RenderTypeLookup.setRenderLayer(BLOCK_WALL, RenderType.cutoutMipped());
            });
        }
    }
}
```

相似的，读者应该补全相对于的其他Part的类和注册(EnergyPort和FluidPort)，读者完成后，项目结构应该类似是这样:

![Structure](https://s2.loli.net/2022/01/25/xc1C8IbWTQaFkDr.png)

之后我们来为能量接口和流体接口添加承载能量和流体的能力，即添加Capability。

对于Capability感兴趣的读者，这里笔者推荐阅读[Boson的相关部分](https://boson.v2mcdev.com/capability/intro.html)。

对于实现流体能力感兴趣的读者，这里笔者推荐阅读[Harbinger的相关部分](https://harbinger.covertdragon.team/chapter-27/container/block.html)。

对于实现能量能力感兴趣的读者，这里笔者推荐阅读[Forge 能量系统简述](https://www.mcbbs.net/forum.php?mod=viewthread&tid=1034965)。

首先，在GlassEnergyPortTileEntity中加入以下内容，为他添加能量储存

```java
位于: powerful_tank/tileentity/GlassEnergyPortTileEntity.java
// 因为我们对能量的需求比较简单，我们这里没有自己手动实现IEnergyHandler，而是直接利用了Forge实现好的EnergyStorage类
public final EnergyStorage energyStorage = new EnergyStorage(2_500_000, 2_500_000);

// 在1.16.5中，我们只需要重写getCapability，而不需要重写hasCapability了
// 这里需要注意，我们应该重写的是包含两个参数Capability<T>和Direction的方法，而不是仅有一个参数的方法，这里读者要注意
@Nonnull
@Override
public <T> LazyOptional<T> getCapability(@Nonnull Capability<T> cap, @Nullable Direction side) {
    if (cap == CapabilityEnergy.ENERGY) {
        return LazyOptional.of(() -> energyStorage).cast();
    }
    return super.getCapability(cap, side);
}

// 读者不要忘记储存和读取TileEntity内部的数据，这里我们把储存的能量储存在energy标签中
@Override
public void syncDataFrom(@Nonnull CompoundNBT data, @Nonnull SyncReason syncReason) {
    super.syncDataFrom(data, syncReason);
    energyStorage.receiveEnergy(data.getInt("energy"), false);
}

@Nonnull
@Override
public CompoundNBT syncDataTo(CompoundNBT data, @Nonnull SyncReason syncReason) {
    data.putInt("energy", energyStorage.getEnergyStored());
    return super.syncDataTo(data, syncReason);
}

// 这个方法并不是必要的，但是方便我们一会查看容器内的能量和流体储存
public String getEnergyText() {
        int cost = 0;
        // 注意处理多方块结构的存在性问题
        if (getMultiblockController().isPresent() && isMachineAssembled()) {
            // 一会我们来补充这个方法
            cost = getMultiblockController().get().getCost();
        }
        return String.format("Energy: %d / %d FE, Cost: %d FE/t", energyStorage.getEnergyStored(), energyStorage.getMaxEnergyStored(), cost);
    }
```

此时进入游戏，我们的能量接口已经支持了各个遵循Forge标准的模组的能量管线。

![Energy Port](https://s2.loli.net/2022/01/26/lxaA5ur9z4BdIco.png)

同样的，这里我们为GlassFluidPortTileEntity添加流体支持

```java
位于: powerful_tank/tileentity/GlassFluidPortTileEntity.java
// 这里我们也没有手动实现IFluidHandler，而是使用了Forge实现好的FluidTank，并且做了小更改
public final FluidTank fluidTank = new FluidTank(Integer.MAX_VALUE) {
    @Override
    public int fill(FluidStack resource, FluidAction action) {
        // 不构成多方块就想填入流体？
        return super.fill(GlassFluidPortTileEntity.this.isMachineAssembled() ? resource : FluidStack.EMPTY, action);
    }

    @Nonnull
    @Override
    public FluidStack drain(int maxDrain, FluidAction action) {
        // 不构成多方块就想取出流体？
        return super.drain(GlassFluidPortTileEntity.this.isMachineAssembled() ? maxDrain : 0, action);
    }
};

@Nonnull
@Override
public <T> LazyOptional<T> getCapability(@Nonnull Capability<T> cap, @Nullable Direction side) {
    if (cap == CapabilityFluidHandler.FLUID_HANDLER_CAPABILITY) {
        return LazyOptional.of(() -> fluidTank).cast();
    }
    return super.getCapability(cap, side);
}

@Override
public void syncDataFrom(@Nonnull CompoundNBT data, @Nonnull SyncReason syncReason) {
    super.syncDataFrom(data, syncReason);
    fluidTank.readFromNBT(data);
}

@Nonnull
@Override
public CompoundNBT syncDataTo(@Nonnull CompoundNBT data, @Nonnull SyncReason syncReason) {
    fluidTank.writeToNBT(data);
    return super.syncDataTo(data, syncReason);
}

// 这个方法并不是必要的，但是方便我们一会查看容器内的能量和流体储存
public String getFluidText() {
    FluidStack fluidStack = fluidTank.getFluid();
    int cost = 0;
    if (getMultiblockController().isPresent() && isMachineAssembled()) {
        cost = getMultiblockController().get().getCost();
    }
    return String.format("Fluid(%s): %d / %s mb, Cost: %d FE/t", I18n.get(fluidStack.getTranslationKey()), fluidStack.getAmount(), "INFINITE", cost);
}
```

为了限制流体只在多方块成立时可以存取，我们这里重写了FluidTank的两个方法。

对于多方块结构的Part，我们可以调用isMachineAssembled来得知多方块结构是否完整，同时也可以调用getMultiblockController来获得到多方块结构自身。这里可能需要读者处理一下Optional问题。

此时进入游戏，我们的流体接口也可以接上其他模组的流体管道了，但此时流体应该还流不进去，因为我们没有构成完整的多方块结构。

![Fluid Port](https://s2.loli.net/2022/01/26/HeDWI8KzTpaqdXx.png)

最后我们来补全MultiblockTank的逻辑。在MultiblockTank中添加下面的内容

```java
位于: powerful_tank/multiblock/MultiblockTank.java
private EnergyStorage energyStorage;
private FluidTank fluidTank;

// 为了让多方块结构找到两个接口，我们添加了这样一个私有方法，我们可以利用_connectedParts成员来获得所有连接的Part
private void findPorts() {
    for (IMultiblockPart<MultiblockTank> connectedPart : _connectedParts) {
        energyStorage = null;
        fluidTank = null;
        // 获得位于各个Part的实例，这种手法很常见
        if (connectedPart instanceof GlassEnergyPortTileEntity)
            energyStorage = ((GlassEnergyPortTileEntity) connectedPart).energyStorage;
        if (connectedPart instanceof GlassFluidPortTileEntity)
            fluidTank = ((GlassFluidPortTileEntity) connectedPart).fluidTank;
    }
}

// 当多方块结构的区块被卸载后重新加载时，会调用这个方法
@Override
protected void onMachineRestored() {
    // 我们希望这个时候重新搜索一次Part
    findPorts();
}

// 当多方块结构形成时，会调用这个方法
@Override
protected void onMachineAssembled() {
    // 这应该是第一次寻找Part
    findPorts();
}

// 最终判定多方块结构是否成立的方法，下文会予以介绍
@Override
protected boolean isMachineWhole(@Nonnull IMultiblockValidator validatorCallback) {
    findPorts();
    if (energyStorage != null && fluidTank != null)
        return super.isMachineWhole(validatorCallback);
    else
        //validatorCallback.setLastError(······);
        return false;
}
```

这里笔者详细介绍一下ZeroCore是如何判断多方块结构是否成立的。读者如果查看父类的isMachineWhole方法大概可以猜出，这个方法对于Part类方块，会利用我们之前提供的isGoodForPosition方法来辨别该方块是否合法，而对于其他方块，则会调用isBlockGoodForXXX的一系列方法来判断是否合法。

我们要做的，便是重写isMachineWhole，添加我们自己所需的判断条件。例如我们上文调用findPorts方法，只有两个接口都存在，才会进一步判断多方块结构是否成立。

这里细心的读者可能对validatorCallback抱有疑问，我们上文多次见到这个参数，且都是在与结构成立判断的相关方法中存在，我们可以猜测这个参数可能与结构判断有关。

其实，validatorCallback是负责处理多方块结构不成立时的错误信息的，当结构出现不符合的情况时，我们希望给予一个错误信息来让玩家知道如何正确摆放多方块机器。在阅读父类isMachineWhole方法中，我们发现ZeroCore已经针对一些常见的问题处理了错误文本，我们只需关心自己的逻辑判断中的问题，并给予提示就可以了。

这里读者需要补全setLastError并且给予错误字符串，这里的字符串是支持lang提供的国际化的，读者可以自行设置。如果希望显示最后一个错误，可以调用validatorCallback.getLastError()方法，下文我们会看到相关的例子。

之后，我们需要添加核心逻辑，相信读者应该对ITickableTileEntity略知一二，他会在每一个游戏刻执行TileEntity内的代码，正是这个功能给Minecraft中无数科技模组的机器提供了可能。相似的，多方块结构也有类似的方法。

对于多方块结构，updateServer方法会在逻辑服务端每一个游戏刻被执行一次，所以请用对待tick方法的态度对待他，尽量减小对服务器性能的影响。

```java
位于: powerful_tank/multiblock/MultiblockTank.java
@Override
protected boolean updateServer() {
    if (energyStorage != null && fluidTank != null) {
        // 每一个tick都扣除指定的能量，energyStorage实例我们上文已经获取到了
        if (energyStorage.getEnergyStored() > 0) energyStorage.extractEnergy(getCost(), false);
        // 能量归零且有流体存在
        if (energyStorage.getEnergyStored() == 0 && fluidTank.getFluidAmount() != 0) {
            fluidTank.setFluid(FluidStack.EMPTY);
        }
        return true;
    }
    return false;

    // 这里，如果
}

// 根据内部保存的流体数量计算耗能，这里对于1000桶流体大概消耗251RF/t
public int getCost() {
    return (int) Math.pow(fluidTank.getFluidAmount() / 1000.0, 0.8);
}
```

读者可能注意到updateServer具有返回值，这个返回值意味着是否需要数据同步。我们操作了储存在世界中的TileEntity中相关数据的值，所以我们在值发生改变的时候返回true。类似于TileEntity中的markDirty方法

现在进入游戏，我们的多方块结构已经能够正常运作了，相信阅读到了这里，读者已经对如何开发一个简单的多方块机器有了自己的理解。我们这里还会补全一些外围方法，来让机器更人性化。首先，我们希望多方块结构在没有构成时告知我们错误的原因。我们编辑GlassWallBlock类并且重写use方法

```java
位于: powerful_tank/block/GlassWallBlock.java
@Nonnull
@Override
public ActionResultType use(@Nonnull BlockState pState, World pLevel, @Nonnull BlockPos pPos, @Nonnull PlayerEntity pPlayer, @Nonnull Hand pHand, @Nonnull BlockRayTraceResult pHit) {
    if (!pLevel.isClientSide && pHand == Hand.MAIN_HAND) {
        GlassWallTileEntity te = (GlassWallTileEntity) pLevel.getBlockEntity(pPos);
        if (te != null && te.getMultiblockController().isPresent() && !te.isMachineAssembled() && pPlayer.isShiftKeyDown()) {
            // 对于可能存在的错误信息，我们使用Optional来处理，调用validationError.getChatMessage()就可以得到本地化后的信息，方便发给玩家。
            Optional<ValidationError> errorOptional = te.getMultiblockController().get().getLastError();
            errorOptional.ifPresent(validationError -> pPlayer.sendMessage(validationError.getChatMessage(), PowerfulTank.POWERFUL_TANK_UUID));
            return ActionResultType.SUCCESS;
        }
    }
    return super.use(pState, pLevel, pPos, pPlayer, pHand, pHit);
```

![Tank Error](https://s2.loli.net/2022/01/26/1ojJCAZdyGvtTO3.png)

玩家可能还想要知道此时储存的能量和流体数量，我们也添加相关方法，编辑GlassEnergyPortBlock和GlassFluidPortBlock类

```java
位于: powerful_tank/block/GlassEnergyPortBlock.java
@Nonnull
@Override
public ActionResultType use(@Nonnull BlockState pState, World pLevel, @Nonnull BlockPos pPos, @Nonnull PlayerEntity pPlayer, @Nonnull Hand pHand, @Nonnull BlockRayTraceResult pHit) {
    if (!pLevel.isClientSide && pHand == Hand.MAIN_HAND) {
        GlassEnergyPortTileEntity te = (GlassEnergyPortTileEntity) pLevel.getBlockEntity(pPos);
        // 只有在多方块结构形成时，才希望读取内部储存的能量
        if (te != null && te.isMachineAssembled()) {
            // 这里调用了我们之前写过的getEnergyText方法
            pPlayer.sendMessage(new StringTextComponent(te.getEnergyText()), PowerfulTank.POWERFUL_TANK_UUID);
            return ActionResultType.SUCCESS;
        }
    }
    return super.use(pState, pLevel, pPos, pPlayer, pHand, pHit);
}
```

```java
位于: powerful_tank/block/GlassFluidPortBlock.java
@Nonnull
@Override
public ActionResultType use(@Nonnull BlockState pState, World pLevel, @Nonnull BlockPos pPos, @Nonnull PlayerEntity pPlayer, @Nonnull Hand pHand, @Nonnull BlockRayTraceResult pHit) {
    if (!pLevel.isClientSide && pHand == Hand.MAIN_HAND) {
        GlassFluidPortTileEntity te = (GlassFluidPortTileEntity) pLevel.getBlockEntity(pPos);
        // 只有在多方块结构形成时，才希望读取内部储存的流体
        if (te != null && te.isMachineAssembled()) {
            // 这里调用了我们之前写过的getFluidText方法
            pPlayer.sendMessage(new StringTextComponent(te.getFluidText()), PowerfulTank.POWERFUL_TANK_UUID);
            return ActionResultType.SUCCESS;
        }
    }
    return super.use(pState, pLevel, pPos, pPlayer, pHand, pHit);
}
```

![Tank Overall](https://s2.loli.net/2022/01/26/ERlQo1hHd75akKn.png)

至此，我们已经完成了一个基于ZeroCore的立体型多方块结构。

---

这里笔者还希望补充一些附加思考，读者可以选择性阅读：

善于逻辑的读者可能看出，这里笔者的逻辑实现方式略显Dirty，读者可能认为，FluidTank这一成员不应是流体接口所拥有的，而是整个多方块结构MultiblockTank所应该拥有的。即拥有储存流体能力的并不应该是流体接口，而是整个多方块储罐。

对于这一问题的答案，读者毋庸置疑是正确的，这一成员确实应该移至MultiblockTank中，这里笔者的确处理的并不是很妥当。但如若读者仅是单纯将它移动至MultiblockTank，并且对Capability的处理是从MultiblockTank中获取(可能需要是否为空的判断)，这样的行为也是不正确的。因为Capability的本意是某一方块具有某一能力，这一能力不应该从别处引用。如果读者这样尝试，会得到一个结果，当重新进入存档时，所有连接的管道均会断开，因为读取存档的时候，各个Mod获取我们流体接口的Capability是在多方块结构形成之前，这时我们无法获取到对应的MultiblockTank实例，也就无法获取到对应的FluidTank(即Capability本身)。

那对于这一问题如何处理呢，如果感兴趣的读者阅读极限反应堆代码可以了解到，原作者是使用一种Forwarder的想法来处理的，即其他模组访问到的不是FluidTank本身，而是一个Forwarder，这个Forwarder负责处理对于多方块机器的连接。从逻辑角度考虑，对于我们的流体接口，他所含有的能力应该是 **连接至多方块结构中的FluidTank** 的能力，而不是具有FluidTank的能力，在逻辑层面编写一个Forwarder也是合理的，对此感兴趣的读者可以自行阅读，并且尝试改进笔者的代码。[其中一个例子](https://github.com/ZeroNoRyouki/ExtremeReactors2/blob/master/src/main/java/it/zerono/mods/extremereactors/gamecontent/multiblock/reprocessor/part/ReprocessorFluidPortEntity.java)

![image.png](https://s2.loli.net/2022/01/26/rehv8tI62gFfLc1.png)

需要注意的是如果改进以后，FluidTank的数据持久化就需要由我们的MultiblockTank来完成，可喜的是，MultiblockTank也提供了对应的syncDataFrom和syncDataTo方法，这有助于我们储存多方块结构整体的数据，也就是这里我们需要储存的FluidTank。

---

这里笔者还希望添加一些上文例子中没有包含到的一些方法的补充：

对于MultiBlockController，还有一些相关的方法读者可能会用到

- onPartAdded 当Part并入Multiblock的时候会被执行，如果读者用一个列表来储存很多Part以作备用，这里可以向列表中添加Part
- onPartRemoved 同理，这里可以从列表中移除某些Part
- onMachinePaused 因为一些特殊原因(比如区块卸载)导致多方块结构无法正常使用时，会调用这个方法
- onMachineRestored 区块重新加载时会调用这个方法
- onMachineDisassembled 当多方块结构被破坏时会调用这个方法(例如爆炸或者被玩家挖掉)
- onAssimilate 当有另外一个结构与当前结构合并时，会调用这个方法，参数中会给出另外一个结构的Controller, 开发者应该在这里接管对方的所有数据(例如把对面的能量输入到自己里面)
- onAssimilated 与上一个相似的，如果自己被合并了，会调用这个方法，通常是清理一些之前的数据。这两个功能一般很少会用到，可以参考极限反应堆的相关代码

对于MultiblockPart，读者可以自行查阅。

## 小结与后记

经过本节的学习，读者应该学会了如何利用ZeroCore快速建立一个属于自己的多方块结构，同时也了解了多方块的各个部件间是如何与主控类建立联系的，这种思路将始终贯穿各种方式开发的各种多方块结构中，相信读者阅读本文后，对进一步阅读并理解IE和容积储存等Mod多方块结构的设计思路有很大帮助。同时这里笔者也针对读者可能产生的想法进行了解读，这些问题和相应的解决方案也是笔者行文时产生并解决的。这里笔者始终希望读者明白，在模组开发的过程中，你所遇到的需求和解决方案并不总是前人总结好的经验，更多的是在阅读前人代码中自行理解和解读出来的。如何让读者尝试去解读，让读者有自己为自己写指南的能力，这正是笔者行文的初衷，也是之前举二反三系列的初衷。

至此，读者可以自行根据需求开发自己的多方块结构，或者依靠本节的经验进一步深挖多方块结构的魅力。

本文代码已开源：[Github - Powerful Tank](https://github.com/TROU2004/PowerfulTank)

## 参考资料

ExtremeReactors2: <https://github.com/ZeroNoRyouki/ExtremeReactors2>

Forge 能量系统简述: <https://www.mcbbs.net/forum.php?mod=viewthread&tid=1034965>

Boson 1.16 Modding Tutorial: <https://boson.v2mcdev.com/>

Harbinger: <https://harbinger.covertdragon.team/>

CapacityStorage: <https://github.com/Phoupraw/CapacityStorage>

ZERONOMODS(过时): <http://zerono.it/zerocore-multiblock-api-tutorial/>

ZeroTest(过时): <https://github.com/ZeroNoRyouki/ZeroTest>
