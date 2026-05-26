# cpp-webserver

用 C++17 在 Linux 下实现的高并发 HTTP 服务器。基于主从 Reactor 模式 + epoll，支持静态资源访问、Keep-Alive、异步日志、连接超时管理等特性。

当前状态： 阶段 0 — 项目骨架搭建中｜最近更新：2026-05-26

项目目标
从零实现一个能并发处理上千 HTTP 请求的 Web 服务器，所有核心模块均独立设计与实现，不直接依赖现成网络库（如 muduo / boost.asio）。
项目并非旨在与成熟开源项目比拼性能，而是把计算机基础四大核心课（数据结构 / 操作系统 / 计算机网络 / 计算机组成原理）的知识在代码层面打通，做到「每个设计决策都讲得出底层原因」。
整体架构（规划）
主从 Reactor 模式：主线程的 EventLoop 负责 accept 新连接，工作线程池中每个线程持有独立的 EventLoop，分管已建立连接的读写。每个连接绑定到固定的从 EventLoop 后不再迁移，避免跨线程数据竞争、无需对连接对象加锁。
              ┌──────────────────┐
              │  Main EventLoop  │  ← accept 新连接
              └────────┬─────────┘     Round-Robin 分发
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ Worker 1 │   │ Worker 2 │   │ Worker N │  ← 每个 Worker 一个 EventLoop
  │ EventLoop│   │ EventLoop│   │ EventLoop│    处理本线程上所有连接的 I/O
  └──────────┘   └──────────┘   └──────────┘
# 模块清单
## 模块职责状态
1 异步日志系统单例 + 阻塞队列实现的异步落盘，分级输出（DEBUG/INFO/WARN/ERROR）⬜ 未开始
2 线程池N 个工作线程 + 任务队列，支持优雅退出⬜ 未开始
3 epoll 事件分发器Channel + EventLoop + Epoller 三层封装，ET 模式 + 非阻塞 socket⬜ 未开始
4 Reactor 主循环主从 Reactor + 连接管理（TcpConnection）⬜ 未开始
5 HTTP 解析与响应主从状态机解析请求；mmap/sendfile 发送响应；支持 Keep-Alive⬜ 未开始
6 定时器小根堆管理空闲连接，超时自动断开⬜ 未开始
7 数据库连接池RAII 风格的 MySQL 连接复用，支持简单用户注册/登录⬜ 未开始
状态图示：⬜ 未开始｜🟡 进行中｜✅ 已完成
技术栈

语言：C++17
平台：Linux（Ubuntu 22.04 / WSL2）
构建：Makefile（后期视情况迁移到 CMake）
关键系统调用：epoll、mmap / sendfile、pthread
数据库：MySQL（连接池阶段引入）
压测工具：wrk / webbench
调试工具：gdb、valgrind

项目结构
cpp-webserver/
├── README.md
├── .gitignore
├── docs/                 # 设计笔记、源码阅读笔记、压测报告
├── examples/             # 学习阶段的练习代码（echo server 等）
├── src/                  # 主项目源码（按模块组织，后期填入）
│   ├── db/
│   ├── http/
│   ├── log/
│   ├── net/
│   ├── thread/
|   └── timer/
└── tests/                # 单元测试与压测脚本
