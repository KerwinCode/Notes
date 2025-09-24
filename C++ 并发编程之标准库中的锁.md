# C++ 并发编程之标准库中的锁

在C++并发编程中，锁是确保线程安全、防止数据竞争的核心工具。标准库（自C++11起）提供了一系列丰富的互斥体（Mutex）和锁管理工具，以应对不同的并发场景。本文将对这些锁进行详细的分析，并提供清晰的代码示例。

这些工具可以分为两大类：

1. **互斥体（Mutexes）**：实现锁定机制的基础构建块。
2. **锁管理器（Lock Managers）**：利用RAII（资源获取即初始化）原则，以更安全、更便捷的方式管理互斥体的生命周期。

## 第一部分：互斥体（Mutexes）

互斥体是基本的锁对象，用于保护共享数据，确保在任何时刻只有一个线程能够访问该数据。

### 1. `std::mutex`

* **特性**：最基础的互斥锁，提供独占式、非递归的所有权。 这意味着：
    * **独占式**：一次只能有一个线程锁定它。
    * **非递归**：一个已经获得锁的线程不能再次对同一个`std::mutex`进行加锁，否则将导致死锁。
* **适用场景**：保护简单的临界区，这是最常用的一种互斥体。
* **核心函数**：
    * `lock()`: 阻塞线程直到获得锁。
    * `try_lock()`: 尝试锁定，如果锁已被占用则立即返回`false`，不阻塞。
    * `unlock()`: 解锁。

#### 示例：保护共享计数器

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

std::mutex mtx; // 全局互斥锁
long long counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock(); // 手动加锁
        counter++;
        mtx.unlock(); // 手动解锁
    }
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(increment);
    }

    for (auto& th : threads) {
        th.join();
    }

    std::cout << "Counter: " << counter << std::endl; // 预期输出: 1000000
    return 0;
}
```

***注意***：手动调用`lock()`和`unlock()`是危险的，如果在加锁和解锁之间发生异常，锁将不会被释放，可能导致死锁。 因此，通常推荐使用锁管理器。

### 2. `std::recursive_mutex`

* **特性**：递归互斥锁。允许同一个线程多次锁定同一个互斥体，而不会产生死锁。 该线程必须调用与`lock()`次数相匹配的`unlock()`才能最终释放锁。
* **适用场景**：当一个持有锁的函数需要调用另一个（可能也是`public`的）函数，而后者也需要获取同一个锁时。这种情况在复杂的类设计中可能出现。
* **忠告**：**优先重构设计而非使用`std::recursive_mutex`**。 递归锁通常意味着设计上存在耦合问题。一个更好的设计模式是将共享锁的公共函数拆分为一个不加锁的私有实现函数和一个加锁的公共接口。

#### 示例：递归函数调用

```cpp
#include <iostream>
#include <thread>
#include <mutex>

class ComplexOperation {
public:
    void operationA() {
        std::lock_guard<std::recursive_mutex> lock(rec_mtx);
        std::cout << "Operation A acquired the lock." << std::endl;
        operationB(); // 调用另一个也需要锁的函数
    }

    void operationB() {
        std::lock_guard<std::recursive_mutex> lock(rec_mtx);
        std::cout << "Operation B acquired the lock again." << std::endl;
        // ... 执行操作
    }

private:
    std::recursive_mutex rec_mtx;
};

int main() {
    ComplexOperation op;
    std::thread t1(&ComplexOperation::operationA, &op);
    t1.join();
    return 0;
}
```

### 3. `std::timed_mutex`

* **特性**：带超时的互斥锁。除了`std::mutex`的功能外，还增加了两个带超时的锁定尝试函数。
    * `try_lock_for(duration)`: 在指定的时间段内尝试获取锁。
    * `try_lock_until(time_point)`: 尝试获取锁直到指定的时间点。
* **适用场景**：当线程不能无限期等待锁时，例如在需要响应用户操作或避免系统假死的场景中。

#### 示例：带超时的任务

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

std::timed_mutex timed_mtx;

void task() {
    using namespace std::chrono_literals;
    // 启动一个长时间持有锁的线程
    std::lock_guard<std::timed_mutex> lock(timed_mtx);
    std::cout << "Task is holding the lock for 5 seconds..." << std::endl;
    std::this_thread::sleep_for(5s);
}

void timed_task() {
    using namespace std::chrono_literals;
    std::cout << "Timed task trying to lock..." << std::endl;

    // 尝试在1秒内获取锁
    if (timed_mtx.try_lock_for(1s)) {
        std::cout << "Timed task acquired the lock." << std::endl;
        timed_mtx.unlock();
    } else {
        std::cout << "Timed task could not acquire the lock within 1 second." << std::endl;
    }
}

int main() {
    std::thread t1(task);
    std::this_thread::sleep_for(std::chrono::milliseconds(100)); // 确保t1先运行
    std::thread t2(timed_task);

    t1.join();
    t2.join();
    return 0;
}
```

