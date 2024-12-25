---
title: 从实例的角度出发浅谈TileEntity渲染
date: 2021-07-11 11:19:59
categories: [minecraft]
tags:
    - Minecraft
    - 模组开发
---

许多模组开发者不单单希望自己创造出来的一个个方块有着非凡的功能, 也希望他们能拥有美观的外表, 笔者曾经也有过这种想法, 而面对国内众多的开发教程, 大多对TESR闭而不淡或浅尝辄止, 而这种现象也是很好理解并体谅的. 许多教程的编写者具备OpenGL的知识, 但这种理论和原理并不是三言两语可以解释清楚的, 也不适合在以Minecraft模组开发为核心的教程中插入过多的OpenGL理论知识, 这可能会打击读者的信心, 或使读者丧失兴趣

笔者曾经也是众多读者中的一个, 直至现在, 也仅仅是满足上文条件中的 **了解OpenGL的读音和拼写 **. 所以本文的写作缘由也因此而生, 笔者希望基于Minecraft模组开发, 介绍简单的渲染知识, 实现TESR简单的一些效果. 而在这个过程中希望读者可以和笔者一起, 从实例的角度出发, 共同探究TESR的渲染方式, 同时对OpenGL的渲染有简单的了解, 启发读者的兴趣.

## 本文的受众对象

- 对于Minecraft和Minecraft Forge开发有较为进阶的了解
- 对实例代码有一定的理解和分析能力
- 简要了解Minecraft模型和材质
- 了解OpenGL的读音和拼写

## 介绍TESR

先说说为什么要引入TESR. Minecraft 1.8以后, 相信读者也对此有所了解, Mojang引入了一套模型系统, 之后我们的Mod开发过程中经常会有这样的操作: 将贴图放在一个特定的目录(texture), 用模型(model)来定义贴图的呈现方式, 再以代码的形式将物品绑定到模型上. 我们一直以来以这样的方式为我们的物品和方块添加贴图.

但如若我们要根据游戏内的具体情况(比如物品内部的NBTTag, 或者方块的TileEntity数据), 对物品或方块的渲染有所干预呢, 这时候静态的模型文件(.json)就不管用了, 我们需要引入一种特殊的渲染方式. 对于物品来说, 读者可以参见笔者之前的文章, 利用ItemMeshDefinition来实现. 而对于方块来说, 我们可以使用TileEntitySpecialRenderer来实现

TileEntitySpecialRenderer(下文简称为TESR)顾名思义, 是基于TileEntity数据的一种特殊渲染器. 其特殊之处, 便是我们可以随心所欲的定制化这种渲染的方式, 当TileEntity刷新时, Minecraft会调用我们事先设计好的渲染方式来渲染整个方块, 而我们要做的, 便是利用手中的一个完整的TileEntity数据来完成中间渲染的所有操作.而这种对于模型的特殊操作, 是靠近底层的

> 这些模型有一个共同特点：它们都在某种意义上靠近底层。   -Harbinger (11.4 TESR)

而这种底层, 即Minecraft的渲染器——OpenGL

> By default OpenGL (via `GlStateManager`) is used to handle rendering in a TESR. See the OpenGL documentation to learn more. -FORGE docs
> 
> 通常来说, 需要OpenGl(通过GlStateManager固定管线)来处理TESR的渲染, 这部分需要查阅OpenGL的文档来了解

## 一、谈谈管线

相信读者对TESR的简单了解中, 大概清晰了思路, 我们需要调用OpenGL的底层方法来完成一个方块的渲染工序. 作为我们的显卡GPU, 所有渲染是以管线的方式进行的, 说起管线, 我们常将其比作流水线. 我们将一堆数据塞进去, 经过一些流水线处理, 处理出一帧帧画面. 在Minecraft世界中, 也就是我们玩家充当观察者角色所能看到的一个个方块, 实体.

对于我们开发者而言, 我们的任务很清晰, 我们只需编排好渲染方块的流水线就可以了. 而这些, 都包含在这个方块对应TESR类的一个render方法中. 我们只需事先准备好一段固定的render方法. 来让Minecraft渲染器在需要渲染的时候不断调用render()就可以了, 这也被称为是固定管线.

