# profile 和 bashrc 的区别

> 发布日期：2022-07-03 21:45:01

## 什么是 shell

#### shell 简介

shell 是系统的用户界面，提供了用户与内核进行交互操作的一种接口。它接收用户输入的命令并把它送入内核去执行。

shell 是一个命令解释器，它解释由用户输入的命令并且把它们送到内核。不仅如此，shell 有自己的编程语言用于对命令的编辑，它允许用户编写由 shell 命令组成的程序。shell 编程语言具有普通编程语言的很多特点，比如它也有循环结构和分支控制结构等，用这种编程语言编写的 shell 程序与其他应用程序具有同样的效果。

每个 Linux 系统的用户可以拥有他自己的用户界面或 shell，用以满足他们自己专门的 shell 需要。

#### 不同版本

同 Linux 本身一样，Shell 也有多种不同的版本。主要有下列版本的 Shell：　

- (**sh**)Bourne Shell：是贝尔实验室开发的。
- (**bash**)BASH：是 GNU 的 Bourne Again Shell，是 GNU 操作系统上默认的 shell。
- (**ksh**)Korn Shell：是对 Bourne Shell 的发展，在大部分内容上与 Bourne Shell 兼容。
- (**csh**)C Shell：是 SUN 公司 Shell 的 BSD 版本。
- (**zsh**)Z Shell：The last shell you’ll ever need! Z 是最后一个字母，也就是终极 shell。它集成了bash、ksh 的重要特性，同时又增加了自己独有的特性。

不同的 Shell 语言的语法有所不同，所以不能交换使用。每种 Shell 都有其特色之处，基本上，掌握其中任何一种就足够了。

Bourne Again Shell，由于易用和免费，Bash 在日常工作中被广泛使用。同时，Bash 也是大多数 Linux 系统默认的 Shell。

在一般情况下，人们并不区分 Bourne Shell 和 Bourne Again Shell，所以，在下面的文字中，我们可以看到 `#!/bin/sh`，它同样也可以改为 `#!/bin/bash`。

由于这层原因在 Debian 中使用 .profile 文件代替 .bash_profile 文件以兼容不同的 Shell。

.profile(由 Bourne Shell 和 Korn Shell 使用)和 .login(由 C Shell 使用)两个文件是 .bash_profile 的同义词，目的是为了兼容其它 Shell。

## 相关概念

#### 交互式 shell 和非交互式 shell

交互式模式就是 shell 等待你的输入，并且执行你提交的命令。这种模式被称作交互式是因为 shell 与用户进行交互。这种模式也是大多数用户非常熟悉的：登录、执行一些命令、签退。当你签退后，shell 也终止了。

shell 也可以运行在另外一种模式：非交互式模式。在这种模式下，shell 不与你进行交互，而是读取存放在文件中的命令，并且执行它们。当它读到文件的结尾，shell 也就终止了。

#### login shell 和 non-login shell

login shell：进入 bash 时需要完整的登陆流程的，就称为 login shell。

non-login shell：取得 bash 接口的方法不需要重复登陆的举动。比如我们登陆一个有可视化界面的 Linux 服务器后，右键启动终端 Terminal，此时这个终端接口是不需要输入账号与密码，那么这个启动的 shell 输入窗口就是 non-login shell。

## 查看和修改系统默认 shell

#### 查看系统所有的 shell

```bash
# 命令：
cat /etc/shells

# 输出：
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

#### 查看当前窗口使用的shell版本

```bash
# 命令：
echo $SHELL

# 输出：
>>>/bin/bash
```

#### 查看系统用户默认shell

可以看出 root 用户用的是 sh，其他用户用的是 bash

```bash
# 命令：
cat /etc/passwd | grep sh

