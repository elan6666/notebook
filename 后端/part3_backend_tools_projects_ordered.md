# 第三部分：MySQL / Redis / Git / Protobuf / CMake / Docker / 现代 C++ / 项目实战

> 目标：这一部分按“数据库与缓存 → 工具链 → 序列化与构建 → 容器化 → 现代 C++ → 项目”的顺序学习，尽量保持每个模块连续完成。MySQL 官方手册、Redis 官方文档、Protocol Buffers 官方教程、CMake 官方教程、Docker 官方文档都长期维护，适合直接作为计划里的主资料。citeturn0search7turn0search2turn0search9turn0search6

## Part 3 总安排
- 周次：第 13-18 周
- 主线：
  1. MySQL 按编号顺序
  2. Git
  3. Redis
  4. Protobuf
  5. CMake
  6. Docker
  7. 现代 C++ 精品课
  8. 项目：并发服务器 / MQ / 搜索引擎或内存池

---

## 第 13 周：MySQL 按编号顺序学习

> 你的 `MySQL.zip` 文件名有编码问题，但能确认是按 `0/01/02/.../14` 编号排列，因此这里按编号顺序推进。

### Day 88：0 + 01
**对应课件**
- `0-MySQL ... 安装部署`
- `01-MySQL ...`

**高质量资料**
- MySQL 8.4 手册：<https://dev.mysql.com/doc/en/>

**当天安排**
1. 安装 MySQL。
2. 建库建表。
3. 理解数据库、表、记录、字段。

---

### Day 89：02 + 03
**高质量资料**
- SELECT 基础：<https://dev.mysql.com/doc/refman/8.4/en/select.html>

**当天安排**
1. 学基础查询。
2. 练 `where/order by/group by`。
3. 做 10 条 SQL。

---

### Day 90：04 + 05
**当天安排**
1. 学表设计与约束。
2. 练增删改查。
3. 做 3 张小表练习。

---

### Day 91：06 + 07
**当天安排**
1. 学索引与视图等中级内容。
2. 做 explain 体验。
3. 记录聚簇索引直觉理解。

---

### Day 92：08 + 09
**当天安排**
1. 学事务与锁相关内容。
2. 总结 ACID / 隔离级别。

**高质量资料**
- 事务隔离级别：<https://dev.mysql.com/doc/refman/8.1/en/innodb-transaction-isolation-levels.html>
- InnoDB 锁：<https://dev.mysql.com/doc/refman/8.2/en/innodb-locking.html>

---

### Day 93：10 + 11
**当天安排**
1. 学高级查询与优化部分。
2. 整理“常见 SQL 面试问题”。

---

### Day 94：12 + 13 + 14
**当天安排**
1. 把后续高阶章节顺完。
2. 不求完全精通，重点建立目录感。
3. 输出 `mysql_summary.md`。

---

## 第 14 周：Git + Redis（连续）

### Day 95：Git 基础
**对应课件**
- `Git 原理与使用.pdf`

**高质量资料**
- Pro Git：<https://git-scm.com/book/en/v2>
- Branches in a Nutshell：<https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell>
- Basic Branching and Merging：<https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging>

**当天安排**
1. init/add/commit/log/status。
2. 分支与合并。
3. 建统一代码仓。

---

### Day 96：Git 进阶
**高质量资料**
- Rebasing：<https://git-scm.com/book/en/v2/Git-Branching-Rebasing>
- git-rebase：<https://git-scm.com/docs/git-rebase>
- Rewriting History：<https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History>

**当天安排**
1. merge vs rebase。
2. amend / reflog / cherry-pick。
3. 记录团队协作注意点。Rebase 会按原引入顺序重放提交，而 merge 是合并分支端点，最终快照可能相同但历史不同。citeturn0search3turn0search10turn0search19

---

### Day 97：Redis 安装与基础数据结构
**对应课件**
- `[子文档] Centos 安装 redis.pdf`
- `[总文档] Redis 的使用和原理.pdf`

**高质量资料**
- Redis docs：<https://redis.io/docs/latest/>
- Commands：<https://redis.io/docs/latest/commands/>

**当天安排**
1. 安装 Redis。
2. 练字符串、哈希、列表、集合、有序集合。

