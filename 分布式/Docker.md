#Docker
##管理程序虚拟化（hypervisor virtualization, HV）
###定义

>通过中间层将一台或者多台独立的机器虚拟运行于物理硬件之上（个人理解：通过虚拟机等中间层虚拟一个操作系统运行于物理硬件之上）

###缺点
>虚拟机虚拟一个完整的操作系统容易被攻击，而且速度慢，开发效率低下

##容器
###定义
>直接运行在操作系统内核之上的用户空间（个人理解：使用当前操作系统的调用接口，降低了开销，但是只能运行底层宿主机相同或类似的操作系统）

###优点
>开销小，可以上千个同时在一台机子上运行

###缺点
1.  只能运行在和底层宿主机相同或相似的操作系统
2. 	容器很复杂，不易安装，管理和自动化难
3. 	相对于彻底隔绝的管理程序，虚拟出来的系统被攻击容易影响到宿主系统，不安全（有了权限和linux内核特性，此时隔离很彻底）

##Docker
###定义
>把开发的应用程序自动部署到容器的容器管理引擎

###作用
1. 把开发程序自动部署到容器当中
2. 开发程序部署到容器中，可以将他们打包起来，以镜像的方式交付，让软件运行在标准的环境当中，做到测试，运行环境相同。
3. 进行版本管理

###指令
>使用没有权限的用户执行指令的时候前面要加上sudo（system user do），代表管理员权限

#####下载docker（国内源）（请确认安装了curl）
```
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```
#####查看所有镜像
```
docker images
```
#####删除所有镜像
```
docker rmi $(docker images | grep none | awk '{print $3}' | sort -r)
```
#####拉取镜像
```
docker pull <镜像名:tag>
如：docker pull sameersbn/redmine:latest
```
#####查看正在运行的容器
```
docker ps
docker ps -a为查看所有的容器，包括已经停止的。
```
#####创建运行一个容器（以grafana为例）
```
docker run -i -p 3000:3000 grafana/grafana bash
```
备注：bash表示进入该容器
#####查看容器信息
```
docker inspect 容器ID
查看ip：docker inspect 容器ID | grep IPAddress
```
#####停止、启动、杀死一个容器
```
docker stop <容器名orID>
docker start <容器名orID>
docker kill <容器名orID>
```
#####退出当前容器
```
exit
```
#####删除容器
```
全部：docker rm $(docker ps -a -q)
单个：docker rm <容器名orID>
```
#####构建自己的镜像
```
docker build -t <镜像名> <Dockerfile路径>
如Dockerfile在当前路径：docker build -t xx/gitlab .
```

#####镜像迁移（从一台机到另一台机）
需要先保存镜像，然后拷贝到另一台机器上，再加载镜像
```
保存：docker save busybox-1 > /home/save.tar
加载：docker load < /home/save.tar
```