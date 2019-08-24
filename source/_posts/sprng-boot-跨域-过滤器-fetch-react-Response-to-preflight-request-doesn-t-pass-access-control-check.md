---
title: >-
  sprng boot 跨域 过滤器 fetch react Response to preflight request doesn't pass
  access control check
date: 2019-06-19 11:20:11
tags: [Spring-Boot, 跨域, Java]
category: [Java, Spring-Boot]
cover: https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/cover/springboot.png?raw=true
---

## 前言
浏览器出于安全考虑，限制了JS发起跨站请求，使用XHR对象发起请求必须遵循同源策略（SOP：Same Origin Policy），跨站请求会被浏览器阻止，这对开发者来说是很痛苦的一件事，尤其是要开发前后端分离的应用时。

在现代化的Web开发中，不同网络环境下的资源数据共享越来越普遍，同源策略可以说是在一定程度上限制了Web API的发展。

简单的说，CORS就是为了请求能够安全跨域而生的。至于CORS的安全性研究，本文不做探讨。


## CORS浅述
名词解释：跨域资源共享（Cross-Origin Resource Sharing）

概念：是一种跨域机制、规范、标准，怎么叫都一样，但是这套标准是针对服务端的，而浏览器端只要支持HTML5即可。

作用：可以让服务端决定哪些请求源可以进来拿数据，所以服务端起主导作用（所以出了事找后台程序猿，无关前端^ ^）

常用场景：

 - 前后端完全分离的应用

## 服务端未允许跨域

![服务端未允许跨域](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/sprng%20boot%20%E8%B7%A8%E5%9F%9F%20%E8%BF%87%E6%BB%A4%E5%99%A8%20fetch%20react%20Response%20to%20preflight%20request%20doesn't%20pass%20access%20control%20check/20190311105909443.png?raw=true)

## 如何解决

```
package pers.yiji.YiJiClientServer.util;

import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {

    /**
     * cors support
     * @return
     */
    @Bean
    public FilterRegistrationBean corsFilter() {
        // 注册CORS过滤器
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true); // 是否支持安全证书
        config.addAllowedOrigin("*"); // 允许任何域名使用
        config.addAllowedHeader("*"); // 允许任何头
        config.addAllowedMethod("*"); // 允许任何方法（post、get等）
        // 预检请求的有效期，单位为秒。
        //        config.setMaxAge(3600L);

        source.registerCorsConfiguration("/**", config);
        FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
        bean.setOrder(0);
        return bean;
    }
}
```

具体每句话的意思基本上注释都有写。

主要就是为了注册一个过滤器，这里是基本上允许所有的请求，在特殊的场景可以使用域名控制等。

```
config.addAllowedOrigin("*"); // 允许任何域名使用（*可以换成特定的域名）
```


## 结果
![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/sprng%20boot%20%E8%B7%A8%E5%9F%9F%20%E8%BF%87%E6%BB%A4%E5%99%A8%20fetch%20react%20Response%20to%20preflight%20request%20doesn't%20pass%20access%20control%20check/20190311110721789.png?raw=true)

![在这里插入图片描述](https://github.com/DerrickTel/DerrickTel.github.io/blob/master/img/sprng%20boot%20%E8%B7%A8%E5%9F%9F%20%E8%BF%87%E6%BB%A4%E5%99%A8%20fetch%20react%20Response%20to%20preflight%20request%20doesn't%20pass%20access%20control%20check/20190311110735834.png?raw=true)
