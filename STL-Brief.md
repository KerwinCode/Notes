STL（Standard Template Library）是 C++ 中的标准模板库，提供了各种容器、算法和迭代器等功能，能够极大地提高 C++ 程序的编写效率和代码质量。

## 常用容器特点

1. vector：动态数组，支持快速随机访问，尾部插入和删除元素效率高，但在中间插入或删除元素效率较低。
2. deque：双端队列，支持快速随机访问，两端插入和删除元素效率高，但在中间插入或删除元素效率较低。
3. list：双向链表，不支持随机访问，但在任意位置插入和删除元素效率都很高。
4. forward_list：单向链表，不支持随机访问，但在任意位置插入和删除元素效率都很高，同时相较于 list 更加轻量级。
5. set：集合，可以自动去重，元素按照一定的顺序排列，支持快速查找、插入和删除元素。
6. multiset：允许元素重复的集合，元素按照一定的顺序排列，支持快速查找、插入和删除元素。
7. map：关联数组，可以自动按照 key 值排序，支持快速查找、插入和删除元素。
8. multimap：允许 key 值重复的关联数组，元素按照一定的顺序排列，支持快速查找、插入和删除元素。
9. unordered_set：无序集合，元素不按照任何顺序排列，支持快速查找、插入和删除元素。
10. unordered_multiset：允许元素重复的无序集合，元素不按照任何顺序排列，支持快速查找、插入和删除元素。
11. unordered_map：无序关联数组，元素不按照任何顺序排列，支持快速查找、插入和删除元素。
12. unordered_multimap：允许 key 值重复的无序关联数组，元素不按照任何顺序排列，支持快速查找、插入和删除元素。

以上这些容器提供了不同的特性和性能优劣，使用时需要根据实际情况选择合适的容器。

## 底层实现

1. vector：底层实现为连续的内存空间，可以通过下标随机访问元素。当元素数量超过当前空间时，会重新分配更大的空间，然后将原有元素复制到新的空间中。这会导致数据复制的开销，但由于空间的连续性，可以获得很好的缓存性能。
2. deque：底层实现为一块中央控制区域和多个缓冲区，中央控制区域维护了多个缓冲区的地址。deque 支持在头尾插入和删除元素，但不支持随机访问元素，因为元素可能存储在不同的缓冲区中。
3. list：底层实现为双向链表，支持在任意位置插入和删除元素。由于元素不是连续存储的，不能通过下标随机访问元素，但是支持高效的元素插入和删除操作。
4. forward_list：底层实现为单向链表，支持在任意位置插入和删除元素。与 list 相比，由于每个节点只有一个后继指针，forward_list 占用更少的空间，但无法进行逆向遍历。
5. set 和 map：底层实现为红黑树，可以高效地完成元素的查找、插入和删除操作。set 中的元素是唯一的，而 map 中的元素是键值对，键是唯一的。
6. multiset 和 multimap：底层实现与 set 和 map 相同，但允许存储重复的元素或键值对。
7. unordered_set 和 unordered_map：底层实现为哈希表，可以高效地完成元素的查找、插入和删除操作。unordered_set 中的元素是唯一的，而 unordered_map 中的键是唯一的。
8. unordered_multiset 和 unordered_multimap：底层实现与 unordered_set 和 unordered_map 相同，但允许存储重复的元素或键值对。
9. stack 和 queue：底层实现可以使用任何一个容器，但通常使用 deque 实现。stack 支持在顶部插入和删除元素，queue 支持在队列的尾部插入元素，在队列的头部删除元素。
10. priority_queue：底层实现可以使用任何一个容器，但通常使用 vector 实现。priority_queue 是一个堆，支持在堆顶插入元素和取出堆顶元素。

下面介绍一些 STL 中常见的容器及其使用方法：

## vector

vector 是一个动态数组，支持随机访问和尾部插入删除等操作，常用的操作有：

- 插入元素：`push_back(element)`，在 vector 的尾部插入一个元素。
- 访问元素：`vector[index]`，返回 vector 中索引为 index 的元素。
- 删除元素：`pop_back()`，删除 vector 的尾部元素。
- 获取大小：`size()`，返回 vector 中元素的数量。