固定管线与可编程渲染管线相对应, 固定管线可以从字面理解, 管线中渲染的方式是完全固定的, 所以我们可能会看到这样的伪代码: 

```java
class MyTESR {
    render(...) {
        GL.RenderALight(x1, y1, z1);
        GL.RenderBLight(x2, y2, z2);
        GL.RenderCShadow(x3, y3, z3);
    } //很容易看出我们是按照一段固定的逻辑渲染ALight BLight CShadow的
}

class Minecraft {
    RenderTask() {
        ...//Render Trees
        myTESR.render(...);
        ...//Render Waters
        ...//Render Items
    }
}
```

我们在今后的渲染中, 需要填充render方法的内容. 而其他事情, 就尽管交给Minecraft和Forge吧.

但是这里还要插入一个注意事项: 我们操作的所谓"OpenGL"(例如下文中的GL11)是一个全局的操作(状态机), 所以我们对全局量进行一些操作后, 记得复原他的状态. 例如下文的伪代码: 

```
......
开发者: 喂, OpenGL, 我要渲染了, 先保存你原来的状态.
OpenGL: 了解, 原来状态已保存, 你可以开始渲染工作了
开发者: xxoo渲染就交给你了
OpenGL: xxoo渲染已完成(我的里面已经变成了你的形状, 已经处理不了别的任务了)
开发者: 喂, OpenGL, 请读取你之前的状态继续工作吧
OpenGL: 好的(满血复活, 继续别的工作)
......
这就是OpenGL上下文
```

到这里为止, 单纯的理论知识部分就已经结束啦, 下面跟随读者一起从实例的角度出发, 探究这种工序的具体编写方法吧.

## 二、从简单实例入手

先看下面一个例子, 我们需要用到TESR吗?

我们有一个饼干罐, 当我们右键点击的时候可以向里面存入曲奇饼, 并且罐子内部会逐渐充满曲奇饼的贴图, 我们也可以空手右键取出曲奇饼, 罐子内部会相应减少曲奇饼的数量.

