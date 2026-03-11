# 第一部分：C++ 初阶→进阶全覆盖学习计划

> 目标：这一部分严格按 `C++课件（初阶+进阶）.zip` 的**课件编号顺序**学习，先完成 C++ 初阶 01-12，再完成 C++ 进阶 01-13，确保**所有 C++ 课件全部覆盖**。这份计划是在原总计划基础上重排得到的，更强调“顺序完整、节奏连续、先语言后工程”。

> 使用原则：每天按“课件 → 官方/高质量教程 → 手写代码 → 复盘总结”完成。LearnCpp 提供了从零开始的现代 C++ 系统教程，cppreference 适合做权威语法与标准库查阅；这两类资源组合非常适合做主线学习资料。citeturn0search1turn0search0turn0search20turn0search8

## Part 1 总安排
- 周次：第 1-6 周
- 主线：C++ 初阶 12 讲 + C++ 进阶 13 讲
- 配套：只加少量巩固题，不穿插 Linux / MySQL / Redis，避免节奏被打断
- 产出：
  - `cpp_notes/`
  - `cpp_code/`
  - `cpp_review/`
  - 每周 1 份总结

---

## 第 1 周：C++ 初阶 01-04

### Day 1：01.C++入门基础（上）
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/01.C++入门基础.pdf`

**高质量资料**
- LearnCpp 首页与 Chapter 0-1：<https://www.learncpp.com/>
- cppreference 总入口：<https://en.cppreference.com/w/cpp>

**学习重点**
- 第一个 C++ 程序
- 编译、链接、运行
- 命名空间
- 输入输出
- 引用与值传递

**当天安排**
1. 配置编译环境：g++ / VS Code / Git。
2. 跟课件完成 5 个最小程序。
3. 笔记：`C++ 比 C 多了什么`。
4. 用 Git 建仓并提交第一次 commit。

**验收标准**
- 能独立写并编译 `hello.cpp`。
- 能解释引用与指针的区别。

---

### Day 2：01.C++入门基础（下）
**高质量资料**
- LearnCpp 变量、作用域、函数基础相关章节：<https://www.learncpp.com/>
- VS Code C/C++ 官方配置：<https://code.visualstudio.com/docs/languages/cpp>

**学习重点**
- 缺省参数
- 函数重载
- inline
- `const` 基础

**当天安排**
1. 手写：函数重载、缺省参数、inline demo。
2. 练习分文件编译：`main.cpp + utils.cpp + utils.h`。
3. 写一页“编译报错排查清单”。

**验收标准**
- 能说清重载与默认参数的常见冲突。

---

### Day 3：02.类和对象(上)
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/02.类和对象(上).pdf`

**高质量资料**
- LearnCpp OOP 入门：<https://www.learncpp.com/>
- cppreference class：<https://en.cppreference.com/w/cpp/language/classes>

**学习重点**
- 类、对象
- 成员变量、成员函数
- 访问限定符
- `this` 指针
- 构造函数、析构函数

**当天安排**
1. 写 `Date` 类。
2. 写 `Student` 类。
3. 用类封装“通讯录”简化版。

**验收标准**
- 能自己定义类并创建对象。

---

### Day 4：03.类和对象(中)
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/03.类和对象(中).pdf`

**高质量资料**
- LearnCpp const 成员函数：<https://www.learncpp.com/>

**学习重点**
- `const` 成员函数
- 取地址重载
- 头文件与源文件分离
- 封装与接口设计

**当天安排**
1. 将 Day 3 的类拆成 `.h/.cpp`。
2. 练习成员函数声明与定义分离。
3. 整理“类设计规范”笔记。

**验收标准**
- 多文件工程能正常编译。

---

### Day 5：04.类和对象(下)
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/04.类和对象(下).pdf`

**高质量资料**
- LearnCpp copy constructor：<https://www.learncpp.com/>

**学习重点**
- 默认成员函数
- 拷贝构造
- 赋值重载
- 浅拷贝 / 深拷贝

**当天安排**
1. 手写简化版 `String`。
2. 观察浅拷贝导致的问题。
3. 记录 Rule of Three 的动机。

**验收标准**
- 能解释为什么资源类需要自定义拷贝控制。

---

### Day 6：周内复盘与编码巩固
**当天安排**
1. 把 Day 1-5 的代码整理到 `cpp_code/week1/`。
2. 重新手写：`Date`、`String` 核心框架。
3. 输出 `week1_review.md`。

**验收标准**
- 不看课件能手写一个简单类。

---

### Day 7：轻量练习日
**当天安排**
1. 做 3-4 道基础语法题。
2. 回看本周错题与编译报错。
3. 预习内存管理与模板。

---

## 第 2 周：C++ 初阶 05-08

