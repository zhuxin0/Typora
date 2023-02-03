- [webpack 配置](#webpack-配置)
  - [一、项目初始化](#一项目初始化)
    - [1、生成 package.json 文件](#1生成-packagejson-文件)
    - [2、下载依赖包](#2下载依赖包)
  - [二、开发环境和生产环境](#二开发环境和生产环境)
    - [1、配置文件](#1配置文件)
    - [2、合并公共配置](#2合并公共配置)
    - [3、配置环境变量 process.env](#3配置环境变量-processenv)
    - [4、配置启动命令](#4配置启动命令)
  - [三、入口出口配置](#三入口出口配置)
    - [1、入口配置](#1入口配置)
    - [2、出口配置](#2出口配置)
  - [四、loader 和 plugin](#四loader-和-plugin)
    - [1、loader](#1loader)
    - [2、plugin](#2plugin)
  - [五、HTML 配置](#五html-配置)
    - [1、下载插件](#1下载插件)
    - [2、配置插件](#2配置插件)
  - [六、SASS 配置](#六sass-配置)
    - [1、下载依赖包](#1下载依赖包)
      - [下载失败的解决方案](#下载失败的解决方案)
    - [2、配置 SASS](#2配置-sass)
  - [七、复制文件](#七复制文件)
    - [1、下载插件](#1下载插件-1)
    - [2、配置插件](#2配置插件-1)
  - [八、source map 配置](#八source-map-配置)
  - [九、配置服务器代理](#九配置服务器代理)
  - [十、配置忽略文件](#十配置忽略文件)
  - [十一、清除多余文件](#十一清除多余文件)
    - [1、下载插件](#1下载插件-2)
    - [2、配置插件](#2配置插件-2)
  - [十二、性能优化](#十二性能优化)
    - [1、开启多进程打包：HappyPack](#1开启多进程打包happypack)
      - [1）下载插件](#1下载插件-3)
      - [2）配置插件](#2配置插件-3)
    - [2、多进程压缩 JS：ParallelUglifyPlugin](#2多进程压缩-jsparalleluglifyplugin)
      - [1）下载插件](#1下载插件-4)
      - [2）配置插件](#2配置插件-4)

# webpack 配置

## 一、项目初始化

### 1、生成 package.json 文件

任何前端项目中，在项目根目录中都会有一个 `package.json` 文件，作为项目的描述文件。

```json
{
    "name": "webpack-demo",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [],
    "author": "",
    "license": "ISC"
}
```

如果项目中没有该目录，可以通过以下命令初始化项目，生成该文件：

```bash
npm init -y
```

### 2、下载依赖包

```bash
npm install webpack webpack-cli webpack-dev-server -D
```

依赖包说明：

- `webpack`：webpack 核心文件；
- `webpack-cli`：webpack 核心文件；
- `webpack-dev-server`：用于配置开发服务器；

## 二、开发环境和生产环境

在日常的前端开发工作中，一般都会有两套构建环境，分别是“开发环境”和“生产环境”。

| 区别       | 开发环境         | 生产环境     |
| ---------- | ---------------- | ------------ |
| 构建结果   | 用于本地开发调试 | 直接用于线上 |
| 代码压缩   | 否               | 是           |
| 打印 debug | 是               | 否           |

webpack 的配置中专门提供了一个 mode 属性，用来设置当前环境：

```js
export default = {
    // mode: 'production',  // 生产环境
    mode: 'development'  // 开发环境
}
```

针对不同的环境，webpack 默认会启用不同的插件，除了默认的以外，后续我们也会根据需求再添加不同的配置。

### 1、配置文件

通常我们会针对开发环境和生产环境创建不同的配置文件：

```bash
|--- webpack.dev.config.js    # 开发环境的配置
|--- webpack.prod.config.js   # 生产环境的配置
```

考虑到开发环境和生产环境还存在很多相同的配置，所以我们通常还会创建一个公共配置文件：

```bash
|--- webpack.base.config.js   # 开发环境和生产环境的公共配置
```

### 2、合并公共配置

公共配置代码提取到 `webpack.base.config.js` 文件后，需要再通过 `webpack-merge` 依赖包和两个环境的独立配置合并。

```bash
npm i webpack-merge -D
```

在 `webpack.dev.config.js` 和 `webpack.prod.config.js` 中引入并合并公共配置：

```js
// webpack.dev.config.js
const base = require('./webpack.base.config');
const { merge } = require('webpack-merge');

module.exports = merge(base, {
	// 其他配置
})
```

```js
// webpack.prod.config.js
const base = require('./webpack.base.config');
const { merge } = require('webpack-merge');

module.exports = merge(base, {
    // 其他配置
})
```

### 3、配置环境变量 process.env

> process 是 nodejs 中的一个全局变量，所有模块都可以直接调用。

虽然 webpack 中提供了一个 mode 属性直接来配置当前环境。但是我们更倾向于通过 `process.env` 来控制 mode 的值，从而区分开发环境和生产环境。

### 4、配置启动命令

配置启动命令前，我们先下载一个插件：`cross-env`。

```bash
npm i cross-env -D
```

cross-env 插件的作用是可以运行跨平台设置和使用环境变量。简单来说就是方便我们配置不同环境的启动命令。

在 `package.json` 文件中配置不同环境的不同启动命令：

```json
{
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "dev": "cross-env NODE_ENV=development webpack-dev-server --config webpack.dev.config.js",
		"build": "cross-env NODE_ENV=production webpack --config webpack.prod.config.js"
    }
}
```

在公共设置中通过环境变量配置 mode：

```js
const { NODE_ENV } = process.env;

module.exports = {
    mode: NODE_ENV
}
```

至此，我们关于生产环境和开发环境的基本配置就处理好了。

我们可以在项目根目录中创建一个 `src/index.js` 文件，来运行以下两个启动命令查看效果。

```bash
# 开发环境
npm run dev

# 生产环境
npm run build
```

## 三、入口出口配置

### 1、入口配置

入口，指的是 webpack 在编译时最先要处理的 `.js` 文件。

入口配置，就是要通过 `entry` 属性告诉 webpack，这个 `.js` 文件所在的路径位置。

入口配置属于公共配置：

```js
module.exports = {
    // 入口文件位置
    entry: {
        index: './src/index.js'
    }
}
```

### 2、出口配置

出口，指的是 webpack 编译完成后，生成的新代码的存储位置。

出口配置，就是要通过 `output` 属性告诉 webpack，这些新文件所在的路径位置。

出口配置属于公共配置：

```js
const path = require("path");
module.exports = {
    // 出口文件位置
    output: {
        // 出口文件的路径(__dirname 获取项目的绝对路径)
        path: path.resolve(__dirname, 'dist'),
        // 出口文件的名称
        filename: 'js/[name].js'
    },
}
```

其中，`filename` 中的 `js/` 表示打包后的文件都存放在 `js` 目录中。

## 四、loader 和 plugin

除了处理 JS 代码外，我们在项目中还需要处理 HTML、CSS、SASS、LESS 等其他很多类型的代码。在配置处理其他代码以前，我们需要先了解两个概念：loader 和 plugin。

### 1、loader

webpack 在默认情况下，只能编译处理 JS 代码。因此，如果我们要处理除了 JS 以外的其他代码，就需要借助第三方的 loader。

例如：

- `style-loader`：用于将 `css` 编译完成的结果添加到页面 `<style>` 标签上；
- `css-loader`：用于编译 `.css` 文件，需要搭配 `style-loader` 使用；
- `sass-loader`：用于编译 `.scss` 文件，将 SASS 代码转换为 CSS 代码；
- `less-loader`：用于编译 `.less` 文件，将 LESS 代码转换为 CSS 代码；
- `babel-loader`：用于将高版本的 JS 语法转换为低版本的 JS 语法；
- `ts-loader`：用于编译 `.ts` 文件，将 TS 代码转换为 JS 代码；
- `vue-loader`：用于编译 `.vue` 文件；
- `eslint-loader`：用于检查代码是否符合规范，是否存在语法错误等；
- ...

### 2、plugin

代码转换的工作由 loader 来处理，除此之外的其他任何工作都可以交由 plugin 来完成。

例如：

- `html-webpack-plugin`：生成 HTML 文件，自动引入打包好的 JS、CSS 文件；
- `uglifyjs-webpack-plugin`：压缩 JS 代码；
- `mini-css-extract-plugin`：将编译后的 CSS 代码提取成独立的文件；
- `copy-webpack-plugin`：复制文件（用来处理一些不需要编译的文件）；
- `clean-webpack-plugin`：自动清除打包后多余无用的文件；
- ...

## 五、HTML 配置

webpack 默认只能处理 `.js` 文件，如果要处理 HTML 文件，我们需要依赖其他的插件。

例如：我们在 `src` 目录中创建一个 `index.html` 文件。

### 1、下载插件

```bash
npm i html-webpack-plugin -D
```

### 2、配置插件

HTML 的配置也属于公共配置：

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',   // 源文件路径
            filename: 'index.html',         //打包后的文件名
            chunks: ['index'],              // 对应的入口 js 文件
        })
    ],
}
```

## 六、SASS 配置

我们可以在 `src` 目录中创建一个 `styles` 目录，用来存放所有的 `.scss` 文件。例如：

```bash
src
 |--- styles
 |       |--- index.scss
```

然后在 `index.js` 引入样式文件：

```js
import './styles/index.scss';
```

### 1、下载依赖包

```bash
npm i node-sass sass-loader css-loader mini-css-。s -D
```

#### 下载失败的解决方案

如果下载失败，大概率是 node-sass 造成的，那我们先去掉 node-sass，下载其他依赖包：

```bash
npm i sass-loader css-loader mini-css-extract-plugin -D
```

然后重新通过以下方式下载 node-sass：

```bash
npm install node-sass --sass-binary-site=https://npm.taobao.org/mirrors/node-sass
```

### 2、配置 SASS

SASS 配置也属于公共配置：

```js
const MineCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    module: {
        rules: [
            {
                test: /\.s[ac]ss$/i,
                exclude: /node_modules/,
                use: [
                    MineCssExtractPlugin.loader,
                    'css-loader',
                    'sass-loader'
                ]
            },
        ]
    },
    plugins: [
        // ...
        new MineCssExtractPlugin({
            filename: 'css/[name].css'  // 配置打包后的 CSS 文件的存储位置
        })
    ]
}
```

附加说明：如果项目中使用的是 Less，就需要下载 less-loader。

## 七、复制文件

在项目中，有一些文件我们是不需要 webpack 处理的。但是，在 webpack 中，“不需要处理”，这个功能也需要我们通过插件来配置。

例如项目 `src` 中有一个 `static` 目录，我们希望将该目录中的所有文件都作为静态资源文件，不需要 webpack 编译处理。

### 1、下载插件

`copy-webpack-plugin` 插件的作用就是用来将指定文件复制到一份到指定位置。所以，我们项目中不需要 webpack 额外处理的文件，都可以通过该插件让 webpack 直接复制即可：

```bash
npm i copy-webpack-plugin -D
```

### 2、配置插件

复制文件也属于公共配置：

```js
// 公共配置
const CopyPlugin = require("copy-webpack-plugin");

module.exports = {
    plugins: [
        new CopyPlugin({
            patterns: [
                // from：原文件的位置，to：复制后的位置
                { from: "./src/static", to: "./static" },
            ]
        })
    ]
}
```

前面我们讲的基本上都是一些公共的配置，这里我们再看一些生产环境和开发环境不同的配置。

## 八、source map 配置

使用 webpack 管理项目后，浏览器中运行的代码实际上是 webpack 编译之后的代码，这样就会导致一个问题，当我们的代码出现错误时，浏览器给出的错误提示是编译后的代码位置。

而 source maps 功能可以将编译后的代码映射回原始源代码。一旦代码出现报错，source map 会明确的告诉我们源代码中错误的代码位置。

因此，在开发环境的配置的中启用 source map：

```js
module.exports = merge(base, {
    devtool: 'inline-source-map'
})
```

## 九、配置服务器代理

现在的开发模式基本上都是前后端分离开发，所以通常调试接口时，都需要在开发环境中配置一下开发代理：

```js
module.exports = merge(base, {
    devServer: {
        hot: true,  // 启动热更新
        port: 3000, // 设置服务器端口号为 3000
        open: true, // 启动服务器时自动打开浏览器
        proxy: {    // 代理跨域配置
            '/api': {
                target: 'http://localhost:8080',
                changeOrigin: true,
                pathRewrite: { "^/api": "" }
            }
        }
    }
})
```

## 十、配置忽略文件

当项目代码发生改变时，webpack 会检测到变化的代码，并重新进行编译。但是实际上有些目录中的代码是不需要 webpack 去检测的，所以我们可以通过配置，将一些文件忽略掉，让 webpack 不用去管理这些目录中的文件。

配置忽略文件也是在开发环境中配置：

```js
module.exports = merge(base, {
    watchOptions: {
        ignored: /node_modules/
    }
})
```

## 十一、清除多余文件

### 1、下载插件

```bash
npm i clean-webpack-plugin -D
```

### 2、配置插件

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
module.exports = merge(base, {
    plugins: [
        new CleanWebpackPlugin()
    ]
})
```

## 十二、性能优化

### 1、开启多进程打包：HappyPack

webpack 是单线程模型，也就是说 webpack 需要一个一个地去处理任务，不能同时处理多个任务。

HappyPack 可以将任务分解给多个子进程去并发执行，子进程处理完后再将结果发送给主进程，从而发挥多核 CPU 电脑的威力。

#### 1）下载插件

```bash
npm i happypack -D
```

#### 2）配置插件

我们在公共配置中进行配置：

```js
const HappyPack = require('happypack');

// 构造出共享进程池，进程池中包含5个子进程
var happyThreadPool = HappyPack.ThreadPool({ size: 5 });

module.exports = {
    module: {
        rules: [
            // 其他 loader 配置
            {
                test: /\.js$/,
                // 将.js文件交给id为babel的 happypack 实例来执行
                // 1) 用 happypack/loader 代原始的 loaders 列表
                use: 'happypack/loader?id=babel',
                // 排除 node_modules 目录下的文件，
                // node_modules 目录下的文件都是采用的 ES5 语法，没必要再通过 Babel 去转换
                exclude: /node_modules/,
            }
        ]
    },
    plugins: [
        // ... 其他插件配置
        // 2)创建一个plugin
        new HappyPack({
            // id 标识 happypack 处理那一类文件
            id: 'babel',
            // 共享线程池
            threadPool: happyThreadPool,
            // 3) 配置一个替代步骤 1) 中的loader
            loaders: ['babel-loader'],
            // 日志输出
            verbose: true
        })
    ]
}
```

### 2、多进程压缩 JS：ParallelUglifyPlugin

#### 1）下载插件

```bash
npm i webpack-parallel-uglify-plugin -D
```

#### 2）配置插件

这个我们可以在开发环境中配置：

```js
const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');

module.exports = merge(base, {
    plugins: [
        // ... 其他插件的配置
        new ParallelUglifyPlugin({
            // 传递给 UglifyJS 的参数
            // （还是使用 UglifyJS 压缩，只不过帮助开启了多进程）
            uglifyJS: {
                output: {
                    beautify: false, // 最紧凑的输出
                    comments: false, // 删除所有的注释
                },
                compress: {
                    // 删除所有的 `console` 语句，可以兼容ie浏览器
                    drop_console: true,
                    // 内嵌定义了但是只用到一次的变量
                    collapse_vars: true,
                    // 提取出出现多次但是没有定义成变量去引用的静态值
                    reduce_vars: true,
                }
            }
        })
    ]
})
```



