# HTTP 下的前后端交互

> 发布日期：2021-08-15 16:42:55

互联网上半场的打法可以说是营销和资本驱动（你在选择安装哪个APP）。而互联网的下半场是数据和技术在驱动（你在选择卸载哪个APP）。

<!--more-->

![我工作的大楼](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210815-http-knowledge/sanfeng-my-office.jpeg)

## HTTP 版本对比
HTTP 版本的演化到现在为止一种经历了四个版本，分别是 HTTP/0.9，HTTP/1.0，HTTP/1.1 和 HTTP/2.0。现在使用最广泛的仍然是 1.1 版本，但是 2.0 版本也慢慢的被各个网站接纳和使用。下面分别介绍上面几个版本的区别。

#### 1991年 HTTP/0.9
- 仅支持 GET 请求，不支持请求头

#### 1996年 HTTP/1.0
- 默认短连接（一次请求建议一次 TCP 连接，请求完就断开）
- 支持 GET、POST、 HEAD 请求

#### 1999年 HTTP/1.1
- 默认长连接（一次TCP连接可以多次请求）
- 支持 PUT、DELETE、PATCH 等六种请求
- 增加 host 头，支持虚拟主机
- 支持断点续传功能

#### 2015年 HTTP/2.0
- 多路复用，降低开销（一次TCP连接可以处理多个请求）
- 服务器主动推送（相关资源一个请求全部推送）
- 解析基于二进制，解析错误少，更高效（HTTP/1.X 解析基于文本）
- 报头压缩，降低开销
- 默认使用加密

#### 目前处于定制和测试阶段 HTTP/3.0
- HTTP/3.0 不是 HTTP/2.0 的扩展，而是一个全新的 WEB 协议
- HTTP/3.0 是 HTTP-over-QUIC (Quick UDP Internet Connection)

## HTTP 的组成
HTTP 的组成一共有三部分，分别是“请求行”，“请求头”和“请求体”。

在这一部分主要讨论常规的请求头/响应头，对于一些复杂一些的比如缓存控制，Cookie 操作和传参方式我们单独拿出来进行介绍。

#### 常规请求行
以下是一个请求的请求行部分，主要由三部分组成，分别是 method，path 和 scheme
```
GET /home/xman/data/tipspluslist?indextype=manht HTTP/1.1
```

#### 常规请求/响应头
下面的代码是一个请求的常规请求头，基本上大多数的请求都会有这些内容，也是我们在进行常规请求的时候不需要自己进行关注和控制的。但是以下几点值得注意：
- `q=0.x` 这个是告诉浏览器其前面部分的权重，比如下面请求头中的 `accept-language` 部分，语言的选择权重高低分别是 en-US > en > zh-CN > zh > fr
- `referer` 是指当前请求是由哪一个请求发起的，也就是其父请求或上一个请求是谁，一般用于防盗链
- `user-agent` 代表向服务器发送当前客户端环境，在写爬虫的时候尤其要注意这个字段
```
accept: application/json, text/plain, */*
accept-encoding: gzip, deflate, br
accept-language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7,fr;q=0.6
content-type: application/json;charset=UTF-8
referer: https://vision-qa.orbitfin.ai/dashboard
user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36
```

下面是一个常规的响应头部分：
- `content-encoding` 服务器发给浏览器数据的压缩格式，一般大多数都是 gzip
- `content-type` 是返回的数据类型格式，区别于文件扩展名，类型是 Mime Type（可以参照：https://www.w3school.com.cn/media/media_mimeref.asp）
- `date` 是返回时间
```
content-encoding: gzip
content-type: application/json
date: Sun, 15 Aug 2021 07:52:06 GMT
```