### 4. `std::shared_mutex` (C++17)

* **特性**：读写锁。它允许多个线程同时进行“读”访问（共享锁），但“写”访问（独占锁）是互斥的。
    * **共享锁 (`lock_shared`)**：多个线程可以同时持有共享锁。
    * **独占锁 (`lock`)**：当一个线程持有独占锁时，其他任何线程（无论是读还是写）都不能获取任何锁。
* **适用场景**：读操作远多于写操作的“读多写少”场景，可以显著提高并发性能。 常见的应用包括缓存系统和数据库连接池。

#### 示例：读写分离

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <shared_mutex>
#include <vector>

class Telemetry {
public:
    int read() const {
        std::shared_lock<std::shared_mutex> lock(sh_mtx); // 读锁
        // 多个读者可以同时进入这里
        std::cout << "Reading value: " << value << std::endl;
        return value;
    }

    void write(int new_value) {
        std::unique_lock<std::shared_mutex> lock(sh_mtx); // 写锁
        // 只有一个写者可以进入这里
        std::cout << "Writing value..." << std::endl;
        value = new_value;
    }

private:
    mutable std::shared_mutex sh_mtx;
    int value = 0;
};
```

---

## 第二部分：锁管理器（Lock Managers / RAII Wrappers）

锁管理器是C++并发编程的最佳实践。它们利用RAII（资源获取即初始化）原则，在构造函数中获取锁，在析构函数中释放锁。 这可以确保即使在发生异常或提前返回的情况下，锁也能被正确释放，极大地增强了代码的健壮性。

### 1. `std::lock_guard`

* **特性**：最简单、轻量的锁管理器。 它在构造时锁定互斥体，并在作用域结束时自动解锁。 `std::lock_guard`是不可复制和移动的。
* **适用场景**：当你需要锁定一个互斥体并确保它在当前作用域结束时被释放的简单场景。
* **选择理由**：意图清晰，开销最小。如果你只需要简单的作用域锁定，这是首选。

#### 示例：`std::lock_guard` 的基本用法

```cpp
std::mutex m;
int shared_data = 0;

void safe_modify() {
    std::lock_guard<std::mutex> guard(m); // 构造时加锁
    shared_data++;
    // 当 guard 离开作用域时，其析构函数会自动调用 m.unlock()
}
```

### 2. `std::unique_lock`

* **特性**：功能最强大、最灵活的锁管理器。 它提供了`std::lock_guard`的所有功能，并增加了额外特性：
    * **延迟锁定**：可以在构造时不立即锁定 (`std::defer_lock`)。
    * **手动控制**：可以手动调用`lock()`, `try_lock()`, `unlock()`等方法。
    * **所有权转移**：`std::unique_lock`是可移动的，可以转移锁的所有权。
    * **与条件变量配合**：是与`std::condition_variable`一起使用的唯一选择。
* **适用场景**：
    * 与`std::condition_variable`一起等待条件。
    * 需要提前解锁以减小锁的粒度。
    * 需要转移锁的所有权。

#### 示例：与条件变量一起使用

```cpp
std::mutex cv_m;
std::condition_variable cv;
bool ready = false;

