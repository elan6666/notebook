# 第二部分：数据结构 / 算法 / Linux（按课件顺序、连续推进）

> 目标：这一部分不再穿插 C++ 语言课，而是顺着“数据结构 → 算法强化 → Linux 系统编程 → Linux 网络编程”的顺序连续推进。这样节奏更稳定，也更适合后面项目开发。原计划已经把这些内容作为 C++ 之后的下一阶段主线。fileciteturn2file0

> 使用原则：尽量**一个模块连续学完**，例如 Linux 系统编程 1-11 基本按编号顺序完成，再进入 Linux 网络编程 1-14。Beej’s Guide、man7、OI Wiki 这些资料长期维护，适合作为配套参考。citeturn0search2turn0search3

## Part 2 总安排
- 周次：第 7-12 周
- 主线：
  1. 数据结构课件
  2. C++ 高阶数据结构公开课
  3. 算法精品课 + 笔试强训
  4. Linux 系统编程
  5. Linux 网络编程
- 产出：
  - `ds_algo_notes/`
  - `linux_notes/`
  - `linux_code/`

---

## 第 7 周：数据结构课件按顺序学习（Lesson1-6）

### Day 44：Lesson1 数据结构前言
**对应课件**
- `数据结构课件/Lesson1--数据结构前言.pdf`

**高质量资料**
- OI Wiki 数据结构总览：<https://oi-wiki.org/ds/>

**当天安排**
1. 明确数据结构在后端/项目中的位置。
2. 整理数据结构总图。
3. 写一页“顺序表、链表、树、图、哈希”的全景图。

---

### Day 45：Lesson2 时间复杂度空间复杂度
**对应课件**
- `数据结构课件/Lesson2--时间复杂度空间复杂度.pdf`

**高质量资料**
- OI Wiki 复杂度：<https://oi-wiki.org/basic/complexity/>

**当天安排**
1. 写 10 个复杂度分析示例。
2. 建“复杂度速查表”。

---

### Day 46：Lesson3 顺序表 / 链表
**对应课件**
- `数据结构课件/Lesson3--顺序表_链表.pdf`

**当天安排**
1. 重温顺序表与链表。
2. 用 C++ 模板封装一版数据结构练习。
3. 做 2 道链表题。

---

### Day 47：Lesson4 栈和队列
**对应课件**
- `数据结构课件/Lesson4--栈和队列.pdf`

**当天安排**
1. 手写顺序栈、链式队列。
2. 刷 3 道栈/队列题。

---

### Day 48：Lesson5 二叉树
**对应课件**
- `数据结构课件/Lesson5--二叉树.pdf`

**高质量资料**
- OI Wiki 二叉树：<https://oi-wiki.org/ds/binary-tree/>

**当天安排**
1. 前中后序 + 层序。
2. 做最大深度 / 层序遍历题。

---

### Day 49：Lesson6 排序
**对应课件**
- `数据结构课件/Lesson6--排序.pdf`

**高质量资料**
- OI Wiki 排序：<https://oi-wiki.org/basic/sort-intro/>

**当天安排**
1. 手写快排、归并、堆排。
2. 输出排序对比表。

---

### Day 50：周复盘
**当天安排**
1. 汇总数据结构代码。
2. 形成 `ds_week1_summary.md`。

---

## 第 8 周：C++ 高阶数据结构公开课 + 算法精品课起步

### Day 51：并查集
**对应课件**
- `C++高阶数据结构公开课课件/Lesson02---并查集.pdf`

**高质量资料**
- OI Wiki 并查集：<https://oi-wiki.org/ds/dsu/>

**当天安排**
1. 手写路径压缩 + 按秩合并。
2. 做 2 道连通块题。

---

### Day 52：图
**对应课件**
- `C++高阶数据结构公开课课件/Lesson03---图.pdf`

**高质量资料**
- OI Wiki 图论入门：<https://oi-wiki.org/graph/basic/>

**当天安排**
1. 邻接表实现图。
2. 写 BFS / DFS 模板。

---

### Day 53：LRU Cache
**对应课件**
- `C++高阶数据结构公开课课件/Lessson04---LRU Cache.pdf`

