# Simple-webpack-dev

> webpack搭建项目结构入门

## 安装

`npm i --save-dev -D webpack`

![](https://rails365.oss-cn-shenzhen.aliyuncs.com/uploads/photo/image/320/2017/5734645b60521d83d889a7716ff5a542.svg)



## 概念

- Entry
- OutPut
- Loader
- Plugins

### Entry 入口

指示webpack应该从哪里开始，开始逐步构建内部依赖（直接和间接），最终输出到bundles文件中。

可以定义一个或者多个入口起点

```javascript
  entry: './entry/file.js',
```

#### context

webpack寻找相对路径的文件，会议context为根目录，context默认为执行webpack时所在的当前工作目录。

如果想改变context的默认配置，可以这样设置：

```javascript
module.exports={
  context: path.resolve(__dirname, 'app');
};
```

context必须是绝对路径字符串，也可以在启动时带上参数来设置context`webpack --context`

#### Entry类型

| 类型     | 例子                                       | 含义                   |
| ------ | ---------------------------------------- | -------------------- |
| string | './app/entry'                            | 入口模块的文件路径，可以使相对      |
| array  | ['./app/entry1', './app/entry2']         | 同上                   |
| object | {a: './app/entry', b:['./app/entry2', './app/entry3'] | 配置多个入口，每个入口生成一个Chunk |

#### Chunk

chunk的名称和Entry配置有关

- 如果entry是一个string或者array，只会生成一个chunk，这时chunk的名称是main。
- 如果entry是一个object，可能会出现多个chunk，每个chunk的名称是object键值对中的键的名称。

### Output 出口

告诉webpack在哪里输出创建的bundles，以及如何命名这些文件。

```javascript
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'my-first-webpack.bundle.js'
  }
```

#### filename

如果只有一个输出文件，为string类型且静态不变`filename:'bundle.js'`

若有多个Chunk要输出时，就需要借助模板和变量了，我们可以用上文提到的Chunk的每个名称来区分输出的文件名：

`filename:'[name].js'`

name为内置变量，还有其他如下内置变量：

| 变量        | 释义              |
| --------- | --------------- |
| id        | chunk的唯一标志，从0开始 |
| name      | chunk的名称        |
| hash      | chunk唯一标志的hash值 |
| chunkhash | chunk内容的hash值   |

其中，hash和thunkhash的长度是可以指定的，[hash:8]代表8位Hash值，默认是20位。

#### chunkFilename

它用来配置无入口的chunk在输出时的文件名。它只用来对运行过程中生成的chunk的输出文件进行命名。

应用场景：CommonChunkPlugin、使用import动态加载。

chunkFilename和filename一样支持内置变量。

#### Path

输出文件的目标位置，通常使用path模块获取绝对路径。

#### publicPath

Output.publicPath用于配置发布到线上资源的URL前缀，string类型。默认值是'',代表使用相对路径。

场景：需要将构建出的资源文件上传到CDN服务上，用来加快页面加载速度。配置代码如下：

```javascript
filename: '[name]_[chunkhash: 8].js',
publicPath: 'https://cdn.meituan.com/assets/',
```

这时发布到线上的HTML在引入js文件时，需要进行以下配置：

```html
<script src='https://cdn.meituan.com/assets/a_12345678.js'></script>
```

path和publicPath都支持字符串模板，变量只有一个：hash，代表依次编译操作的hash值。

#### crossOriginLoading

webpack输出部分代码可能需要异步加载，异步加载通过[JSONP](https://zh.wikipedia.org/wiki/JSONP)方式实现的。

JSONP的原理是动态地像HTML中插入一个`<script src='url'>`标签，用来加载异步资源。

crossOriginLoading用于配置crossorigin值。

crossorigin是script标签的一个属性，可能有一下取值：

- anonymous(default): 在加载脚本的时候不会带上用户的Cookies
- use-credentials：会带上用户的cookies

通常可以用crossorigin来获取异步加载的脚本执行时的详细错误信息。

#### LibraryTarget 和 Library

用于构建一个可以被其他模块导入的库。

- LibraryTarget配置导出库的方式
- Library 配置导出库的名称

LibraryTarget是枚举类型，支持下面几种配置：

1. var（default）

   编写的库通过var的方式赋值给对应library指定的变量

 2.commonjs

​       编写的库将通过commonjs规范导出

1. commonjs2

​       编写的库通过CommonJs规范导出，module.exports~

​       这种情况下，output.library将没有意义

commjs和commonjs2的区别在于，commonjs2增加了module.exports

1. this
2. window
3. Global

#### libraryExport

略

### Loader

loader的本意是让webpack去处理**非javascript文件**。loader可以将所有类型的文件转换为自己能够处理的有效模块，对他们进行处理。

一般来说，在webpack配置loader有两个注意点：

1. test属性: 识别出需要进行转换的文件
2. use属性：使用对应的loader进行转换

```javascript
const path = require('path');

const config = {
  entry: '/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};

module.exports = config;
```

以上配置在module对象中定义了`rues`属性，它内部包含两个必须属性： `test`和`use`

实际过程是：当webpack进行编译的过程中，在遇到import和require时如果引用的是.txt文件时，先对它使用raw-loader转换一下，再进行打包。

### Plugins

插件用于发散webpack的各种功能，世界上所有的插件基本上都是这个作用。它涉及的范围，从打包优化、压缩甚至是帮你做一件你一直懒得做的小事，都是webpack插件可能的内容。

插件分内部插件和外部插件，内部插件，从webpack中可以直接使用，其他插件则需要先下载对应的包再进行使用。

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件
const path = require('path');

const config = {
  entry: './entry/file.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

上面配置中的`HtmlWebpackPlugin`就是一个插件，它是需要单独下载的（npm），它用于简单创建html文件。而``HtmlWebpackPlugin`为webpack内部的插件，用于压缩js文件。





