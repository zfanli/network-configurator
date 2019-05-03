# netwrok-configurator

网络配置 guide or tool。

由于暂时没折腾软路由，每台机器的网络配置都是独立的，并且每次更换环境都要做一次冗长的配置，重复阅读各种资料...所以这里单独对各种网络工具的配置方法做一个记录，如果可以的话做成一个一键安装工具。

只针对科学上网的网络配置。

## 工具选择

仅个人观点。

### SS

用了近两年 SS，中间多次更换 VPS 重新配置，所以较为熟悉流程，目前由于需要快速恢复网络配置，所以依然采用 SS 方案。之后计划转移到下一个方案上。

### V2Ray

V2Ray 由于诞生缘由导致其特性上天生优于 SS，也必然是未来科学上网的趋势。但是另一方面其配置复杂程度是远超过 SS 的，这是接下来准备使用的方案。

## 基于 SS 快速搭建科学网络环境

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