### Day 8：05.内存管理
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/05.内存管理.pdf`

**高质量资料**
- cppreference memory model：<https://en.cppreference.com/w/cpp/language/memory_model>
- cppreference malloc：<https://en.cppreference.com/w/cpp/memory/c/malloc>

**学习重点**
- 栈、堆、静态区、常量区
- `new/delete`
- `malloc/free`
- 定位 new 基础认知

**当天安排**
1. 画内存分区图。
2. 写 `new/delete` 与 `malloc/free` 对比 demo。
3. 整理 10 个面试问答。

**验收标准**
- 能说清 `new` 和 `malloc` 的至少 3 个差异。

---

### Day 9：06.模板初阶
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/06.模板初阶.pdf`

**高质量资料**
- cppreference function template：<https://en.cppreference.com/w/cpp/language/function_template>
- cppreference templates：<https://en.cppreference.com/w/cpp/language/templates>

**学习重点**
- 函数模板
- 类模板
- 模板实例化
- 模板与头文件

**当天安排**
1. 写模板版 `Swap`、`Max`。
2. 把顺序表改成 `SeqList<T>`。
3. 记录模板放头文件的原因。

**验收标准**
- 模板类能同时存 `int/string`。

---

### Day 10：07.STL简介
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/07.STL简介.pdf`

**高质量资料**
- cppreference containers：<https://en.cppreference.com/w/cpp/container>

**学习重点**
- 容器、算法、迭代器
- 顺序容器 / 关联容器 / 无序容器 / 适配器

**当天安排**
1. 画 STL 总图。
2. 归类常见容器。
3. 总结 `vector/list/map/unordered_map/stack/queue` 适用场景。

**验收标准**
- 能口述 STL 三大块结构。

---

### Day 11：08.string
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/08.string.pdf`

**高质量资料**
- cppreference string：<https://en.cppreference.com/w/cpp/string/basic_string>

**学习重点**
- string 构造、拼接、查找、替换
- `c_str()` 与字符数组关系

**当天安排**
1. 完成 10 个 string API 练习。
2. 做 2 道字符串题。
3. 记录 string 与 `char*` 的差异。

**验收标准**
- 能熟练用 `find/substr/append/replace`。

---

### Day 12：09.vector
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/09.vector.pdf`

**高质量资料**
- cppreference vector：<https://en.cppreference.com/w/cpp/container/vector>

**学习重点**
- `size/capacity/reserve/resize`
- 迭代器
- 扩容与均摊复杂度

**当天安排**
1. 用 vector 重写数组题 3 道。
2. 对比自己写的顺序表。
3. 总结 `reserve` 与 `resize`。

**验收标准**
- 能解释 vector 为什么适合尾插不适合头插。

---

### Day 13：本周收口
**当天安排**
1. 把内存管理 / 模板 / STL / string / vector 总结成一张知识图。
2. 做 15 个判断题式口头问答。

---

### Day 14：周复盘
**当天安排**
1. 输出 `week2_review.md`。
2. 重做本周所有 demo。
3. 预习 list / stack / queue / 模板进阶。

---

## 第 3 周：C++ 初阶 10-12 + 初阶总复习

### Day 15：10.list
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/10.list.pdf`

**高质量资料**
- cppreference list：<https://en.cppreference.com/w/cpp/container/list>

**学习重点**
- 双向链表
- 迭代器失效的直觉理解
- vector 与 list 对比

**当天安排**
1. 手写双向链表简版。
2. 对比 `std::list`。
3. 输出表格：`vector/list/deque` 对比。

**验收标准**
- 能说清 list 的优缺点。

---

### Day 16：11.stack 和 queue
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/11.stack和queue.pdf`

**高质量资料**
- cppreference stack：<https://en.cppreference.com/w/cpp/container/stack>
- cppreference queue：<https://en.cppreference.com/w/cpp/container/queue>

**学习重点**
- 适配器
- LIFO / FIFO
- 典型应用场景

**当天安排**
1. 用栈做括号匹配。
2. 用队列做层序遍历模板。
3. 记录适配器与底层容器关系。

**验收标准**
- 能解释为什么 stack/queue 本身不是底层容器。

---

### Day 17：12.模板进阶
**对应课件**
- `C++课件（初阶+进阶）/C++初阶课件/12.模板进阶.pdf`

**高质量资料**
- cppreference templates：<https://en.cppreference.com/w/cpp/language/templates>

**学习重点**
- 非类型模板参数
- 模板特化 / 偏特化
- 模板分离编译直觉

**当天安排**
1. 写类模板特化 demo。
2. 写函数模板与普通函数共存 demo。
3. 总结模板进阶最易错点。

**验收标准**
- 能解释全特化与偏特化的区别。

---

### Day 18：C++ 初阶总复习（语法）
**当天安排**
1. 从 01-12 课件中抽 30 个知识点做口头复述。
2. 回写所有重点笔记。
3. 整理 `cpp_basic_summary.md`。

---

### Day 19：C++ 初阶总复习（代码）
**当天安排**
1. 重写：`Date`、`String`、`SeqList<T>`、双向链表。
2. 做 4 道 STL 基础题。
3. 清理仓库目录。

---

### Day 20：进阶预备：面向对象与树结构预热
**高质量资料**
- cppreference derived classes：<https://en.cppreference.com/w/cpp/language/derived_class>

**当天安排**
1. 预习继承、多态概念。
2. 预习 BST / 平衡树 / 哈希表发展路线。
3. 做知识地图。

---

### Day 21：过渡日
**当天安排**
1. 只做轻量复习。
2. 进入进阶阶段前，确认初阶所有课件都已过一遍。

---

## 第 4 周：C++ 进阶 01-04

### Day 22：01.继承
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/01.继承.pdf`

