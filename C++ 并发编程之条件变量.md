# C++ 并发编程之条件变量

在C++并发编程中，`std::mutex` 和 `std::condition_variable` 是实现线程间同步与通信的基石。正确地结合使用它们，可以构建出高效、健壮的多线程应用程序。本文将通过实例，详细讲解如何快速写出正确的条件变量代码以防止线程竞争，并总结出关键的最佳实践。

## 核心概念：为何需要条件变量？

`std::mutex`（互斥锁）用于保护共享数据，确保同一时间只有一个线程能够访问该数据，从而避免数据竞争。然而，仅有互斥锁在某些场景下是不够的。例如，在一个经典的“生产者-消费者”模型中，消费者线程需要等待生产者线程准备好数据后才能进行处理。如果消费者线程不断地加锁、检查数据是否就绪、然后解锁，将会造成大量的CPU资源浪费。

`std::condition_variable`（条件变量）正是为了解决这类问题而生。它允许一个或多个线程等待某个特定条件成立。当条件不满足时，等待的线程会进入休眠状态，并自动释放其持有的互斥锁，从而避免了忙等待（busy-waiting）。当另一个线程（通常是生产者）改变了条件并发出通知后，等待的线程才会被唤醒，并重新获取互斥锁以继续执行。

## 关键问题与解决方案

在使用条件变量时，有两个核心问题必须正确处理，否则极易引入难以调试的并发Bug：

1. **虚假唤醒（Spurious Wakeups）**：等待中的线程有时可能会在没有收到任何通知的情况下被唤醒。这是操作系统调度器的底层实现细节所致，为了性能优化，操作系统允许这种情况发生。如果线程被唤醒后不检查它所等待的条件是否真的成立，就可能在错误的状态下继续执行，导致程序出错。

2. **丢失唤醒（Lost Wakeups）**：如果生产者线程在消费者线程调用`wait()`之前就发送了通知，那么这个通知就会丢失。当消费者之后再进入等待状态时，它将永远等不到那个已经错过的通知，从而可能导致死锁。

**解决方案**：始终在循环中结合**谓词（Predicate）**来使用`wait()`。谓词是一个返回布尔值的函数或lambda表达式，用于检查等待的条件是否为真。

`std::condition_variable::wait()`的重载版本`wait(lock, predicate)`内部实现了一个循环，它会自动处理虚假唤醒和丢失唤醒的问题。其工作流程如下：

1. 调用`wait`时，如果谓词返回`false`，则原子地释放锁并将线程置于等待状态。
2. 当线程被通知（或虚假唤醒）后，它会醒来并重新获取锁。
3. 然后，`wait`函数会再次检查谓词。如果返回`true`，函数返回，线程继续执行；如果返回`false`（即虚假唤醒），线程将继续等待。

## 实例：生产者-消费者模型

下面通过一个经典的生产者-消费者队列实例，来展示`mutex`和`condition_variable`的最佳实践。

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <chrono>

std::mutex mtx;                 // 全局互斥锁
std::condition_variable cv;     // 全局条件变量
std::queue<int> data_queue;     // 共享的队列
bool finished = false;          // 标志生产者是否完成生产

// 生产者线程函数
void producer(int n) {
    for (int i = 0; i < n; ++i) {
        std::this_thread::sleep_for(std::chrono::milliseconds(100)); // 模拟生产耗时
        {
            // 1. 修改共享数据前，加锁
            std::lock_guard<std::mutex> lock(mtx);
            
            // 2. 修改共享数据（生产数据）
            std::cout << "Producing data: " << i << std::endl;
            data_queue.push(i);
        } // lock_guard 在此作用域结束时自动解锁

        // 3. 通知一个等待的消费者线程
        // 可以在解锁后通知，以减少消费者线程被唤醒后立即因无法获取锁而阻塞的时间
        cv.notify_one(); 
    }

    // 生产结束后，设置标志位并通知所有消费者
    {
        std::lock_guard<std::mutex> lock(mtx);
        finished = true;
    }
    cv.notify_all(); // 唤醒所有可能在等待的消费者，告知生产已结束
}

