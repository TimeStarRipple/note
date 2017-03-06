# pinpoint hbase 分布式部署



>为解决hbase崩溃，导致的应用程序崩溃的问题，采用分布式部署的方式，增强系统的稳定性







## 部署方案



我们将在3个docker容器中搭建1个master，2个slave的集群方案。每台主机上都有一个hadoop，zookeeper和hbase







## 软件环境



- jdk 1.8 镜像



- hadoop 2.6.5



- zookeeper 3.4.6



- hbase 1.2.4



- 时区相同







**注：如果不用jdk镜像的话，可以在ubuntu14.04上配置jdk1.8代替**







## 环境配置







### jdk8镜像



```



# 下载或更新java镜像，然后运行该镜像形成一个新的容器，并进入到容器当中去



sudo docker run -ti registry.saas.hand-china.com/tools/java 



```



**注：这里用的是自己公司做的openjdk8的镜像，不想用的话可以去dockerhub上另外选一个镜像**







### 配置hosts



在每台主机上修改host文件，下面的ip换成容器的ip，同时把容器id也对应ip写进文件当中



```



vi /etc/hosts







10.42.70.86      master



10.42.58.146     slave1



10.42.33.125  slave2



10.42.70.86     a38f9755b4d2



10.42.58.146     4a698482cc97



10.42.33.125     3197f6c8e429







```



配置之后ping一下用户名看是否生效



```



ping slave1



ping slave2



```



**注：在rancher上面配置的时候，不要用内部的容器ip，要用rancher分配的在外面显示的容器ip，如下图：**



![](/images/pinpoint/Hadoop-Zookeeper-Hbase-3.png)







### SSH 免密码登录



>使用hadoop集群的前提是要配置ssh免密码登录







安装Openssh server



```



sudo apt-get install openssh-server



```



在所有机器上都生成私钥和公钥



```



ssh-keygen -t rsa   #一路回车



```



需要让机器间都能相互访问，就把每个机子上的id_rsa.pub发给master节点，传输公钥可以用scp来传输。



```



scp ~/.ssh/id_rsa.pub spark@master:~/.ssh/id_rsa.pub.slave1



```



当然也可以用Python传文件



```



python -m SimpleHTTPServer



curl -o id_rsa.pub.slave1 slave1:8000/id_rsa.pub



curl -o id_rsa.pub.slave2 slave2:8000/id_rsa.pub



```



在master上，将所有公钥加到用于认证的公钥文件authorized_keys中



```



cat ~/.ssh/id_rsa.pub* >> ~/.ssh/authorized_keys



```



将公钥文件authorized_keys分发给每台slave



```



scp ~/.ssh/authorized_keys spark@slave1:~/.ssh/



或



curl -O master:8000/authorized_keys



```



在每台机子上面启动SSH服务



```



service ssh start



```



在每台机子上验证SSH无密码通信



```



ssh master



ssh slave1



ssh slave2



```



注：如果登陆测试不成功，则可能需要修改文件authorized_keys的权限（权限的设置非常重要，因为不安全的设置安全设置,会让你不能使用RSA功能 ）



```



chmod 600 ~/.ssh/authorized_keys



```







## Hadoop安装配置







### Hadoop配置文件



`cd ~/workspace/hadoop-2.6.0/etc/hadoop`进入hadoop配置目录，需要配置有以下7个文件：`hadoop-env.sh`，`yarn-env.sh`，`slaves`，`core-site.xml`，`hdfs-site.xml`，`maprd-site.xml`，`yarn-site.xml`







#### 在hadoop-env.sh中配置JAVA_HOME



```



The java implementation to use.



export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64



```



这里默认用环境里面openjdk路径







#### 在yarn-env.sh中配置JAVA_HOME



```



# some Java parameters



export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64







```







#### 在slaves中配置slave节点的ip或者host



```



slave1



slave2



```







#### 修改core-site.xml



```



<configuration>



    <property>



        <name>fs.defaultFS</name>



        <value>hdfs://master:9000/</value>



    </property>



    <property>



         <name>hadoop.tmp.dir</name>



         <value>file:/home/spark/workspace/hadoop-2.6.0/tmp</value>



    </property>



</configuration>



```







