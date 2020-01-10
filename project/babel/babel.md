# babel

>简单来说把 JavaScript 中 es2015/2016/2017/20... 的新语法转化为 es5，让低端运行环境(如浏览器和 node )能够认识并执行。

## preset(预置条件)

1. stage-x, 这里面包含的都是当年最新规范的草案，每年更新。
  
    还可以细分为：

      - Stage 0 - 稻草人: 只是一个想法，经过 TC39 成员提出即可。
      - Stage 1 - 提案: 初步尝试。
      - Stage 2 - 初稿: 完成初步规范。
      - Stage 3 - 候选: 完成规范和浏览器初步实现。
      - Stage 4 - 完成: 将被添加到下一年度发布。

      低一级的 stage 会包含所有高级 stage 的内容，例如 stage-1 会包含 stage-2, stage-3 的所有内容。stage-4 在下一年更新会直接放到 env 中，所以没有单独的 stage-4 可供使用。

2. 执行顺序

  - Plugin 会运行在 Preset 之前。
  - Plugin 会从前到后顺序执行。
  - Preset 的顺序则 刚好相反(从后向前)

```
.babelrc 文件，presets中会先调用 "stage-0"，plugins中会先调用 "transform-es2015-arrow-functions"

{
  "presets": [
    "es2015",
    "react",
    "stage-0"
  ],
  "plugins": [
    "transform-es2015-arrow-functions", //转译箭头函数
    "transform-es2015-classes", //转译class语法
    "transform-es2015-spread", //转译数组解构
    "transform-es2015-for-of" //转译for-of
]
}
```


```
//如果要为某个插件添加配置项，按如下写法：
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage", // polyfills are automatically imported when needed
        "corejs": 3
      }
    ]
  ],
  "plugins":[
      //改为数组，第二个元素为配置项
      ["transform-es2015-arrow-functions", { "spec": true }]
  ]
}

```



3. 配置项

>插件和 preset 只要列出字符串格式的名字即可。但如果某个 preset 或者插件需要一些配置项(或者说参数)，就需要把自己先变成数组。第一个元素依然是字符串，表示自己的名字；第二个元素是一个对象，即配置对象。
```
"presets": [
    // 带了配置项，自己变成数组
    [
        // 第一个元素依然是名字
        "env",
        // 第二个元素是对象，列出配置项
        {
          "module": false
        }
    ],

    // 不带配置项，直接列出名字
    "stage-2"
]
```

插件得短名称

3.1 如果插件名称为 `@babel/plugin-XXX`，可以使用短名称`@babel/XXX`

```
{
    "plugins": [
        "@babel/transform-arrow-functions" //同 "@babel/plugin-transform-arrow-functions"
    ]
}
```

3.2 如果插件名称为 `babel-plugin-XXX`，可以使用端名称 `XXX`，该规则同样适用于带有 `scope` 的插件

```
{
	"plugins": [
		"newPlugin", // 同"babel-plugin-newPlugin"
		"@scp/muPlugin" //同 "@scp/babel-plugin-muPlugin"
	]
}
```



4. env 

env的核心目的是通过配置得知目标环境的特点，然后只做必要的转换。例如目标浏览器支持 es2015，那么 es2015 这个 preset 其实是不需要的，于是代码就可以小一点(一般转化后的代码总是更长)，构建时间也可以缩短一些。

如果不写任何配置项，env 等价于 latest，也等价于 es2015 + es2016 + es2017 三个相加(不包含 stage-x 中的插件)

```
{
  "presets": [
    ["env", {
      "targets": {
        "browsers": ["last 2 versions", "safari >= 7"]
      }
    }]
  ]
}
```
如上配置将考虑所有浏览器的最新2个版本(safari大于等于7.0的版本)的特性，将必要的代码进行转换。而这些版本已有的功能就不进行转化了。这里的语法可以参考 browserslist.

```
{
  "presets": [
    ["env", {
      "targets": {
        "node": "6.10"
      }
    }]
  ]
}

```
如上配置将目标设置为 nodejs，并且支持 6.10 及以上的版本。也可以使用 node: 'current' 来支持最新稳定版本。例如箭头函数在 nodejs 6 及以上将不被转化，但如果是 nodejs 0.12 就会被转化了。


## babel 工具

