# 如何使用 pdf.js 在线显示 pdf 文档

> 发布日期：2020-10-21 15:44:37

由于项目中需要在线显示 PDF 文档，在搜索了很多资料之后，除了一些付费的方案，基本锁定了 pdf.js 作为最好的解决方案。

<!--more-->

![努力+不要脸=人生赢家](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20201021-how-to-use-pdfjs-viewer/buyaolian_qinfen_renshengyingjia.jpg)

在做调研的过程中，用的比较多的，发现一共有三种基本方案。第一种是直接通过浏览器来预览。这种方案非常依赖于浏览器及其版本，因此不推荐。第二种是使用 Google 的 online docs viewer，但是在国内没有良好的体验，并且定制行不强。剩下的就是 pdf.js。pdf.js 基本可以满足项目中的基础使用，因此也是最好的选择。

我们可以在 github 中找到这个项目。https://github.com/mozilla/pdf.js/ 。但是官网的文旦写的有点少，不够具体，因此在这里分享一下使用的过程。

### 第一种使用方式（不推荐）

第一种使用方式也就是官网介绍的一种方式，就是将加载完毕的 pdf 文档通过 canvas 来绘制出来。在尝试了这种方式之后发现这种方式就像是在看图片，内部文字没法进行选择，并且加载效率比较慢。因此这种方式不推荐使用，有兴趣的可以尝试一下这种展示方式。

```
pdf.js: 将 PDF 文件解析后生成一张 .png 图片,利用 canvas 元素显示在页面上,此方法不推荐使用,
呈现在页面上的pdf会模糊,目前没有找到有效解决办法,给爱钻研的小伙伴提供个思路,在pdf.js官网上有这样一句话 :
 Each PDF page has its own viewport which defines the size in pixels(72DPI) and initial rotation. 
猜想如果可以改变默认72DPI就可改变呈现的清晰度
```

上面是是引用的一片文章的一段话，可以看出 canvas 实际上是不推荐在正式项目中使用的。

### 第二种使用方式（服务调取的方式）

pdf.js 的整个项目是可以直接作为一个静态文件被比如 nginx 来进行服务托管的，也就是说这就提供给了我们一种如同 `google docs viewer` 的方式进行在线查看 pdf 文档。我们只需要调用比如 `https://service-name/web/viewer.html?file=http://servier-name/abc.pdf` 来进行在线的 pdf 预览。

具体配置过程如下：

#### 第一步

准备一台服务器，并且在服务器上配置好 nginx 服务。

在服务器上准备好 node.js 的运行时环境。

#### 第二步

下载 pdf.js 的项目工程，并进入工程的根目录运行 `npm install` 来安装必要的依赖包。

配置 nginx 静态托管，将工程给拉起来，下面是我的 nginx 基本配置

```nginx
server {
    listen       1004;
    server_name  localhost;
    
    location / {
        add_header Cache-Control "no-cache, no-store";
        root /Users/crown/Projects/pdfjs;
        try_files $uri $uri/ /index.html;
    }
}

```

比如我的服务器的地址是 `101.23.56.18`，那么我就可以通过 `http://101.23.56.18:1004/web/viewer.html` 进行测试访问了。但是这还不够，我们只能访问在工程内部的文件，并不能通过传递参数的方式进行自定义的 pdf 目标的访问。

参考这篇文章做 pdf.js 的基本使用配置：https://blog.csdn.net/tang242424/article/details/82696763

#### 第三步

我们想使用我们自己配置的服务预览其他服务器的 pdf 文档的时候，比如 aws s3 上的 pdf 文件，我们可以通过传递参数的方式。就是在url 后面加上参数 `?file=xxx` 来进行访问，但是这在 pdf.js 中是跨域的，因此我们需要服务器代理访问，参考下面配置：

```nginx
server {
    listen       1004;
    server_name  localhost;
    
    location / {
        add_header Cache-Control "no-cache, no-store";
        root /Users/crown/Projects/pdfjs;
        try_files $uri $uri/ /index.html;
    }

    location /filing-reports/ {
        proxy_pass https://filing-reports.s3.amazonaws.com/;
    }
}
```

当我们要访问 `https://filing-reports.s3.amazonaws.com/abc.pdf` 这个文件的时候，我们可以通过上面的代理配置进行访问。

修改前的 url 是`http://101.23.56.18:1004/web/viewer.html?file=https://filing-reports.s3.amazonaws.com/abc.pdf`，修改后的url 是 `http://101.23.56.18:1004/web/viewer.html?file=/filing-reports/abc.pdf` 。

这样代理就将路径 `/filing-reports/`转到了 `https://filing-reports.s3.amazonaws.com/` ，因此就可以正常的进行访问了。

### 优点

- 通过这种方式我们可以独立出 pdf 显示这个模块，这样可以和项目进行了分离。当我们需要在某一个网页上调用自己的服务的时候，我们可以很方便的把服务加到 `iframe` 中去预览pdf。比如下面代码

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
  </head>
  <body>
      <iframe src="http://localhost:8888/web/viewer.html??file=/filing-reports/reports-data/stock_au/XASX/2020/08/28/44m1wb41p7m6gr.pdf?AWSAccessKeyId=AKIAZ2SDT5DU46K54RGA&Signature=WNm48xMIlawgIwCj5R%2FlCidcIQM%3D&Expires=1603178276" width="1000" height="500"></iframe>
  </body>
  </html>
  ```

- 比较好的安全性。因为服务是我们自己搭建，因此不涉及其他服务私自存储项目的 pdf 文件。我们自己的项目是利用 aws 的 `Auto scale group` 来管理服务的，如果服务压力比较大，asg 会自动增加服务器进行扩展。当然也可以把服务很方便的搭建在集群上。

### Tips

- 参考这篇文章修改 pdf 的跨域使其允许跨域 [使用pdf.js实现前端页面预览pdf文档，解决了跨域请求](https://www.cnblogs.com/-lile/p/11451131.html)。不推荐，会增加项目的安全隐患。
- 参考pdf GitHub 上的[WIKI](https://github.com/mozilla/pdf.js/wiki/Viewer-options) 来获得更多的初始化参数。
- 参考这篇文章 [pdfjs使用指南](https://blog.csdn.net/atLuckStar/article/details/77688972) ，来获得一些长问问题的解答。

### References:

[1] Hello World Walkthrough: https://mozilla.github.io/pdf.js/examples/  
[2] pdf.js使用方法: https://blog.csdn.net/bianliuzhu/article/details/80622215  
[3] pdf.js简易教程: https://blog.csdn.net/tang242424/article/details/82696763  
[4] PDF.js 超简单使用教程，一看就会！: https://blog.csdn.net/tangran0526/article/details/103274303?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.add_param_isCf&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.add_param_isCf  
[5] 使用pdf.js实现前端页面预览pdf文档，解决了跨域请求: https://www.cnblogs.com/-lile/p/11451131.html  
[6] Viewer options: https://github.com/mozilla/pdf.js/wiki/Viewer-options  
[7] pdfjs使用指南: https://blog.csdn.net/atLuckStar/article/details/77688972

