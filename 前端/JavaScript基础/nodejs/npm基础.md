## npm使用及理论基础
NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：

 - 允许用户从NPM服务器 **下载别人编写的第三方包到本地使用**。
 - 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
 - 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

### --save
npm install --save会把包安装并且条目写入package.json.

同理还有--save-dev

### npm run XXX 配置
npm run XXX是执行配置在package.json中的脚本，比如：

``` javascript
"scripts": {
    "dev": "node build/dev-server.js",
    "build": "node build/build.js",
    "unit": "karma start test/unit/karma.conf.js --single-run",
    "e2e": "node test/e2e/runner.js",
    "test": "npm run unit && npm run e2e",
    "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs"
  },
```

要了解这些命令做了什么，就要去scripts中看具体执行的是什么代码。这里就像是一些命令的快捷方式，免去每次都要输入很长的的命令（比如unit那行）。

### npm start [-- <args>]
相当于自动运行 node server.js

**命令问题多查阅文档。**

### npm package.json中的dependencies和devDependencies的区别
一个node package有两种依赖，一种是dependencies一种是devDependencies，其中前者依赖的项该是正常运行该包时所需要的依赖项，而后者则是开发的时候需要的依赖项，像一些进行单元测试之类的包。

如果你将包下载下来在包的根目录里运行

npm install
默认会安装两种依赖，如果你只是单纯的使用这个包而不需要进行一些改动测试之类的，可以使用

npm install --production
只安装dependencies而不安装devDependencies。

如果你是通过以下命令进行安装

npm install packagename
那么只会安装dependencies，如果想要安装devDependencies，需要输入

npm install packagename --dev  

### package-lock.json
在执行npm install的时候，有时会在当前目录里生成一个package-lock.json文件。

这个文件，记录了 **当前实际安装的各个pacakge来源和版本**。

要注意的，是package.json文件中，定义的依赖条目可能是这样的

```
"dependencies": {
"@types/node": "^8.0.33",
}
```

这里可以看到有个^符号，这个符号表示向后兼容，也就是说版本超过8.0.33就行，实际可能下载的是8.0.35.

大多数时候，这样都没问题。但有时，下载最新的可能会导致一些不兼容的问题。因此npm提供了生成lock文件的功能，方便保存快照，之后如果遇到问题，根据这个lock文件下载具体版本即可。
