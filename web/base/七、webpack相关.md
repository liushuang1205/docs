### 1 打包体积 优化思路

- 提取第三方库或通过引用外部文件的方式引入第三方库
- 代码压缩插件`UglifyJsPlugin`
- 服务器启用 gzip 压缩
- 按需加载资源文件 `require.ensure`
- 优化`devtool`中的`source-map`
- 剥离`css`文件，单独打包
- 去除不必要插件，通常就是开发环境与生产环境用同一套配置文件导致

### 2 打包效率

- 开发环境采用增量构建，启用热更新
- 开发环境不做无意义的工作如提取`css`计算文件 hash 等
- 配置`devtool`
- 选择合适的`loader`
- 个别`loader`开启`cache` 如`babel-loader`
- 第三方库采用引入方式
- 提取公共代码
- 优化构建时的搜索路径 指明需要构建目录及不需要构建目录
- 模块化引入需要的部分

### 3 Loader

编写一个 loader

> `loader`就是一个`node`模块，它输出了一个函数。当某种资源需要用这个`loader`转换时，这个函数会被调用。并且，这个函数可以通过提供给它的`this`上下文访问`Loader API`。 `reverse-txt-loader`

    // 定义
    module.exports = function(src) {
      //src是原文件内容（abcde），下面对内容进行处理，这里是反转
      var result = src.split('').reverse().join('');
      //返回JavaScript源码，必须是String或者Buffer
      return `module.exports = '${result}'`;
    }
    //使用
    {
    	test: /\.txt$/,
    	use: [
    		{
    			'./path/reverse-txt-loader'
    		}
    	]
    },

### 4 说一下 webpack 的一些 plugin，怎么使用 webpack 对项目进行优化

**构建优化**

- 减少编译体积 `ContextReplacementPugin`、`IgnorePlugin`、`babel-plugin-import`、`babel-plugin-transform-runtime`
- 并行编译 `happypack`、`thread-loader`、`uglifyjsWebpackPlugin`开启并行
- 缓存 `cache-loader`、`hard-source-webpack-plugin`、`uglifyjsWebpackPlugin`开启缓存、`babel-loader`开启缓存
- 预编译 `dllWebpackPlugin && DllReferencePlugin`、`auto-dll-webapck-plugin`

**性能优化**

- 减少编译体积 `Tree-shaking`、`Scope Hositing`
- `hash`缓存 `webpack-md5-plugin`
- 拆包 `splitChunksPlugin`、`import()`、`require.ensure`

### 5 webpack Plugin 和 Loader 的区别

**Loader：**

用于对模块源码的转换，`loader` 描述了 `webpack` 如何处理非 `javascript` 模块，并且在 `build` 中引入这些依赖。l`oader` 可以将文件从不同的语言（如 `TypeScript`）转换为 `JavaScript`，或者将内联图像转换为 `data URL`。比如说：`CSS-Loader`，`Style-Loader` 等

**Plugin**

目的在于解决 `loader` 无法实现的其他事,它直接作用于 `webpack`，扩展了它的功能。在 `webpack` 运行的生命周期中会广播出许多事件，`plugin` 可以监听这些事件，在合适的时机通过 `webpack` 提供的 `API` 改变输出结果。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务

### 6 tree shaking 的原理是什么

- `ES6 Module`引入进行静态分析，故而编译的时候正确判断到底加载了那些模块
- 静态分析程序流，判断那些模块和变量未被使用或者引用，进而删除对应代码

### 7 common.js 和 es6 中模块引入的区别

> CommonJS 是一种模块规范，最初被应用于 Nodejs，成为 Nodejs 的模块规范。运行在浏览器端的 JavaScript 由于也缺少类似的规范，在 ES6 出来之前，前端也实现了一套相同的模块规范 (例如: `AMD`)，用来对前端模块进行管理。自 ES6 起，引入了一套新的 `ES6 Module`规范，在语言标准的层面上实现了模块功能，而且实现得相当简单，有望成为浏览器和服务器通用的模块解决方案。但目前浏览器对 `ES6 Module` 兼容还不太好，我们平时在 `Webpack` 中使用的 `export`和 `import`，会经过 `Babel` 转换为 `CommonJS` 规范

**在使用上的差别主要有：**

- `CommonJS` 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
- `CommonJS` 模块是运行时加载，ES6 模块是编译时输出接口（静态编译）。
- `CommonJs` 是单个值导出，`ES6 Module` 可以导出多个
- `CommonJs` 是动态语法可以写在判断里，`ES6 Module` 静态语法只能写在顶层
- `CommonJs` 的 `this` 是当前模块，`ES6 Module` 的 `this` 是 `undefined`

### 8 babel 原理

Babel 是一个 JavaScript 编译器。他把最新版的 javascript 编译成当下可以执行的版本，简言之，利用 babel 就可以让我们在当前的项目中随意的使用这些新最新的 es6，甚至 es7 的语法

> `ES6、7`代码输入 -> `babylon`进行解析 -> 得到`AST`（抽象语法树）-> `plugin`用 b`abel-traverse`对`AST`树进行遍历转译 ->得到新的`AST`树->用`babel-generator`通过`AST`树生成`ES5`代码

**Babel 的三个主要处理步骤分别是： 解析（parse），转换（transform），生成（generate）**

- **解析** 将代码解析成抽象语法树（`AST`），每个 js 引擎（比如 Chrome 浏览器中的 V8 引擎）都有自己的 `AST` 解析器，而 `Babel` 是通过 `Babylon` 实现的。在解析过程中有两个阶段：词法分析和语法分析，词法分析阶段把字符串形式的代码转换为令牌（`tokens`）流，令牌类似于 `AST` 中节点；而语法分析阶段则会把一个令牌流转换成 AST 的形式，同时这个阶段会把令牌中的信息转换成 AST 的表述结构
- **转换** 在这个阶段，`Babel` 接受得到 `AST` 并通过 `babel-traverse` 对其进行深度优先遍历，在此过程中对节点进行添加、更新及移除操作。这部分也是 `Babel` 插件介入工作的部分
- **生成** 将经过转换的 `AST` 通过 `babel-generator`再转换成 `js` 代码，过程就是深度优先遍历整个 `AST`，然后构建可以表示转换后代码的字符串。
