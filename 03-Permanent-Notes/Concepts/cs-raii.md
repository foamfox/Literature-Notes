---
type: permanent
subtype: concept
source: web
status: 🟢成熟
created: 2026-10-21
tags: [C++, RAII, 资源管理, 异常安全]
domain: [编程语言, 软件工程]
related-concepts: [对象生命周期, 所有权, 析构函数, 栈展开, 智能指针, 异常安全]
source-literature:
  - C++ Core Guidelines
  - C++ 工作草案
  - Bjarne Stroustrup — C++ Style and Technique FAQ
  - 中文维基百科 — RAII
---

# RAII（资源获取即初始化）

> **类型**：概念卡片  
> **领域**：C++ / 资源管理 / 异常安全  
> **全称**：Resource Acquisition Is Initialization  
> **核心问题**：如何让资源在正常返回、提前返回和异常退出时得到一致的清理？

## 1. 定义

### 一句话定义

> RAII 是一种将资源的管理责任封装进对象、并利用对象析构自动完成资源释放的编程惯用法。[1]

这里的“资源”包括动态内存、文件句柄、套接字和互斥锁的持有权等需要成对获取与释放的事物。RAII 将这种配对关系写进类型，使调用方主要关注资源的使用。[1]

### 核心机制

1. **建立管理责任**：初始化资源管理对象时，获取资源或接管已有资源的所有权。
2. **使用资源**：对象维持约定的有效状态；接口可以允许明确的“空状态”，例如空的 `std::unique_ptr`。[4]
3. **自动清理**：对象析构时，释放其仍持有的资源。对自动存储期的局部对象，离开作用域通常就会触发析构。[2]

因此，“初始化”不要求每个构造函数都实际申请资源；关键在于**资源由谁负责释放，以及这份责任何时结束**。例如 `std::unique_ptr` 可以为空、接管指针，也可以通过移动转交所有权。[4]

### 类比

把资源交给一位随任务同行的保管员：任务结束时，保管员负责归还自己仍保管的物品。调用方无需在每条退出路径上分别记住“归还”。

## 2. 生命周期与异常安全

RAII 主要依赖语言的对象生命周期规则，适合用执行顺序说明，无须引入数学公式。

```text
初始化管理对象 → 获取／接管资源 → 使用资源 → 析构管理对象 → 释放所持资源
```

对同一作用域中依次成功构造的局部对象，正常退出或异常栈展开时，析构顺序与构造完成顺序相反。例如先构造 A、再构造 B，退出时先析构 B、再析构 A。[2]

**构造失败的细节**：普通的非委托构造函数抛出异常时，完整对象的析构函数不会执行；已经完成构造的基类和成员子对象会被析构。因此，应让已获取的资源尽早进入 RAII 成员，避免先把资源存入裸指针、随后因构造失败而泄漏。[2]

## 3. 最小示例：自动解锁

以下示例适用于 C++11 及之后版本。调用方保证 `mutex` 和 `values` 有效，并对共享数据使用一致的加锁方式。

```cpp
#include <mutex>
#include <stdexcept>
#include <vector>

void append(std::mutex& mutex, std::vector<int>& values, bool fail)
{
    std::lock_guard<std::mutex> guard(mutex); // 构造时加锁
    values.push_back(42);                    // 也可能抛出异常
    if (fail) {
        throw std::runtime_error("operation failed");
    }
} // 正常返回或异常栈展开时，guard 析构并解锁
```

`std::lock_guard` 的普通构造函数调用 `lock()`，析构函数调用 `unlock()`；被引用的互斥量必须比守卫对象活得更久。[3]

**本例的保证**：成功加锁后，正常返回或异常传播到调用方的处理器时，锁会释放。若 `push_back` 已成功，后续抛出异常不会自动删除刚加入的元素。这里的资源清理不等于业务状态回滚；后者需要额外的事务或回滚设计。

## 4. 使用原则与边界

