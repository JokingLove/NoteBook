---
title: maven install
date: 2017-12-07
categories: 上古神器
tags: maven
---
## 使用maven install安装jar包到本地仓库
我们经常用遇到一个问题，那就是每次在项目更新的时候，maven 就一直卡在那不动了，等半天都等不住，尤其是在eclipse中，有时候啥也做不了就一直等着。打开进程发现，原来在下jar包的时候，一直在找，但就是下载不下来，有些就1kb，2kb的在那下载这，看着特别蛋疼。这时，我们就可以去maven中心仓库或者其他地方手动下载jar包，然后通过maven install命令，将我们下载好的jar包直接安装到本地仓库中。
<!-- more -->
#### 1、将Jar包安装到本地仓库
* 命令
```bash
mvn install:install-file -Dfile=D:\thrift-0.9.2.jar -DgroupId=org.apache.thrift  -DartifactId=libthrift -Dversion=0.9.2 -Dpackaging=jar  
```
其中：
- DgroupId和DartifactId构成了该jar包在pom.xml的坐标， 对应依赖的DgroupId和DartifactId
- Dfile表示需要上传的jar包的绝对路径
- Dpackaging 为安装文件的种类
#### 2、 上传Jar到私服  
* 命令
```bash
mvn deploy:deploy-file -DgroupId=org.apache.thrift -DartifactId=libthrift  -Dversion=1.12 -Dpackaging=jar -Dfile=D:\thrift-0.9.2.jar  -Durl=http://ip:port/nexus/content/repositories/thirdparty/  -DrepositoryId=thirdparty  
```
其中：
- DgroupId和DartifactId构成了该jar包在pom.xml的坐标， 对应依赖的DgroupId和DartifactId
- Dfile表示需要上传的jar包的绝对路径
- Durl私服上仓库的url精确地址(打开nexus左侧repositories菜单，可以看到该路径)
- DrepositoryId服务器的表示id，在nexus的configuration可以看到

-  这样我们就可以直接在我们的pom.xml文件中直接导入dependecy了，不需要去网上下载了，不会因为某几个jar包的问题，而耽误做其他的事了。
