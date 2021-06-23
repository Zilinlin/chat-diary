项目说明
======
首先说明前端以及后端使用到的技术，之后再描述每个功能用到了提及的哪些技术。最后给出使用和参考内容。

前端技术
------
图片
++++++
图片在聊天页面和动态页面的显示使用ImageView组件。
图片的选取和全屏预览使用第三方的 **PictureSelector** 库。该库在获取相应的文件和相机权限后，
可以以网格形式浏览和预览手机中的图片，并且支持从相机新建图片。

视频
++++++
视频在聊天页面和动态页面的缩略图使用 **android.media.ThumbnailUtils** 生成。由于该库的效率问题，
缩略图的生成会有一定延迟。

视频的选取和全屏预览使用第三方的 **PictureSelector** 库。该库在获取相应的文件和相机权限后，可以以网格的形式浏览和预览手机中的视频，
并且支持从相机中录制视频。由于服务器速度较慢，该app使用的录制画质非常低，让视频尽可能小。

音频
++++++
音频的录制使用 **android.media.<ediaRecorder** 库实现。视频的播放和音频时长的倒计时使用
**android.media.MediaPlayer** 库实现。

位置信息
++++++
app支持共享当前的位置信息到会话中。其通过系统的 **LocstionManager** 来获取当前的经纬度信息并发送，并支持通过其他地图APP来查看该地理位置的详细信息
（使用intent实现）。

例如百度地图APP支持使用外连来共享地理位置。可以将该外链以文本形式分享，在会话中可以以超链接的形式点击，
系统会自动用浏览器APP打开，进入该网页，得到对应的地理位置。

页面的显示
++++++
所有页面的显示均使用安卓最新的 **Paging 3** 进行分页处理，能够很好的应对数据规模非常大的情况，如动态页面和消息页面。
其自底向上分别经过 **HttpClient, DataSource, ViewModel, Adapter** 并最终显示在 **recyclerView** 上，对于页面整体的刷新、页面单个项目的刷新都有较好的支持。
如收到新消息时，会话列表中会产生红色的新消息提醒；动态中点赞和评论时，被操作的那个动态会被刷新而其他动态不会重新加载，这节省了资源，提高了效率。

登录&登出
++++++
用户的登出、登录过期、收到新消息时各个组件的更新等功能通过 **BroadCast** 实现。
在 **websocket client** 收到新消息时会发送相应的广播。
MainActivity 收到广播后，便弹出通知、刷新消息列表和会话列表。

通知
++++++
通知的弹出使用 **NotificationManager** 等实现。
某些机型需要用户自行设置允许通知栏通知和允许横幅通知。
在收到新消息时，通知栏会弹出提醒，并且在会话列表会以红色标注收到新消息的会话。


后端技术
------
对后端（ChatApp-Server）的代码结构进行了一定的重构，将入参和出参的类定义作为前后端项目的公共代码（ChatApp-Common），
大幅减轻前后端衔接的工作量，并根据这些类编写了跨平台的 Java 客户端（ChatApp-Client），
代码可复用，这使得可以直接在每次部署时进行所有网络请求操作的单元测试，有效保证了服务的安全性和健壮性。
通过 **netty** 和 **mongodb** 的文件存储实现了静态文件服务

聊天与会话
------
（符景州）

将消息分为元数据、数据两部分。元数据包括消息的唯一id、该消息所属的会话的id、该消息对应的数据的id。
数据包括数据的唯一id、图片/视频/文本/音频等。

1. 会话中发送、接收、展示信息

- 消息的发送使用 http/https 协议。通过 okhttp 框架。用户终端首先向服务器发送请求表示将要发送消息，服务器收到该请求后，在数据库中创建相应消息的元数据，服务器响应会返回消息的元数据给用户终端。用户终端再将消息中的文本数据/图片数据等以 http/https 请求的方式传输给服务器，并在请求中提供该消息的元数据来告知服务器这是哪个消息的数据。