示例代码：

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;
    vec.push_back(1);
    vec.push_back(2);
    vec.push_back(3);

    for (int i = 0; i < vec.size(); i++) {
        std::cout << vec[i] << " ";
    }
    // Output: 1 2 3

    vec.pop_back();
    std::cout << std::endl;
    for (int i = 0; i < vec.size(); i++) {
        std::cout << vec[i] << " ";
    }
    // Output: 1 2

    return 0;
}
```

## list

list 是一个双向链表，支持在头部和尾部插入删除等操作，常用的操作有：

- 插入元素：`push_front(element)` 和 `push_back(element)`，在 list 的头部或尾部插入一个元素。
- 访问元素：由于 list 不支持随机访问，只能通过迭代器访问元素，例如 `std::list<int>::iterator it`。
- 删除元素：`pop_front()` 和 `pop_back()`，删除 list 的头部或尾部元素。
- 获取大小：`size()`，返回 list 中元素的数量。

示例代码：

```cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> lst;
    lst.push_back(1);
    lst.push_back(2);
    lst.push_front(0);

    for (auto it = lst.begin(); it != lst.end(); it++) {
        std::cout << *it << " ";
    }
    // Output: 0 1 2

    lst.pop_front();
    std::cout << std::endl;
    for (auto it = lst.begin(); it != lst.end(); it++) {
        std::cout << *it << " ";
    }
    // Output: 1 2

    return 0;
}
```

## deque

deque 是一个双端队列，支持在头部和尾部插入删除等操作，也支持随机访问，常用的操作有：

- 插入元素：`push_front(element)` 和 `push_back(element)`，在 deque 的头部或尾部插入一个元素。
- 访问元素：`deque[index]`，返回 deque 中索引为 index 的元素。
- 删除元素：`pop_front()` 和 `pop_back()`，删除 deque 的头部或尾部元素。
- 获取大小：`size()`，返回 deque 中元素的数量。

示例代码：

```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<int> deq;
    deq.push_back(1);
    deq.push_back(2);
    deq.push_front(0);

    for (int i = 0; i < deq.size(); i++) {
        std::cout << deq[i] << " ";
    }
    // Output: 0 1 2

    deq.pop_front();
    std::cout << std::endl;
    for (int i = 0; i < deq.size(); i++) {
        std::cout << deq[i] << " ";
    }
    // Output: 1 2

    return 0;
}
```

## set

set 是一个集合，其中的元素是不重复的，支持插入、删除、查找等操作，常用的操作有：

- 插入元素：`insert(element)`，将 element 插入 set 中。
- 删除元素：`erase(element)`，将 set 中的 element 删除。
- 查找元素：`find(element)`，返回指向 set 中元素为 element 的迭代器，如果不存在则返回 `set::end()`。
- 获取大小：`size()`，返回 set 中元素的数量。

示例代码：

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> st;
    st.insert(1);
    st.insert(2);
    st.insert(3);

    auto it = st.find(2);
    if (it != st.end()) {
        std::cout << "Element found in set: " << *it << std::endl;
    } else {
        std::cout << "Element not found in set" << std::endl;
    }
    // Output: Element found in set: 2

    st.erase(3);
    std::cout << "Set size: " << st.size() << std::endl;
    // Output: Set size: 2

    return 0;
}
```

## map

map 是一个映射表，支持 key-value 的存储方式，其中 key 是不重复的，常用的操作有：

- 插入元素：`insert(std::make_pair(key, value))`，将 key-value 插入 map 中。
- 删除元素：`erase(key)`，将 map 中的 key 删除。
- 查找元素：`find(key)`，返回指向 map 中 key 的迭代器，如果不存在则返回 `map::end()`。
- 获取大小：`size()`，返回 map 中元素的数量。

示例代码：

```cpp
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> mp;
    mp.insert(std::make_pair("apple", 1));
    mp.insert(std::make_pair("banana", 2));
    mp.insert(std::make_pair("orange", 3));

    auto it = mp.find("banana");
    if (it != mp.end()) {
        std::cout << "Value of key 'banana': " << it->second << std::endl;
    } else {
        std::cout << "Key 'banana' not found in map" << std::endl;
    }
    // Output: Value of key 'banana': 2
}
```

## unordered_set

unordered_set 是一个无序集合，其中的元素是不重复的，支持插入、删除、查找等操作，常用的操作与 set 类似，但 unordered_set 采用的是哈希表的方式实现，查找操作的时间复杂度为常数级别，而 set 则是红黑树实现，时间复杂度为对数级别。

