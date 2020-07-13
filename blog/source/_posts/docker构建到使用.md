

#### 一、操作环境

centos7.x系列

docker17.x系列

```
uname -r 查看linux内核版本
cat /etc/redhat-release 查看centos版本
```

<!--more-->

#### 二、安装Docker

```
curl https://releases.rancher.com/install-docker/17.03.sh | sh
docker version
```



#### 三、docker的操作

##### 3.1 查看容器和镜像

执行docker ps或者docker images进行查看容器或者镜像

##### 3.2 下载一个镜像

执行docker pull 镜像名 即可下载一个镜像，例如：docker pull redis

##### 3.3 创建一个tomcat运行环境

	###### 	3.3.1 上传java和tomcat

​		分别下载安装包，上传到服务器

```
apache-tomcat-9.0.0.M21.tar.gz
jdk-8u131-linux-x64.tar.gz
```

	###### 	3.3.2 解压到指定的目录

​		在Docker宿主机上创建了一个tomcat-docker目录，把解压好的jdk和tomcat放到该目录

###### 	3.3.3 创建用来生成镜像的Dockerfile文件

​		Vi Dockerfile内容如下，之后保存

```
# 设置继承的镜像
FROM ubuntu:16.04
# 创建者信息
MAINTAINER wxj "kingstudy@vip.qq.com"
# 设置环境变量，所有操作都是非交互式的
ENV DEBIAN_FRONTEND noninteractive
# 设置tomcat的环境变量
ENV CATALINA_HOME /tomcat
ENV JAVA_HOME /jdk
# 复制tomcat和jdk文件到镜像中
ADD apache-tomcat-9.0.0.M21 /tomcat
ADD jdk1.8.0_131 /jdk
# 复制启动脚本至镜像，并赋予脚本可执行权限
ADD run.sh /run.sh
RUN chmod +x /*.sh
RUN chmod +x /tomcat/bin/*.sh
# 暴露接口8091，这是我的tomcat接口，默认为8080
EXPOSE 8091
# 设置自启动命令
CMD ["/run.sh"]
```

 	###### 	3.3.4创建自动启动文件run.sh

在创建Dockerfile的时候我们可以发现最后的一行设置自启动命令里面指定了/run.sh

这里的’/’是根目录的意思，和Linux操作系统的’/’目录有所区别，只要Dockerfile和run.sh在同一个目录下这个地方就可以用/来指定，下面我们在tomcat-docker路径下执行touch run.sh创建一个run.sh文件,内容如下

```
#!/bin/bash
# 启动tomcat
exec ${CATALINA_HOME}/bin/catalina.sh run
```

run.sh的作用是在启动容器的时候启动tomcat服务

###### 	3.3.5 根据Dockerfile生成镜像

注意：这个命令一定要在tomcat-docker这个目录下执行才可以，因为里面有复制文件的操作用的是相对目录

```
docker build -t tomcat:test1 -f /root/tomcat-docker/Dockerfile .
```



	###### 	3.3.6 用生成的tomcat镜像来启动一个容器

执行如下命令

```
docker run -d -p 50080:8080 tomcat:test1
```

-p是指定宿主主机和容器的端口映射， 用宿主主机的50080端口映射容器的8080端口

	###### 	3.3.7 **测试容器是否部署成功**

容器启动以后，我们就可以通过宿主主机来访问tomcat容器了，由于我们启动容器的时候做了端口映射所以访问的地址为：

http://192.168.1.51:50080/









