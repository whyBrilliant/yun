# Parcel

## usage

1.  Parcel 可以使用任何类型的文件作为入口，但是最好还是使用 HTML 或 JavaScript 文件。 



2.  Parcel 内置了一个当你改变文件时能够自动重新构建应用的开发服务器，而且为了实现快速开发，该开发服务器支持[热模块替换](https://parceljs.org/hmr.html)。 



3.  现在在浏览器中打开 http://localhost:1234/。如果模块热重载没有生效，你可能需要[配置你的编辑器](https://parceljs.org/hmr.html#safe-write)。你也可以使用 `-p ` 选项覆盖默认的端口。  --host 选项覆盖默认的地址。



4.  在本地浏览器中使用不支持的语法 async/await ，切记需要在你的应用程序引入 `babel-polyfill` 或在你的库中引入 `babel-runtime` + `babel-plugin-transform-runtime` 。 



5. package.json 的脚本文件

   ```
   {
     "scripts": {
       "dev": "parcel <your entry file>",
       "build": "parcel build <your entry file>"
     }
   }
   ```


   