**高质量资料**
- cppreference derived classes：<https://en.cppreference.com/w/cpp/language/derived_class>
- LearnCpp 继承相关章节：<https://www.learncpp.com/>

**学习重点**
- 继承方式
- 作用域与隐藏
- 基类与派生类构造析构

**当天安排**
1. 写 `Person/Student/Teacher` 继承体系。
2. 记录 `public/protected/private` 继承差异。
3. 总结“继承是否等于代码复用”。

**验收标准**
- 能画出对象构造析构顺序。

---

### Day 23：02.多态
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/02.多态.pdf`

**高质量资料**
- LearnCpp dynamic casting 与多态相关章节：<https://www.learncpp.com/cpp-tutorial/dynamic-casting/>

**学习重点**
- 虚函数
- override
- 动态绑定
- 抽象类

**当天安排**
1. 写 `Animal` 多态 demo。
2. 用基类指针数组管理多个派生类对象。
3. 记录“虚函数表”只做概念理解。

**验收标准**
- 能解释多态的运行时本质。

---

### Day 24：03.二叉搜索树
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/03.二叉搜索树.pdf`

**高质量资料**
- OI Wiki BST：<https://oi-wiki.org/ds/bst/>

**学习重点**
- 插入、查找、删除
- 中序有序特性

**当天安排**
1. 手写 BST。
2. 测试插入/删除/查找。
3. 记录删除节点 3 种情况。

**验收标准**
- 能独立写 BST 插入与查找。

---

### Day 25：04.map 和 set 的使用
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/04.map和set的使用.pdf`

**高质量资料**
- cppreference map：<https://en.cppreference.com/w/cpp/container/map>
- cppreference set：<https://en.cppreference.com/w/cpp/container/set>

**学习重点**
- 有序关联容器
- 键值对与去重
- 迭代器遍历

**当天安排**
1. 用 map 做词频统计。
2. 用 set 做去重与排序输出。
3. 对比 BST 与标准库容器。

**验收标准**
- 能熟练使用 map/set 解决基础问题。

---

### Day 26：复盘日
**当天安排**
1. 输出 `advanced_week1_review.md`。
2. 重写继承、多态、BST 关键代码。

---

### Day 27：练习日
**当天安排**
1. 做 3 道基于 map/set/BST 的题。
2. 口述 10 个面试问答。

---

### Day 28：缓冲日
**当天安排**
1. 补前面没完成的内容。
2. 预习 AVL / 红黑树。

---

## 第 5 周：C++ 进阶 05-08

### Day 29：05.AVL树实现
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/05.AVL树实现.pdf`

**高质量资料**
- OI Wiki 平衡树：<https://oi-wiki.org/ds/bst/#平衡二叉搜索树>

**学习重点**
- 平衡因子
- LL / RR / LR / RL 旋转

**当天安排**
1. 只先写旋转函数。
2. 再补插入后平衡。
3. 画图理解四种失衡。

**验收标准**
- 能口述四种旋转情况。

---

### Day 30：06.红黑树实现
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/06.红黑树实现.pdf`

**高质量资料**
- OI Wiki 红黑树：<https://oi-wiki.org/ds/rbtree/>

**学习重点**
- 红黑性质
- 插入调整
- 旋转 + 变色思想

**当天安排**
1. 先理解性质与调整规则。
2. 不要求一天内完整手写到无 bug。
3. 输出“AVL vs 红黑树”对比。

**验收标准**
- 能解释为什么红黑树常用于标准库。

---

### Day 31：07.封装红黑树实现 mymap 和 myset
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/07.封装红黑树实现mymap和myset.pdf`

**学习重点**
- 迭代器雏形
- 泛型封装
- value_type / key_type 设计

**当天安排**
1. 基于现有红黑树实现最简 `mymap/myset`。
2. 不追求全部 STL 兼容。
3. 写一页“容器封装思想”。

