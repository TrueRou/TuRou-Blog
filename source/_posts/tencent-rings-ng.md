---
title: 基于虚拟化和后台集成的腾讯会议自动打铃管理系统
date: 2023-08-20 11:19:59
tags:
    - 参赛作品
    - 打铃系统
---

**第37届全国青少年科技创新大赛 参赛作品**

**荣获第37届全国青少年科技创新大赛 三等奖**

![1706236406658.webp](https://s2.loli.net/2024/01/26/v3MtlSbHXuWR7nr.webp)

## 研究背景

如今，国内已进入后疫情时代，网课掀起的在线会议浪潮尚未停息。越来越多的教育机构转为线上，腾讯会议成为了许多在线教育平台面对面施教的重要平台。作为腾讯会议网上教学的受益者之一，笔者逐渐对腾讯会议的优缺点有了更深的了解，这个过程中，自动打铃逐渐引起了笔者的注意。

学校的网课教学中，教师往往需要时刻注意当前的时间，使网上教学工作量加大，如果将提示上下课的工作交给学生，也会一定程度影响学生的上课质量和上课效率。在教育机构的网课教学中，课程提醒等需求均需要助教或老师手动进行，增大了工作量的同时不易管理。是否可以搭建一个平台，批量管理多个会议室的打铃日程，同时提供稳定可靠的打铃服务呢？这成为了笔者曾思考的问题。

经过仔细的思考和对相关技术的进一步了解和学习，笔者撰写了此文，希望能对未来的后学有启示作用。笔者开展相关研究经历了两个研究周期，项目也进行了多个版本迭代。从仅基于虚拟化的硬编码日程的打铃软件，到现在基于虚拟化和后台集成的打铃管理系统，需要的相关技术也逐渐多样和丰满，最终构建出了这样一个包含客户端、服务端、网页前端的多实例的打铃管理系统。

## 研究目的

为在线会议室提供一套易部署、高灵活、可复制的自动打铃解决方案，包含前端管理面板、基于后端的日程表系统和VITS语音合成等部分。

## 研究方法

使用从需求到实现的溯源性思维，结合现有的虚拟化(VT-d)技术，虚拟音视频技术，前后端分离的网页开发技术，TCP网络编程等技术，利用C#、Python、JavaScript编程语言开发相关的客户端、前端、后端。并且调用外部VITS接口提供语音合成。

探究过程分为项目速览、概念突破、技术解读、研究过程几个部分，从理论到实践全方位进行探究。


## 项目速览

这里笔者将针对项目中的关键部分简单剖析，使读者快速理解本项目的特点

虚拟化：为多会议实例提供可能。在虚拟机容器中运行腾讯会议和虚拟摄像头、虚拟声卡等程序，将打铃程序的音视频呈递到腾讯会议中。单一物理机经由虚拟化可以承载多个会议室，同时做到了稳定可靠和易扩展的特点。

美观的桌面程序：给参会者直观的课程展示和提醒。精美的UI界面实时在会议室中展示，方便参会者实时了解课程信息，为课程做好准备。

后台集成：解决了各教师实时调整课程表，管理者统一管理的需求。使用服务端统一分发课程表，并且与前端配合，使教师和管理者可以在前端统一查看会议室状态，随时更改课程。

VITS语音合成：使用TTS语音合成进行课程提醒。上课后使用AI语音合成提醒所上课节，可随时切换各角色的语音模型，为枯燥无趣的学习生活加点料。


## 概念突破

### 虚拟化技术与打铃

服务端（也称wakatta后端）作为数据源给各客户端（也称wakaru客户端）提供信息，使各个客户端在相互平行的虚拟机容器中提供打铃服务。

![图片1.png](https://s2.loli.net/2024/01/26/SLQFMxveEOni9V7.png)

### 后台集成

为了实现数据的分发，我们搭建了客户端和服务端，而服务端的数据需要做到可视化地修改，因此我们搭建了网页前端来进行数据的增删改查。这里服务端将作为中转的桥梁，承载所有的网络请求和数据库连接。

![图片2.png](https://s2.loli.net/2024/01/26/er7wlXHLvfWK1AF.png)

客户端采用心跳包模型来保活，每秒向服务端发送请求证明自己的存活，并且从服务端的消息队列中拉取最新的消息（Packet），以保证网页前端调用的部分操作即时抵达客户端。

这里客户端的操作包含刷新日程表、重连、手动播放上下课铃、VITS语音提醒等


### VITS语音合成

VITS属于一种基于机器学习的TTS技术，在上课前我们希望提醒学生所上课节，在网页前端使用语音提醒时希望客户端可以读出我们的文本内容，所以我们使用了VITS

由于承载服务端的VPS往往缺乏较高性能的显卡，语音演算往往需要在其他算力机上进行，这里我们使用了frp进行了反向代理，使客户端经由服务端作为跳板后，可以间接访问到算力机。

![图片3.png](https://s2.loli.net/2024/01/26/pbgKP4YhsUt6yA5.png)

在测试环境中，若算力机与服务端位于同一台主机或处于同一网段，也可省去反向代理的过程


## 技术解读

除虚拟化技术外，我们项目主要由服务端（wakatta）、客户端（wakaru）、网页前端（wakatta-web）以及vits-simple-api构成。

### 服务端（wakatta）

服务端主要承载客户端和网页前端的网络请求，使用Python编写，基于FastAPI异步Web框架开发，遵循restful接口设计范式。由于设计数据的存取和增删改查，数据库选用了高可靠性的MySQL。这里我们使用SQLAlchemy框架做到了数据库的全ORM，便于后续开发，提升了代码的可读性。服务端有时需要执行一些诸如每天更新当天日程表、定时清除失活客户端等方法，这里使用了APScheduler库，做到了定时的计划任务。

下面是服务端部分关键的设计逻辑：

当客户端的课程在网页前端被修改，向后端发出PATCH请求时，后端会先请求数据库的session，然后异步调用SQLAlchemy获取对应课程的ORM对象，进行相关修改并且commit到数据库中。之后会调用位于clients.py的_notify_client方法，将“刷新日程表”这一“包”压入暂存队列中，当下次客户端发送心跳包（最慢1秒）时，将所有暂存的“包”从队列中取出，将包含packet_id和payload的“包”作为response传给客户端。客户端收到后会重新请求日程表，做到刷新课程的效果。

当一天过去时，往往需要加载下一天的日程（比如周一跨到周二），APScheduler会在每天零点调用schedules.py的update_subscription方法，该方法将替换所有订阅了该日程表的客户端的课程，加载对应星期的课程表。同时会调用clients.py的_notify_client方法，要求所有在线客户端刷新日程表来应用最新的课程表。

当修改日程表中的课程时，进行数据库相关操作后，会调用schedules.py的_broadcast_schedule方法，该方法将会把课程更改应用到所有订阅该课程表的客户端中。

### 客户端（wakaru）

客户端部分主要承载打铃服务，并且与后端通信获取最新的日程表，同时需要有美观且直观的界面，方便会议室中的参会人员可以对当前状态和日程一目了然

界面部分采用WPF编写，仅使用xaml标记语言便可搭建出美观的界面布局，并且支持动态绑定和组件化编程。Wakaru中的每一个卡片均对应Views中的一个组件。

逻辑部分采用C#语言编写，基于.net6.0，运行需要准备好.net 6.0运行时。客户端使用Quartz提供定时任务的功能，Quartz不仅提供打铃任务（位于RingJobs.cs）的定时执行，同时也包括每秒发送心跳包（位于WakattaHeartbeat.cs）等过程。

下面是客户端部分关键的设计逻辑：

在客户端程序启动时，会向后端发送get_client请求，后端会返回客户端的所有基本信息，包括当前的日程表、客户端ID和名称、上下课铃等。之后客户端会创建WakattaHeartbeat的定时任务，向服务端每秒发送心跳包。WakattaHeartbeat中同时还包含处理收到的“包”的HandlePacket方法

RingJobs类中存放着所有关于播放上下课铃的相关代码，通过调用SoundPlayer的Play方法，即可实现播放指定音频。若VITS可用，也会在此时向相关算力机发送合成VITS音频的请求，合成的音频传回后也会直接进行播放

### 网页前端（wakatta-web）

网页前端主要提供了美观且便利的数据增删改查操作环境，包含会议管理、日程管理、用户系统等部分。

前端部分使用Vue搭建，UI部分基于element-plus框架和tailwind-css，使用pinia进行数据持久化，axios处理与后端的网络请求。当发送网络请求时，会与后端基于jwt进行鉴权，只有具有相关权限才能修改课程等数据。没有相关权限的用户会被vue-router路由到DeniedView.vue页面中。

### vits-simple-api

为了承载客户端对VITS合成的需求，这里我们需要引入一个项目提供语音合成部分的支持，为了不干扰wakatta后端项目，这里我们引入了一个第三方项目来提供我们的需求，vits-simple-api由Artrajz开发，使用MIT协议开源，使用该项目可快速调用VITS的相关接口
根据需求，我们主要调用了voice/speakers获取模型中所有的角色，调用voice进行语音合成。VITS项目本身需要pytorch和NVIDIA CUDA的支持，这里笔者强烈建议使用一台额外的算力机搭载该项目。

关于模型，我们使用了zomehwh基于公开语音训练的vits-uma-genshin-honkai模型，需要注意的是，该模型不属于本项目的一部分，详见项目仓库中的vits-models目录。

### 关于生产环境

在本地环境调试本项目无需VPS、云服务器等主机，但若要承载多台宿主机中的虚拟化容器，或希望身处不同网络环境中的教师、管理者在家便能编辑课程等信息，项目上云是必不可少的，在后文中笔者会以个人香港服务器为例，介绍如何基于宝塔面板快速将该项目的后端和网页前端上云。

## 研究过程

这部分将着重讲述一下如何从零开始基于HyperV和本地主机打算调试环境，并简要描述如何在生产环境搭建本项目。在研究的过程中，笔者也会探究该项目的思维过程，建立需求引导解决问题的溯源性思维。

### 实现自动打铃
实现自动打铃主要分为两步，即播放音视频和呈现音视频。播放音视频是指将我们预先设置好的音视频在适当的时机播放出来，呈现音视频是指将播放的音视频以某种方式投射或呈现到腾讯会议中。

定时播放音视频是一个十分常见的需求，市面上也有许多现成可复用的解决方案，但对于在线会议来说，我们的播放需求较为复杂，存在切换日程、显示状态、提示课节等需求，这也是笔者决定使用C# + WPF来开发客户端程序的原因。

呈现音视频的难度较高，基于我们的需求分析，我们希望腾讯会议能提供一个接口或者方法来捕捉我们提供的音视频。这里读者可能联想到腾讯会议的屏幕分享功能，笔者当然也考虑并尝试过这种方式。实际上，会议室中同时只有一位用户可以屏幕分享，如果需要对打铃程序使用屏幕分享，就会严重妨碍主持人正常使用屏幕分享功能，这显然违背了我们的初心，为线上教学带来了不便，我们固然需要舍弃这种方案。

反向来思考，怎样做能多个用户共享音视频呢，很容易想到，同学们在会议室中打开摄像头、打开麦克风正是一种“共享音视频”的形式，如果我们能对这种形式加以利用，便可以在不妨碍正常屏幕共享的前提下提供音频播放和视频播放的功能。显然，我们需要“伪装”一个麦克风和摄像头，来让腾讯会议主动地呈现我们的音视频。

伪装麦克风，即虚拟一个麦克风，虚拟麦克风可以模拟一个麦克风硬件，该麦克风将虚拟地捕获到目标音频，这里我们希望将扬声器播放的声音重新传入虚拟麦克风，所以我们使用了Virtual Audio Cable软件(下文简称“VAC”)，该软件可以满足我们的需求

伪装摄像头，即虚拟一个摄像头，虚拟的摄像头模拟了一个摄像头硬件，摄像头捕捉的画面可以由我们提供，这里我们希望客户端程序美观的界面可以被传入摄像头，方便同学们查看课节信息和下课时间，我们使用了开源的Open Broadcaster Software软件(下文简称“OBS”)，该软件可以将预设的场景变为虚拟摄像头，只需将场景中添加窗口捕获，来捕获我们的GUI程序，便可以满足我们的需求了

解决了音视频的播放和呈现之后，另一个问题便浮出水面，显然打铃程序需要运行在一台计算机上，并且该计算机需要让VAC接管音频，使用OBS来捕捉打铃程序的GUI画面。该计算机最好不受外界操作的干扰，单独负责腾讯会议自动打铃的服务。不仅如此，设想一下，我们可能希望为多个会议室提供打铃服务，如何使我们的方案变得可扩展而且高灵活呢。显然，购置多台计算机来分布式完成这个工作是及其不合理的，打铃服务显然不需要极强的计算能力，购置计算机也放大了没有必要的成本，虚拟化技术便是解决该问题的唯一答案。

虚拟化技术可以将一台物理机拆分为几个虚拟层，即做到一台电脑模拟多台电脑的效果。借助Intel的VT-d技术，我们可以高性能地将一台计算机的硬件IO虚拟化，使用统一的层来处理物理机和虚拟机的指令，大大减少虚拟化的性能损耗。这里，我们可以将物理计算机拆分为多台虚拟机，为每一个会议室单独提供虚拟的一台计算机，在虚拟计算机中运行我们上述提到的音视频播放和呈现功能。

Hyper-V是Windows附带的虚拟机平台，它可以批量管理多台虚拟机，并且方便地进行虚拟机复制和容器内操作，并且不需要支付额外的使用费用，这里我们使用HyperV来作为虚拟机容器，实际上，也可以使用VirtualBox等开源免费虚拟机平台。

总结一下，我们可以将实现自动打铃的需求抽象成下面的流程图，这个流程图其实就是上文概念突破中的流程图1中的一部分

![图片4.png](https://s2.loli.net/2024/01/26/U9MdWHIC13spufr.png)


 
接下来我们进行实践，动手创建HyperV环境并且部署客户端相关程序，首先我们要准备HyperV虚拟机容器，并且将所需程序复制入虚拟机容器中

首先需要在Windows的可选功能中开启Hyper-V和虚拟机平台，开启后需要重启计算机，Windows将以Windows更新的方式来安装所需的组件，安装过后，就可以在开始菜单打开Hyper-V管理器了

![图片2.png](https://s2.loli.net/2024/01/26/UqOJZpsmRyEckal.png)

![图片3.png](https://s2.loli.net/2024/01/26/dLyXfitvouheZqn.png)
  
这里笔者的Hyper-V中已经包含了一些虚拟机了，为了创建专门用于自动打铃的虚拟机，我们可以新建一个空白的虚拟机，点击新建 -> 虚拟机来创建，具体参数笔者这里不做过多介绍，仅给出一些推荐参数给读者参考

内存：推荐分配1024-2048MB，防止腾讯会议因内存不足而发生错误。硬盘：推荐分配30GB以上，Hyper-V默认会按需扩张磁盘大小。系统：推荐使用LTSB或LTSC来避免Windows更新，同时缩减因虚拟机系统带来的不必要的性能损失

接下来我们启动创建的虚拟机，完成第一次安装Windows的OOBC过程，直至成功进入系统桌面。之后我们需要安装自动打铃中涉及的一些软件，这部分笔者不做过多赘述，读者可以按照在物理机安装软件的方法来安装软件

接下来我们重点介绍有关客户端开发的内容，由于代码量较大，这里笔者只能抓住几个关键点略讲一二，如若希望深入了解，可以clone本项目仓库，在wakaru目录找到相关代码，也可以在原始实验数据文档中找到所有源代码。

首先，我们先总览一下项目结构，并且关注一下程序的入口
 
![Snipaste_2024-01-26_10-15-36.png](https://s2.loli.net/2024/01/26/yHwrpNS8dl6FcDb.png)

项目主要由Online包、Quartz包、Views包构成，Online包主要包含了与后端的网络请求相关代码，是整个项目数据承载的核心。WakattaClient描述了整个客户端，其中包含客户端的Json模型等内容，在右侧图1可以看出，WakattaClient.Create()方法是项目的入口之一，右侧图2主要展示了客户端创建的过程，在软件启动之后，客户端会尝试获取HardwareID并且向后端请求客户端相关数据，之后会准备日程表和合成相关信息，应用当前上下课状态，并且准备VITS。很容易看出，所有的过程都是异步方法，这意味着所有网络请求不会阻塞客户端的主线程，获取数据都在后台进行。

Quartz包主要包含了关于打铃的相关内容，ScheduleManager类储存了日程表的实例，同时也提供一些便利的方法方便客户端创建打铃的定时任务，RingJobs中存放了打铃的相关实现。这里打铃时会检测VITS是否可用，如果可用，也会将合成的音频在铃声播放过后一并播放

```c#
public class ClassBeginRingJob : RingJob
{
    public override Task Execute(IJobExecutionContext context)
    {
        if (WakattaClient.CurrentClient != null)
        {
            var label = context.Get("label")?.ToString();
            Task.Run(async () => {
                PlaySoundSync(WakattaClient.CurrentClient.ClassBeginRingtone);
                if (label != null) await VITSManager.PlaySoundFormat(label);
            });
            WakattaClient.ApplyStatus();
        }
        return Task.CompletedTask;
    }
}
```

VITSManager类中的PlaySound方法会调用上文提到的vits-simple-api接口，将获得的wav文件暂时储存在内存流中，并且让SoundPlayer播放该内存流。

```c#
public static async Task PlaySound(string content)
{
    if (!vitsObject.Enabled) return;
    if (WakattaClient.CurrentClient == null) return;
    var vitsId = WakattaClient.CurrentClient.VITSId;
    using HttpClient client = new();
    string url = vitsObject.Entrypoint + $"voice?id={vitsId}&text={content}";
    HttpResponseMessage response = await client.GetAsync(url);
    MemoryStream memoryStream = new();
    using (Stream stream = await response.Content.ReadAsStreamAsync())
    {
        await stream.CopyToAsync(memoryStream);
    }
    memoryStream.Seek(0, SeekOrigin.Begin);

    SoundPlayer player = new(memoryStream);
    player.Play();

    response.Dispose();
    memoryStream.Dispose();
}
```

Views包中主要包含了主页中显示的所有卡片对应的组件，这里我们以SchedulePanel为例简要说明一下，SchedulePanel组件中运用了WPF的动态绑定，在ListView中展示所有的课节

```xml
<ListView Name="SchedulerListView">
    <ListView.ItemTemplate>
        <DataTemplate DataType="{x:Type local:WakattaClass}">
            <DockPanel>
                <materialDesign:PackIcon Margin="0 1 5 0" Kind="Book" />
                <TextBlock Text="{Binding Label}" HorizontalAlignment="Center"></TextBlock>
                <Separator Visibility="Hidden" Width="20"/>
                <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
                    <materialDesign:Chip Content="{Binding TimeString}" Padding="0" Margin="0" Height="20"></materialDesign:Chip>
                    <materialDesign:Chip Content="{Binding TimeDurationString}" Padding="0" Margin="0" Height="20"></materialDesign:Chip>
                </StackPanel>
            </DockPanel>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

```c#
public WakattaSchedule()
{
    Instance = this;
    InitializeComponent();
    SchedulerListView.ItemsSource = ClassList;
    SetSelectedIndex(Utils.IndexClass(ClassList));
}
```

只需给ListView的ItemsSource指定一个集合，便可做到动态绑定，当集合中元素被修改时，UI中对应的列表也会随着修改

接下来运行wakaru项目，并且运行必要的服务端，即可看到客户端界面

![图片5.png](https://s2.loli.net/2024/01/26/kos6m2JtSvZKrq3.png)

接下来我们将把客户端部署到虚拟机中，注意的是，由于该项目经历过多次迭代，虚拟机中的UI已经与目前开发进度中的UI样式不同，不过这并不影响部署到虚拟机中这一过程的说明与讲解。

我们将编译好的打铃软件复制到虚拟机中，并且安装.NET 6.0运行时，保证虚拟机能够运行打铃软件，为了使SoundPlayer播放的音频正确输入到虚拟声卡中，我们需要将系统输出音频设置为CABLE Input (VB-Audio Virtual Cable)，C#的SoundPlayer将在系统当前默认扬声器中播放声音。

这里读者可能遇到虚拟机的音频仅有“远程音频”这一个选项，这是因为Hyper-V增强会话默认会启用音频转发，将虚拟机中的音频自动转发到宿主机，这显然不满足我们的预期，这里我们可以在连接虚拟机的时候不使用增强会话，即出现“连接至XX”对话框时不点击连接，直接关掉该对话框，即可正确识别虚拟机内安装的虚拟声卡服务。

增强会话同时也会导致关闭虚拟机连接时使远程计算机自动休眠，这会导致腾讯会议的麦克风自动关闭，自动打铃受到阻碍，也可以通过上述方法来禁用增强会话，从而使自动打铃服务可以完全隐藏在宿主机的后台静默运行。

接下来我们需要配置OBS的场景，来将打铃程序的界面呈现到腾讯会议中，注意，为了使OBS支持虚拟摄像头功能，需要安装版本号为26以上的OBS。

Hyper-V默认没有虚拟化NVENC QSV等编码器的能力，这意味着我们开启虚拟摄像头时需要利用CPU的x264进行软件编码，我们最好适量降低OBS的编码质量来达到最大的能效比，这里给出笔者使用的参数来供读者参考。编码器：软件（x264），比特率：200 Kbps，输出分辨率：786x446，缩小方法：双直线法，帧率：10FPS。这里输出分辨率可以根据打铃程序界面的大小进行合理的调整。

之后我们创建“窗口采集”作为来源，并且选中打铃程序的GUI窗体，接下来便能看到OBS中已经成功捕获打铃程序的界面，这里需要注意的是，OBS无法捕捉最小化的窗口，这里我们需要保证打铃程序位于前台。

勾选启动虚拟摄像机并启动腾讯会议，接下来我们将在腾讯会议中应用我们配置的虚拟摄像头和虚拟声卡。

摄像头中选中OBS Virtual Camera，麦克风选中CABLE Output (VB-Audio Virtual Cable)，音频选项可以按照下图设置，以保证铃声音乐播放的流畅性

![图片10.png](https://s2.loli.net/2024/01/26/SgpomVOfCXyIqFL.png)

至此，我们完成了客户端部分的搭建

如果读者有多会议室的需求，也可以克隆虚拟机，同时运行多个虚拟机以承载多个会议室的自动打铃服务，这里笔者不再过多赘述。除此之外，当读者想要关闭虚拟机时，可以使用Hyper-V虚拟机的“保存”功能，该功能将拍摄当前虚拟机状态为一个快照，方便下次启动虚拟机时还原快照，避免重复配置多次。

### 搭建后端与网页前端

项目仓库地址：https://github.com/TrueRou/wakatta
关于安装Python3和Nodejs开发环境这里笔者不再过多赘述，笔者假设读者具有完成基本软件开发的能力，下面读者可以同笔者一起搭建该项目的后端与网页前端，并且将其部署到生产环境

首先，为了部署项目，需要先拉取前后端的代码，需要注意的是，该项目已经在Github上开源，读者可以Fork或者Clone项目的源码到自己的仓库，或者调试机器、生产环境机器上。注意，本项目遵循GPLv3开源协议。这里的部署代码适用于Linux，如果在Windows上部署大同小异，这里笔者不过多赘述

首先，打开终端，克隆项目并安装相关依赖，并且配置后端的config.py

```bash
git clone https://github.com/TrueRou/wakatta.git
cd wakatta; pip install -r requirements.txt
cd wakatta-web; npm install
```

之后，在wakatta项目下运行python3 main.py即可启动后端项目，在wakatta-web下运行npm run dev即可在调试环境下临时运行前端项目，实际上，我们可以将前端项目使用npm run build语句构建成静态文件，使用nginx进行托管，这部分笔者将在下面生产环境部署中详细讲解

接下来我们将抽出后端项目中的几个细节进行简单讲解，实际上，部分逻辑在上面的技术解读中已经讲解过，这里我们将展示相关的代码。

![图片6.png](https://s2.loli.net/2024/01/26/pFoOl2W7ETB3DIc.png)

在models.py中，我们定义了数据库ORM中的实体类，实际上，根据笔者的经验，使用Python构建前后端项目时，实体类往往有两种存在形式，即服务于SQLAlchemy的实体类，这里笔者习惯命名为models.py，还有服务于FastAPI的Pydantic实体类，这部分主要用于后端请求的类型检查和response_model，这里笔者习惯命名为schemas.py

利用全ORM的好处便是易于指定relationship，这也是关系型数据的关键之一，这里比如客户端应该拥有多个课程，通过外键class.id连接，客户端可能有一个订阅的日程表，通过外键schedule.id连接，这里读者可以自行阅读相关的代码，或者查阅SQLAlchemy的文档，需要注意的是，这里笔者使用的是SQLAlchemy2.0，读者需要自行阅读较新的文档，或者阅读了开启future=True的较旧文档

![图片7.png](https://s2.loli.net/2024/01/26/jyIrkluwpSBA9iP.png)

在sessions.py中, 储存了online_clients列表, client_data字典，以及client_packets字典，client_data主要包含客户端运行返回的实时数据，通常包含在心跳包的请求中，client_packets则是上文提到过的消息缓存队列，会在客户端发送心跳包时取出

tick_session方法会在每秒执行，主要为了踢出不活跃的客户端，保持online_clients列表有效，而tick_client方法则是每次心跳包送达时被执行，主要是为了记录客户端活跃时间，方便超时踢出

![图片8.png](https://s2.loli.net/2024/01/26/wAbhe2Np1t7kYPH.png)

在schedule.py中，当日程表有应用操作时，会调用apply_schedule方法，具体逻辑可以参考技术解读部分

前端部分基于vue3和element-plus，鉴于前端项目较差的组织性，笔者这里不单独展开分析前端的代码，仅放出前端的部分界面，使读者感受到前端的可操作性

![图片9.png](https://s2.loli.net/2024/01/26/mHuaXIpfFqVOxSg.png)

![图片10.png](https://s2.loli.net/2024/01/26/JwKpmT2AkeYSOl8.png)

![图片11.png](https://s2.loli.net/2024/01/26/VXY7Il4n9d5zHLh.png)

![图片12.png](https://s2.loli.net/2024/01/26/nwf84Q9WPYjgD1t.png)

![图片13.png](https://s2.loli.net/2024/01/26/zsFC2OeZMX4akKL.png)

![图片14.png](https://s2.loli.net/2024/01/26/C7Za345LDEdQNXS.png)

![图片15.png](https://s2.loli.net/2024/01/26/4qgsL6ZSxRJtDmv.png)

### 生产环境搭建

接下来我们讲述一下如何在生产环境搭建该项目，我们主要以反向代理nginx和frp的配置为主，其他搭建方式可以参考上文调试环境搭建的相关内容

在配置前，请确保云服务器已经搭载宝塔面板，并且安装了Nginx和MySQL的较新版本，这里笔者不推荐使用MySQL8，MySQL8的性能弱于5.5。之后，将MySQL生成的账号密码填入wakatta项目的config.py中，之后运行后端项目

接下来我们为后端项目配置反代，我们希望在“https://域名/wakatta/api”路径中访问到后端，所以我们修改对应域名的配置文件

![图片16.png](https://s2.loli.net/2024/01/26/TmepUhZogOqlQKV.png)

使用nginx提供的proxy_pass，即可完成反代

接下来我们部署网页前端，进入wakatta-web目录，输入npm run build来构建出静态页面，并且把静态页面复制到宝塔对应网站目录下，这里需要注意的是，请保证前端项目config.js中对应的后端地址已经指向生产环境的后端地址，而不是之前调试环境中的。为了免除不必要的跨域问题，请不要把API和前端放在不同域名下，在nginx中，我们可以使用alias进行目录的路由


![图片17.png](https://s2.loli.net/2024/01/26/Vng7PXcLKtabkYI.png)

这样当我们访问https://域名/wakatta，就会被跳转到我们的静态页面中，至此，前后端的部署完毕。

Frp的配置要点主要在于配置frps.ini和frpc.ini，这里限于篇幅，笔者决定不再展开叙述，这部分内容读者可以自行阅读frp的相关文档

### 后记

#### 性能调优

我们希望打铃服务能尽可能减少性能的占用，以提高能效比，在单位物理机中可容纳更多的打铃服务，这里笔者给出自己的几点看法和思路。

首先，腾讯会议本身会随着会议室人数的增多而占用更多处理单位，我们可以通过关闭接收参会者视频的方式来缓解这个问题。在会议室的常规设置中勾选语音模式，即可暂停接收参会者视频，降低性能消耗。

同时，腾讯会议可能默认会对虚拟摄像头的画面进行降噪或者美颜处理，这也会导致没有必要的性能开销，这里我们进入视频设置，取消勾选所有的对钩，来关闭所有的视频处理功能。
对于OBS，其内置的场景预览也会造成一定的性能消耗，这里可以右键预览的部分，将开启预览的功能关闭。如果读者不需要虚拟摄像头展示打铃情况，仅仅需要播放打铃音乐的功能，也可以跳过OBS的部署，彻底消除编码器造成的性能消耗。

针对操作系统层面，精简程度较高的系统能减少额外开销，如果可以的话可以选择32位的系统来降低内存需求，但这里笔者不推荐使用Windows 7，Windows 8等操作系统，这会降低程序运行的稳定性，因为这些系统已经不再被微软维护，在其环境中运行.NET 6.0程序也可能产生不必要的麻烦。这里笔者使用的是极限精简的Tiny10镜像，这里不再过多赘述

#### 后记

从2023年2月，笔者完成了Wakaru的第一个版本，到2023年5月，笔者将项目完全重构。从单一的C# + WPF桌面项目，到综合三个语言的客户端+后台集成项目，笔者在紧张的开发周期中也学到了许多，获得了许多，笔者认为，创新本身不仅仅在于研究结果，更在于创新的研究思路和研究过程。在这个过程中，我们创新的不仅仅是知识本身, 更是获取知识的方式。