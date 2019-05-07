## GitHub SSH & GPG keys

恢复与 GitHub 的 git 读写权限。

首先我们需要安装 Git。Mac 下安装 Xcode 会自动安装 git，win10 下则需要手动到 git 官网去下载安装包。

安装完成后，运行下面的命令来对 git 做一些基础配置。

```shell
$ git config --global user.name your_name
$ git config --global user.email your_email@example.com
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