![cookie.png](https://i.loli.net/2021/01/31/MQr3ojCsBul1HTU.png)

答案是肯定的, 饼干罐一定有自己的TileEntity储存内部的饼干数量, 并且会有一个TESR指导渲染工作, 于是我们可以找到如下代码

```java
public class CookieRenderer extends TileEntitySpecialRenderer<TileEntityCookieJar> {
    @Override
    public void render(TileEntityCookieJar cookieJar, double x, double y, double z, float partialTicks, int destroyStage, float alpha) {
        //Render work
    }
}
/*
仓库地址: https://github.com/MrCrayfish/MrCrayfishFurnitureMod/
节选自: /1.12.2/src/main/java/com/mrcrayfish/furniture/render/tileentity/CookieRenderer.java
*/
```

不出我们所料, 果然有一个render方法负责渲染整个方块, 但细心的读者可能发现, 里面的渲染工作似乎与我们之前所描述的过程有一些区别, 我们可以看到代码中是围绕曲奇饼的EntityItem进行渲染, 但我们并没有发现这个玻璃罐子的渲染. 这里其实是因为TileEntitySpecialRender中强调了Special一词, 这也就表面我们只需渲染其特殊的一部分就可以了, 静态的罐子并不特殊, 他是以普通的模型->材质的方式来定义的, 所以不在TESR渲染的范围内.

接下来我们来具体看我们省略的Render work部分.

```java
GL11.glPushMatrix(); //保存变换前的位置和角度
    //Some Code
GL11.glPopMatrix(); //读取变换前的位置和角度(恢复原状)
```

根据上文我们提到的OpenGL状态机和OpenGL上下文的特点, 相信读者不难想出为什么这两行代码是成对出现的. 其实, 这两句代码就是我们上文提到的一个保存和读取的功能, 而这里, 我们保存和读取的是矩阵(这里可以看做是Minecraft世界中的坐标). 我们先保存之前的坐标状态, 然后在中间的渲染步骤中随意的变更, 操作坐标, 最后读取回原来的状态, 渲染顺利进行的同时不会破坏OpenGL原有的工作.

```java
GL11.glDisable(GL11.GL_LIGHTING);
    //Some Code
GL11.glEnable(GL11.GL_LIGHTING);
```

这两句代码也是成对出现的, 按照字面意思, 是先关闭了OpenGL的光线, 又打开了OpenGL的光线, 这也就意味着我们的渲染过程中不需要光线(希望GL_LIGHTING处于关闭状态), 而完成后为了恢复状态, 再一次开启了GL_LIGHTING, 这也符合我们所介绍的状态机的设定. 而这里为什么我们的渲染过程要关闭GL_LIGHTING呢, 这是因为我们的渲染过程中没有光源, 我们不希望OpenGL帮我们进行光源的演算和渲染, 如果没有光源强行渲染, 会出现很黑的情况(没有光源当然很黑嘛). 所以我们索性直接关掉光线演算, 避免这个问题, 详情请查看鸣谢部分的具体解读.

好, 状态保存的问题已经解决了, 接下来就该轮到我们大搞特搞地来渲染具体物品的环节了, 而这时我们看到了translate语句.

```java
GL11.glTranslatef((float) x + 0.5F, (float) y + 0.05F, (float) z + 0.18F); //设置"原点"
...
GlStateManager.translate(0, -0.1, 0); //沿y轴负方向移动0.1个单位
```

![20190415170233683.png](https://i.loli.net/2021/01/31/9qphezAWxMIcK1D.png)

相信读者看懂这张图之后, translate的用途问题不攻自破了, 其实它就相当于一个绝对坐标转换为相对坐标的操作, 方便我们接下来对这个方块的具体位置进行一定的操作做铺垫工作. 而translate提供的参数, 将作为新坐标的原点位置, 而为了让这个原点不停留在方块边缘, 对方块坐标进行加减运算的用途不言而喻.

当开发者需要对当前操作坐标进行旋转变换时, 可能需要多次进行translate操作, 相信读者可以在之后的工作中逐步理解这个概念. 而这里, 我们猜测MrCrayfish是将渲染原点定在了靠近罐子中心底部的位置, 以便进行接下来的操作.

```java
GL11.glRotatef(180, 0, 1, 1); //沿着y轴和z轴方向旋转180度
```

之后, 我们对操作的矩阵进行了旋转变换, 目的是为了在下一步的渲染中让曲奇饼"平躺"在罐子的底部, 而不是竖直立在罐子里面. 

```java
GL11.glScalef(0.9F, 0.9F, 0.9F); //xyz轴各缩放0.9倍
```

最后, 我们对操作的矩阵进行了简单的缩放, 让曲奇饼缩放到0.9倍的大小, 相信读者可以在今后的操作中逐步明白并理解, 这里笔者还想插入一些注意事项. 读者可以选择性阅读, 或者遇到相关问题后再回来阅读

> glTranslate ，把当前矩阵和一个表示移动物体的矩阵相乘。三个参数分别表示了在三个坐标上的位移值。
> 
> glRotate ，把当前矩阵和一个表示旋转物体的矩阵相乘。物体将绕着 ( 0 , 0 , 0 )到 ( x , y , z ) 的直线以逆时针旋转，参数提供了旋转的角度。
> 
> glScale ，把当前矩阵和一个表示缩放物体的矩阵相乘。 x , y , z分别表示在该方向上的缩放比例。
> 
> 注意，“先移动后旋转”和“先旋转后移动”得到的结果很可能不同。对于未曾接触过数学中矩阵运算的读者, 这里笔者不以矩阵为数学基础展开, 相信依靠读者丰富的想象力和Minecraft培养出来的空间能力, 可以在自己摸索的过程中逐步理解这些概念.
> 
> ![20170419132833166.jpg](https://i.loli.net/2021/01/31/aCZUmfqk8jF5wAp.png)

到这里, 坐标的预处理工作已经完成了, 也就是我们现在坐标的位置已经位于饼干罐中心靠近底部的位置了, 我们只需在当前位置渲染出一个Minecraft原版曲奇的贴图, 就完成了, 而MrCrayfish也是这样做的, 用一对for循环嵌套, 逐个渲染出物品的位置, 完成我们的工作

```java
for(int i = 0; i < metadata; i++) {
    Minecraft.getMinecraft().getRenderManager().renderEntity(entityItem, 0.0D, 0.0D, 0.1D * i, 0.0F, 0.0F, false);
}
```

我们直接调用Minecraft内部RenderManager, 使其渲染一个entity就可以了. 而且笔者也相信, 此时的读者已经不会问出"为什么调用这个方法不需要提供饼干罐的具体坐标呀"这种问题了.

当我们阅读后, 我们发现实际代码与我们的预想有一些偏差, 即饼干罐内部的数量并不是TileEntity类内部的一个成员. 而是以方块状态(metadata)的形式储存的, 对此抱有疑问的读者可以自行查阅Harbinger相关部分, 这里读者就不多作介绍了.

至此, 我们对游戏内调用OpenGL渲染的过程有了初步了解, 相信读者也可以自己根据需求, 渲染出自己想要的效果了. 这里笔者补充一点. 在我们编写矩阵变换时, 通常很难把握物品的具体位置, 使用尺子测量这种操作也是完全没有必要的, 毕竟我们不是进行精密的数学运算, 这里笔者推荐使用自己IDE的调试功能, 利用代码热重载来完成变换操作, 这一点读者可以自行查询自己IDE的热重载方式. 对于IDEA来说, 请使用调试按钮和Debugging Action中的重载按钮来重载.

------

根据以上的学习, 笔者自己也尝试写了一个TESR, 读者可以用自己学到的知识简单解析一下

```java
@Override
public void render(TestBlockTile te, double x, double y, double z, float partialTicks, int destroyStage, float alpha) {
    GlStateManager.PushMatrix();
    GlStateManager.disableLighting();
    GlStateManager.translate(x, y, z);
    GlStateManager.translate(0.5,0.2,0.4);
    GlStateManager.scale(0.6, 0.6, 0.6);
    Minecraft.getMinecraft().getRenderManager().renderEntity(new EntityPig(te.getWorld()), 0,0,0,0,0, false);
    GlStateManager.enableLighting;
    GlStateManager.popMatrix();
}
```

![Snipaste_2021-01-31_19-50-21.png](https://i.loli.net/2021/01/31/2JEMwQeFZRKpLA1.png)

> 一些提示: 
> 
> ​    对于GLStateManager和GL11: 读者在阅读代码的时候经常会看到不同的开发者可能会使用这两个不同的类来调用OpenGL, GL11是org.lwjgl.opengl包下的, 这意味着我们如果调用GL11, 是直接利用lwjgl提供的方法来进行渲染的, 而GLStateManager是net.minecraft.client.renderer包下的, 这是Minecraft为我们封装的渲染方法. 这里读者推荐使用GLStateManager, 一是为了尽量使用Minecraft提供的规范, 二是GL11相比GLStateManager更底层, 对于TESR来说调用Minecraft的方法就足够了
> 
> ​    GLStateManager和GL11中的方法名可能略有不同, 这里希望读者可以自行区分. 例如:
> 
> ​    GL11.glPushMatrix(); 和 GlStateManager.pushMatrix();
> 
> ​    GL11.disable(GL11.GL_LIGHTING); 和 GlStateManager.disableLighting();

## 三、我还想要更多, 更多

之后我们来看一看如何用OpenGL在Minecraft世界中渲染出文字图片和平面图形. 相信聪明的读者会尝试在getRenderManager()的Manager中寻找类似名为renderText或renderTexture的方法, 不过IDE给我们的自动补全则给了我们当头一棒, RenderManager中并没有类似的方法. 这里我们深入思考一下, 文字和平面图形并不是Minecraft中独创的内容, 而游戏中会动的长方体(生物或物品实体)则是游戏内独创的要素, 所以游戏给我们提供相应的接口是理所当然的. 那对于文字图片来说, 游戏有没有给我们封装相应的方法呢?

这里我们从EnderUtilities的储物桶中可以得到相关的代码实现, 储物桶上方有物品数量的悬浮提示, 储物桶有红色的容量指示条, 储物桶还有一些需要渲染的图片(例如上锁的储物桶有小锁头的图片)或许这些功能的实现可以解答我们的疑问, 并且教会我们一些知识

![Snipaste_2021-02-06_10-07-31.png](https://i.loli.net/2021/02/06/AYwOITRqxinLkrW.png)

### 文字渲染 —— fontRenderer

```java
private void renderText(String text, double x, double y, double z, EnumFacing side, EnumFacing barrelFront)
    {
        FontRenderer fontRenderer = Minecraft.getMinecraft().fontRenderer;
        int strLenHalved = fontRenderer.getStringWidth(text) / 2;

        GlStateManager.pushMatrix();
        //...省略了translate过程...
        GlStateManager.scale(-0.01F, -0.01F, 0.01F);
        GlStateManager.color(1.0F, 1.0F, 1.0F, 1.0F); //将画笔置为黑色, 便于进行绘画(这里没有进行绘画)

        GlStateManager.disableLighting();
        GlStateManager.enablePolygonOffset();
        GlStateManager.depthMask(false);
        GlStateManager.enableBlend(); //开启混合器(使GL支持Alpha透明通道)
        GlStateManager.doPolygonOffset(-1, -20);

        fontRenderer.drawString(text, -strLenHalved, 0, 0xFFFFFFFF);

        GlStateManager.disableBlend();
        GlStateManager.depthMask(true);
        GlStateManager.disablePolygonOffset();
        GlStateManager.enableLighting();

        GlStateManager.popMatrix();
    }
/*
仓库地址: https://github.com/maruohon/enderutilities/
节选自: /MC_1.11.x/src/main/java/fi/dy/masa/enderutilities/client/renderer/tileentity/TESRBarrel.java
*/
```

这里笔者将translate部分语句省略了, 作为储物桶来说, 有一个面是玩家的操作面, 所有的文字和图标应该渲染在这个方向上, 作者的translate语句正是将操作方向转移到操作面上. 感兴趣的读者可以自行阅读相关代码, 这里我们一起来看看一些我们没有介绍过的GL语句以及字体渲染的方法

```java
GlStateManager.enablePolygonOffset();
GlStateManager.doPolygonOffset(-1, -20);
GlStateManager.disablePolygonOffset();
```

PolygonOffset解决了一个贴图打架的问题, 即z-fighting, 如果我们不指定贴图的层叠关系, 贴图们会挤在同一层中发生闪烁.

所以当我们想要将两个贴图相互重叠时, 我们的想法是让A叠在B上(或者说让B在后面, 稍微跟A隔开一点距离, 防止打架), 所以我们得将这种层叠关系表示出来. 这就是PolygonOffset

![z-fighting.jpg](https://i.loli.net/2021/02/06/2N8ZkPg1EHwxej4.jpg)

doPolygonOffset的两个参数为factor和units, 这两个量会影响AB两个面隔开的距离, 如果想要非常好地使用PolygonOffset，你需要做一些数学上的研究。不过一般而言，只需把1.0和0.0这样简单的值赋给glPolygonOffset即可满足需要。

```
GlStateManager.depthMask(false);
GlStateManager.depthMask(true);
```

depthMask也与我们刚刚介绍过的z-fighting有关, depthMask也就是深度测试(深度缓冲), 参考一些文章后, 有这样的解释

> 开启深度测试后OpenGL就不会再去绘制模型被遮挡的部分，这样实现的显示画面更为真实，但是由于深度缓冲区精度的限制，对于深度相差非常小的情况（例如在同一平面上进行两次绘制），**OpenGL就不能正确判定两者的深度值，会导致深度测试的结果不可预测**，显示出来的现象时交错闪烁的前后两个画面，这种情况称为z-fighting。

这也不难推知我们为什么要关闭depthMask并且手动处理PolygonOffset了, 因为我们的需求是在方块的一个面上绘制内容, 即"在同一平面上进行两次绘制", 所以我们这种操作也不难理解了.

```
FontRenderer fontRenderer = Minecraft.getMinecraft().fontRenderer;
int strLenHalved = fontRenderer.getStringWidth(text) / 2;
fontRenderer.drawString(text, -strLenHalved, 0, 0xFFFFFFFF);
```

相信读者已经可以回答我们这一节开头提出的问题了, Minecraft的确为我们封装了字体渲染的相关内容, 即Minecraft.getMinecraft().fontRenderer, 我们只需调用drawString()方法并传入字符串, x, y轴位置和十六进制颜色值就可以了. 至于为什么这里计算了字符串一半的长度, 相信聪明的读者已经想到了什么...这里进一步提问, 我们前文省略的translate语句, 是将画笔的位置移动到了哪里呢？ 对了, 是移动到了贴面顶端的中心位置.

### 图片渲染 —— bindTexture和Tessellator

事情到了这里, 我们终于不可避免的要涉及到Tessellator了, 这里我们需要先引入一些理论知识, 才能继续向下理解.

> 注: Tessellator与图形学中的细分曲面(Tessellation)无关, Tessellator是OpenGL立即模式的一种封装

我们之前的操作都直接调用了OpenGL去绘制我们在屏幕上看到的事物, 但直接与OpenGL交互在某种层面来说是很麻烦的, 所以我们引入了Tessellator来简化这个绘制过程同时提高效率.

> Tessellator代替了我们直接调用GL的一个个方法调用, Tessellator收集了关于渲染细节的一个数组, 并且规定这个数组哪些部分代表什么(以便于渲染器正确组织数组中的数据), 并且批量提交给GPU进行绘制, 最后这些细节会被传进VBO, 然后用 glDrawArray 批量画出来.

```java
Minecraft.getMinecraft().getTextureManager().bindTexture(TEXTURE_LOCK);
Tessellator tessellator = Tessellator.getInstance(); //获取Tessellator的一般方式
VertexBuffer buffer = tessellator.getBuffer(); //获取记录顶点信息的"数组"

buffer.begin(GL11.GL_QUADS, DefaultVertexFormats.POSITION_TEX); //指定数组的组织方式(位置 + UV方式), 以及要画的图像的顶点数(矩形四个顶点)

buffer.pos(  0,   0, 0).tex(0, 0).endVertex(); //提供矩形的四个顶点, 并绑定UV
buffer.pos(  0, 0.15, 0).tex(0, 1).endVertex(); //提供矩形的四个顶点, 并绑定UV
buffer.pos(0.15, 0.15, 0).tex(1, 1).endVertex(); //提供矩形的四个顶点, 并绑定UV
buffer.pos(0.15,   0, 0).tex(1, 0).endVertex(); //提供矩形的四个顶点, 并绑定UV

tessellator.draw(); //将数组和渲染方式提交到GPU
```

这里我们调用了Minecraft.getMinecraft().getTextureManager()的bindTexture方法, 将锁的图片载入到GL状态机中, 之后我们利用Tessellator来将矩形的四个顶点与对应UV的四个角落绑定, 就完成了操作, 如下图

![Snipaste_2021-02-06_12-28-05.png](https://i.loli.net/2021/02/06/XeRuVsivqlWPcaK.png)

这里读者应该对Tessellator矩形绑定材质的渲染方式有了初步的了解. 下面我们来进一步巩固一下我们的理解

```java
buffer.begin(GL11.GL_QUADS, DefaultVertexFormats.POSITION_COLOR); //指定数组的组织方式(位置 + 颜色值), 以及要画的图像的顶点数(矩形四个顶点)

int r_b = 0x03;
int g_b = 0x03;
int b_b = 0x20;

buffer.pos(  0,    0, 0).color(r_b, g_b, b_b, 255).endVertex();
buffer.pos(  0, 0.08, 0).color(r_b, g_b, b_b, 255).endVertex();
buffer.pos(0.6, 0.08, 0).color(r_b, g_b, b_b, 255).endVertex();
buffer.pos(0.6,    0, 0).color(r_b, g_b, b_b, 255).endVertex(); //提供矩形的四个顶点, 传入相应的颜色值

int r_f = 0x20;
int g_f = 0x90;
int b_f = 0xF0;
float e = fullness * 0.57f;

buffer.pos(0.585    , 0.065, -0.001).color(r_f, g_f, b_f, 255).endVertex();
buffer.pos(0.585    , 0.015, -0.001).color(r_f, g_f, b_f, 255).endVertex();
buffer.pos(0.585 - e, 0.015, -0.001).color(r_f, g_f, b_f, 255).endVertex();
buffer.pos(0.585 - e, 0.065, -0.001).color(r_f, g_f, b_f, 255).endVertex();//提供矩形的四个顶点, 传入相应的颜色值, 顶点的位置随着储物桶的内部容量而更改.

tessellator.draw(); //将数组和渲染方式提交到GPU
```

这样, 我们就成功在储物桶上绘制好了两个矩形. 相信到这里, 读者已经对Tessellator的使用方式和基本原理有了初步的理解和认识. 读懂其他TESR的渲染过程也不会一头雾水了吧.

> 注: 其实除了Tessellator, 还有一种更为原始的操作方式, 这里笔者就不详细说明了. 笔者希望读者能使用Tessellator这种便利而高效的方式, 对于glBegin() glEnd()这种原始操作, 笔者这里放出一个例子, 感兴趣的读者可以自行查阅, 相信理解Tessellator的读者不难体会其中的原理:
> 
> https://github.com/MrCrayfish/MrCrayfishFurnitureMod/blob/1.12.2/src/main/java/com/mrcrayfish/furniture/render/tileentity/CupRenderer.java

## 四、小结

### 小结

在本文中, 我们以OpenGL初学者角度从简单的实例入手, 由浅入深地对OpenGL在Minecraft中的使用方式进行了简要的介绍, 对Tessellator的使用方式和原理进行了简要说明, 相信读者已经可以自己上手写出成型的TESR, 也会对之后GUI界面的绘制有很大的帮助. 笔者水平有限, 希望多多包涵. 如果你有任何的反馈和建议欢迎在本帖下方留言.

### 鸣谢

Harbinger提供知识框架: [harbinger.covertdragon.team](harbinger.covertdragon.team)

**御坂御坂001个人Blog提供基础知识补充**: [www.cnblogs.com/xiegaosen/p/10809022.html](www.cnblogs.com/xiegaosen/p/10809022.html)

MrCrayfish家具Mod提供部分代码分析: [github.com/MrCrayfish/MrCrayfishFurnitureMod](github.com/MrCrayfish/MrCrayfishFurnitureMod)

McJty的模组教程提供部分知识基础: [wiki.mcjty.eu/modding/index.php](wiki.mcjty.eu/modding/index.php)

GL_LIGHTING关闭再打开的原因解答: http://forum.lwjgl.org/index.php?topic=4753.0

矩阵变换相关理解: [https://blog.csdn.net/hebbely/article/details/70225045](https://blog.csdn.net/hebbely/article/details/70225045)

一些Tessellator的灵感 [https://forums.minecraftforge.net/topic/66168-1122-using-minecrafts-tessellator-and-bufferbuilder/](https://forums.minecraftforge.net/topic/66168-1122-using-minecrafts-tessellator-and-bufferbuilder/)

**土球球解答了笔者的一些疑惑**: [https://www.mcbbs.net/home.php?mod=space&uid=1480882](https://www.mcbbs.net/home.php?mod=space&uid=1480882)

PolygonOffset和z-fighting: [https://www.cnblogs.com/bitzhuwei/p/polygon-offset-for-stitching-andz-fighting.html](https://www.cnblogs.com/bitzhuwei/p/polygon-offset-for-stitching-andz-fighting.html)

**Tessellator**: [http://greyminecraftcoder.blogspot.com/2013/08/the-tessellator.html](http://greyminecraftcoder.blogspot.com/2013/08/the-tessellator.html)

EnderUtilities提供部分代码分析: [https://github.com/maruohon/enderutilities/](https://github.com/maruohon/enderutilities/)