---
title: 深入浅出高版本渲染系统  —— GuiComponent的消逝与GuiGraphics的新生
date: 2023-11-12 11:19:59
tags:
    - Minecraft
    - 模组开发
---

渲染一直是困扰众多开发者的难题, 国内外开发文档较少的同时, 不断迭代的版本更是让现状雪上加霜, 如何让想象的美观界面元素化为现实, 成为了很多开发者不断追求的目标.

笔者也不例外, 经常阅读版本快讯的读者可能发现, 在 Minecraft 1.20 系列版本快讯的评论区内, "优化" 一词多次出现, 难道Mojang真的负重前行, 成功优化了渲染系统吗?

众多问题和想法推动笔者查阅相关资料和实现, 笔者希望借此机会与各位读者共同探清高版本渲染的迷雾, 共同了解高版本横空出世的GuiGraphics类, 及其背后的三两事.

按照笔者的行文习惯, 我们会通过游戏中几个常见的特性出发, 衍生出需求, 再以思维为导向来解决问题, 进而在实践中摸清渲染的基本逻辑.

笔者希望传授的不是知识本身, 更是获取知识的方式.

本文编写使用环境: Minecraft 1.20.1 Forge 47.1.0 Parchment 2023.07.23-1.20.1

本文的受众对象:  

- 对Minecraft有着基本认识和了解的读者

- 对Fabric或Forge开发比较熟悉的读者

- 有独立阅读相关文档, 解决问题能力的读者

- 曾经接触过渲染, 或者阅读过笔者有关渲染的文章

## 1.19.3 到 1.20.1 究竟发生了什么

实际上, 帖子评论中所谓优化并不是空穴来风, Mojang在这几个版本中对代码结构进行了大量优化, 对渲染相关代码进行了拆分和整理, 最终诞生了GuiGraphics

过去, 对于渲染的过程, 我们经常要与PoseStack, GuiComponent, RenderSystem, AbstractGui这几个概念打交道, 经常的, 我们有下面这样的逻辑.

```
从渲染上下文中获取到PoseStack实例, 对PoseStack堆栈进行操作
调用RenderSystem的一些方法修改OpenGL状态机
使用GuiComponent封装好的一些方法绘制Minecraft元素 (物品, 文字, 等等)
使用AbstractGui的blit方法绘制贴图
```

实际上, PoseStack, GuiComponent, RenderSystem, BufferSource几个概念往往是同时出现, 其逻辑又是强耦合的, Mojang当然也意识到了这些.

在 1.19.3 -> 1.19.4 版本迭代中, GuiComponent中所有的方法都变成了静态方法, 这为逻辑拆分做了准备. renderOutline, blitNineSliced, setBlitOffset, getBlitOffset 等方法被拆分, 并入PoseStack类中, offset的概念转变为PostStack的z轴概念.

在 1.19.4 -> 1.20 版本迭代中, Mojang决定淡化GuiComponent, RenderSystem的概念, 将有关PoseStack操作的方法封装为一个GuiGraphics类.

实际上, GuiGraphics可以看作是为PostStack进行了具象化(使其具有Minecraft风格).

不难猜想, PoseStack实例成为了GuiGraphics对象的成员变量, 曾经GuiComponent.doSomething(poseStack)的形式已经变为了guiGraphics.doSomething().

实际上, 这次优化是对多对耦合关系进行了集成和封装, 之后, 我们只需将视线放在GuiGraphics类身上, 便可以解决渲染的大部分问题了.

限于篇幅, 这里笔者仅举一例. 在新GuiGraphics类中, 出现了一个名为flush的方法, 它实现了在关闭深度测试的情况下处理BufferSource.

```java
public void flush() {
    RenderSystem.disableDepthTest();
    this.bufferSource.endBatch();
    RenderSystem.enableDepthTest();
}
```

bufferSource也是GuiGraphics的成员变量, 这使得 填充缓存 和 清空缓存 在渲染上下文中变得一气呵成, 方便了许多.

> 实际上, flush的逻辑在GUI批渲染中大量出现, 因为大多数情况下二维渲染不需要深度测试, 深度测试某种意义上来说是一个三维概念.

> 关于BufferSource和VertexConsumer, 笔者这里暂不展开讨论, 将作为进阶思考内容在后日谈中与读者深入探讨

在我们日常游玩中, 附魔台中灵活运动的书本的渲染就在新版本得到了改善.

