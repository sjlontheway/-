# WebRTC 基础学习

## 什么是WebRTC

WebRTC 全称是 Web Real-Time Communication,它使web应用或者网站可以捕获音视频流在浏览器之间点对点传输的技术，它无需中间人进行转发。

### 兼容性

由于WebRTC技术仍然处于发展阶段，不同的浏览器对codecs(编解码器) 与 WebRTC的特性支持程度不一致,官方文档强力推荐[Adapter.js](https://github.com/webrtcHacks/adapter) 库来处理兼容性问题.

Adapterjs 使用shims 与 polyfills 来抹平浏览器之间的差异性,另外它还会处理CSS前缀以及一些特性命名的差异,让我们更加方便的进行WebRTC应用开发

### 概念以及用法

WebRTC 有多种用途, 基于媒体捕获与流API,WebRTC技术为web端提供强大的多媒体能力,主要用途有音视频会议、文件传输、屏幕分享、身份管理、以及与传统电话系统的接口，包括支持发送DTMF(按键式拨号)信号;

点对点连接(```Connections```)无需借助其他的驱动以及插件就能进行，通常也不需要中间服务器即可完成

两个端的连接被 ```RTCPeerConnection``` 接口表示,当一个RTCPeerConnection 被建立和打开时，媒体信息流(```MediaStreams```)和 数据Channels(```RTCDataChannels```) 就可以添加到连接

媒体流由任意数量的媒体信息轨道组成，轨道数据通常用 ```MediaStreamTrack```对象来表示，可能包含多种媒体数据类型中的一种，包括音频、视频和文本(如字幕甚至章节名称)。大多数流包括至少一个音轨和一个视频音轨，可以用来发送和接收现场媒体或存储的媒体信息(如流媒体电影).

也可以利用 RTCDataChanel 进行点对点任意二进制数据流传输, 它可以用于后台通道信息、元数据交换、游戏状态包、文件传输，甚至可以用作数据传输的主要通道.

### 使用指南

#### 连接建立与管理

接下来是对建立、打开、管理WebRTC连接所需用到的接口、字典以及类型一些说明

##### Interfaces

- RTCPeerConnection:
    表示一个本地电脑与远端web WebRTC 连接，用来处理两端有效数据流

- RTCDataChannel
    表示一个点对点连接的双向数据通道

- RTCDataChannelEvent
    表示一个双向数据通道依附到一个连接产生的事件

- RTCSessionDescription
    表示session的参数，用来描述session的信息,是SDP(session description protocol)

- RTCStatsReport
    提供连接或者一个连接上单个轨道的详细统计信息，可以通过RTCPeerConnection.getStats()来获取

- RTCIceCandidate
    表示一个待建立连接服务器,ICE 全称为(Interactive Connectivity Establishment)

- RTCIceTransport
    表示ICE传输的信息

- RTCPeerConnectionIceEvent
    表示ICE候选对象与目标建立连接相关的事件。只有一种事件类型:icecandidate

- RTCRtpSender
    管理一个连接的流媒体数据的编码与传输

- RTCRtpReceiever
    管理一个连接流媒体数据的接收与解码

- RTCTrackEvent
    表示音轨事件,表示一个 RTCRtpReceiver 对象加入 RTCPeerConnection 对象，表示一个新的媒体流轨道数据的创建以及加入到RTCPeerConnection

- RTCSctpTransport
    提供描述流控制传输协议(SCTP)的传输信息,还提供了一种访问底层数据报传输层安全(DTLS)传输的方法，RTCPeerConnection的所有数据通道的SCTP包都是通过该传输发送和接收的

##### 字典数据

- RTCConfiguration
    用于提供RTCPeerConnection 参数

- RTCIceServer
    定义如何连接到单个ICE服务(如STUN或TURN服务)

- RTCRtpContributiongSource
    包含有关给定贡献源（CSRC）的信息，包括该源贡献的数据包最近播放的时间


## 协议介绍

### ICE
