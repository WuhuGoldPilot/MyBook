<!--
 * @Author: jhliang
 * @Date: 2020-11-19 17:24:52
 * @LastEditTime: 2020-11-26 17:53:19
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \MyBook\Kubernetes\Docker命令.md
-->

# Docker操作命令

## 帮助命令

````shell
docker version #查看版本

docker info #更详细

docker 命令 --help  #帮助命令
````

## 镜像命令

### 查看本地镜像

````shell
docker images
````

#### 搜索镜像

````shell
docker search
````

#### 拉取镜像

````shell
docker pull 镜像名[:tag] #联合文件系统
````

#### 删除镜像

````shell
docker rmi #删除镜像
````

#### 进阶操作

````shell
#骚操作：删除所有镜像
docker rmi -f $(docker images -aq)
````

## 容器命令

### 容器运行

````shell
### 核心命令：docker run [OPTION] image

#OPTIONS:

-name  #容器命名

-d  #后台运行

-e #环境配置

-it  #使用交互方式运行，进入容器查看内容

-P  #(大写P)随机指定端口

-p  #指定容器端口(端口映射)

  -p 主机端口:容器端口

  -p 容器端口
-v 本地目录:容器目录 #挂载，数据卷技术
# 匿名挂载 -v 容器目录
````

#### 查看容器

````shell
docker ps [OPTION]#查看容器

OPTIONS:

-a #历史所有运行过的容器

-n=? #显示最近创建的容器个数

-q #只显示容器的编号
````

#### 退出容器

````shell
exit #直接容器退出
Ctrl + P + Q #容器不停止退出
````

#### 删除容器

````shell
docker rm 容器id #删除指定容器
#骚操作：删除所有容器
docker rm -f $(docker ps -aq)
docker ps -a -q|xargs docker rm
````

#### 启动和停止

````shell
docker start 容器id  #启动容器
docker restart 容器id  #重启
docker stop 容器id  #停止
docker kill 容器id  #强制停止
````

## 其他命令

### 后台启动容器

````shell
docker run -d [IMAGE]
#必须要有一个前台进程，centos没前台应用就会自动停止
````

#### 查看日志

````shell
docker logs [OPTIONS] 容器id
````

#### 查看进程信息

````shell
docker top [OPTIONS] 容器id
````

#### *查看容器元信息

````shell
docker inspect [OPTIONS] 容器id
````

#### *进入当前正在运行的容器

````shell
#方式一 进入容器后开启新的终端，可以在里面操作
docker exec -it 容器id bashshell
#方式二 进入正在执行终端，不会启动新的进程
docker attach 容器id
````

#### 从容器内拷贝文件

````shell
docker cp 容器id:容器内路径 目的主机路径
````

#### 查看cpu状态

````shell
docker stats
````

#### commit自己的镜像

````shell
docker commit -m="描述信息" -a="作者" 容器id 目标镜像:[TAG]
````

## 容器数据卷

## DockerFile

### Dockerfile组成

````shell
FROM [image]  #基础镜像
MAINTAINER usr<mail> #维护者信息：姓名+邮箱
RUN commend   #镜像构建的时候需要运行的命令
ADD    #步骤，添加的内容。如tomcat镜像
WORKDIR   #镜像的工作目录
VOLUME   #数据卷，挂载
EXPOSE   #暴露端口配置 -p
CMD    #指定这个容器启动时要运行的命令,只有最后一个会生效,可被替换
ENTRYPOINT  #指定这个容器启动时要运行的命令,可以追加命令
ONBUILD   #当构建一个被继承的DockerFile的时候，这个时候被运行ONBUILD指令
COPY   #类似ADD命令，将文件拷贝到镜像中
ENV    #构建的时候设置环境变量
````

````dockerfile
#99%的镜像都是基于这个镜像scratch构建的
FROM scratch
````

#### 构建命令

````shell
vi Dockerfile
````

````dockerfile
#Dockerfile配置参考
FROM centos
MAINTAINER joker<2246473845@qq.com>
RUN yum -y install vim
RUN yum -y install net-tools
WORKDIR /usr/local
VOLUME ["joker","lisa"]
EXPOSE 90
CMD echo "hello centos"
CMD echo "----end----"
CMD /bin/bash
````

````shell
#最后的点一定不能少
docker build -f [Dokerfile] -t [imageName] .
````

#### sprintboot简单配置

````dockerfile
FROM java:8
#将当前目录的项目jar包copy到容器中
COPY demo.jar /app.jar
#启动后执行 java -jar app.jar
ENTRYPOINT ["java","-jar","app.jar"]
暴露端口
EXPOSE 8081
````

## Docker网络

### docker网络默认为桥接模式

````shell
#bridge为内置的虚拟网卡，名字为docker0
[root@izwz96gwsbqox1uu62jzfwz ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
87a5216a25ff        bridge              bridge              local
ed0d54a35682        host                host                local
a52ce7ad7439        jokernet            bridge              local
d6cd81f78f67        none                null                local
````

````shell
[root@izwz96gwsbqox1uu62jzfwz /]# docker network --help

Usage: docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
````

#### 自定义网络

````shell
docker network create [OPTION] NETWORK

#example
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.0.0.1 mynet
````

## portainer（docker可视化界面）

````shell
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
````

## 冷知识

阿里云服务器上tomcat启动慢，解决方法

````shell
yum install rng-tools #安装rngd服务（熵服务）
systemctl start rngd #启动服务
#如果你的CPU不支持DRNG特性或者像我一样使用虚拟机，可以使用/dev/unrandom来模拟。
````

这样安装了熵服务后问题就解决了，Tomcat启动又回到了以往的速度。
