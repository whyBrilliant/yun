# webpack 插件

## 常用类

### 1. CleanWebpackPlugin

CleanWebpackPlugin默认情况下，此插件将删除 webpack output.path目录中的所有文件，以及每次成功重建后所有未使用的 webpack 资产。

tips:

```
如果安装的 clean-webpack-plugin 是 3.0 版本的
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

如果安装的 clean-webpack-plugin 是 < 3.0 版本的
const CleanWebpackPlugin = require('clean-webpack-plugin')
```

### 2. splitChunksPlugins
业务代码更新会比较频繁，第三方代码基本不会更改,每次重新打包后，都会重新请求造成加载时间很长。由于浏览器的缓存机制，我们可以将第三方代码缓存起来，就不用去发送http请求，而是直接从缓存中取，这样可以在刷新页面或者第二次进入的时候可以加快网页的加载速度。

在 webpack4 之前是使用 commonsChunkPlugin 来拆分公共代码，v4 之后被废弃，并使用 splitChunksPlugins。splitChunksPlugins 是 webpack 主模块中的一个细分模块，无需 npm 引入

webpack会根据下述条件自动进行代码块分割：

  - 新代码块可以被共享引用，或者这些模块都是来自node_modules文件夹里面
  - 新代码块大于30kb（min+gziped之前的体积）
  - 按需加载的代码块，并行请求最大数量应该小于或者等于5
  - 初始加载的代码块，并行请求最大数量应该小于或等于3


```
optimization: {
  splitChunks: {
    chunks: 'async',
    minSize: 30000,
    maxSize: 0,
    minChunks: 1,
    maxAsyncRequests: 5,
    maxInitialRequests: 3,
    automaticNameDelimiter: '~',
    name: true,
    cacheGroups: {
      vendors: {
        test: /[\\/]node_modules[\\/]/,
        priority: -10
      },
      default: {
        minChunks: 2,
        priority: -20,
        reuseExistingChunk: true
      }
    }
  }
}

```

参数

* chunks ['initial'(初始块), 'async'(按需加载块), 'all'(全部块)],默认为async
  1. async 情况下，异步导入的模块会被单独打包，
  2. initial, all模式会将所有来自node_modules的模块分配到一个叫vendors的缓存组；所有重复引用至少两次的代码，会被分配到default的缓存组。
  3. initial模式下会分开优化打包异步和非异步模块。而all会把异步和非异步同时进行优化打包。也就是说moduleA在indexA中异步引入，indexB中同步引入，initial下moduleA会出现在两个打包块中，而all只会出现一个。

  参考文章 []

* minSize: 表示在压缩前的最小模块大小，默认为30000kb

* minChunks: 表示被引用次数，默认为1

* maxAsyncRequests: 按需加载时候最大的并行请求数，默认为5

* maxInitialRequests: 一个入口最大的并行请求数，默认为3

* automaticNameDelimiter: 打包分隔符

* name: 拆分出来块的名字，默认由块名和hash值自动生成, 值类型：String和Function

  注意：当不同分割代码块被赋予相同名称时候，他们会被合并在一起。这个可以用于在：比如将那些多个入口/分割点的共享模块(vendor)合并在一起，不过不推荐这样做。这可能会导致加载额外的代码。

  如果赋予一个神奇的值true，webpack会基于代码块和缓存组的key自动选择一个名称。除此之外，可以使用字符串或者函数作为参数值。

  当一个名称匹配到相应的入口名称，这个入口会被移除。

### cacheGroups: 缓存组。

* 这里开始设置缓存的 chunks.缓存组的属性除上面所有属性外，还有test, priority, reuseExistingChunk。cacheGroups 会继承和覆盖splitChunks的配置项

  1. test: 用于控制哪些模块被这个缓存组匹配到, 值类型：RegExp、String和Function
  2. priority: 缓存组打包的先后优先级
  3. reuseExistingChunk: 如果当前代码块包含的模块已经有了，就不在产生一个新的代码块

*  默认模式会将所有来自node_modules的模块分配到一个叫vendors的缓存组；所有重复引用至少两次的代码，会被分配到default的缓存组

* 一个模块可以被分配到多个缓存组，优化策略会将模块分配至跟高优先级别（priority）的缓存组，或者会分配至可以形成更大体积代码块的组里。

* 可以通过optimization.splitChunks.cacheGroups.default: false禁用default缓存组。


#### Conditions
在满足下述所有条件时，那些从相同代码块和缓存组来的模块，会形成一个新的代码块（译注：比如，在满足条件下，一个vendoer可能会被分割成两个，以充分利用并行请求性能）。

有四个选项可以用于配置这些条件：

* minSize(默认是30000)：形成一个新代码块最小的体积

* minChunks（默认是1）：在分割之前，这个代码块最小应该被引用的次数（译注：保证代码块复用性，默认配置的策略是不需要多次引用也可以被分割）

* maxInitialRequests（默认是3）：一个入口最大的并行请求数
* maxAsyncRequests（默认是5）：按需加载时候最大的并行请求数。

### 3. html-webpack-plugin
打包完之后自动生成 HTML 文件，并自动引入打包后的 js 文件

