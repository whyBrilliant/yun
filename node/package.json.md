# package.json

eg: antd

```
{
  "name": "antd",
  "version": "4.1.5",
  "description": "An enterprise-class UI design language and React components implementation",
  "keywords": [
    "ant",
	...
  ],
  "homepage": "http://ant.design/",
  "bugs": {
    "url": "https://github.com/ant-design/ant-design/issues"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/ant-design/ant-design"
  },
  "license": "MIT",
  "contributors": [
    "ant"
  ],
  "funding": {
    "type": "opencollective",
    "url": "https://opencollective.com/ant-design"
  },
  "files": [
    "dist",
    "lib",
    "es"
  ],
  "sideEffects": [
    "dist/*",
    "es/**/style/*",
    "lib/**/style/*",
    "*.less"
  ],
  "main": "lib/index.js",
  "module": "es/index.js",
  "unpkg": "dist/antd.min.js",
  "typings": "lib/index.d.ts",
  "scripts": {
    "build": "npm run compile && npm run dist",
    ...
  },
  "husky": {
    "hooks": {
      "pre-commit": "pretty-quick --staged"
    }
  },
  "browserslist": [
    "last 2 versions",
    "Firefox ESR",
    "> 1%",
    "ie >= 11"
  ],
  "dependencies": {
    "@ant-design/icons": "^4.0.0",
	...
  },
  "devDependencies": {
    "@ant-design/bisheng-plugin": "^2.3.0",
	...
  },
  "peerDependencies": {
    "react": ">=16.9.0",
    "react-dom": ">=16.9.0"
  },
  "publishConfig": {
    "registry": "https://registry.npmjs.org/"
  },
  "bundlesize": [
    {
      "path": "./dist/antd.min.js",
      "maxSize": "305 kB"
    },
    {
      "path": "./dist/antd.min.css",
      "maxSize": "65 kB"
    },
    {
      "path": "./dist/antd.dark.min.css",
      "maxSize": "65 kB"
    },
    {
      "path": "./dist/antd.compact.min.css",
      "maxSize": "65 kB"
    }
  ],
  "title": "Ant Design",
  "__npminstall_done": "Mon Apr 27 2020 10:04:51 GMT+0800 (中国标准时间)",
  "_from": "antd@4.1.5",
  "_resolved": "https://registry.npm.taobao.org/antd/download/antd-4.1.5.tgz?..."
}
```

## sideEffects

通常我们发布到 npm 上的包很难保证其是否包含副作用（可能是代码也可能是 [transformer 导致](https://zhuanlan.zhihu.com/p/32831172)），

但是我们基本能确保这个包是否会对包以外的对象产生影响，比如是否修改了 window 上的属性，是否复写了原生对象方法等。如果我们能保证这一点，其实我们就能知道整个包是否能设置 `sideEffects: false`了，至于是不是真的有副作用则并不重要，[这对于 webpack 而言都是可以接受的](https://stackoverflow.com/questions/49160752/what-does-webpack-4-expect-from-a-package-with-sideeffects-false#comment85528470_49203452)。这也就能解释为什么能给 vue(比如很多函数都有对 `Vue.prototype` 的引用甚至修改) 这个本身充满副作用的包加上 `sideEffects: false` 了。

所以其实 webpack 里的 `sideEffects: false` 的意思并不是我这个模块真的没有副作用，而只是为了在tree-shaking时告诉 webpack：**我这个包在设计的时候就是期望没有副作用的，即使他打完包后是有副作用的，webpack 同学你摇树时放心的当成无副作用包摇就好啦！**

也就是说，只要你的包不是用来做 polyfill 或 shim 之类的事情，就尽管放心的给他加上 `sideEffects: false`



## husky

