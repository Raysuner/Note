---
date: 2025-05-27
tags:
  - preconnect
  - prefetch
  - preload
  - 性能优化
---
## 定义

preconnect、prefetch 以及 preload 这三者都是 HTML 中 link 标签 rel 属性的值，它们常
用于前端页面性能优化场景，通过提前加载资源或者提前建立连接的方式来优化页面性能。

## 属性介绍

#### preconnect

preconnect 属性用于提前在客户端与指定网址之间建立 dns 解析、tcp 三次握手、tls 协商。

一般用于提前与 cdn 地址和接口地址建立连接

```html
<link rel="preconnect" href="xxx.com" />
```

例如阿里巴巴网站首页，就有通过 preconnect 提前与 cdn 网址建立连接。

![Pasted image 20250611222233.png](http:8.149.242.20:9000/storage/uploads/202506/11/Pasted%20image%2020250611222233.png?srg4d6aDwc)

当页面需要访问该网址下的资源时，就无需再建立连接，而是使用之前建立的连接，通过提前与关键网址建立连接，可以提高网站首屏性能。

需要注意的是，过度使用 preconnect 不仅不会提高页面性能，可能反而会起到副作用，建议只对关键域名进行提前建联。

#### preload

preload 属性强制浏览器对指定的资源进行预加载，而不是按照默认优先级来加载。一般用于加载图片、字体、核心 js 、核心 css。

```html
<link rel="preload" href="fcp.png" as="image" />
<link rel="preload" href="font.woff2" as="font" type="font/woff2" />
```

通过提升资源下载优先级，从而更早的加载资源，能够让资源更快体现在页面上。

#### prefetch

prefetch 属性用于提前加载下一条页面的资源，当当前前面资源全部加载完成且网络处于空闲状态时，prefetch 指定的资源会被加载。

我们要想起到性能优化的目的，可以配合强缓存来实现。当跳转到下一跳页面再次加载该资源时，直接从缓存里面加载该资源，而不需要重新发送请求。



