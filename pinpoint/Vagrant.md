# Vagrant

## vagrant介绍

### vagrant定义
Vagrant是一个基于Ruby的工具，用于创建和部署虚拟化开发环境。它使用Oracle的开源VirtualBox虚拟化系统，使用 Chef创建自动化虚拟环境。

**说的简单点，他就是一个自动化创建虚拟机的工具。**

### vagrant核心
通过运行vagrantfile文件配置自动化构建虚拟机在虚拟机的运行平台上

### vagrant优点
- 能够自动化部署环境，使得开发人员只需要专注于开发而不是环境的搭建
- 能够解决开发环境不一致性的问题，能够使用命令随时随地构建一致的开发环境
- 对编写的开发环境版本控制
- 文件同步，可以同步主机vagrantfile根目录下的文件到虚拟机

## vagrant安装使用

> 基于windows操作系统

### 安装vagrant
https://releases.hashicorp.com/vagrant/1.9.1/vagrant_1.9.1.msi

### 安装虚拟机运行平台
- [virtualbox](http://download.virtualbox.org/virtualbox/5.0.10/VirtualBox-5.0.10-104061-Win.exe)默认使用

### 镜像库
https://atlas.hashicorp.com/boxes/search

### 新建vagrant项目流程

#### 新建vagrant项目目录
创建一个文件夹，用于存放虚拟机

#### 添加vagrant项目依赖镜像
添加
```
vagrant box add hashicorp/precise64
```

#### 初始化vagrant项目目录
```
vagrant init
```
这个命令会创建vagrantfile文件，vagrant是根据这个文件来安装配置虚拟机的

#### 安装运行vagrant虚拟机
```
vagrant up
```
根据vagrantfile文件重新安装配置虚拟机，启动虚拟机

#### 重启虚拟机
```
vagrant reload
```
当更改vagrantfile文件的时候，并且虚拟机
依旧在运行，运行这个命令会重启虚拟机，和上个命令相比，如果虚拟机
没有运行就用上面的命令

#### 连接虚拟机
```
vagrant ssh
```
使用ssh远程连接虚拟机

#### 退出虚拟机
在ssh的shell界面中用这个`CTRL+D`快捷键可以退出ssh，或者输入下列命令退出
```
logout
```

#### 关闭虚拟机
```
vagrant halt
```
对应就是关机

#### 暂停虚拟机
```
vagrant suspend
```
只是暂停，虚拟机内存等信息将以状态文件的方式保存在本地，可以执行恢复操作后继续使用

#### 恢复虚拟机
```
vagrant resume
```
与前面的暂停相对应

#### 打包虚拟机
```
vagrant package --output NAME --vagrantfile FILE

可选参数：
--output NAME ： （可选）设置通过NAME来指定输出的文件名
--vagrantfile FILE：（可选）可以将Vagrantfile直接封进box中
```
注：如果网络模式中使用 private_network 的话，在打包之前需要清除一下private_network的设置，避免不必要的错误：
```
sudo rm -f /etc/udev/rule.d/70-persistent-net.rules
```
制作完成之后直接将box文件拿到其他计算机上配置即可使用。

#### 删除虚拟机
```
vagrant destroy
```
删除后在当前虚拟机所做进行的除开Vagrantfile中的配置都不会保留

#### 删除box
```
vagrant box remove
```

## vagrant示例

```
Vagrant.configure("2") do |config|
  config.vm.define :web do |web|
    web.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "web", "--memory", "512"]
    end
    web.vm.box = "CentOs7"
    web.vm.hostname = "web"
    web.vm.network :private_network, ip: "192.168.33.10"
  end

  config.vm.define :redis do |redis|
    redis.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "redis", "--memory", "512"]
    end
    redis.vm.box = "CentOs7"
    redis.vm.hostname = "redis"
    redis.vm.network :private_network, ip: "192.168.33.11"
  end
end
```

## 问题
1. 直接安装virtualbox最新版本，安装vagrant不识别，会自己重新下载安装，并且自己下载virtualbox的速度非常慢，很容易失败，下载好了之后又停在那不动，没有任何提示信息，建议下好之后，停止安装程序，手动安装，就可以了。

## 参考文献
官方网址：
https://www.vagrantup.com/docs/provisioning/basic_usage.html
