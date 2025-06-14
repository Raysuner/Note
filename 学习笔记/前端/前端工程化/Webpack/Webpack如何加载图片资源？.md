Webpack 在默认情况下是不认识图片文件的，需要在配置文件对图片文件进行处理后，Webpack才能正确处理和加载图片资源。
## Webpack4
Webpack4需要借助loader来处理图片资源，
### file-loader
file-loader会将图片复制到最终的打包目录中去，并返回URL。无论文件大小，file-loader都会复制生成一个新的图片，对于大文件的图片因为需要单独加载该图片，那么就可以利用浏览器的缓存机制，不会影响页面其它内容的加载。
其配置如下：
```js
// webpack.config.js
module.exports = {
  // ...
  module: {
    rules: [{
      test: /\.(png|jpg)$/,
      use: ['file-loader']
    }],
  },
};
```
### url-loader
url-loader可以设置一个阈值，文件大小小于阈值时，会为传入的内容生成一个DataURL。这意味着文件内容将被内联到bundle中，减少了HTTP请求的数量，这对小文件来说非常有用，但是对于大文件来说，如果大量的文件内容内嵌到bundle中，会导致请求的内容变大，增加请求的耗时时间。所以超过阈值的情况下将会使用file-loader。
```js
module.exports = {
  // ...
  module: {
    rules: [{
      test: /\.(png|jpg)$/,
      use: [{
        loader: 'url-loader',
        options: {
          limit: 1024,
          fallback: 'file-loader',
        }
      }]
    }],
  },
};
```
### raw-loader
raw-loader会将文件内容不经任何处理直接文件内容嵌入到bundle中，适用于处理svg这种图片
