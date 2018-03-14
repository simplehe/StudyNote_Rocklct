## Gulp基础
Gulp 是一个自动化工具(**就是一个工具链**)，前端开发者可以使用它来处理常见任务：

 - 搭建web服务器
 - 文件保存时自动重载浏览器
 - 使用预处理器如Sass、LESS
 - 优化资源，比如压缩CSS、JavaScript、压缩图片

类似Gulp的工具，我们通常称之为构建工具。如今最流行的两个构建工具是Gulp和Grunt。

主要区别是你如何使用他们构建自动化工作流。与Grunt相比，Gulp更加简洁，执行效率更高。

让我们继续学习Gulp并搭建一个简单的工作流。

在学习前，先谈谈大致使用gulp的步骤，给读者以初步的认识。首先当然是安装nodejs，通过nodejs的npm全局安装和项目安装gulp，其次在项目里安装所需要的gulp插件，然后新建gulp的配置文件gulpfile.js并写好配置信息（定义gulp任务），最后通过命令提示符运行gulp任务即可。

<a>http://www.ydcss.com/archives/18</a>

用npm安装插件，注意npm的后面 -g是全局 --save-dev是指在当前工程安装

这里，我们使用—-save-dev来更新package.json文件，更新devDependencies值，以表明项目需要依赖gulp。

gulp会将css文件，js文件做一堆编译压缩处理，最后根据依赖合成一个文件输出。

教程
<a>https://markpop.github.io/2014/09/17/Gulp%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B/</a>

gulp基于流操作，api极少


简而言之就是写一个gulpfile文件，内有任务清单，然后直接命令行执行命令，就会自动化执行任务。


### Browserify
brwserify用来打包CommonJs模块，使得它可以在浏览器中运行node模块。打包后的js文件可以直接运行在浏览器环境中

### gulp-concat
用来合并js文件

### gulp-react 
JSX语法转化工具,相当于之前的broswer.js。


### gulp-connect 
在做前端开发的时候，可能也需要把项目放到本地的服务器上去运行。gulp-connect 这个插件可以很容易创建一个本地的服务器去运行我们的项目。
ulp.task('server', function () {
connect.server();
});

在这个方法里面，可以去做一些设置，比如服务器的主机名，端口号，服务器的根目录等等 .. 给它一个对象 .. 然后设置一下 root 这个选项 ... 它的值就是服务器的根目录 ... 这里我们设置成 dist 这个目录 ..

connect.server({root: 'dist'});

打开一个浏览器 ... 输入 localhost:8080 .. 回车 .. 这里显示的就是服务器根目录下面的 dist 目录下面的静态文件 

