---
title: webpack中的loader及其配置
categories: webpack
---

### 一、loader概述

在webpack当中，每个loader本质上就是一个函数，用公式可以解释为：

output = loader(input)。这点从loader的源码结构中也可以看出：

```
module.exports = function loader(content, map, meta) {
    const callback = this.async;
    const result = handler(content, map, meta);
    callback(
        null, // error
        result.content, // 转换后的内容
        result.map, // 转换后的source map
        result.meta // 转换后的AST
    )
}
```
loader本身就是一个函数，在该函数当中对接收的内容进行转换，然后返回转换

后的结果（可能包含source map 和 AST对象）.


loader的字面意思是装载器，在webpack当中它的实际功能更像是一个预处理器，

webpack本身只认识javascript，对于其他类型的资源必须预先定义一个或多个

loader对其进行转译，输出为webpack能够接收的形式再继续，因此loader做的

实际上是一个预处理的工作。

### 二、引入loader

直接从一个JS文件当中加载一个CSS文件

```
// index.js
import addContent from './add-content.js'
import './style.css'
document.write('My first Webpack App.<br/>');
addContent();
```

```
// style.css
body {
    text-align: center;
    padding: 100px;
    color: #ffffff;
    background-color: #09c;
}
```

因为此时没有引入任何loader，因此直接打包将会报错：
![image](/public/images/loader-not-found.png)

下面我们把css-loader加入到项目当中。loader都是一些第三方的npm模块，webpack本身

不包含任何loader,因此我们首先要从npm安装:

```
npm install css-loader
```

接下来我们在webpack.conf.js中配置loader:


```
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
    },
    // 在module当中配置loader
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['css-loader']
            }
        ]
    }
}
```

与loader相关的配置都在module对象中，其中module.rules代表了模块的处理规则。每条规则内部可以包含很多配置项，

这里我们只使用了最中要的两项--test和use。

- test可接收一个正则表达式或者一个元素为正则表达式的数组，只有正则匹配的模块才会运用这条规则，在本例当中用
/\.css$/结尾的文件。
- use可接收一个数组，数组包含该规则所使用的loader，在本例中只配置了一个css-loader，在只有一个loader时

也可以简化为字符串"css-loader"。

此时我们再进行打包，之前的错误已经消失了。但是css的样式仍然没有在页面当中生效。这是因为css-loader的作用仅仅是

处理css的各种加载语法(@import和url()函数等)。如果要使样式起作用还需要style-loader来把样式插入页面。css-loader

和style-loader通常是搭配在一起使用的。

只使用css-loader:

![image](/public/images/css-loader-1.png)


同时使用css-loader和style-loader：


![image](/public/images/css-loader-2.png)