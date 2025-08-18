---
title: muduo网络库
comments: false
tags:
  - C++
  - muduo
excerpt: 山河远阔，人间星河
toc: false
date: 2025-08-11 15:25:00
cover: 'https://pic.netbian.com/uploads/allimg/250708/233123-17519886834bc3.jpg'
---
# muduo网络库
## 总体架构
Muduo 主要由三大部分组成：
- 基础工具模块（Base）

Logger、Timestamp、Thread、ThreadPool等提供线程安全、时间戳、日志等基础设施

- 网络模块（Net）

核心类：EventLoop、Channel、Poller、Socket、Acceptor、TcpServer、TcpConnection、Connector、Buffer基于 Reactor 模型，用 epoll 实现 I/O 复用

- 应用层接口
事件回调（connectionCallback / messageCallback / writeCompleteCallback）与 Muduo 交互的
## 核心运行机制 — 主从Reactor 模型
```
[主线程]
  ↓
EventLoop(主) — 监听新连接 —> 新连接交给某个 I/O 线程的 EventLoop
  ↓
[多个I/O线程]
EventLoop(IO) — 监听读写事件 —> 调用回调函数处理
```
## 三大核心组建
### EventLoop
EventLoop 是 一条线程内的**事件调度器**：负责阻塞等待内核 I/O 事件（通过 Epoller），把就绪的 Channel 分发处理，并按需执行定时任务与跨线程提交的回调。
#### loop(int timeout = -1)
```
主循环：while (!quit_) { poller->poll(...); for each activeChannel -> channel->handleEvent(...); doPendingFunctors(); }

真正“执行事件”的地方是：`channel->handleEvent(...)` → 再进而调用 `readCallback/writeCallback`（即你的业务回调）。
```
#### quit()
```
设置 quit_ = true，并 wakeup()（如果在另一个线程里调用），确保阻塞在 epoll_wait 的线程能及时退出循环。
```
#### runInLoop(Functor cb)
```
如果当前调用线程是 loop 所在线程，则直接执行 cb()（立即执行、同步）。

否则调用 queueInLoop(cb)（放入队列并 wakeup() loop），保证 cb 在 loop 线程执行。
```
#### queueInLoop(Functor cb)
```
将 cb 推入 pendingFunctors_（用 mutex 保护）。

如果调用线程不是 loop 线程，或当前 loop 正在执行 pendingFunctors_（callingPendingFunctors_ 为 true），要 wakeup() loop，让它尽快处理新的任务。
```
### Epoller
Epoller 是对 Linux epoll 的封装，它把操作系统的 I/O 多路复用（epoll_create1/epoll_ctl/epoll_wait）封装成更易用、面向对象的接口。它的职责包括：
```
- 创建并持有一个 epoll 实例（epollfd）。

- 将 Channel 注册/更新/移除到 `epoll`（通过 epoll_ctl）。

- 调用 epoll_wait 等待事件，收集就绪事件并把对应的 Channel* 塞回上层（fillActiveChannels）。

- 处理一些错误/信号中断（EINTR）和数组扩容（events_ vector）。
```
#### poll(timeoutMs, activeChannels)
调用 int n = ::epoll_wait(epollfd_, events_.data(), events_.size(), timeoutMs);
返回值通常还会封装一个 Timestamp（表示 epoll_wait 返回的时间），传给上层 EventLoop。
#### fillActiveChannels(numEvents, activeChannels)
把`events_[i]` 中的 `data.ptr`（或 data.fd）转换为 `Channel*`，设置 **Channel 的 revents（就绪事件位）**，并 `push_back` 到 `activeChannels`;
#### updateChannel(Channel*)
- 把 Channel 的期望事件注册/修改到 epoll。
- 维护 Channel 的状态（new/added），用来决定 ADD / MOD。
- 若 epoll_ctl 失败，要根据 errno 做合理处理并记录日志（例如 EEXIST 表明意外重复 add）。
#### removeChannel(Channel*)
从 epoll 中删除 fd（EPOLL_CTL_DEL）;
#### epoll_event.events 常见标志（以及 Muduo 常用）
```
- EPOLLIN / POLLIN：可读
- EPOLLOUT / POLLOUT：可写
- EPOLLERR：错误
- EPOLLHUP：挂起（连接被对端关闭/异常）
- EPOLLPRI / POLLPRI：紧急数据（很少用）
- EPOLLRDHUP：对端关闭写端（半关闭），Muduo 常监听它以更快感知 EOF
- EPOLLET：边缘触发（ET）
- EPOLLONESHOT：one-shot 行为（常用于多线程共享 fd 场景）
```
### Channel
- Channel 封装 一个 fd + 它关心的事件掩码 + 这些事件发生时要调用的回调。
- 它不持有 fd 的生命周期（不 close fd），只是作为 EventLoop ⇄ Poller ⇄ 业务回调 的桥梁。
#### handleEvent — 事件真正被执行的地方
判断处理的类型，并执行对应的回调函数
#### 线程安全与归属
- Channel 属于某个 EventLoop（创建时记录 loop_），其大部分方法必须在该 loop 线程调用（assertInLoopThread()）。