| 关注点 | 要点 |
| --- | --- |
| 优先使用现有类型 | 常见选择包括 `std::vector`、`std::string`、`std::unique_ptr` 和 `std::lock_guard`。[1][3][4] |
| 明确所有权 | 独占所有权可用 `std::unique_ptr`；只有确需共享所有权时才选 `std::shared_ptr`。共享所有权环需要通过弱引用等方式打破。[1] |
| 管理对象也必须被销毁 | RAII 不限于栈对象。成员对象和动态分配对象也可使用 RAII，但泄漏管理对象仍可能泄漏其资源。[4] |
| 析构不应向外抛出异常 | 析构函数应提供不抛出异常的清理；异常栈展开期间，析构再次向外抛出异常会触发 `std::terminate`。[1][5] |
| 自动清理有前提 | `std::abort`、`std::_Exit` 不执行对象析构；直接调用 `std::exit` 不析构自动存储期对象。未捕获异常导致终止时，也不能假定会完整栈展开。[5][6] |

RAII 为异常安全提供资源管理基础；一个操作能否满足“失败后状态不变”的**强异常保证**，还要检查该操作对外部状态的修改，不能仅凭使用 RAII 推断。

## 5. 知识网络

### 上位概念

- [[资源管理]]：RAII 是将清理责任绑定到对象生命周期的一种方法。
- [[对象生命周期]]：构造、析构及其执行规则是 RAII 的语言基础。

### 典型实现与支撑机制

- [[智能指针]]：RAII 在动态对象所有权管理中的典型实现；`std::unique_ptr` 支持移动而禁止复制。[4]
- [[作用域锁]]：`std::lock_guard` 将锁的持有期绑定到守卫对象的生命周期。[3]
- [[移动语义]]：支持所有权转交，使资源可以跨函数或对象传递。[4]
- [[栈展开]]：异常向处理器传播时，清理已成功构造的局部对象。[2]

### 相关概念与区别

- [[异常安全]]：RAII 支持可靠清理；是否维持其他不变量或回滚修改，需要额外设计。
- [[finally]]：通常在使用处安排清理；RAII 把清理封装到可复用的资源类型中，减少调用处重复代码。[7]
- [[作用域守卫]]：可通过析构执行预先登记的清理动作；中文维基条目以 RRID（Resource Release Is Destruction）讨论这一侧重点。[8]

## 6. 资料来源

以下链接均于 **2026-10-21** 查阅。中文维基百科用于概念导入；具体语义以 C++ 工作草案核对，工程建议参考 Core Guidelines。工作草案是持续更新的文本，不等同于已出版的 ISO 标准版本。

| 编号 | 来源 | 本笔记中的用途 |
| --- | --- | --- |
| 1 | [C++ Core Guidelines][1]：R.1、R.12、R.21、R.24、C.36–C.37 | RAII 原则、所有权选择、清理要求 |
| 2 | [C++ 工作草案：Stack unwinding][2]（`[except.ctor]`） | 异常清理顺序、构造失败时的子对象析构 |
| 3 | [C++ 工作草案：lock_guard][3]（`[thread.lock.guard]`） | 加锁／解锁及互斥量生命周期要求 |
| 4 | [C++ 工作草案：Unique-ownership pointers][4]（`[unique.ptr]`） | 独占所有权、空状态、移动与释放 |
| 5 | [C++ 工作草案：std::terminate][5]（`[except.terminate]`） | 析构抛出异常、未捕获异常的边界 |
| 6 | [C++ 工作草案：Startup and termination][6]（`[support.start.term]`） | 不执行常规析构的终止路径 |
| 7 | [Bjarne Stroustrup：Why doesn't C++ provide a “finally” construct?][7] | RAII 与显式清理代码的设计比较 |
| 8 | [中文维基百科：RAII][8] | 用户指定的入门材料与 RRID 术语背景 |

[1]: https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-raii
[2]: https://eel.is/c++draft/except.ctor
[3]: https://eel.is/c++draft/thread.lock.guard
[4]: https://eel.is/c++draft/unique.ptr
[5]: https://eel.is/c++draft/except.terminate
[6]: https://eel.is/c++draft/support.start.term
[7]: https://www.stroustrup.com/bs_faq2.html#finally
[8]: https://zh.wikipedia.org/wiki/RAII

---

*创建：2026-10-21*