#### HTTP/2.0 的伪头字段
伪头字段是 HTTP/2.0 内置的几个特殊的以“:”开头的 key，用于替代 HTTP/1.x 中请求行/响应行的信息。
- :method 目标 URL 方法部分（请求）
- :scheme 目标 URL 模式部分（请求）
- :authority 目标 URL 认证部分（请求）
- :path 目标 URL 的路径和查询部分（绝对路径产生式和一个跟着“?”字符的查询产生式）（请求）
- :status 响应头中的 HTTP 状态码部分（响应）
![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210815-http-knowledge/http2-de-wei-tou-zi-duan.png)

## 缓存协商
#### 影响缓存的请求头字段

http 中和缓存相关的头一共有以下几个：

| 响应头        | 请求头                                            |
| ------------- | ------------------------------------------------- |
| Cache-Control | Cache-Control（一般只用Cache-Control: max-age=0） |
| Expires       | N/A                                               |
| Etag          | If-None-Match                                     |
| Last-Modified | If-Modified-Since                                 |

优先级从高到低分别是 `Cache-Control`，`Expires`，`Etag`，`Last-Modified`

`Expires` 这个响应头比较简单，一般就是过期时间，也叫做强缓。缺点是由于时区问题可能导致服务器时间和浏览器时间不一致从而导致缓存的混乱，并且一下杀毒软件比如 360 可能会误删混存。导致客户端缓存不可用。

`Cache-Control` 是 HTTP/1.0 加上的，为了避免 `Expires` 客户端和服务端时间不一致的情况。内容比较多，下面依次介绍

- `public`: 表示所有的节点，包括代理服务器可以进行缓存。
- `private`: 表示只有浏览器才可以缓存。
- `no-cache`: 表示可以缓存，但是每次要去服务器验证可不可以使用缓存来决定是否使用（一般配合 Etag 和 Last-Modified）。
- `no-store`: 浏览器和代理服务器都彻底不可以缓存。
- `max-age=<seconds>`: 用于告诉浏览器缓存的时间。
- `s-maxage=<secnds>`: 专门用于代理服务器的缓存时间，浏览器则依然用max-age（不常用）。
- `max-stale`: 这是发起请求一方主动带的头，当即便缓存过期，也使用缓存（不常用）。
- `must-revalidate`: 当缓存内容过期，必须去源头验证（不常用）。
- `proxy-revalidate`: 用于代理服务器，当缓存过期，必须去源头请求缓存（不常用）。
- `no-transform`: 一些代理服务器可能会将缓存进行压缩等操作以优化传输速度，这个标志告诉代理服务器不做任何操作（不常用）。

`If-None-Match/Etag` 主要是通过内通的哈希进行缓存验证。

`If-Modified-Since/Last-Modified` 主要是通过所设置的内容过期时间进行验证。

#### 缓存协商的过程

下图是缓存进行协商的过程，我们逐步解释。

当浏览器去向服务器请求资源的时候，服务器主要是先通过 `Expires` 和 `Cache-Control: max-age=3600` 对浏览器进行时间缓存的控制。这里注意 `Expires` 由于时区问题可能导致服务器时间和浏览器时间不一致从而导致缓存的混乱，因此很多时候这两个缓存头字段一般会同时使用的。

当缓存过期之后，如果之前的请求服务器**没有返回** `Etag` 或 `Last-Modified` 缓存头字段，那么浏览器会直接去服务器重新请求新的内容。

当缓存过期之后，如果之前的请求服务器**已经返回** `Etag` 或 `Last-Modified` 缓存头字段，那么这时候浏览器会带上对应的 `If-None-Match` 和 `If-Modified-Since` 字段去服务器进行验证，从上一部分我们得知，一个是通过时间进行验证，另一个是通过哈希进行缓存验证。

- 如果缓存不能用，这时候请求的状态码一般就是 200，表示重新获取了内容。
- 如果可以继续使用缓存，那么状态码则会是 304，表示继续使用浏览器缓存。

当 `Cache-Control: max-age=3600, no-cache`，也就是 `Cache-Control` 中有 `no-cache` 的时候，虽然有过期时间，当页面刷新或跳转的时候浏览器每次还是要通过 `Etag` 或 `Last-Modified`  和服务器进行缓存协商。

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210815-http-knowledge/cache-negotiation.png)