---

### Day 98：Redis 原理与 C++ 使用
**对应课件**
- `[子文档] Redis C++使用 样例列表.pdf`
- `[子文档] 服务端高并发分布式结构演进之路.pdf`

**高质量资料**
- Pub/Sub：<https://redis.io/docs/latest/develop/pubsub/>

**当天安排**
1. 看 C++ 接入样例。
2. 理解缓存、分布式演进场景。
3. 写一个最小 Redis demo。

---

### Day 99：Redis Pub/Sub 与缓存设计
**当天安排**
1. 做 publish/subscribe demo。
2. 对比消息队列与 Redis Pub/Sub。
3. 输出 `redis_summary.md`。

---

## 第 15 周：Protobuf + CMake（连续）

### Day 100：Protobuf 基础
**对应课件**
- `ProtoBuf 基础.pdf`

**高质量资料**
- C++ Tutorial：<https://protobuf.dev/getting-started/cpptutorial/>
- proto3 guide：<https://protobuf.dev/programming-guides/proto3/>

**当天安排**
1. 写 `contacts.proto`。
2. 生成 C++ 代码。
3. 序列化 / 反序列化 demo。

---

### Day 101：Protobuf C++ 代码理解
**对应课件**
- `ProtoBuf ... C++.pdf`

**高质量资料**
- C++ Generated Code Guide：<https://protobuf.dev/reference/cpp/cpp-generated/>
- Overview：<https://protobuf.dev/overview/>

**当天安排**
1. 看生成代码。
2. 理解字段编号、嵌套消息。
3. 输出 `protobuf_summary.md`。

---

### Day 102：CMake 入门
**对应课件**
- `CMake工程指南.pdf`

**高质量资料**
- CMake tutorial：<https://cmake.org/cmake/help/latest/guide/tutorial/>
- Step 1 tutorial：<https://cmake.org/cmake/help/latest/guide/tutorial/Getting%20Started%20with%20CMake.html>

**当天安排**
1. 建最小 CMake 工程。
2. source tree / build tree。
3. 练 `cmake -S . -B build`。

---

### Day 103：CMake 工程拆分
**高质量资料**
- `target_link_libraries`：<https://cmake.org/cmake/help/latest/command/target_link_libraries.html>
- `add_library`：<https://cmake.org/cmake/help/latest/command/add_library.html>
- `cmake-buildsystem(7)`：<https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html>

**当天安排**
1. 写静态库 + 主程序。
2. 模块拆分。
3. 统一 include/lib/src/tests 结构。CMake 教程本身就是一步一步解决常见构建系统问题的官方样例。citeturn0search2turn0search6turn0search18turn0search21

---

### Day 104：CMake 安装与测试
**高质量资料**
- install：<https://cmake.org/cmake/help/latest/command/install.html>

**当天安排**
1. 练 `install()`。
2. 练 `ctest`。
3. 输出 `cmake_summary.md`。

---

## 第 16 周：Docker + 现代 C++（连续）

### Day 105：Docker 概念与安装
**对应课件**
- `01 技术架构演进之路/02 技术架构演进之路.pdf`
- `02 Docker使用/01 Docker使用.pdf`
- `02 Docker使用/02 Docker安装.pdf`

**高质量资料**
- Docker Overview：<https://docs.docker.com/get-started/docker-overview/>

**当天安排**
1. 安装 Docker。
2. 跑 hello-world。
3. 理解镜像、容器、仓库。

---

### Day 106：Docker 使用与容器隔离
**对应课件**
- `03 空间隔离实战.pdf`
- `04 资源控制实战.pdf`
- `05 LXC容器实战.pdf`

**当天安排**
1. 理解 namespace/cgroup 只做概念。
2. 记录 Docker 实际解决了什么问题。

---

### Day 107：Dockerfile 与多阶段构建
**对应课件**
- `03 Docker核心技术和实现原理.pdf`
- `03-01 C++镜像制作.pdf`
- `03-03 CMD与EntryPoint实战.pdf`
- `03-05 多阶段构建.pdf`
- `03-06 合理使用缓存.pdf`

**高质量资料**
- Compose Quickstart：<https://docs.docker.com/compose/gettingstarted/>

