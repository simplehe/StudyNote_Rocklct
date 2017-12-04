## maven打包jar与发布jar到本地仓库

### 打包jar
假如我们想打包我们的maven项目成为一个jar包，根据maven的构建过程。我们可以执行命令

```
mvn clean package
```

也可以在idea中，在maven窗口找到package的lifecycle然后执行。

### 发布jar到本地仓库
有时候我们想要手动添加jar依赖，这个依赖可能是网上下载的，也可能是自己打包的。

添加的命令是

```
mvn install:install-file -Dfile=my-jar.jar -DgroupId=org.richard
-DartifactId=my-jar -Dversion=1.0 -Dpackaging=jar
```

进入到jar包所在目录，执行这个cmd命令。其中参数也要调对，DgroupID是groupid，artifactid verison，这些都需要在pom里配置正确才能读取。

当然也可以在idea中，project structure里找到dependency选项，里面找到local maven jar包的位置，然后手动复制jar包到那个目录也可以。