# 输出：
root:*:0:0:System Administrator:/var/root:/bin/sh
_sshd:*:75:75:sshd Privilege separation:/var/empty:/usr/bin/false
_update_sharing:*:95:-2:Update Sharing:/var/empty:/usr/bin/false
_mbsetupuser:*:248:248:Setup User:/var/setup:/bin/bash
```

#### 修改系统默认 shell为 bash

```bash
chsh -s /bin/bash
```

## 系统环境变量设定文件分类

#### 按照是否作用于 login shell 分类

我们可以将系统的环境变量设定文件分为 profile 和 *rc 两大类：

- profile 一般用于 login shell
- *rc 用于 non-login shell

#### 按照作用范围优先级分类

一般在 /etc 目录下会有下面几个文件，作用范围是全局。

/etc/profile

> 此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行，并从 /etc/profile.d 目录的配置文件中搜集 shell 的设置。
>
> 如果你有对 /etc/profile 有修改的话必须得重启你的系统才会生效，此修改对每个用户都生效。

/etc/bashrc

> 为每一个运行 bash shell 的用户执行此文件，当 bash shell 被打开时，该文件被读取，每次用户打开一个终端时，即执行此文件。
>
> 如果你想对所有的使用 bash 的用户修改某个配置并在以后打开的 bash 都生效的话可以修改这个文件，修改这个文件不用重启，重新打开一个 bash 即可生效。
>
> 另外在用户家目录下会有一些当前用户的配置文件，这些文件的作用范围则是用户级别。

~/.profile

> 每个用户都可使用该文件输入专用于自己使用的 shell 信息，当用户登录时，该文件仅仅执行一次！默认情况下，他设置一些环境变量，执行用户的 .bashrc 文件。
>
> 此文件类似于 /etc/profile，也是需要需要重启才会生效，/etc/profile 对所有用户生效，~/.bash_profile 只对当前用户生效。

~/.bash_profile

>  在 Debian 中使用 .profile 文件代替 .bash_profile 文件以兼容不同的 shell。
>
> .profile(由 Bourne Shell 和 Korn Shell 使用)和 .login(由 C Shell 使用)两个文件是 .bash_profile的同义词，目的是为了兼容其它 Shell。

~/.bash_login

>  同上

~/.bash_logout

>  .bash_logout 在退出 shell 时被读取。所以我们可把一些清理工作的命令放到这文件中。

~/.bashrc

>  该文件包含专用于你的 bash shell 的 bash 信息，当登录时以及每次打开新的 shell 时，该文件被读取。（每个用户都有一个 .bashrc 文件，在用户目录下）
>
> 此文件类似于 /etc/bashrc，不需要重启生效，重新打开一个 bash 即可生效，/etc/bashrc 对所有用户新打开的 bash 都生效，但 ~/.bashrc 只对当前用户新打开的 bash 生效。

~/.zprofile

>  .zprofile 是 .zlogin 的替代品，.zlogin 则在 login shell 的时候读取，比如系统启动的时候会读取此文件。

~/.zlogin

> 同上

~/.zlogout

>  .zlogout 退出终端的时候读取，用于做一些清理工作。

~/.zshrc

>  该文件包含专用于你的 zsh shell 的信息，当登录时以及每次打开新的 shell 时，该文件被读取。

#### 按照是否依赖于具体的 shell 分类

另外，我们可以认为 /etc/profile 和 .profile 是系统特定的环境变量设定文件，不依赖于具体的 shell，而其他的配置文件则要依据具体的 shell，这取决于你使用哪个作为系统默认的 shell。

- .bash_profile .bash_login .bashrc 依赖于 bash
- .zprofile .zshrc 依赖于 zsh

## 不同 shell 的执行优先级

#### bash 的配置文件执行优先级

一般情况下，Linux 的默认 shell 是 bash，这也是 Linux 上最广泛使用的 shell。其执行优先级如下：

![](https://s1.ax1x.com/2022/07/03/jGZKdf.png)

当系统的默认 shell 是 bash 的时候，对于 non-login shell，bash 会默认执行 .bashrc 文件。

对于 login shell，bash 会默认执行 .profile 文件，但是当系统有 .bash_profile 或 .bash_login 存在的时候则不会执行 .profile。

**在刚登录 Linux 时：**

首先启动 /etc/profile 文件；

然后再启动用户目录下的 ~/.bash_profile、 ~/.bash_login 或 ~/.profile 文件中的其中一个，执行的顺序为：~/.bash_profile、 ~/.bash_login、 ~/.profile。

**开始执行用户的 bash 设置：**

如果 ~/.bash_profile 文件存在的话，一般会以下面的方式执行用户的 ~/.bashrc 文件。

在 ~/.bash_profile 文件中一般会有下面的代码：

```bash
# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
    . "$HOME/.bashrc"
    fi
