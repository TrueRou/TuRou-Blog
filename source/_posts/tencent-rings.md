---
title: 基于HyperV和VAC的腾讯会议自动打铃解决方案
date: 2023-03-05 11:19:59
tags:
    - 参赛作品
    - 腾讯会议
    - 打铃系统
---

**第37届吉林省青少年科技创新大赛 参赛作品**

**荣获第37届吉林省青少年科技创新大赛 最佳作品奖**

![1706233693783.jpg](https://s2.loli.net/2024/01/26/QNbg8OYEeFRrHyW.jpg)

## 研究目的

为腾讯会议创建的会议室提供一套易部署、高灵活、可复制的自动打铃解决方案

## 研究方法

使用从需求到实现的溯源性思维，结合现有的虚拟化(VT-d)技术，虚拟音视频技术，利用C#编程语言，提供了一套切实可行的解决方案

## 研究结论

- 腾讯会议本身未提供第三方播放音视频的接口，需要模拟一位用户来播放音视频
- 播放音频本质是腾讯会议监听系统麦克风，为了将本地音频模拟成麦克风，需要借助虚拟声卡等软件模拟系统麦克风
- 播放视频本质是腾讯会议调用系统摄像头，为了将目标画面投射到系统摄像头中,需要借助Open Broadcaster Software等软件模拟摄像头
- 为了按需求灵活适应打铃的时间需求，需要编写一个GUI程序来实时显示日程状态，并且在适当时机播放打铃音频
- C# + WPF可以编写美观且高性能的GUI程序

## 前言

腾讯会议作为目前最流行的在线即时会议软件之一，在空中课堂、网上教学等领域发挥重要作用。笔者作为腾讯会议网上教学的受益者之一，在使用中逐渐对腾讯会议的优缺点有了更深的了解，这个过程中，上下课铃播放的问题逐渐引起了笔者的注意。

在过去，教师往往需要时刻注意当前的时间，使网上教学工作量加大，如果将提示上下课的工作交给学生，也会一定程度影响学生的上课质量和上课效率，如何将打铃的过程自动化，成为了笔者近期思考的问题。

经过笔者的思考和对相关技术的学习和了解，笔者撰写了此文，希望能对未来的后学有启示作用。

## 研究过程

### 确定研究思路

实现自动打铃主要分为两步，即播放音视频和呈现音视频。播放音视频是指将我们预先设置好的音视频在适当的时机播放出来，呈现音视频是指将播放的音视频以某种方式投射或呈现到腾讯会议中。

定时播放音视频是一个十分常见的需求，市面上也有许多现成可复用的解决方案，但对于空中课堂来说，我们的播放需求较为复杂，存在切换日程、显示状态、提示课节等需求，这里笔者决定使用目前流行的Windows桌面程序开发方案C# + WPF来开发定时播放音视频的程序(下文简称“打铃程序”)
呈现音视频的难度较高，基于我们的需求分析，我们希望腾讯会议能提供一个接口或者方法来捕捉我们提供的音视频。这里读者可能联想到腾讯会议的屏幕分享功能，笔者当然也考虑并尝试过这种方式。实际上，会议室中同时只有一位用户可以屏幕分享，如果需要对打铃程序使用屏幕分享，就会严重妨碍主持人正常使用屏幕分享功能，这显然违背了我们的初心，为线上教学带来了不便，我们固然需要舍弃这种方案。

反向来思考，怎样做能多个用户共享音视频呢，很容易想到，同学们在会议室中打开摄像头、打开麦克风正是一种“共享音视频”的形式，如果我们能对这种形式加以利用，便可以在不妨碍正常屏幕共享的前提下提供音频播放和视频播放的功能。显然，我们需要“伪装”一个麦克风和摄像头，来让腾讯会议主动地呈现我们的音视频。

伪装麦克风，即虚拟一个麦克风，虚拟麦克风可以模拟一个麦克风硬件，该麦克风将虚拟地捕获到目标音频，这里我们希望将扬声器播放的声音重新传入虚拟麦克风，所以我们使用了Virtual Audio Cable软件(下文简称“VAC”)，该软件可以满足我们的需求

伪装摄像头，即虚拟一个摄像头，虚拟的摄像头模拟了一个摄像头硬件，摄像头捕捉的画面可以由我们提供，这里我们希望打铃程序美观的界面可以被传入摄像头，方便同学们查看课节信息和下课时间，我们使用了开源的Open Broadcaster Software软件(下文简称“OBS”)，该软件可以将预设的场景变为虚拟摄像头，只需将场景中添加窗口捕获，来捕获我们的GUI程序，便可以满足我们的需求了
解决了音视频的播放和呈现之后，另一个问题便浮出水面，显然打铃程序需要运行在一台计算机上，并且该计算机需要让VAC接管音频，使用OBS来捕捉打铃程序的GUI画面。该计算机最好不受外界操作的干扰，单独负责腾讯会议自动打铃的服务。不仅如此，设想一下，我们可能希望为多个会议室提供打铃服务，如何使我们的方案变得可扩展而且高灵活呢。显然，购置多台计算机来分布式完成这个工作是及其不合理的，打铃服务显然不需要极强的计算能力，购置计算机也放大了没有必要的成本，虚拟化技术便是解决该问题的唯一答案。

虚拟化技术可以将一台物理机拆分为几个虚拟层，即做到一台电脑模拟多台电脑的效果。借助Intel的VT-d技术，我们可以高性能地将一台计算机的硬件IO虚拟化，使用统一的层来处理物理机和虚拟机的指令，大大减少虚拟化的性能损耗。这里，我们可以将物理计算机拆分为多台虚拟机，为每一个会议室单独提供虚拟的一台计算机，在虚拟计算机中运行我们上述提到的音视频播放和呈现功能。

Hyper-V是Windows附带的虚拟机平台，它可以批量管理多台虚拟机，并且方便地进行虚拟机复制和容器内操作，并且不需要支付额外的使用费用，这里我们使用HyperV来作为虚拟机容器，实际上，也可以使用VirtualBox等开源免费虚拟机平台。

总结一下，我们可以将实现自动打铃的需求抽象成下面的流程图

![图片1.png](https://s2.loli.net/2024/01/26/AOhWmTYf1lZouvz.png)

### 进行实践

#### 准备Hyper-V虚拟机容器

首先需要在Windows的可选功能中开启Hyper-V和虚拟机平台，开启后需要重启计算机，Windows将以Windows更新的方式来安装所需的组件，安装过后，就可以在开始菜单打开Hyper-V管理器了

![图片2.png](https://s2.loli.net/2024/01/26/UqOJZpsmRyEckal.png)

![图片3.png](https://s2.loli.net/2024/01/26/dLyXfitvouheZqn.png)

这里笔者的Hyper-V中已经包含了一些虚拟机了，为了创建专门用于自动打铃的虚拟机，我们可以新建一个空白的虚拟机，点击新建 -> 虚拟机来创建，具体参数笔者这里不做过多介绍，仅给出一些推荐参数给读者参考

内存：推荐分配1024-2048MB，防止腾讯会议因内存不足而发生错误。硬盘：推荐分配30GB以上，Hyper-V默认会按需扩张磁盘大小。系统：推荐使用LTSB或LTSC来避免Windows更新，同时缩减因虚拟机系统带来的不必要的性能损失

接下来我们启动创建的虚拟机，完成第一次安装Windows的OOBC过程，直至成功进入系统桌面。之后我们需要安装自动打铃中涉及的一些软件，这部分笔者不做过多赘述，读者可以按照在物理机安装软件的方法来安装软件。

这里我们需要安装腾讯会议，OBS，和VAC软件，VAC软件可能在安装的过程中需要装额外的音频驱动，这里我们选择同意安装驱动即可。

至此，对于虚拟机的提前准备工作已经完成，我们可以着手进行打铃程序的开发了

#### 开发打铃程序

我们的打铃程序基于C#和WPF开发，首先我们要安装IDE（集成开发环境）来进行程序的开发和打包工作。这里我们选用流行的 Visual Studio开发环境（以下简称VS）来进行开发工作。由于我们的软件用于个人和学习用途，这里我们选择Community版本，这里笔者不再过多赘述

之后我们选择新建项目，创建一个新的WPF窗口程序（注意使用.NET CORE）这样我们就可以利用WPF来快速开发一个美观的GUI界面了。

我们先专注于业务的核心逻辑，即定时打铃，为了使定时更符合我们的需求并且减小开发压力，我们使用C#流行的日程库Quartz来设计定时任务，至于打铃（播放音频），我们可以直接用.NET核心库中的SoundPlayer类来实现音频播放。

C#是一门面向对象编程语言，这里我们先实现一个抽象的Class类来承载课节这一信息，我们的课节应该包含上课时间、课节长度、以及下课的休息时间，并且我们的抽象课节应该可以被添加到Quartz的Scheduler中，成为Quartz的计划任务。Quartz的计划任务添加至Scheduler时需要实现IJob接口的Execute方法，这个方法描述了当计划任务被执行时要执行的一系列代码。整理我们的思路，可以得到以下UML图

![图片4.png](https://s2.loli.net/2024/01/26/cg68EM9PlIRrhLO.png)

根据UML图，我们搭建了Class类并补全了方法的具体实现

![图片5.png](https://s2.loli.net/2024/01/26/urgk6jplSKqHYRA.png)

这里代码中我们有调用MainWindow的部分方法，这些方法与GUI界面的展示和刷新有关，下面我们来编写与GUI有关的代码

WPF的界面设计分为XAML设计和代码逻辑编写两部分，实际页面控件的添加使用WPF独创的XAML语法，而对控件的逻辑控制都采用C#代码来进行，这里我们为了使界面更美观使用了MaterialDesignThemes包，这个包内置了一些符合MD设计规范的设计模式控件样式，简化了我们的开发流程。

这里笔者不再赘述如何导入MaterialDesignThemes包，仅贴出最终效果和XAML代码供读者参考，在界面中，我们预留了暂停打铃，切换日程，更换铃声等功能，我们之后将在控件逻辑部分补全他们的实现

![图片6.png](https://s2.loli.net/2024/01/26/zm1OnbVtJsUDiqW.png)

![图片7.png](https://s2.loli.net/2024/01/26/xpPJfTusUv7m1RA.png)

之后我们来补全GUI的逻辑，这里同时也包含程序启动时的初始化进程，以及切换打铃模式等常见逻辑的实现，这里我们仅挑出几个方法加以展示

```c#
public MainWindow()
{
    Instance = this;
    foreach (var item in Directory.GetFiles(SoundsPath, "*.wav"))
    {
        sounds.Add(Path.GetFileNameWithoutExtension(item));
    }
    AddLog("Wakaru正在启动...");
    InitializeComponent();
    ClassBeginBox.ItemsSource = sounds;
    ClassOverBox.ItemsSource = sounds;
    ChangeStatus(Status.WAIT_FOR_CLASS);
    DispatcherTimer dispatcherTimer = new DispatcherTimer()
    {
        Interval = new TimeSpan(0, 0, 0, 1, 0)
    };
    dispatcherTimer.Tick += new EventHandler((o, e) => {
        Instance.Dispatcher.BeginInvoke(new Action(() => {
            Instance.TimeNowText.Text = "当前时间: " + DateTime.Now.ToString();
            Instance.TimeNextText.Text = "";
            Instance.TimeIntervalText.Text = "";
            if (NextTime != DateTime.MinValue) 
            {
                Instance.TimeNextText.Text = "下一时间: " + NextTime.ToString();
                Instance.TimeIntervalText.Text = "剩余时间: " + new TimeSpan(NextTime.Ticks - DateTime.Now.Ticks).ToString(@"hh\:mm\:ss");
            }
            
        }));
    });
    dispatcherTimer.Start();
}
```

```c#
public async static void LoadClasses(List<Class> classes)
{
    if (scheduler == null)
    {
        var schedulerFactory = new StdSchedulerFactory();
        scheduler = await schedulerFactory.GetScheduler();
    }
    ClearLog();
    await scheduler.Clear();
    foreach (var item in classes)
    {
        scheduler = item.AddToScheduler(scheduler);
    }
    AddLog("加载了" + classes.Count + "个课节, 时间: " + DateTime.Now.ToString());
    await scheduler.Start();
}
```

```c#
public static void ChangeStatus(Status status)
{
    Instance.Dispatcher.BeginInvoke(new Action(() => {
        Instance.StateNowText.Text = "当前状态: " + GetStatusString(status);
        Instance.StateNextText.Text = "下一状态: " + GetStatusString(status + 1);
        if (status == Status.IN_CLASS)
        {
            Instance.ClassStateText.Text = "上课中";
            Instance.IconPic.Kind = MaterialDesignThemes.Wpf.PackIconKind.Book;
        }
        if (status == Status.CLASS_OVER)
        {
            Instance.ClassStateText.Text = "下课中";
            Instance.IconPic.Kind = MaterialDesignThemes.Wpf.PackIconKind.ExitRun;
        }
        if (status == Status.WAIT_FOR_CLASS)
        {
            Instance.ClassStateText.Text = "等待上课中";
            Instance.IconPic.Kind = MaterialDesignThemes.Wpf.PackIconKind.Sleep;
        }
    }));
}
```

至此，我们完成了对于打铃程序的开发，笔者实际上在稍早的时候就已经完成了这个打铃程序项目，已经在Github开放源代码，本文涉及的所有源代码都基于CC-BY 4.0协议开源，如有使用请遵守开源协议

需要注意的是，使用VS编译项目时，默认生成的是64位Win32程序，如果读者的虚拟机操作系统为32位，需要修改目标平台至x86架构

#### 将音视频呈现到腾讯会议中

利用Hyper-V的增强会话功能，我们可以轻松地将宿主机的文件直接复制到虚拟机中，这为我们的部署提供了便利。我们将编译好的打铃软件复制到虚拟机中，并且安装.NET 6.0运行时，保证虚拟机能够运行打铃软件，这里是笔者部署好以后的情况

![图片9.png](https://s2.loli.net/2024/01/26/K3kOvMcXCi7oRbh.png)

为了使SoundPlayer播放的音频正确输入到虚拟声卡中，我们需要将系统输出音频设置为CABLE Input (VB-Audio Virtual Cable)，C#的SoundPlayer将在系统当前默认扬声器中播放声音。

这里读者可能遇到虚拟机的音频仅有“远程音频”这一个选项，这是因为Hyper-V增强会话默认会启用音频转发，将虚拟机中的音频自动转发到宿主机，这显然不满足我们的预期，这里我们可以在连接虚拟机的时候不使用增强会话，即出现“连接至XX”对话框时不点击连接，直接关掉该对话框，即可正确识别虚拟机内安装的虚拟声卡服务。

增强会话同时也会导致关闭虚拟机连接时使远程计算机自动休眠，这会导致腾讯会议的麦克风自动关闭，自动打铃受到阻碍，也可以通过上述方法来禁用增强会话，从而使自动打铃服务可以完全隐藏在宿主机的后台静默运行。

接下来我们需要配置OBS的场景，来将打铃程序的界面呈现到腾讯会议中，注意，为了使OBS支持虚拟摄像头功能，需要安装版本号为26以上的OBS。

Hyper-V默认没有虚拟化NVENC QSV等编码器的能力，这意味着我们开启虚拟摄像头时需要利用CPU的x264进行软件编码，我们最好适量降低OBS的编码质量来达到最大的能效比，这里给出笔者使用的参数来供读者参考。编码器：软件（x264），比特率：200 Kbps，输出分辨率：786x446，缩小方法：双直线法，帧率：10FPS。这里输出分辨率可以根据打铃程序界面的大小进行合理的调整。

之后我们创建“窗口采集”作为来源，并且选中打铃程序的GUI窗体，接下来便能看到OBS中已经成功捕获打铃程序的界面，这里需要注意的是，OBS无法捕捉最小化的窗口，这里我们需要保证打铃程序位于前台。

勾选启动虚拟摄像机并启动腾讯会议，接下来我们将在腾讯会议中应用我们配置的虚拟摄像头和虚拟声卡。

摄像头中选中OBS Virtual Camera，麦克风选中CABLE Output (VB-Audio Virtual Cable)，音频选项可以按照下图设置，以保证铃声音乐播放的流畅性

![图片10.png](https://s2.loli.net/2024/01/26/SgpomVOfCXyIqFL.png)

至此，我们完成了基于Hyper-V和VAC的腾讯会议自动打铃系统。

如果读者有多会议室的需求，也可以克隆虚拟机，同时运行多个虚拟机以承载多个会议室的自动打铃服务，这里笔者不再过多赘述。除此之外，当读者想要关闭虚拟机时，可以使用Hyper-V虚拟机的“保存”功能，该功能将拍摄当前虚拟机状态为一个快照，方便下次启动虚拟机时还原快照，避免重复配置多次。

这里笔者同时运行两台虚拟机来承载自动打铃服务，在笔者i5-9400F的处理器上共占约9%的性能

![图片11.png](https://s2.loli.net/2024/01/26/NsItjbe8qTnBhEF.png)

### 性能调优与反思

#### 性能调优

我们希望打铃服务能尽可能减少性能的占用，以提高能效比，在单位物理机中可容纳更多的打铃服务，这里笔者给出自己的几点看法和思路。

首先，腾讯会议本身会随着会议室人数的增多而占用更多处理单位，我们可以通过关闭接收参会者视频的方式来缓解这个问题。在会议室的常规设置中勾选语音模式，即可暂停接收参会者视频，降低性能消耗。

同时，腾讯会议可能默认会对虚拟摄像头的画面进行降噪或者美颜处理，这也会导致没有必要的性能开销，这里我们进入视频设置，取消勾选所有的对钩，来关闭所有的视频处理功能。

对于OBS，其内置的场景预览也会造成一定的性能消耗，这里可以右键预览的部分，将开启预览的功能关闭。如果读者不需要虚拟摄像头展示打铃情况，仅仅需要播放打铃音乐的功能，也可以跳过OBS的部署，彻底消除编码器造成的性能消耗。

针对操作系统层面，精简程度较高的系统能减少额外开销，如果可以的话可以选择32位的系统来降低内存需求，但这里笔者不推荐使用Windows 7，Windows 8等操作系统，这会降低程序运行的稳定性，因为这些系统已经不再被微软维护，在其环境中运行.NET 6.0程序也可能产生不必要的麻烦。这里笔者使用的是极限精简的Tiny10镜像，这里不再过多赘述

#### 反思

限于个人能力与精力，笔者的方案仍有许多不足之处，Windows系统本身造成的额外开销是难以消除或被忽略的，如果想要扩大自动打铃的规模，或者使用Docker技术统一管理，基于Linux级的实现是不可避免的。

由于腾讯会议提供了Linux版的支持，.NET 6.0也默认具有跨平台的特性，OBS也对Linux平台有支持。仅需找到虚拟声卡的解决方案，就有可能将整个项目部署在Linux上，如果有机会，笔者将就这一课题继续研究。