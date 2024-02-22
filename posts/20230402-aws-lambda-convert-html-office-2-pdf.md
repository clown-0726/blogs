# aws lambda 转换 office/txt/html 为 pdf

> 发布日期：2023-04-02 22:59:00

简洁的写作需要勇气。让事物变小是一种深思熟虑的、困难的和有价值的行为。大多数书籍本应是一篇博客文章。大多数博客文章本应是一条微博。大多数微博本应不写。

[![ppfvMM4.md.png](https://s1.ax1x.com/2023/04/03/ppfvMM4.md.png)](https://imgse.com/i/ppfvMM4)

## 概述

#### aws lambda

AWS Lambda 是一项无服务器事件驱动型计算服务，该服务使您可以运行几乎任何类型的应用程序或后端服务的代码，而无需预置或管理服务器。可以从 200 多个 AWS 服务和软件即服务 (SaaS) 应用程序中触发 Lambda，且只需按您的使用量付费。

#### 为什么选择使用 aws lambda

文件格式转换是一种非常消耗内存和 CPU 的计算，并且存在很多不确定性。如果使用传统的方式去实现，比如 api + 多线程的方式运行，如果在大批量转换的时候，很多失败的任务没有进行有效的资源释放，这很容易引起系统的内存泄漏或 CPU 过载。并且如果任务过多，服务的横向扩展会变得比较复杂（实现横向扩展远比转换服务本身复杂）。

aws lambda 引入能有效的解决上述产生的问题，通过 lambda + sqs 的方式可以很方便的实现一个高可用的格式转换服务。
- aws lambda 支持 docker 的方式调用，不管运行运行环境多复杂，也能有效的进行环境打包。
- 每一个 lambda 函数的执行是独立的，失败任务不会影响到其他任务的执行。
- sqs 消息队列的方式触发 lambda，可以对任务过多时进行有效的“削峰”处理。

## 构建转换环境

由于 aws lambda 支持 docker 的方式运行，因此我们可以构建好基础的转换服务运行环境，这样只需要关注具体的业务逻辑编写即可。
- 对于 html 的转换方案，采用 wkhtmltopdf
- 对于 office(word/excel/powerpoint) 则采用开源的 libreoffice

由于我的业务环境使用 python 居多，因此这里选择的基础镜像为 `python:3.7`，这同时也是一个基于 Debain 的系统。

#### html 转换方案

wkhtmltopdf 是一个使用 Qt WebKit 引擎做渲染的，能够把 html 文档转换成 pdf 文档 或 图片(image) 的命令行工具。支持多个平台，可在 win，linux，os x 等系统下运行。

优点：
- 生成 PDF 时会自动根据 HTML 页面中 H 标签生成树形目录结构；
- 小巧方便，转换速度快；
- 跨平台，可以在 Liunx 下用，也可以在 win 使用；

下面是 Dockerfile 的关于安装 wkhtmltopdf 的命令，其实过程主要是从 wkhtmltopdf 官网下载对应的版本，只有通过 dpkg 进行安装。

```
# Install wkhtmltopdf
RUN wget -O wkhtmltox_0.12.6.1-2.bullseye_amd64.deb https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.bullseye_amd64.deb && \
		apt-get -y update && apt-get -y install xfonts-encodings xfonts-utils xfonts-base xfonts-75dpi && \
		dpkg -i wkhtmltox_0.12.6.1-2.bullseye_amd64.deb  && \
		apt-get -y autoremove && apt-get clean && \
		rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
		wkhtmltopdf --version
```

安装完之后可以通过 python 的 os 调用系统命令进行 html 文件的转换，详细的参数配置选择可以参考[官网](https://wkhtmltopdf.org/usage/wkhtmltopdf.txt)。
```
wkhtmltopdf --page-size A4 --orientation Portrait --dpi 300 --image-dpi 600 --image-quality 94 <input name> <output name>
```

#### office 转换方案

对于转换 office 到 pdf，最好的方案是调用 office 的底层服务，但这会紧紧局限在 windows 平台，如果要进行跨平台使用，LibreOffice 是一个不错的选择，它能最大程度的兼容 word/excel/powerpoint，最终转换效果也是很不错的。

LibreOffice 在 linux 的安装会比较复杂一些，同时需要安装一些相应的字体库等。其安装过程可以分为下面三步：
- 增加 LibreOffice “fresh” PPA
- 安装语言包
- 安装额外必须的包
- 安装 LibreOffice

可以参考这篇[文章](https://libre-software.net/linux/how-to-install-libreoffice-on-ubuntu-linux-mint/)获得详细的解释，下面是 Dockerfile 的关于安装 LibreOffice 的命令

```
# Install LibreOffice
RUN	apt-get -y update && apt-get -y dist-upgrade &&\
		apt-get -y install software-properties-common && \
		add-apt-repository -y ppa:libreoffice/ppa && \
		apt-get -y install locales libreoffice libreoffice-writer libreoffice-impress libreoffice-common && \
		apt-get -y install fonts-opensymbol && \
		apt-get -y install hunspell-en-us hyphen-en-us mythes-en-us libreoffice-help-en-us && \
		apt-get -y install libreoffice-l10n-de hunspell-de-de-frami hyphen-de mythes-de libreoffice-help-de && \
		apt-get -y install libreoffice-l10n-fr hunspell-fr-comprehensive hyphen-fr mythes-fr libreoffice-help-fr && \
		apt-get -y install libreoffice-l10n-es hunspell-es hyphen-es mythes-es libreoffice-help-es && \
		apt-get -y install fonts-dejavu fonts-dejavu-core fonts-dejavu-extra fonts-droid-fallback fonts-dustin fonts-f500 fonts-fanwood && \
		apt-get -y install fonts-freefont-ttf fonts-liberation fonts-lmodern fonts-lyx fonts-sil-gentium fonts-texgyre fonts-tlwg-purisa && \
		apt-get -y autoremove && apt-get clean && \
		rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
		localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 && \
		libreoffice --version
```

安装完之后可以通过 python 的 os 调用系统命令进行 html 文件的转换。

```
libreoffice --headless --convert-to pdf <input name>
```

注意，如果在 lambda 上运行的时候，一定要重新设置 HOME 路径到 `/tmp` 下，否则会因为无权限创建缓存文件而报错。

```
export HOME=/tmp && libreoffice --headless --convert-to pdf <input name>
```

#### txt 转换方案

将 txt 转换成 pdf 可以直接借用 libreoffce，命令也是一样的，如下：
```
libreoffice --headless --convert-to pdf <input name>
```

## 参考文档
[1] HTML 转 PDF 之 wkhtmltopdf 工具简介 https://www.jianshu.com/p/559c594678b6/  
[2] wkhtmltopdf https://wkhtmltopdf.org/downloads.html  
[3] Ubuntu18.04中安装wkhtmltopdf http://events.jianshu.io/p/07c8e7837c7e  
[4] How to install LibreOffice 7.4 on Linux Mint, Ubuntu, MX Linux, Debian… https://libre-software.net/linux/  how-to-install-libreoffice-on-ubuntu-linux-mint/  
[5] convert-document https://github.com/occrp-attic/convert-document/blob/master/Dockerfile  
[6] Converting Office Docs to PDF with AWS Lambda https://gist.github.com/madhavpalshikar/96e72889c534443caefd89000b2e69b5  
