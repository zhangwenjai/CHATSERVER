# 基于muduo网络库实现的集群聊天服务器
# 关键技术点
muduo c++网络库、MySQL数据库编程、json、事件回调编程模式、单例模式、观察者模式、nginx配置tcp负载均衡、服务器中间件redis消息队列等

# 项目背景
本项目是一个网络服务器的项目，主要分为四个模块，首先就是网络模块，网络模块使用了muduo网络库，这样可以解耦网络模块和业务模块的代码，更专注于开发业务代码。服务层主要使用了一些c++11的技术例如map，绑定器，主要在设计了一系列消息id以及对于这些消息发生之后，对回调操作进行绑定，相当于做了一个回调机制，当网络I/O发送消息请求时，解析出json中的消息id，通过回调处理相应的消息。数据存储层使用MySQL进行数据存储，主要设计了五个表用来存储关键数据。单机模式下主要就是以上内容，但是单机模式下并发能力有限，于是考虑到多机扩展，部署多台服务器就用到了nginx实现TCP负载均衡器，由于服务器还需要向用户推送消息，所以是一种长连接。由于在不同服务器上注册/登录的用户需要通信，所以考虑到了使用服务器中间件，使用redis的发布——订阅来实现一个消息队列的功能，完成跨服务器的通信功能。

# 项目业务
1. 客户端新用户注册
2. 客户端用户登录
3. 添加好友和添加群组
4. 好友聊天
5. 群组聊天
6. 离线消息
7. nginx配置tcp负载均衡
8. 集群聊天系统支持客户端跨服务器通信

# 数据库设计
## User表

| 字段名称 | 字段类型 | 字段说明 | 约束 |
|---|---|---|---|
| id | int | 用户id | PRIMARY KEY、AUTO_INCREMENT |
| name | varchar(50) | 用户名 | NOT NULL, UNIQUE |
| password | varchar(50) | 用户密码 | NOT NULL |
| state | enum('online','offline') | 当前登陆状态 | DEFAULT 'offline' |

## Friend表

| 字段名称 | 字段类型 | 字段说明 | 约束 |
|---|---|---|---|
| userid | int | 用户id | NOT NULL、联合主键 |
| friendid | int | 好友id | NOT NULL、联合主键 |

## OfflineMesasge表

| 字段名称 | 字段类型 | 字段说明 | 约束 |
|---|---|---|---|
| userid | int | 用户id | NOT NULL |
| message | varchar(50) | 离线消息(存储json字符串) | NOT NULL |

## AllGroup表

| 字段名称 | 字段类型 | 字段说明 | 约束 |
|---|---|---|---|
| id | int | 组id | PRIMARY KEY、AUTO_INCREMENT |
| groupname | varchar(50) | 组名称 | NOT NULL,UNIQUE |
| groupdesc | varchar(200) | 组功能描述 | DEFAULT '' |

## GroupUser表

| 字段名称 | 字段类型 | 字段说明 | 约束 |
|---|---|---|---|
| groupid | int | 组id | NOT NULL、联合主键 |
| userid | int | 组员id | NOT NULL、联合主键 |
| grouprole | enum('creator','normal') | 组内角色 | DEFAULT ‘normal’ |

# 开发平台以及对应版本
本项目在windows 10使用vscode通过ssh远程连接linux进行开发，linux使用Ubuntu 20.04系统。json的使用添加json.hpp即可，redis使用apt-get install安装即可，nginx下载后配置使用即可，配置在8000端口，监听linux的6000和6002两个端口，为这两个端口做均衡。

IDE:VScode; MySQL:5.7.17.0;Linux:Ubuntu 20.04;

# 使用流程

clone项目到本地，./autobuild.sh即可完成一件编译，进入bin目录可以看到ChatServer和ChatClient，运行即可，客户使用连接8000端口，服务器登陆在6000和6002两个端口。


