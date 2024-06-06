# aws lambda 转换 office/txt/html 为 pdf

> 发布日期：2023-04-02 22:59:00

简洁的写作需要勇气。让事物变小是一种深思熟虑的、困难的和有价值的行为。大多数书籍本应是一篇博客文章。大多数博客文章本应是一条微博。大多数微博本应不写。

[![ppfvMM4.md.png](https://s1.ax1x.com/2023/04/03/ppfvMM4.md.png)](https://imgse.com/i/ppfvMM4)

## 概述

- 
- sqs 消息队列的方式触发 lambda，可以对任务过多时进行有效的“削峰”处理。

## 构建转换环境


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
