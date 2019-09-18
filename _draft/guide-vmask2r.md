## [优选方案] 搭建 v2r 网络环境

### 前提

请确保你已经了解下面的主题，如果遇到你不了解的主题，请先去百度。

- 什么是 VPS
- 如何购买 VPS
- 什么是 JSON

### 简介

v2r 的搭建可以参考官方文档，说明比较详尽，但指导性不是很强，如果你第一次看这篇文档，可能会因为不清楚需要配置什么而云里雾里。

> Mask：官网是其全称加 `.com` 后缀。

官方文档中有推荐一篇教程，从新手的视角引导我们一步一步搭建出一个刚好能用的配置，然后再递进的说明了大部分设置的配置方法和用途。

https://toutyrater.github.io/

这篇教程最大的用处还是让我们对 v2r 的配置文件结构和含义有一个整体上的概念。但是摸清楚套路之后，还是建议按需去看官方文档。毕竟第三方教程更新速度可能是个问题，而 v2r 又是一个周更的软件。

这里记载一个安装方式和一般的配置方式，但由于其更新速度可能很快就会过时，所以建议还是以理解配置需求为主，了解自己需要什么样的配置之后，再去官方文档查看最新的配置方式。

和 SS 一样，v2r 也需要服务器端和客户端两边的配置相匹配才能工作。出于诞生缘由和面向用户的不同，v2r 在某些方面有着独到的优势，它弥补了一些 SS 的缺陷，集成了 kcp 等实用的功能。并且，v2r 自带兼容 SS 的属性，这意味着如果你的服务器搭建的是 SS 环境，你可以使用 v2r 的客户端连接 SS 服务器。

### 服务器端配置

#### 安装

先说服务器的系统选择，之前使用 SS 时我一般选择 CentOS，原因是我使用的一键安装脚本的作者是使用的这个发行版的系统，我在这个发行版上使用一键安装脚本不会出现任何问题。

而 v2r 官方建议使用 Debain or Ubuntu。这并不是在说其他发行版无法使用这个一键安装脚本，如果你有能力解决其它系统上可能会出现的问题，你使用任何系统都没有问题，但是如果你无法做到，还是建议使用官方推荐的系统。

这里我使用的是 Debian 8.x 的发行版。

v2r 本体的下载可以选择从官方 repo 拉取最新的预编译压缩包，也可以使用官方提供的一键安装脚本。建议使用后者，这里也仅以后者作为演示。

> 官方也提供预编译的 Docker image 文件，理论上你可以不用关注兼容性，拉取最新的 image 文件就可以使用完全的 v2r 功能，如果你熟悉 Docker 可以前往 [v2r/official](https://hub.docker.com/r/v2r/official/) 拉取最新的稳定版本。本文暂时不关注 Docker 版本的使用。

运行下面的命令下载一键安装脚本并且运行。

```shell
$ bash <(curl -L -s https://install.direct/go.sh)
```

安装结束后，程序本体会被安装在下面的路径。

```shell
/usr/bin/v2r/
```

配置文件在下面的路径。

```shell
/etc/v2r/config.json
```

> 一键安装脚本会自动检测 v2r 的安装情况，如果没有安装则自动安装最新的稳定版程序，如果已经安装则自动更新到最新的稳定版本。

> 据官方提示：目前自动运行脚本只支持带有 Systemd 的系统，以及 Debian / Ubuntu 全系列。

> 脚本运行完成后，你需要：
>
> - 编辑 /etc/v2r/config.json 文件来配置你需要的代理方式；
> - 运行 service v2r start 来启动 v2r 进程；
> - 之后可以使用 service v2r start|stop|status|reload|restart|force-reload 控制 v2r 的运行。

#### 配置文件

v2r 的配置文件出于方便阅读的考虑默认使用 JSON with comment 格式。JSON 源于 JavaScript 的对象标记，后来逐渐成为前后端通信数据格式的首选，本文默认你已经了解什么是 JSON，如果你并不了解，请自行百度。

v2r 的配置文件格式大致如下。服务器端和客户端的配置格式是一致的，区别在于入口和出口的配置内容不同。

```json
{
  "log": {},
  "api": {},
  "dns": {},
  "stats": {},
  "routing": {},
  "policy": {},
  "reverse": {},
  "inbounds": [],
  "outbounds": [],
  "transport": {}
}
```

我们只需要定义我们需要的配置，对于不需要的配置对象，可以省略。

After settings in the outbound.

```json
  "streamSettings": {
    "network": "kcp",
    "security": "none",
    "tlsSettings": {
      "allowInsecure": true
    },
    "kcpSettings": {
      "readBufferSize": 1,
      "downlinkCapacity": 100,
      "mtu": 1350,
      "header": {
        "type": "utp"
      },
      "writeBufferSize": 1,
      "uplinkCapacity": 5
    }
  }
```
