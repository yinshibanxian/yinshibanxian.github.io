---
title: webpack配置文件及webpack-dev-server
categories: webpack
---
### 一、使用配置文件
使用npm srcipts是一种解决方案，但是当项目需要越来越多的配置时，

就要往命令行中添加更多参数，后期维护会相当困难，为了解决这个问题，

可以将这些参数改为对象的形式专门放在一个配置文件当中，在webpack每次打包

的时候读取该配置文件即可。

webpack的默认配置文件为webpack.config.js，在工程根目录下创建webpack.config.js，

并添加如下代码：

```
const path = require('path')
module.exports = {
    entry: './src/index.js', // Javascript执行入口文件
    // 需要指定以下输出的路径Path和输出的文件名filename
    output: {
        filename: 'bundle.js', // 自定义输出文件名
        path: path.resolve(__dirname,'./dist') // 自定义输出文件所在目录，这里需要使用绝对路径
    },
    // 设置mode
    mode: 'development'
}
```

此时就可以去掉package.json当中配置的打包参数了
```
"scripts": {
    "build": "webpack"
  },
```
执行npm run build，webpack就会预先读取webpack.config.js，然后进行打包。

### 二、使用webpack-dev-server

单纯使用Webpack及它的命令行工具来进行调试的效率并不是很高，因为多了一步打包的步骤。

因此webpack也提供了一个便捷的本地开发工具--webpack-dev-server。安装命令：

```
npm install webpack-dev-server --save-dev
```

为了便捷的使用webpack-dev-server，可以在package.json添加npm scripts：
```
"scripts": {
    "build": "webpack",
    "dev": "webpack-dev-server"
  },
```

同时，还要在webpack.config.js对webpack-dev-server进行配置:
```
const path = require('path')
module.exports = {
    entry: './src/index.js', // Javascript执行入口文件
    // 需要指定以下输出的路径Path和输出的文件名filename
    output: {
        filename: 'bundle.js', // 自定义输出文件名
        path: path.resolve(__dirname,'./dist') // 自定义输出文件所在目录，这里需要使用绝对路径
    },
    // 设置mode
    mode: 'development',
    devServer: {
        publicPath: '/dist'
    }
}
```

最后，执行npm run dev 并且在浏览器打开http://localhost:8080，即可看到输出结果

值得注意的是，直接使用webpack和webpack-dev-server有一个很大的区别，前者每次都会生成

bundle.js，而webpack-dev-server只是将打包结果放在内存中，并不会写入实际的bundle.js，

只是在每次浏览器请求时放回内存中的打包结果。