- 如果要从其他线程修改 Channel 的事件，应该通过 `EventLoop::runInLoop/queueInLoop` 让 loop 线程执行修改（以确保线程安全）。
## muduo业务层
### TcpConnection
- 表示一个 TCP 连接（客户端连接到服务器，或者客户端连接到远程服务器）
- 管理该连接的状态（连接中、已连接、断开等）
- 封装读写操作和缓冲区管理，实现高效的非阻塞 IO
- 绑定一个 Channel 来监听该连接的读写事件
- 保存各种业务回调，当连接建立、关闭、读写事件时通知业务层
- 保证线程安全，大部分操作在 EventLoop 线程里执行
#### 构造函数
里面设置了各种回调，去处理读写事件，关闭连接，和错误
#### send(message)
- 先尝试直接写入 socket（如果缓冲区为空），写不完的数据放入 outputBuffer_。
- 打开 `Channel` 的写事件监听（enableWriting()），等待 epoll 告诉你可写后继续写。
- 发送完成后调用 `writeCompleteCallback_`，通知业务层数据已经全部发出。
### TcpServer
#### 构造函数
初始化主循环、Acceptor 监听指定地址端口
默认不启用端口复用，可以选用 kReusePort
关联新连接回调，准备接收连接
#### start()
启动服务器：
```
启动 IO 线程池
启动 Acceptor 开始监听 socket
```
这个函数只允许调用一次
#### newConnection(int sockfd, const InetAddress& peerAddr)
- 新连接建立时被调用（由`Acceptor` 触发）
- 生成唯一连接名字
- 选择一个 `IO EventLoop`（通过线程池轮询）

- 创建新的 `TcpConnection` 对象，绑定对应 EventLoop 和 socket fd

- 绑定 `TcpConnection` 的各种回调（连接、消息、关闭、写完成等）

- 把连接加入 `connections_` 容器管理

- 调用 `TcpConnection::connectEstablished()` 通知连接建立
#### removeConnection(const TcpConnectionPtr& conn)

- 连接关闭时被调用

- 从 `connections_` 容器移除连接

- 调用 `TcpConnection::connectDestroyed()` 执行清理

- 这个函数会被 `CloseCallback` 触发
## Muduo 的网络封装层
### Acceptor
#### 构造函数
```
- 创建非阻塞监听 socket
- 设置 SO_REUSEADDR / SO_REUSEPORT
- 绑定监听地址
- 用 Channel 监控监听 socket 的读事件
- 绑定读事件回调到 handleRead()
```
#### Acceptor::listen()
调用 listen() 后，监听 socket 的 EPOLLIN 事件就会被 EventLoop 关注。
当有新连接到来，epoll_wait() 会返回监听 fd 可读，触发 handleRead()。
#### handleRead()
新连接建立后，去触发回调函数

### Socket（套接字封装类）
```
- 管理 socket 文件描述符的生命周期（RAII，析构时自动 close）
- 提供安全、易用的 socket API 调用接口
- 避免忘记设置 SO_REUSEADDR、TCP_NODELAY 等常见坑
```
### InetAddress（网络地址类）
```
- 封装 IP 地址 和 端口号（IPv4 / IPv6）
- 提供与 Linux sockaddr_in 互转的方法
- 负责字节序转换（host ↔ network）
```
## muduo基础工具
### Buffer
- 封装了 TCP 收发数据的内存缓冲区
- 提供高效、连续的字节存储，避免手动 malloc/free 和拷贝
- 方便按需读取/写入，不用一次性全部处理
```
TcpConnection 有两个 Buffer：inputBuffer_（接收） 和 outputBuffer_（发送）
```
### Timer / TimerQueue（定时器）
#### 作用

- 在 EventLoop 中实现高效的定时任务管理

- 支持一次性任务和周期性任务

- 使用 Linux timerfd 机制与 epoll 集成

#### 为什么要有 Timer？

在网络编程中，你可能需要：
- 定时关闭空闲连接（心跳超时）

- 定时发送心跳包

- 延迟执行某个任务
这些任务必须和 Reactor 的事件循环集成，否则就得自己写一套线程安全的定时器机制。
## 自己实现的muduo功能

├── base
│   ├── CurrentThread.cc
│   ├── CurrentThread.h
│   ├── logger.cc
│   ├── logger.h
│   ├── logStream.cc
│   ├── logStream.h
│   ├── noncopyable.h
│   ├── Timestamp.cc
│   └── Timestamp.h
├── CMakeLists.txt
└── net
    ├── Acceptor.cc
    ├── Acceptor.h
    ├── Buffer.cc
    ├── Buffer.h
    ├── Channel.cc
    ├── Channel.h
    ├── Connector.cc
    ├── Connector.h
    ├── Epoller.cc
    ├── Epoller.h
    ├── EventLoop.cc
    ├── EventLoop.h
    ├── EventLoopThread.h
    ├── EventLoopThreadpool.cc
    ├── EventLoopThreadpool.h
    ├── InetAddress.h
    ├── Poller.h
    ├── sigpipe.h
    ├── Socket.h
    ├── SocketOps.cc
    ├── SocketOps.h
    ├── TcpClient.cc
    ├── TcpClient.h
    ├── TcpConnection.cc
    ├── TcpConnection.h
    ├── TcpServer.cc
    ├── TcpServer.h
    ├── Timer.cc
    ├── Timer.h
    ├── TimerId.h
    ├── TimerQueue.cc
    └── TimerQueue.h