#### 修改hdfs-site.xml



```



<configuration>



    <property>



        <name>dfs.namenode.secondary.http-address</name>



        <value>master:9001</value>



    </property>



    <property>



        <name>dfs.namenode.name.dir</name>



        <value>file:/home/spark/workspace/hadoop-2.6.0/dfs/name</value>



    </property>



    <property>



        <name>dfs.datanode.data.dir</name>



        <value>file:/home/spark/workspace/hadoop-2.6.0/dfs/data</value>



    </property>



    <property>



        <name>dfs.replication</name>



        <value>3</value>



    </property>



</configuration>



```







#### 修改mapred-site.xml



```



<configuration>



    <property>



        <name>mapreduce.framework.name</name>



        <value>yarn</value>



    </property>



</configuration>



```







#### 修改yarn-site.xml



```



<configuration>



    <property>



        <name>yarn.nodemanager.aux-services</name>



        <value>mapreduce_shuffle</value>



    </property>



    <property>



        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>



        <value>org.apache.hadoop.mapred.ShuffleHandler</value>



    </property>



    <property>



        <name>yarn.resourcemanager.address</name>



        <value>master:8032</value>



    </property>



    <property>



        <name>yarn.resourcemanager.scheduler.address</name>



        <value>master:8030</value>



    </property>



    <property>



        <name>yarn.resourcemanager.resource-tracker.address</name>



        <value>master:8035</value>



    </property>



    <property>



        <name>yarn.resourcemanager.admin.address</name>



        <value>master:8033</value>



    </property>



    <property>



        <name>yarn.resourcemanager.webapp.address</name>



        <value>master:8088</value>



    </property>



</configuration>



```







### 启动 Hadoop



在 master 上执行以下操作，就可以启动 hadoop 了。



```



cd ~/workspace/hadoop-2.6.0     #进入hadoop目录



bin/hadoop namenode -format     #格式化namenode



sbin/start-dfs.sh               #启动dfs 



sbin/start-yarn.sh              #启动yarn



```







### 验证 Hadoop 是否安装成功



可以通过jps命令查看各个节点启动的进程是否正常。在 master 上应该有以下几个进程：



```



$ jps  #run on master



3407 SecondaryNameNode



3218 NameNode



3552 ResourceManager



3910 Jps



```







在每个slave上应该有以下几个进程：



```



$ jps   #run on slaves



2072 NodeManager



2213 Jps



1962 DataNode



```



或者在浏览器中输入 http://master:8088 ，应该有 hadoop 的管理界面出来了，并能看到 slave1 和 slave2 节点。







## Zookeeper安装配置







### Zookeeper文件配置



在$ZOOKEEPER_HOME/路径下执行以下命令



```



cp zoo_sample.cfg zoo.cfg



```







复制配置文件的模板进行配置，配置如下：



```



tickTime=2000



initLimit=10



syncLimit=5



dataDir=/usr/data/zookeeper-3.4.6/data



dataLogDir=/usr/data/zookeeper-3.4.6/logs



clientPort=2181



server.0=master:2888:3888



server.1=slave1:2888:3888



server.2=slave2:2888:3888



```







