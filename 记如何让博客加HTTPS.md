# 记如何折腾让博客加HTTPS

> 折腾了一晚上的博客的HTTPS问题，最后才明白一个道理，世上没有免费的午餐，如果有，那么也是你用最宝贵的时间或是其他换来的。
>
> 最终折腾完一圈，回到原点的时候，明白了需求分析而不是说走就走的重要。

## 背景

### 1 - Google 抓取重定向301

在Google搜索site:lincentma.men，查找自己的网站是否被Google收录。

自己的在Google Search Console 中，自己添加了sitemap之后，尝试抓取网站页面，显示：

| [http://lincentma.men/![img](https://www.google.com/webmasters/tools/images/url_icon.png)](http://lincentma.men/) | 请求编入索引                                  |
| ---------------------------------------- | --------------------------------------- |
|                                          | Googlebot类型： 桌面                         |
| ![img](https://www.google.com/webmasters/tools/images/alert.png) | 已重定向 抓取时间：2017年6月26日星期一 GMT-7 下午8:25:37 |
|                                          | 此网址已重定向至：https://lincentma.men/         |

```json
HTTP/1.1 301 Moved Permanently
Content-Type: application/x-gzip
Location: https://lincentma.men/
Server: Coding Pages
Vary: Accept-Encoding
Date: Tue, 27 Jun 2017 03:25:39 GMT
Content-Length: 57
```

导致在Google上没有自己博客更新的文章。

#### 原因：Coding Pages 产生中间页

从上面的HTTP响应头部的标志可以看出，Google重定向到了一个Coding Pages的服务商，也就是我双线同步博客的coding pages。

在其官网上我发现：

> 银牌会员的 Coding Pages 在访问时默认会先加载 Pages 跳转页，您可选择在网站首页任意位置放置「Hosted by Coding Pages」的文字版或图片版，然后勾选下方的「已放置 Hosted by Coding Pages」选项，通过审核后您的 Pages 将不会显示跳转页。请务必将「Hosted by Coding Pages」持续保留在网站首页，撤掉后跳转页会再次出现。

也就是没有银牌会员或者没有在博客上加载广告的时候，就会先跳转到中间页。Google在抓取页面的时候就会发现和目标地址不一致，导致抓取失败。所以没有免费的午餐。

### 2 - 百度 HTTPS站点认证失败

失败详情：您的站点有链接未通过https检验。

### 知识点

- 抓取操作不会跟踪重定向。如果您抓取的网页存在重定向，您将需要手动前往重定向到的网址。
- 301转向(或叫301重定向，301跳转)是当用户或搜索引擎向网站服务器发出浏览请求时，服务器返回的HTTP数据流中头信息(header)中的状态码的一种，表示本网页永久性转移到另一个地址。
- 302重定向是暂时的重定向，搜索引擎会抓取新的内容而保留旧的网址。因为服务器返回302代码，搜索引擎认为新的网址只是暂时的。301重定向是永久的重定向，搜索引擎在抓取新内容的同时也将旧的网址替换为重定向之后的网址。302 重定向会造成网址URL 劫持现象。
- HTTP请求格式：

| **请求头**             | **说明**                            |
| ------------------- | --------------------------------- |
| **Host**            | 接受请求的服务器地址，可以是IP:端口号，也可以是域名       |
| **User-Agent**      | 发送请求的应用程序名称                       |
| **Connection**      | 指定与连接相关的属性，如Connection:Keep-Alive |
| **Accept-Charset**  | 通知服务端可以发送的编码格式                    |
| **Accept-Encoding** | 通知服务端可以发送的数据压缩格式                  |
| **Accept-Language** | 通知服务端可以发送的语言                      |

- HTTP响应格式：

| **响应头**              | **说明**               |
| -------------------- | -------------------- |
| **Server**           | 服务器应用程序软件的名称和版本      |
| **Content-Type**     | 响应正文的类型（是图片还是二进制字符串） |
| **Content-Length**   | 响应正文长度               |
| **Content-Charset**  | 响应正文使用的编码            |
| **Content-Encoding** | 响应正文使用的数据压缩格式        |
| **Content-Language** | 响应正文使用的语言            |

- HTTPS：HTTPS 协议就是 HTTP+SSL/TLS，即在 HTTP 基础上加入 SSL /TLS 层，提供了内容加密、身份认证和数据完整性3大功能，目的就是为了加密数据，用于安全的数据传输。

- 其中一个问题：网站改用 HTTPS 以后，由 HTTP 跳转到 HTTPS 的方式增加了用户访问耗时（多数网站采用 301、302 跳转）

- [推荐文章]: http://support.upyun.com/hc/kb/article/1044299/


- Google宣布了，从2017年1月份正式发布的Chrome 56开始，Google将把某些包含敏感内容的https页面标记为“不安全”。

HTTPS是趋势，然而Github不支持自定义域名的强制HTTPS，Coding国内的必须加广告才能强制HTTPS且没有中间跳转页。所以自己就开始了折腾HTTPS的道路。

## 折腾HTTPS：

### 折腾：配置CloudFare失败

终究会有免费的午餐，那就是CloudFare。

#### 配置教程

[Secure and fast GitHub Pages with CloudFlare]: https://blog.cloudflare.com/secure-and-fast-github-pages-with-cloudflare/
[通过CloudFlare配置Github Pages HTTPS访问]: https://xtimer.org/2017/02/24/%E9%80%9A%E8%BF%87CloudFlare%E9%85%8D%E7%BD%AEGithub-Pages-HTTPS%E8%AE%BF%E9%97%AE/
[为自定义域名的GitHub Pages添加SSL 完整方案]: https://www.yicodes.com/2016/12/04/free-cloudflare-ssl-for-custom-domain/

#### 知识点

- CloudFlare作为一家CDN提供商，他为免费用户提供的服务室不完整的，根据官网SSL服务的介绍，**CloudFlare仅会在浏览器与CloudFlare的通讯中加密，CloudFlare与本地服务器的通讯本身并没有加密。**这也是Flexible和Full模式的区别所在。

- SSL：在客户端与服务器间传输的数据是通过使用对称算法（如 DES 或 RC4）进行加密的。公用密钥算法（通常为 RSA）是用来获得加密密钥交换和数字签名的，此算法使用服务器的SSL数字证书中的公用密钥。有了服务器的SSL数字证书，客户端也可以验证服务器的身份。SSL 协议的版本 1 和 2 只提供服务器认证。版本 3 添加了客户端认证，此认证同时需要客户端和服务器的数字证书。

- 关于Let's Encrypct：

  [使用 Let&#39;s Encrypt 给站点添加 https 小绿锁]: http://www.cnblogs.com/xinpureZhu/articles/lets-encrypt-to-add-https-site-little-green-lock.html

但是，都设置好了。然后就无法访问网页的了。DNS的锅。

发现阿里云的DNS修改后仍处于未更改的状态，即使CloudFlare显示Status：Active。

自己认为可能的是国外DNS被墙，或者是强制HTTPS生效时间过长导致没有及时生效。

### 返回Coding Pages 加广告

无奈返回去添加广告。

#### 知识点

- Hexo ，快速、简单且功能强大的 [Node.js](http://lib.csdn.net/base/nodejs) 博客框架。
- 页面布局
- Hexo中自己感觉一个很有意思的特点是，在_config.yml以及md文件中，通过对于指定字段的true或者false的设置来实现功能，对于使用者来说是黑盒操作，降低了使用难度，提高了使用的体验。这是一种很好的方式。

### 阿里云更改域名配置

买了一个域名：lincentma.men

参考文章：

[hexo博客添加域名实现双线部署（github和coding]: http://blog.csdn.net/qiuchengjia/article/details/52923156

#### 知识点

- 域名解析就是国际域名或者国内域名以及中文域名等域名申请后做的到IP地址的转换过程。IP地址是网路上标识您站点的数字地址，为了简单好记，采用域名来代替ip地址标识站点地址。域名的解析工作由DNS服务器完成。
- 域名解析的A：A (Address) 记录是用来指定主机名（或域名）对应的IP地址记录。用户可以将该域名下的网站服务器指向到自己的web server上。同时也可以设置您域名的二级域名。
- 域名解析的CNAME：别名记录。这种记录允许您将多个名字映射到另外一个域名。通常用于同时提供WWW和MAIL服务的计算机。
- A记录就是把一个域名解析到一个**IP地址**（Address，特制数字IP地址），而CNAME记录就是把域名解析到另外一个**域名**。
- 还有MX记录（邮件记录）和NS记录（解析服务器记录，NS记录只对子域名生效）

## TODO

- [ ] 站点地图的HTTPS是安全的，其他页面是信息不安全的，why？
- [ ] Ngnix与SSL
- [ ] 自己在Github博客申请SSL证书