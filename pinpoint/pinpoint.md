# pinpoint



## pinpoint介绍



### 背景（APM定义）



> In the fields of information technology and systems management, Application Performance Management (APM) is the monitoring and management of performance and availability of software applications. APM strives to detect and diagnose complex application performance problems to maintain an expected level of service. APM is “the translation of IT metrics into business meaning.”







APM监控和管理软件应用的性能和可用性，探测和诊断复杂的应用性能问题。



### pinpoint说明



> Pinpoint provides a solution to help analyze the overall structure of the system and how components within them are interconnected by tracing transactions across distributed applications.



Install agents without changing a single line of code



Minimal impact on performance (approximately 3% increase in resource usage)







Pinpoint能帮助分析整个系统架构，在分布式应用中通过追踪事务解释系统组件间是怎么进行连接的。







### pinpoint架构



![pinpoint架构图](https://github.com/TimeStarRipple/note/raw/master/%E5%88%86%E5%B8%83%E5%BC%8F/images/pinpoint1.png)



如上图，Pinpoint由4部分组成：



1. Pinpoint Agent：放在监听的应用服务器当中，对应用进行监听，并发送数据给collector
2. Pinpoint Collector：接收从agent发出的数据，并进行汇总，写入数据库
3. Hbase：存储从collector发送过来的数据，对数据进行持久化
4. Pinpoint Web UI：将hbase中的数据提取出来，在界面中展示







其中2,4必须使用web容器。



Pinpoint使用了字节码增强技术，在类加载时，注入到观察的应用Class文件中。



所以在启动应用时，必须设置-javaagent为Pinpoint的agent。







### pinpoint支持的模块



- JDK 6+

- Tomcat 6/7/8, Jetty 8/9, JBoss EAP 6

- Spring, Spring Boot

- Apache HTTP Client 3.x/4.x, JDK HttpConnector, GoogleHttpClient, OkHttpClient, NingAsyncHttpClient

- Thrift Client, Thrift Service, DUBBO PROVIDER, DUBBO CONSUMER

- MySQL, Oracle, MSSQL, CUBRID, DBCP, POSTGRESQL, MARIA

- Arcus, Memcached, Redis, CASSANDRA

- iBATIS, MyBatis

- gson, Jackson, Json Lib

- log4j, Logback





## pinpoint快速安装（以1.6.0版本为例）

> 采用快速安装的方式，能看到一个简单的demo，了解pinpoint是如何使用的。但是，它仅仅是一个demo，只能监控自己的testapp，不能监控其它应用，在实际应用中，还是要自己完整安装一个pinpoint。



### 环境要求

- JDK 6 installed

- JDK 7 installed

- JDK 8 installed

- Maven 3.2.x+ installed

- JAVA_6_HOME environment variable set to JDK 6 home directory.

- JAVA_7_HOME environment variable set to JDK 7 home directory.

- JAVA_8_HOME environment variable set to JDK 8+ home directory.

安装的系统可以是linux，Window，OSX



### 下载编译pinpoint

有两种方式

- 下载源码并编译

- 下载docker镜像（里面已经帮你把环境配好了，源码下好了，程序编译了）



##### 下载源码并编译



1. 下载源码
可以使用`git clone https://github.com/naver/pinpoint.git`克隆项目代码，也可以直接下载zip格式的[源码](https://github.com/naver/pinpoint/archive/master.zip)使用unzip命令直接解压缩得到

2. 编译源码
进入源码第一子目录下，使用maven指令`mvn clean install -Dmaven.test.skip=true`进行项目编译



**注：这里如果maven不是3.2以上的话会报错的；

一定要注意这里要下很多jar包，网速不好甚至要下一天，也可能下载失败，要自己翻墙下载，然后移进仓库当中，注意翻墙下载的和所需的jar包名字可能不大一样，要改一下**



##### 下载docker镜像（目前镜像最新的是1.5.2版本的pinpoint）

1. 安装[docker](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
2. 从docker hub中下载[pinpoint](https://hub.docker.com/r/yous/pinpoint/)



### 下载运行HBase

> pinpoint可以直接运行命令行下载安装Hbase，但windows系统必须自己下载安装Hbase



##### 手动安装

- 下载[HBase-1.2.4-bin.tar.gz](http://apache.mirror.cdnetworks.com/hbase/)并且解压它

- 更改文件名为Hbase，并移动文件夹至quickstart/hbase下

- 下载运行Hbase：`quickstart/bin/start-hbase.cmd`

- 初始化表结构：`quickstart/bin/init-hbase.cmd`



##### 使用命令

在源码第一层子目录下运行下列命令

- 下载运行Hbase：`quickstart/bin/start-hbase.sh`

- 初始化表结构：`quickstart/bin/init-hbase.sh`



### 运行pinpoint

在源码第一层子目录下运行下列命令

- 运行数据收集器Collector：`quickstart/bin/start-collector.sh`

- 运行测试应用TestApp：`quickstart/bin/start-testapp.sh`

- 运行数据显示界面Web：`quickstart/bin/start-web.sh`



命令运行成功后的界面如下：

Collector

![收集器运行成功界面](https://github.com/TimeStarRipple/note/raw/master/%E5%88%86%E5%B8%83%E5%BC%8F/images/ss_quickstart-collector-log.png)



TestApp

![测试应用运行成功界面](https://github.com/TimeStarRipple/note/raw/master/%E5%88%86%E5%B8%83%E5%BC%8F/images/ss_quickstart-testapp-log.png)



Web

![数据显示运行成功界面](https://github.com/TimeStarRipple/note/raw/master/%E5%88%86%E5%B8%83%E5%BC%8F/images/ss_quickstart-web-log.png)

注：以上所有应用启用，代码都给了180秒的启动时间，如果180秒内，没有检测到启动成功，程序会杀死启动进程，如果自己通过查看日志发现其实程序启动成功了，但是超过了180s被杀死了，可以注释掉相应`.sh`文件倒数第9行的kill指令，这样程序就能运行成功。



### 查看应用

- 查看收集的数据信息Web UI：http://localhost:28080

- 查看当前监听了哪些应用Web：http://localhost:28080/stat.html

- 查看TestApp：http://localhost:28081



### 停止应用

- Web UI - Run `quickstart/bin/stop-web.sh`

- TestApp - Run `quickstart/bin/stop-testapp.sh`

- Collector - Run `quickstart/bin/stop-collector.sh`

- HBase - Run `quickstart/bin/stop-hbase.sh`



## pinpoint安装配置（以1.6.0为例）

> 以下配置全部基于ubuntu



### 安装配置内容

- HBase (作为数据库)

- Pinpoint Collector (部署在一个web容器当中)

- Pinpoint Web (部署在一个web容器当中)

- Pinpoint Agent (和监控的应用放在一起)



其中hbase和collector以及web部署在一台机器上，监控的应用部署在其他机子上



### 安装要求

1. jdk1.8

2. hbase1.2



### 下载配置环境

jdk

```

wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u112-b15/jdk-8u112-linux-x64.tar.gz



#环境变量配置

export JAVA_HOME="/usr/lib/jvm/jdk1.8.0_112"

export PATH=$JAVA_HOME/bin:$PATH:$JAVA_HOME/jre/bin

export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tool.jar:$JAVA_HOME/jre/lib

```



tomcat

```

wget http://archive.apache.org/dist/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz

```



### Hbase安装

>pinpoint1.6.0要求Hbase的版本为1.2



##### 下载解压HBase

使用wget命令下载HBase并解压

```

wget http://apache.mirror.cdnetworks.com/hbase/1.2.4/hbase-1.2.4-bin.tar.gz

```



##### 编辑hbase-env.sh文件

```

# The java implementation to use.  Java 1.7+ required.

export JAVA_HOME="/usr/lib/jvm/jdk1.8.0_112"

```



#####运行hbase

进入hbase目录，运行bin下面的start-hbase.sh

```

bin/start-hbase.sh

```

**注：可以通过Jps命令验证是否启动成功，如出现HMaster表示启动成功**



##### 初始化表

使用wget命令下载初始化脚本

```

wget https://raw.githubusercontent.com/naver/pinpoint/master/hbase/scripts/hbase-create.hbase

```

在HBase目录下，运行初始化脚本

```

bin/hbase shell hbase-create.hbase

```

再检查是否表初始化成功

```

#进入hbase的shell窗口

bin/hbase shell

#查看表

list

```





### 下载pinpoint

有两种方法

- 直接下载打包后的war包

- 下载源码，编译后打包成war包（编译太浪费时间，不建议使用）



##### 直接下载

```

wget https://github.com/naver/pinpoint/releases/download/1.6.0/pinpoint-agent-1.6.0.tar.gz



wget https://github.com/naver/pinpoint/releases/download/1.6.0/pinpoint-collector-1.6.0.war



wget https://github.com/naver/pinpoint/releases/download/1.6.0/pinpoint-web-1.6.0.war

```



将web和collector分别放到web容器下，最好两个容器，我将web的端口设为8086，collector的端口设为8085



##### collector配置

替换Tomcat容器中的ROOT项目为pinpoint-collector项目，最好把Tomcat的名字改成collector相关的名字，这样也好辨认，然后修改server.xml文件的端口配置，以防和后面的web相冲突

```

$ mkdir -p /data/service/

$ tar xf /root/pp/apache-tomcat-8.0.32.tar.gz -C /data/service/;cd /data/service/

$ mv apache-tomcat-8.0.35/ pinpoint-collector

$ vim /data/service/pinpoint-collector/conf/server.xml       # 因为我们的pp-collector和pp-web部署在同台设备，所以请确认tomcat启动端口不会冲突

<Server port="8005" shutdown="SHUTDOWN">

<Connector port="8085" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />

<!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->     #  注释该行

```

这里大概说一下: applicationContext-collector.xml, applicationContext-hbase.xml 这俩个配置文件时collector与agent和Hbase之间通信时需要设定的一些参数，在后续调优的时候需要用到，hbase.properties 主要是设定后端存储的连接配置，log4j.xml那就是log相关了。



##### web配置

和上面的一样，需要另外配置一个Tomcat容器部署web，替换掉容器中的ROOT项目，并修改server.xml文件

```

$ cd /data/service/

$ tar xf /root/pp/apache-tomcat-8.0.35.tar.gz -C /data/service/

$ mv apache-tomcat-8.0.35 pinpoint-web

$ cd pinpoint-web/webapps/;rm -rf *;mkdir ROOT;cd ROOT/

$ unzip /root/pinpoint-web-1.5.2.war

$ vim /data/service/pinpoint-web/conf/server.xml       # 因为我们的pp-collector和pp-web部署在同台设备，所以请确认tomcat启动端口不会冲突

<Server port="8006" shutdown="SHUTDOWN">      

<Connector port="8086" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />

<!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->     #  注释该行

```

这里说明一下:



- hbase.properties 配置我们pp-web从哪个数据源获取采集数据，这里我们只指定Hbase的zookeeper地址。

- jdbc.properties pp-web连接自身Mysql数据库的连接认证配置。

sql目录 pp-web本身有些数据需要存放在MySQL数据库中，这里需要初始化一下表结构。

- pinpoint-web.properties 这里pp-web集群的配置文件，如果你需要pp-web集群的话。

- applicationContext-*.xml 这些文件在后续的调优工作中会用到。

- log4j.xml 日志相关配置。



##### agent集成到应用



将agent放到app所在的服务器，然后再启动参数中加入下面的命令（springboot项目可以采用这种方式）

```

java  -javaagent:$AGENT_PATH/pinpoint-bootstrap-$VERSION.jar -Dpinpoint.agentId=$AGENT_ID  -Dpinpoint.applicationName=$APPLICATION_NAME

```

- AGENT_PATH：agent存放的位置，

- VERSION：我用的是1.6.0

- AGENT_ID ：自定义。就是一个唯一标识

- APPLICATION_NAME：自定义



如果是tomcat的话修改catalina.sh，添加-javaagent, -Dpinpoint.agentId, -Dpinpoint.applicationName 到 CATALINA_OPTS 参数里去。如下：

```

#pinpoint agent路径

CATALINA_OPTS="$CATALINA_OPTS -javaagent:/data1/hugang/pinpoint/pinpoint-agent-1.6.0/pinpoint-bootstrap-1.6.0.jar"



#被监控工程使用agent的标识号

CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=0000002"



#被监控工程名字，要和tomcat里面的项目名对应

CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=10.75.0.101_8086_自定义"

```



##### 应用测试



Web UI: http://WEB_IP:8086



App: http://APP_IP:8080



在应用中点击按钮来发送请求，我们能看到大体下图的样子：

![](/images/pinpoint/pinpoint_server-map.png)



## 学习总结



#### 经验



1. 目前使用的版本只能支持到jdk1.7 需要在path中建JAVA_7_HOME这个环境变量
2. 我将hbase当中的测试数据全部清除后, 然后调用被监控的接口，然后前端总是不显示agent的application信息，然后去查询agentinfo表，确实没有数据，但是，agentStatus当中的表都有了数据，只有agentInfo表怎么都没有数据....折腾半天原来被监控的服务需要重启，才能将agent信息重新注册到hbase中。。。。。。
3. 清除了hbase的data文件之后，通过hbase的list命令返回为空库，但是重新初始化pinpoint的表结构时，提示：表已存在，，，，，很奇怪，最后发现，虽然删除了hbase的表数据，但是hbase的表都注册到zk中了，所以需要把zk中的数据也要清理掉......
4. 所有数据正常显示，只有CPU数据没有显示，是因为公司镜像里面的jdk使用的是openjdk，pinpoint默认不支持，需要在agent的pinpoint.config中加入一个参数`profiler.jvm.vendor.name=Oracle`就行了
5. Hbase的standalone模式需要运行在127.0.0.1上，你可以去/etc/hosts里检查一下，看看你的用户名所对应的IP是不是127.0.0.1，像ubuntu这种linux一般来说是会把你的IP设置成127.0.1.1的
6. 必须保证监控应用，collector和web时间同步，否则数据可能显示不出来，如果采用dockerfile的话，可以使用如下命令更换时区，保证时间相同。但是，可能用了以下命令还是没有用，因为虚拟机时间不对，还需要调整一下。
7. 在使用IDE运行Springboot项目的时候，项目一般都是采用tomcat运行的，agent无法识别项目是springboot项目，可以在`pingpoint.config`中配置强制指定当前项目是springboot，`profiler.applicationservertype=SPRING_BOOT`，由于采用tomcat容器运行项目，但是是一个springboot项目，所以这里不能使用tomcat注册类，要加上配置`profiler.tomcat.conditional.transform=false`才可以，详细可以查看https://github.com/naver/pinpoint/issues/2616
8. User Group这个功能在官网发布的安装版是没有的，需要自己下源码进行编译，自己构建一个maven项目，继承接口来二次开发，详细可以查看https://github.com/naver/pinpoint/issues/1878
9. 调用链请求丢失了很多时，可以调整agent中pinpoint.config的`profiler.sampling.rate`配置，这个配置代表的是请求的采样率，默认是20也就是5%，可以改为1，就是收集100%的请求

```
ENV TZ=Asia/Shanghai

RUN ln -snf /usr/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

## 参考文档

- pinpoint介绍：
https://github.com/naver/pinpoint

- Demo安装：
https://github.com/naver/pinpoint/blob/master/quickstart/README.md

- 完整安装：
https://github.com/naver/pinpoint/blob/master/doc/installation.md

- 分布式完整安装：
https://sconts.com/11
