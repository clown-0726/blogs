# https 是如何保护数据传输的

> 发布日期：2022-05-10 11:43:19

## 为什么需要 https

https 是 http + ssl，也就是加密的 http 数据传输。我们都知道 https 的最主要的作用在于保证数据的安全，但具体来说，https 的安全性主要体现在以下两点：

- 保证**数据传输**不被中间人盗用和信息的泄漏。
- 保证**数据内容**不被劫持、篡改。

针对上面两点，https 的保护策略如下：

- 对传输内容进行加密
- 对请求进行身份验证

## 对称加密和非对称加密

#### 对称加密

只有一个密钥，同时用来进行加密和解密。

![](https://img-blog.csdnimg.cn/img_convert/72e3f774870e8b171fb8ff4d7ba04f33.png)

#### 非对称加密

有一对密钥，分别是公钥和私钥。公钥加密，私钥解密。

![](https://img-blog.csdnimg.cn/img_convert/08f9a552d55eb47b2322a8e0ba2ca33e.png)

## https 的两种数据保护策略

我们知道 https 对数据的保护主要有两种策略，一种是数据加密传输，另一种是请求身份认证。下面的图包含了两种策略的具体处理过程，我们将针对两种保护策略分别阐述。

![](https://img-blog.csdnimg.cn/img_convert/f2fc3a85eecf1060fb6375a999cda92e.png)

#### 如何对传输内容进行加密

我们知道 https 其实就是启用了 SSL 的 http 协议，对于 SSL(Secure Sockets Layer) ，从网络的角度看，其实是位于应用层和传输层之间的安全套接字。如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9b0ce677c1c44734aea2f63c6183d459.png#pic_center)



上面我们介绍了对称加密和非对称加密，对于两种加密手段来说，对称加密的执行效率是高于非对称加密的。

https 在建立连接的过程中，一共发起两次请求，分别使用了对称加密和非对称加密。

我们先忽略身份验证的过程（也就是 CA 证书验证的过程）来看上面的八个步骤。

1. 客户端发送一个 https 的请求到服务端

2. 服务端准备公钥和私钥

3. 服务端将公钥传给客户端

4. 生成一个随机值（用于对称加密，也即是真正数据传输的密钥），然后用服务端的公钥对随机值进行非对称加密

5. 客户端将加密后的随机值传送到服务端（双方都拥有了数据传输的随机值密钥）

6. 服务端使用私钥非对称解密得到客户端的随机值，用获取的随机值将传输的明文内容进行对称加密

7. 服务端把对称加密后的数据传输到客户端

8. 客户端通过随机值对称解密获取明文内容

#### 如何对请求进行身份验证

https 通过对数据进行进行加密传输，保护了数据在传输过程中的安全问题。但是，客户端并不知道所请求的服务端一定是所期望请求的服务端。如果有一个中间人劫持了客户端的请求并同时模拟目标服务端的服务，这样就可以对服务端进行非法劫持操作。

通过 CA 证书所进行的身份验证，目的是让客户端能验证所请求的服务端是自己的目标服务端。CA 是由具有网络公信力的第三方结构进行发放和管理，从而保证了校验的有效性。

CA 证书是服务端通过私钥和公钥向第三方机构申请的，因此第三方机构有权管理和撤销服务端的证书。当建立 https 连接的时候，客户端的第一次请求返回的就不仅仅是公钥了，而是一个 CA 证书（里面包含公钥），得到证书后客户端就可以验证 CA 证书的有效性以确定所请求的服务端是自己的目标服务端。

对上述的八个请求过程增加 CA 证书过程如下：

1. 客户端发送一个https的请求到服务端

2. **服务端准备申请配置好的数字证书（包含公钥）**和私钥

3. **服务端将证书传送给客户端，证书中包含了很多信息，比如证书的颁发机构，过期时间，网址，公钥等**

4. **客户端解析证书，由客户端的 TLS 完成，首先会验证公钥是否有效，比如颁发机构，过期时间等。如果有异常，就会弹出警告信息，并结束通信。**如果正常，则生成一个随机值（用于对称加密，也即是真正数据传输的密钥），然后用服务端的公钥对随机值进行非对称加密

5. 客户端将加密后的随机值传送到服务端（双方都拥有了数据传输的随机值密钥）

6. 服务端使用私钥非对称解密得到客户端的随机值，用获取的随机值将传输的明文内容进行对称加密

7. 服务端把对称加密后的数据传输到客户端

8. 客户端通过随机值对称解密获取明文内容

## 证书签名生成 CA 证书

我们一般通过 openssl 进行密钥和 CA 证书的生成，因此首先保证电脑上安装有 openssl。

```bash
# 验证 openssl 是否安装成功和版本信息
openssl version
```

具体的步骤如下：

#### 步骤一，生成 key 密钥

主要通过 openssl 生成自己的公钥和私钥信息。

通过下面命令生成自己的 idea 密钥

```bash
openssl genrsa -idea -out leaf.key 1024
```

执行后会得到下面的信息，要求输入一个密码，这个密码要牢记。

```bash
Generating RSA private key, 1024 bit long modulus (2 primes)
........+++++
........+++++
e is 65537 (0x010001)
Enter pass phrase for leaf.key:
```

#### 步骤二，生成证书签名请求文件（CSR 文件）

通过 spenssl 生成证书签名请求文件，这个文件其实就是一个申请模版，将这个生成好的模版和公钥私钥一同发送给第三方机构，来申请具备网络公信力的 CA 证书。

可以通过下面的命令结合生成的密钥来生成 CSR 文件。

```bash
openssl req -new -key leaf.key -out leaf.csr
```

执行后会得到下面的信息，我们要根据实际情况来填写下面的信息，当我们要向第三方机构申请 CA 证书的时候，第三方机构也是通过下面的信息对我们进行身份核验。

```bash
Enter pass phrase for leaf.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Liaoning
Locality Name (eg, city) []:Dalian
Organization Name (eg, company) [Internet Widgits Pty Ltd]:My company
Organizational Unit Name (eg, section) []:My unit
Common Name (e.g. server FQDN or YOUR name) []:My name
Email Address []:123456@123.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

#### 步骤三，生成证书签名文件（CA 文件）

第三方结构会根据第二步的请求申请来颁发 CA 证书。

通过上一步我们可以把 csr 文件连同密钥打包发给[证书颁发机构 ](https://baijiahao.baidu.com/s?id=1674868871070040575&amp;wfr=spider&amp;for=pc)，然后证书颁发机构会返回给我们签名的 CA 证书，这里我们通过 openssl 自己模拟证书颁发机构生成 CA 证书，也就是 crt 文件。

通过下面的命令生成自己的 crt 证书

```bash
openssl x509 -req -days 3650 -in leaf.csr -signkey leaf.key -out leaf.crt
```

这样我们最终得到了三个文件：

```bash
leaf.crt
leaf.csr
leaf.key
```

## nginx 配置 CA 证书

首先我们要确定 nginx 已经编译了 ssl 模块，可以通过 `nginx -V` 来进行验证，查看编译参数中是否有 `--with-http_ssl_module` 模块。

#### nginx 的 https 语法配置

```nginx
syntax: ssl on | off;
default: ssl off;
context: http, server;

syntax: ssl_certificate file;
default: -;
context: http, server;

syntax: ssl_certificate_key file;
default: -;
context: http, server;
```

#### 配置实例

```nginx
server {
  listen  443;
  server_name  115.63.100.28 leaf.orbitfin.ai;
  ssl on;
  ssl_certificate /etc/nginx/ssl_key/leaf.crt;
  ssl_certificate_key /etc/nginx/ssl_key/leaf.key;
  
  index index.html index.htm;
  location / {
    root /opt/app/code;
  }
}
```

#### nginx 优化 https 服务

- 激活 keepalive 长链接
- 设置 ssl session 缓存

## 推荐阅读文章

HTTPS explained with carrier pigeons: https://baida.dev/articles/https-explained-with-carrier-pigeons

这篇文章形象的通过信鸽的角度讲述了 HTTPS 是如何进行数据传输进行保密的：
- 通过盒子来传递数据加密的密钥
- 通过第三方对盒子签名确定盒子不是伪造的

## 参考文档

[1] HTTPS 建立连接的过程 https://blog.csdn.net/weixin_42123737/article/details/107370628  
[2] HTTPS建立连接详细过程 https://zhuanlan.zhihu.com/p/107573461  
[3] Let's Encrypt介绍 https://www.jianshu.com/p/449047437697  
[4] 什么是证书颁发机构（CA）https://baijiahao.baidu.com/s?id=1674868871070040575&wfr=spider&for=pc
