# Drone

## 介绍

### 定义
Drone is a Continuous Integration platform built on container technology. Every build is executed inside an ephemeral Docker container, giving developers complete control over their build environment with guaranteed isolation.

翻译：Drone是一个基于容器的持续集成平台技术，每一个构件在暂时的docker容器中执行，通过隔离的形式给开发人员完整的控制他们的构件环境

### 开发目标
Drone's prime directive is to help teams ship code like GitHub. Drone is easy to install, setup and maintain and offers a powerful container-based plugin system. Drone aspires to eventually offer an industry-wide replacement for Jenkins.

翻译：Drone的主要指令是帮助团队像GitHub一样运输代码。 Drone易于安装，设置和维护，并提供强大的基于容器的插件系统。Drone希望最终能在全行业范围内替换Jenkins。

### 工作流程
Drone需要和一个版本控制系统集合使用，Drone如果和gogs连用的话，在gogs创建仓库的时候，会自动把仓库同步到drone的仓库中，可以在drone中启用操作响应，就会在gogs中产生一个钩子，用于在对程序进行操作，也可以在gogs中生成钩子，使用http请求通知drone进行响应，响应的操作是根据在gogs提交的文件中的`.drone.yml`文件代码进行操作。

`.drone.yml`文件写的是项目的构件步骤，里面有几个部分，参考[原文](http://readme.drone.io/0.5/usage/repository/configuration/)后，我将分别进行讨论
#### Repository（仓库）：
作为项目构建的环境，可以随便选择一个有效的docker 镜像。drone会自动把项目克隆到容器当中。

#### Services（服务）：
Drone supports launching service containers as part of the build process. This can be very helpful when your unit tests require database access, for example. Service containers share the same network (ie localhost) as your build containers.

翻译：Drone支持启动服务容器作为构建过程的一部分。 例如，当您的单元测试需要数据库访问时，这可能非常有用。 服务容器与构建容器共享同一网络（即localhost）。

#### Plugins（插件）：
Drone supports publish, deployment and notification capabilities through external plugins. Plugins are Docker containers that are automatically downloaded, attach to your build, and perform a specific task.

翻译：Drone通过外部插件支持发布，部署和通知功能。 插件是Docker容器，会自动下载，附加到您的构建，并执行特定任务。

#### matrix（矩阵）
Drone has integrated support for matrix builds. Drone executes a separate build task for each combination in the matrix, allowing you to build and test a single commit against multiple configurations.

翻译：Drone集成了对矩阵构建的支持。 Drone为矩阵中的每个组合执行单独的构建任务，允许您针对多个配置构建和测试单个提交。

可以写多个配置进行参数替换

参考文档：http://readme.drone.io/0.5/usage/build-matrix/

可以在`.drone.yml`中的配置过程中加入判断条件，进行执行。可以对它构建时返回的状态码进行条件判定

## 文档
Documentation is published to [readme.drone.io](http://readme.drone.io/)

## 安装
Please see our [installation guide](http://readme.drone.io/admin/installation-guide/) to install the official Docker image.

### 安装步骤
1. 安装集成工具[gogs](https://github.com/gogits/gogs/tree/master/docker)
2. 安装server
3. 安装agent
4. 安装CLI

## 限制
Pull Request and Tag events are not supported. We are interested in patches to include this functionality. If you are interested in contributing to Drone and submitting a patch please contact us.
