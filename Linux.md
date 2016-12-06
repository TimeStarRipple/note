#Linux

>不同系统库里面软件名不一样



##Ubuntu

###指令

#####更新库指令

`sudo apt-get update`

#####安装指令

`sudo apt-get install ...`

#####扩展个人源命令（ppa）

`sudo apt-get install software-properties-common`

#####vim（文本编辑器）

`sudo apt-get install vim`

#####wget（能根据url下载文件）

`sudo apt-get install wget`

#####查找进程占用端口

`netstat -ap | grep 8080`

#####文件操作（复制，移动，删除）

```

复制：sudo cp

删除：sudo rm

移动：sudo mv

```

备注：单词缩写

#####下载jdk

```

wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u67-b01/jdk-7u67-linux-x64.tar.gz

wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u112-b15/jdk-8u112-linux-x64.tar.gz

```

#####配置jdk

```

export JAVA_HOME="/home/zyb/jdk1.8.0_112"

export PATH=$JAVA_HOME/bin:$PATH:$JAVA_HOME/jre/bin

export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tool.jar:$JAVA_HOME/jre/lib

```

#####下载maven

```

wget http://apache.mirrors.tds.net/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz

wget https://archive.apache.org/dist/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz

```

#####下载tomcat

```

wget http://archive.apache.org/dist/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz

```

#####下载zookeeper

```

wget http://apache.mirror.anlx.net/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz

```

#####解压文件

```

tar -xvzf pinpoint-agent-1.5.1.tar.gz

```



#####卸载软件

1.删除软件

如果知道具体删除软件具体名

```

sudo apt-get remove --purge 软件名称

sudo apt-get autoremove --purge 软件名称

```

如果不知道具体删除软件具体名

```

dpkg --get-selections | grep ‘软件相关名称’

```



2.清除配置文件

```

dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P

```









###快捷键

ctrl + a - 光标移到行首

ctrl + e - 光标移到行尾

ctrl + u - 清除光标到行首的字符

ctrl + w - 清除光标之前一个单词

ctrl + k - 清除光标到行尾的字符