示例代码：

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> st;
    st.insert(1);
    st.insert(2);
    st.insert(3);

    auto it = st.find(2);
    if (it != st.end()) {
        std::cout << "Element found in unordered_set: " << *it << std::endl;
    } else {
        std::cout << "Element not found in unordered_set" << std::endl;
    }
    // Output: Element found in unordered_set: 2

    st.erase(3);
    std::cout << "unordered_set size: " << st.size() << std::endl;
    // Output: unordered_set size: 2

    return 0;
}
```

## unordered_map

unordered_map 是一个无序的映射表，支持 key-value 的存储方式，常用的操作与 map 类似，但 unordered_map 采用的是哈希表的方式实现，查找操作的时间复杂度为常数级别，而 map 则是红黑树实现，时间复杂度为对数级别。

示例代码：

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> mp;
    mp.insert(std::make_pair("apple", 1));
    mp.insert(std::make_pair("banana", 2));
    mp.insert(std::make_pair("orange", 3));

    auto it = mp.find("banana");
    if (it != mp.end()) {
        std::cout << "Value of key 'banana': " << it->second << std::endl;
    } else {
        std::cout << "Key 'banana' not found in unordered_map" << std::endl;
    }
    // Output: Value of key 'banana': 2

    return 0;
}
```

## forward_list

forward_list 是 C++11 中新增的容器，是一个单向链表，每个节点只包含了其后继节点的地址，因此只能向后遍历，不能像 vector 和 deque 一样通过下标随机访问元素。

示例代码：

```cpp
#include <iostream>
#include <forward_list>

int main() {
    std::forward_list<int> flist;

    // 插入元素
    flist.push_front(1);
    flist.push_front(2);
    flist.push_front(3);

    // 遍历元素
    for (auto it = flist.begin(); it != flist.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;
    // Output: 3 2 1

    // 在特定位置插入元素
    flist.insert_after(flist.begin(), 4);
    flist.insert_after(flist.begin(), {5, 6});

    // 删除元素
    flist.pop_front();
    flist.remove(5);

    // 输出元素
    for (auto n : flist) {
        std::cout << n << " ";
    }
    std::cout << std::endl;
    // Output: 4 6 2 1

    return 0;
}
```

需要注意的是，forward_list 不支持逆向遍历，因此不能使用反向迭代器 rbegin 和 rend。另外，由于 forward_list 是单向链表，因此不能像 list 一样通过迭代器进行移动，只能从头开始遍历或在指定位置插入或删除元素。

## Cheat Sheet

```
vector, 变长数组，倍增的思想
    size()  返回元素个数
    empty()  返回是否为空
    clear()  清空
    front()/back()
    push_back()/pop_back()
    begin()/end()
    []
    支持比较运算，按字典序

pair<int, int>
    first, 第一个元素
    second, 第二个元素
    支持比较运算，以first为第一关键字，以second为第二关键字（字典序）

string，字符串
    size()/length()  返回字符串长度
    empty()
    clear()
    substr(起始下标，(子串长度))  返回子串
    c_str()  返回字符串所在字符数组的起始地址

queue, 队列
    size()
    empty()
    push()  向队尾插入一个元素
    front()  返回队头元素
    back()  返回队尾元素
    pop()  弹出队头元素

priority_queue, 优先队列，默认是大根堆
    size()
    empty()
    push()  插入一个元素
    top()  返回堆顶元素
    pop()  弹出堆顶元素
    定义成小根堆的方式：priority_queue<int, vector<int>, greater<int>> q;

stack, 栈
    size()
    empty()
    push()  向栈顶插入一个元素
    top()  返回栈顶元素
    pop()  弹出栈顶元素

deque, 双端队列
    size()
    empty()
    clear()
    front()/back()
    push_back()/pop_back()
    push_front()/pop_front()
    begin()/end()
    []

set, map, multiset, multimap, 基于平衡二叉树（红黑树），动态维护有序序列
    size()
    empty()
    clear()
    begin()/end()
    ++, -- 返回前驱和后继，时间复杂度 O(logn)

    set/multiset
        insert()  插入一个数
        find()  查找一个数
        count()  返回某一个数的个数
        erase()
            (1) 输入是一个数x，删除所有x   O(k + logn)
            (2) 输入一个迭代器，删除这个迭代器
        lower_bound()/upper_bound()
            lower_bound(x)  返回大于等于x的最小的数的迭代器
            upper_bound(x)  返回大于x的最小的数的迭代器
    map/multimap
        insert()  插入的数是一个pair
        erase()  输入的参数是pair或者迭代器
        find()
        []  注意multimap不支持此操作。 时间复杂度是 O(logn)
        lower_bound()/upper_bound()

unordered_set, unordered_map, unordered_multiset, unordered_multimap, 哈希表
    和上面类似，增删改查的时间复杂度是 O(1)
    不支持 lower_bound()/upper_bound()， 迭代器的++，--

bitset, 圧位
    bitset<10000> s;
    ~, &, |, ^
    >>, <<
    ==, !=
    []

    count()  返回有多少个1

    any()  判断是否至少有一个1
    none()  判断是否全为0

    set()  把所有位置成1
    set(k, v)  将第k位变成v
    reset()  把所有位变成0
    flip()  等价于~
    flip(k) 把第k位取反

```