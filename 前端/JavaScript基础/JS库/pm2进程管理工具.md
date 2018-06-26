## pm2
PM2是node进程管理工具，可以利用它来简化很多node应用管理的繁琐任务，如性能监控、自动重启、负载均衡等，而且使用非常简单。

pm2是目前最常见的线上部署项目工具之一。其主要特性：

 - 内建负载均衡（使用Node cluster 集群模块）
 - 后台运行
 - 0秒停机重载
 - 具有Ubuntu和CentOS 的启动脚本
 - 停止不稳定的进程（避免无限循环）
 - 控制台检测
 - 提供 HTTP API
 - 远程控制和实时的接口API ( Nodejs 模块,允许和PM2进程管理器交互 )


### 对比nohup

```
$ nohup node /home/zhoujie/ops/app.js &
[1] 31490nohup: ignoring input and appending output to `nohup.out'
```

即此时程序已启动，直接访问即可，原程序的的标准输出被自动改向到当前目录下的nohup.out文件，起到了log的作用。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。nohup就是不挂起的意思( no hang up)。

这个不太靠谱，经常默默的进程在后台就挂了

### pm2使用
安装后即可用

```
npm install -g pm2
```

```
$ pm2 start app.js
[PM2] Spawning PM2 daemon
[PM2] Success
[PM2] Process app.js launched
┌──────────┬────┬──────┬─────┬────────┬───────────┬────────┬─────────────┬──────────┐
│ App name │ id │ mode │ PID │ status │ restarted │ uptime │      memory │ watching │
├──────────┼────┼──────┼─────┼────────┼───────────┼────────┼─────────────┼──────────┤
│ app      │ 0  │ fork │ 308 │ online │         0 │ 0s     │ 21.879 MB   │ disabled │
└──────────┴────┴──────┴─────┴────────┴───────────┴────────┴─────────────┴──────────┘
 Use `pm2 info <id|name>` to get more details about an app
```

终止程序也很简单：pm2 stop

```
pm2 stop app.js
```

列举出所有用pm2启动的程序：pm2 list

```
$ pm2 list
┌──────────┬────┬──────┬─────┬────────┬───────────┬────────┬─────────────┬──────────┐
│ App name │ id │ mode │ PID │ status │ restarted │ uptime │      memory │ watching │
├──────────┼────┼──────┼─────┼────────┼───────────┼────────┼─────────────┼──────────┤
│ app      │ 0  │ fork │ 984 │ online │         1 │ 3s     │ 64.141 MB   │ disabled │
└──────────┴────┴──────┴─────┴────────┴───────────┴────────┴─────────────┴──────────┘
 Use `pm2 info <id|name>` to get more details about an app
```

查看pm2启动程序信息用`pm2 describe id`,还能启动一个监控页面`pm2 web`
