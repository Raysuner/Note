## Babel

用于将新版本的`JavaScript`代码转为向后兼容的旧版本，也可以用来转换其他语言编写的代码，例如`Typescript`

```zsh
pnpm i -D @babel/core @babel/preset-env babel-loader @babel/preset-react
```

其中`@babel/preset-env`里面包含了大量的配置、插件以及`polyfill`。可以通过配置的目标环境来选取需要的转换插件来执行以及自动引入`polyfill`，确保在目标环境中能够正常运行。

## React

```zsh
pnpm i react react-dom
pnpm i -D @types/react @types/react-dom
```

## Typescript

1）使用`ts-loader`做转换以及类型检查

```zsh
pnpm i -D typescript ts-loader
```

2）使用`@babel/preset-typescript`做转换，类型检查还需要`ts-loader`来处理，并在`Typescript`相关的配置文件里把`noEmit`设置为`true`，这样`ts-loader`就不会产生`JavaScript`文件了。

```zsh
pnpm i -D @babel/preset-typescript
```

## Webpack

安装`webpack`及相关`loader`和插件

```zsh
pnpm install -D webpack webpack-cli sass sass-loader css-loader style-loader babel-loader html-webpack-plugin webpack-bundle-analyzer 
```

与src目录同层级新建public目录，并在public目录下创建index.html文件

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>react admin practice</title>
  </head>

  <body>
    <div id="root"></div>
  </body>
</html>
```

### webpack.config.js

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const BundleAnalyzerPlugin =
  require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

module.exports = {
  mode: "development",
  entry: {
    bundle: path.resolve(__dirname, "src", "main.tsx"),
  },
  output: {
    filename: "[name].[contenthash:8].js", // hash 防止浏览器缓存
    path: path.resolve(__dirname, "dist"), 
    clean: true,
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src"), // @ === src
    },
    extensions: [".tsx", ".ts", "..."], // 在引入这些类型的文件不需要携带后缀
  },
  externals: [{ lodash: "_" }, { React: "React" }, { "react-dom": "ReactDOM" }],
  devServer: {  // 本地服务器配置
    hot: true,
    compress: true,
    open: true,
  },
  module: {
    rules: [
      {
        test: /\.less$/,  // 支持less
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              modules: {
                localIdentName: "[path][name]__[local]--[hash:5]",
              },
            },
          },
          "less-loader", 
        ],
      },
      {
        test: /\.tsx?$/,  // 支持tsx
        exclude: /node_modules/,
        use: [
          {
            loader: "babel-loader",
            options: {
              presets: [
                "@babel/preset-env",
                "@babel/preset-typescript",
                "@babel/preset-react",
              ],
            },
          },
        ],
      },
    ],
  },
  plugins: [
    // 根据public里面的html文件为模板，产生页面html
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, "public", "index.html"),
    }),
  ],
};
```

## ESLint

因为`tslint`已经被放弃，现在解析`Typescript`也是`ESLint`，但是它原本只用来解析`JavaScript`的，那么需要插件来支持ts的解析

```zsh
pnpm i -D eslint eslint-plugin-react @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-plugin-react-hooks
```

`eslint`相关配置文件

```json
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": ["plugin:@typescript-eslint/recommended"]
}
```

在`webpack`构建过程也报出`eslint`的报错信息

```zsh
pnpm i -D eslint-webpack-plugin
```

## Stylelint

Stylelint，一个强大的现代化样式 Lint 工具，用来帮助你避免语法错误和统一代码风格。

```zsh
pnpm i -D stylelint stylelint-prettier stylelint-config-recess-order stylelint-config-standard stylelint-config-standard-scss
```

## Prettier

首先安装`prettier`

```zsh
pnpm install prettier -D
```

然后安装解决`prettier`与`eslint`冲突的插件

```zsh
pnpm install eslint-config-prettier -D
```

安装好之后需要再`eslint`配置文件里面解决冲突，把`prettier`放到后面，就可以覆盖掉与`eslint`冲突的规则。

```json
{
  extends: ["eslint:recommended", "prettier"],
}
```

然后安装`eslint-plugin-prettier`，让`eslint`在检查代码时，能够按照`prettier`的规则检查代码规范性，并进行修复

```zsh
pnpm install eslint-plugin-prettier -D
```

在`eslint`配置文件中增加配置

```json
{
	"rules":{
	    "prettier/prettier":"error"
	},
	"plugins": ["prettier"]，
}
```

## Husky

安装`husky`

```zsh
pnpm dlx husky-init && pnpm install
```

修改当前工作目录下`.husky`目录下的`pre-commit`

```zsh
- npm test
+ pnpm run lint
```

## Lint-Staged

之前`husky`会执行`pnpm run lint`，这会对仓库里面所有代码进行`lint`，即使文件内容完全没有改动，也会走一遍`lint`检查，当代码越来越多的时候，提交代码的过程会越来越慢，以至于影响开发进度。

使用`lint-staged`可以只对git暂存区里面的变动进行`lint`检查，把`lint`的执行动作交给`lint-staged`。

需要在`package.json`里面配置

```json
{
  "lint-staged": {
    "**/*.{js,jsx,tsx,ts}": [
      "pnpm run lint:script",
      "git add ."
    ],
    "**/*.{scss,sass,css}": [
      "pnpm run lint:style",
      "git add ."
    ]
  }
}
```

## commit信息规范校验

规范团队内的commit信息提交规范

```zsh
pnpm i commitlint @commitlint/cli @commitlint/config-conventional -D
```

创建对应配置文件`.commitlintrc.js`

```js
module.exports = {
  extends: ["@commitlint/config-conventional"]
};
```

触发`commitlint`校验`commit`信息

```zsh
npx husky add .husky/commit-msg "npx --no-install commitlint -e $HUSKY_GIT_PARAMS"
```