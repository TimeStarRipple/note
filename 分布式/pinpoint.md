#pinpoint

##pinpoint介绍

###背景（APM定义）

>In the fields of information technology and systems management, Application Performance Management (APM) is the monitoring and management of performance and availability of software applications. APM strives to detect and diagnose complex application performance problems to maintain an expected level of service. APM is “the translation of IT metrics into business meaning ([i.e.] value).”



APM监控和管理软件应用的性能和可用性，探测和诊断复杂的应用性能问题。

###pinpoint说明

>Pinpoint provides a solution to help analyze the overall structure of the system and how components within them are interconnected by tracing transactions across distributed applications.

Install agents without changing a single line of code

Minimal impact on performance (approximately 3% increase in resource usage)



Pinpoint能帮助分析整个系统架构，在分布式应用中通过追踪事务解释系统组件间是怎么进行连接的。



###pinpoint架构

![pinpoint架构图](https://github.com/TimeStarRipple/note/raw/master/%E5%88%86%E5%B8%83%E5%BC%8F/images/pinpoint1.png)

如上图，Pinpoint由4部分组成：

1. Pinpoint Agent：放在监听的应用服务器当中，对应用进行监听，并发送数据给collector

2. Pinpoint Collector：接收从agent发出的数据，并进行汇总，写入数据库

3. Hbase：存储从collector发送过来的数据，对数据进行持久化

4. Pinpoint Web UI：将hbase中的数据提取出来，在界面中展示



其中2,4必须使用web容器。

Pinpoint使用了字节码增强技术，在类加载时，注入到观察的应用Class文件中。

所以在启动应用时，必须设置-javaagent为Pinpoint的agent。



###pinpoint支持的模块

>JDK 6+
Tomcat 6/7/8, Jetty 8/9, JBoss EAP 6
Spring, Spring Boot
Apache HTTP Client 3.x/4.x, JDK HttpConnector, GoogleHttpClient, OkHttpClient, NingAsyncHttpClient
Thrift Client, Thrift Service, DUBBO PROVIDER, DUBBO CONSUMER
MySQL, Oracle, MSSQL, CUBRID, DBCP, POSTGRESQL, MARIA
Arcus, Memcached, Redis, CASSANDRA
iBATIS, MyBatis
gson, Jackson, Json Lib
log4j, Logback


##pinpoint快速安装（以1.6.0版本为例）
>采用快速安装的方式，能看到一个简单的demo，了解pinpoint是如何使用的。++但是，它仅仅是一个demo，只能监控自己的testapp，不能监控其它应用，在实际应用中，还是要自己完整安装一个pinpoint。++

###环境要求
- JDK 6 installed
- JDK 7 installed
- JDK 8 installed
- Maven 3.2.x+ installed
- JAVA_6_HOME environment variable set to JDK 6 home directory.
- JAVA_7_HOME environment variable set to JDK 7 home directory.
- JAVA_8_HOME environment variable set to JDK 8+ home directory.
安装的系统可以是linux，Window，OSX

###下载编译pinpoint
有两种方式
- 下载源码并编译
- 下载docker镜像（里面已经帮你把环境配好了，源码下好了，程序编译了）

#####下载源码并编译

1. 下载源码
可以使用`git clone https://github.com/naver/pinpoint.git`克隆项目代码，也可以直接下载zip格式的[源码](https://github.com/naver/pinpoint/archive/master.zip)使用unzip命令直接解压缩得到

2. 编译源码
进入源码第一子目录下，使用maven指令`mvn clean install -Dmaven.test.skip=true`进行项目编译
++注：
这里如果maven不是3.2以上的话会报错的；
一定要注意这里要下很多jar包，网速不好甚至要下一天，也可能下载失败，要自己翻墙下载，然后移进仓库当中，注意翻墙下载的和所需的jar包名字可能不大一样，要改一下++

#####下载docker镜像（目前镜像最新的是1.5.2版本的pinpoint）
1. 安装[docker](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

2. 从docker hub中下载[pinpoint](https://hub.docker.com/r/yous/pinpoint/)

###下载运行HBase
>pinpoint可以直接运行命令行下载安装Hbase，但windows系统必须自己下载安装Hbase

#####手动安装
- 下载[HBase-1.2.4-bin.tar.gz](http://apache.mirror.cdnetworks.com/hbase/)并且解压它
- 更改文件名为Hbase，并移动文件夹至quickstart/hbase下
- 下载运行Hbase：`quickstart/bin/start-hbase.cmd`
- 初始化表结构：`quickstart/bin/init-hbase.cmd`

#####使用命令
在源码第一层子目录下运行下列命令
- 下载运行Hbase：`quickstart/bin/start-hbase.sh`
- 初始化表结构：`quickstart/bin/init-hbase.sh`

###运行pinpoint
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

###查看应用
- 查看收集的数据信息Web UI：http://localhost:28080
- 查看当前监听了哪些应用Web：http://localhost:28080/stat.html
- 查看TestApp：http://localhost:28081

###停止应用
- Web UI - Run `quickstart/bin/stop-web.sh`
- TestApp - Run `quickstart/bin/stop-testapp.sh`
- Collector - Run `quickstart/bin/stop-collector.sh`
- HBase - Run `quickstart/bin/stop-hbase.sh`

##pinpoint安装配置（以1.6.0为例）

>以下配置全部基于ubuntu

###安装配置内容
- HBase (作为数据库)
- Pinpoint Collector (部署在一个web容器当中)
- Pinpoint Web (部署在一个web容器当中)
- Pinpoint Agent (和监控的应用放在一起)


###安装要求
**1.jdk1.7，jdk1.8，jdk1.6（可以用7代替）**
**2.hbase1.2**
**3.maven3.2版本以上**

###安装下载包
jdk
```
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u112-b15/jdk-8u112-linux-x64.tar.gz

export JAVA_HOME="/home/zyb/jdk1.8.0_112"
export PATH=$JAVA_HOME/bin:$PATH:$JAVA_HOME/jre/bin
export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tool.jar:$JAVA_HOME/jre/lib
```
tomcat
```
wget http://archive.apache.org/dist/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz
```
Hbase
```
wget http://apache.mirror.cdnetworks.com/hbase/1.2.4/hbase-1.2.4-bin.tar.gz

wget https://raw.githubusercontent.com/naver/pinpoint/master/hbase/scripts/hbase-create.hbase
```
pinpoint
```
wget https://github.com/naver/pinpoint/releases/download/1.6.0-RC2/pinpoint-agent-1.6.0-RC2.tar.gz
wget https://github.com/naver/pinpoint/releases/download/1.6.0-RC2/pinpoint-collector-1.6.0-RC2.war
wget https://github.com/naver/pinpoint/releases/download/1.6.0-RC2/pinpoint-web-1.6.0-RC2.war
```

###Hbase安装
>pinpoint1.6.0要求Hbase的版本为1.2

#####下载运行HBase
使用wget命令下载HBase
```
wget http://apache.mirror.cdnetworks.com/hbase/1.2.4/hbase-1.2.4-bin.tar.gz
```

直接解压然后运行bin下面的start-hbase.sh(如果没有配置过jdk环境变量，需要配置hbase-env.sh中的jdk path)
注：可以通过Jps命令验证是否启动成功，如出现HMaster表示启动成功
#####初始化表
使用wget命令下载初始化脚本
```
wget https://raw.githubusercontent.com/naver/pinpoint/master/hbase/scripts/hbase-create.hbase
```
在HBase目录下，运行初始化脚本
```
/bin/hbase shell hbase-create.hbase
```


###下载pinpoint
有两种方法
- 直接下载打包后的war包
- 下载源码，编译后打包成war包（编译太浪费时间，不建议使用）

#####直接下载


```
wget https://github.com/naver/pinpoint/releases/download/1.6.0-RC2/pinpoint-agent-1.6.0-RC2.tar.gz

wget https://github.com/naver/pinpoint/releases/download/1.6.0-RC2/pinpoint-collector-1.6.0-RC2.war

wget https://github.com/naver/pinpoint/releases/download/1.6.0-RC2/pinpoint-web-1.6.0-RC2.war
```

将web和collector分别放到web容器下，最好两个容器，我将web的端口设为28080，collector的端口设为28082，

如果hbase和web,collector在同一台机器，不需要任何配置，默认会去读本地的hbase zookeeper。



#####3.集成到应用

将agent放到app所在的服务器，然后再启动参数中加入

比如我现在用的是dubbo服务（最新的pinpoint已经支持了dubbo）

```

java  -javaagent:$AGENT_PATH/pinpoint-bootstrap-$VERSION.jar -Dpinpoint.agentId=$AGENT_ID  -Dpinpoint.applicationName=$APPLICATION_NAME

```

AGENT_PATH：agent存放的位置，

VERSION：我用的是1.5.2

AGENT_ID ：自定义。就是一个唯一标识

APPLICATION_NAME：自定义



如果是tomcat的话修改catalina.sh，添加-javaagent, -Dpinpoint.agentId, -Dpinpoint.applicationName 到 CATALINA_OPTS 参数里去。



#####4.应用测试

Web UI: http://localhost:28080

TestApp: http://localhost:28081


##pinpoint分布式监听使用

>前提是已经安装完成，并能监听自身demo



###运行主机

按照上面的命令执行



###给需要检测的应用配置agent

####配置监听

#####1.解释

将agent放到app所在的服务器，然后再启动参数中加入

比如我现在用的是dubbo服务（最新的pinpoint已经支持了dubbo）

```

java  -javaagent:$AGENT_PATH/pinpoint-bootstrap-$VERSION.jar -Dpinpoint.agentId=$AGENT_ID  -Dpinpoint.applicationName=$APPLICATION_NAME

```

AGENT_PATH：agent存放的位置，

VERSION：我用的是1.5.2

AGENT_ID ：自定义。就是一个唯一标识

APPLICATION_NAME：自定义



#####2.举例

如果是tomcat的话，要在bin/catalina.sh中添加-javaagent, -Dpinpoint.agentId, -Dpinpoint.applicationName 到 CATALINA_OPTS 参数里去。如下：



```

#pinpoint agent路径

CATALINA_OPTS="$CATALINA_OPTS -javaagent:/data1/hugang/pinpoint/pinpoint-agent-1.5.1/pinpoint-bootstrap-1.5.1.jar"

#被监控工程使用agent的标识号

CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=0000002"

#被监控工程名字

CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=10.75.0.101_8086_自定义"

```



####配置数据发送

并且需修改pinpoint-agent的配置文件$AGENT_PATH/pinpoint.config，指定pinpoint Collector的ip

```

profiler.collector.ip=10.210.228.50

```



####运行应用

将应用运行



###界面测试

在web界面中查看结果



##学习总结

####经验

1.目前使用的版本只能支持到jdk1.7 需要在path中建JAVA_7_HOME这个环境变量

2.我将hbase当中的测试数据全部清除后, 然后调用被监控的接口，然后前端总是不显示agent的application信息，然后去查询agentinfo表，确实没有数据，但是，

agentStatus当中的表都有了数据，只有agentInfo表怎么都没有数据....折腾半天原来被监控的服务需要重启，才能将agent信息重新注册到hbase中。。。。。。

3.清除了hbase的data文件之后，通过hbase的list命令返回为空库，但是重新初始化pinpoint的表结构时，提示：表已存在，，，，，很奇怪，

最后发现，虽然删除了hbase的表数据，但是hbase的表都注册到zk中了，所以需要把zk中的数据也要清理掉......

##参考文档
pinpoint介绍：
https://github.com/naver/pinpoint

Demo安装：
https://github.com/naver/pinpoint/blob/master/quickstart/README.md

完整安装：
https://github.com/naver/pinpoint/blob/master/doc/installation.md

分布式完整安装：
https://sconts.com/11

Hbase的standalone模式需要运行在127.0.0.1上，你可以去/etc/hosts里检查一下，看看你的用户名所对应的IP是不是127.0.0.1，像ubuntu这种linux一般来说是会把你的IP设置成127.0.1.1的



