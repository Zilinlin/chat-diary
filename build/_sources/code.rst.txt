首页
======

项目简介
------

畅聊，定位为即时通讯软件。软件支持用户注册登录、添加好友、创建群聊、实时聊天、发布动态、查看动态、修改个人信息等功能。
其中聊天功能支持发送和查看普通文字信息、图片信息、视频信息、音频信息和位置信息。

项目地址
------
代码库：`ChatApp-Android <https://github.com/fuboat/ChatApp-Android>`_

后端子模块代码库：`ChatApp-FullServer <https://github.com/fuboat/ChatApp-FullServer>`_ , `ChatApp-Common <https://github.com/fuboat/ChatApp-Common>`_ ,
`ChatApp-Client <https://github.com/fuboat/ChatApp-Client>`_


系统部署
------
使用jenkins自动部署，当git仓库的代码发生变化时自动抓取代码进行打包，并将打包之后的jar包发布到服务器上并重启后端服务。

在docker环境下运行MongoDB和redis实例。

jenkins 服务: 39.107.92.50:8080

websocket 服务：39.107.92.50:520

http 服务：39.107.92.50:7000

mongodb 服务：39.107.92.50:27017


团队成员
------
符景州 申子琳
