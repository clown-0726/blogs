# AWS 配置 Ubuntu 可视化界面

> 发布日期：2021-08-09 17:18:07

仗义每多屠狗辈，负心多是读书人。在年少的认知能力下设立的本心一定在将来的高认知下适用吗？

<!--more-->

![呼伦贝尔-月亮](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210809-aws-config-ubuntu-gui/moon-in-hulubbeier.jpeg)

最近在写爬虫的时候，有一些网站当输入验证码之后可以进行长时间的爬取。但是如果不进行初次验证码验证，基本上是不能进行爬取的。本地测试是没有任何问题的，但是如果程序需要长期运行的话，还是要放到服务器上。因此配置一台带有 GUI 图形界面的服务器还是很有必要的，况且服务器的网速也能有很好的保证。

我们在购买 linux 服务的时候，大部分的服务器都是不带有桌面的，这里不讨论如何在一个无桌面的 linux 上装上比如 GNOME 桌面环境，而是申请已经具备桌面程序的 linux 服务器。由于我要爬的网站在国外，因此 aws 服务器也就成为了我的首选。

如果小伙伴们的爬虫符合下面的条件，推荐继续往下读：
- 需要一台带 GUI 的 liunx 服务器
- 使用 aws 云服务（EC2）
- 要爬的网站需要进行人工介入进行验证，并且验证后可以爬取很长时间
- 要爬的网站在国外（非必要）

## 申请 aws 带有 GUI 的 linux 服务器
第一步，先申请一个带有 GUI 的服务器，如下图所示，我们选择 "Noricum Cloud Solutions" 的产品，这个应该是 aws 上 GUI 做的最好的产品了。

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210809-aws-config-ubuntu-gui/choose_gui_ec2.png)

第二步，选择实例类型的时候可以根据我们的需求选择一个合适的配置。我这里选择的是 `t2large`。

第三步，我们注意到接下来会有 `Configure Instance`，`Add storage`，`Add Tags` 这几个配置，都是一些常规的配置，根据实际需要进行配置即可。

第四步，对于 `security group` 的配置，我们需要开放几个端口，SSH 的 22 端口，TCP 的 5901 端口，RDP 的 3389 端口，如下所示。
![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210809-aws-config-ubuntu-gui/sg-config.png)

最后一步我们选择一个自己的 `key pair` 并且下载到本地，之后我们通过这个 `key pair` 进行启动就 OKAY 了。

## 连接带有 GUI 的服务器

```
Usage Instructions
Find detailed documentation on: https://noricum.cloud/docs/ubuntu-gui.html

1) The standard user name is 'ubuntu' and the password for RDP and VNC is your Instance ID (e.g. 'i-0ed22c98429479e1c')

2.a) Open RDP client on your local machine and connect to the public ip-address of your instance on port 3389. See step 1 for username and password.

2.b) Open VNC client on your local machine and connect to the public ip-address of your instance on port 5901. See step 1 for the required password.
```

上面是官方文档的使用指导，从上面看出这台服务器已经给我们提供了 VNC 和 RDP 两种连接方式，下面分解进行介绍。

#### 使用 RDP 进行连接
如果你是在 mac 上，推荐使用 Parallels Client 进行连接，免费，速度也可以接受。参考下面配置

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210809-aws-config-ubuntu-gui/parallels-client-rdp-connection.png)


#### 使用 VNC 进行连接
一种方式是使用 realvnc 进行连接，直接去 realvnc 官网去下载就可以了。

如果你的电脑是 mac，可以直接用 Spotlight 搜索 Screen Sharing 然后输入地址就可以了。

连接地址为：18.xxx.xxx.80:5901，注意端口号一定要指定

#### 使用 teamviewer 进行连接
使用 teamviewer 是连接速度最快的一种方式，因为 teamviewer 对网速做了一定的优化。但这种方式也是配置稍微复杂的。

首先先要使用 RDP 或者 VNC 连接上服务器，然后在服务器上安装 teamviewer 程序。参考下面连接进行安装：https://blog.csdn.net/tap880507/article/details/85601337。

最后在本地下载一个 teamviewer 就可以进行连接了。具体 teamviewer 的使用是另一个话题，也比较简单，这里不做赘述。


## Reference
[1] Ubuntu16.04 安装Teamviewer：https://blog.csdn.net/tap880507/article/details/85601337  
[2] dpkg安装以及卸载软件：https://blog.csdn.net/u012300744/article/details/80267225  
[3] Ubuntu Desktop 20.04 LTS - GUI Gnome with VNC and RDP：https://aws.amazon.com/marketplace/pp/prodview-zbk3mpk2s6psk?ref=cns_1clkPro  
[4] Documentation for Ubuntu with GUI Desktop：https://noricum.cloud/docs/ubuntu-gui.html
