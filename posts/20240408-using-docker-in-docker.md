# [翻译] 在 CI 或测试环境中使用 Docker-in-Docker，三思而后行

> 发布日期：2024-04-08 18:01:01

原文地址：[Using Docker-in-Docker for your CI or testing environment? Think twice.](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)

Docker-in-Docker 的主要目的是帮助 Docker 本身的开发。许多人使用它来运行 CI（例如使用 Jenkins），起初这似乎很好，但他们遇到了许多“有趣”的问题，这些问题可以通过将 Docker 套接字（socket）“绑定安装”到 Jenkins 容器中来避免。

我们看看这意味着什么。如果您想要不包含详细信息的快速解决方案，只需滚动到本文底部即可。☺

更新（2020 年 7 月）：当我在 2015 年写这篇博客文章时，运行 Docker-in-Docker 的唯一方法是在 Docker 上使用 `-privileged` 标签。今天，情况大不相同。容器安全性和沙盒技术（sandboxing）有了显著的进步，例如可以通过 rootless 容器和 sysbox 等工具实现。后者允许运行 Docker-in-Docker 时不需要 `-privileged` 标签，甚至还为一些特定场景提供了优化，比如将 Kubernetes 集群的多个节点作为普通容器运行。这篇文章已经针对这一点进行了更新！

## Docker-in-Docker：好的方面

两年多前，我在 Docker 中贡献了 `-privileged` 标签，并编写了 dind 的第一个版本。目标是帮助核心团队更快地进行 Docker 开发。在 Docker-in-Docker 出现之前，典型的开发周期是：
- 方案尝试（hackity hack）
- 构建
- 停止当前运行的 Docker 的守护进程
- 运行新的 Docker 的守护进程
- 测试
- 重复

如果你想要一个漂亮的、可复制的构建（即在容器中），它就有点复杂了：
- 方案尝试（hackity hack）
- 确保 Docker 的可用版本正在运行
- 使用旧 Docker 构建新 Docker
- 停止 Docker 守护程序
- 运行新的 Docker 守护进程
- 测试
- 停止新的 Docker 守护进程
- 重复

随着 Docker-in-Docker 的出现，这被简化为：
- 方案尝试（hackity hack）
- 同时做构建 + 运行
- 重复

好多了，对吧？

## Docker-in-Docker：坏的方面

然而，与主流的观点相对的是，Docker-in-Docker 并不是 100% 由火花、小马和独角兽组成的（这里表示并不完美）。我在这里的意思是，有几个问题需要注意。

一个是关于像 AppArmor 和 SELinux 这样的 LSM（Linux 安全模块）：当启动一个容器时，“内部 Docker”可能会在尝试应用安全配置文件时与“外部 Docker”产生冲突或混淆。合并 `-privileged` 标签的原始实现是当时最难解决的问题。我的更改在 Debian 机器和 Ubuntu 测试虚拟机上有效（所有测试都会通过），但它会在 Michael Crosby 的机器上崩溃（如果我记得很清楚的话，那就是 Fedora）。我记不清问题的确切原因了，但可能是因为 Mike 是一个聪明的人，他使用 `SELINUX=enforce` 运行（我当时使用的是 AppArmor），我的更改并没有考虑 SELINUX 的配置文件。


## Docker-in-Docker：丑陋的方面

第二个问题与存储驱动器有关。当你在 Docker 中运行 Docker 时，外部 Docker 运行在普通的文件系统（EXT4、Btrfs 等）之上，但内部 Docker 运行在写时复制（copy-on-write）系统（AUFS、Btrfs、Device Mapper 等，具体取决于外部 Docker 设置使用什么）之上。这会有许多组合不起作用。例如，你不能在 AUFS 之上运行 AUFS。如果你在 Btrfs 之上运行 Btrfs，它应该首先工作，但一旦你有嵌套的子卷，删除父卷就会失败。Device Mapper 不支持命名空间，所以如果多个 Docker 实例在同一台机器上使用它，它们都可以看到（并影响）彼此的镜像和容器支持设备。没有好办法。

对于许多这些问题，都有妥协方案；例如，如果你想在内部的 Docker 中使用 AUFS，只需将 `/var/lib/docker` 提升为卷，一切就都正常了。Docker 为 Device Mapper 目标名称添加了一些基本的命名空间，这样如果同一台机器上运行了多个 Docker 调用，它们就不会互相影响了。

然而，这些设置并不简单直接，正如您在 GitHub 上的 dind 存储库上看到的那些问题一样。

## Docker-in-Docker：情况越来越糟

那么构建缓存呢？这可能也会变得相当棘手。人们经常问我，“我正在运行 Docker-in-Docker；我如何使用位于我的主机上的镜像，而不是在我内部的 Docker 中再次拉取所需镜像？”

一些喜欢冒险的人试图将 `/var/lib/docker` 从主机绑定到 Docker-in-Docker 容器中。有时他们与多个容器共享 `/var/lib/docker`。

