## 定义

Script 标签是 HTML 中的一种标签

## 属性

### `async`

浏览器解析 html 遇到 async 属性的 script，浏览器会下载 src 指向的 js 资源，html 文档的解析也会继续直到资源下载完成，浏览器开始执行 js。

多个 async 属性的 script 不能保证执行顺序，看哪个资源先下载完成就先执行哪个

### `defer`

浏览器解析 html 遇到 async 属性的 script，浏览器会下载 src 指向的 js 资源，html 文档的解析也会继续直到资源下载完成，浏览器开始执行 js。

包含 `defer` 属性的脚本将阻塞 `DOMContentLoaded` 事件触发，直到脚本完成加载并执行。

对应 js 脚本的执行时机是在 html 解析完成后，`DOMContentLoaded` 事件执行前。

设置 defer 的脚本会按照顺序依次执行，这个和设置 async 的脚本不一样。


![async和defer的执行情况](http:8.149.242.20:9000/storage/uploads/202506/10/async-defer.jpg?ITf86B0B0l)

### `crossorigin`

Crossorigin 主要用于控制脚本资源的跨域行为。

它的核心作用主要体现在下面几个方面

- 跨域错误信息捕获：当请求跨域资源时，如果未设置 crossorigin，浏览器默认只会提示 <font style="color: red">script error </font> 的提示。但是设置 crossorigin 属性后，可以使用 `window.onerror` 来捕获错误信息。
- Cors 凭证模式控制： 可以选择在发起跨域请求是否携带 cookie

该属性目前有两个值，分别是 anonymous 和 use-credentials 这两个值，使用 use-credentials 时会携带凭证，设置 anonymous 则不会。

服务端需要配置设置如下：

```text
Access-Control-Allow-Origin: *               # 允许所有域
# 或
Access-Control-Allow-Origin: https://your-domain.com  # 允许特定域

Access-Control-Allow-Credentials: true       # 允许携带凭据（仅 use-credentials 时需要）
```

