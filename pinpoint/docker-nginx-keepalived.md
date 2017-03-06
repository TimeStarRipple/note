# docker+nginx+keepalived主主集群搭建

## 目的
**搭建一个高可用，负载均衡的反向代理服务器的集群。**

### docker的作用
通过使用容器搭建成本较低的集群。

### nginx的作用
nginx用于反向代理容器集群，同时起到负载均衡的作用。

### keepalived的作用
为了防止nginx出现单点故障的情况，使用keepalived搭建高可用的集群，同时为了不造成资源浪费，这里不采用主从的配置方式，而采用主主双热备的配置方式。

## 搭建思路
1. 在两台服务器上同时运行一个nginx容器反向代理各个集群，不建议搭在一台服务器上，容易出现单点故障，一旦这台服务器崩溃了，两个容器都会一起崩溃，没有起到高可用的作用。
2. 在nginx容器所在的两台服务器上都装上keepalived，两台keepalived互为主从，虚拟出两个ip提供服务。这样能保证两台之中任何一台崩溃了，另一台都能够进行转移提供服务。
3. 将2个虚拟ip绑定到同一个域名之上，采用DNS轮询的方式，配合keepalived提供高可用的性能。

## 环境准备
1. 两台服务器（可以是两台虚拟机）
2. 装有ubuntu14.04（采用root权限进行操作）
3. 装好docker

假设第一台服务器ip为`192.168.202.128`，下面简称为128主机；第二台服务器ip为`192.168.202.130`，下面简称为130主机；第一台服务器虚拟的ip为`192.168.202.131`；第二台服务器虚拟的ip为`192.168.202.132`；大家在参考的时候请对照自己的ip进行更改。

## 集群搭建

### 安装运行nginx
两台机子都要执行下面的命令
```
#拉取nginx镜像
docker pull nginx:1.7.6

#新建nginx测试页面目录，用于同步到容器当中去
mkdir -p /tmp/docker

#新建nginx测试页面（另一台机子记得改ip）
echo "<h2 >This is nginx official container running on 192.168.202.128 </h2><br /> static files:/tmp/docker/index.html" > /tmp/docker/index.html

#运行docker的nginx容器
docker run --name nginx_m --restart=always -v /tmp/docker:/usr/share/nginx/html:ro -p 80:80 -d nginx:1.7.6
```
注：
1. `/tmp/docker`目录下的文件在主机重启后会消失，需要重新写
2. `--restart=always`是指在docker服务重启后，该容器服务也会自动重启
3. 如果没有使用root权限，在执行echo的时候加上sudo也会报错，因为它仅仅是对echo加上了权限，`>`并没有加上权限，要在命令前加上`sh -c`才行，完整如下：


```
sudo sh -c "echo '<h2 >This is nginx official container running on 192.168.202.128 </h2><br /> static files:/tmp/docker/index.html' > /tmp/docker/index.html"
```


### 安装配置keepalived
两台机子都要执行下面的命令

```
#更新ubuntu索引
apt-get update

#安装keepalived前置软件
apt-get install -y libssl-dev openssl libpopt-dev

#安装keepalived
apt-get install -y keepalived

#复制配置文件
cp /usr/share/doc/keepalived/samples/keepalived.conf.sample /etc/keepalived/keepalived.conf

#编辑配置文件，修改如下
vim /etc/keepalived/keepalived.conf
```

128主机配置如下：

```
!Configuration File for keepalived

global_defs {
    router_id Node-100
}

vrrp_script chk_nginx {
    script "killall -0 nginx"
    interval 1
    weight -20 # if failed, decrease 40 of the priority 
    fall 1 # require 2 failures for failures
    rise 1 # require 1 sucesses for ok
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 120
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass handhand
    }
    virtual_ipaddress {
        192.168.202.131
    }
    track_script {
        chk_nginx
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface eth0
    virtual_router_id 52
    nopreempt
    priority 115
    advert_int 1 {
        auth_type PASS
        auth_pass handhand
    }
    virtual_ipaddress {
        192.168.202.132
    }
    track_script {
        chk_nginx
    }
}
```

130主机的配置文件