// 消费者线程函数
void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx); // 使用 unique_lock，因为它能与条件变量协作

        // 4. 使用带谓词的 wait，防止虚假唤醒和丢失唤醒
        //    等待条件：队列不为空 或 生产已结束
        cv.wait(lock, [] { return !data_queue.empty() || finished; });

        // 5. wait返回后，线程持有锁，并再次检查条件
        //    如果队列非空，则消费
        if (!data_queue.empty()) {
            int data = data_queue.front();
            data_queue.pop();
            std::cout << "Consuming data: " << data << std::endl;
        } else if (finished) {
            // 如果队列为空且生产结束，则退出循环
            std::cout << "Consumer finished." << std::endl;
            break;
        }
    }
}

int main() {
    std::thread prod(producer, 10);
    std::thread cons1(consumer);
    std::thread cons2(consumer);

    prod.join();
    cons1.join();
    cons2.join();

    return 0;
}
```

代码剖析：

* **`std::mutex mtx`**：用于保护对共享资源`data_queue`和`finished`的访问。任何对这两个变量的读写操作都必须在持有`mtx`锁的情况下进行。
* **`std::condition_variable cv`**：用于协调生产者和消费者线程。消费者在队列为空时在此条件变量上等待。
* **`std::lock_guard` vs `std::unique_lock`**：
    * 在生产者中，`std::lock_guard`足以满足需求，它在构造时加锁，在析构时自动解锁，简单方便。
    * 在消费者中，必须使用`std::unique_lock`。因为`cv.wait()`在使线程休眠前，需要先解锁互斥体，并在被唤醒后重新加锁，而`std::unique_lock`提供了这种灵活的锁管理能力。
* **生产者逻辑**：
    1. 在循环中生产数据。
    2. 每次生产数据时，先通过`std::lock_guard`获取锁，然后将数据放入队列。
    3. 放完数据后，调用`cv.notify_one()`唤醒一个可能在等待的消费者线程。**一个重要的优化**是，可以在解锁后进行通知，这可以避免被唤醒的消费者线程立即尝试获取一个尚未被生产者释放的锁，从而减少了线程上下文切换的开销。
    4. 生产全部结束后，设置`finished`标志并调用`cv.notify_all()`，确保所有可能因队列为空而等待的消费者都能被唤醒并最终退出。
* **消费者逻辑**：
    1. 在`while(true)`循环中准备消费。
    2. 使用`std::unique_lock`锁定互斥体。
    3. 调用`cv.wait(lock, predicate)`。这里的谓词是`[] { return !data_queue.empty() || finished; }`。这意味着，只有当“队列不为空”或“生产已结束”这两个条件之一满足时，`wait`才会返回。这完美地处理了虚假唤醒和丢失唤醒的问题。
    4. 当`wait`返回时，消费者线程已经重新获得了锁。此时，它需要再次检查条件（特别是当有多个消费者时），因为在它被唤醒和实际获得锁之间，其他消费者可能已经处理了数据。

## 总结：最佳实践清单

为了快速、正确地使用`mutex`和`condition_variable`，请遵循以下核心准则：

1. **始终与互斥锁配对使用**：条件变量必须与互斥锁一起使用，以保护被检查的条件（共享状态）。
2. **使用`std::unique_lock`进行等待**：调用`wait`, `wait_for`, 或 `wait_until`的线程必须使用`std::unique_lock`，因为它提供了`wait`函数内部所需的解锁和重新加锁的灵活性。
3. **在循环和谓词中等待**：这是最重要的一条规则。始终使用带有谓词的`wait`版本（`cv.wait(lock, []{ return condition; });`），或者手动在`while`循环中调用无谓词的`wait`（`while (!condition) { cv.wait(lock); }`）。这能从根本上防止虚假唤醒和丢失唤醒导致的问题。
4. **修改条件前必须加锁**：任何可能改变条件变量所等待的条件（即谓词状态）的线程，在进行修改操作时必须持有与等待线程相同的互斥锁。
5. **谨慎选择 `notify_one()` 和 `notify_all()`**：
    * 当你确定只有一个等待的线程需要被唤醒，且哪个线程被唤醒都无所谓时，使用`notify_one()`，它的效率更高。
    * 当你改变的条件可能满足多个等待线程的需求，或者当有多个消费者在等待，并且你想结束所有线程时（如本例中的`finished`标志），使用`notify_all()`。
6. **考虑在解锁后通知**：在修改完共享状态并释放锁之后再调用`notify_one()`或`notify_all()`，这是一种性能优化，可以减少被唤醒线程不必要的等待和上下文切换。

遵循以上实践，你将能够更加准确地编写出无线程竞争的条件变量代码，从而构建稳定高效的并发系统。