fi
```

同样 ~/.bashrc 中，一般还会在文件的前面有以下代码，来执行 /etc/bashrc

```bash
if [ -f /etc/bashrc ] ; then
　. /etc/bashrc
fi
```



~/.bash_profile 是交互式、login 方式进入 bash 运行的。~/.bashrc 是交互式 non-login 方式进入 bash 运行的。通常二者设置大致相同，所以通常前者会调用后者。

上面这三个文件是 bash shell 的用户环境配置文件，位于用户的主目录下。其中 .bash_profile 是最重要的一个配置文件，它在用户每次登录系统时被读取，里面的所有命令都会被 bash 执行。

.bashrc 文件会在 bash shell 调用另一个 bash shell 时读取，也就是在 shell 中再键入 bash 命令启动一个新 shell 时就会去读该文件。这样可有效分离登录和子 shell 所需的环境。但一般来说都会在 .bash_profile 里调用 .bashrc 脚本以便统一配置用户环境。

#### zsh 的配置文件查找优先级

mac 系统下，我们会安装 zsh 来代替系统默认的 bash，zsh 在用户体验方面有着更优秀的特性，比如色彩高亮，命令提示，智能补全，快速跳转等。

当系统的默认 shell 是 zsh 的时候：

- 首先当系统启动的时候会执行 /etc/.profile 全局的配置文件
- 之后会在用户级别执行 .zprofile，.zshrc 两个文件

当我们安装了 zsh 之后，一般会安装 [Oh my zsh](https://ohmyz.sh/) 这个扩展工具集，来丰富我们的日常操作。另外，可以参考下面的文章配置比较流行的 agnoster 主题。

- https://blog.csdn.net/weixin_44294408/article/details/124742002

- https://blog.csdn.net/weixin_31983133/article/details/112610463

#### 执行优先级场景

1. 图形模式登录时，顺序读取：/etc/profile 和 ~/.profile

2. 图形模式登录后，打开终端时，顺序读取：/etc/.bashrc 和 ~/.bashrc

3. 文本模式登录时，顺序读取：/etc/.bashrc，/etc/profile 和 ~/.bash_profile

4. 从其它用户 su 到该用户，则分两种情况：

   （1）如果带 `-l` 参数（或 `-` 参数，`-login` 参数），如：`su -l username`，则 bash 是 login 的，它将顺序读取以下配置文件：/etc/.bashrc，/etc/profile 和 ~/.bash_profile。

   （2）如果没有带 `-l` 参数，则 bash 是 non-login 的，它将顺序读取：/etc/.bashrc 和 ~/.bashrc

5. 注销时，或退出 su 登录的用户，如果是 longin 方式，那么 bash 会读取：~/.bash_logout

6. 执行自定义的 shell 文件时，若使用 `bash -l a.sh` 的方式，则 bash 会读取行：/etc/profile 和~/.bash_profile，若使用其它方式，如 `bash a.sh`，`./a.sh`，`sh a.sh`（这个不属于 bash shell），则不会读取上面的任何文件。

## 参考资料

[1] profile和bashrc的区别 https://www.cnblogs.com/Zzbj/p/15419982.html  
[2] bashrc与profile的区别 https://blog.csdn.net/heybeaman/article/details/87289405  
[3] Linux下profile和bashrc四种的区别 https://blog.csdn.net/ishulei/article/details/108237502  
[4] Linux中profile、bashrc、~/.bash_profile、~/.bashrc、~/.bash_profile之间的区别和联系以及执行顺序 https://kernel.blog.csdn.net/article/details/45064705