#### 实践中的缓存策略
对于公共的库，我们一般推荐使用上面四个缓存头来进行常规的设置。

但是对于业务代码是经常变化的，我们一般使用 `localStorage` 进行缓存，推荐库 `basket.js`。

## 长链接

我们知道浏览器在进行数据 http 数据传输之前是要建立 TCP 链接的，早在 HTTP/1.0 之前，没建立一次 TCP 只能供一个 HTTP 链接使用，这就造成了很大的资源消耗，并且造成了传输的低效。HTTP/1.0 引入 `Keep-Alive` 这个头主要用于让多个 HTTP 来复用一个 TCP 链接。

我们可以在 chrome 开发工具的 network 标签中打开 `Connection-ID` 来查看 TCP 链接复用情况。

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210815-http-knowledge/open-connection-id-in-network.png)

`Keep-Alive` 解决的核心问题就在此，一定时间内，同一域名多次请求数据，只建立一次 HTTP 请求，其他请求可复用每一次建立的链接通道，已达到提高请求效率的问题。Chrome 默认的同一域名下最大链接数为 6。

HTTP/1.1 还是存在效率问题

- 串行的文件传输
- 连接数过多

HTTP/2.0 对同一域名下所有请求都是基于流也就是同一域名不管访问多少文件，也只建立一路链接。同样 Apache 的最大链接数为 300，因为有了这个特性，最大的并发就可以提升到 300，比原来能提高 6倍。

## Cookie

Cookie 是服务端通过 `set-cookie` 这个响应头字段往浏览器上设置的信息，格式为键值对的形式，下面是一段例子。

```
set-cookie: sessionid=a2ajp684854go8isa38rnvjjmcdxs3j9; expires=Sun, 29 Aug 2021 08:07:28 GMT; HttpOnly; Max-Age=1209600; Path=/; SameSite=Lax
```

Cookie 中有许多在字段单独解释一下：

| Key      | Meaning                           |
| -------- | --------------------------------- |
| expires  | 设置过期时间                      |
| Max-Age  | 设置过期时间                      |
| Path     | cookie 所在的域地址               |
| HttpOnly | 无法通过 document.cookie 进行访问 |
| Secure   | 只在 https 请求中进行发送         |

下面是一段浏览器发动服务端的 `cookie` 请求字段

```
cookie: _ga=GA1.2.863750735.1629014837; _gid=GA1.2.239074351.1629014837; token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6IjU3NjQ2OTJAcXEuY29tIiwiaWF0IjoxNjI5MDE0ODQ3LCJleHAiOjE2MjkxMDEyNDcsImp0aSI6IjMzYTM0NjA5LWU4ZmUtNDVlZC05NDZkLWU4ZDViYjJhYzVhYiIsInVzZXJfaWQiOjUsIm9yaWdfaWF0IjoxNjI5MDE0ODQ3fQ.YjwkXL9kAiztT17JNWT3pbwWAyRAUl_u4MJsZJQk11k; sessionid=a2ajp684854go8isa38rnvjjmcdxs3j9
```

## 页面重定向

页面重定向是服务端通知浏览器的行为。主要是 301 和 302 两个状态码配合 `Location: /new`  这个响应头字段来告诉浏览器要重定向到哪个新地址。

301 是永久重定向，也就是第一次通知浏览器重定向之后浏览器如果再请求一个重定向后的网址后会直接去新的地址，不需要再去请求服务端来。

302 是临时重定向，每次浏览器都要请求服务端询问新的重定向的地址。

## 请求跨域 CORS

跨域错误是前段开发中最常见的错误之一了。我们要知道，不管跨域行为有没有产生，浏览器请求数据的行为都发生了，但是浏览器在进行数据解析的时候发现 header 中没有 `Access-Control-Allow-Origin` 这个请求头对是否跨域做声明，所有把请求内容忽略掉，并且在控制台中报出跨域的错误。