**当天安排**
1. 用哈希 + 双向链表实现 LRU。
2. 记录 O(1) 的来源。

---

### Day 54：跳表
**对应课件**
- `C++高阶数据结构公开课课件/Lessson05---跳表.pdf`

**当天安排**
1. 重点做理解，不强求完整实现。
2. 输出图解笔记。

---

### Day 55：算法精品课 - 递归/回溯
**对应课件**
- `算法精品课/递归、搜索与回溯算法精品课.pdf`

**高质量资料**
- 代码随想录回溯：<https://programmercarl.com/回溯算法理论基础.html>

**当天安排**
1. 写回溯模板。
2. 做组合、子集、全排列。

---

### Day 56：算法精品课 - 动态规划
**对应课件**
- `算法精品课/动态规划精品课.pdf`

**高质量资料**
- 代码随想录动态规划：<https://programmercarl.com/动态规划理论基础.html>

**当天安排**
1. 学 DP 五部曲。
2. 做爬楼梯、打家劫舍、背包入门。

---

### Day 57：笔试强训起步
**对应课件**
- `笔试强训/第01周`、`第02周` 相关资料

**当天安排**
1. 做 5-6 道基础题。
2. 建错题本。

---

## 第 9 周：Linux 系统编程 1-5（连续）

### Day 58：1. Linux基础指令
**对应课件**
- `Linux课件2025/Linux 系统编程/1. Linux基础指令.pdf`

**高质量资料**
- man7 总入口：<https://man7.org/linux/man-pages/>
- tldr：<https://tldr.sh/>

**当天安排**
1. 完成文件、权限、查找、压缩、进程查看练习。
2. 写 30 条高频命令速查。

---

### Day 59：2. 基础开发工具
**对应课件**
- `Linux课件2025/Linux 系统编程/2. 基础开发工具.pdf`

**高质量资料**
- GCC 官方文档：<https://gcc.gnu.org/onlinedocs/>
- GDB 文档：<https://www.sourceware.org/gdb/documentation/>

**当天安排**
1. g++ 多文件编译。
2. gdb 调试链表/BST 程序。
3. 写第一个 Makefile。

---

### Day 60：3. 进程概念
**对应课件**
- `Linux课件2025/Linux 系统编程/3. 进程概念.pdf`

**高质量资料**
- fork 手册：<https://man7.org/linux/man-pages/man2/fork.2.html>

**当天安排**
1. 写 `fork()` demo。
2. 观察父子进程。
3. 写“程序 vs 进程”笔记。

---

### Day 61：4. 进程控制
**对应课件**
- `Linux课件2025/Linux 系统编程/4. 进程控制.pdf`

**高质量资料**
- exec 手册：<https://man7.org/linux/man-pages/man3/exec.3.html>
- wait 手册：<https://man7.org/linux/man-pages/man2/wait.2.html>

**当天安排**
1. 做 fork-exec-wait 完整 demo。
2. 画进程生命周期图。

---

### Day 62：5. 基础IO
**对应课件**
- `Linux课件2025/Linux 系统编程/5. 基础IO.pdf`

**高质量资料**
- open：<https://man7.org/linux/man-pages/man2/open.2.html>
- read：<https://man7.org/linux/man-pages/man2/read.2.html>
- write：<https://man7.org/linux/man-pages/man2/write.2.html>

**当天安排**
1. 写 mini-cat。
2. 写日志文件追加程序。

---

### Day 63：周复盘
**当天安排**
1. 输出 `linux_sys_week1.md`。
2. 把 demo 整理入仓。

---

### Day 64：缓冲日
**当天安排**
1. 补漏。
2. 预习文件系统、库、IPC、信号、线程。

---

## 第 10 周：Linux 系统编程 6-11（连续）

### Day 65：6. Ext系列文件系统
**对应课件**
- `Linux课件2025/Linux 系统编程/6. Ext系列文件系统.pdf`

**当天安排**
1. 理解 inode / block 只做概念层。
2. 写“文件从路径到数据”的过程笔记。

---

