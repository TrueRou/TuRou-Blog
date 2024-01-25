---
title: 跨加载器开发实战 - 经验储存器
date: 2022-07-12 11:35:12
categories: [arch-tutor]
tags:
    - Minecraft
    - 模组开发
    - Architectury
---

![效果图](https://s2.loli.net/2022/04/07/FMYQtNmyU5n9Wdb.png)

观察图片可以猜测, 本节运用到的知识内容:

* **DeferredRegister** - 创建并注册游戏元素
* **I18n** - 为元素添加本地化和材质
* **CompoundTag** - 为物品添加功能
* **ClientTooltipEvent** - 为物品添加Tooltips (使用封装好的事件)

下面我们来基于Architectury API来编写我们的第一个物品 —— 经验储存器.

### 创建并注册物品

在 **trou.arch.item** 包下创建 **ItemExpContainer** 类, 使其继承自**原版的**Item类

```java
位于common: trou/arch/item/ItemExpContainer.java
public class ItemExpContainer extends Item {
    public ItemExpContainer() {
        super(new Properties().stacksTo(1).tab(CreativeModeTab.TAB_TOOLS));
    }
}
```


熟悉Forge开发的读者在这里可能产生疑问, 读者可能尝试设置RegistryName和TranslationKey, 结果发现没有这样的方法。

实际上, Forge为原版的Item进行了许多层封装, 读者可以尝试查阅forge模块中的Item类, 会发现它实现了IForgeItem, 继承自ForgeRegistryEntry. Forge为了易用性舍弃了轻量的结构. Forge将注册名的设置封装在了物品内部, 这与原版的逻辑不同.

而Fabric中并没有额外的封装, 而是基于Minecraft原本的样子. 读者可能意识到, 实际上Architectury是更倾向于原版, 或者说Fabric的. 所以我们在开发中, RegistryName实际上是在注册的时候提供的。

Architectury这里采用原版(Fabric)的架构, 而为Forge进行兼容是合理的。在跨平台开发中，往往要**基于低**而**适配高**，这也是可理解的。正如DisplayPort转HDMI容易, 而HDMI转DisplayPort困难的道理。


下面我们要注册这个物品, 我们采用Architectury封装的**DeferredRegister**进行注册. 相同的, 如果读者要注册其他的方块、附魔、BlockEntityType等元素, 也可以使用相同的方法. 读者可以查阅Registry中的常量

在 **trou.arch.object** 包下创建 **ModItems** 类

```java
位于common: trou/arch/object/ModItems.java
public class ModItems {
    private static final DeferredRegister<Item> ITEMS = DeferredRegister.create(Arch.MOD_ID, Registry.ITEM_REGISTRY);
    public static final RegistrySupplier<Item> EXP_CONTAINER_ITEM = ITEMS.register("exp_container_item", ItemExpContainer::new);
    public static void register() {
        ITEMS.register(); //DeferredRegister的register方法需要在init阶段被调用
    }
}
```

为了注册物品, 我们调用`DeferredRegister.create` 来创建一个注册器

针对每一个物品, 我们需要调用注册器的`register`方法来注册, 需要传入注册名和构造方法

之后, 我们在模组的**init**阶段调用`ModItems.register()`方法

<mark style="color:red;">**因为Forge的某些限制, 我们需要在forge模块中手动注册一次Architectury的事件总线**</mark>

```java
位于forge: trou/arch/forge/ArchForge.java
@Mod(Arch.MOD_ID)
public class ArchForge {
    public ArchForge() {
        //这里一定要注册上Architectury的EventBus
        EventBuses.registerModEventBus(Arch.MOD_ID, FMLJavaModLoadingContext.get().getModEventBus());
        Arch.init();
    }
}
```

之后分别进入fabric和forge客户端，应该都能看到我们的紫黑块物品了。


笔者这里储存了ITEMS.register返回的RegistrySupplier, 如果在代码的别处需要用到ItemExpContainer的实例时, 可以调用RegistrySupplier的get方法.

不难理解, 其实RegistrySupplier充当了保存对象和它的注册名的功能


### 为物品添加本地化和材质

common模块中的assets在编译的时候会被合并进forge和fabric模块中, 所以我们只需按照熟悉的方式将本地化文件和材质放在common模块中就可以了.

```json
位于common: assets/arch/lang/zh_cn.json
{
  "item.arch.exp_container_item": "经验储存器",
  "tooltip.exp_container": "已储存经验: %s exp"
}

位于common: assets/arch/lang/en_us.json
{
  "item.arch.exp_container_item": "Experience Container",
  "tooltip.exp_container": "Stored experience: %s exp"
}

位于common: assets/arch/models/item/exp_container_item.json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "arch:item/exp_container_item"
  }
}

位于common: assets/arch/textures/item/exp_container_item.png
// 在这里放置物品的材质
```

### 为物品添加功能

根据我们的需求, 需要覆盖物品的use方法, 并编写我们的逻辑, 这里相信有经验的读者可以完成

```java
位于common: trou/arch/item/ItemExpContainer.java
@Override
public InteractionResultHolder<ItemStack> use(@NotNull Level level, @NotNull Player player, @NotNull InteractionHand usedHand) {
    ItemStack stack = player.getItemInHand(usedHand);
    if (level.isClientSide || usedHand == InteractionHand.OFF_HAND) return InteractionResultHolder.fail(stack);
    CompoundTag tag = stack.hasTag() ? stack.getTag() : new CompoundTag();
    assert tag != null;
    if (player.isShiftKeyDown()) {
        player.giveExperiencePoints(tag.getInt("exp"));
        tag.putInt("exp", 0);
    } else {
        int exp = player.totalExperience;
        player.giveExperiencePoints(-exp);
        tag.putInt("exp", tag.getInt("exp") + exp);
    }
    stack.setTag(tag);
    return InteractionResultHolder.sidedSuccess(stack, level.isClientSide());
}
```

### 监听Tooltip事件

****

**ArchtecturyAPI**建立了一套位于Forge和Fabric之上的事件封装层. 在Forge中, 我们创建方法并添加@SubscribeEvent注解来让Forge注册我们的事件. 而在**ArchtecturyAPI**中, 我们使用一种更为"静态"的方式



ArchitecturyAPI为我们提供了一些常用的事件, 在上一章节可以找到.&#x20;

这里我们需要监听**ClientTooltipEvent**事件, 首先创建一个类来注册所有的事件

```java
位于common: trou/arch/object/ModEvents.java
public class ModEvents {
    public static void register() {
        // 需要传入的是事件处理器的方法
        ClientTooltipEvent.ITEM.register(ItemExpContainer::append);
    }
}

位于common: trou/arch/Arch.java
public class Arch {
    public static void init() {
        ModEvents.register(); // 在init阶段注册事件
    }
}
```

然后我们按照代码提示, 在ItemExpContainer类中补全append方法来处理事件

```java
位于common: trou/arch/item/ItemExpContainer.java
public static void append(ItemStack stack, List<Component> lines, TooltipFlag flag) {
    if (stack.getItem() instanceof ItemExpContainer) {
        CompoundTag tag = stack.hasTag() ? stack.getTag() : new CompoundTag();
        assert tag != null;
        lines.add(new TranslatableComponent("tooltip.exp_container", tag.getInt("exp")));
    }
}
```


ClientTooltipEvent.ITEM.register背后实际上是利用@ExpectPlatform来处理的

针对两个不同的平台, API分别写了相应的事件处理器的实现, 因此简化了我们对于事件的注册

对于一个特定的需求, 首先我们要检查API是否已经为我们提供了相应的方法

如果没有提供相应的方法, 我们再去手动针对两个平台进行实现


最后进入游戏, 我们的第一个物品已经成功被加入了

# 再次开始

首先我们先从官方文档出发，看看Architectury API都为我们封装了哪些易于方法的方法和类。

**读者可以暂时跳过这部分, 进入下文的实例环节, 在以后的开发中如果有需要再来查阅.**


注: 某些类的描述添加了笔者自己的理解，并且附加了可能的应用方法，以便于读者自行摸索下文的例子中没有提到的使用方法。


### 实用的抽象层

| 类名                       | 描述                                                          |
| ------------------------ | ----------------------------------------------------------- |
| Platform                 | 提供了获取当前加载器的一些规范的类(比如配置文件位置, Mod列表等方法)                       |
| Registries               | 提供了注册物品, 方块等元素的一系列方法和接口的类(比如RegistryProvider)               |
| KeyBindings              | 提供了一些关于按键绑定的方法                                              |
| CreativeTabs             | 提供了创建CreativeTab的方法                                         |
| MenuRegistry             | 提供了关于Menu界面(例如统计信息, 设置)的一些注册方法                              |
| RenderTypes              | 统一了两个加载器的RenderTypes                                        |
| ReloadListeners          | 重载Listeners的注册器, 现在已经改成了ReloadListenerRegistry, 但是官方文档还没有更新 |
| CriteriaTriggersRegistry | 提供了一些关于成就触发器的方法, 现在这个类已经弃用, 如果读者要使用, 请调用CriteriaTriggers    |
| ColorHandlers            | 提供了一些关于颜色的方法                                                |
| BlockEntityRenderers     | 提供了一些绑定BlockEntity渲染器的方法                                    |
| BiomeModifications       | 提供了一些关于生物群系的方法                                              |
| PackRepositoryHooks      | 提供了一些关于高版本行为包的钩子方法                                          |

### 网络和网络包的抽象层

| 类名             | 描述                         |
| -------------- | -------------------------- |
| NetworkManager | 提供了一个类似于Fabric的网络包管理系统     |
| NetworkChannel | 提供了一个类似Forge Channel的包收发系统 |

### 抽象出来的一些钩子方法

| 类名               | 描述                                                  |
| ---------------- | --------------------------------------------------- |
| BiomeHooks       | 提供了一套重新封装的生物群系属性. 需要获取生物群系属性的可以调用这个类中的方法            |
| BlockEntityHooks | 将BlockEntity的数据变化推送至客户端的方法, 会调用区块的blockChanged方法来推送 |
| DyeColorHooks    | 提供了将DyeColor转换成颜色的方法                                |
| EntityHooks      | 提供了关于Entity的Collision相关方法, 源文档中提供的已弃用               |
| ExplosionHooks   | 提供了从爆炸(Explosion)获取爆炸位置的方法                          |
| ItemEntityHooks  | 可以获取一个物品实体的lifespan                                 |
| PlayerHooks      | 可以获取玩家是否是fakePlayer                                 |
| ScreenHooks      | 提供了一些关于在屏幕上渲染的方法                                    |

### 抽象出来的一些事件

![事件](https://s2.loli.net/2022/04/07/DxcKzeACnqY41iN.png)

Architectury API提供了一些我们常用的事件, 这里笔者可以根据名称自行理解用途.