所以我们在做服务器请求服务器的请求中是不存在跨域问题的。这也是浏览器规定的 ajax 的同域限制。

浏览器是允许在比如 `link`，`script`，`image` 等标签在请求时的跨域行为的。`JSONP` 也就是利用了这个特性，请求服务器的一段 js 代码，从而绕过浏览器的跨域限制。

是否我们设置了 `Access-Control-Allow-Origin: *` 之后跨域就全部解决了呢？

其实这只是开始。

其实浏览器在跨域请求之前会先发一个预请求来询问服务器的跨域行为。

![](https://lilu-pic-bed.oss-cn-beijing.aliyuncs.com/my-blog/20210815-http-knowledge/cors-pre-request.png)

CORS 默认的限制很多，并不只是上面提到的域名限制，在请求方法，请求头等行为上都有一些限制，可以简单参考如下列表。

| 主题         | 内容                                                         |
| ------------ | ------------------------------------------------------------ |
| 允许方法     | GET, HEAD, POST                                              |
| Content-Type | text/plain, multipart/form-data, application/x-www-form-urlencoded |
| 请求头       | Accept, Accept-Language 等                                   |

更详细的 CORS 限制可以参考网址：https://fetch.spec.whatwg.org/#cors-safelisted-request-header

比如我们在跨域请求中带上一个自定义的请求头，或者我们使用了不被允许的请求方法，那么这个跨域仍然是请求不通的，浏览器还是会给拦截的。

我们可以开放更多的跨域行为来让跨域请求可以完成，下面代码基本上开放了跨域限制，但是这是不安全的，还要根据业务去开放一部分的跨域限制。

```
// 指定允许其他域名访问
'Access-Control-Allow-Origin:*'
// 是否允许后续请求携带认证信息（cookies），该值只能是 true，否则不返回
'Access-Control-Allow-Credentials:true'
// 预检结果缓存时间
'Access-Control-Max-Age: 1800'
// 允许的请求类型
'Access-Control-Allow-Methods:*'
// 允许的请求头字段
'Access-Control-Allow-Headers:*'
```

## 内容安全策略（CSP）

CSP 是 Content Security Policy 的缩写，CSP 的主要目标是减少和报告 XSS 攻击，主要有以下两个主要功能：

1. 限制资源获取
2. 报告资源获取越权

CSP 可限制的内容种类非常的全，默认 `default-src` 是对全种类资源进行显示，当然也可以限制单独的种类，比如 `connect-src` 来限制 ajax，`img-src` 来限制图片的加载，`style-src` 来限制样式资源的加载。详细可以参照 MDN 的介绍：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP。

#### 示例1:

一个网站管理者允许内容来自信任的域名及其子域名 (域名不必须与CSP设置所在的域名相同)

```
Content-Security-Policy: default-src 'self' *.trusted.com
```

 #### 示例2:

一个网站管理者允许网页应用的用户在他们自己的内容中包含来自任何源的图片, 但是限制音频或视频需从信任的资源提供者(获得)，所有脚本必须从特定主机服务器获取可信的代码。

```
Content-Security-Policy: default-src 'self'; img-src *; media-src media1.com media2.com; script-src userscripts.example.com
```

#### 通过 http 的 meta 标签设置

CSP 的使用不仅仅可以通过 response 返回头进行内容限制，同样可以通过 html 的 meta 标签进行限制，比如：

```
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">
```

#### 违例报告

我们在进行限制的时候同样可以启用违例报告，让服务器知道都有哪些资源的使用时不合法的：

```
Content-Security-Policy: default-src 'self'; report-uri http://reportcollector.example.com/collector.cgi
```

## 参考文档

[1] 面试带你飞：这是一份全面的 计算机网络基础 总结攻略：https://juejin.cn/post/6844903592965439501  
[2] 网络分层架构（七/四层协议）：https://blog.csdn.net/qq_38560742/article/details/88398270  
[3] 内容安全策略( CSP )：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP