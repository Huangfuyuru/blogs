## http

### http特点

##### 高延迟：对头堵塞

队头阻塞是指当顺序发送的请求序列中的一个请求因为某种原因被阻塞时，在后面排队的所有请求也一并被阻塞，会导致客户端迟迟收不到数据。

对于对头堵塞的解决方案：

* 将同一页面的资源分散到不同域名下，提升连接上限。

* Spriting合并多张小图为一张大图,再用JavaScript或者CSS将小图重新“切割”出来的技术。

* 内联(Inlining)是另外一种防止发送很多小图请求的技巧，将图片的原始数据嵌入在CSS文件里面的URL里，减少网络请求次数。
* 拼接(Concatenation)将多个体积较小的JavaScript使用webpack等工具打包成1个体积更大的JavaScript文件

##### 无状态特性--带来的巨大HTTP头部

##### 明文传输--带来的不安全性

### http2新特性

谷歌推出SPDY，才算是正式改造HTTP协议本身。降低延迟，压缩header等等，SPDY的实践证明了这些优化的效果，也最终带来HTTP/2的诞生。

SPDY位于HTTP之下，TCP和SSL之上，这样可以轻松兼容老版本的HTTP协议(将HTTP1.x的内容封装成一种新的frame格式)，同时可以使用已有的SSL功能。

SPDY 协议在Chrome浏览器上证明可行以后，就被当作 HTTP/2 的基础，主要特性都在 HTTP/2 之中得到继承。

##### 二进制传输

HTTP/2 采用二进制格式传输数据，而非HTTP/1.x 里纯文本形式的报文 ，二进制协议解析起来更高效。

##### header压缩

HTTP/2并没有使用传统的压缩算法，而是开发了专门的"HPACK”算法，

##### 多路复用

很多浏览器有一个限制，例如chrome，对于同一个域名，默认允许同时建立6个TCP持久连接，使用持久连接时，虽然能公用一个tcp管道，但是在一个管道中同一时刻只能处理一个请求。在当前请求没有结束之前，其他的请求只能处于堵塞状态。另外如果在同一个域名下同时有10个请求发生，那么其中4个请求会进入排队等待状态，直至进行中的请求完成。

在 HTTP/2 中，有了二进制分帧之后，HTTP /2 不再依赖 TCP 链接去实现多流并行了

* 同域名下所有通信都在单个连接上完成。

* 单个连接可以承载任意数量的双向数据流。

* 数据流以消息的形式发送，而消息又由一个或多个帧组成，多个帧之间可以乱序发送，因为根据帧首部的流标识可以重新组装。

##### server push

HTTP2还在一定程度上改变了传统的“请求-应答”工作模式，服务器不再是完全被动地响应请求，也可以新建“流”主动向客户端发送消息。比如，在浏览器刚请求HTML的时候就提前把可能会用到的JS、CSS文件发给客户端，减少等待的延迟，这被称为"服务器推送"

##### 提高安全性

互联网上通常所能见到的HTTP/2都是使用"https”协议名，跑在TLS上面。



## https

### http存在的问题

* 通信使用明文（不加密），内容可能被窃听：**HTTP报文使用明文（指未经过加密的报文）方式发送**。
* 无法证明报文的完整性，所以可能遭篡改：**没有任何办法确认，发出的请求/响应和接收到的请求/响应是前后相同的**。
* 不验证通信方的身份，因此有可能遭遇伪装：**HTTP协议中的请求和响应不会对通信方进行确认**

### https的解决方案

##### 解决内容被窃听：加密

http直接和tcp通讯，HTTPS并非是应用层的一种新协议。只是HTTP通信接口部分用SSL和TLS协议代替而已。

TLS/SSL 的功能实现主要依赖于三类基本算法：散列函数 、对称加密和非对称加密，其利用非对称加密实现身份认证和密钥协商，对称加密算法采用协商的密钥对数据加密，基于散列函数验证信息的完整性

* 对称加密：加密解密使用同一个密钥（获取到密钥即可破解）
* 非对称加密：使用公钥加密，私钥解密（公钥公开，对于私钥加密信息，黑客后去公钥后即可破解；公钥不包括服务器信息，存在中间人攻击风险；降低传输速率）
* 对称加密+非对称加密(HTTPS采用这种方式)：**发送密文的一方使用对方的公钥进行加密处理“对称的密钥”，然后对方用自己的私钥解密拿到“对称的密钥”，这样可以确保交换的密钥是安全的前提下，使用对称加密方式进行通信**。

##### 解决报文被篡改：数字签名

数字签名的功效：

- 能确定消息确实是由发送方签名并发出来的，因为别人假冒不了发送方的签名。
- 数字签名能确定消息的完整性,证明数据是否未被篡改过。

签名生成：

将一段文本先用Hash函数生成消息摘要，然后用发送者的私钥加密生成数字签名，与原文文一起传送给接收者。接下来就是接收者校验数字签名的流程了。

校验签名：

接收者只有用发送者的公钥才能解密被加密的摘要信息，然后用HASH函数对收到的原文产生一个摘要信息，与上一步得到的摘要信息对比。如果相同，则说明收到的信息是完整的，在传输过程中没有被修改，否则说明信息被修改过，因此数字签名能够验证信息的完整性。	

##### 	解决通讯身份可能被伪装：数字证书

数字证书认证机构的业务流程：

- 服务器的运营人员向第三方机构CA提交公钥、组织信息、个人信息(域名)等信息并申请认证;
- CA审核;
- CA会向申请者签发认证文件-证书。证书包含以下信息：申请者公钥、申请者的组织信息和个人信息、签发机构 CA的信息、有效时间、证书序列号等信息的明文，同时包含一个签名。 其中签名的产生算法：首先，使用散列函数计算公开的明文信息的信息摘要，然后，采用 CA的私钥对信息摘要进行加密，密文即签名;
- 客户端 Client 向服务器 Server 发出请求时，Server 返回证书文件;
- 客户端 Client 读取证书中的相关的明文信息，采用相同的散列函数计算得到信息摘要，然后，利用对应 CA的公钥解密签名数据，对比证书的信息摘要，如果一致，则可以确认证书的合法性，即服务器的公开密钥是值得信赖的。
- 客户端还会验证证书相关的域名信息、有效时间等信息; 客户端会内置信任CA的证书信息(包含公钥)，如果CA不被信任，则找不到对应 CA的证书，证书也会被判定非法。



### https工作流程

![img](https://user-gold-cdn.xitu.io/2019/4/22/16a45839ceacbb52?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



1.Client发起一个HTTPS（比如`https://juejin.im/user/5a9a9cdcf265da238b7d771c`）的请求，根据RFC2818的规定，Client知道需要连接Server的443（默认）端口。

2.Server把事先配置好的公钥证书（public key certificate）返回给客户端。

3.Client验证公钥证书：比如是否在有效期内，证书的用途是不是匹配Client请求的站点，是不是在CRL吊销列表里面，它的上一级证书是否有效，这是一个递归的过程，直到验证到根证书（操作系统内置的Root证书或者Client内置的Root证书）。如果验证通过则继续，不通过则显示警告信息。

4.Client使用伪随机数生成器生成加密所使用的对称密钥，然后用证书的公钥加密这个对称密钥，发给Server。

5.Server使用自己的私钥（private key）解密这个消息，得到对称密钥。至此，Client和Server双方都持有了相同的对称密钥。

6.Server使用对称密钥加密“明文内容A”，发送给Client。

7.Client使用对称密钥解密响应的密文，得到“明文内容A”。

8.Client再次发起HTTPS的请求，使用对称密钥加密请求的“明文内容B”，然后Server使用对称密钥解密密文，得到“明文内容B”。