- 消息的接收结合 websocket 和 http/https 协议。当一个用户要接收消息时，服务器通过 websocket 将消息的元数据传输给用户。用户接收到元数据后，再通过 http/https 协议向服务器请求该消息的具体数据。
该部分都以多线程的方式（Thread, Runnable）运行，防止阻塞 UI 线程。

- 消息中涉及文字信息，图片信息，视频信息，音频信息以及位置。使用intent等和安卓的其他组件、软件进行交互，例如麦克风、相机、图库等，通过view、adapter等课程所学的安卓内容进行UI的绘制、图片的显示、声音的播放等。

2. 聊天信息在云端进行储存同步

- 聊天信息需要和云端同步，为了支持没有联网时也可以浏览的功能，各类数据需要同步机制。

- 可以将数据封装成若干数据管理类，在类中实现同步机制，该同步机制对上层不透明，当上层通过该类获取图片、视频等数据时，可以选择是否和云端同步。

- 以多线程的方式（Thread, Runnable）运行，防止阻塞UI线程。

用户与联系人
------
（符景州，申子琳）

- 用户注册登录、修改密码
- 用户修改用户名、个人信息、头像
- 用户查找、添加、删除联系人
其中遇到的比较困难的技术是 **登录&登出** 。


群聊与管理
------
（符景州）

- 发起群聊
创建群聊、邀请好友等是用往弹出的 AlterDialog 中嵌入 recyclerView 的方式来实现在会话框中进行多个用户的筛选。

- 查看群聊成员
群成员列表的显示是使用的基于 GridLayoutManager 的 recylerview。能够以网格形式显示用户列表。

- 主动退出群聊

动态发布
------
（申子琳，符景州）

- 浏览联系人发布的动态
- 发布图文动态
- 发布视频动态
- 点赞，展示点赞人信息
- 评论，展示评论信息
这部分涉及到 **页面显示** **图片** **视频** 等前端技术。

使用
------
后端 netty 的静态文件服务：基于官方项目代码进行修改: `netty <https://github.com/netty/netty/blob/d34212439068091bcec29a8fad4df82f0a82c638/example/src/main/java/io/netty/example/http/file/HttpStaticFileServerHandler.java>`_

安卓图片选择、图片预览、视频选择、视频预览：使用第三方库: `PictureSlector <https://github.com/LuckSiege/PictureSelector>`_

安卓消息列表的 UI 设计（在此基础上进行了大量的修改和添加，包括音频、视频、地理位置、图片的支持，不过最基础的 xml 代码是来自于该网页的）: `android-chat <https://sendbird.com/developer/tutorials/android-chat-tutorial-building-a-messaging-ui>`_ 

安卓 HTTP Client 和 WebSocket Client：使用 okhttp3 库: `okhttp3 <https://github.com/square/okhttp>`_

安卓部分登录的UI层和ViewModel 层：基于 Android 新建 LoginActivity 时自动生成的代码。

参考
------
UI 的设计：参考了课程提供的 Homework2、微信示例。见网络学堂。

音频播放：参考了课程提供的预习项目。

音频录制：参考了菜鸟教程的代码 https://www.runoob.com/w3cnote/android-tutorial-mediarecord.html

分页功能 Paging3，本地数据库：参考了 google 官方文档 https://developer.android.com/topic/libraries/architecture/paging/v3-overview

AsyncTask Deprecated 后，如何优雅使用多线程：参考 stackoverflow 上的回答 https://stackoverflow.com/questions/58767733/android-asynctask-api-deprecating-in-android-11-what-are-the-alternatives

视频缩略图：参考 stackoverflow 上的回答 https://stackoverflow.com/questions/37314105/android-how-to-display-video-thumbnail-in-imageview

Fragment 的切换：参考 CSDN 的方案 https://blog.csdn.net/gsw333/article/details/51858524

通知提醒功能：参考 CSDN 的方案 https://blog.csdn.net/qq_35507234/article/details/90676587