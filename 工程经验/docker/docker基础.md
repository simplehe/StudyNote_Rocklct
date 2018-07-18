## docker基础
Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。

**Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化**。

Docker是一个开源的商业产品，有社区办CE和企业版EE，一般个人开发用社区版就可以了。

### Docker安装
以下以WSL Ubuntu18.04下作为示例来安装Docker，参考文档
[Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

首先当然是确保源要稳定，比如说有人就推荐国内用中科大的源：

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn' /etc/apt/sources.list
sudo apt update
```

安装apt包：

```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```



添加官方GPG key

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

添加稳定仓库进源：

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

接下来就能安装啦！

```
sudo apt-get update
sudo apt-get install docker-ce
```

### 参考
[Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

[Docker教程](http://www.runoob.com/docker/docker-tutorial.html)