void worker_thread() {
    // 等待主线程准备数据
    std::unique_lock<std::mutex> lk(cv_m);
    cv.wait(lk, []{ return ready; }); // wait会自动解锁lk，被唤醒后会重新加锁

    std::cout << "Worker thread is processing data." << std::endl;
}
```

### 3. `std::scoped_lock` (C++17)

* **特性**：`std::lock_guard`的现代替代品，主要优势是能够同时锁定**多个**互斥体而不会产生死锁。 它内部使用了一种死锁避免算法。
* **适用场景**：
    * 需要原子地锁定两个或更多互斥体（例如，在两个账户之间转账）。
    * 在C++17及以后的代码中，作为`std::lock_guard`的通用替代品。
* **选择理由**：提供了比`std::lock_guard`更强的安全保证（防止多锁死锁），并且在只锁一个互斥体时同样高效。

#### 示例：避免死锁

```cpp
std::mutex m1, m2;

void transfer(int amount) {
    // C++17方式: 安全地同时锁定两个互斥体
    std::scoped_lock lock(m1, m2);
    // ... 执行转账操作 ...
}

// C++17之前的传统方式 (更繁琐)
void old_transfer(int amount) {
    std::lock(m1, m2); // 死锁避免算法
    std::lock_guard<std::mutex> lock1(m1, std::adopt_lock);
    std::lock_guard<std::mutex> lock2(m2, std::adopt_lock);
    // ... 执行转账操作 ...
}
```

### 4. `std::shared_lock` (C++14)

* **特性**：专门与`std::shared_mutex`配合使用的锁管理器，用于获取共享（读）锁。
* **适用场景**：实现读写锁模式中的“读”操作部分。

#### 示例：与`std::shared_mutex`配合

```cpp
#include <shared_mutex>
#include <iostream>

std::shared_mutex sh_mtx;
int shared_resource = 0;

void reader() {
    std::shared_lock<std::shared_mutex> lock(sh_mtx);
    // 多个reader线程可以同时执行到这里
    std::cout << "Reader reads: " << shared_resource << std::endl;
}

void writer() {
    std::unique_lock<std::shared_mutex> lock(sh_mtx); // 写操作仍需 unique_lock
    shared_resource++;
}
```

## 总结与选择指南

| 锁类型                     | 何时使用                                       | 优点                             | 缺点                                         |
| :------------------------- | :--------------------------------------------- | :------------------------------- | :------------------------------------------- |
| **`std::mutex`**           | 最常见的互斥需求，保护临界区。                 | 简单，高效。                     | 非递归，手动管理易出错。                     |
| **`std::recursive_mutex`** | 同一线程需要多次获取同一把锁（应尽量避免）。   | 允许递归锁定。                   | 开销比`std::mutex`大，通常暗示不良设计。     |
| **`std::timed_mutex`**     | 需要在有限时间内获取锁，避免无限等待。         | 防止线程永久阻塞。               | 略微增加开销。                               |
| **`std::shared_mutex`**    | “读多写少”的场景。                             | 允许多个读者并发，提高性能。     | 比`std::mutex`复杂，开销更大。               |
| **`std::lock_guard`**      | 简单地在作用域内锁定单个互斥体。               | 轻量，安全（RAII），意图明确。   | 功能有限，不能手动解锁或用于条件变量。       |
| **`std::unique_lock`**     | 需要高级锁操作：条件变量、延迟锁定、手动控制。 | 功能最强大、最灵活。             | 开销比`lock_guard`稍大，功能多也可能被误用。 |
| **`std::scoped_lock`**     | C++17及以后，需要锁定一个或多个互斥体。        | 可安全锁定多个互斥体，防止死锁。 | 仅限C++17及以后。                            |
| **`std::shared_lock`**     | 与`std::shared_mutex`一起使用，获取读锁。      | 专门用于实现读写锁模式。         | 必须与`std::shared_mutex`配合。              |

**通用建议**：

1. **始终使用锁管理器**：优先使用`std::scoped_lock` (C++17+) 或 `std::lock_guard`，而不是手动调用`lock()`/`unlock()`。
2. **选择最简单的工具**：如果`std::lock_guard`或`std::scoped_lock`能满足需求，就不要使用更复杂的`std::unique_lock`。
3. **小心死锁**：当需要锁定多个互斥体时，请使用`std::scoped_lock`（或`std::lock`），并始终保持一致的锁定顺序。
4. **谨慎使用递归锁**：在选择`std::recursive_mutex`之前，请三思你的设计。
