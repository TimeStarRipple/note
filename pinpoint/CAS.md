# CAS单点登录
## 介绍

### 组件

#### 架构图
![](../images/pinpoint/CAS-1.png)

其中包括CAS Server以及CAS Clients两个组件

#### CAS Server
定义：CAS服务器是基于Spring Framework构建的Java servlet

作用：通过发出和验证票证来验证用户并授予对启用CAS的服务（通常称为CAS客户端）的访问权限

工作流程：当服务器成功登录时向用户发出票证授予票证（TGT）时，将创建SSO会话。通过使用TGT作为令牌的浏览器重定向，根据用户的请求向服务发出服务票证（ST）。随后通过反向通道通信在CAS服务器处验证ST。
#### CAS Clients

## 使用

## 总结

## 参考文档
- 官方文档：https://apereo.github.io/cas/4.2.x/index.html