---
title: 由现象到本质的Minecraft源码注入艺术
date: 2021-06-07 11:19:59
tags:
    - Minecraft
    - 模组开发
---

## 地狱放水? 吟唱魔法? 储物工作台? 人生苦短, 我要Mixin

---

Mixin以其易用性与强大性, 逐渐取代了传统意义上的CoreMod编写流程, 这使得将自己的代码注入到Minecraft的内部变得十分容易且便利. 这里笔者将从三个从浅入深的例子入手, 带领读者快速入门Mixin, 欣赏Minecraft源码的注入艺术

本文推荐使用开发环境:  

开发工具: IntelliJ IDEA with Minecraft Devlopment Plugin

开发环境: Fabric Loader 0.11.1 with fabric-api 0.30.0

Mapping版本: Yarn-1.16.5 build.4:v2

游戏版本: Minecraft 1.16.5 with Fabric

本文的受众对象:  

- 对Minecraft有着基本认识和了解的读者

- 曾经有进行过Fabric或Forge开发的读者

- 有独立阅读相关文档, 解决问题能力的读者

- 对Mixin的工作原理, 使用方式**简单了解**的读者

---

## 一、揭开Minecraft的面纱 - 水桶美学

### 1. 慧眼观察

本节, 我们将以一个十分经典的需求: 在地狱放水, 打开读者的思路, 培养读者阅读源码的能力, 并且对Mixin建立基本的认识.

相信读者在以往的开发过程中常常会不经意间用到IDEA的内置反编译功能, 当我们对某一方法或类抱有疑问时, 我们常常会Shift左击来查看某个原版方法或类的定义. 如果我们试图对我们查看到的源码进行更改时, IDEA会提示我们, 该文件是只读的, 我们深切的知道, 已经编译到class的文件是不能被直接修改的.

但正是Mixin的出现, 使这种更改成为了可能, 我们可以利用一些表达方式来在class文件被Java虚拟机执行前修改它, 所以我们的工作就需要先利用IDEA的反编译功能定位代码, 之后用Mixin进行修改.

说到这里, 相信聪明的读者已经对我们的定位工作有了头绪, 很容易想到, 地狱不能放水的原因就出在我们用水桶在地狱放置的一瞬间, 所以我们只需在水桶交互的实现方法中找到这部分就可以了.

这里我们使用IDEA的查找功能, 双击Shift, 我们可以打开一个快速搜索框

这里我们搜索水桶Bucket, 我们可以找到原版Item包下的BucketItem类, 这里要注意的是, 我们要选中Include non-project items. 不然的话Minecraft包内的方法是没办法查找到的

