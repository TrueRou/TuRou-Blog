---
title: 基于 Oven Media Engine 低成本部署亚秒级直播平台
date: 2026-03-17 14:56:52
categories: [rambling]
tags:
    - 杂谈
---

# 基于 Oven Media Engine 低成本部署亚秒级直播平台

OvenMediaEngine (OME) 是一款亚秒级延迟、支持大规模高清直播的服务器。借助 OME，您可以创建平台/服务/系统，以亚秒级延迟向数十万观众传输高清视频，并可根据并发观众数量进行扩展。

这里笔者将带领你从选购 VPS 开始，使用 OME 快速搭建一个亚秒级直播平台，适合与朋友共同观影使用，或者在小范围内进行直播。

## 0. 准备工作

1. 良好的网络环境，同时[让 VPS 也拥有良好的网络环境](https://github.com/nelvko/clash-for-linux-install)
2. 我们建议以全新的 VPS 实例来部署 OME，如果你想在已有的 VPS 上部署，请注意安全组相关设置，不要随意关闭防火墙

## 1. 选购 VPS

首先，为了承载一人推流，多人拉流的高带宽需求，建议选择带宽较大的 VPS，这里笔者比较推荐的是阿里云 200M 峰值带宽的轻量应用服务器。虽然在周末等业务繁忙的时间段可能会有一定的网络波动，但对于个人使用来说已经足够了。

如果是学生可以领取 300 元云工开物的优惠券，可以免费白嫖半年。也可以准点抢购 38 元一年同配置的服务器，性价比非常高，如果抢不到可以选择 68 的，实际上是与 38 的一样的。

![11ffc839-d14c-4f82-8058-868b547ff222.png](https://files.seeusercontent.com/2026/03/17/kaI6/11ffc839-d14c-4f82-8058-868b547f.png)

这里我们选择 Debian 12 的系统实例，地区建议选择离你较近的，或者是与你和观众所处地理位置构成的较小的三角形的地区，这样可以减少网络延迟。提交后等待实例创建。

## 2. 设置 VPS

### IP 地址 & 密码

实例创建完成后，进入实例详情页，点击重置密码:

![Snipaste_2026-03-17_12-14-56.png](https://files.seeusercontent.com/2026/03/17/8oXc/Snipaste_2026-03-17_12-14-56.png)

在红框处可以找到公网 IP 地址，后续我们将使用这个地址来连接 VPS 和访问 OME 服务，记录下来

密码设置方面，我们非常推荐[随机生成一个密码](https://1password.com/zh-cn/password-generator)，我们选择20位随机，设置并记录下来

### 关闭安全组

如果仅使用 OME 搭建直播站，安全组的配置必要性不大，反而可能会因为配置不当导致无法访问 VPS，所以这里我们直接关闭安全组：

点击防火墙 - 添加规则 - 应用类型选择全部 TCP + UDP - 点击确认添加：

![pasted-image-1773721571838.webp](https://files.seeusercontent.com/2026/03/17/t7Mh/pasted-image-1773721571838.webp)

### 连接主机

如果你还不知道如何登录 Linux 主机，这里推荐安装 [Termius](https://termius.com/) 工具。

在 Termius 中添加主机，输入记录的公网 IP 地址、密码，用户名 root，这样我们就可以通过 Termius 连接到 VPS 上了，连接后，我们先安装一些必要的工具：

```bash
apt update && apt install -y curl wget git unzip ca-certificates
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

## 3. 安装 OME

下面的内容参考 [OME 官方文档](https://docs.ovenmediaengine.com/getting-started/getting-started-with-ome-docker-launcher)，我们将使用 OME Docker Launcher 来部署。

连接至主机后，执行以下命令来安装 OME Docker Launcher，如果等待时间过长，请检查 准备工作 中提到的网络环境问题（建议使用TUN模式）：

```bash
curl -OL 'https://raw.githubusercontent.com/AirenSoft/OvenMediaEngine/v0.20.0/misc/ome_docker_launcher.sh' && chmod +x ome_docker_launcher.sh
./ome_docker_launcher.sh setup
```

当在命令行中看到 `OvenMediaEngine is ready to start!` 的提示时，说明 OME 已经安装完成了，接下来我们可以启动 OME 服务：

```bash
OME_HOST_IP=YOUR_IP ./ome_docker_launcher.sh start
```

![pasted-image-1773725836508.webp](https://files.seeusercontent.com/2026/03/17/Nmk7/pasted-image-1773725836508.webp)

看到对应端口即可说明 OME 已经成功启动了，接下来我们就可以使用 OBS 进行推流了。

## 4. 推流设置

我们推荐下面几个推流方案，推流几乎不影响直播延迟：

1. RTMP：最常见的协议，兼容性好，几乎所有直播软件都支持。
2. SRT：说是适用于网络不稳定的环境，提供优化，并且支持 HEVC 编码。

这里笔者测试以后，二者的延迟没有明显差距，但是 SRT 支持 HEVC 编码，保证画质的情况下可以适当降低码率，这里笔者比较推荐。

- 使用 RTMP 推流（推荐使用OBS）：`rtmp://8.137.114.514:1935/app/stream`
- 使用 SRT 推流（推荐使用OBS）：`srt://8.137.114.514:9999?streamid=vhost/app/stream`

可以参考的设置：

![pasted-image-1773726820761.webp](https://files.seeusercontent.com/2026/03/17/9Sfu/pasted-image-1773726820761.webp)

![pasted-image-1773726719654.webp](https://files.seeusercontent.com/2026/03/17/a6Yd/pasted-image-1773726719654.webp)

## 5. 拉流设置

拉流对延迟的影响很大，为了实现亚秒级的直播延迟，这里只推荐使用 WebRTC 协议进行拉流。

WebRTC 可以实现 1 秒以下的直播延迟，HLS 的延迟通常在 2-5 秒左右，SRT 不支持直接在浏览器播放，这里笔者没有进行测试。

- 使用 WebRTC 拉流（推荐使用OvenPlayer）：`ws://8.137.114.514:3333/app/stream`
- 使用 HLS 拉流（推荐使用OvenPlayer）：`http://8.137.114.514:3333/app/stream/llhls.m3u8`

这里推荐使用 OvenPlayer 来播放，OvenPlayer 是 OME 官方推出的播放器，支持多协议播放，兼容性好，性能优越，这里我们可以先测试一下我们的部署：

1. 使用 OBS 推流到 OME：使用 SRT 推流 `srt://8.137.114.514:9999?streamid=vhost/app/stream`，IP 地址替换为你的 VPS 公网 IP 地址。
2. 使用 OvenPlayer 拉流：在浏览器中打开 `http://demo.ovenplayer.com/`，在源处输入 `ws://8.137.114.514:3333/app/stream`，点击播放，如果能够看到视频画面，说明部署成功了。

> 注意：如果无法播放，请注意 demo.ovenplayer.com 的前缀是 HTTP 而不是 HTTPS，因为我们的 WebRTC 没有使用安全连接，如果你在 HTTPS 的页面中使用 HTTP 的地址，浏览器会阻止加载不安全的内容。

## 6. 更好的体验

到这里，我们已经成功搭建了一个亚秒级直播平台，但是每次都需要用户输入 IP 地址和流名称来推流和拉流，体验并不好，我们可以通过一些简单的前端页面来优化这个体验。

为了创建一个简单的网页展示给朋友们，我们可以先安装 Nginx 来托管我们的前端页面：

```bash
apt install -y nginx && cd /var/www/html && rm -rf * && wget https://raw.githubusercontent.com/TrueRou/PureOvenPlayer/refs/heads/main/index.html
nano /var/www/html/index.html
```

在 `index.html` 的前半部分可以找到配置项，我们将其中的 `demo.ovenplayer.com` 替换为我们的 VPS 公网 IP 地址：

![pasted-image-1773730635894.webp](https://files.seeusercontent.com/2026/03/17/3hJc/pasted-image-1773730635894.webp)

修改完成后保存，然后重新载入 Nginx 即可：

```bash
nginx -s reload
```

现在我们就可以通过访问公网地址 `http://8.137.114.514/` 来观看直播了，当然你也可以将这个地址分享给你的朋友们，让他们也能观看直播。