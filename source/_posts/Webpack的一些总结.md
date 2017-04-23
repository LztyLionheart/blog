---
title: Webpack的一些总结
date: 2017-04-23 08:06:42
tags:
- Webpack
categories: 
- Javascript
---
# webpack的一些基本操作
   ## 1. `webpack` 基本介绍
   
   1.  `webpack` 作为当前最为流行的前端模块化打包工具 可以通过快速建立应用程序的依赖图表并以正确顺序打包他们来简化你的工作流，通过针对性的配置对`webpack`进行自定义优化配置，比如为生产环境拆分`vendor/css/js` 代码， 通过运行开发服务器来实现无刷新热重载等一系列功能。<!-- more -->
   2. 创建一个示例目录尝试`webpack`
   
   	``` shell
   		mkdir webpack-demo && webpack-demo
   		npm init -y
   		npm install --save-dev webpack
   	```  
   
   	创建app目录 并新建一个`index.js`文件
   
   	``` shell
   	function component () {
   			var element = document.createElement('div');
   
   			/* 需要引入lodash， 下一行才能正常工作 */
   			element.innerHTML = _.join(['Hello','webpack'], ' ');
   
   			return element;
   
   	}
   	document.body.appendChild(component());
    ```  
    
   ## 要运行这段代码 通常需要一下`html`:
   
   	### index.html
   	``` shell
   	<!DOCTYPE html>
   	<html>
   		<head>
   			<meta charset="utf-8">
   			<title>webpack demo</title>
   			<script type="text/javascript" src="https://unpkg.com/lodash@4.16.6"></script>
   		</head>
   		<body>
   			<script type="text/javascript" src="app/index.js"></script>
   		</body>
   	</html>
   
   	```
   
     ### 这个示例中 `<script>` 标签之间存在隐式依赖关系
   
   	### 运行`index.js` 的前提是必须依赖提前引入的`lodash`。 之所以说是隐式是因为index.js 并未显式申明需要引入`ladash`, 只能假定推测已经存在一个全局变量`_`。
   
     ### 使用这种方法去管理javascript 项目会有一些问题：
   
     *  如果依赖不存在， 或者引入顺序错误， 应用程序将无法正常运行。
     *  如果依赖被引入但是并没有没使用, 那样就会存在浏览器下载了无用代码。
   
     ### 要在 `index.js` 中打包 `lodash`依赖,  首先我们需要安装 `lodash`。
   
   	``` shell
   	npm install --save lodash
   	```
   
   	### 然后引入（import）它。
   	### app/index.js
     ``` shell
     + import _ from 'lodash';
     ```
   
   	### 之后修改`index.html`, 来引入打包好的单个`js` 文件。
   
   	``` shell
   	<!DOCTYPE html>
   		<html>
   			<head>
   				<meta charset="utf-8">
   				<title>webpack demo</title>
   				<!-- <script type="text/javascript" src="https://unpkg.com/lodash@4.16.6"></script> -->
   			</head>
   			<body>
   				<script src="dist/bundle.js"></script>
   			</body>
   		</html>
   	```  
   
   	### 	在这里 `index.js` 显示要求引入的`lodash`必须存在，然后他将以`_`的别名绑定(不会造成全局范围变量污染)。
   
   	### 通过声明模块所需的依赖，`webpack` 能够利用这些信息去构建依赖图表，然后使用图表生成一个优化过的，会以正确代码顺序被运行的 `bundle`。并且没有用到的依赖将不会被 `bundle` 引入。
   
   	### 现在在此文件夹下带上以下参数运行 `webpack`，其中 `index.js` 是入口文件，`bundle.js` 是已打包所需的所有代码的输出文件。
   
   	``` shell
   	$ webpack app/index.js dist/bundle.js
   	Hash: 8127988a792147791f3d
   	Version: webpack 2.3.3
   	Time: 437ms
   		Asset    Size  Chunks                    Chunk Names
   	bundle.js  544 kB       0  [emitted]  [big]  main
   		[0] ./~/.4.17.4@lodash/lodash.js 540 kB {0} [built]
   		[1] ./app/index.js 260 bytes {0} [built]
   		[2] (webpack)/buildin/global.js 509 bytes {0} [built]
   		[3] (webpack)/buildin/module.js 517 bytes {0} [built]
   	```
     ### 在浏览器中打开 `index.html`, 查看吗构建成功的bundle的结果。你应该能看到带有以下文本的页面：‘Hello webpack’。
   
   
   ## 1.1 在`webpack`中使用`ES2015`模块
   
   1. ### 在`app/index.js` 中 使用了ES2015模块(module import), 尽管`import/ export` 并未在全部浏览器中支持 但依旧可以正常使用，因为`webpack`将会将其替换为`ES5`兼容的代码。
   
   2. ### 但`webpack` 不会更改除了 `import / export`除外的代码 如果需要使用`ES6`语法 请使用`babel` 转译器。
   
   ## 1.2 使用带有配置的 `webpack`
   
   1. ### 对于复杂配置  我们可以使用一个配置文件 webpack会根据配置文件来打包代码创建一个 `webpack.config.js`
   
   	### webpack.config.js
   
   	``` shell
   	var path = require('path');
   		module.exports = {
   			entry: './app/index.js',
   			output: {
   				filename: 'bundle.js',
   				path: path.resolve(__dirname, 'dist')
   			}
   		};
   	```
   	### 此文件可以被__webpack__运行
   
   	``` shell
   	$ webpack --config webpack.config.js
   	Hash: 8127988a792147791f3d
   	Version: webpack 2.3.3
   	Time: 388ms
   		Asset    Size  Chunks                    Chunk Names
   	bundle.js  544 kB       0  [emitted]  [big]  main
   		[0] ./~/.4.17.4@lodash/lodash.js 540 kB {0} [built]
   		[1] ./app/index.js 260 bytes {0} [built]
   		[2] (webpack)/buildin/global.js 509 bytes {0} [built]
   		[3] (webpack)/buildin/module.js 517 bytes {0} [built]
   	```
   
   > 如果存在 **webpavk.config.js**, __webpack__ 命令将默认选择使用它.
   
   > 删除上述**bundle**文件 重新执行脚本  验证输出内容
   
   ## 1.3 配合npm 使用
   
   ###  考虑到用cli 这种方式来运行webpack 不是特别方便 可以设置快捷方式
   ``` shell
   {
   	...
   	"scripts": {
   		"build": "webpack"
   	},
   	...
   }
   ```
   ### 现在你可以通过使用 `npm run build`，命令来实现与上面相同的效果， npm 通过命令选取脚本，并临时修补执行环境，使脚本可以在运行时包含bin 命令 。 你可以在很多项目中看到这种使用习惯。
   > 你可以通过向 **npm run build** 命令添加两个中横线， 给webpack传递自定义参数，例： ** npm run build -- --colors **.
   
   
   
   ## 2. 安装
   
   1. ### 前提条件
   
   	#### 在开始前， 先要确认已安装Node.js的最新版本。建议使用最新`LTS`, 使用旧版本 可能会遇到各种问题， 因为它们可能缺少webpack功能或缺少相关package包。
   
   2. ### 本地安装
   ``` shell
   npm  install webpack --save-dev
   npm  install webpack@<version> --save-dev
   ```
   	#### 如果你在项目中使用 npm 执行脚本 npm 首先会在你的本地模块中寻找webpack。
   	``` 
   	"scripts": {
   		"start": "webpack --config webpack.config.js"
   	}
   	```
     #### 上面是npm的标准配置 也是我们推荐的实践。
   	> 当你在本地安装webpack后 你能够在 *node_modules/./bin/webpack* 中找到它的二进制程序。
   3. ### 全局安装(全局安装会锁定webpack到指定版本)
       ``` shell
       npm install webpack -g
       ```
       #### `webpack` 命令可以全局执行了.
   4. ### 体验最新版本
   	``` shell
   	npm install webpack/webpack#<tagname/branchname>
   	```
   	
   ## 3. 生产环境构建
   1. ### 自动方式
      #### 运行`webpack -p`(也可以运行 `webpack --optimize-minimize --define process.enc.NODE_ENV="'production'"`, 他们是等效的)。
      * 使用`UglifyJsPlugin` 进行JS 文件压缩;
      * 运行 `LoaderOptionsPligin`;
      * 设置Node环境变量。
      ##### 1.1 JS 文件压缩
      ###### webpack自带了 `UglifyJsPlugin`, 运行**UglifyJS** 来压缩输出文具文件。 此插件支持所有的**UglifyJS选项**。在命令行中指定 --optimize-minimize 或在plugins配置中添加：
      
       ``` 
       const webpack = require('webpack);
           module.exports = {
             /*...*/
             plugins: [
               new webpack.optimize.UglifyJsPlugin({
               souceMap: options.devtool && (options.devtool.indexOf("sourcemap") >=0 || options.devtool.indexOf("soucre-map") >=0)
               })
             ]
       };
       ```
   2. ### Node环境变量
      #### 运行`webpack -p`(或者 `--define process.env.NODE_ENV="'production'"`) 会通过如下方式调用 `DefinePlugin`
      
     ```
     const webpack = require('webpack');
     module.exports = {
       /*...*/
       plugins:[
           new webpack.DefinePlugin({
             'process.env.NODE_ENV': JSON.stringify('production')
           })
       ]
     };
     ```



