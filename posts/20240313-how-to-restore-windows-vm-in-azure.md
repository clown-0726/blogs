# 如何在 Azure 上备份 windows 虚拟机并恢复

> 发布日期：2024-03-13 19:20:11

[![pFcg1iV.jpg](https://s21.ax1x.com/2024/03/13/pFcg1iV.jpg)](https://imgse.com/i/pFcg1iV)

## 起因

Azure 在 windows 虚拟机备份/还原上和通常的虚拟机备份有所区别，一般的虚拟机备份在控制台的上的操作通常是选择将目标虚拟机备份成镜像，还原的时候选择备份好的镜像即可。

但是对于 windows 虚拟机的备份/还原需要借助其磁盘进行操作。下面是操作步骤，仅供参考。

## 操作步骤

前提已经开通了一个正在运行的 windows 服务器。

首先进入 Azure 的 Disk 服务，找到将要做备份的目标虚拟机所正在使用的 Disk。

[![pFcgzWT.png](https://s21.ax1x.com/2024/03/13/pFcgzWT.png)](https://imgse.com/i/pFcgzWT)

点进进入选中 Disk 的详细页面，选择“Create snapshot”

[![pFcgxYV.png](https://s21.ax1x.com/2024/03/13/pFcgxYV.png)](https://imgse.com/i/pFcgxYV)

进入创建页面后，填写 snapshot 的名称，并选择创建一个完整的版本。

[![pFc2efK.png](https://s21.ax1x.com/2024/03/13/pFc2efK.png)](https://imgse.com/i/pFc2efK)

进入 Azure 的 Snapshots 服务，找到刚建立的 Snapshot。

[![pFc2B0s.png](https://s21.ax1x.com/2024/03/13/pFc2B0s.png)](https://imgse.com/i/pFc2B0s)

然后进入这个这个 Snapshot 的详细页面，选择“Create disk”创建一个新的 disk。

[![pFc20mj.png](https://s21.ax1x.com/2024/03/13/pFc20mj.png)](https://imgse.com/i/pFc20mj)

进入创建页面后，填写 disk 的名称，注意，由于这个是 windows 的 disk，各种属性都是默认 windows 的配置，是不可修改的。

[![pFc2dXQ.png](https://s21.ax1x.com/2024/03/13/pFc2dXQ.png)](https://imgse.com/i/pFc2dXQ)

进入刚创建好的 disk 的详细页面并选择“Create VM”创建一个新的虚拟机。

[![pFcfOjU.png](https://s21.ax1x.com/2024/03/13/pFcfOjU.png)](https://imgse.com/i/pFcfOjU)

进入虚拟机创建页面后，填写虚拟机的名称和其他配置，之后进行创建即可。

[![pFc24B9.png](https://s21.ax1x.com/2024/03/13/pFc24B9.png)](https://imgse.com/i/pFc24B9)

通过上述步骤，将一个正在运行的 windows 虚拟机进行了备份还原操作。

## 参考文档：

N/A
