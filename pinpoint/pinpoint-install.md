# pinpoint docker install

## 安装条件
一台安装了docker的ubuntu的主机

## pinpoint安装
### docker 安装

```
$ docker run -d --net=host -p 2181:2181 -p 60000:60000 -p 16010:16010 -p 60020:60020 -p 16030:16030 --name=pinpoint-hbase registry.saas.hand-china.com/tools/pinpoint-hbase-alone

$ docker run -d --net=host -e HBASE_HOST=localhost -e HBASE_PORT=2181 -e COLLECTOR_TCP_PORT=9994 -e COLLECTOR_UDP_STAT_LISTEN_PORT=9995 -e COLLECTOR_UDP_SPAN_LISTEN_PORT=9996 -p 9994:9994 -p 9995:9995/udp -p 9996:9996/udp --name=pinpoint-collector registry.saas.hand-china.com/tools/pinpoint-collector-alone

$ docker run -d -p 8080:8080 --net=host -e HBASE_HOST=localhost -e HBASE_PORT=2181 --name=pinpoint-web registry.saas.hand-china.com/tools/pinpoint-web-alone
```
[点击进入查看dockerfile](https://rdc.hand-china.com/gitlab/HAPM/hapm-integration-guide/tree/master/pinpoint/dockerfile/pinpoint-alone-cluster)

### docker-compose安装

[安装docker-compose](https://github.com/docker/compose/releases)
把[docker-compose.yml](https://rdc.hand-china.com/gitlab/HAPM/hapm-integration-guide/blob/master/pinpoint/dockerfile/pinpoint-alone-cluster/docker-compose.yml)文件移动到当前目录

在当前目录下，运行以下指令
```
docker-compose up
```

## pinpoint-agent安装

如果是要制作镜像，请依赖我的镜像制作一个镜像`registry.saas.hand-china.com/tools/pinpoint-collector-alone`，当然也可以查看我的[Dockerfile](https://rdc.hand-china.com/gitlab/HAPM/hapm-integration-guide/tree/master/pinpoint/dockerfile/pinpoint-agent)进行改写。

### Springboot项目
因为项目中使用的使用的是springboot这门技术，使用jar文件加命令行直接运行，所以在运行的时候，加上运行参数就好了

模板如下：
```
-javaagent:$AGENT_PATH/pinpoint-bootstrap-$VERSION.jar -Dpinpoint.agentId=$AGENT_ID  -Dpinpoint.applicationName=$APPLICATION_NAME
```
- $AGENT_PATH：为pinpoint agent文件的根目录，可以是相对路径也可以是绝对路径
- $VERSION：为pinpoint agent的版本
- $AGENT_ID：可以是任意字符串，必须唯一，但长度不大于24
- $APPLICATION_NAME：可以是任意字符串，必须唯一，但长度不大于24

在镜像中的实际运行参数为：
```
-javaagent:/pinpoint-agent-1.6.0/pinpoint-bootstrap-1.6.0.jar -Dpinpoint.agentId=api_1  -Dpinpoint.applicationName=hmall-api-gateway
```

### tomcat项目
如果是运行在tomcat上跑的应用，请修改`bin/catalina.sh`，添加-javaagent, -Dpinpoint.agentId, -Dpinpoint.applicationName 到 CATALINA_OPTS 参数里去。如下：

```
#pinpoint agent路径
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/pinpoint-agent-1.6.0/pinpoint-bootstrap-1.6.0.jar"

#被监控工程使用agent的标识号
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=0000001"

#被监控工程名字，要和tomcat里面的项目名对应
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=10.75.0.101_8086_自定义"
```

## 问题
如果hbase容易崩溃，除了搭建集群这个解决办法，还有一个办法就是增大hbase的内存，更改dockerfile配置中[hbase的配置](https://rdc.hand-china.com/gitlab/HAPM/hapm-integration-guide/tree/master/pinpoint/dockerfile/pinpoint-alone-cluster/pinpoint-hbase)，修改`hbase-env.sh`，调大HBASE_HEAPSIZE，HBASE_MASTER_OPTS，以及HBASE_REGIONSERVER_OPTS，这样能保证不容易崩溃hbase不容易崩溃