```
!
Configuration File for keepalived

global_defs {
    router_id Node-103
}

vrrp_script chk_nginx {
    script "killall -0 nginx"
    interval 1
    weight -20 # if failed, decrease 40 of the priority 
    fall 1 # require 2 failures for failures
    rise 1 # require 1 sucesses for ok
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    nopreempt
    priority 115
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass handhand
    }
    virtual_ipaddress {
        192.168.202.131
    }
    track_script {
        chk_nginx
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 120
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass handhand
    }
    virtual_ipaddress {
        192.168.202.132
    }
    track_script {
        chk_nginx
    }
}
```

### 启动keepalived
两台机子都要执行下面的命令

```
service keepalived start[stop,restart]
```
运行结果如下：

![](/images/pinpoint/docker-nginx-keepalived-1.png)

然后重启服务器
```
shutdown -r now
```
注：keepalived要绑定虚拟ip，需要重新启动虚拟机。

### 查看ip绑定结果
分别在两台服务器上运行`ip a`查看ip绑定情况

192.168.202.128主机结果：

![](/images/pinpoint/docker-nginx-keepalived-2.png)

192.168.202.130主机结果：

![](/images/pinpoint/docker-nginx-keepalived-3.png)

### 查看界面访问结果

服务器ip访问结果：

128主机访问结果：
![](/images/pinpoint/docker-nginx-keepalived-4.png)

130主机访问结果：
![](/images/pinpoint/docker-nginx-keepalived-5.png)

虚拟ip访问结果：

![](/images/pinpoint/docker-nginx-keepalived-6.png)

![](/images/pinpoint/docker-nginx-keepalived-7.png)

## 集群测试
正常情况下，当提供服务的主机崩溃了，从机应该会进行故障转移提供服务，用户应该还能继续访问。当主机重启时，服务就会还原到主机继续提供运行。因此我们尝试分别关闭主机后重启，同时查看访问结果。
1.将128主机关机或关闭128主机的keepalived，查看是否虚拟ip131是否绑定到130主机上，正确结果应该如下：

130主机ip绑定结果：
![](/images/pinpoint/docker-nginx-keepalived-8.png)

主机访问结果：
![](/images/pinpoint/docker-nginx-keepalived-9.png)

![](/images/pinpoint/docker-nginx-keepalived-10.png)

2.然后再启动128主机，查看虚拟ip131是否重新绑回128主机，正确结果如下：

128主机ip绑定结果：
![](/images/pinpoint/docker-nginx-keepalived-11.png)
此时130主机绑定的ip131转移到了128主机

主机访问结果：
![](/images/pinpoint/docker-nginx-keepalived-13.png)

130主机的关机和重启效果和128相同，所以我也就不进行描述了，大家自己配置的话最好自己测试一下。

注：我这里其实是因为关闭nginx容器没有作用，所以我才采取关闭服务器的方法来转移ip，之所以会出现这样的问题，其实是因为keepalived的检测脚本`vrrp_script chk_nginx`有问题所导致，后面需要对shell进行研究写一个合适的脚本去做到。

## 总结
1. nopreempt属性允许一个priority比较低的节点作为master，即使有priority更高的节点启动。因为只有state为BACKUP的节点才会决定是否成为master，因此nopreempt和backup搭配才有意义。nopreempt的配置主要是为了防止抢占式的存在。
2. 如果集群当中配置了state为master的主机，只要主机存在并且当前优先级比master节点高，那么他就会抢占虚拟ip，作为master节点。如果我们不想ip被抢占，那么所有的主机都配置state为BACKUP，加上nopreempt，这样即使有更高优先级的主机启动，也不会抢占虚拟ip成为主机，或者把master节点优先级调小也可以。配置不抢占最好用前一个方法。
3. 主节点选举方法，当集群没有主节点时，用master做主节点，其他节点无法抢占，如果没有master，根据优先级大小选出主节点，如果已经有了主节点那就进行抢占ip判定

## 参考文献
- docker nginx1.7.6+keepalived实现双机热备：http://www.cnblogs.com/lys_013/archive/2016/08/18/5783107.html

- keepalived工作原理和配置说明：http://outofmemory.cn/wiki/keepalived-configuration
- Keepalived百度百科：http://baike.baidu.com/link?url=HueiV1mRPXx7Q389AmN2sCNFqeht18wo5M-ZhLKgEC1SkhwYJ8UGjBhrPw7Top8J8VNwHOL08QVPox95pEV6biIqnKyn8Q0Tc6lLuQJAF8K