### Day 66：7. 库制作与原理
**对应课件**
- `Linux课件2025/Linux 系统编程/7. 库制作与原理.pdf`

**当天安排**
1. 做静态库与动态库 demo。
2. 理解链接过程。

---

### Day 67：8. 进程间通信
**对应课件**
- `Linux课件2025/Linux 系统编程/8. 进程间通信.pdf`
- `Linux系统部分加餐课/加餐 - System V - 基于建造者模式的信号量.pdf`
- `Linux系统部分加餐课/加餐 - System V - 责任链模式与消息队列.pdf`

**高质量资料**
- pipe：<https://man7.org/linux/man-pages/man2/pipe.2.html>

**当天安排**
1. 管道 demo。
2. 理解消息队列、共享内存、信号量适用场景。

---

### Day 68：9. Linux进程信号
**对应课件**
- `Linux课件2025/Linux 系统编程/9. Linux进程信号.pdf`

**当天安排**
1. 做 signal demo。
2. 记录常见信号用途。

---

### Day 69：10. 线程概念与控制
**对应课件**
- `Linux课件2025/Linux 系统编程/10. 线程概念与控制.pdf`

**当天安排**
1. pthread 创建与回收。
2. 和进程对比。

---

### Day 70：11. 线程同步与互斥
**对应课件**
- `Linux课件2025/Linux 系统编程/11. 线程同步与互斥.pdf`

**当天安排**
1. 生产者消费者模型。
2. 记录 mutex / 条件变量 / 死锁。

---

### Day 71：加餐收口
**对应课件**
- Makefile进阶、mmap、读写锁、自旋锁等加餐

**当天安排**
1. 只抓与你项目最相关的：Makefile、mmap、读写锁。
2. 不求全刷完。

---

## 第 11 周：Linux 网络编程 1-7（按顺序）

### Day 72：1 网络基础概念
### Day 73：2 Socket编程UDP + 2-1/2-2 加餐
### Day 74：3 Socket编程TCP + 3-1/3-2 加餐
### Day 75：4 应用层自定义协议与序列化 + 4-1/4-2 加餐
### Day 76：5 应用层协议HTTP + 5-1/5-2 加餐
### Day 77：6 传输层协议UDP
### Day 78：7 传输层协议TCP + 7-1 抓包加餐

**高质量资料（本周通用）**
- Beej's Guide：<https://beej.us/guide/bgnet/>
- socket：<https://man7.org/linux/man-pages/man2/socket.2.html>

**每天统一安排**
1. 先看课件。
2. 再配合协议栈知识和 API 文档。
3. 每天至少做一个网络 demo 或画一个协议流程图。

**本周验收标准**
- 能独立写阻塞式 TCP/UDP client-server。
- 能解释 HTTP、TCP、UDP 的层次关系。

---

## 第 12 周：Linux 网络编程 8-14 + Part2 总复习

### Day 79：8 网络层 + 8-1 IP 分片加餐
### Day 80：9 数据链路层 + 9-1 ARP 加餐
### Day 81：10 NAT、代理、内网穿透 + 10-1/10-2 加餐
### Day 82：11 五种IO模型与非阻塞IO
### Day 83：12 select + 12-1 poll 改写
### Day 84：13 epoll
### Day 85：14 Reactor反应堆模式

**高质量资料**
- epoll：<https://man7.org/linux/man-pages/man7/epoll.7.html>
- timerfd：<https://man7.org/linux/man-pages/man2/timerfd_create.2.html>

**每天统一安排**
1. 先过课件。
2. 对照 man7 与网络教程补概念。
3. 第 82-85 天重点做 IO 模型、epoll、Reactor 理解与 demo。

### Day 86：Part2 综合复盘
**当天安排**
1. 输出：
   - `ds_algo_summary.md`
   - `linux_system_summary.md`
   - `linux_network_summary.md`
2. 做一次综合口述：从数据结构讲到 epoll / Reactor。

### Day 87：缓冲与补漏
**当天安排**
1. 补掉之前没跟上的 demo。
2. 进入数据库与工程化阶段前，确认 Linux 主线已打通。
