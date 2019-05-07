# network-configurator

快速恢复科学网络配置和 GitHub 的访问 keys。

### 简介

近期经历了 win10 换机器，MacBook 送修等事情，硬盘前前后后格式化了两三次。并且每次格式化之后，都要花 1、2 天时间来将系统恢复到日常使用的状态。

时间基本都花在了翻查资料和安装软件上，由于网络配置基本上配置完成一次之后很长一段时间都不需要改动，所以这方面只记得一个流程，很多细节都需要重新翻查，所以这次计划将翻查过的各种 guide 整合起来，简化、节省翻查资料的时间，并且如果可行的话，制作成一个一键配置脚本，来最大程度简化步骤。

### 内容

> TODO

- VPS 相关配置

VPS 购买，服务器创建，安装系统。

- [（备选方案）SS 相关配置](#备选方案ss-相关配置)

SS 的诞生之初只是作者的一个自用工具，后来作者觉得不错，开源出来，功能一步步完善。安装简单，配置方便。

- V2Ray 相关配置

V2Ray 从诞生以来目标就是实现一个功能化的网络加密数据传输模块。

- [GitHub SSH & GPG keys](#GitHub-SSH-&-GPG-keys)

快速恢复 GitHub 访问权限。

## 工具选择

仅个人观点。

### SS

用了近两年 SS，中间多次更换 VPS 重新配置，所以较为熟悉流程，依旧作为一个备选方案记录。

### V2Ray

V2Ray 由于诞生缘由导致其特性上天生优于 SS，也必然是未来科学上网的趋势。另一方面其配置相比 SS 来说稍微繁琐一点。

## （备选方案）SS 相关配置

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

## GitHub SSH & GPG keys

恢复与 GitHub 的 git 读写权限。

首先我们需要安装 Git。Mac 下安装 Xcode 会自动安装 git，win10 下则需要手动到 git 官网去下载安装包。

安装完成后，运行下面的命令来对 git 做一些基础配置。

```shell
$ git config --global user.name
$ git config --global user.email
```

> 在 Mac 环境下需要使用 `sudo` 前缀。

### SSH 访问 keys

Official guide: https://help.github.com/en/articles/connecting-to-github-with-ssh

#### 生成 SSH key

以 macOS 为例。

首先生成一个 SSH key。在命令行输入下面的命令。

```shell
$ ssh-keygen -t rsa -b 4096 -C "你的邮箱@example.com"
```

你会看到下面的输出

```
> Generating public/private rsa key pair.
```

接下来你会看到下面的询问，一般情况按回车默认。

```
> Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
```

最后你会看到下面的提示，要求你设定一个密码。

```
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```

SSH key 生成就完成了。

#### 添加 SSH key 到 ssh-agent 中管理

接下来将这个 key 添加到 ssh-agent。运行下面的命令启动 ssh-agent。

```shell
$ eval "$(ssh-agent -s)"
```

macOS Sierra 10.12.2 及以上需要将下面的信息保存到 `~/.ssh/config` 文件中，来保持 ssh-agent 自动加载 keys 并将密码储存在钥匙串中。

```shell
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa
```

输入下面的命令将 key 添加到 ssh-agent，可能会要求输入密码。

```shell
$ ssh-add -K ~/.ssh/id_rsa
```

添加完成。

#### 添加 SSH key 到 GitHub

输入下面的命令，将公钥拷贝到剪贴板。

```shell
$ pbcopy < ~/.ssh/id_rsa.pub
```

打开 `your account` -> `settings` -> `SSH and GPG keys` -> `New SSH key`。

将剪贴板中到公钥粘贴到输入框，命名并保存。

添加完成。

### GPG Signature keys

Official guide: https://help.github.com/en/articles/managing-commit-signature-verification

#### 生成 GPG key

GPG 工具并非系统自带，所以首先确认系统中有没有 GPG 工具。输入下面的命令，如果报错了，说明系统中不存在 GPG 工具，这时需要去 [GPG 官网下载](https://www.gnupg.org/download/)。

> Mac 下可以使用这个地址下载安装器：https://sourceforge.net/p/gpgosx/docu/Download/

> 某些安装器要求使用 `gpg2` 命令。没关系，我们先创建一个软连接 `sudo ln -s /usr/local/bin/gpg2 /usr/local/bin/gpg`，这样输入 `gpg` 命令一样有效。

```shell
$ gpg
```

命令行不再报错，说明系统中的 gpg 工具已经正常工作。接下来我们创建新的 GPG key。

运行下面的命令生成新的 key。这里的命令可能会因为版本而稍有不同，如果遇到问题用 `man gpg` 命令查看说明。

```shell
$ gpg --full-generate-key
```

你会看到一个提示，要求选择加密模式，直接回车选择默认的 `RSA and RSA`。、

```
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection?
```

接下来设定 keysize，根据 GitHub 的推荐我们将其设成 `4096`。

```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
```

然后会提示设定 key 的有效期，按需选择，或者直接回车默认 key 不会过期。

```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
```

接着会要求你确认设置是否正确，然后需要你输入名称、邮件和备注，并设定密码。做完这些，稍等片刻 key 的生成就完成了。

#### 添加 GPG key 到 GitHub

现在使用下面的命令查询存在的 GPG key。

```shell
$ gpg --list-secret-keys --keyid-format LONG
```

你会看到类似下面的输出。

```
/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot
ssb   4096R/42B317FD4BA89E7A 2016-03-10
```

使用下面的命令输出 `ASCII armor format` 的 GPG key ID，这里以 `3AA5C34371567BD2` 为例，可以看上面的信息确定 GPG key ID 的位置。

```shell
$ gpg --armor --export 3AA5C34371567BD2
```

拷贝以 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 开始并以 `-----END PGP PUBLIC KEY BLOCK-----` 结束的 GPG key ID 内容。

登陆 GitHub，打开 `your account` -> `settings` -> `SSH and GPG keys` -> `New GPG key`。

将剪贴板的内容粘贴到输入框内并保存，GPG key 到添加就完成了。

#### 告诉 Git 使用 GPG

输入下面的命令配置 git 使用的 GPG key ID。这里依旧以 `3AA5C34371567BD2` 为例，你需要将之替换为你自己的 ID。

```shell
$ git config --global user.signingkey 3AA5C34371567BD2
```

并且开启自动使用 GPG 验证提交。

```shell
$ git config --global commit.gpgsign true
```

在某些情况下你可能需要下面的命令，将 GPG key 添加到你的 bash profile 中。

```shell
$ test -r ~/.bash_profile && echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
$ echo 'export GPG_TTY=$(tty)' >> ~/.profile
```

到此 GPG 的设置就完成了，每次当你执行提交的时候，会弹出 GPG 密码框要求你验证密码。并且你的提交在 GitHub 上会出现 `Verified` 标记，表示这个提交来自可信源。
