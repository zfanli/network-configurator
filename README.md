# netwrok-configurator

网络配置 guide or tool。

由于暂时没折腾软路由，每台机器的网络配置都是独立的，并且每次更换环境都要做一次冗长的配置，重复阅读各种资料...所以这里单独对各种网络工具的配置方法做一个记录，如果可以的话做成一个一键安装工具。

目前正在总结配置 guide。只针对科学上网的网络配置。

## 工具选择

仅个人观点。

### SS

用了近两年 SS，中间多次更换 VPS 重新配置，所以较为熟悉流程，依旧作为一个备选方案记录。

### V2Ray

V2Ray 由于诞生缘由导致其特性上天生优于 SS，也必然是未来科学上网的趋势。另一方面其配置相比 SS 来说稍微繁琐一点。

## [优选方案] 搭建 V2Ray 网络环境

### 前提

请确保你已经了解下面的主题，如果遇到你不了解的主题，请先去百度。

- 什么是 VPS
- 如何购买 VPS
- 什么是 JSON

### 简介

V2Ray 的搭建可以参考官方文档，说明比较详尽，但指导性不是很强，如果你第一次看这篇文档，可能会因为不清楚需要配置什么而云里雾里。

https://www.v2ray.com/

官方文档中有推荐一篇教程，从新手的视角引导我们一步一步搭建出一个刚好能用的配置，然后再递进的说明了大部分设置的配置方法和用途。

https://toutyrater.github.io/

这篇教程最大的用处还是让我们对 V2Ray 的配置文件结构和含义有一个整体上的概念。但是摸清楚套路之后，还是建议按需去看官方文档。毕竟第三方教程更新速度可能是个问题，而 V2Ray 又是一个周更的软件。

这里记载一个安装方式和一般的配置方式，但由于其更新速度可能很快就会过时，所以建议还是以理解配置需求为主，了解自己需要什么样的配置之后，再去官方文档查看最新的配置方式。

和 SS 一样，V2Ray 也需要服务器端和客户端两边的配置相匹配才能工作。出于诞生缘由和面向用户的不同，V2Ray 在某些方面有着独到的优势，它弥补了一些 SS 的缺陷，集成了 kcp 等实用的功能。并且，V2Ray 自带兼容 SS 的属性，这意味着如果你的服务器搭建的是 SS 环境，你可以使用 V2Ray 的客户端连接 SS 服务器。

### 服务器端配置

#### 安装

先说服务器的系统选择，之前使用 SS 时我一般选择 CentOS，原因是我使用的一键安装脚本的作者是使用的这个发行版的系统，我在这个发行版上使用一键安装脚本不会出现任何问题。

而 V2Ray 官方建议使用 Debain or Ubuntu。这并不是在说其他发行版无法使用这个一键安装脚本，如果你有能力解决其它系统上可能会出现的问题，你使用任何系统都没有问题，但是如果你无法做到，还是建议使用官方推荐的系统。

这里我使用的是 Debian 8.x 的发行版。

V2Ray 本体的下载可以选择从官方 repo 拉取最新的预编译压缩包，也可以使用官方提供的一键安装脚本。建议使用后者，这里也仅以后者作为演示。

