# network-configurator

快速恢复科学网络配置和 GitHub 的访问 keys。

## 进展

> 2019/6/2 近期杂事繁多，配置器暂时搁浅。接下来准备配置家庭 NAS + 软路由一体机，届时还会走一遍网络配置流程，并且对已经安装了网络代理的机器清除设置，到时候再完善配置流程，考虑如何脚本化。目前为止的内容足够应对紧急恢复网络配置。

> 2019/6/3 突发情况，救命配置。

```json
      "streamSettings": {
        "network": "mkcp",
        "kcpSettings": {
          "uplinkCapacity": 5,
          "downlinkCapacity": 100,
          "congestion": true,
          "header": {
            "type": "none"
          }
        }
      }
```

## 简介

近期经历了 win10 换机器，MacBook 送修等事情，硬盘前前后后格式化了两三次。并且每次格式化之后，都要花 1、2 天时间来将系统恢复到日常使用的状态。

时间基本都花在了翻查资料和安装软件上，由于网络配置基本上配置完成一次之后很长一段时间都不需要改动，所以这方面只记得一个流程，很多细节都需要重新翻查，所以这次计划将翻查过的各种 guide 整合起来，简化、节省翻查资料的时间，并且如果可行的话，制作成一个一键配置脚本，来最大程度简化步骤。

## 内容

> TODO

- VPS 相关配置

VPS 购买，服务器创建，安装系统。

- （备选方案）S&S 相关配置

不建议配置，作为救急方案可以尝试。自行查看 guide4backup(s&s).md 文件。

> 曾全军覆没。

- v2r 相关配置

v2r 从诞生以来目标就是实现一个功能化的网络加密数据传输模块。

- [GitHub SSH & GPG keys](#github-SSh--gpg-keys)

快速恢复 GitHub 访问权限。

# 工具选择

仅个人观点。

## v2r

v2r 由于诞生缘由导致其特性上天生优于 S&S，也必然是未来科学上网的趋势。另一方面其配置相比 S&S 来说稍微繁琐一点。

# GitHub SSH & GPG keys

恢复与 GitHub 的 git 读写权限。

首先我们需要安装 Git。Mac 下安装 Xcode 会自动安装 git，win10 下则需要手动到 git 官网去下载安装包。

安装完成后，运行下面的命令来对 git 做一些基础配置。

```shell
$ git config --global user.name your_name
$ git config --global user.email your_email@example.com
```

> 在 Mac 环境下需要使用 `sudo` 前缀。

## SSH 访问 keys

Official guide: https://help.github.com/en/articles/connecting-to-github-with-SSH

### 生成 SSH key

以 macOS 为例。

首先生成一个 SSH key。在命令行输入下面的命令。

```shell
$ SSH-keygen -t rsa -b 4096 -C "你的邮箱@example.com"
```

你会看到下面的输出

```
> Generating public/private rsa key pair.
```

接下来你会看到下面的询问，一般情况按回车默认。

```
> Enter a file in which to save the key (/Users/you/.SSH/id_rsa): [PreS&S enter]
```

最后你会看到下面的提示，要求你设定一个密码。

```
> Enter paS&Sphrase (empty for no paS&Sphrase): [Type a paS&Sphrase]
> Enter same paS&Sphrase again: [Type paS&Sphrase again]
```

SSH key 生成就完成了。

### 添加 SSH key 到 SSH-agent 中管理

接下来将这个 key 添加到 SSH-agent。运行下面的命令启动 SSH-agent。

```shell
$ eval "$(SSH-agent -s)"
```

macOS Sierra 10.12.2 及以上需要将下面的信息保存到 `~/.SSH/config` 文件中，来保持 SSH-agent 自动加载 keys 并将密码储存在钥匙串中。

```shell
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.SSH/id_rsa
```

输入下面的命令将 key 添加到 SSH-agent，可能会要求输入密码。

```shell
$ SSH-add -K ~/.SSH/id_rsa
```

添加完成。

### 添加 SSH key 到 GitHub

输入下面的命令，将公钥拷贝到剪贴板。

```shell
$ pbcopy < ~/.SSH/id_rsa.pub
```

打开 `your account` -> `settings` -> `SSH and GPG keys` -> `New SSH key`。

将剪贴板中到公钥粘贴到输入框，命名并保存。

添加完成。

## GPG Signature keys

Official guide: https://help.github.com/en/articles/managing-commit-signature-verification

### 生成 GPG key

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

你会看到一个提示，要求选择加密模式，直接回车选择默认的 `RSA and RSA`。

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

### 添加 GPG key 到 GitHub

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
S&Sb   4096R/42B317FD4BA89E7A 2016-03-10
```

使用下面的命令输出 `ASCII armor format` 的 GPG key ID，这里以 `3AA5C34371567BD2` 为例，可以看上面的信息确定 GPG key ID 的位置。

```shell
$ gpg --armor --export 3AA5C34371567BD2
```

拷贝以 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 开始并以 `-----END PGP PUBLIC KEY BLOCK-----` 结束的 GPG key ID 内容。

登陆 GitHub，打开 `your account` -> `settings` -> `SSH and GPG keys` -> `New GPG key`。

将剪贴板的内容粘贴到输入框内并保存，GPG key 到添加就完成了。

### 告诉 Git 使用 GPG

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
