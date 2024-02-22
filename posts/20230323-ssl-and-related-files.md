# 深入浅出 SSL/CA 证书及其相关证书文件（pem、crt、cer、key、csr）

> 发布日期：2023-03-23 20:35:00

互联网是虚拟的，通过互联网我们无法正确获取对方真实身份。数字证书是网络世界中的身份证，数字证书为实现双方安全通信提供了电子认证。数字证书中含有密钥对所有者的识别信息，通过验证识别信息的真伪实现对证书持有者身份的认证。数字证书可以在网络世界中为互不见面的用户建立安全可靠的信任关系，这种信任关系的建立则源于 PKI/CA 认证中心，因此，构建安全的 PKI/CA 认证中心是至关重要的。

[![ppyyJUJ.jpg](https://s1.ax1x.com/2023/03/27/ppyyJUJ.jpg)](https://imgse.com/i/ppyyJUJ)

所有与数字证书相关的各种概念和技术，统称为 PKI（ Public Key Infrastructure 公钥基础设施）。PKI 通过引入 CA，数字证书，LDAP，CRL，OCSP 等技术并制定相应标准，有效地解决了公钥与用户映射关系，集中服务性能瓶颈，脱机状态查询等问题。同时为促进并提高证书应用的规范性，还制定了很多与证书应用相关的各种标准。

PKI 的核心是在客户端、服务器和证书颁发机构 (CA) 之间建立的信任。这种信任是通过证书的生成、交换和验证来建立和传播的。

下图例举出了 Authentication 和 Certification 的差别，（双方和三方的区别）。

[![pp6WTAK.md.png](https://s1.ax1x.com/2023/03/28/pp6WTAK.md.png)](https://imgse.com/i/pp6WTAK)

## 概述

SSL 证书是数字证书的一种，类似于驾驶证、护照和营业执照的电子副本。因为配置在服务器上，也称为服务器证书。

SSL 证书只有正确安装到 Web 服务器，才能实现客户端与服务器间的 https 通信。由于涉及到不同类型 Web 服务器的配置，需要在证书签发后，根据实际服务器环境来安装证书。

CA 是一种电子商务认证授权机构，也称为电子商务认证中心，是负责发放和管理数字证书的权威机构，并作为电子商务交易中受信任的第三方，承担公钥体系中公钥的合法性检验的责任。其发布的证书就是 CA 证书。

对于 SSL 证书和 CA 证书的关系，可以从下面两个角度来考量。

#### CA 证书包含 SSL 证书

CA 机构除了可以颁发 SSL 证书之外，还可以颁发其他数字证书，比如：代码签名证书和电子邮件证书等等。从这个角度来说，SSL 证书是一种 CA 证书。

#### CA 证书等于 SSL 证书
证书颁发机构英文简称为 CA，负责签发、作废和保存证书，CA 签发的证书称为 CA 证书，CA 证书的本质是利用 SSL/TLS 协议保护传输数据的安全，因此又称为 SSL 证书。

## 什么是 SSL/TSL

#### SSL（Secure Socket Layer，安全套接字层）

位于可靠的面向连接的网络层协议和应用层协议之间的一种协议层。SSL 通过互相认证、使用数字签名确保完整性、使用加密确保私密性，以实现客户端和服务器之间的安全通讯。该协议由两层组成：SSL 记录协议和 SSL 握手协议。

#### TLS（Transport Layer Security，传输层安全协议）

用于两个应用程序之间提供保密性和数据完整性。该协议由两层组成：TLS 记录协议和 TLS 握手协议。

#### 两者的关系

TLS 与 SSL 连接过程无任何差异，可以理解为 SSL 是 TLS 的前世，TLS 是 SSL 的今生。并且 TLS 与 SSL 的两个协议（记录协议和握手协议）协作工作方式是一样的。

但是，SSL 与 TLS 两者所使用的算法是不同的，同时 TLS 增加了许多新的报警代码。由于这些区别的存在，我们可认为 TLS 是 SSL 的不兼容增强版。在认证证书时 TLS 必须与 TLS 之间交换证书， SSL 必须与 SSL 之间交换证书。

## 证书的签发

#### 证书格式

从分类标准上分，SSL 证书格式主要有

- 公钥证书格式标准 X.509 中定义的 PEM 和 DER
- 公钥加密标准 PKCS 中定义的 PKCS#7 和 PKCS#12
- Java环境专用的 JKS

从文件格式上分，SSL 证书格式主要有：

1. 一种是 Base64 (ASCII) 编码的文本格式。这种证书文件是可以通过文本编辑器打开，甚至进行编辑，常见有 PEM 证书格式，扩展名包括 PEM、CRT 和 KEY。

2. 另外一种是 Binary 二进制文件。常见有 DER 证书格式，扩展名包括 DER 和 CER。

Linux 系统使用 CRT，Windows 系统使用 CER。

| 名词          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| X.509         | 一种通用的证书格式，包含证书持有人的公钥，加加密算法等信息   |
| pkcs1 ~pkcs12 | 公钥加密（非对称加密）的一种标准(Public Key Cryptography Standards)，一般存储为 *.pN，,*.p12 是包含证书和密的封装格式 |
| *.der         | 证书的二进制存储格式（不常用）                               |
| *.pem         | 证书或密钥的 Base64 文本存储格式，可以单独存放证书或密钥，也可以同时存放证书或密钥 |
| *.key         | 单独存放的 pem 格式的密钥，一般保存为 *.key                  |
| *.cer *.crt   | 两个指的都是证书，Linux 下叫 crt，Windows 下叫 cer；存储格式可以是 pem，也可以是 der |
| *.csr         | 证书签名请求（Certificate signing request），包含证书持有人的信息，如：国家，邮件，域名等信息 |
| *.pfx         | 微软 IIS 的实现                                              |
| *.jks         | Java 的 keytool 实现的证书格式                               |

#### 签发过程

[![pp64iOs.png](https://s1.ax1x.com/2023/03/28/pp64iOs.png)](https://imgse.com/i/pp64iOs)

生成 CA 的私钥（后缀名可以用 .pem 也可以用 .key）

```bash
openssl genrsa -out ca.key 2048
```

生成 CA 证书请求文件，会车后会询问一系列基本信息

```bash
openssl req -new -key ca.key -out ca.csr
```

生成证书（公钥包含在证书中），正常情况下，要拿私钥和请求文件去被认可的 CA 机构进行证书申请和签发。这里选择使用 openssl 模拟 CA 机构来签发一个证书。

```bash
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
```

上面三步完成后，在文件夹下会得到三个文件：

```
ca.key
ca.csr
ca.crt
```

可以将生成的证书文件当作 root 证书，以辅助后续的证书信任链的讨论。

生成的证书格式一般是通用的 X509 格式的，其包含证书持有人的公钥，加加密算法等信息。

#### X.509 证书

基于 X.509 V3 标准的证书通过将身份与一对可用于对数字信息进行加密、签名和解密的电子密钥绑定，以实现认证和数据安全（一致性、保密性）的保障。

每一个 X.509 证书都是根据公钥和私钥组成的密钥对来构建的，它们能够用于加解密、身份验证、信息安全性确认。证书的格式和验证方法普遍遵循 X.509 国际标准。

X.509 标准使用了一种抽象语法表示法 One (ASN.1)的接口描述语言，来定义、和编解码客户端与证书颁发机构之间传输的证书请求和证书。

以下便是使用 ASN.1 的证书表示语法。

```json
SignedContent ::= SEQUENCE 
{
  certificate         CertificateToBeSigned,
  algorithm           Object Identifier,
  signature           BITSTRING
}

CertificateToBeSigned ::= SEQUENCE 
{
  version                 [0] CertificateVersion DEFAULT v1,
  serialNumber            CertificateSerialNumber,
  signature               AlgorithmIdentifier,
  issuer                  Name
  validity                Validity,
  subject                 Name
  subjectPublicKeyInfo    SubjectPublicKeyInfo,
  issuerUniqueIdentifier  [1] IMPLICIT UniqueIdentifier OPTIONAL,
  subjectUniqueIdentifier [2] IMPLICIT UniqueIdentifier OPTIONAL,
  extensions              [3] Extensions OPTIONAL
}
```

公钥和私钥都是由一长串随机数组成的。公钥是公开的，由长度来决定保护强度，但是信息会通过公钥来加密。私钥只在接受者处秘密存储，接受者通过使用公钥关联的私钥才能解密读取信息。

使用 openssl 以文本模式查看公钥证书：`openssl x509 -in ca.crt -noout -text ` ，下图是其主要内容的组成。

[![ppcbLN9.md.png](https://s1.ax1x.com/2023/03/29/ppcbLN9.md.png)](https://imgse.com/i/ppcbLN9)



证书包含以下信息：申请者公钥、申请者的组织信息和个人信息、签发机构 CA 的信息、有效时间、证书序列号等信息的明文，同时包含一个签名；

下面是证书操作的常见操作：

```bash
# 查看证书序列号
openssl x509 -in ca.crt -noout -serial
# 打印证书名称以 RFC2253 规定的格式打印出证书的拥有者名字
openssl x509 -in ca.crt -noout -subject
# 打印出证书的 MD5 特征参数
openssl x509 -in ca.crt -noout -fingerprint
# 打印出证书的 SHA 特征参数
openssl x509 -sha1 -in ca.crt -noout -fingerprint
```

#### 格式转换

证书格式的转换其实本质上是编码格式的转换，比如 der 和 pem 的转换。

PEM 转 DER 格式：

```bash
openssl x509 -inform pem -in certificate.pem -outform der -out certificate.der
```

DER 转 PEM 格式：

```bash
openssl x509 -inform der -in certificate.der -outform pem -out certificate.pem
```

#### 重要说明

说明一：证书 = 公钥 + 申请者与颁发者信息 + 签名。

说明二：证书文件的后缀并不能作为证书是哪一种编码的判断依据。对于私钥/公钥的文件后缀有时候用 key/crt，有时候用 pem，其实这不重要，重要的是文件中的内容格式。

## 颁发及信任链

#### 信任链（Trust Chain）

CA 体系是一个树形结构，每一个 CA 可以有一到多个子 CA ，顶层 CA 被称为根 CA 。除了根 CA 以外，其他的 CA 证书的颁发者是它的上一级 CA 。这种层级关系组成了信任链（Trust Chain）。

[![pp6W2p4.png](https://s1.ax1x.com/2023/03/28/pp6W2p4.png)](https://imgse.com/i/pp6W2p4)

以一个实际的例子来看，比如 baidu，在查看证书的时候，就可以看到它的根是 GlobalSign，中间证书是 Validation CA - SHA256 -G2，最后则是 baidu.com。

[![pp6WfXR.jpg](https://s1.ax1x.com/2023/03/28/pp6WfXR.jpg)](https://imgse.com/i/pp6WfXR)

证书分为两种类型（没有本质区别）：

- CA证书（CA Certificate）

- 终端实体证书（End Entity Certificate）  接受CA证书的最终实体。

 #### 认证实例

把在上面已经生成 ca.crt 作为根证书，来签发一个新的证书。

生成一个证书的私钥

```bash
openssl genrsa -out server.key 1024
```

生成证书的请求文件

```bash
openssl req -new -key server.key -out server.csr
```

还是将 openssl 模拟为 CA 机构，用上述生成的 ca.crt 根证书来签发新的证书。

```bash
openssl x509 -req -days 3000 -sha1 -extensions v3_req -CA ca.crt -CAkey ca.key -CAserial ca.srl -CAcreateserial -in server.csr -out server.crt
```

- -CA：指定CA证书的路径

- -CAkey：指定 CA 证书的私钥路径

- -CAserial：指定证书序列号文件的路径

- -CAcreateserial：表示创建证书序列号文件（即上方提到的 serial 文件），创建的序列号文件默认名称为 -CA，指定的证书名称后加上 .srl 后缀

证书校验，用下面命令来校验是否签发成功

```bash
openssl verify -CAfile ca.crt server.crt
# server.crt: OK
```

## 参考文档
[1] SSL证书与CA证书的区别 https://baijiahao.baidu.com/s?id=1653402538679672349&wfr=spider&for=pc
[2] SSL和TSL的区别和联系，以及HTTPS是如何加密解密的 https://www.cnblogs.com/hanzhengjie/p/13920581.html
[3] ssl和tsl区别 https://blog.csdn.net/M_0307/article/details/73543591
[4] CA证书扫盲，https讲解 https://www.cnblogs.com/handsomeBoys/p/6556336.html
[5] PKI/CA与数字证书 https://blog.csdn.net/u013066292/article/details/79538069
[6] SSL证书格式有什么区别？ https://www.gworg.com/problems/1194.html
[7] 如何将.pem转换为.crt和.key？ https://vimsky.com/article/3608.html
[8] 工具：openssl查看pem格式证书细节 https://blog.csdn.net/du_lijun/article/details/115367633
[9] http系列---OpenSSL生成根证书CA及签发子证书 https://blog.csdn.net/lipviolet/article/details/109456104
[10] CA证书详解 https://zhuanlan.zhihu.com/p/267047441
[11] Let's Encrypt介绍 https://www.jianshu.com/p/449047437697
[12] Kubernetes 证书管理系列（一）https://mp.weixin.qq.com/s?__biz=MzI2ODAwMzUwNA==&mid=2649298078&idx=1&sn=24d17a25ccf1c97337e0ed7bc951a8a2&chksm=f2eb8541c59c0c576b3dbbc0fc32bbb0874955a6a83d852b8aa6f35685220d1eb8a135253b47&token=972017317&lang=zh_CN&scene=21#wechat_redirect
[13] CA数字证书简介 https://zhuanlan.zhihu.com/p/413401722
[14] SSL中，公钥、私钥、证书(pem、crt、cer、key、csr)的后缀名都是些啥？ https://blog.csdn.net/HD243608836/article/details/127441701