![](https://i.loli.net/2021/04/07/QpLDCKqPjFhVNcB.png)

这里我们直接双击, 让IDEA用内置的反编译器来查看BucketItem类的源码

相信细心的读者很快就能找到对应的条件判断语句, 我们在placeFluid方法中找到了我们想要操作的目标,我们发现,这与我们印象中地狱放水的现象相似(先播放桶倒水的声音, 之后在生成烟的粒子效果)

![](https://i.loli.net/2021/04/07/SmsMcXEVBdwjKq4.png)

这里Dimension, Ultrawarm等字样引起了我们的注意, 我们进一步向下查阅发现, 这个isUltrawarm方法返回了成员变量ultrawarm.

![](https://i.loli.net/2021/04/09/hfHTpryCaibUqez.png)

不难猜想, 这个成员应该是DimensionType类初始化时传入的, 而代表地狱的DimensionType创建时参数可能传入了true, 继续向下阅读, 我们找到了具体的实例化的代码, 这证实了我们的猜想

![](https://i.loli.net/2021/04/09/48zRFVMT6Li7aJj.png)

所以最简单的解决方案就清晰了, 我们需要修改的正是这个isUltrawarm方法. 使其一律返回false, 这样就允许了所有维度水的放置.

#### 2. 动手实践

相信对Mixin有简单了解的读者已经动手写出自己的代码了, 这里笔者仅简单提几点注意事项, 以供对Mixin了解不深的读者参考, 相信你们会在之后的学习中逐渐理解Mixin的实质

- 所有Mixin的相关"类"必须放在一个独立的包下, 并且在对应json中明确声明

- Mixin类需要使用@Mixin来修饰, 参数提供想要注入的类

- 被@Mixin修饰的类, 在实质的角度上来说不是类, 而是指导代码注入过程的一个说明

- @Overwrite表示用下列代码完全替换目标方法的内容

这里笔者放出自己的代码, 并简要说明一下定位与修改的过程.

```java
@Mixin(DimensionType.class) // 只有被@Mixin修饰的类才会被识别并注入
public class DimensionTypeMixin {
    /**
     * @author TuRou
     * Make water can place at all dimensions.
     */
    @Overwrite() // 将目标方法完全替换为下方的代码
    public boolean isUltrawarm() {
        return false; // 所有维度都返回false, 包括地狱
    }
}
```

只需要短短的五行代码, 我们便达成了我们的目的, 简单地修改一个Minecraft的内部方法. 之后将Mixin类添加到对应json文件中就可以了

![](https://i.loli.net/2021/04/07/G4bQY6fZJ3oW9kI.png)

package指向所有Mixin文件的存放包, mixins是存放所有Mixin类的列表. 最后不要忘记在主json中加入mixins.json的引用, 之后我们就可以进游戏测试了. 在之后部分的文章中, 将不会再提及这部分内容, 如果读者发现自己的Mixin没有被正确加载, 不妨检查一下有没有将其添加到mixins中.

![](https://i.loli.net/2021/04/09/U2dbxGQTF1HC7th.png)

---

## 二、方块与魔法的世界 - 吟唱施法

本部分将以聊天栏吟唱施法为实例, 使读者对定位注入的位置, 以及源代码的感知进入一个更深层次的境界, 同时启发读者产生更多的想法和点子. 我们将从聊天栏消息的注入出发, 深入浅出揭秘Minecraft聊天的处理方式, 这里也包含一小部分Fabric网络通信的方式, 读者可以选择性阅读.

### 1. 明确需求

作为一名出色的魔法术士, 我们应有流畅的魔法吟唱的能力, 在Minecraft世界中, 我们可以用聊天的形式呈现, 在聊天栏中说出对应魔咒, 我们就可以触发相应的魔法, 产生一些特殊的效果. 我们希望自己的魔咒不在公屏上显示, 甚至我们输入魔咒的时候会发出悦耳的声音, 并且也不允许玩家使用方向上键来直接输入历史魔法...这样, 我们可以做出一个趋于完善的吟唱施法系统. 这里我们重新梳理一下需求

1. 拦截聊天栏魔咒消息的发送, 判断施法是否成立

2. 在输入魔咒时发出悦耳的声音

3. 禁止魔咒消息储存至消息历史

如此这样, 我们把目光主要定位在了**聊天**的相关实现上, 我们想要寻找的类, 大概率包含Chat, Message字样. 之后, 我们就可以开始着手分析Minecraft关于聊天相关实现的源码了.

#### 2. 渐入佳境

搜索后我们发现, 我们找到了三个可疑目标, 分别是ChatHud, ChatMessages, ChatScreen

![](https://i.loli.net/2021/04/10/qrYTpGudO8S1if7.png)

经过简单的分析我们发现:

- ChatMessages主要负责聊天信息中文字的渲染, 这对我们的工作没有帮助

- ChatScreen中绘制了我们聊天栏的GUI, 聊天信息的发送可能在这里被处理

- ChatHud着重于聊天栏的文本部分, 我们聊天信息的历史可能保存在这里

为了印证我们的猜测与分析, 我们尝试依靠我们的直觉来寻找我们需要的代码. 让我们回想在游戏中我们发送聊天信息的操作流程. 我们首先按T打开聊天栏, 在文本框中输入内容, 最后**单击回车发送消息**.

细心的读者可能已经发现, 这个按T打开的聊天栏正是ChatScreen所描述的界面, 而文本框就是ChatScreen类中发现的一个名为chatField的属性. 而点击回车这个动作, 在keyPressed方法中有所体现.

```java
String string = this.chatField.getText().trim();
if (!string.isEmpty()) { // 如果不为空
    this.sendMessage(string);
}

this.client.openScreen((Screen)null); //关闭界面
return true;
位于: net.minecraft.client.gui.screen.ChatScreen
```

不难猜想, 我们需要深入探究sendMessage是以怎样的方式发送玩家的聊天信息的, 我们查询相关实现, 发现了如下的代码.

```java
public void sendMessage(String message, boolean toHud) {
    if (toHud) {
        this.client.inGameHud.getChatHud().addToMessageHistory(message);
    }

    this.client.player.sendChatMessage(message);
}
位于: net.minecraft.client.gui.screen.Screen
```

这里我们同时有了两个发现, 发送消息之前, 会调用ChatHud类中的addToMessageHistory方法来将该聊天添加到聊天历史中, 发送消息时调用ClientPlayerEntity的sendChatMessage方法来以客户端玩家的身份将消息发送至server-side. 这里不难猜想是发送了一个C2S的网络包

```java
public void sendChatMessage(String message) {
    this.networkHandler.sendPacket(new ChatMessageC2SPacket(message));
}
位于: net.minecraft.client.network.ClientPlayerEntity
```

接下来我们的工作便清晰了, 注入这两个方法并按照我们的需求进行修改

#### 3. 唯手熟尔

在我们的mixin包中新建ClientPlayerEntityMixin类, 写入如下内容

```java
@Mixin(ClientPlayerEntity.class)
public class ClientPlayerEntityMixin {
    @Inject(method = "sendChatMessage", //要注入的方法sendChatMessage
            at = @At("HEAD"), //表明插入的位置在方法的头部
            cancellable = true //表明我们可以中途取消(return)这个方法
    )
    private void sendChatMessage(String message, CallbackInfo ci) {
        if (message.toLowerCase().startsWith("maho shoot")) {
            ClientPlayNetworking.send(ChantConstant.CHANT_IDENTIFIER, PacketByteBufs.create().writeString(message));
            //将施法网络包发送, 这里会在下面部分进行讲解
            ci.cancel();
        }
    }
}
```

这个类被@Mixin修饰, 意味着它是一个用来补充ClientPlayerEntity内容的"类", 与上文不同的是, 这里我们对想要修改的方法添加了@Inject注解, 而不是@Overwrite注解, 相信读者已经通过字面意思猜到了二者之间的区别

@Inject注解表示将下面的代码**插入**到目标方法内, 我们在目标方法("sendChatMessage")的头部("HEAD")插入一个可取消的(cancellable = true)的方法.

方法体内部, 我们判断message内容是否以"maho shoot"开头, 如果是的话, 就把施法的动作发送给server-side(这里涉及到一个C2S的网络包, 会在下文讲解), 以便于我们在逻辑服务端部分处理施法的具体效果. 最后, 我们调用ci.cancel();方法, 代表我们希望在这里调用return()结束这个方法, 不再继续向下执行, 因为我们不想让施法的聊天消息再发出了.

所以在游戏运行时, Minecraft会把我们的sendChatMessage方法当成是这样的:

```java
public void sendChatMessage(String message) {
    if (message.toLowerCase().startsWith("maho shoot")) {
        ClientPlayNetworking.send(ChantConstant.CHANT_IDENTIFIER, PacketByteBufs.create().writeString(message));
        return();
    }
    this.networkHandler.sendPacket(new ChatMessageC2SPacket(message));
}
```

这符合我们的预期效果. 接下来我们要做的事情就是在server-side处理我们这个网络包, 这已经超出我们本节教程的范围, 会在下文简略介绍. 接下来我们来处理messageHistory.

在我们的mixin包中新建ChatHudMixin类, 写入如下内容

```java
@Mixin(ChatHud.class)
public class ChatHudMixin {
    @Inject(method = "addToMessageHistory",
            at = @At("HEAD"),
            cancellable = true
    )
    public void addToMessageHistory(String message, CallbackInfo ci) {
        if (message.toLowerCase().startsWith("maho shoot")) {
            ci.cancel();
        }
    }
}
```

相信这里笔者不用加以解释, 读者已经充分理解并明白了这段代码的含义和作用.

最后, 我们来完成第二个需求, 在输入魔咒时发出悦耳的声音. 先前认真阅读过ChatScreen类的读者可能会发现这样一个方法

```java
private void onChatFieldUpdate(String chatText) {
    String string = this.chatField.getText();
    this.commandSuggestor.setWindowActive(!string.equals(this.originalChatText));
    this.commandSuggestor.refresh();
}
位于: net.minecraft.client.gui.screen.ChatScreen
```

看起来这个类原本是为了在输入指令的时候进行自动补全的, 这里刚好迎合了我们的需求, 有了之前的经验, 现在我们很清晰地明白, 我们将使用Inject注解在这个方法中填上有关播放音效的代码.

在我们的mixin包中新建ChatScreenMixin类, 写入如下内容

```java
@Inject(method = "onChatFieldUpdate",
        at = @At("RETURN") //这里我们尝试在方法结束前插入这段代码
)
public void onChatFieldUpdate(String message, CallbackInfo ci) {
    ClientPlayerEntity player = MinecraftClient.getInstance().player;
    if (message.toLowerCase().startsWith("maho shoot ") && player != null) {
        player.playSound(SoundEvents.ENTITY_EXPERIENCE_ORB_PICKUP,0.1f, 0.8f);
    }
}
```

方法体内部, 我们用MinecraftClient.getInstance().player获取到的玩家对象是运行在逻辑客户端上的玩家, 因为我们仅仅是想播放声音, 这种操作是可取的.

之后, 我们判断聊天信息是否是以施法的前缀"maho shoot"开头的, 如果是这样的话, 就播放经验球捡起的声音. 需要注意的是, 这个声音因为是在客户端玩家对象上播放的, 固然只有施法的玩家自己能听见, 若想要让服务器中附近的所有玩家听到, 也要用我们上文类似的方法发送一个C2S网络包, 并让播放音效的语句在逻辑服务端执行.

最后, 将我们新建的两个Mixin类添加到mixins.json中, 我们就可以打开游戏测试了.

#### 4. 请问您今天要来点额外知识吗？

##### Client/ServerPlayNetworking

上文我们调用了`ClientPlayNetworking.send(ChantConstant.CHANT_IDENTIFIER, PacketByteBufs.create().writeString(message));`方法, 将施法的网络包发给了server-side. 这里我们提供了两个参数,一个是我们要发送的字节集数据, 另一个是网络包的标识符. 这里我们的标识符在一个名为ChantConstant的类中定义`public static final Identifier CHANT_IDENTIFIER = new Identifier("shinto", "chant");`

接下来我们来到服务端侧, 来看看处理施法的过程

新建network包, 创建ServerPacketHandler类, 写入以下内容

```java
public class ServerPacketHandler {
    public static void init() {
        ServerPlayNetworking.registerGlobalReceiver(ChantConstant.CHANT_IDENTIFIER, (server, player, handler, buf, responseSender) -> {
            String str = buf.readString(32767);
            server.execute(() -> {
                new StatementResolver().resolve(str, player);
            });
        });
    }
}
```

这里我们调用ServerPlayNetworking类来注册一个监听器, 等到网络包一到达, 我们读取其中的字符串, 并且用服务端线程执行施法的相关代码, 这里笔者施法的解析放在了StatementResolver类中, 对笔者魔法Mod感兴趣的读者可以前来阅读

##### @Inject注解的locals参数

当我们注入一个方法时, 可能会有需要访问方法内的局部变量的时候, 虽然本教程尚未涉及, 这里笔者也希望给予补充. 这种情况下, 我们通常会给@Inject注解添加locals参数, 这样我们就可以捕获局部变量表了.

比如说我们可以让@Inject注解变成这个样子

```java
@Inject(method = "myFunction",
 at = @At("HEAD"), 
cancellable = true, 
locals = LocalCapture.CAPTURE_FAILHARD //我们需要捕捉这个方法的局部变量表
 //FAILHARD表示一旦捕捉失败, 游戏不会继续加载, 发生崩溃
 //FAILSOFT表示一旦捕捉失败, 游戏继续加载, 放弃注入这个方法
)
```

那么我们为什么要用这种复杂的方式呢? 简要来说, 局部变量表具有不确定性, Mixin只能根据字节码推测局部变量, 这导致当其他Mod一旦与你有共同的需求, 都**注入了同一个方法**并且**给这个方法添加了局部变量**, 事情可能就会复杂起来, 因为此时局部变量表对于两个Mod双方在游戏实际跑起来之前都是不确定的. 所以双方都需要做好一旦方法注入失败之后的打算.

所以, 我们的局部变量列表并不是所见即所得的, 而且我们也不能通过变量的确切名字来访问变量(因为Mixin也不知道这个名字究竟指哪个变量, Mixin只有一个标记着数据类型的有序列表), 所以推荐读者第一次注入该方法时将`locals`设为`LocalCapture.PRINT`，以便确定具体的局部变量表, 如果知道可能产生冲突的Mod, 可以将那个Mod加入进开发环境, 再重新PRINT并编写重载后的代码, 将可能冲突的情况避免。mixin会告诉你此时的方法应该怎么写, 读者可以加以尝试

这里笔者推荐读者阅读[CoreModTutor中的相关内容](https://xfl03.gitbook.io/coremodtutor/5/5.3), 结合Minecraft原版中的具体例子进行理解, 这里笔者就不过多赘述了.

> 注: 这里笔者特别感谢xfl03的CoreModTutor中对Mixin系统而详细的介绍, 笔者的知识储备部分来源于此, 当读者希望对核心概念有更深理解时, 可以阅读xfl03的[CoreModTutor]([5 Mixin - CoreModTutor (gitbook.io)](https://xfl03.gitbook.io/coremodtutor/5))

---

## 三、关于工作台上现在可以保持物品不掉下来这档事

本部分将为读者展现一个非常常规的需求: 将物品保持在工作台中不掉落. 在探究过程中, 我们将继续沿用我们上面现象到本质的探究思路, 从工作台本身出发, 一步步探求其背后的代码实现以及注入方案, 培养读者探究一个未知问题的能力.

### 1. 切入要点

同样的, 我们利用idea的搜索功能搜索Crafting关键字, 并且筛选我们可能需要用到的类, 我们最后留下了下面这些看起来与我们需求有关的类

- CraftingTablework Block类的子类, 定义了工作台这个方块

- CraftingScreen Screen类的子类, 定义了合成界面是如何渲染的

- CraftingScreenHandler 处理合成界面的类, 可能与创建合成GUI有关

- CraftingInventory 处理合成界面九宫格上的物品的类

经过对代码简单的阅读, 我们先排除掉了CraftingScreen类, 因为CraftingScreen类中大多是关于界面渲染有关的方法和语句, 这显然与我们的需求无关.

之后为了理清各个类之间的关系, 我们开始按照我们的思路分析问题. 设想一下, 我们在游戏中使用工作台的第一个动作, 就是对工作台右键单击, 我们可以以此为切入点, 看看右击工作台会发生什么. 在CraftingTableBlock类的onUse方法中, 调用了createScreenHandlerFactory来打开合成的GUI界面.

```java
public NamedScreenHandlerFactory createScreenHandlerFactory(BlockState state, World world, BlockPos pos) {
    return new SimpleNamedScreenHandlerFactory((i, playerInventory, playerEntity) -> {
        return new CraftingScreenHandler(i, playerInventory, ScreenHandlerContext.create(world, pos));
    }, TITLE);
    //在玩家的GUI中显示新创建的CraftingScreenHandler界面
}
位于: net.minecraft.block.CraftingTableBlock
```

可以看到, 这里右键单击时, 是实例化了一个CraftingScreenHandler类, 在这个类的构造方法中, 我们可以找到下面这样的语句

```java
public CraftingScreenHandler(int syncId, PlayerInventory playerInventory, ScreenHandlerContext context) {
    super(ScreenHandlerType.CRAFTING, syncId);
    this.input = new CraftingInventory(this, 3, 3);
    //在构造方法中创建一个3*3的CraftingInventory, 用来存放待合成品
    ......
}
位于: net.minecraft.screen.CraftingScreenHandler
```

当我们继续向下查询CraftingInventory类, 我们发现了下面这样的代码

```java
private final DefaultedList<ItemStack> stacks;
public CraftingInventory(ScreenHandler handler, int width, int height) {
    this.stacks = DefaultedList.ofSize(width * height, ItemStack.EMPTY);
    //CraftingInventory中维护一个有序列表, 来存放一些Itemstack
    ......
}
位于: net.minecraft.inventory.CraftingInventory
```

相信聪明的读者已经从这些代码中明确了三个类之间的关系.

我们期望CraftingScreenHandler类中能有一个类似于setInputItems()和getInputItems()的方法, 如果这样的话, 我们只需要在关闭界面的时候把get到的物品列表储存在方块内, 并且在打开界面的时候把物品列表set进去就可以了, 我们清楚的知道, mixin应该能帮助我们做到这一点.

我们的思路已经基本明确, 现在的首要任务便是利用mixin为我们添加进去这一组getset

#### 2. 做出尝试

在我们的mixin包中创建CraftingScreenHandlerMixin类, 按照我们的直觉, 我们的代码可能是这样的

```java
@Mixin(CraftingScreenHandler.class)
public class CraftingScreenHandlerMixin {
    private DefaultedList<ItemStack> getInputItems() {
        return input.stacks;
    }

    private void setInputItems(DefaultedList<ItemStack> itemStacks) {
        if (itemStacks != null && !itemStacks.isEmpty()) {
            for (int i = 0; i < itemStacks.size(); i++) {
                input.stacks.set(i, itemStacks.get(i));
            }
            onContentChanged(null); //为了让合成产物更新, 改变完物品列表后应该手动进行刷新
        }
    }
}
```

idea此时为我们报出了许多错误, 其中最为明显的就是找不到input这个变量, 我们清楚地记得, input这个成员是在CraftingScreenHandler中明确定义了的, 为什么这里却找不到了呢...

了解Java的读者此时可能已经意识到了事情的不对, 虽然我们目标类中有input这个成员, 但是对于Java编译器, CraftingScreenHandlerMixin中确确实实没有声明input, 这里mixin为我们提供了一种方法, @Shadow注解

@Shadow注解意味着被标记的成员或方法在目标类中出现. 当我们需要引用这个成员或方法的时候, 我们可以用@Shadow注解来欺骗Java编译器, 所以我们在代码中添加下面几行

```java
@Shadow @Final private CraftingInventory input;
//这里还添加了@Final注解, 这个很好理解, 目标类中原本被final修饰的成员推荐在mixin类中添加@Final注解, 这样当你不小心更改了这个成员的值的时候Mixin会提醒你的
@Shadow public abstract void onContentChanged(Inventory inventory);
//这里@Shadow修饰的方法onContentChanged被定义为了抽象方法, 这也符合实际情况, 因为onContentChanged的具体实现是存放在CraftingScreenHandler类中的
//这里我们的Mixin类可能需要随之更改为抽象类, 不过这没什么影响
```

然后, 我们发现代码中仍旧有红色的报错, 它告诉我们stacks是CraftingInventory的私有属性, 我们无法访问. 强大的mixin当然能简单地破除访问限制, 这里我们需要新建一个接口, 来读取stacks.

在mixin包下创建一个类IMixinCraftingInventory, 写入如下代码

```java
@Mixin(CraftingInventory.class)
public interface IMixinCraftingInventory {
    @Accessor
    DefaultedList<ItemStack> getStacks();
}
```

@Accessor注解会帮助我们获取我们想要的字段, 而直接忽略访问限制. 需要注意的是方法名需要以get开头, 并且返回值就是想要获取的字段的类型, 我们可以使用如下的方式来访问这个字段

```java
((IMixinCraftingInventory) input).getStacks()
```

所以最终, 我们的CraftingScreenHandlerMixin类应该变成这样子

```java
@Mixin(CraftingScreenHandler.class)
public abstract class CraftingScreenHandlerMixin {
    @Shadow @Final private CraftingInventory input;
    @Shadow public abstract void onContentChanged(Inventory inventory);
    private final DefaultedList<ItemStack> itemList = ((IMixinCraftingInventory) input).getStacks();

    private DefaultedList<ItemStack> getInputItems() {
        return itemList;
    }

    private void setInputItems(DefaultedList<ItemStack> itemStacks) {
        if (itemStacks != null && !itemStacks.isEmpty()) {
            for (int i = 0; i < itemStacks.size(); i++) {
                itemList.set(i, itemStacks.get(i));
            }
            onContentChanged(null);
        }
    }
}
```

至此, 我们彻底完成了我们需要的这两个方法

#### 3. 最后一步

之后, 我们就需要适时地保存或者读取我们的物品列表了. 相信聪明的读者现在立马脑中会想到一件事情, 我们起初查询crafting关键字的时候, 没有发现工作台有储存数据的能力, 即我们没有找到类似CraftingTableBlockEntity的字眼, 这意味着我们需要自己创建一个BlockEntity来储存需要的数据.

新建blockentity包, 创建CraftingTableBlockEntity类并写入以下内容

```java
public class CraftingTableBlockEntity extends BlockEntity {
    public DefaultedList<ItemStack> innerItemStacks = DefaultedList.ofSize(9, ItemStack.EMPTY);

    public CraftingTableBlockEntity() {
        super(MyMod.CRAFTINGTABLE_BLOCKENTITY);
    }

    public CompoundTag toTag(CompoundTag tag) {
        super.toTag(tag);
        Inventories.toTag(tag, innerItemStacks);
        return tag;
    }

    @Override
    public void fromTag(BlockState state, CompoundTag tag) {
        super.fromTag(state, tag);
        if (innerItemStacks != null && !innerItemStacks.isEmpty()) {
            Inventories.fromTag(tag, innerItemStacks);
        }
    }
}
```

我们更改Mod主类, 注册BlockEntity

```java
public class MyMod implements ModInitializer {
    public static BlockEntityType<CraftingTableBlockEntity> CRAFTINGTABLE_BLOCKENTITY;

    @Override
    public void onInitialize() {
        CRAFTINGTABLE_BLOCKENTITY = Registry.register(Registry.BLOCK_ENTITY_TYPE, "mymod:crafting_table", BlockEntityType.Builder.create(CraftingTableBlockEntity::new, Blocks.CRAFTING_TABLE).build(null));
    }
}
```

到这里, 我们的工作台已经有了存取物品的能力, 下面我们添加相关代码. 如果仔细阅读过CraftingScreenHandler类的读者可能会注意到一个名为close的方法

```java
public void close(PlayerEntity player) {
    super.close(player);
    this.context.run((world, blockPos) -> {
        this.dropInventory(player, world, this.input);
    });
}
位于: net.minecraft.screen.CraftingScreenHandler
```

很明显, 原版的逻辑是关闭界面后掉落所有物品. 这里我们将更改close方法, 使关闭界面时工作台内物品得以保存. 我们在CraftingScreenHandlerMixin中添加下面的代码

```java
@Shadow @Final private ScreenHandlerContext context;
/**
 * @author TuRou
 * Filled the BlockEntity with the data in the crafting grid.
 */
@Overwrite()
public void close(PlayerEntity player) {
    context.run((world, pos) -> {
        BlockEntity blockEntity = world.getBlockEntity(pos);
        if (blockEntity != null) {
            ((CraftingTableBlockEntity) blockEntity).innerItemStacks = getInputItems();
        }
    });
}
```

之后我们需要让合成GUI被打开的时候从BlockEntity中读取并填充列表, 依据我们之前的分析, GUI打开的语句在在CraftingTable类中, 我们在mixin包中新建类CraftingTableMixin, 并先写入以下内容

```java
@Mixin(CraftingTableBlock.class)
public class CraftingTableMixin extends Block implements BlockEntityProvider {
    @Nullable
    @Override
    public BlockEntity createBlockEntity(BlockView world) {
        return new CraftingTableBlockEntity();
    }

    @Override
    public void afterBreak(World world, PlayerEntity player, BlockPos pos, BlockState state, @Nullable BlockEntity blockEntity, ItemStack stack) {
        super.afterBreak(world, player, pos, state, blockEntity, stack);
        if (blockEntity != null) {
            for (ItemStack innerItemStack : ((CraftingTableBlockEntity) blockEntity).innerItemStacks) {
                world.spawnEntity(new ItemEntity(world, pos.getX(), pos.getY(), pos.getZ(), innerItemStack));
            }
        }
    }
}
```

我们给合成台添加BlockEntity, 并要求在方块被破坏的时候掉出所有的物品. 之后我们来重写createScreenHandlerFactory方法, 在CraftingTableMixin中添加以下内容

```java
/**
 * @author TuRou
 * Handle Crafting Progress.
 */
@Overwrite
    public NamedScreenHandlerFactory createScreenHandlerFactory(BlockState state, World world, BlockPos pos) {
        BlockEntity blockEntity = world.getBlockEntity(pos);
        if (blockEntity == null) {
            blockEntity = new CraftingTableBlockEntity();
            world.setBlockEntity(pos, blockEntity);
        }
        DefaultedList<ItemStack> itemStacks = ((CraftingTableBlockEntity) blockEntity).innerItemStacks;
        return new SimpleNamedScreenHandlerFactory((i, playerInventory, playerEntity) -> {
            CraftingScreenHandler screenHandler = new CraftingScreenHandler(i, playerInventory, ScreenHandlerContext.create(world, pos));
            if (!itemStacks.isEmpty()) screenHandler.setInputItems(itemStacks);
            return screenHandler;
        }, new TranslatableText("container.crafting"));
    }
```

这时idea给了我们红色报错, 提示我们setInputItems方法不存在, 这里相信读者已经清楚发生了什么事情, 读者可能尝试解决问题, 给screenHandler添加了类型强转: `(CraftingScreenHandlerMixin) (Object) screenHandler` 但当此时读者启动游戏, 游戏却给我们抛了一个ClassNotFoundException

这里的正确做法是使用一个接口来过度, 在在**非mixin包**中新建类ICraftingScreenHandler, 并且写入以下内容

```java
public interface ICraftingScreenHandler {
    void setInputItems(DefaultedList<ItemStack> itemStacks);
}
```

之后我们用`((ICraftingScreenHandler) screenHandler).setInputItems()`的方式, 就可以顺利调用setInputItems方法了

最后, 将我们涉及到的所有类添加到mixins.json中, 启动游戏, 相信读者已经测试到了自己想要的效果.

> 注：我们上文强调, ICraftingScreenHandler必须放在非mixin包中, 很明显地, 这个类是为了欺骗Java虚拟机, 使其相信screenHandler一定实现了ICraftingScreenHandler接口, 而事实上, 这个接口没有被目标类使用implements显式实现, 但接口里面的方法(setInputItems)是的的确确存在的(我们利用Mixin加进去的), 我们敢于使用这种方式来蒙骗Java虚拟机从而调用到目标方法. 所以显然, ICraftingScreenHandler并不属于我们整个Mixin流程中的一部分, 它是没有必要在游戏加载之前提前被注入的.
> 读者这时可能会想到我们上文声明的IMixinCraftingInventory, 这个接口是实打实需要注入然后被目标类实现的, 可是回忆一下, 我们并没有在目标类中实现这个接口中的方法......其实是Mixin帮我们根据接口的内容自动生成的(因为我们的需求很固定, 就是访问目标类的成员变量), 固然, 这个接口是需要在游戏加载前被注入的, 所以它应放在mixin包内

---

## 总结

如果读者跟随笔者的思路, 对这三个实例进行了由浅入深的分析, 相信读者已经对mixin有了新的认识和理解, 并且有了自己解决问题的能力. 相信读者已经体会到了Minecraft源码注入的精巧之处和强大之处, 希望各位开发者能活用mixin技术, 创造出属于自己的技术.

本文的创作大力感谢xfl03的[CoreModTutor]([5 Mixin - CoreModTutor (gitbook.io)](https://xfl03.gitbook.io/coremodtutor/5)).

感谢[dengyu](https://www.mcbbs.net/home.php?mod=space&uid=897946)和[洞穴夜莺](https://www.mcbbs.net/home.php?mod=space&uid=2853776)给笔者提出改进意见

本文部分源代码和思路摘自笔者曾经开发的Mod: [Shinto]([TROU2004/Shinto (github.com)](https://github.com/TROU2004/Shinto)), [Benchworks]([TROU2004/Benchworks (github.com)](https://github.com/TROU2004/Benchworks))

