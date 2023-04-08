下面是一些常用的 C++线程同步机制:

1. 信号量：信号量是一种计数器，可以用来控制多个线程之间的访问权限。当一个线程需要访问共享资源时，可以先通过增加信号量的数量来申请访问权限，当访问完成后，再通过减少信号量的数量来释放访问权限。
2. 互斥量：互斥量是一种保护共享资源的同步机制，当一个线程需要访问共享资源时，需要先获取互斥量的保护，只有在获取到保护后，才能访问共享资源。当线程需要退出共享资源时，需要释放互斥量的保护。
3. 条件变量：条件变量是一种用于线程等待和唤醒的同步机制。当一个线程需要等待某个条件满足时，可以通过等待条件变量来阻塞自己;而当条件满足后，可以通过唤醒条件变量来通知其他线程。
4. 读写锁：读写锁是一种允许多个线程同时读取共享资源的同步机制，但只允许一个线程写入共享资源。当一个线程需要读取共享资源时，可以通过获取读写锁来获取访问权限;而当需要写入共享资源时，需要先释放读写锁，以便其他线程读取共享资源。

下面是一个简单的示例，展示如何使用 C++中的信号量、互斥量、条件变量和读写锁来实现线程同步:

```cpp
#include <iostream>  
#include <thread>  
#include <mutex>  
#include <condition_variable>  
#include <queue>

using namespace std;

// 用于线程同步的类  
class ThreadSync {  
public:  
    // 线程同步的入口函数  
    void start() {  
        // 创建一个互斥量，用于控制线程的访问权限  
        mutex_ = new mutex();  
          
        // 创建一个信号量，用于控制线程的并发访问  
        queue_ = new queue<int>();  
        count_ = 0;  
          
        // 创建一个条件变量，用于线程等待和唤醒  
        condition_ = new condition_variable();  
          
        // 创建一个线程，用于等待条件变量  
        thread_ = new thread([&]() {  
            while (true) {  
                // 等待条件变量  
                mutex_->lock();  
                condition_->wait(mutex_->mutex());  
                  
                // 增加计数器  
                count_++;  
                  
                // 释放互斥量和保护条件变量  
                mutex_->unlock();  
                condition_->notify_one();  
            }  
        });  
    }

    // 停止线程  
    void stop() {  
        // 释放互斥量和信号量  
        mutex_->unlock();  
        queue_->pop();  
        count_--;  
          
        // 唤醒线程  
        condition_->notify_all();  
    }

private:  
    // 互斥量  
    mutex* mutex_;  
      
    // 信号量  
    queue<int>* queue_;  
      
    // 条件变量  
    condition_variable* condition_;  
      
    // 线程  
    thread* thread_;  
      
    // 计数器  
    int count_;  
};

int main() {  
    ThreadSync sync;  
    sync.start();  
      
    while (sync.count_ > 0) {  
        sync.mutex_->lock();  
        sync.condition_->wait(sync.mutex_->mutex());  
          
        sync.queue_->push(sync.count_);  
        sync.count_--;  
          
        sync.mutex_->unlock();  
        sync.condition_->notify_one();  
    }  
      
    sync.stop();  
      
    return 0;  
}
```

上面的示例使用了互斥量、信号量、条件变量和读写锁来实现线程同步。在实际开发中，可以根据具体的业务场景选择合适的同步机制，以提高程序的并发性能和稳定性。