[配置文件说明](http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html#sc_RunningReplicatedZooKeeper)







### 分发配置文件



把配置好的zookeeper分发到其他主机上







### 配置myid文件



在我们配置的dataDir指定的目录下面，创建一个myid文件，里面内容为一个数字，用来标识当前主机，`conf/zoo.cfg`文件中配置的server.X中X为什么数字，则myid文件中就输入这个数字，例如：



```



root@master:~ echo "0" > /usr/data/zookeeper-3.4.6/data/myid



root@slave1:~ echo "1" > /usr/data/zookeeper-3.4.6/data/myid



root@slave2:~ echo "2" > /usr/data/zookeeper-3.4.6/data/myid



```







### 启动zookeeper集群



在主机上运行下面的指令



```



/usr/software/zookeeper-start.sh



```







可以通过运行下面的指令分别查看每个容器是否运行成功



```



/usr/software/zookeeper-3.4.6/bin/zkServer.sh status



```







## Hbase安装配置







### 配置Hbase







#### 编辑hbase-env.sh文件







```



# The java implementation to use.  Java 1.7+ required.



export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64



```







#### 编辑conf/hbase-site.xml文件



```



<configuration>



  <property>



    <name>hbase.rootdir</name>



    <value>hdfs://master:9000/hbase</value>



  </property>



  <property>



    <name>hbase.cluster.distributed</name>



    <value>true</value>



  </property>



  <property>



    <name>hbase.zookeeper.quorum</name>



    <value>master,slave1,slave2</value>



  </property>



  <property>



    <name>hbase.zookeeper.property.dataDir</name>



    <value>/home/spark/workspace/zookeeper/data</value>



  </property>



</configuration>



```







其中第一个属性指定本机的hbase的存储目录，必须与Hadoop集群的core-site.xml文件配置保持一致；第二个属性指定hbase的运行模式，true代表全分布模式；第三个属性指定 Zookeeper 管理的机器，一般为奇数个；第四个属性是数据存放的路径。这里我使用的默认的 HBase 自带的 Zookeeper。







#### 配置regionservers



在regionservers文件中添加如下内容：



```



slave1



slave2



```



regionservers文件列出了所有运行hbase的机器（即HRegionServer)。此文件的配置和Hadoop中的slaves文件十分相似，每行指定一台机器的主机名。当HBase启动的时候，会将此文件中列出的所有机器启动。关闭时亦如此。我们的配置意为在 slave1, slave2, slave3 上都将启动 RegionServer。







#### 分发hbase



```



scp -r hbase-1.2.4 root@slave1:~/workspace/



scp -r hbase-1.2.4 root@slave2:~/workspace/



```







### 运行Hbase



在master上运行



```



cd ~/workspace/hbase-1.2.4



bin/start-hbase.sh



```







单独启动regionserver（当某一个节点崩溃了，可以使用下面的命令做恢复）



```



#启动集群中所有的regionserver



./hbase-daemons.sh start regionserver



#启动某个regionserver



./hbase-daemon.sh start regionserver



```







### 验证 HBase 成功安装



在 master 运行 jps 应该会有HMaster进程。在各个 slave 上运行jps 应该会有HQuorumPeer,HRegionServer两个进程。







在浏览器中输入 http://master:16010 可以看到 HBase Web UI 。







## 总结







### 问题



1. 本机上3个容器，相互通信，运行都是正常的，但是在rancher上hadoop和zookeeper集群通信正常，但是hbase就出现了问题，后来发现单独启动zookeeper集群，然后再启动hbase集群就可以了。不太清楚为什么，需要进一步研究







2. 服务器连接zookeeper集群，发现连接不上，连接拒绝，其实是因为外部服务器在第一次连接zookeeper之后会拿到zookeeper所在主机的主机名，第二次查找zookeeper的时候，是用主机名连接的，如果`/etc/hosts`文件当中没有配置zookeeper集群的主机名就会连接失败。因此要在`/etc/hosts`文件中加入zookeeper集群的主机名。







3. hbase连接zookeeper集群时，各种奇怪报错，连接不上，有可能就是因为问题2中的原因，需要配置zookeeper集群的主机名。







4. hbase集群是需要暴露16020端口的，用来连接Regionserver的







5. pinpoint collector 集群连接hbase集群的时候，有一台collector一直连不上hbase集群中的一台，使用curl也不行，后来能发现这个容器是和hbase其中的一个容器在同一台主机上。我能分析得出，容器是无法连接主机的端口的，所以同一台主机的容器相连，可以直接使用rancher给容器分配的地址就行，不需要使用主机地址加暴露端口。







### 个人总结



分布式部署确实能增强系统的稳定性，但是如果部署到同一台机器上的不同容器当中，服务器崩溃依旧会导致系统无法正常使用，所以需要部署到不同主机上。







因为配置很多，所以我做了一个docker镜像，镜像名是`registry.saas.hand-china.com/tools/pinpoint-hbase`，镜像里面已经配置好了所有的文件，但还是有一些东西必须要容器启动后才能配置，这里可以依旧跑3个容器，然后进行如下配置



1. 配置hosts



2. SSH 免密码登录



3. 配置myid文件







参照上面步骤进行配置







注：rancher要开放2181,16010,16020,16030端口方便连接和排除错误











