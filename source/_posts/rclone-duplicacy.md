---
title: Rclone搭配Duplicacy实现小文件高效备份到OneDrive (世纪互联)
date: 2024-02-14 11:00:04
tags:
    - OneDrive
    - 文件备份
    - 杂谈
---

Duplicacy是一个开源的, 基于无锁去重算法开发的云备份工具, 他的核心思路类似Git等版本控制软件

- 多台电脑可以备份/同步到同一(云)储存 (类似远程Repo和本地Repo的关系)
- 可以将工作区还原到任意一个Revision (类似把Origin切换到任意一个Commit)
- 支持文件加密, 压缩, 打包, 快照, 同步云服务等功能

Duplicacy直接支持下面这些(云)存储方式

* Local disk
* SFTP
* Dropbox
* Amazon S3
* Wasabi
* DigitalOcean Spaces
* Google Cloud Storage
* Microsoft Azure
* Backblaze B2
* Google Drive
* **Microsoft OneDrive (支持Personal, Business, 不支持世纪互联)**
* Hubic
* OpenStack Swift
* WebDAV (under beta testing)
* pcloud (via WebDAV)
* Box.com (via WebDAV)
* File Fabric by [Storage Made Easy](https://storagemadeeasy.com/)

Rclone 是一个用于多个云平台之间同步文件和目录的命令行工具, 他的定位于Duplicacy不同

- 支持将本地文件以Sync, Move, Copy等方式直接上传到云储存
- **支持利用VFS技术将云储存挂载为本地磁盘**
- **支持OneDrive世纪互联**

本文将利用Rclone的VFS虚拟磁盘, 配合Duplicacy, 实现小文件高效备份到OneDrive世纪互联的效果

## Rclone

### 下载

首先我们要下载并配置Rclone, 这里我们以Windows和Linux为例

#### Windows

首先, 我们要在Github下载二进制文件, 读者需要选择合适的架构, 这里我们选择amd64

- Rclone: https://github.com/rclone/rclone/releases

下载后将二进制文件改名为rclone.exe, 丢入C:\Windows\System32

为了后文挂载虚拟磁盘, Windows系统需要安装WinFSP来提供VFS的功能, 下载后按照步骤安装就可以了

- WinFSP: https://winfsp.dev/rel/

> 如果读者有安装过天翼云盘, 那么就不必再次安装WinFSP了, 安装天翼云盘的时候会自动帮你安装好WinFSP

#### Linux

```bash
curl https://rclone.org/install.sh | sudo bash
```

为了后文挂载虚拟磁盘, Linux需要安装fuse, 请根据发行版选择正确的安装方式

```bash
# Debian/Ubantu
apt-get update && apt-get install -y fuse
# CentOS
yum install -y fuse
```

在任意位置打开终端, 输入rclone, 应该已经能看到帮助信息了

### 配置存储库

#### 创建Azure应用

这里我们将使用Rclone在本地挂载远端OneDrive, 然后使用Duplicacy在虚拟磁盘中创建存储库

我们首先需要先配置OneDrive的相关鉴权行为, 允许Rclone访问我们的OneDrive.

访问 https://portal.azure.cn/ 登录完成后按下面步骤进行

1. 访问 https://portal.azure.cn/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade -> 点击新注册

2. 名称: 任意名称 -> 受支持的账户类型: **任何组织目录(任何 Azure AD 目录 - 多租户)中的帐户** -> 重定向 URI: http://localhost:53682 -> 注册

![image.png](https://s2.loli.net/2024/02/14/MWtvcQJGEB9IVy2.png)

3. 注册后请记住并保存这里的 应用程序(客户端) ID, 这是我们一会要用到的 **client_id**

![image.png](https://s2.loli.net/2024/02/14/OL9Ua2nVlkYXWBJ.png)

4. 点击左侧的API权限 -> 点击蓝色的Microsoft Graph (1) -> 在搜索框中搜索并允许下面这些权限

```bash
Files.Read
Files.Read.All
Files.ReadWrite
Files.ReadWrite.All
offline_access
User.Read
```

勾选后, 应该是这样的效果

![image.png](https://s2.loli.net/2024/02/14/sIvQu9prRC3iL4N.png)

5. 点击左侧的证书和密码 -> 新客户端密码 -> 任意输入说明 -> 截止期限选24个月 -> 点击添加

6. 添加后请记住并保存这里的 **值**, 这是我们一会要用到的 **client_secret**

![image.png](https://s2.loli.net/2024/02/14/1kZnCUpeGucFxRz.png)

#### 添加Rclone远程存储库

1. 在任意位置打开终端, 输入`rclone config`, 输入n新建一个远程存储库

2. name> 输入一个你喜欢的名称(这个名称也会成为挂载后的磁盘名称)

3. Storage> 根据提示, 找到**Microsoft OneDrive**的序号并填入

4. client_id> 填入我们上文刚刚获取到的client_id

5. client_secret> 填入我们上文刚刚获取到的client_secret

6. region> 这里选择 **Azure and Office 365 operated by Vnet Group in China** 的序号

7. Edit advanced config? 这里输入n, 不进行进阶的配置

8. Use auto config? 这里输入y, 自动进行Token获取

9. 接下来, 浏览器应该会弹出世纪互联账户的登录页面, 按照要求进行登录就可以了

10. Your choice> 这里输入1, 使用常规的OneDrive Personal or Business

11. Chose drive to use:> 这里输入0, 因为只能输入0 (bushi

12. Is that okay? 这里确认一下输入的信息无误后, 输入y

13. 接下来Rclone会进行自动获取Token生成配置文件的过程, 当获取完毕后, 会再次让你确认是否无误, 输入y确认

14. 确认后应该已经能看到自己新配置的存储库了

![image.png](https://s2.loli.net/2024/02/14/RlwAicfVHvZYkWC.png)

### 挂载虚拟磁盘

在任意位置打开终端, 输入`rclone mount <存储库名称>:/ Z:`, 应该就能在资源管理器中看到刚刚挂载的Z盘了

当我们关闭终端或者Ctrl+C结束当前命令的时候, 挂载的盘符消失

关于rclone mount的缓存相关配置, 可以阅读下面笔者的扩展阅读部分, 这里仅给出笔者的配置作为参考

如果要使用Duplicacy, 请务必保证**缓存目录的剩余空间大于要备份文件的总大小**

```bash
rclone mount OneDrive:/ Z: --cache-dir <缓存目录> --vfs-cache-mode writes --vfs-cache-max-size 30G --vfs-cache-max-age 1h
```

接下来我们要配置后台执行和开机启动, 这里我们以Windows和Linux系统为例

#### Windows

这里我们使用WinSW来配置Windows自动启动服务, 在Github下载WinSW

- WinSW: https://github.com/winsw/winsw/releases

下载后将二进制文件改名为winsw.exe, 放到rclone的配置文件所在目录 (%AppData%\rclone)

在相同的目录创建rclone-mount.xml, 填写以下内容

需要注意, 这里rclone的配置文件需要手动指出, 这里笔者已经填入了默认配置文件的位置, --config参数不可删除

```xml
<service>
    <id>rclone</id>
    <name>rclone</name>
    <description>This service run rclone mount network file systems to local disk</description>
    <executable>rclone</executable>
    <arguments>mount OneDrive:/ Z: --config %BASE%\rclone.conf --vfs-cache-mode writes --vfs-cache-max-size 30G --vfs-cache-max-age 1h</arguments>
    <log mode="roll"/>
</service>
```

使用终端打开当前目录, 输入`winsw install rclone-mount.xml`

现在, 这个目录的结构大概应该是这样的, 同时, 如果读者打开资源管理器, 应该已经看到挂载的盘符了

![image.png](https://s2.loli.net/2024/02/14/WlarEdiZFM15gYx.png)

#### Linux

Linux推荐使用systemctl配置开机自启服务, 这里我们创建rclone-mount服务

```bash
vi /usr/lib/systemd/system/rclone-mount.service
```

填入并按需修改以下内容

```bash
[Unit]
Description=This service run rclone mount network file systems to local disk
After=network.target

[Service]
Type=simple
User=root
ExecStartPre=/bin/sh -c 'until ping -c1 baidu.com; do sleep 1; done;'
ExecStart=rclone mount OneDrive:/ /mnt/OneDrive --allow-other --allow-non-empty --vfs-cache-mode writes --vfs-cache-max-size 30G --vfs-cache-max-age 1h
ExecStop=fusermount -u /mnt/OneDrive

[Install]
WantedBy=multi-user.target
```

之后, 我们配置服务的开机自启: `systemctl enable rclone-mount.service`

## Duplicacy

### 下载

接下来我们下载Duplicacy, 这里我们依旧以Windows和Linux为例

#### Windows

- Duplicacy: https://github.com/gilbertchen/duplicacy/releases

下载后将二进制文件改名为duplicacy.exe, 丢入C:\Windows\System32

#### Linux

```bash
wget -O /usr/local/bin/duplicacy https://github.com/gilbertchen/duplicacy/releases/download/v3.2.3/duplicacy_linux_x64_3.2.3
chmod 0755 /usr/local/bin/duplicacy
```

在任意位置打开终端, 输入duplicacy, 应该已经能看到帮助信息了

### 配置存储库

Duplicacy把位于计算机本地的待备份目录称为Repository, 这里笔者在E盘创建了一个名为Vault的文件夹, 我们接下来将对这个文件夹进行备份

首先, 我们需要先切换到Vault文件夹中, 并且创建Duplicacy的仓库, 这里vault为仓库的标识名, 后面的路径为远程库的路径

这里我们使用上文创建好的Rclone虚拟磁盘作为远程库.

```bash
cd E:\Vault
duplicacy.exe init -c 128M -min 32M -max 1024M vault Z:\Vault
```

Duplicacy使用Hash块来储存打包后的文件, 我们通过-c -min -max参数来指定单个块的大小.

块过小会导致文件零散, 上传时性能比较低, 块过大会导致微小的更改就需要重建整个块, 这里请读者根据需要进行修改.

如果读者的数据不常修改, 推荐使用稍大一些的块

### 进行备份

备份的线程越多, 占用的CPU和内存越多, 备份速度越快.

这里32线程已经能吃到16G以上的内存, 请读者酌情配置.

```bash
cd E:\Vault
duplicacy.exe backup -threads 32
```

### 更多操作

这里笔者就不过多介绍Duplicacy了, 如果读者希望进一步了解使用, 欢迎阅读其他博主的文章.
Duplicacy CLI 备份工具的基本使用: https://blog.dejavu.moe/posts/duplicacy-cli-basic-guide/

## 扩展阅读

### Rclone 传输优化

当传输大文件时, 为了让Rclone的传输速率达到最大, 这里我们可以配置下面几个参数

要注意的是, 过度拉高这些参数可能会导致高内存占用, 不建议在VPS等性能较弱的机器上调整

#### --cache-chunk-size

The size of a chunk (partial file data).

Use lower numbers for slower connections. If the chunk size is changed, any downloaded chunks will be invalid and cache-chunk-path will need to be cleared or unexpected EOF errors will occur.

#### --transfers

文件并行数量, 不建议设置太多, 即使多线程, 也受OneDrive的速度限制, 这里建议测试一下, 寻找能达到最大速度的最少线程数

#### --onedrive-chunk-size

Onedrive的chunk-size设置 必须是320KB的倍数, 取100MB适合G口服务器, 如果我们的接口速率达不到那么大, 没必要设置的太大

这里笔者仅给出自己的配置作为参考

```bash
rclone sync Vault OneDrive:Vault -P --cache-chunk-size 4M --transfers=16 --onedrive-chunk-size 10240
```

### Rclone VFS优化

`rclone mount`关于缓存方式有下面几个参数, 我们这里直接对重要的几个参数进行讲解

#### --vfs-cache-mode

vfs-cache-mode: 虚拟磁盘的缓存方式, 主要有下面几种选择

- --vfs-cache-mode off: 所有读取和写入都直接操作远程
- --vfs-cache-mode minimal: 只读和只写打开的文件直接从远程读取, 读 / 写的文件首先缓冲到磁盘.
- --vfs-cache-mode writes: 只读打开的文件直接从远程读取, 只写和读 / 写的文件首先缓冲到磁盘.
- --vfs-cache-mode full: 所有读取和写入都缓冲到磁盘

off < minimal < writes < full, 需要注意的是, OneDrive只支持大于等于writes的模式

这里更为详细的介绍推荐阅读官网的相关文档: https://rclone.org/commands/rclone_mount/

这里笔者只针对几个情形进行针对性的讲解和分析: 

- writes模式: 适合磁盘空余空间较大的读者, 适合除了Duplicacy备份, 还想平常使用OneDrive的读者. 当使用writes模式向OneDrive上传文件时, 这些文件会首先储存在本地VFS的缓存目录中慢慢上传, 等文件全部上传到云端后, 再释放掉缓存. 这会导致如果用户第一次使用Duplicacy备份100G的文件时, 缓存目录会先占满约100G的空间, 再慢慢释放.

- off模式: 适合磁盘没有空余空间的读者, 或者要备份的文件占用空间过大的读者. 但可惜OneDrive不支持流式传输, 所以读者在使用OneDrive时不能选择这个模式

#### --vfs-cache-max-size

当启用缓存时, 文件缓存的最大占用空间. 这个最大空间限制**只对从远端读取时生效**, 例如, 如果你向网盘中复制100G的文件, 缓存目录会先占用约100G的空间, 然后再慢慢释放

#### --vfs-cache-max-age

当启用缓存时, 文件缓存的最大存活时间. 这个最大时间限制**只对从远端读取时生效**, 例如, 如果你观看网盘中的电影, Rclone会从网盘中边下边播这部电影, 在超过存活时间后, 再次观看, 又要从头开始重新下载

#### --buffer-size

设置内存缓存大小, 这里如果日常使用建议100M左右就够了, 如果只作为Duplicacy备份使用, 那建议不配置

#### --cache-dir

文件缓存的目录, 如果不设置, 默认目录为C:\Users\chenb\AppData\Local\rclone\

如果要使用Duplicacy, 请务必保证**缓存目录的剩余空间大于要备份文件的总大小**

这里笔者仅给出自己的配置作为参考, 注意--vfs-cache-mode参数在使用OneDrive时必须大于等于writes

```bash
rclone mount OneDrive:/ Z: --vfs-cache-mode writes --vfs-cache-max-size 30G --vfs-cache-max-age 1h
```

### Duplicacy世纪互联

Duplicacy论坛有这样一个帖子: https://forum.duplicacy.com/t/add-support-for-onedrive-business-china/3869

这个博主只是修改了登录的Entrypoint, 没有修改client_id, 因为Duplicacy在世纪互联没有注册相应的应用, 必然不能用

我们按照上文的方法创建自己的client_id并授予响应的权限、打开响应的回调URL后, 再次尝试. 这次我们成功从微软相关网站跳转到了Duplicacy的odb_start页面, 但是odb_start无法获取到相应的token和refresh_token. 因为OAuth回访获得token也需要传入对应的client_id, client_secret, 但是odb_start中没有实现用户侧填写这些信息.

再次整理后, 我们发现, 可以手动获取token, refresh_token并且填入json中, 将json提供给Duplicacy. 但由于refresh_token的过程仍需响应的client_id, client_secret. 这意味着我们需要使用外部的一个程序负责给Duplicacy获得的token进行续约操作. 这种操作比较复杂.

希望Duplicacy早日提供支持世纪互联的接口, 或者将client_id, client_secret侧做的更灵活.