1. babel-cli

   命令行工具, 安装了 babel-cli 就能够在命令行中使用 babel 命令来编译文件.这样可以查看编译后得文件

   ```
   // package.json
   "scripts": {
       "compiler": "babel src --out-dir lib --watch"
   }
   ```

   

2. babel-node

    babel-node 是 babel-cli 的一部分，它不需要单独安装。它的作用是在 node 环境中，直接运行 es2015 的代码，而不需要额外进行转码。例如我们有一个 js 文件以 es2015 的语法进行编写(如使用了箭头函数)。我们可以直接使用 babel-node es2015.js 进行执行，而不用再进行转码了。

    babel-node = babel-polyfill + babel-register。


3. babel-register

    babel-register 模块改写 require 命令，为它加上一个钩子。此后，每当使用 require 加载 .js、.jsx、.es 和 .es6 后缀名的文件，就会先用 babel 进行转码。

    使用时，必须首先加载 require('babel-register')。babel-register 只会对 require 命令加载的文件转码，而 不会对当前文件转码。

    另外，由于它是实时转码，所以 只适合在开发环境使用。


4. babel-polyfill

    babel 默认只转换 js 语法，而不转换新的 API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法(比如 Object.assign)都不会转码。

    举例来说，es2015 在 Array 对象上新增了 Array.from 方法。babel 就不会转码这个方法。如果想让这个方法运行，必须使用 babel-polyfill。(内部集成了 core-js 和 regenerator)

    使用时，在所有代码运行之前增加 require('babel-polyfill')。或者更常规的操作是在 webpack.config.js 中将 babel-polyfill 作为第一个 entry。因此必须把 babel-polyfill 作为 dependencies 而不是 devDependencies

    babel-polyfill 主要有两个缺点：
    
    ```
    - 使用 babel-polyfill 会导致打出来的包非常大，因为 babel-polyfill 是一个整体，把所有方法都加到原型链上。比如我们只使用了 Array.from，但它把 Object.defineProperty 也给加上了，这就是一种浪费了。这个问题可以通过单独使用 core-js 的某个类库来解决，core-js 都是分开的。
    ```
    
    ```
    - babel-polyfill 会污染全局变量，给很多类的原型链上都作了修改，如果我们开发的也是一个类库供其他开发者使用，这种情况就会变得非常不可控。
    ```
    
    ```
    因此在实际使用中，如果我们无法忍受这两个缺点(尤其是第二个)，通常我们会倾向于使用 babel-plugin-transform-runtime。
    
    但如果代码中包含高版本 js 中类型的实例方法 (例如 [1,2,3].includes(1))，这还是要使用 polyfill。
    ```
    
    

5. babel-runtime 和 babel-plugin-transform-runtime

    我们时常在项目中看到 .babelrc 中使用 babel-plugin-transform-runtime，而 package.json 中的 dependencies (注意不是 devDependencies) 又包含了 babel-runtime，那这两个是不是成套使用的呢？他们又起什么作用呢？
    先说 babel-plugin-transform-runtime。
    babel 会转换 js 语法，之前已经提过了。以 async/await 举例，如果不使用这个 plugin (即默认情况)，转换后的代码大概是：

    ```
    // babel 添加一个方法，把 async 转化为 generator
    function _asyncToGenerator(fn) { return function () {....}} // 很长很长一段

    // 具体使用处
    var _ref = _asyncToGenerator(function* (arg1, arg2) {
      yield (0, something)(arg1, arg2);
    });

    ```

    不用过于纠结具体的语法，只需看到，这个 _asyncToGenerator 在当前文件被定义，然后被使用了，以替换源代码的 await。但每个被转化的文件都会插入一段 _asyncToGenerator 这就导致重复和浪费了。
    在使用了 babel-plugin-transform-runtime 了之后，转化后的代码会变成

    ```
    // 从直接定义改为引用，这样就不会重复定义了。
    var _asyncToGenerator2 = require('babel-runtime/helpers/asyncToGenerator');
    var _asyncToGenerator3 = _interopRequireDefault(_asyncToGenerator2);

    // 具体使用处是一样的
    var _ref = _asyncToGenerator3(function* (arg1, arg2) {
      yield (0, something)(arg1, arg2);
    });

    ```

    从定义方法改成引用，那重复定义就变成了重复引用，就不存在代码重复的问题了。**节省代码，减小体积。**

    但在这里，我们也发现 babel-runtime 出场了，它就是这些方法的集合处，也因此，在使用 babel-plugin-transform-runtime 的时候必须把 babel-runtime 当做依赖。
    再说 babel-runtime，它内部集成了:
    
    ```
    core-js: 转换一些内置类 (Promise, Symbols等等) 和静态方法 (Array.from 等)。绝大部分转换是这里做的。自动引入。
    ```
    
    ```
    regenerator: 作为 core-js 的拾遗补漏，主要是 generator/yield 和 async/await 两组的支持。当代码中有使用 generators/async 时自动引入。
    ```
    
    ```
    helpers, 如上面的 asyncToGenerator 就是其中之一，其他还有如 jsx, classCallCheck 等等，可以查看 babel-helpers。在代码中有内置的 helpers 使用时(如上面的第一段代码)移除定义，并插入引用(于是就变成了第二段代码)。
    ```
    
    ```
    babel-plugin-transform-runtime 不支持 实例方法 (例如 [1,2,3].includes(1))
    
    此外补充一点，把 helpers 抽离并统一起来，避免重复代码的工作还有一个 plugin 也能做，叫做 babel-plugin-external-helpers。但因为我们使用的 transform-runtime 已经包含了这个功能，因此不必重复使用。而且 babel 的作者们也已经开始讨论这两个插件过于类似，正在讨论在 babel 7 中把 external-helpers 删除.
    ```
    
    

