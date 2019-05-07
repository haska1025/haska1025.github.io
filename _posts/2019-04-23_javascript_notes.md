# 概念
## npm

### 介绍

npm是基于node.js实现的一个JavaScript软件包管理工具。

安装方法：

1. 直接下载安装

2. 根据nvm安装

### 基本命令

查看npm版本：npm -v

更新最新: npm install npm@latest -g

安装软件包：npm install <module name>
  
卸载软件包: npm uninstall <module name>  

查看当前目录(包含node_modules目录)安装的软件包：npm list

查看模块的package.json文件: npm view <module name>
  
### 更新依赖包

安装npm-check: npm install -g npm-check

全局更新包：npm-check -u -g

更新项目包：npm-check -u



## nvm (node version manager)

主要解决多个nodejs版本，导致的permission error。

安装参考：https://docs.npmjs.com/downloading-and-installing-node-js-and-npm

### 基本命令

查看nvm版本：nvm -v

查看安装的node：nvm list

安装node: nvm install <版本号>, 例如：nvm install v10.15.1

卸载node: nvm uninstall <版本号>

应用node: nvm use <版本号>

### 遇到的问题

1. 安装v10.15.1的时候报如下错误，导致npm安装失败。

```c
Downloading npm version 6.4.1... Error while downloading https://github.com/npm/cli/archive/v6.4.1.zip - unexpected EOF
Complete
Installing npm v6.4.1...2019/04/24 11:06:04 Failed to extract npm. Could not find C:\Users\Administrator\AppData\Roaming\nvm\temp\nvm-npm\npm-6.4.1\bin
```
原因: 可能默认下载库的软件包有bug导致，修改成taobao的库，可以解决此问题。

解决办法：设置nvm settings.txt文件，增加如下内容：

node_mirror: https://npm.taobao.org/mirrors/node/

npm_mirror: https://npm.taobao.org/mirrors/npm/
   
## grunt

## gulp

是一个基于流的(构建工程中不需要生成中间文件)项目构建工具。

使用gulp来构建项目，需要一个gulpfile.js文件。

### 问题

- Gulp AssertionError [ERR_ASSERTION]: , how to specify task function?

解决办法：gulp3 和gulp4在task添加上有区别，后者需要用gulp.series()或者gulp.parallel()

参考：https://stackoverflow.com/questions/54677485/gulp-assertionerror-err-assertion-how-to-specify-task-function


- concat 的时候，不同js文件之间依赖顺序问题

https://stackoverflow.com/questions/21961142/concat-scripts-in-order-with-gulp

### 安装

npm install gulp --save-dev; 这是安装gulp包，--save-dev是填加到package.json的dev选项

npm install gulp-cli;安装gulp 命令行工具



## uglify

代码压缩工具

## babel

npm install gulp-babel @babel/core @babel/preset-env --save-dev

编译es6到es5

## jshint

静态JavaScript代码检查工具，从jslint演化而来。

### 说明

对浏览器内置对象，比如XMLHttpRequest会报错，“line 15, col 19, 'XMLHttpRequest' is not defined.”

需要增加在脚本开头增加“/* jshint browser: true */”

## 通过Node.js启动http服务

安装： npm install http-server -g

启动：http-server -p 10086 --cors

注意，启动服务的时候，在你工作目录下面执行上面的命令。


## 术语

### Compiling vs Transpiling

二者都是对源码进行转换，前者是从一种级别语言到另外一种级别语言，比如，c->机器码，也就是gcc完成的工作

后者是同级别语言转换，比如，es6->es5

参考：https://stackoverflow.com/questions/44931479/compiling-vs-transpiling

### Tree shaking

摇树？在 JavaScript 中删除无用代码。

### shim vs polyfill

shim : 对 API 调用进行拦截，适配目标浏览器。比如，某个 API 在不同浏览器实现不一样，shim 可以拦截，进行适配。

polyfill: 能让某个不支持新API的浏览器，也能执行此接口。

参考：

https://stackoverflow.com/questions/6599815/what-is-the-difference-between-a-shim-and-a-polyfill

## 学习资源

promise：
http://liubin.org/promises-book/

es6入门:
http://es6.ruanyifeng.com/

模块化：
https://medium.freecodecamp.org/javascript-modules-a-beginner-s-guide-783f7d7a5fcc
