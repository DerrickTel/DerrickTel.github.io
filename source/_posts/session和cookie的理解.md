---
title: session和cookie的理解
date: 2020-04-11 10:31:35
tags: [HTTP]
categories: [HTTP]
cover: /image/cover/http.png

---


## 曾经的我

cookie是浏览器中传输自带的; balabala...

session是存在服务端的; 存一些用户信息.......balabala....

## 现在的我

从以下几个角度出发

### 一. http协议是无状态的;

当我们开发一些对状态有要求的接口时, cookie和session就可以弥补这一块的不足

- 对于http协议, cookie只是请求头中的一个字段, 和别的字段并没有特别大的差别;
- 浏览器对cookie做了默认的支持, 但是也限制了cookie; 比如同源策略;
  - 什么是同源策略?
    - 同源策略就是域名, 端口, 协议; 必须都相同才可以访问cookie的内容
  - 为什么要做同源策略?
    - 同源策略是浏览器基于安全的角度的一个机制, 限制了只有同域才可以访问cookie的内容

### 二.当我们要做单点登录sso功能的时候

- 什么是单点登录和sso?
  - 单点登录（Single Sign On），简称为 SSO，是比较流行的企业业务整合的解决方案之一。
  - 最简单的例子就是, 我们是打开淘宝网的时候, 我们打开一个商品详情页有可能是是重新打开一个页面, 那我们刚刚登录的信息可能就消失了, 要是每次打开一个页面都需要登录那样会非常的麻烦
    - 这样时候, 就需要cookie的帮助了;我们可以考虑吧域名种在可以访问的域名下, 通常都是二级域名
      - 什么是一级, 二级, 三级....域名?
      - 举例: www.taobao.com
        - 一级域名是指com(又称顶级域名; [维基百科, 点我!!](https://zh.wikipedia.org/wiki/域名)
        - 二级域名就是taobao
        - 所以三级域名就是www
    - 这样我们在淘宝这个域名底下的所有页面都可以畅通无阻
    - 但是也会面临信息泄露的危险
      - 虽然可以用时效来限制但是效果也不是很好
      - **Secure Cookie机制**
        - 设置了cookie只能在https上面传输不能在http上传输
        - 但是也不是万无一失, 因为还是可以在客户端, 进行读写的;
      - **HTTPOnly属性**
        - Cookie的HttpOnly属性，指浏览器不要在除HTTP（和 HTTPS)请求之外暴露Cookie。
        - 这样可以阻止非http的攻击, 如JavaScript
      - **Same-Site属性**
        - Cookie 的`SameSite`属性用来限制第三方 Cookie，从而减少安全风险。
          - 主要是为了限制CSRF攻击
            - 什么是CSRF攻击?
              - 跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并运行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。
            - 还可以用于跟踪用户信息, 比如在你的网站内部放入一个你看不见的
            - `<img src="facebook.com" style="visibility:hidden;">`
            - 这样就可以知道你访问了那些网站做一些相应的推荐
            - 那么有些同学就会联想到; 比如我在拼多多看了iPhone11; 再去看朋友圈很可能有就相应的推荐, 这些我个人猜测可能是pdd直接把你的信息卖给了腾讯, 或者做了交易(逃
          - 具体的三个值Strict; Lax; None;感兴趣的同学可以自己去查这里就不展开了
      - cookie又分为本地cookie和内存cookie
        - 本地cookie与内存cookie，区别在于cookie设置的expires字段。如果没有设置过期时间，就是内存cookie。随着浏览器的关闭而从内存中消失。
        - 还是一样都会泄露用户信息的风险
        - 哪怕是内存cookie攻击者可以设置时效使其成为本地cookie

### 三.session

session是服务器为web用户独立开辟的一个空间, 里面可以有用户一些信息等等

- 如果是一个服务器可能还好, 但是如果是多个服务器或者说多层转发的话就会引发一个问题, session命中问题; 所以我们需要把信息存在MySQL或者Redis里面

### 四.token

除了session和cookie来辅助http这个无状态请求, 还有什么办法? 

- token
  - token分为很多种, 常见的有JWT, sessionId, mac地址等等
  - token可以存储在很多地方, 比如本地的localStorage或者sessionStorage, 然后在请求头中携带

### 五.sessionStorage

刚刚说到sessionStorage; 下面来说说我碰到的一个问题

- sessionStorage如果打开一个新标签页, 他的sessionStorage是否共享?
- 大家先想想再看答案

以前的我以为是可以共享的.其实是半错半对的

为什么这么说?

MDN是这么说的

> ...data stored in sessionStorage gets cleared when the page session ends...**Opening a page in a new tab or window will cause a new session to be initiated**, which differs from how session cookies work.

大家可以做一个实验

>1. 在浏览器中打开这个 index.html，我们称之为标签页 A。注意：需要用 http 协议打开！例如 http://localhost/index.html
>2. 点击页面上的链接，此时会弹出来标签页 B。
>3. 在标签页 B 中打开控制台并执行 `sessionStorage.getItem('j')`，得到 `'s'`
>4. 新建一个新标签页 D，然后在地址栏内输入 http://localhost/index.html 打开同样的页面， 然后执行 `sessionStorage.getItem('j')` 。

按照我的预期，标签页 D 得到的应该还是 `'s'`，毕竟我认为 sessionStorage 的数据是在同一网站的多个标签页之间共享的。但是**我错了**，得到的结果是 `null`。

细心的同学可能已经发现了

细心的同学可能已经发现了，标签页 B 和标签页 D 之间唯一的不同就是它们被打开的方式：**标签页 B 是通过在标签页 A 中点击链接打开的，但标签页 D 是在浏览器地址栏输入地址打开的。**

所以现在我明白了：通过点击链接（或者用了 `window.open`）打开的新标签页之间是属于同一个 session 的，但新开一个标签页总是会初始化一个新的 session，即使网站是一样的，它们也不属于同一个 session。

