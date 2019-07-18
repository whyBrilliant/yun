# webpack 插件

## 1. CleanWebpackPlugin

CleanWebpackPlugin默认情况下，此插件将删除 webpack output.path目录中的所有文件，以及每次成功重建后所有未使用的 webpack 资产。

tips:

```
如果安装的 clean-webpack-plugin 是 3.0 版本的
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

如果安装的 clean-webpack-plugin 是 < 3.0 版本的
const CleanWebpackPlugin = require('clean-webpack-plugin')
```

## 2. splitChunksPlugins
业务代码更新会比较频繁，第三方代码基本不会更改,每次重新打包后，都会重新请求造成加载时间很长。由于浏览器的缓存机制，我们可以将第三方代码缓存起来，就不用去发送http请求，而是直接从缓存中取，这样可以在刷新页面或者第二次进入的时候可以加快网页的加载速度。

在 webpack4 之前是使用 commonsChunkPlugin 来拆分公共代码，v4 之后被废弃，并使用 splitChunksPlugins。splitChunksPlugins 是 webpack 主模块中的一个细分模块，无需 npm 引入