**当天安排**
1. 给一个 C++ 小项目写 Dockerfile。
2. 做多阶段构建。

---

### Day 108：Docker 网络与编排
**对应课件**
- `03-09 Dockerfile配合docker-compose搭建微服务.pdf`
- `03-10 Dockerfile结合dockercompose搭建C++微服务.pdf`
- `03-11/12/13` 网络相关课件

**高质量资料**
- Networking：<https://docs.docker.com/compose/how-tos/networking/>
- Volumes：<https://docs.docker.com/engine/storage/volumes/>

**当天安排**
1. 用 compose 拉起 app + redis / mysql。
2. 输出 `docker_summary.md`。

---

### Day 109：现代 C++ 10.C++11扩展学习
**对应课件**
- `现代C++精品课课件/10.C++11扩展学习.pdf`

**高质量资料**
- cppreference history / feature testing：<https://en.cppreference.com/w/cpp/language/history.html>

**当天安排**
1. auto / nullptr / range-for / lambda / move。
2. 用新特性重构旧代码。

---

### Day 110：现代 C++ 10.C++11并发支持库
**对应课件**
- `现代C++精品课课件/10.C++11并发支持库.pdf`
- `C++11 future学习教程文档`

**高质量资料**
- cppreference thread：<https://en.cppreference.com/w/cpp/thread>
- cppreference lock_guard：<https://en.cppreference.com/w/cpp/thread/lock_guard>

**当天安排**
1. mutex / lock_guard / condition_variable / future。
2. 用标准库重写一个同步 demo。

---

### Day 111：现代 C++ 11/12/13
**对应课件**
- `11.C++14.pdf`
- `12.C++17.pdf`
- `13.C++20（上）.pdf`
- `13.C++20（下）.pdf`
- `CPU缓存知识`
- `无锁队列的实现`

**当天安排**
1. C++14/17/20 做功能感知，不求全背。
2. CPU cache / 无锁队列只做理解。
3. 输出 `modern_cpp_summary.md`。

---

## 第 17-18 周：项目实战（连续完成）

> 这一阶段直接参考你上传的项目资料：
- `C++ - 仿mudou库one thread one loop式并发服务器实现.pdf`
- `C++ - 仿RabbitMQ实现消息队列.pdf`
- `C++ - Boost搜索引擎.pdf`
- `C++ - 高性能内存池.pdf`

这些资料已经定义了项目骨架：并发服务器包含 Buffer / Channel / Poller / EventLoop / TcpServer 等模块；RabbitMQ 项目用 C++、Protobuf、muduo、SQLite3、GTest 组合实现消息队列；搜索引擎覆盖 Parser、正排/倒排索引、分词等；高性能内存池围绕 thread cache / central cache / page cache 设计。fileciteturn1file1 fileciteturn1file2 fileciteturn1file0 fileciteturn1file3

### Day 112-116：项目一 仿 muduo 并发服务器
**当天安排**
1. Day112：读项目资料、画模块图。
2. Day113：做 Buffer / Socket / Channel。
3. Day114：做 Poller / EventLoop。
4. Day115：做 Connection / TcpServer / echo。
5. Day116：补 TimerQueue / README。

---

### Day 117-121：项目二 仿 RabbitMQ 消息队列
**当天安排**
1. Day117：读项目资料、画消息流转图。
2. Day118：Producer / Broker / Consumer 最小链路。
3. Day119：exchange / queue / binding 抽象。
4. Day120：Protobuf 协议接入。
5. Day121：日志、测试、README。

---

### Day 122-125：项目三 二选一
**路线 A：Boost 搜索引擎**
1. Parser。
2. 正排 / 倒排索引。
3. 简单查询。
4. README。

**路线 B：高性能内存池**
1. 定长对象池。
2. thread cache。
3. central cache / page cache 理解。
4. README。

---

### Day 126：总收尾
**当天安排**
1. 给所有项目补 CMakeLists、Dockerfile、README。
2. 输出简历项目 bullet。
3. 准备面试讲解稿。

**最终产出**
- `backend_project_portfolio.md`
- `resume_project_bullets.md`
- `interview_talk_track.md`
