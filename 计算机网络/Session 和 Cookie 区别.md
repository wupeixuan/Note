会话跟踪是Web程序中常用的技术，用来跟踪用户的整个会话。常用的会话跟踪技术是Cookie与Session。**Cookie通过在客户端记录信息确定用户身份，Session通过在服务器端记录信息确定用户身份。**

本文将讲解Cookie和Session以及它们的区别。

## Cookie
HTTP 协议是无状态的，主要是为了让 HTTP 协议尽可能简单，使得它能够处理大量事务。HTTP/1.1 引入 Cookie 来保存状态信息。

Cookie 是服务器发送给客户端的数据，该数据会被保存在浏览器中，并且客户端的下一次请求报文会包含该数据。通过 Cookie 可以让服务器知道两个请求是否来自于同一个客户端，从而实现保持登录状态等功能。

### 创建过程
服务器发送的响应报文包含 Set-Cookie 字段，客户端得到响应报文后把 Cookie 内容保存到浏览器中。
```
HTTP/1.1 200 OK
Server: nginx/1.10.3 (Ubuntu)
Date: Wed, 04 Apr 2018 07:52:53 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
Vary: Cookie
X-Frame-Options: SAMEORIGIN
Set-Cookie: csrftoken=Unoyq4GhHrctiYdxp02xjl4exWS5JYmTzYm2UHJUwjeR0UFMIyv4CxUFicFDcGyu; expires=Wed, 03-Apr-2019 07:52:53 GMT; Max-Age=31449600; Path=/; secure
Strict-Transport-Security: max-age=315360000; includeSubDomains
Content-Encoding: gzip
```
客户端之后发送请求时，会从浏览器中读出 Cookie 值，在请求报文中包含 Cookie 字段。
```
GET / HTTP/1.1
Host: leetcode-cn.com
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.75 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: Hm_lvt_fa218a3ff7179639febdb15e372f411c=1522568819,1522574611,1522657411,1522828353; Hm_lpvt_fa218a3ff7179639febdb15e372f411c=1522828370; csrftoken=Unoyq4GhHrctiYdxp02xjl4exWS5JYmTzYm2UHJUwjeR0UFMIyv4CxUFicFDcGyu
```
### 分类
- 会话期 Cookie：浏览器关闭之后它会被自动删除，也就是说它仅在会话期内有效。
- 持久性 Cookie：指定一个特定的过期时间（Expires）或有效期（Max-Age）之后就成为了持久性的 Cookie。
```
Set-Cookie: csrftoken=Unoyq4GhHrctiYdxp02xjl4exWS5JYmTzYm2UHJUwjeR0UFMIyv4CxUFicFDcGyu; expires=Wed, 03-Apr-2019 07:52:53 GMT; Max-Age=31449600; 
```
### Set-Cookie

属性 | 说明 
---|---
NAME=VALUE | 赋予 Cookie 的名称和其值（必需项） 
expires=DATE | Cookie 的有效期（若不明确指定则默认为浏览器关闭前为止）
path=PATH | 将服务器上的文件目录作为 Cookie 的适用对象（若不指定则默认为文档所在的文件目录）
domain=域名 | 作为 Cookie 适用对象的域名（若不指定则默认为创建 Cookie 的服务器的域名）
Secure | 仅在 HTTPs 安全通信时才会发送 Cookie
HttpOnly | 加以限制，使 Cookie 不能被 JavaScript 脚本访问

## Session
Session 是服务器用来跟踪用户的一种手段，使用上比Cookie简单一些，相应的也增加了**服务器的存储压力**。每个 Session 都有一个唯一标识：Session ID。当服务器创建了一个 Session 时，给客户端发送的响应报文包含了 Set-Cookie 字段，其中有一个名为 sid 的键值对，这个键值对就是 Session ID。客户端收到后就把 Cookie 保存在浏览器中，并且之后发送的请求报文都包含 Session ID。

当Cookie被禁用时，可以跟在url的后面，或者以表单的形式提交到服务器端，从而使服务器端了解客户端的状态。

## 两者比较
联系：

Cookie与Session都是用来跟踪浏览器用户身份的会话方式。

区别：

- Cookie数据存放在客户的浏览器上，Session数据放在服务器上。
- Cookie不是很安全，别人可以分析存放在本地的Cookie并进行Cookie欺骗,如果主要考虑到安全应当使用加密的Cookie或者Session。
- Session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，如果主要考虑到减轻服务器性能方面，应当使用Cookie。
- 单个Cookie在客户端的限制是4K，很多浏览器都限制一个站点最多保存20个Cookie。
