## 需求
- 必须使用HTTP服务运行而不是使用文件形式进行预览
- 修改完代码过后，webpack能够自动重新打包，并且浏览器可以显示最新结果
- 需要提供Source Map支持
## 初始解决方案
使用--watch参数，但是浏览器不会使用最新内容
## 最终解决方案
使用webpack提供的dev server，它能够在代码更新时重新打包
