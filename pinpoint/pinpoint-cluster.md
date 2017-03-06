# Pinpoint集群搭建



## 目的

搭建一个高可用，负载均衡的集群，以此来防止pinpoint出现单点故障，导致程序无法正常提供服务。



## 搭建思路

pinpoint集群通过三个部分来构建一个高可用，负载均衡的集群，分别是pinpoint collector集群，pinpoint web集群，以及zookeeper集群

### pinpoint collector集群

用于收集应用发送过来的调用链的数据，通过集群的形式来做到负载均衡以及高可用。



### zookeeper集群

用于维持集群状态的一致性，并做到实时同步collector的数据给web端进行展示。



### pinpoint web集群

用于展示收集到的数据，通过集群的形式来做到负载均衡以及高可用



这里我搭建了3个pinpoint collector，3个zookeeper，以及3个pinpoint web，这里为了节约服务器，我用了3台服务器，每台服务器都装了一个pinpoint collector，zookeeper以及pinpoint web。



为了方便后期的快速部署，我把写了一个dockerfile，尽可能的简化部署后的工作。在容器运行后，只需要配置ip以及集群id就行了。



## 环境配置

1. 三台时区相同的服务器

2. 每台系统均为ubuntu14.04

3. 三台服务器都部署在rancher上



注：一定要保证3台服务器的时区相同，要不然生成的容器即使时区是一样的，时间也是不一样的。不一样的时间，会造成集群连接不同步，无法连接



## 运行容器

我们这里最好在rancher上指定主机，保证3个容器不在同一台机器上，并保证时区相同。并使用links来保证3个容器之间能相互通信。



注：因为是使用links，所以容器启动的时候是按照依赖关系的顺序启动的，要不然在rancher中，可能会link不上。



### 总体配置

1. 镜像：`registry.saas.hand-china.com/tools/pinpoint-server`

2. 暴露端口：9994,9995,9996,8086



注：9995,9996端口，tcp和udp都要打开，要不然接收不到数据



### rancher配置

pinpoint1的rancher配置：

![](/images/pinpoint/pinpoint-cluster-1.png)



![](/images/pinpoint/pinpoint-cluster-2.png)

配置好了之后，点击create，等待创建成功后，配置下一个



pinpoint2的rancher配置：

![](/images/pinpoint/pinpoint-cluster-1.png)



![](/images/pinpoint/pinpoint-cluster-2.png)

配置好了之后，点击create，等待创建成功后，配置下一个



pinpoint3的rancher配置：

![](/images/pinpoint/pinpoint-cluster-1.png)



![](/images/pinpoint/pinpoint-cluster-2.png)

配置好了之后，点击create，直到创建成功



### 检查配置

1. 每个容器之间相互ping一下ip，检查是否都能ping通

2. 运行`date`命令，查看是不是所有的容器时间都相同



注：ip不是容器ip，而是rancher给容器分配的ip



## 配置hosts

在每台主机上修改host文件，下面的ip换成容器的ip，写入容器ip，容器id，以及连接的hbase集群id都写进文件当中

```

vi /etc/hosts



#写入容器ip

10.42.70.86      pinpoint1

10.42.58.146    pinpoint2

10.42.33.125    pinpoint3



#写入hbase集群ip

172.20.1.20      master

172.20.1.19      slave1

172.20.0.179    slave2



#写入容器id

10.42.70.86     a38f9755b4d2

10.42.58.146    4a698482cc97

10.42.33.125    3197f6c8e429



```

注：如果pinpoint容器和hbase容器在同一台宿主机上，那么就不要用宿主机的ip，而是用hbase的容器ip来替换。因为容器不能连接宿主机的端口。



## 配置运行zookeeper

### 配置myid文件

在我们配置的dataDir指定的目录下面，创建一个myid文件，里面内容为一个数字，用来标识当前主机，`conf/zoo.cfg`文件中配置的server.X中X为什么数字，则myid文件中就输入这个数字，例如：

```

root@pinpoint1:~ echo "0" > /usr/data/zookeeper-3.4.6/data/myid

root@pinpoint2:~ echo "1" > /usr/data/zookeeper-3.4.6/data

/myid

root@pinpoint3:~ echo "2" > /usr/data/zookeeper-3.4.6/data

/myid

```

###运行zookeeper

分别在3个容器中都执行下列命令

进入zookeeper根目录

```

cd /usr/software/zookeeper

```



启动zookeeper

```

bin/zkServer.sh start(status, stop)

```



然后使用下面的命令，检查每台机子的zookeeper是否启动成功

```

bin/zkServer.sh status

```



## 配置运行pinpoint collector

分别在3个容器中都执行下列命令

进入pinpoint-collector根目录

```

cd /usr/software/pinpoint-collector

```



修改hbase集群ip地址

```

vim webapps/ROOT/WEB-INF/classes/hbase.properties

```

修改第一行`hbase.client.host`的值，用ip用逗号隔开



修改zookeeper集群地址

```

vim webapps/ROOT/WEB-INF/classes/pinpoint-collector.properties

```

修改`cluster.enable`值为`true`

修改`cluster.zookeeper.address`值为zookeeper集群ip地址，用逗号隔开



运行pinpoint-collector

```

bin/startup.sh

```



查看运行日志

```

 tail -f logs/catalina.out

```

没有报错就没有问题



## 配置运行pinpoint web

分别在3个容器中都执行下列命令

进入pinpoint-collector根目录

```

cd /usr/software/pinpoint-web

```



修改hbase集群ip地址

```

vim webapps/ROOT/WEB-INF/classes/hbase.properties

```

修改第一行`hbase.client.host`的值，用ip用逗号隔开



修改zookeeper集群地址

```

vim webapps/ROOT/WEB-INF/classes/pinpoint-web.properties

```

修改`cluster.enable`值为`true`

修改`cluster.zookeeper.address`值为zookeeper集群ip地址，用逗号隔开



运行pinpoint-collector

```

bin/startup.sh

```



查看运行日志

```

 tail -f logs/catalina.out

```



查看界面

在浏览器上输入ip加28086端口，可以查看最后的结果



注：为了达到负载均衡的作用，应该还要使用nginx反向代理pinpoint-collector集群，但这个并不是我们讲解的重点，大家可以查看nginx的文章。





## 总结

1. pinpoint collector集群在官网中是使用了DNS做负载均衡的，但是我们这里用的是nginx，考虑到collector崩溃的可能性不大，所以这里只有一台在使用。