[![pFO18Gq.jpg](https://s21.ax1x.com/2024/04/09/pFO18Gq.jpg)](https://imgse.com/i/pFO18Gq)

Docker 守护进程被明确设计为具有对 `/var/lib/docker` 的独占访问权限。其他任何东西都不应该触及、戳动或触动隐藏在那里的任何 Docker 文件。

这是为什么呢？这是从 dotCloud 时代学到的最难的一课。dotCloud 容器引擎通过让多个进程同时访问 `/var/lib/dotcloud` 来工作。像原子文件替换（而不是就地编辑）、在代码中添加自愿和强制锁定（peppering the code with advisory and mandatory locking）等聪明的技巧，以及像 SQLite 和 BDB 这样的安全系统的其他实验，只让我们走到现在；当我们重构我们的容器引擎（最终成为 Docker）时，一个重大的设计决定是将所有容器操作集中到一个守护进程中，并完成所有的无意义的并发访问。

（不要误解我的意思：完全有可能做一些漂亮、可靠、快速的事情，来支持多个进程和最先进的并发管理；但我们认为，使用 Docker 的单角色模型更简单，也更易于编写和维护。）

这意味着，如果你在多个 Docker 实例之间共享 `/var/lib/docker` 目录，你会遇到麻烦。当然，它可能会起作用，特别是在早期测试期间。“看，妈妈，我可以 `docker run ubuntu`！”但是试着做一些更复杂的事情（从两个不同的实例中提取相同的镜像……），然后看着世界燃烧。

这意味着，如果你的 CI 系统进行构建和重新构建，每次重启 Docker-in-Docker 容器时，你可能会清除其缓存。这真的不好。

## Docker-in-Docker：然后它会变得更好

你肯定听过马克·吐温那句名言的变体：“他们不知道这是不可能的，所以他们做了。”

许多人试图在安全地运行 Docker-in-Docker。几年前，我在用户名称空间和一些非常讨厌的黑客攻击方面取得了一定的成功（包括在 tmpfs 挂载上伪造的 cgroups pseudo-fs 结构，以便容器运行时就不会各种报警或报错；真是有趣），但看起来一个干净的解决方案将是一项重大努力。

这种干净的解决方案现在已经存在：它被称为 sysbox。Sysbox 是一个 OCI 运行时，可以代替 runc 使用，也可以在 runc 之外使用。它使通常需要拥有特权标志才能运行的“系统容器”，在不需要特权标志时也可以运行；并且在这些容器之间以及在这些容器与其宿主之间提供足够的隔离。

Sysbox 还提供了在容器中运行容器（containers-in-containers）的优化。具体来说，当并行运行 Docker 的多个实例时，可以用一组共享的镜像“播种”它们。这节省了大量的磁盘空间和时间，我认为这在容器中运行例如 Kubernetes 节点时会产生巨大的不同。

（当您希望部署 Kubernetes 暂存应用程序（staging app）或在其自己的集群中运行测试时，在容器中运行 Kubernete 节点对 CI/CD 特别有用，节省了在专用机器上部署完整集群基础设施的成本和时间开销。）

长话短说：如果你的用例真的绝对要求 Docker-in-Docker 时，看看 sysbox，它可能就是你所需要的。

## 基于套接字（socket）的解决方案

让我们后退一步。你真的需要 Docker-in-Docker 吗？或者，当 CI 系统本身在一个容器中时，你只是想从你的 CI 系统中运行 Docker（特别是：构建、运行，有时推送容器和映像）吗？

我敢打赌，大多数人都想要后者。您所需要的只是一个解决方案，以便像 Jenkins 这样的 CI 系统可以启动容器。

最简单的方法是将 Docker 套接字暴露给 CI 容器，方法是使用 `-v` 标志绑定安装它。

简单地说，当你启动你的 CI 容器（ Jenkins 或其他）时，不要在 Docker-in-Docker 使用一些 hacking 的手段，而是通过：

```
docker run -v /var/run/docker.sock:/var/run/docker.sock ...
```

现在，这个容器将可以访问 Docker 套接字（socket），因此拥有了启动容器的能力。不同的是，它将启动“兄弟”容器，而不是启动“子”容器。

使用 docker 官方图像（其中包含 docker 二进制文件）进行尝试：

```
docker run -v /var/run/docker.sock:/var/run/docker.sock \
           -ti docker
```

⚠️ Former versions of this post advised to bind-mount the docker binary from the host to the container. This is not reliable anymore, because the Docker Engine is no longer distributed as (almost) static libraries.

If you want to use e.g. Docker from your Jenkins CI system, you have multiple options:

installing the Docker CLI using your base image’s packaging system (i.e. if your image is based on Debian, use .deb packages),
using the Docker API.

这看起来像 Docker-in-Docker，感觉就像 Docker-in-Docker，但它不是 Docker-in-Docker：当这个容器将创建更多的容器时，这些容器将在顶级 Docker 中创建。您将不会遇到嵌套的副作用，并且构建缓存将在多个调用之间共享。

⚠️ 这篇文章的前几个版本建议将 docker 二进制文件从主机绑定到容器。这不再可靠，因为 Docker 引擎不再作为（几乎）静态库分发。

如果您想在 Jenkins CI 系统中使用 Docker，您有多种选择：
- 使用基础映像的打包系统安装 Docker CLI（即，如果镜像基于 Debian，请使用 .deb 包），
- 使用 Docker API。


## 参考文档：

[1] Using Docker-in-Docker for your CI or testing environment? Think twice. https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/
