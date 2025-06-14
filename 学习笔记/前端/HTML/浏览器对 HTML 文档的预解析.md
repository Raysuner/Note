---
date: 2025-06-12
tags:
  - HTML预解析
---


## 定义

浏览器对 HTML 的解析生成 DOM 前，提前扫描整个 HTML 文档对 link、script、img 等元素的指向的网络文件资源，预先加载这些资源。

## 作用

减少资源加载延迟

## 原理 

浏览器主线程在获取到 HTML 文档后开始进行，开始将 HTML 解析成 DOM 节点，而浏览器的预解析线程只需要扫描整个 HTML 文档即可，即使主线程解析过程卡住了，预解析线程还是会把后面的资源进行下载。

## 场景验证

一起做下验证，可以发现即使 script 阻塞了两秒钟，图片也不会在执行 js 后两秒后才开始加载图片资源。

```html
<!-- 主解析器在解析脚本时，预解析器已提前请求 image1.jpg 和 image2.jpg -->
<script src="blocking-script.js"></script> <!-- 阻塞 2s -->
<img src="image1.jpg">  
<img src="image2.jpg">
```

## 注意

虽然 script 后的资源也会提前下载，但是主线程解析 HTML 的任务会被 js 任务阻塞，仍然会导致页面白屏时间过长。