> 官方也提供预编译的 Docker image 文件，理论上你可以不用关注兼容性，拉取最新的 image 文件就可以使用完全的 V2Ray 功能，如果你熟悉 Docker 可以前往 [v2ray/official](https://hub.docker.com/r/v2ray/official/) 拉取最新的稳定版本。本文暂时不关注 Docker 版本的使用。

运行下面的命令下载一键安装脚本并且运行。

```shell
$ bash <(curl -L -s https://install.direct/go.sh)
```

安装结束后，程序本体会被安装在下面的路径。

```shell
/usr/bin/v2ray/
```

配置文件在下面的路径。

```shell
/etc/v2ray/config.json
```

> 一键安装脚本会自动检测 V2Ray 的安装情况，如果没有安装则自动安装最新的稳定版程序，如果已经安装则自动更新到最新的稳定版本。

> 据官方提示：目前自动运行脚本只支持带有 Systemd 的系统，以及 Debian / Ubuntu 全系列。

> 脚本运行完成后，你需要：
> - 编辑 /etc/v2ray/config.json 文件来配置你需要的代理方式；
> - 运行 service v2ray start 来启动 V2Ray 进程；
> - 之后可以使用 service v2ray start|stop|status|reload|restart|force-reload 控制 V2Ray 的运行。

#### 配置文件

V2Ray 的配置文件处于方便阅读的考虑默认使用 JSON with comment 格式。JSON 源于 JavaScript 的对象标记，后来逐渐成为前后端通信数据格式的首选，本文默认你已经了解什么是 JSON，如果你并不了解，请自行百度。

V2Ray 的配置文件格式大致如下。服务器端和客户端的配置格式是一致的，区别在于入口和出口的配置内容不同。

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

## [备选方案] 搭建 SS 网络环境

### 服务器端的安装和配置

#### SS 环境服务器端一键安装

下面这个 repo 有对各个语言版本的安装 guide。如果打算研究，选自己看得懂的语言的版本，如果不，根据系统随意选就行。

https://github.com/teddysun/shadowsocks_install

我一般选择 `shadowsocks-all.sh`。以此为例，运行下面命令安装。

```shell
$ wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
$ chmod +x shadowsocks-all.sh
$ ./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

安装途中，端口、密码、加密方式需要选择，其他如果不清楚建议直接默认就行。加密方式我一般选择 `rc4-md5`。

如果需要卸载，运行下面的命令。并且要卸载多个版本，需要多次运行命令。

```shell
$ ./shadowsocks-all.sh uninstall
```

安装完成后 SS 默认运行中。可以使用下面的命令更改或者查看 SS 的运行状态。

```shell
$ /etc/init.d/your-shadowsocks-version start | stop | restart | status
```

`your-shadowsocks-version` 替换为你安装的版本，一般是下面几个。我一般选择 libev 版本，因为轻量。

- shadowsocks-python
- shadowsocks-r
- shadowsocks-go
- shadowsocks-libev

SS 的服务器端安装就完成了，这时需要记录几个信息，这些信息在安装完 SS 之后会输出到控制台，并且你可以通过安装 log 重新找到它们。

- Server IP
- Server Port
- Password
- Encryption Method

#### kcptun 插件安装

kcptun 用来加速 SS，且双重加密更加安全、难以被侦测。个人使用感受是加速效果很明显。下面这个 repo 是一个一键安装脚本和 guide。

https://github.com/kuoruan/shell-scripts

这里摘抄几个命令。

安装时。

```shell
$ wget --no-check-certificate https://github.com/kuoruan/shell-scripts/raw/master/kcptun/kcptun.sh
$ chmod +x kcptun.sh
$ sh kcptun.sh
```

安装途中全程有中文提示，解释的比较仔细应该没什么需要注意的。中间需要输入加速的地址，填入 SS 的端口就行。

随时查看帮助信息，全中文。

```shell
$ sh kcptun.sh help
```

安装完成后控制行会打出一串 json 配置，这个保存下来备用。服务器端的 kcptun 安装就结束了。

### 客户端的安装和配置

#### SS 客户端配置

根据系统下载客户端，我常用 macOS 和 windows 10 系统，点下面链接下载最新客户端。

macOS 10.11+: https://github.com/shadowsocks/ShadowsocksX-NG/releases

windows: https://github.com/shadowsocks/shadowsocks-windows/releases

启动后填入服务器端生成的配置即可。建议点上开机启动。

一般到这里配置已经完成，但是 SS 直连干扰比较大，速度一般不会太理想，需要配合插件提速。

锐速或者 BBR 相对配置较简单，但是个人使用感觉效果不佳，建议使用 kcptun，缺点是配置稍微繁琐一点。

#### kcptun 客户端配置

kcptun 客户端可以从下面链接下载。

https://github.com/xtaci/kcptun/releases

根据系统下载相应文件。kcptun 我习惯使用命令行，如果需要图形化可以自行上谷歌百度一下。

Macbook 送修了，下面以 Windows 10 为例。

将服务器端生成的 json 配置拷贝到本地文件，这里我复制到了 `config.json` 里。

为了方便启动，建议写一个启动文件来后台运行 kcptun。下面是一个 `runKcptun.vbs` 的例子。

注意将配置文件和 exe 文件放在同一个目录，`your-path-to-kcptun` 替换成你的路径。

```vbs
Set ws = CreateObject("Wscript.Shell")
ws.run "/your-path-to-kcptun/client_windows_amd64.exe -c config.json", 0
```

将这个文件创建快捷方式，并丢到 windows 的 startup 目录里使其开机自动启动。

> windows 10 下 startup 目录快速打开方式：win + r 打开运行窗口，输入 `shell:startup` 回车。

kcptun 客户端程序运行后还需要 SS 配置配合。

在 SS 中添加一个服务器配置，和之前配置信息一致，但是服务器地址改为 `127.0.0.1`，且端口号改为 kcptun 配置文件里 `localaddr` 字段的值。

> kcptun 仅代理流量，根本上 SS 只是走了 kcptun 的代理去连接远程服务器，虽然将配置中的服务器和端口号改成了 kcptun 的本地代理，但是密码和加密方式还是需要填入 SS 原本的设置。

到此网络环境的设置就完成了。
