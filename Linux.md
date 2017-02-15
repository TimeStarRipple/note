#Linux

>不同系统库里面软件名不一样，目前了解比较多的是ubuntu和centos，ubuntu软件很全，也很新，易于使用；centos很稳定，不过东西就不是最新的，有些软件也没有。



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



##### 查找应用进程



`ps -ef | grep java`
`ps -aux | grep java`
`pgrep java`



#####访问其他主机端口



`curl -XGET '172.17.0.8:9994'



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

/usr/lib/jvm/java-1.8.0-openjdk-amd64



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



#####下载hadoop



```

wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.6.5/hadoop-2.6.5.tar.gz

```

#####解压文件



```

tar -xvzf pinpoint-agent-1.5.1.tar.gz

```



######ssh启动



```

service ssh start



apt-get autoremove --purge openssh-server



apt-get purge openssh-server



```

######python传文件



```

python -m SimpleHTTPServer



curl -o id_rsa.pub.slave02 slave02:8000/id_rsa.pub



authorized_keys

```

######后台程序弹出来运行



```



fg



```



#####设置时区



```

sudo dpkg-reconfigure tzdata



tzselect



```



##### 利用chmod修改权限：



chmod [-cfvR] [--help] [--version] mode file



[-cfvR]：

```

-c : 若该档案权限确实已经更改，才显示其更改动作

-f : 若该档案权限无法被更改也不要显示错误讯息

-v : 显示权限变更的详细资料

-R : 对目前目录下的所有档案与子目录进行相同的权限变更(即以递回的方式逐个变更)

```



--help : 显示辅助说明



--version : 显示版本



mode : 权限设定字串

格式如下 : [who] [opt] [权限]

其中who表示对象，是以下字母中的一个或组合：

```

u：表示文件所有者

g：表示同组用户

o：表示其它用户

a：表示所有用户

```



opt则是代表操作，可以为：

```

+：添加某个权限

-：取消某个权限

=：赋予给定的权限，并取消原有的权限

```



而mode则代表权限：

```

r：可读

w：可写

x：可执行

```



用数字设定比较简单：



我们将rwx看成二进制数，对应u，g，o的权限，如果有则有1表示，没有则有0表示，那么rwx r-x r- -则可以表示成为：`111 101 100 `，再将其每三位转换成为一个十进制数，就是754。命令如下：

```

chmod 754 test.txt

```



对Document/目录下的所有子文件与子目录执行相同的权限变更：



```

chmod -R 700 Document/

```



-R参数是递归 处理目录下的所有文件以及子文件夹



700是变更后的权限表示（只有所有者有读和写以及执行的权限）



Document/ 是需要执行的目录



##### 文件内容操作

使用sed修改文件内容（可以使用正则表达式）

1、sed [sed选项] 'sed命令' 要修改的文件

2、sed [sed选项] -f sed脚本 要修改的文件

3、sed脚本 [sed选项] 要修改的文件



-i：直接修改源文件，不输出结果

-e：多次编辑，等于合并了多条sed

-f：指定sed脚本文件名

-n：取消默认的输出（不打印），可以与p结合使用，过滤信息



s替换：

`sed -n 's/nurse/little &/p' test.txt`



p打印匹配行：

`sed -n '2p' test.txt`



d 删除定位的行，例如：2d 代表删除第2行：

`sed '1d' test.txt`



参考：http://www.jb51.net/article/33509.htm



##### 卸载软件



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



3.2.清除配置文件

```

dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P

```

###快捷键





```

ctrl + a - 光标移到行首



ctrl + e - 光标移到行尾



ctrl + u - 清除光标到行首的字符



ctrl + w - 清除光标之前一个单词



ctrl + k - 清除光标到行尾的字符



shift + v - 选中当前行

```