![enchant.gif](https://s2.loli.net/2023/10/02/vkt9ruxPWGpwQB2.gif)

```java
// 1.20.1 : net.minecraft.client.gui.screens.inventory.EnchantmentScreen
protected void renderBg(GuiGraphics pGuiGraphics, float pPartialTick, int pMouseX, int pMouseY) {
    pGuiGraphics.blit(ENCHANTING_TABLE_LOCATION, i, j, 0, 0, this.imageWidth, this.imageHeight);
    this.renderBook(pGuiGraphics, i, j, pPartialTick);
}

private void renderBook(GuiGraphics pGuiGraphics, int pX, int pY, float pPartialTick) {
    pGuiGraphics.pose().pushPose();
    pGuiGraphics.pose().translate((float)pX + 33.0F, (float)pY + 31.0F, 100.0F);
    VertexConsumer vertexconsumer = pGuiGraphics.bufferSource().getBuffer(this.bookModel.renderType(ENCHANTING_BOOK_LOCATION));
    this.bookModel.renderToBuffer(pGuiGraphics.pose(), vertexconsumer, 15728880, OverlayTexture.NO_OVERLAY, 1.0F, 1.0F, 1.0F, 1.0F);
    pGuiGraphics.flush();
    pGuiGraphics.pose().popPose();
}
```

```java
// 1.19.3 : net.minecraft.client.gui.screens.inventory.EnchantmentScreen
protected void renderBg(PoseStack pPoseStack, float pPartialTick, int pX, int pY) {
    this.blit(pPoseStack, i, j, 0, 0, this.imageWidth, this.imageHeight);
    pPoseStack.pushPose();
    pPoseStack.translate((1.0F - f1) * 0.2F, (1.0F - f1) * 0.1F, (1.0F - f1) * 0.25F);
    MultiBufferSource.BufferSource multibuffersource$buffersource = MultiBufferSource.immediate(Tesselator.getInstance().getBuilder());
    VertexConsumer vertexconsumer = multibuffersource$buffersource.getBuffer(this.bookModel.renderType(ENCHANTING_BOOK_LOCATION));
    this.bookModel.renderToBuffer(pPoseStack, vertexconsumer, 15728880, OverlayTexture.NO_OVERLAY, 1.0F, 1.0F, 1.0F, 1.0F);
    multibuffersource$buffersource.endBatch();
    pPoseStack.popPose();
}
```

首先, 新版本将PoseStack, BufferSource并入GuiGraphics, 关于这一点我们不必多说

之后, 新版本把blit统一放入GuiGraphics, 而不是作为GuiComponent的内部方法

最后, 新版本在关闭深度测试以后再对Buffer进行批处理(flush方法), 优化了部分性能

总结一下, 新版本对于渲染系统的改进主要在提升集成度和规范代码风格上, 对于性能优化有部分改进, 但没有大改. 对于使用过去渲染类教程的读者, 相信阅读到这里已经知道如何将之前自己熟悉的各类方法和函数迁移到新版的GuiGraphics上了

> 实际上, Mojang还在1.20对光照系统进行了大改, 修改后的光照系统的性能有较大幅度的提升, 但由于篇幅和笔者技术水平原因, 这里暂不展开讲解, 感兴趣的读者可以自行查阅, 或者到致谢部分查看具体的技术细节, 以便进一步查阅了解

## Let's talk about Tooltips

实际上, 无论是GuiComponent还是GuiGraphics, 其名字中从未消失的就是GUI. 在日常游戏中, 我们常把 可以打开的界面 称作是GUI, 但在开发层面, 大多数游戏界面中的2D元素都可以看作是GUI. 这里笔者希望以一个常见的2D元素 —— Tooltips为例, 带领读者从实例出发, 由浅入深了解新版本渲染.

当然, 渲染是一个很深的话题, 这里笔者仅选取常见需求的一个剖面, 希望读者阅读过后能对流程有一个较为熟悉的认知, 产生自己探索更多知识的能力.

话不多说, 这里笔者先放出最终效果图, 激发读者的阅读兴趣.

![sword.png](https://s2.loli.net/2023/10/02/kctUTqpYGB2hxR6.png)
![chestplate.png](https://s2.loli.net/2023/10/02/T2SvB9NmfhCWGxX.png)

从图片中, 读者能看到多少自己熟悉的渲染内容呢? 下面请读着跟随笔者的步伐，一步一步完成精美的Tooltip, 并且探究其中的奥妙

### 欲穷千里目

为了绘制我们自己的Tooltip元素, 我们首先需要去除原版的样式. 关于Tooltip, Forge很贴心的为我们准备了RenderTooltipEvent的事件, 通过监听RenderTooltipEvent, 我们可以对Tooltip元素进行一定程度的自定义

查阅RenderTooltipEvent的源码, 我们发现下面这三部分:

- RenderTooltipEvent.GatherComponents 在Tooltip组件被添加的时候触发, 在这个阶段通常可以自定义修改Components列表

- RenderTooltipEvent.Pre 在Tooltip被渲染前触发, 这个阶段我们可以拿到GuiGraphics上下文, 或者调整Tooltip的位置

- RenderTooltipEvent.Color 在Tooltip渲染背景和边框时触发, 这个阶段我们可以修改背景和边框的颜色, 或者进行一定程度的颜色自定义

不难发现, 按照描述, 我们应该监听RenderTooltipEvent.Color, 在渲染背景和边框的时候取消掉原来的颜色, 并且插入我们的渲染逻辑.

先创建ClientEventHandler类, 按照我们的想法尝试一下

```java
// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.ClientEventHandler

@Mod.EventBusSubscriber(modid = ForgeDev.MODID, bus = Mod.EventBusSubscriber.Bus.FORGE, value = Dist.CLIENT)
public class ClientEventHandler {
    @SubscribeEvent(priority = EventPriority.LOWEST)
    public static void colorTooltip(RenderTooltipEvent.Color event) {
        TooltipRenderer.removeColors(event);
    }
}
```

```java
// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.render.TooltipRenderer

public static void removeColors(RenderTooltipEvent.Color event) {
    // make the border and background transparent here so that we can redraw it later.
    event.setBorderStart(0);
    event.setBorderEnd(0);
    event.setBackgroundStart(0);
    event.setBackgroundEnd(0);
}
```

这里我们订阅事件, 并且调用event实例的相关方法, 成功去掉了Tooltip的边框和背景, 现在我们进入游戏, 应该是这样的效果: 

![transparent.png](https://s2.loli.net/2023/10/12/gjw9lP7VdfoB4cz.png)

光秃秃的下界合金剑显然并不美观, 笔者想要为我们的剑添加一个娘化形象悬浮在左侧, 我们先来看一下效果: 

![sword_middle.png](https://s2.loli.net/2023/10/12/ejqN4IUOEcYCz6T.png)

结合上文对GuiGraphics的介绍, 相信有一定经验的读者已经略有头绪, 我们或许可以在GuiGraphics类中找到blit方法, 然后在适当的时候渲染一张图片.

当然, 按照我们的想法, 在RenderTooltipEvent.Color时期渲染这张图片再适合不过了, 我们可以顺理成章的认为这个图片也属于边框(背景)的一部分, 但是真的能如我们所愿吗?

首先, 我们很清楚的是, 现在我们迫切需要GuiGraphics上下文, 只有获得了GuiGraphics实例, 我们才能便利地调用那些封装的成员方法. 不幸的是, event对象中并没有类似getGraphics这样的方法.

不过, 这并不能直接难倒我们, 我们可以尝试观察一下Pre或者GatherComponents事件中是否能提供GuiGraphics上下文, 实际上, 笔者上文已经提过, Pre事件中可以拿到GuiGraphics实例. 拿到以后我们敢不敢存起来用呢, 这里笔者考察了一下RenderTooltipEvent被触发的实际情况.

![pre_event_constructor.png](https://s2.loli.net/2023/10/12/JthXPuSYOUN2Rvf.png)

通过对RenderTooltipEvent.Pre的构造方法的查找用法, 我们找到了ForgeHooksClient中的对应方法, 在进一步查找, 我们找到了位于Minecraft源码**GuiGraphics**中的renderTooltipInternal方法, 很容易可以猜到, 这个方法其实是Minecraft本身真正负责渲染Tooltip的内部方法, 我们的大部分事件都是在合理的位置Hook进去的.

到这里, 其实我们已经不必细致阅读这个内部方法了, 不难看出, 实际上被引用的实例都是 "**this**" 对应的, 我们可以大胆地暂存Pre事件中获取到的GuiGraphics实例的引用地址了.

```java
// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.render.TooltipRenderer
public class TooltipRenderer {
    private static GuiGraphics guiGraphicsContext;

    public static void setGuiGraphicsContext(GuiGraphics guiGraphics) {
        guiGraphicsContext = guiGraphics;
    }
}

// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.ClientEventHandler
public class ClientEventHandler {
    @SubscribeEvent(priority = EventPriority.LOWEST)
    public static void preInitTooltip(RenderTooltipEvent.Pre event) {
        TooltipRenderer.setGuiGraphicsContext(event.getGraphics());
    }
}
```

这里我们已经将GuiGraphics上下文保存在TooltipRenderer类中了, 我们已经可以兴奋地准备进行blit了.

```java
// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.TooltipRenderer
public class TooltipRenderer {
    private static ResourceLocation getPath(String part) {
        return new ResourceLocation(ForgeDev.MODID, "textures/tooltip/" + part + ".png");
    }
    
    private static void innerBlit(ResourceLocation pAtlasLocation, int pX, int pY, int pWidth, int pHeight) {
        guiGraphicsContext.blit(pAtlasLocation, pX, pY, 0, 0, pWidth, pHeight, pWidth, pHeight);
    }

    private static void renderSwordCharacter(int x, int y) {
        guiGraphicsContext.pose().pushPose();
        guiGraphicsContext.pose().translate(0, 0, 400);
        RenderSystem.enableBlend();
        innerBlit(getPath("sword_character"), x - 66, y - 8, 48, 76);
        RenderSystem.disableBlend();
        guiGraphicsContext.pose().popPose();
    }

    public static void tryRenderCharacter(ItemStack itemStack, int x, int y) {
        if (itemStack.getItem() instanceof SwordItem) {
            renderSwordCharacter(x, y);
        }
    }
}

// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.ClientEventHandler
public class ClientEventHandler {
    @SubscribeEvent(priority = EventPriority.LOWEST)
    public static void colorTooltip(RenderTooltipEvent.Color event) {
        TooltipRenderer.removeColors(event);
        TooltipRenderer.tryRenderCharacter(event.getItemStack(), event.getX(), event.getY());
    }
}
```
这里我们主要关注一下**renderSwordCharacter**方法, 相信熟悉渲染的读者已经对这些方法很熟悉了, 我们再次复习一下.

```
pushPose 保存OpenGL状态机原来的状态
translate 对光标相对位置进行偏移
enableBlend 启动Alpha混合, 方便我们渲染透明通道
blit Minecraft为我们封装好的渲染图片的方法
disableBlend 还原Blend状态
popPose 恢复OpenGL状态机的原状态
```

这里笔者对blit进行了简单的封装, 这里相信已经不必多说, 读者已经能够理解这个渲染图片的过程了.

当然, 对于附加元素的渲染在Color阶段当然无可厚非, 这里的X, Y是指Tooltip左上角的坐标. 如果笔者尝试在Pre阶段进行, 是无法获取到有关位置的任何信息的, 实际上, 位置的相关处理是在Pre事件之后进行的.

至此, 读者应该可以进入游戏, 查看效果了.

---

接下来, 我们来给胸甲类物品制作左侧大图展示, 具体的操作方式与绘制图片类似, 但这次我们要调用的是来自GuiGraphics提供的renderItem方法.

```java
// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.TooltipRenderer
public static void tryRenderCharacter(ItemStack itemStack, int x, int y) {
    if (itemStack.getItem() instanceof SwordItem) renderSwordCharacter(x, y);
    else if (itemStack.getItem() instanceof ArmorItem) renderArmorItem(itemStack, x - 54, y + 16);
}

private static void renderArmorItem(ItemStack itemStack, int x, int y) {
    guiGraphicsContext.renderItem(itemStack, x, y);
}
```

进入游戏后, 我们发现效果不尽人意, 显然, 我们需要对PoseStack进行scale操作, 将胸甲渲染的大小进行调整.

查找了renderItem的所有重载后我们尴尬地发现, 没有与scale相关的重载, 我们只好不复用renderItem方法, 而只复用逻辑了.

找到renderItem一系列重载方法的根方法后, 我们剔除了不必要的部分, 并且将this引用改为了我们自己的GuiGraphics上下文.

```java
// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.TooltipRenderer
private static void renderArmorItem(ItemStack itemStack, int x, int y) {
    Minecraft minecraft = Minecraft.getInstance();
    BakedModel bakedmodel = minecraft.getItemRenderer().getModel(itemStack, minecraft.level, null, 0);
    guiGraphicsContext.pose().pushPose();
    guiGraphicsContext.pose().translate((float)(x + 8), (float)(y + 8), 399f);
    guiGraphicsContext.pose().mulPoseMatrix((new Matrix4f()).scaling(1.0F, -1.0F, 1.0F));
    guiGraphicsContext.pose().scale(64.0f, 64.0f, 1.0f); // Enlarge our chestplate.
    if (!bakedmodel.usesBlockLight()) {
        Lighting.setupForFlatItems();
    }

    minecraft.getItemRenderer().render(itemStack, ItemDisplayContext.GUI, false, guiGraphicsContext.pose(), guiGraphicsContext.bufferSource(), 15728880, OverlayTexture.NO_OVERLAY, bakedmodel);
    guiGraphicsContext.flush();
    if (!bakedmodel.usesBlockLight()) {
        Lighting.setupFor3DItems();
    }
    guiGraphicsContext.pose().popPose();
}
```

由于GuiGraphics中的Minecraft实例是私有的, 我们不妨自己获取Minecraft实例, 这里我们调用了熟悉的 `Minecraft.getInstance()` 方法.

现在进入游戏, 相信读者已经看到了想要的效果.

### 更上一层楼

仅仅渲染一张图片显然是不够的, 对于Tooltip, 我们还能做什么炫酷的效果呢. 在开发过程中, 很多开发者希望让自己的模组更有风格化, 希望能够自定义Tooltip的背景或者边框, 很显然, 我们现在似乎已经有了实现的能力.

分析一下需求, 我们需要知道渲染边框的起点位置, 以及Tooltip的宽度和高度. 为了实现边框的大小自适应变化, 我们最好对边框进行切割, 然后按照一定的次序和位置组合起来, 形成一个自适应的整体.

相信阅读过上文的读者已经迫不及待地监听了RenderTooltipEvent.Color方法准备大展身手了, 但是, 事情好像没有那么简单......

当读者试图Color事件读取Tooltip的Width, Height等字段时, 发现这并不是一件可以做到的事情, 我们被迫返回renderTooltipInternal方法里面一探究竟

这里我们部分选取一些对我们有意义的代码来展示, 这里我们为了展示清晰, 也略去了一部分参数

```java
// 1.20.1: net.minecraft.client.gui.GuiGraphics
private void renderTooltipInternal(ItemStack itemStack, ...) {
      
    RenderTooltipEvent.Pre preEvent = ForgeHooksClient.onRenderTooltipPre(...);

    int i = 0;
    int j = pComponents.size() == 1 ? -2 : 0;

    for(ClientTooltipComponent clienttooltipcomponent : pComponents) {
        int k = clienttooltipcomponent.getWidth(preEvent.getFont());
        if (k > i) {
        i = k;
        }

        j += clienttooltipcomponent.getHeight();
    }
    
    Vector2ic vector2ic = pTooltipPositioner.positionTooltip(...);
    
    this.pose.pushPose();
    this.drawManaged(() -> {
        RenderTooltipEvent.Color colorEvent = ForgeHooksClient.onRenderTooltipColor(this.tooltipStack, this, l, i1, preEvent.getFont(), pComponents);
        TooltipRenderUtil.renderTooltipBackground(this, l, i1, i2, j2, 400, colorEvent.getBackgroundStart(), colorEvent.getBackgroundEnd(), colorEvent.getBorderStart(), colorEvent.getBorderEnd());
    });

    this.pose.popPose();
}
```

在 Pre 和 Color 事件之间, 这段对于 ClientTooltipComponent 的迭代引起了我们的注意, 很容易想象, Tooltip的宽度和高度应该是由组成Tooltip的组件(TooltipComponent)决定的, 宽度应该是最宽的一个组件决定的, 高度则是组件高度的叠加. 分析相关逻辑后, 读者应该已经肯定了我们的想法.

很明显, 这里的参数 i 则是整体的宽度, 而 j 则是整体的高度, 令我们沮丧的是, i 和 j 没有被传入 ForgeHooksClient.onRenderTooltipColor 的参数列表中, 这意味着我们没有办法通过监听Forge提供的事件来获得到宽度和高度了. 相信读者已经与笔者不约而同的想到了 **Mixin**, 不过, 我们所想可能略有不同

读者可能会想到, 我们可以直接Hook **TooltipRenderUtil** 类, 对 **renderTooltipBackground** 进行修改, 仔细查看后, 这确实契合我们的需求, 不过这也会导致一个问题.

调用**renderTooltipBackground**时并没有传入itemStack实例, 这意味着itemStack在调用链中丢失了, 在渲染背景元素的时候我们就无法针对itemStack的内容做一些进阶操作.

所以, 笔者考虑的是如何让保留itemStack上下文成为可能, 那显然我们不能脱离renderTooltipInternal的方法体, 最终的答案, 我们要Hook掉GuiGraphics类

首先, 我们需要对Mixin环境进行简单的配置, 对这部分的细节感兴趣的读者也可以查阅笔者之前有关 Mixin 的文章

```js
// 位于: forgedev(opengl-1.20.1) build.gradle
plugins {
    id 'org.spongepowered.mixin' version '0.7-SNAPSHOT'
}

dependencies {
    annotationProcessor 'org.spongepowered:mixin:0.8.5:processor'
}

mixin {
    add sourceSets.main, 'forgedev.refmap.json'
    config 'forgedev.mixins.json'
}
```
```json
// 位于: resources/forgedev.mixins.json
{
  "required": true,
  "refmap": "forgedev.refmap.json",
  "package": "dev.nogu.turou.forgedev.mixin",
  "compatibilityLevel": "JAVA_17",
  "client": [
    "GuiGraphicsMixin"
  ],
  "injectors": {
    "defaultRequire": 1
  },
  "minVersion": "0.8.5",
  "target": "@env(DEFAULT)"
}
```

为了实现需求, 我们仍然需要事先将背景颜色置为透明, 然后使用Mixin来Hook掉 **GuiGraphics** 类, 在 **popPose()** 处插入我们的渲染逻辑

```java
// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.mixin.GuiGraphics
@Mixin(GuiGraphics.class)
public class GuiGraphicsMixin {
    @Final
    @Shadow()
    private PoseStack pose;

    @Inject(method = "renderTooltipInternal",
            at = @At(value = "INVOKE", target = "Lcom/mojang/blaze3d/vertex/PoseStack;popPose()V", shift = At.Shift.AFTER), locals = LocalCapture.CAPTURE_FAILHARD)
    private void renderTooltipInternalMixin(Font font, List<ClientTooltipComponent> components, int x, int y, ClientTooltipPositioner positioner, CallbackInfo ci, RenderTooltipEvent.Pre preEvent, int width, int height, int i2, int j2, Vector2ic postPos) {
        // Do render calls
    }
}
```

使用Final, Shadow注解拿到原来类中 private final 的 PoseStack 实例

Inject的@At可以传入target参数来标识实际的注入位置, 这里我们标识了popPose方法, 由于这个方法中只调用过一次popPose方法, 我们即可精确定位到该位置.

参数表前半段为方法的参数表, 继续向后延伸即可获取到方法体内部的局部变量, 这里不建议使用IDEA的自动补全, 以及LocalCapture.PRINT提供的信息, 他们都不够准确.

基于方法体内部的上下文, 我们可以从方法定义向后延伸, 捕获到 i, j 两个局部变量, 延伸方式大概如下图所示.

![mixin_args.png](https://s2.loli.net/2023/11/12/gzwcWsNpBETik4S.png)

> 可能有读者抱有疑问, 为什么我们要定位popPose()方法的位置, 而不是直接 `at = @At(value = "TAIL")` 呢, 首先, 我们的渲染逻辑应该是在Components判空的作用域范围内的, 如果注入在TAIL, 我们就需要进行二次判空的处理. 其次, 重新审视我们的逻辑, 我们是要在渲染结束后进行Post处理, 在popPose()后面再适合不过, 如果放在TAIL, 虽然最终效果相同, 但造成了更大的影响疮面, 更容易与其他Mixin产生冲突, 这是使用Mixin的良好习惯.

接下来, 我们来分析绘制边框的过程, 为了绘制自适应的边框, 我们肯定需要根据一定的规律拆分边框, 并且分片进行渲染, 才能得到最好的效果.

![depart.png](https://s2.loli.net/2023/11/12/4N6OEn8cuUB3aZy.png)

红色矩形部分应该是固定比例渲染的. 橙色矩形部分是应该根据长宽进行自适应的. 橙色圆形的部分是浮于背后的背景. 红色矩形框住的星星是应该始终居中渲染的.

读者可以根据需要对自己的边框进行切割, 尽量减小拉伸对效果的影响, 这里我们对自适应的部分直接拉伸, 由于大部分自适应的部分是渐变色的直线, 所以几乎不会影响最终效果.

> 对于边框的自适应部分, 读者也可以直接选择绘制原生的渐变色直线, 推荐参考TooltipRenderUtil类的Gradient相关方法

使用 Paint.NET 或 Photoshop 等工具切割缩放后, 我们应该会得到10个透明PNG文件, 这里我们提前将他们丢到 resources\assets\forgedev\textures 目录下

![depart_explorer.png](https://s2.loli.net/2023/11/12/e2MtVc1mfGqWLOa.png)

接下来, 我们需要对这10个贴图按照一定的次序和组织渲染出来.

```java
// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.mixin.GuiGraphic
@Inject(...)
private void renderTooltipInternalMixin(...) {
    TooltipRenderer.tryRenderBorder(preEvent.getItemStack(), pose, postPos.x(), postPos.y(), font, width, height, components);
}

// 位于: forgedev(opengl-1.20.1) dev.nogu.turou.forgedev.TooltipRenderer
public static void tryRenderBorder(ItemStack itemStack, PoseStack pose, int x, int y, Font font, int width, int height, List<ClientTooltipComponent> components) {
    if (itemStack.getItem() instanceof SwordItem) renderFancyBorder(x, y, width, height);
}

private static void renderFancyBorder(int x, int y, int width, int height) {
    guiGraphicsContext.pose().pushPose();
    guiGraphicsContext.pose().translate(0, 0, 400); // beneath the text and upon other items
    RenderSystem.enableBlend();

    // render for border
    var widthOffset = 2;
    width += widthOffset;
    var horizonStart = x - 16;
    var horizonDuration = width - 55;
    var horizonEnd = x - 30 + width;
    var verticalStart = y - 14;
    var verticalDuration = height - 38;
    var verticalEnd = y - 20 + height;
    var horizonCenter = (int) (horizonStart + 0.5 * width);
    innerBlit(getBorderPath("left_top"), horizonStart, verticalStart, 41, 37);
    innerBlit(getBorderPath("top"), horizonStart + 41, verticalStart + 7, horizonDuration, 4);
    innerBlit(getBorderPath("right_top"), horizonEnd, verticalStart, 40, 37);
    innerBlit(getBorderPath("left_bottom"), horizonStart, verticalEnd, 46, 32);
    innerBlit(getBorderPath("left"), horizonStart + 9, verticalStart + 37, 1, verticalDuration);
    innerBlit(getBorderPath("right"), horizonEnd + 30, verticalStart + 37, 1, verticalDuration);
    innerBlit(getBorderPath("right_bottom"), horizonEnd - 6, verticalEnd, 46, 32);
    innerBlit(getBorderPath("bottom"), horizonStart + 46, verticalEnd + 25, width - 66, 2);
    innerBlit(getBorderPath("stars"), horizonCenter + 7, verticalStart + 8, 12, 3);

    // render for background
    guiGraphicsContext.pose().translate(0, 0, -1); // beneath the text and upon other items (under the border)
    innerBlit(getPath("fancy_bg"), x - 6, y - 3, width + 8, height + 6);

    RenderSystem.disableBlend();
    guiGraphicsContext.pose().popPose();
}
```

相信读者很容易想到笔者这样的代码形式, 但对每一个物件定位的动态思维过程是怎样的呢, 这里笔者希望读者稍作思考.

这里笔者将从几个点出发, 来与读者共同发觉代码中的常量和计算过程是如何得来的.

首先, width 和 height 是由原图切片的大小比例缩放得来的, 这里笔者大约是将原图的宽高都除以14取整, 得到了游戏内较为合适的大小. 笔者在处理自己的边框宽高时, 可以先针对一个部分(比如left_top)进行等比缩放, 找到自己比较满意的转换比例, 然后按照这种方式来缩放所有的部分.

之后, 我们来进行定位, 很容易想到, 我们应该先从left_top部分进行定位, 观察代码块, 很明显这里笔者找到的数值是(-16, -14), 这个点应该是边框的左上角, 我们将这个坐标作为起点, 命名为horizonStart, verticalStart

之后, right_top应该与left_top位于同一高度, 他们应该具有相同的y = verticalStart. 经过多次的修改和调试, 我们能得到horizonEnd对应的数值, 同时计算出horizonDuration的长度.

通过填写确定的部分和推测不确定的部分, 我们最终成功将所有图片以较小的误差拼起.

之后, 通过 `horizonStart + 0.5 * width` 我们可以计算出Tooltip中点的位置, 最终经过一定的offset, 我们能找到stars应该被放置的位置.

> 将 z-axis translate 到400可以保证边框在物品上方, 在文字下方. 我们渲染背景的时候将 z-axis 又向下 offset 了-1, 即此时 z-axis 为399, 399也能保证在物品上方和文字下方, 同时在边框下方.

> 这里我们修改 offset 查看效果的过程主要用到了调试. 当我们使用调试按钮启动客户端的时候, 如果修改一些常量和方法体内部的逻辑, 可以直接应用更改而无需重启游戏.

![debug_client.png](https://s2.loli.net/2023/11/12/8uCUlOMPnWkcVXb.png)

![reload_class.png](https://s2.loli.net/2023/11/12/5qypBTZCP32u74X.png)

至此, 我们已经成功将具有科技感的边框添加到所有剑类物品上.

## 后日谈

> 注: 这里的部分知识属于较为高阶的知识, 其中涉及到渲染的原理、计算机图形学等知识, 读者可以选择阅读.

首先, 阅读过笔者先前TileEntitySpecialRender相关文章的读者可能对OpenGL渲染顶点的过程有一定的了解:

![Snipaste_2021-02-06_12-28-05.png](https://s2.loli.net/2023/11/12/zQXr9ijxZeyLP3O.png)

在渲染过程中, **数据**与**指令**是分离的, 指令标识了读取数据的方式. 我们适时执行 draw(), 告知OpenGL可以开始按照指令开始渲染了.

```

我们可以收集四个顶点, 调用 draw() 绘制一个矩形 (此时draw方法调用一次)

我们可以重复上面这个操作两次, 绘制两个矩形 (此时draw方法调用两次)

我们可以收集八个顶点, 调用 draw() 绘制两个矩形 (此时draw方法调用一次)

```

在可以允许的范围内, 我们希望尽量少的调用draw方法以提升性能, 我们可以尽量多的收集信息, 并且一次性将他们提交.

> 3TUSK: 这个效果实际上也正是旧版 Forge 提供的 FastTESR 的效果。当 TileEntity 的数量大到一定程度的时候，反复调用 Tessellator.draw() 会显著影响性能。通过批量 (“batch”) 收集顶点数据，我们可以将调用 draw() 的开销降到最低。

在FastTESR中, 我们已经把可允许的范围扩大到TileEntity的一次tick, 那么这种范围在原版的渲染中有没有体现呢, 那必然是存在的

> 3TUSK: 题外话，当年给 Forge 写出 FastTESR 的 RainWarrior（或者，我们更多称他为 fry） 现在在 Mojang 工作有三年多了。虽然没有证据，但 Modder 们普遍认为是他把这玩意写进了原版底层，因为他也是当时 Forge 开发团队中唯一精通渲染的开发者，同时也是 Forge 那些渲染相关类库的作者。

重新展开上文我们提到的渲染附魔台GUI界面的部分代码, 我们应该能看到这种逻辑的影子

```java
private void renderBook(GuiGraphics pGuiGraphics, int pX, int pY, float pPartialTick) {

    VertexConsumer vertexconsumer = pGuiGraphics.bufferSource().getBuffer(this.bookModel.renderType(ENCHANTING_BOOK_LOCATION));
    this.bookModel.renderToBuffer(pGuiGraphics.pose(), vertexconsumer, 15728880, OverlayTexture.NO_OVERLAY, 1.0F, 1.0F, 1.0F, 1.0F);
    pGuiGraphics.flush();
}
```

VertexConsumer是一个接口, BufferBuilder是这个接口的一个实现, BufferBuilder实际上就是我们上文提到的数据的一个序列.

那RenderType如何理解呢, 我们是根据RenderType生成的BufferBuilder, RenderType实际上就是指令.

### VertexConsumer

这里我们观察一下bufferSource的getBuffer方法

```java
public VertexConsumer getBuffer(RenderType pRenderType) {
    BufferBuilder bufferbuilder = this.getBuilderRaw(pRenderType);
    if (this.startedBuffers.add(bufferbuilder)) {
        bufferbuilder.begin(pRenderType.mode(), pRenderType.format());
    }
    return bufferbuilder;
}
```

非常明显的看到, BufferBuilder引入了基于RenderType的模板(mode, format), 应该是保存顶点数据时进行校验使用的.

进一步查看getBuilderRaw方法可以看到, BufferBuilder对于RenderType单例, 这非常说得通. 相同类型的操作, 当然要合并在一起执行才高效.

```java
private BufferBuilder getBuilderRaw(RenderType pRenderType) {
    return this.fixedBuffers.getOrDefault(pRenderType, this.builder);
}
```

### renderToBuffer

顺着 `renderToBuffer -> render -> root.render -> compile -> cube.compile` 一条线查下去, 我们顺利找到了正在偷偷向VertexConsumer中填充数据的方法, 这符合我们的想法.

![Snipaste_2023-11-12_21-49-30.png](https://s2.loli.net/2023/11/12/FQ1NkOZAVJtYK8X.png)

### flush

这里我们回看我们之前提到的flush方法, 我们可能对bufferSource的存在不再感觉到神秘.

```java
public void flush() {
    RenderSystem.disableDepthTest();
    this.bufferSource.endBatch();
    RenderSystem.enableDepthTest();
}

public void endBatch() {
    for(RenderType rendertype : this.fixedBuffers.keySet()) {
        this.endBatch(rendertype);
    }
}
```

至于 fixedBuffers 是什么, 为什么以 RenderType 为单位进行批量渲染, 相信阅读到这里的读者心中已经自然得到答案了.

## 尾声

本文的引子是 GuiComponent的消逝与GuiGraphics的新生, 但是笔者在创作过程中又旁伸出了许多与渲染有关的实用知识与底层知识, 虽然本文不能带领读者完全走进渲染的世界, 不能对渲染系统的方方面面加以解读, 但是笔者希望借此方式使读者对渲染产生兴趣, 同时对渲染不再胆怯. 除此之外, 希望读者也能在笔者的耳濡目染之下建立起正确的思考方式, 从而能够按照这种思维范式解决自己的问题, 实现自己的需求.

这里特别感谢:

3TUSK: 在TESR文章下方的回复对笔者产生了巨大启发

shblock_(Integrated Proxy): 对渲染底层的一部分解读, 解答笔者的疑惑

BloCamLimb: 一些渲染问题的帮助, 精神支持

DAYGood_Time: 与笔者共同讨论Mojang的前世今生, 提供临时 1.19.3 开发环境