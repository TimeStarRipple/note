# Ansible

## Ansible简介

### Ansible定义
Ansible是新出现的自动化运维工具，基于Python开发，集合了众多运维工具（puppet、cfengine、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。

Ansible是基于模块工作的，本身没有批量部署的能力。真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。

### Ansible本质
- 自动化配置远程服务器
- 使用SSH批量的在远程服务器上执行命令

### Ansible主要内容及作用
1. 连接插件connection plugins：负责和被监控端实现通信
2. host inventory：指定操作的主机，是一个配置文件里面定义监控的主机
3. 各种模块核心模块、command模块、自定义模块
4. 借助于插件完成记录日志邮件等功能
5. playbook：剧本执行多个任务时，非必需可以让节点一次性运行多个任务

## Ansible安装
将ansible安装在管理主机上对托管的节点进行远程配置，

### Ansible安装要求
ansible分成了两个部分：一个是管理主机，另一个是托管节点

#### 管理主机配置
1. 软件：Python 2.6 或 Python 2.7
2. 环境：SSH
2. 系统：Red Hat, Debian, CentOS, OS X, BSD的各种版本,等等
3. 子进程限制：运行`sudo launchctl limit maxfiles 1024 2048`增加子进程限制

自2.0版本开始,ansible使用了更多句柄来管理它的子进程,对于OS X系统,你需要增加ulimit值才能使用15个以上子进程,否则你可能会看见”Too many open file”的错误提示.

**注：windows系统不可以做控制主机**

#### 托管节点配置
1. 软件：Python2.6或Python2.7
2. 环境：SSH

通常我们使用 ssh 与托管节点通信，默认使用 sftp.如果 sftp 不可用，可在 ansible.cfg 配置文件中配置成 scp 的方式. 在托管节点上也需要安装 Python 2.4 或以上的版本.如果版本低于 Python 2.5 ,还需要额外安装一个模块:

### Ansible安装方式
下面介绍两种安装方式

1. github源码安装
2. ubuntu安装

#### github源码安装
1.下载源码
```
$ git clone git://github.com/ansible/ansible.git --recursive
$ cd ./ansible
```
2.使用bash
```
$ source ./hacking/env-setup
```
3.使用 Fish:
```
$ . ./hacking/env-setup.fish
```

#### ubuntu安装
```
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```
也可以在源码中打包出Debian/Ubuntu 软件包，在源码根目录下执行：
```
$ make deb
```
然后执行安装命令
```
$ sudo apt-get install ansible
```

##Ansible初步使用

### 远程连接前提配置
你的public SSH key必须在这些系统的`authorized_keys`中:
需要把管理主机的秘钥加入托管节点的主机上的`authorized_keys`中

### 配置hosts文件
编辑(或创建)/etc/ansible/hosts 并在其中加入一个或多个远程系统.
```
192.168.1.50
```

### 运行指令
现在ping 你的所有节点
```
ansible all -m ping
```

Ansible会像SSH那样试图用你的当前用户名来连接你的远程机器.要覆写远程用户名,只需使用’-u’参数. 如果你想访问 sudo模式,这里也有标识(flags)来实现:
```
# as bruce
$ ansible all -m ping -u bruce
# as bruce, sudoing to root
$ ansible all -m ping -u bruce --sudo
# as bruce, sudoing to batman
$ ansible all -m ping -u bruce --sudo --sudo-user batman
```

(如果你碰巧想要使用其他sudo的实现方式,你可以通过修改Ansible的配置文件来实现.也可以通过传递标识给sudo(如-H)来设置.) 现在对你的所有节点运行一个命令:
```
$ ansible all -a "/bin/echo hello"
```

##　Inventory文件配置
Ansible 可同时操作属于一个组的多台主机,组和主机之间的关系通过 inventory 文件配置. 默认的文件路径为 /etc/ansible/hosts

## 运行指令

格式：
```
ansible <pattern_goes_here> -m <module_name> -a <arguments>
```
- pattern_goes_here：节点的集合
- module_name：模块名
- arguments：命令
示例如下：
```
ansible webservers -m service -a "name=httpd state=restarted"
```

## ansible举例

```
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```



## 参考文献
官方文档：
http://www.ansible.com.cn/docs/playbooks_intro.html