6. babel-loader

    前面提过 babel 的三种使用方法，并且已经介绍过了 babel-cli。但一些大型的项目都会有构建工具 (如 webpack 或 rollup) 来进行代码构建和压缩 (uglify)。理论上来说，我们也可以对压缩后的代码进行 babel 处理，但那会非常慢。因此如果在 uglify 之前就加入 babel 处理，岂不完美？

    所以就有了 babel 插入到构建工具内部这样的需求。以(我还算熟悉的) webpack 为例，webpack 有 loader 的概念，因此就出现了 babel-loader。和 babel-cli 一样，babel-loader 也会读取 .babelrc 或者 package.json 中的 babel 段作为自己的配置，之后的内核处理也是相同。唯一比 babel-cli 复杂的是，它需要和 webpack 交互，因此需要在 webpack 这边进行配置。比较常见的如下：

    ```
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /(node_modules|bower_components)/,
          loader: 'babel-loader',
          options: {
           // 在这里传入 babel 的配置项, 这里的配置项优先级是最高的
          }
        }
      ]
    }

    ```


7. npm package 名称的变化 

    babel 7 的一个重大变化，把所有 babel-* 重命名为 @babel/*，例如：
    babel-preset-env 变成了 @babel/preset-env。进一步，还可以省略 preset 而简写为 @babel/env

    这个变化不单单应用于 package.json 的依赖中，包括 .babelrc 的配置 (plugins, presets) 也要这么写.



​	 ⭐ **注:** 从 Babel v7 开始，所以针对标准提案阶段的功能所编写的预设(stage preset)都已被弃用，官方已经移除了 `@babel/preset-stage-x`



 8. @babel/preset-env

    `@babel/preset-env` 主要作用是对我们所使用的并且目标浏览器中缺失的功能进行代码转换和加载 `polyfill`，在不进行任何配置的情况下，`@babel/preset-env` 所包含的插件将支持所有最新的JS特性(ES2015,ES2016等，不包含 stage 阶段)，将其转换成ES5代码。

    

    这样就解决了我们需要一个个配置插件非常繁琐得问题，我们只需要配置特别的插件即可。

    

    例如，如果你的代码中使用了可选链(目前，仍在 stage 阶段)，那么只配置 `@babel/preset-env`，转换时会抛出错误，需要另外安装相应的插件。

    

    `@babel/preset-env` 会根据你配置的目标环境，生成插件列表来编译。对于基于浏览器或 `Electron` 的项目，官方推荐使用 `.browserslistrc` 文件来指定目标环境。默认情况下，如果你没有在 `Babel` 配置文件中(如 .babelrc)设置 `targets` 或 `ignoreBrowserslistConfig`，`@babel/preset-env` 会使用 `browserslist` 配置源。如果你不是要兼容所有的浏览器和环境，推荐你指定目标环境，这样你的编译代码能够保持最小。

    ```
    //.browserslistrc
    > 0.25%
    not dead
    ```

    

    例如，仅包括浏览器市场份额超过0.25％的用户所需的 `polyfill` 和代码转换（忽略没有安全更新的浏览器，如 IE10 和 BlackBerry）

    ```
    last 2 Chrome versions
    ```

    然后再执行 `npm run compiler`，你会发现箭头函数不会被编译成ES5，因为 `chrome` 的最新2个版本都能够支持箭头函数。现在，我们将 `.browserslistrc` 仍然换成之前的配置。

    这个编译出来的代码在低版本浏览器中使用的话，显然是有问题的，因为低版本浏览器中数组实例上没有 `includes` 方法，也没有 `Promise` 构造函数。

    因为语法转换只是将高版本的语法转换成低版本的，但是新的内置函数、实例方法无法转换。这时，就需要使用 `polyfill` 上场了，顾名思义，`polyfill`的中文意思是垫片，所谓垫片就是垫平不同浏览器或者不同环境下的差异，让新的内置函数、实例方法等在低版本浏览器中也可以使用。

	9.  Polyfill

    `@babel/polyfill` 模块包括 `core-js` 和一个自定义的 `regenerator runtime` 模块，可以模拟完整的 ES2015+ 环境（不包含第4阶段前的提议）。

    这意味着可以使用诸如 `Promise` 和 `WeakMap` 之类的新的内置组件、 `Array.from` 或 `Object.assign` 之类的静态方法、`Array.prototype.includes` 之类的实例方法以及生成器函数(前提是使用了 `@babel/plugin-transform-regenerator` 插件)。为了添加这些功能，`polyfill` 将添加到全局范围和类似 `String` 这样的内置原型中

    ```
    npm install --save @babel/polyfill
    ```

    注意：不使用 `--save-dev`，因为这是一个需要在源码之前运行的垫片。`@babel/polyfill` 需要在其它代码之前引入，或者我们也可以在 `webpack` 中进行配置。

    ```
    //入口文件app.js
    import '@babel/polyfill'
    
    // webpack
    entry: [    require.resolve('./polyfills'),    path.resolve('./index')]
    ```

    我们的代码不管在低版本还是高版本浏览器(或node环境)中都能正常运行了。不过，很多时候，我们未必需要完整的 `@babel/polyfill`，这会导致我们最终构建出的包的体积增大，`@babel/polyfill`的包大小为89K.

    我们更期望的是，如果我使用了某个新特性，再引入对应的 `polyfill`，避免引入无用的代码。值得庆幸的是， `Babel` 已经考虑到了这一点。

    `@babel/preset-env` 提供了一个 `useBuiltIns` 参数，设置值为 `usage` 时，就只会包含代码需要的 `polyfill` 。有一点需要注意：配置此参数的值为 `usage` ，必须要同时设置 `corejs` (如果不设置，会给出警告，默认使用的是"corejs": 2) ，注意: 这里仍然需要安装 `@babel/polyfill`(当前 `@babel/polyfill` 版本默认会安装 "corejs": 2):

    首先说一下使用 `core-js@3` 的原因，`core-js@2` 分支中已经不会再添加新特性，新特性都会添加到 `core-js@3`。例如你使用了 `Array.prototype.flat()`，如果你使用的是 `core-js@2`，那么其不包含此新特性。为了可以使用更多的新特性，建议大家使用 `core-js@3`

    ⭐注意tree-shaking
    
    tree-shaking的主要作用是用来清除代码中无用的部分。目前在webpack4，我们设置mode为production时已自动开启了tree-shaking，但是要想其生效，生成的代码必须是ES6模块，不能使用其它类型的模块比如CommonJS之流。如果使用Babel，因为Babel的预案preset默认会将任何模块类型都转译成CommonJS类型，这会导致tree-shaking失效。修正这个问题，需要添加module属性
    
    ```
    npm install --save core-js@3
    
    //.babelrc
    const presets = [
    	[
    		"@babel/env"，
    		{
    			“useBuiltIns": "usage",
    			"corejs": 3,
			"modules": false
    		}
	]
    ]
    ```
    
    `Babel` 会检查所有代码，以便查找在目标环境中缺失的功能，然后仅仅把需要的 `polyfill` 包含进来
    
    ```
    const isHas = [1,2,3].includes(2);
    const p = new Promise((resolve, reject) => {    
    	resolve(100);
    });
    
    // 编译后
    "use strict";
    require("core-js/modules/es.array.includes");
    require("core-js/modules/es.object.to-string");
    require("core-js/modules/es.promise");
var isHas = [1, 2, 3].includes(2);
    var p = new Promise(function (resolve, reject) {    
	resolve(100);
    });
```
    
同样的代码，我们用 `webpack` 构建一下(`production` 模式)，能看到最终的代码体积会变小。
    

    
在 `useBuiltIns` 参数值为 `usage` 时，仍然需要安装 `@babel/polyfill`，虽然我们上面的代码转换中看起来并没有使用到，但是，如果我们源码中使用到了 `async/await`，那么编译出来的代码需要 `require("regenerator-runtime/runtime")`，在 `@babel/polyfill` 的依赖中，当然啦，你也可以只安装 `regenerator-runtime/runtime` 取代安装 `@babel/polyfill`
    
    
    
    `Babel` 会使用很小的辅助函数来实现类似 `_createClass` 等公共方法。默认情况下，它将被添加(`inject`)到需要它的**每个**文件中。如果你有10个文件中都使用了这个 `class`，是不是意味着 `_classCallCheck`、`_defineProperties`、`_createClass` 这些方法被 `inject` 了10次。这显然会导致包体积增大，最关键的是，我们并不需要它 `inject` 多次。
    
    ```
    就是 `@babel/plugin-transform-runtime` 插件大显身手的时候了，使用 `@babel/plugin-transform-runtime` 插件，所有帮助程序都将引用模块 `@babel/runtime`，这样就可以避免编译后的代码中出现重复的帮助程序，有效减少包体积。
    ```
    
    

10. `@babel/plugin-transform-runtime` 是一个可以重复使用 `Babel` 注入的帮助程序，以节省代码大小的插件。`@babel/plugin-transform-runtime` 需要和 `@babel/runtime` 配合使用。

    首先安装依赖，`@babel/plugin-transform-runtime` 通常仅在开发时使用，但是运行时最终代码需要依赖 `@babel/runtime`，所以 `@babel/runtime` 必须要作为生产依赖被安装

    ```
    npm install --save-dev @babel/plugin-transform-runtime
    npm install --save @babel/runtime
    ```

    除了前文所说的，`@babel/plugin-transform-runtime` 可以减少编译后代码的体积外，我们使用它还有一个好处，它可以为代码创建一个沙盒环境，如果使用 `@babel/polyfill` 及其提供的内置程序（例如 `Promise` ，`Set` 和 `Map` ），则它们将污染全局范围。虽然这对于应用程序或命令行工具可能是可以的，但是如果你的代码是要发布供他人使用的库，或者无法完全控制代码运行的环境，则将成为一个问题。

    
    
    @babel/plugin-transform-runtime` 会将这些内置别名作为 `core-js` 的别名，因此您可以无缝使用它们，而无需 `polyfill

    ```
    //.babelrc
    {
    	"presets": [
    		[
                "@babel/preset-env",
                {
                    "useBuiltIns": "usage",
                    "corejs": 3
                }
             ]
    	],
    	"plugins":[
    		[
    			"@babel/plugin-transform-runtime"
    		]
    	]
    }
    ```
    
    
    
    如果我们希望 `@babel/plugin-transform-runtime` 不仅仅处理帮助函数，同时也能加载 `polyfill` 的话，我们需要给 `@babel/plugin-transform-runtime` 增加配置信息。
    
    首先新增依赖 `@babel/runtime-corejs3`
    
    ```
    npm install @babel/runtime-corejs3 --save
    ```
    
    修改配置文件如下(移除了 `@babel/preset-env` 的 `useBuiltIns` 的配置，不然不就重复了.
    
    ```
    {
    	"presets": [
    		[
    			"@babel/preset-env"
    		]
    	],
    	"plugins": [
    		[
    			"@babel/plugin-transform-runtime": {
    				"corejs": 3
    			}
    		]
    	]
    }
    ```
    
    给 `@babel/plugin-transform-runtime` 配置 `corejs` 可以将帮助函数变成引用的形式，又可以动态引入 `polyfill`，并且不会污染全局环境。给 `@babel/preset-env` 提供 `useBuiltIns` 还有必要嘛？结果是`@babel/runtime-corejs3/XXX` 的包本身比 `core-js/modules/XXX` 要大一些，打包结果会大一些吧
    
    
    
    参考文章：
    
    [不容错过的 Babel7 知识](https://juejin.im/post/5ddff3abe51d4502d56bd143)
    
    
    
    
    
    
    
    