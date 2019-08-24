---
title: HTTPH/TTPS协议
date: 2019-08-24 19:02:25
tags: [HTTP]
categories: [HTTP]
cover: https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/HTTP%20HTTP%20%E5%8D%8F%E8%AE%AE%20URI/%E4%B8%8B%E8%BD%BD.png?raw=true
---

## 协议

网络协议是计算机之间为了实现网络通信而达成的一种“约定”或者”规则“，有了这种”约定“，不同厂商的生产设备，以及不同操作系统组成的计算机之间，就可以实现通信。

### HTTP协议
HTTP协议是超文本传输协议的缩写，英文是Hyper Text Transfer Protocol。它是从WEB服务器传输超文本标记语言(HTML)到本地浏览器的传送协议。

设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。

HTPP有多个版本，目前广泛使用的是HTTP/1.1版本。

## HTTP
HTTP 全称是 HyperText Transfer Protocal ，即：超文本传输协议，从 1990 年开始就在 WWW 上广泛应用，是现今在 WWW 上应用最多的协议，HTTP 是应用层协议，当你上网浏览网页的时候，浏览器和 web 服务器之间就会通过 HTTP 在 Internet 上进行数据的发送和接收。HTTP 是一个基于请求/响应模式的、无状态的协议。即我们通常所说的 Request/Response

### HTTP原理

HTTP是一个基于TCP/IP通信协议来传递数据的协议，传输的数据类型为HTML 文件,、图片文件, 查询结果等。

HTTP协议一般用于B/S架构（）。浏览器作为HTTP客户端通过URL向HTTP服务端即WEB服务器发送所有请求。

### HTTP协议

HTTP协议是超文本传输协议的缩写，英文是Hyper Text Transfer Protocol。它是从WEB服务器传输超文本标记语言(HTML)到本地浏览器的传送协议。

设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。

HTPP有多个版本，目前广泛使用的是HTTP/1.1版本。

### HTTP特点

 - 支持客户端/服务器模式
 - 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。由于 HTTP 协议简单，使得 HTTP 服务器的程序规模小，因而通信速度很快
 - 灵活：HTTP 允许传输任意类型的数据对象。正在传输的类型由 Content-Type 加以标记
 - 无连接：无连接的含义是限制每次链接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开链接，采用这种方式可以节省传输时间
 - 无状态：HTTP 协议是无状态协议。无状态是指协议对于事物处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能会导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就比较快

## HTTPS
HTTPS 协议（HyperText Transfer Protocol over Secure Socket Layer）：一般理解为HTTP+SSL/TLS，通过 SSL证书来验证服务器的身份，并为浏览器和服务器之间的通信进行加密。

解决HTTP的一些问题
 - 请求信息明文传输，容易被窃听截取。
 - 数据的完整性未校验，容易被篡改
 - 没有验证对方身份，存在冒充危险


## 总结HTTPS和HTTP的区别

 - HTTPS是HTTP协议的安全版本，HTTP协议的数据传输是明文的，是不安全的，HTTPS使用了SSL/TLS协议进行了加密处理。
 - http和https使用连接方式不同，默认端口也不一样，http是80，https是443。

## URL详解

URL（Uniform Resource Locator）是统一资源定位符的简称，有时候也被俗称为网页地址（网址），如同是网络上的门牌，是因特网上标准的资源的地址

### 基本组成

| 名称         | 功能                                                         |
| ------------ | ------------------------------------------------------------ |
| scheme       | 访问服务器以获取资源时要使用哪种协议，比如，http，https 和 FTP 等 |
| host         | HTTP 服务器的 IP 地址或域名                                  |
| port#        | HTTP 服务器的默认端口是 80，这种情况下端口号可以省略，如果使用了别的端口，必须指明，例如www.cnblogs.com：8080 |
| path         | 访问资源的路径                                               |
| query-string | 发给 http 服务器的数据                                       |
| anchor       | 锚                                                           |

![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/HTTP%20HTTP%20%E5%8D%8F%E8%AE%AE%20URI/15fc2525666dc96e.jpeg?raw=true)

## HTTP请求
### 类型
| 名称    | 功能                                                         |
| ------- | ------------------------------------------------------------ |
| GET     | 向指定的资源发出“显示”请求，使用 GET 方法应该只用在读取数据上，而不应该用于产生“副作用”的操作中 |
| POST    | 指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求文本中。这个请求可能会创建新的资源或者修改现有资源，或两者皆有。 |
| PUT     | 向指定资源位置上传其最新内容                                 |
| DELETE  | 请求服务器删除 Request-URI 所标识的资源                      |
| OPTIONS | 使服务器传回该资源所支持的所有HTTP请求方法。用*来代替资源名称，向 Web 服务器发送 OPTIONS 请求，可以测试服务器功能是否正常运作 |
| HEAD    | 与 GET 方法一样，都是向服务器发出指定资源的请求，只不过服务器将不传回资源的本文部分，它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中关于该资源的信息（原信息或称元数据） |
| TRACE   | 显示服务器收到的请求，主要用于测试或诊断                     |
| CONNECT | HTTP/1.1 中预留给能够将连接改为通道方式的代理服务器。通常用于 SSL 加密服务器的链接（经由非加密的 HTTP 代理服务器） |

其中，最常见的是 GET 和 POST 方法，如果是 RESful 接口的话一般会用到 PUT、DELETE、GET、POST（分别对应增删查改

### 请求头
| 名称              | 功能                                                         |
| ----------------- | ------------------------------------------------------------ |
| Authorization     | 用于设置身份认证信息                                         |
| User-Agent        | 用户标识，如：OS 和浏览器的类型和版本                        |
| If-Modified-Since | 值为上一次服务器返回的Last-Modified值，用于确定某个资源是否被更改过，没有更改过就从缓存中读取 |
| If-None-Match     | 值为上一次服务器返回的 ETag 值，一般会和If-Modified-Since    |
| Cookie            | 已有的Cookie                                                 |
| Referer           | 标识请求引用自哪个地址，比如你从页面 A 跳转到页面 B 时，值为页面 A 的地址 |
| Host              | 请求的主机和端口号                                           |

### 状态码
| 状态码 | 对应的信息                                                   |
| ------ | ------------------------------------------------------------ |
| 1XX    | 提示信息—表示请求已接收，继续处理                            |
| 2XX    | 用于表示请求已被成功接收、理解、接收                         |
| 3XX    | 用于表示资源（网页等）被永久转移到其它 URL，也就是所谓的重定向 |
| 4XX    | 客户端错误—请求有语法错误或者请求无法实现                    |
| 5XX    | 服务器端错误—服务器未能实现合法的请求                        |

### 响应头
| 名称              | 功能                                                         |
| ----------------- | ------------------------------------------------------------ |
| Date              | 服务器的日期                                                 |
| Last-Modified     | 该资源最后被修改的时间                                       |
| Transfer-Encoding | 取值一般为 chunked，出现在 Content-Length 不能确定的情况下，表示服务器不知道响应板体的数据大小，一般同时出现Content-Encoding响应头 |
| Set-Cookie        | 设置 Cookie                                                  |
| Location          | 重定向到另一个 URL，如输入浏览器就输入 baidu.com 回车，会自动跳转到www.baidu.com 就是通过这个响应头控制的 |
| Server            | 后台服务器                                                   |

>  Reference 
>  https://zhuanlan.zhihu.com/p/72616216
>  https://juejin.im/post/5a0ce1d95188253e24708454