**验收标准**
- 能用 `mymap` 完成简单插入查询。

---

### Day 32：08.unordered_map 和 unordered_set 的使用
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/08.unordered_map和unordered_set的使用.pdf`

**高质量资料**
- cppreference unordered_map：<https://en.cppreference.com/w/cpp/container/unordered_map>

**学习重点**
- 哈希容器平均 O(1)
- bucket、冲突、装载因子概念

**当天安排**
1. 做 4 道哈希题。
2. 对比 map 与 unordered_map。
3. 总结适用场景。

**验收标准**
- 能说清无序容器为什么“快但不稳定有序”。

---

### Day 33：复盘日
**当天安排**
1. 回写 AVL / 红黑树 / mymap / unordered 使用笔记。
2. 重新画红黑树调整流程图。

---

### Day 34：练习日
**当天安排**
1. 基于 map / unordered_map 刷 4 题。
2. 基于树结构做 2 道题。

---

### Day 35：缓冲日
**当天安排**
1. 修代码。
2. 预习哈希表实现、C++11、异常、智能指针。

---

## 第 6 周：C++ 进阶 09-13 + 总收尾

### Day 36：09.哈希表实现
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/09.哈希表实现.pdf`

**学习重点**
- 开散列 / 闭散列
- 哈希冲突解决
- 负载因子

**当天安排**
1. 手写简版哈希表。
2. 做插入、查找、删除。
3. 对比标准库 unordered 容器。

**验收标准**
- 能解释链地址法与开放寻址法区别。

---

### Day 37：10.用哈希表封装 myunordered_map 和 myunordered_set
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/10.用哈希表封装myunordered_map和myunordered_set.pdf`

**当天安排**
1. 基于 Day 36 封装简化容器。
2. 只实现最核心接口。
3. 记录自定义容器设计套路。

**验收标准**
- 能完成基础插入 / 查询 / 遍历。

---

### Day 38：11.C++11
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/11.C++11.pdf`

**高质量资料**
- cppreference history / feature testing：<https://en.cppreference.com/w/cpp/language/history.html>
- feature test macros：<https://en.cppreference.com/w/cpp/experimental/feature_test.html>

**学习重点**
- auto / nullptr / range-for / lambda
- 右值引用初步
- move 语义入门

**当天安排**
1. 用 C++11 新特性重构旧代码。
2. 写 move demo。
3. 记录“哪些特性能直接提升工程体验”。

**验收标准**
- 能正确使用 auto、nullptr、range-for、lambda。

---

### Day 39：12.异常
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/12.异常.pdf`

**高质量资料**
- cppreference exceptions 入口：<https://en.cppreference.com/w/cpp/error>

**学习重点**
- try / catch / throw
- 标准异常类
- 异常安全只做基础理解

**当天安排**
1. 写 3 个异常 demo。
2. 记录异常与错误码的使用边界。
3. 笔记：构造函数抛异常要注意什么。

**验收标准**
- 能说清异常传播基本过程。

---

### Day 40：13.智能指针的使用及其原理
**对应课件**
- `C++课件（初阶+进阶）/C++进阶课件/13.智能指针的使用及其原理.pdf`

**高质量资料**
- cppreference unique_ptr：<https://en.cppreference.com/w/cpp/memory/unique_ptr>
- cppreference shared_ptr：<https://en.cppreference.com/w/cpp/memory/shared_ptr>

**学习重点**
- RAII
- unique_ptr / shared_ptr / weak_ptr
- 引用计数
- 循环引用

**当天安排**
1. 用智能指针改写一个资源类。
2. 写 shared_ptr 循环引用 demo。
3. 总结“为什么智能指针不是万能的”。

**验收标准**
- 能解释 `unique_ptr` 与 `shared_ptr` 的根本区别。`std::unique_ptr` 是独占所有权智能指针，生命周期结束时自动释放资源。citeturn0search8

---

### Day 41：C++ 进阶总复习（概念）
**当天安排**
1. 继承、多态、BST、AVL、红黑树、哈希、C++11、异常、智能指针全部做口头串讲。
2. 输出 `cpp_advanced_summary.md`。

---

### Day 42：C++ 进阶总复习（代码）
**当天安排**
1. 挑 3 个最重要模块重新手写：BST / 哈希表 / 智能指针应用 demo。
2. 修复全部遗留 bug。

---

### Day 43：C++ Part1 总结
**当天安排**
1. 确认 `C++ 初阶 12 讲 + 进阶 13 讲` 全部覆盖。
2. 产出：
   - `cpp_all_lessons_checklist.md`
   - `cpp_interview_qa.md`
   - `cpp_project_ready_notes.md`

**验收标准**
- 你已经能从“看课件”切到“进入系统 / 后端 / 项目阶段”。
