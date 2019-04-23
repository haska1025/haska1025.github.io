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

## nvm (node version manager)

## grunt

## gulp

是一个基于流的(构建工程中不需要生成中间文件)项目构建工具。

使用gulp来构建项目，需要一个gulpfile.js文件。

### 安装

npm install gulp --save-dev; 这是安装gulp包，--save-dev是填加到package.json的dev选项

npm install gulp-cli;安装gulp 命令行工具

## 学习资源

promise：
http://liubin.org/promises-book/

es6入门:
http://es6.ruanyifeng.com/

