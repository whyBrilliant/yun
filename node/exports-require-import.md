# export import exports export default

> 使用范围

```
1. require: node支持的引入

2. export/import: 只有es6支持的导出引入

3. module.exports/exports: 只有node支持的导出
```



## node模块

> node中的模块遵循CommonJS规范。
>
> CommonJS定义的模块分为：模块标识(module)、模块定义（exports）、模块引用(require)

当node执行一个文件时，会给这个文件内生成一个exports和module对象，而module又有一个exports属性。它们指向同一块内存区域{}

```
exports = module.exports = {}
```

eg:

```
// utils.js
const a = 100

console.log(module.exports) // {}
console.log(exports) // {}

exports.a = 200 // 这里将module.exports的内容改成 { a: 200 }
exports = '指向其它内存区域' // 这里将exports的指向改变

console.log(module.exports) // { a: 200 }
console.log(exports) // '指向其它内存区域'

// test.js
const a = require('./exports')
console.log(a) // { a: 200 }
```

可以看出，require导入的内容是module.exports的指向内存块的内容，并不是exports的。它们的区别是exports只是module.exports的引用，辅助后者添加内容的。

因此，为了避免混淆，尽量使用module.exports导出，用require导入



## ES中的模块导入导出

export 和 export default的区别

```
1. export 和 export default均可用于导出常量、函数、文件、模块，但export能直接导出变量表达式,但export default 不行

2. 在一个文件或模块中，export、import 可以有多个，但export default只能有一个

3. 通过export导出，在导入时需要加{}, export default则不需要
```

eg:

```
// utils
导出方式一：
// 导出变量
export const m = '100'
// 导出方法
export const dogSay = function () {
	console.log('wangwang')
}

导出方式二:
function catSay() {
	console.log('miaomiao')
}

export catSay // 错误
export { catSay }

// export default的导出
export default const n = '100' //错误
const n = '100'
export default n

// test.js
import { dogSay, catSay } from './utils'
import n from './utils'
import * as utilsModule from './utils'

dogSay() // wangwang
catSay() // miaomiao
console.log(n) // 100
utilsModule.dogSay() // wangwang
utilsModule.n // undefined
utilsModule.default // 100

```

因为as导出是把零散的export聚集在一起作为一个对象导出，而export default是导出为default的属性