```
new HtmlWebpackPlugin({
      // 打包输出HTML
      title: '自动生成 HTML',
      minify: {
        // 压缩 HTML 文件
        removeComments: true, // 移除 HTML 中的注释
        collapseWhitespace: true, // 删除空白符与换行符
        minifyCSS: true // 压缩内联 css
      },
      filename: 'index.html', // 生成后的文件名
      template: 'index.html', // 根据此模版生成 HTML 文件
      chunks: ['app'] // entry中的 app 入口才会被打包
    })
```

参数含义如下：
* title: 打包后生成 html 的 title
* filename：打包后的 html 文件名称
* template：模板文件（例子源码中根目录下的 index.html）
* chunks：和 entry 配置中相匹配，支持多页面、多入口,只有该配置的入口才会被打包
* minify：压缩选项

注意：由于使用了 title 选项，则需要在 template 选项对应的 html 的 title 中加入 <%= htmlWebpackPlugin.options.title %>

```
<head>
···
<title><%= htmlWebpackPlugin.options.title %></title>
···
</head>
```

### 4 mini-css-extract-plugin
单独把 CSS 文件分离出来，我们需要使用 mini-css-extract-plugin 插件.之前是使用 extract-text-webpack-plugin 插件，此插件与 webpack4 不太匹配，现在使用 mini-css-extract-plugin。目前还不支持热更新，也就是在开发环境下更改了 css，需要手动的刷新页面才会看到效果，目前这个插件一般在生产环境中使用，开发环境下还是使用 'style-loader'

```
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/, // 针对 .css 后缀的文件设置 loader
        use: [
          {
            loader: MiniCssExtractPlugin.loader
          },
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css',
      chunkFilename: '[id].css'
    })
  ]
}
```
### 5 optimize-css-assets-webpack-plugin
压缩 css文件

```
new OptimizeCssAssetsPlugin({
  assetNameRegExp: /\.css$/g,
  cssProcessor: require('cssnano'), //用于优化\最小化 CSS 的 CSS处理器，默认为 cssnano
  cssProcessorOptions: { safe: true, discardComments: { removeAll: true } }, //传递给 cssProcessor 的选项，默认为{}
  canPrint: true //布尔值，指示插件是否可以将消息打印到控制台，默认为 true
  })
```

### 6 postcss-loader autoprefixer
有两种方式来配置 postcss，第一种是直接写在 webpack.config.js 中

```
rules: [
    {
      test: /\.(sa|sc|c)ss$/, // 针对 .sass .scss 或者 .css 后缀的文件设置 loader
      use: [
        {
          loader: MiniCssExtractPlugin.loader
        },
        'css-loader',
        // 使用 postcss 为 css 加上浏览器前缀
        {
          loader: 'postcss-loader',
          options: {
            plugins: [require('autoprefixer')]
          }
        },
        'sass-loader' // 使用 sass-loader 将 scss 转为 css
      ]
    }
  ]

```

第二种方式，在 webpack.config.js 同级目录下，新建 postcss.config.js 配置文件

```
// postcss.config.js
module.exports = {
  plugins: [require('autoprefixer')]
}

// webpack.config.js
rules: [
    {
      test: /\.(sa|sc|c)ss$/, // 针对 .sass .scss 或者 .css 后缀的文件设置 loader
      use: [
        {
          loader: MiniCssExtractPlugin.loader
        },
        'css-loader',
        'postcss-loader', // 使用 postcss 为 css 加上浏览器前缀
        'sass-loader' // 使用 sass-loader 将 scss 转为 css
      ]
    }
  ]
```

补充：在 css-loader 中使用 importLoaders 属性
importLoaders 表示：在一个 css 中引入了另一个 css，也会执行之前两个 loader，即 postcss-loader 和 sass-loader

```
rules: [
    {
      test: /\.(sa|sc|c)ss$/, // 针对 .sass .scss 或者 .css 后缀的文件设置 loader
      use: [
        {
          loader: MiniCssExtractPlugin.loader
        },
        {
          loader: css - loader,
          options: {
            importLoaders: 2
          }
        },
        'postcss-loader', // 使用 postcss 为 css 加上浏览器前缀
        'sass-loader' // 使用 sass-loader 将 scss 转为 css
      ]
    }
  ]
```

### 7 NamedModulesPlugin

```
  plugins: [
    ...
    new webpack.HotModuleReplacementPlugin(), // 热部署模块
    new webpack.NamedChunksPlugin(), // 热加载时直接返回更新文件名，而不是文件的id。
    ...
  ]
```

路由文件中需要这样定义
```
// request 代表变量filePath
const lazyLoad = filePath => lazy(
  () => {
    return new Promise(resolve => setTimeout(resolve, 100))
    .then(() => import(/* webpackChunkName: "[request]" */ `pages/${filePath}`))
    .catch(() => import('pages/Error'))
  }
)
```

### 8 DefinePlugin 定义全部变量
```
// 此处定义全局变量__API__BASE__
plugins: [
  new webpack.DefinePlugin({ __API__BASE__: JSON.stringify(config.apiBase) }),
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NamedChunksPlugin(),
  ]
```


## 非常用类

### 1. webpack-bundle-analyzer
打包结果分析工作

```
1. npm i webpack-bundle-analyzer -D

2. 定义
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

3. 然后在plugins数组中添加
new BundleAnalyzerPlugin()
```

