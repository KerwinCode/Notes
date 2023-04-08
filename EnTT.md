EnTT 是一个仅面向头文件的，轻量且易于使用的游戏编程库，是基于 C++17 标准的 ECS 架构的实现，旨在为游戏和交互式应用程序提供高效和可扩展性的技术解决方案。Minecraft、ArcGIS Runtime SDK、Ragdoll等都使用了EnTT。

后面各节简要说明了如何使用实体组件系统（整个库的核心部分）。

- 实体（ECS 的 E）是用户应按原样使用的不透明标识符。不建议检查标识符，因为它的格式将来可能会更改，而且注册表具有所有的函数，可以即时查询这些标识符。`entt::entity` 类型实现实体标识符的概念。
- 组件（ECS 的 C）必须是可移动构造 (move constructible) 的和可移动赋值 (move assignable)。通过使用提供的参数来构造组件本身初始化它们。不需要向注册表或实体组件系统注册组件或它们的类型。
- 系统（ECS 的 S）可以是普通函数，函数对象，lambda 等。在任何情况下都不要求也不需要暴露它们。

代码示例：

```c++
#include <entt/entt.hpp>
#include <cstdint>

struct position {
    float x;
    float y;
};

struct velocity {
    float dx;
    float dy;
};

void update(entt::registry &registry) {
    auto view = registry.view<position, velocity>();

    for(auto entity: view) {
        // 只获取将要使用的组件 ...

        auto &vel = view.get<velocity>(entity);

        vel.dx = 0.;
        vel.dy = 0.;

        // ...
    }
}

void update(std::uint64_t dt, entt::registry &registry) {
    registry.view<position, velocity>().each([dt](auto &pos, auto &vel) {
        // 一次获取视图的所有组件 ...

        pos.x += vel.dx * dt;
        pos.y += vel.dy * dt;

        // ...
    });
}

int main() {
    entt::registry registry;
    std::uint64_t dt = 16;

    for(auto i = 0; i < 10; ++i) {
        auto entity = registry.create();
        registry.emplace<position>(entity, i * 1.f, i * 1.f);
        if(i % 2 == 0) { registry.emplace<velocity>(entity, i * .1f, i * .1f); }
    }

    update(dt, registry);
    update(registry);

    // ...
}
```

# 1. 注册表 (registry)

registry 可以存储和管理实体，也可以创建 views 和 groups 以迭代底层 (underlying) 数据结构。

## create() & destroy()

```c++
// 构造一个没有组件的裸实体并返回其标识符
auto entity = registry.create();

// 摧毁一个实体及其所有组件
registry.destroy(entity);
```

## view

```c++
// 摧毁范围内的所有实体
auto view = registry.view<a_component, another_component>();
registry.destroy(view.begin(), view.end());
```

## valid & version & current

```c++
// 如果实体仍然有效，则返回 true，否则返回 false
bool b = registry.valid(entity);

// 获取包含在实体标识符中的版本
auto version = registry.version(entity);

// 获取给定实体的实际版本
auto curr = registry.current(entity);
```

## emplace

```c++
// 创建、初始化给定组件并将其分配给实体。如果组件存在，emplace 接受可变数量的参数来构造组件本身
registry.emplace<position>(entity, 0., 0.);

// ...

auto &vel = registry.emplace<velocity>(entity);
vel.dx = 0.;
vel.dy = 0.;
```

## insert

当类型被指定为模板参数或实例被传递为参数时，立即为所有实体分配相同的组件

```c++
// 将默认初始化组件通过复制分配给所有实体
registry.insert<position>(first, last);

// 将用户定义初始化组件实例通过复制分配给所有实体
registry.insert(from, to, position{0., 0.});
// first 和 last 指定实体的范围，instances 指向组件范围中的第一个元素
registry.insert<position>(first, last, instances);
```

## patch & replace

```c++
// 替换原来的组件
registry.patch<position>(entity, [](auto &pos) { pos.x = pos.y = 0.; });

// 使用参数列表中的参数构造一个新组件实例并替换原来的组件
registry.replace<position>(entity, 0., 0.);
```

## emplace_or_replace

```c++
// 如过不知道实体是否已经拥有该组件的实例，可以使用该函数
registry.emplace_or_replace<position>(entity, 0., 0.);
```

## has & any

```c++
// 如果实体具有给定的所有组件，则为真
bool all = registry.has<position, velocity>(entity);

// 如果实体至少有给定组件中的一个，则为真
bool any = registry.any<position, velocity>(entity);
if(registry.has<velocity>(entity)) {
    registry.replace<velocity>(entity, 0., 0.);
} else {
    registry.emplace<velocity>(entity, 0., 0.);
}
```

## remove

```c++
// 从实体中删除组件
registry.remove<position>(entity);
```

## remove_if_exists

```c++
// 实体有该组件就删除，否则安全返回
registry.remove_if_exists<position>(entity);
```

## clear

```c++
// 删除拥有给定组件的实体
registry.clear<position>();

// 立即销毁 registry 中的所有实体:
registry.clear();
```

## get

可以通过以下方式获得对组件的引用

```c++
const auto &cregistry = registry;

// 常量和非常量引用
const auto &crenderable = cregistry.get<renderable>(entity);
auto &renderable = registry.get<renderable>(entity);

// 常量和非常量引用
const auto [cpos, cvel] = cregistry.get<position, velocity>(entity);
auto [pos, vel] = registry.get<position, velocity>(entity);
```

Get 提供对存储在 registry 中实体组件的直接访问。还有一个名为 try_get 的成员函数，在指定组件存在时，返回一个指向实体所拥有组件的指针，不存在则返回一个空指针。

## on_construct & on_destroy & on_destroy

创建对组件的侦听器

```c++
// 连接一个非成员函数
registry.on_construct<position>().connect<&my_free_function>();

// 连接一个成员函数
registry.on_construct<position>().connect<&my_class::member>(instance);

// 断开一个非成员函数
registry.on_construct<position>().disconnect<&my_free_function>();

// 断开一个成员函数
registry.on_construct<position>().disconnect<&my_class::member>(instance);
```

- on_construct：在创建分配组件时得到通知。(在组件被分配给实体之后被调用)
- on_destroy：在销毁组件时得到通知。(在从实体中删除组件之前被调用)
- on_update：当组件的实例被用户显式替换时得到通知。(只会在调用 replace 或 patch 之后被调用。)

侦听器函数的类型

```c++
void(entt::registry &, entt::entity);
```

一些使用限制：

- 在**所有**侦听器主体中都应避免 Connecting 或 disconnecting 另一个函数。这在某些情况下可能导致未定义行为。
- 在**创建**和**更新**侦听器的主体中不允许删除组件，该侦听器观察给定类型实例的构造或更新。
- 在**销毁**侦听器的主体中应该避免分配和删除组件，该侦听器观察给定类型的实例的销毁情况。这在某些情况下可能导致未定义行为。这种类型的侦听器只是为用户提供一个简单的执行清理的方法。

事件和侦听器不能用作系统的替代品。它们不应该包含太多的逻辑，与注册表的交互应该保持在最低限度。此外，监听器的数量越多，在创建或销毁组件时性能受到的影响就越大。

更多相关部分参考 signal 的文档

# 2. 响应式系统 (Reactive System)

## observer

```c++
entt::observer observer{registry, entt::collector.update<sprite>()};
```

这个类默认是可构造的，并且可以在任何时候通过 connect 成员函数重新配置。此外，可以通过 disconnect 成员函数将实例与 registry 断开连接。

观察者还提供了查询内部状态所需的信息，并知道它是否为空或者包含多少实体。此外，它还可以返回一个指向它所包含的实体列表的原始指针。

```c++
for(const auto entity: observer) {
    // ...
}

observer.clear();
```

下面代码和上面的等效

```c++
observer.each([](const auto entity) {
    // ...
});
```

## collector

collector 是一个实用程序，旨在生成一个匹配器列表 (实际规则) ，以便与观察者一起使用。

Observing 匹配器：返回所有还活着的实体，且其给定组件要是已更新但未销毁的

```c++
entt::collector.update<sprite>()
```

在这种情况下，update 意味着调用了附加到 on_update 上的所有监听器。为了实现这一点，必须使用诸如 patch 等特定函数。有关详细信息，请参阅特定的文档。

Grouping 匹配器：返回所有还活着的实体，且其属于给定组并且没有被销毁

```c++
entt::collector.group<position, velocity>(entt::exclude<destroyed>);
```

分组匹配器还支持排除列表以及单个组件。

粗略地说，观察匹配器拦截更新了给定组件的实体，而分组匹配器跟踪自上次请求以来已经分配了给定组件的实体。

## where

使用 where 子句进行筛选

```c++
entt::collector.update().where(entt::exclude);
```

在上面的例子中，当一个实体的精灵组件被更新时，观察者就会对实体进行检测，验证它至少要有位置组件，并且没有速度组件。如果过滤器的两个条件中的一个没有得到遵守，那么这实体就会被筛选掉。

## registry.sort

组件可以直接排序：

```c++
registry.sort<renderable>([](const auto &lhs, const auto &rhs) {
    return lhs.z < rhs.z;
});
```

也可以访问他们的实体:

```c++
registry.sort<renderable>([](const entt::entity lhs, const entt::entity rhs) {
    return entt::registry::entity(lhs) < entt::registry::entity(rhs);
});
```

根据另一个组件的顺序对组件进行排序:

```c++
registry.sort<movement, physics>();
```

这种情况下，会将 movement 实例安排 (arranged) 在内存中，以便在两个组件一起迭代时，将缓存未命中率降至最低。

附带说明，组的使用限制了对组件池进行排序的可能性。有关更多详细信息，请参阅特定的文档。

# 3. 辅助工具

## entt::null

```c++
registry.valid(entt::null); // 返回 false
```

注册表在所有情况下都拒绝空实体，因为它不被认为是有效的。这也意味着空实体不能拥有组件。

空实体的类型是内部的，除了用来定义空实体本身外，不应该用于任何其他目的。但是，存在从空实体隐式转换到允许的类型标识符：

```c++
entt::entity null = entt::null;
```

类似地，可以将空实体与任何其他标识符进行比较：

```c++
const auto entity = registry.create();
const bool null = (entity == entt::null);
```

请注意，`entt::null` 和为 0 的实体不是一回事。同样，初始化为 0 的实体与 `entt::null` 也是不同的。`entt::null{}` 在某种意义上是 0 实体的别名，且它们都不能用于创建空实体。

## entt::to_entity

它接受 registry 和组件实例，并返回与给定组件相关联的实体：

```c++
const auto entity = entt::to_entity(registry, position);
```

此实用程序不执行任何对组件有效性的检查。因此，尝试获取无效元素或与给定 registry 不关联组件实例的实体可能会导致未定义的行为。

## 依赖关系

当给实体分配 my_type 组件时，添加或替换 a_type 组件：

```c++
registry.on_construct<my_type>().connect<&entt::registry::emplace_or_replace<a_type>>();
```

当给实体分配 my_type 组件时，从实体中删除 a_type 组件：

```c++
registry.on_construct<my_type>().connect<&entt::registry::remove<a_type>>();
```

销毁依赖关系

```c++
registry.on_construct<my_type>().disconnect<&entt::registry::<a_type>>();
```

还有许多其他类型的依赖关系。大多数将实体作为第一个参数的函数，都可以很好的用于此目的。

## 调用 (invoke)

有时能够直接调用组件的成员函数作为回调很有用。在实践中已经可以实现，但是需要用户扩展其类，而这并非总是可能的。

### entt::invoke

```c++
registry.on_construct<clazz>().connect<entt::invoke<&clazz::func>>();
```

它所做的就是为接收到的实体选择合适的组件并调用请求的方法，并在必要时传递参数。

## Handle

句柄是围绕实体和注册表的薄包装。它提供了注册表提供的与组件一起使用的函数，例如 Emplace，Get，Patch，Remove 等。区别在于实体是隐式传递给注册表的。

它默认构造为包含空注册表和空实体的无效句柄。当它包含一个空注册表时，调用执行委托给注册表的函数会导致未定义的行为，因此建议在不确定时使用隐式强制转换为 bool 来检查句柄的有效性。

有两个使用 `entt::entity` 作为其默认实体的别名：`entt::handle` 和 `entt::const_handle`。用户还可以轻松地为自定义标识符创建自己的别名，如下所示：

```c++
using my_handle = entt::basic_handle<my_identifier>;
using my_const_handle = entt::basic_handle<const my_identifier>;
```

句柄可以直接转换为 const 句柄，但不能反过来。且句柄可以移动和复制。

句柄存储指向注册表的非常量指针，因此它可以完成非常量注册表可以完成的所有操作。另一方面，const 处理存储指向注册表的 const 指针，并提供一组受限制的函数。

句柄主要用于简化函数签名。如果函数需要一个注册表和一个实体，并在该实体上完成其大部分工作，则用户可以考虑使用句柄。

## 上下文变量 (Context variables)

将上下文变量分配给注册表通常很方便。

```c++
// 创建一个使用给定值初始化的新上下文变量
registry.set<my_type>(42, 'c');

// 获取上下文变量
const auto &var = registry.ctx<my_type>();

// 如果有疑问，检查注册表以避免断言出现错误
if(auto *ptr = registry.try_ctx<my_type>(); ptr) {
   // 使用与注册表关联的上下文变量（如果有）
}

// 取消上下文变量
registry.unset<my_type>();
```

上下文变量的类型必须是默认可构造且可以移动的。

### set

根据给定类型创建上下文变量。(要么创建上下文变量的新实例，要么覆盖已经存在的实例)

### ctx & try_ctx

均可用于检索新创建的实例

try_ctx 成员函数返回指向上下文变量的指针，不存在则返回空指针。

### unset

unset 则用于在需要时清除变量

## 组织者 (Organizer)

组织者类模板提供最小的支持（但在许多情况下足够），以根据函数及其对资源的要求创建执行图 (execution graph)。

产生的任务在任何情况下都不会执行。这不是这个工具的目标。相反，它们以图形的形式返回给用户，从而允许安全执行。

这些函数按执行顺序添加到组织者。非成员函数和成员函数被提升为模板参数，但是也有可能将指向非成员函数的指针或 lambdas 作为参数传递给 emplace 成员函数：

```c++
entt::organizer organizer;

// 向组织者添加非成员函数
organizer.emplace<&free_function>();

// 添加一个成员函数和一个实例，以便在其上向管理器调用它
clazz instance;
organizer.emplace<&clazz::member_function>(&instance);

// 直接添加一个 lambda
organizer.emplace(+[](const void *, entt::registry &) { /* ... */ });``` 
```

对注册表的可能常量引用。在运行时传递给任务的值也将按原样传递给函数。

带有任何可能的类型组合的 `entt::view`。将从传递给任务的注册表中创建，并直接提供给函数。

对任意类型 T 的一个可能的常量引用。将被解释为上下文变量，在注册表中创建并传递给函数。

作为参数传递给 emplace 的非成员函数和 lambda 函数的函数类型为 `void(const void*，entt::registry＆)`。注册表与提供给任务的注册表相同。第一个参数是指向用户定义数据的可选指针，以在注册时提供：

```c++
clazz instance;
organizer.emplace(+[](const void *, entt::registry &) { /* ... */ }, &instance);``` 
```

在所有情况下，创建时都可以将名称与任务相关联。例如：

```c++
organizer.emplace<&free_function>("func");
```

同样，用户可以通过模板再次覆盖类型的访问模式

```c++
organizer.emplace<&free_function, const renderable>("func");
```

在这种情况下，即使 renderable 的在函数的参数中显示为非常量，它在任务图的生成中也将被视为常量。

为了生成任务图，组织者提供了图成员函数：

```c++
std::vector<entt::organizer::vertex> graph = organizer.graph();
```

图是以邻接表的形式返回的。每个顶点具有以下特征:

- ro_count、rw_count：返回以只读或读写模式访问的资源数。
- ro_dependency、rw_dependency：对于检索与 underlying 函数参数相关联的类型信息对象非常有用。
- top_level：节点是否为顶层节点，即没有进入边 (entering edges)。
- info：返回与 underlying 函数关联的类型信息对象。
- name：返回与给定顶点关联的名称（如果有），否则返回空指针。
- callback：指向要执行的函数的指针，其函数类型为 `void(const void *，entt::registry＆)`。
- data：提供给回调的可选数据。
- children：返回从给定顶点 (vertices) 可到达的节点列表。

由于在注册表中创建池和资源不一定是线程安全的，因此每个顶点还提供了 prepare 函数，可以调用这个函数来设置一个注册表，以便用创建的图来执行:

```c++
auto graph = organizer.graph();
entt::registry registry;

for(auto &&node: graph) {
    node.prepare(registry);
}
```

任务的实际调度是用户的责任，可以使用首选工具 (preferred tool)。

# 4. Meet the runtime

## visit

注册表提供了访问它并获取其管理的组件类型的函数：

```c++
registry.visit([](const auto component) {
    // ...
});
```

访问特定实体存在重载：

```c++
registry.visit(entity, [](const auto component) {
    // ...
});
```

这有助于在主要程基于 C++ 类型系统的注册表和其他不能选择编译时的上下文之间建立一个桥梁。例如：插件系统，元系统，序列化等。

## 克隆注册表 (registry)

克隆注册表并不是一种建议的做法，因为它可能会触发许多副本并降低性能。此外，由于注册表类的设计，将其作为内置特性来支持的话，还会对不需要的用户增加编译时间。更糟糕的是，在实际使用时，很难为不同类型定义克隆策略。

这就是为什么用于克隆的函数定义已移至用户空间的原因。注册表类的 visit 函数以及 insert 函数可以帮助填补空白。

```c++
template<typename Type>
void clone(const entt::registry &from, entt::registry &to) {
    const auto *data = from.data<Type>();
    const auto size = from.size<Type>();

    if constexpr(ENTT_IS_EMPTY(Type)) {
        to.insert<Type>(data, data + size);
    } else {
        const auto *raw = from.raw<Type>();
        to.insert<Type>(data, data + size, raw, raw + size);
    }
}
```

这可能是在注册表不一定为空时，注入实体和组件的最快方法。所有新元素都将附加到现有元素（如果有）上。

此函数还可以进行类型擦除，以便在类型标识符和不透明的克隆方法之间创建映射：

```c++
using clone_fn_type = void(const entt::registry &, entt::registry &);
std::unordered_map<entt::id_type, clone_fn_type *> clone_functions;

// ...

clone_functions[entt::type_hash<position>::value()] = &clone<position>;
clone_functions[entt::type_hash<velocity>::value()] = &clone<velocity>;
```

使用这样的映射，使得对注册表进行 Stamping 变得很简单：

```c++
entt::registry from;
entt::registry to;

// ...

from.visit([this, &to](const auto type_id) {
    clone_functions[type_id](from, to);
});``` 
```

自定义克隆函数也很容易定义。此外，也可以通过这种方式克隆专门使用不同标识符的注册表。

作为附带说明，也可以将克隆函数附加到反射系统，在该反射系统中，使用运行时类型标识符解析元类型。

## Stamping 一个实体

同时使用多个 registry 是很常见的。例如 UI 与模拟的分离，或者在后台加载不同的场景 (可能在单独的线程上)，而不必跟踪哪个实体属于哪个场景。

事实上，在 EnTT 中这甚至是推荐做法，因为注册表只不过是一个可以随时交换的不透明容器。

一旦有多个注册表可用，就需要一种或多种方法将信息从一个容器传输到另一个容器。

由于 Stamping 组件可能需要针对不同类型的不同方法，而且并非所有用户都需要这一特性，因此函数定义已经从 registry 移动到用户空间。

这有助减少编译时间，并提供最大的灵活性，尽管它需要用户设置自己的 Stamping 函数。

这里最好的办法可能是定义一个反射系统或者类型标识符来和用于 Stamping 的不透明函数之间建立映射。举个例子:

```c++
template<typename Type>
void stamp(const entt::registry &from, const entt::entity src, entt::registry &to, const entt::entity dst) {
    to.emplace_or_replace<Type>(dst, from.get<Type>(src));
}
```

如果上面的定义被当作一个通用函数来处理，你可以很容易地构造一个类似下面这样的映射作为一个专用系统的数据成员:

```c++
using stamp_fn_type = void(const entt::registry &, const entt::entity, entt::registry &, const entt::entity);
std::unordered_map<entt::id_type, stamp_fn_type *> stamp_functions;

// ...

stamp_functions[entt::type_hash<position>::value()] = &stamp<position>;
stamp_functions[entt::type_hash<velocity>::value()] = &stamp<velocity>;
```

然后将不同注册表中的实体 Stamping 为：

```c++
entt::registry from;
entt::registry to;

// ...

from.visit(src, [this, &to, dst](const auto type_id) {
    stamp_functions[type_id](from, src, to, dst);
});
```

如果需要，还可以很容易地通过这种方式为特殊类型定义定制 Stamping 函数。此外，在实践中，在具有不同标识符的注册表间对实体进行 Stamping 在实践中也是可行的。

## 快照 (Snapshot): complete 和 continuous

注册表类提供了对序列化的基本支持。

它不会将组件直接转换为字节，因此不需要其他工具来进行序列化。相反，它接受具有对应接口（即档案 *archive* ）的不透明对象，并使用这个对象来序列化其内部数据结构，并在以后需要的时候还原它们。而怎么将类型和实例转换为一堆字节完全由档案来负责。

序列化部分的目标是允许用户生成整个注册表的转储或小范围快照，即只关注用户感兴趣的组件。

要使用 snapshot 类获取注册表的快照，

```c++
output_archive output;

entt::snapshot{registry}
    .entities(output)
    .component<a_component, another_component>(output);
```

不必每次都调用所有函数。要使用哪些函数取决于目的。

entities 成员函数使快照可以序列化所有实体（包括仍然存在和已销毁的实体）及其版本。

component 成员函数是函数模板，其目的是存储备用组件。

使用模板参数列表的原因：

- 首先，没有理由强迫用户一次序列化所有组件，并且在大多数情况下都不需要这样。例如，如果出于某种原因将游戏中 HUD 的内容放入了注册表中，则可以在序列化步骤中自由丢弃 HUD 组件，因为软件可能已经知道如何正确重构它们。
- 此外，注册表在内部大量使用了类型擦除技术，而且在任何时候都不知道它包含了哪些组件类型，因此在调用的时候必须明确指出组件类型。

component 成员函数还有另一个版本，它接受一系列需要序列化的实体。因为一些内部原因，它会多次迭代实体范围，所以会比另一个版本慢一些。但是可以用来过滤掉不需要序列化的实体。

```c++
const auto view = registry.view<serialize>();
output_archive output;

entt::snapshot{registry}.component<a_component, another_component>(output, view.cbegin(), view.cend());``` 
```

这个版本可以在不调用 entities 成员函数的情况下正常工作。

## 快照加载器 (Snapshot loader)

创建快照后，主要有两种方法加载快照：整体加载、连续加载。

连续加载器旨在将数据从源注册表加载到 (可能) 非空目的地。加载器可以在一个注册表中容纳一个以上的快照，以一种连续加载的方式更新。

实体最初拥有的标识符不会传输到目标。相反，加载器在还原快照时将远程标识符映射到本地标识符。因此，这种加载器提供了一种自动更新组件中的标识符的方法 (例如，作为数据成员或在收集在容器)。

连续加载器有一个随时间持续存在的内部状态。因此，不应该将其生命周期限制为临时对象的生命周期。

```c++
entt::continuous_loader loader{registry};
input_archive input;

loader.entities(input)
    .component<a_component, another_component, dirty_component>(input, &dirty_component::parent, &dirty_component::child)
    .orphans()
    .shrink();
```

不必每次都调用所有函数。要使用什么函数主要取决于需求。但要注意，还原数据的顺序要与序列化时的顺序对应。

### .entities

还原实体组，并在需要时将每个实体映射到本地副本。换句话说，对于每个尚未注册到加载器的远程实体标识符，会创建一个本地标识符，以便使本地实体与远程实体保持同步。

### .component

仅还原指定组件，并将它们分配给对应实体。

如果组件本身包含实体（作为 `entt::entity` 类型的数据成员或作为实体的容器），则加载器可以自动更新它们。为此，只需如示例所示指定要更新的数据成员即可。

### .orphans

会在恢复后销毁那些没有组件的实体。它具有与前一节中描述的完全相同的目的，并以相同的方式工作。

### .shrink

帮助清除不再具有远程连接的本地实体。用户应该在每个快照都恢复后调用这个成员函数，除非他们确切地知道自己在做什么。

## 档案 (Archives)

档案必须暴露一组预定义的成员函数。这个 API 很简单，只是一组由快照类和加载器调用的函数调用运算符组成。

### 输出档案

创建快照时使用的档案，必须暴露具有以下签名的函数调用运算符以存储实体：

```c++
void operator()(entt::entity);
```

其中 `entt::entity` 是注册表使用的实体的类型。

请注意，快照类的所有成员函数还会进行一次初始调用，以存储要存储的集合的大小。在这种情况下，函数调用运算符的预期函数类型为：

```c++
void operator()(std::underlying_type_t<entt::entity>);
```

此外，对于要序列化的每种类型，档案必须接受一对实体和组件。因此，给定一个类型 T，档案必须包含具有以下签名的函数调用运算符：

```c++
void operator()(entt::entity, const T &);``` 
```

如何序列化数据由输出档案自己自由决定，注册表不受其影响。

### 输入档案

还原快照时使用的档案，必须暴露具有以下签名的函数调用运算符以加载实体：

```c++
void operator()(entt::entity &);``` 
```

其中 `entt::entity` 是注册表使用的实体的类型。每次调用该函数时，档案都必须从基础存储 (underlying storage) 中读取下一个元素，并将其复制到给定的变量中。

请注意，加载器类的所有成员函数还会进行一次初始调用，以读取它们将要加载的集合的大小。在这种情况下，函数调用运算符的预期函数类型为：

```c++
void operator()(std::underlying_type_t<entt::entity> &);
```

此外，对于要还原的每种类型，存档必须接受一对对实体及其组件的引用。因此，给定类型 T，档案必须包含具有以下签名的函数调用运算符：

```c++
void operator()(entt::entity &, T &);
```

每次调用此类运算符时，档案都必须从基础存储中读取下一个元素，并将其复制到给定的变量中。

### 示例

EnTT 附带了一些示例（实际上是一些测试），这些示例显示了如何将众所周知的库进行序列化以集成为存档。在后台使用 [Cereal C ++](https://uscilab.github.io/cereal/)，主要是因为我想在编写代码时学习它的工作原理。

该代码不是为生产准备的，它既不是唯一的，可能也不是最好的方法。使用它需要自担风险。

基本思路是将所有内容存储在内存中的一组队列中，然后使用不同的加载器将所有内容带回到注册表中。

# 5. 视图 (Views) 和组 (Groups)

视图和组是执行单一职责的好工具。有权访问注册表的系统可以创建和销毁实体，以及分配和删除组件。而有权访问视图或组的系统只能迭代、读取、更新实体和组件。

- 视图：是一种非侵入性工具，用于访问实体和组件，而不会影响其他功能或增加内存消耗。
- 组：是一种侵入性工具，可以在关键路径上实现更高的性能，但会为此导致一些其他问题。

视图主要有两种：编译时 *compile-time* （view）和运行时 runtime （runtime_view）。

前者需要组件类型的编译时列表，因此可以进行多项优化。后者可以在运行时构造，而不是使用整型标识符，不过迭代速度要慢一些。

这两种情况下，视图在创建和销毁时的开销并不大，因为它们没有任何类型的初始化。

组有三种不同的类型：完全拥有 (full-owning) 组、部分拥有 (partial-owning) 组和非拥有 (non-owning) 组。它们之间的主要区别在于性能。

组实际上可以拥有一种或多种组件类型。并允许他们重新排列池，以加快迭代速度。简单来说：一个组拥有的组件越多，它们的迭代速度就越快。

一个给定的组件只有在它们是嵌套的时候才能属于多个组，因此用户必须仔细定义组，才能充分利用它们。

## 视图 (Views)

构造视图时，是为单个组件，还是为迭代多个组件，会导致视图行为有所不同。甚至这两种情况下，API 也会略有不同。

单组件视图是专门设计的，以便在所有情况下提高性能。这种视图可以直接访问底层数据结构，避免多余的检查。没有什么比单个组件视图更快了。事实上，它们会遍历已经打包的组件数组，然后一次返回一个组件。

单组件视图提供了很多函数来获取它们将要返回的实体的数量，以及对实体列表和组件列表的原始访问。也可以询问视图是否包含给定的实体。

多组件视图迭代包中包含所有给定组件的实体。在构建过程中，视图查看每个组件可用的实体数量，并获取对最小候选集的引用，以加速迭代。

多组件视图提供的函数比单组件视图少。需要注意的是，多组件视图暴露 utility 函数，以获取它将要返回实体的估计数量，并确定它是否包含给定的实体。

不要需存储视图，因为它们的构建成本非常低，可以任意地复制并重用它们。视图还会在调用 begin 或 end 时，返回新创建且正确初始化了的迭代器。

两种视图都是通过注册表来创建：

```c++
// 单组件视图
auto single = registry.view<position>();

// 多组件视图
auto multi = registry.view<position, velocity>();
```

支持按组件过滤实体:

```c++
auto view = registry.view<position, velocity>(entt::exclude<renderable>);``` 
```

要迭代视图，使用 range-for 循环:

```c++
auto view = registry.view<position, velocity, renderable>();

for(auto entity: view) {
    // 一次一个组件 ...
    auto &position = view.get<position>(entity);
    auto &velocity = view.get<velocity>(entity);

    // ... 多组件 ...
    auto [pos, vel] = view.get<position, velocity>(entity);

    // ... 一次所有组件
    auto [pos, vel, rend] = view.get(entity);

    // ...
}
```

或者也可以使用 each 函数同时迭代实体和组件:

```c++
// 通过回调
registry.view<position, velocity>().each([](auto entity, auto &pos, auto &vel) {
    // ...
});

// 使用输入迭代器
for(auto &&[entity, pos, vel]: registry.view<position, velocity>().each()) {
    // ...
}
```

请注意，当通过回调接收到实体时，可以从参数列表中过滤实体，这可以进一步提高迭代期间的性能。

因为没有显式地实例化它们，所以在任何情况下都不返回空组件。顺便说一下，对于单个组件视图，get 不严格要求提供模板参数，因为类型是隐式定义的，只不过，如果没有指定类型，为了与多组件视图保持一致，实例将使用一个元组返回:

```c++
auto view = registry.view<const renderable>();

for(auto entity: view) {
    auto [renderable] = view.get(entity);
    // ...
}
```

注意：在迭代期间，最好使用视图的 get 成员函数，而不是注册表的 get 成员函数，以便获取视图本身迭代的类型。

### 视图包 (View pack)

视图包允许用户将多个视图组合成一个类似于视图的可迭代对象，同时还使他们可以完全控制哪个视图应该导致迭代。

该对象返回所有视图中存在的实体。它的主要用途是自定义存储和视图，但是在日常使用中也非常方便。

视图包的创建试图模仿 C++ 20 ranges：

```c++
auto view = registry.view<position>();
auto other = registry.view<velocity>();

auto pack = view | other;
```

返回类型是类模板 `entt::view_pack` 的特化。这只是一个类似于视图的迭代对象，它将多个视图组合到一个实例中。

用于创建包的第一个视图会和导致迭代的视图相同。

视图包提供了很多类似于多组件视图的函数。另外，如果直接迭代，它只会返回实体：

```c++
for(auto entt: pack) {
    // ...
}
```

另一方面，当使用 each 成员函数时，都将返回（可选）实体和组件，无论是使用回调还是带有扩展的可迭代对象：

```c++
// 带回调
pack.each([](const auto entt, auto &pos, auto &vel) { /* ... */ });

// 带有扩展的可迭代对象
for(auto [entt, pos, vel]: pack.each()) {
    // ...
}
```

此外，视图包返回的类型的常量性直接继承自组成它的视图:

```c++
(registry.view<position>() | registry.view<const velocity>()).each([](auto &pos, const auto &vel) {
    // ...
});
```

另请阅读 dedicated 部分，以了解在自定义存储和池的创建和使用中如何使用视图包。

## 运行时视图 (Runtime views)

运行时视图会迭代它的包中所包含的所有给定组件的实体。在构建期间，这些视图查看每个组件可获得 (available) 实体的数量，并选择对最小候选集的引用，以加快迭代速度。

它们或多或少提供了与多组件视图的相同函数。但是不暴露 get 成员函数，用户应该引用生成视图的注册表来访问组件。特别是运行时视图暴露了 utility 函数，以获取它将要返回的实体的估计数量，并确定其是否为空。也可以通过运行时视图查询是否包含给定的实体。

运行时视图的构建成本非常低廉，并且任何时候都不应该长久存储它们。在创建后应立即使用它们，然后将其丢弃。不过需要这样做的原因远远超出了本文档的范围。

要迭代运行时视图，可以使用 range-for 循环：

```c++
entt::id_type types[] = { entt::type_hash<position>::value(), entt::type_hash<velocity>::value() };
auto view = registry.runtime_view(std::cbegin(types), std::cend(types));

for(auto entity: view) {
    // ...
}
```

或者使用 each 成员函数来迭代实体:

```c++
entt::id_type types[] = { entt::type_hash<position>::value(), entt::type_hash<velocity>::value() };

registry.runtime_view(std::cbegin(types), std::cend(types)).each([](auto entity) {
    // ...
});
```

两种情况下的性能完全相同。

这种视图也支持按组件过滤实体：

```c++
entt::id_type components[] = { entt::type_hash<position>::value() };
entt::id_type filter[] = { entt::type_hash<velocity>::value() };
auto view = registry.runtime_view(std::cbegin(components), std::cend(components), std::cbegin(filter), std::cend(filter));
```

注意：运行时视图适用于用户在编译时不知道要使用哪些组件迭代实体的情况。如果可能的话，请尽量不要使用运行时视图，因为它们的性能不如其他视图。

## 组 (Groups)

组旨在一次迭代多个组件，并为多组件视图提供更快的替代方法。

组有着比其他可用工具更好的性能，但是需要获得组件的所有权，这对池设置了一些约束。另一方面，组并不是一种增加内存消耗、影响功能并试图为所有可能的组件组合优化其迭代的自动行为。

组最有趣的方面是它们符合使用模式。其他解决方案通常会尝试优化所有内容，但就性能和内存使用而言，这会导致很高的成本。而且可能让会用户为不需要的东西付出代价，而这并不是我最喜欢的东西。更糟糕的是，很难防止这种行为。组的工作方式有所不同，设计的目的是只在用户发现他们需要的时候才对实际用例进行优化。

组的另一个的特性是，它们对内存消耗没有影响，将非拥有群组放在一边，这种群组非常罕见，应该尽可能避免。

所有组都在一定程度上影响其组件的创建和销毁。这是因为他们必须观察目标池中的变化，并在需要时为它们所拥有的类型正确地排列数据。

组提供了很多函数来获取将要返回的实体的数量，以及对实体列表以及所拥有组件的组件列表的原始访问。也可以询问组是否包含给定的实体。

无需存储组，因为它们的构建成本非常低廉，可以随意复制并重用它们。组在第一次请求时执行初始化步骤，这可能会非常昂贵。为避免这种情况，请考虑在尚未分配任何组件时创建组。如果注册表为空，则准备工作非常快。每当调用 begin 或 end 时，组也会返回新创建并正确初始化的迭代器。

要迭代组，可以使用 range-for 循环：

```c++
auto group = registry.group<position>(entt::get<velocity, renderable>);

for(auto entity: group) {
    // 一次一个组件 ...
    auto &position = group.get<position>(entity);
    auto &velocity = group.get<velocity>(entity);

    // ... 多个组件 ...
    auto [pos, vel] = group.get<position, velocity>(entity);

    // ... 一次所有组件
    auto [pos, vel, rend] = group.get(entity);

    // ...
}
```

或者使用 each 函数同时迭代实体和组件:

```c++
// 通过回调
registry.group<position>(entt::get<velocity>).each([](auto entity, auto &pos, auto &vel) {
    // ...
});

// 使用输入迭代器
for(auto &&[entity, pos, vel]: registry.group<position>(entt::get<velocity>).each()) {
    // ...
}
```

请注意，当通过回调接收到实体时，可以从参数列表中过滤实体，这可以进一步提高迭代期间的性能。

由没有显式实例化它们，因此无论如何都不返回空组件。

注意：在迭代过程中，最好不要使用注册表的 get 成员函数，而是使用组的 get 成员函数来获取由组本身迭代的类型。

### Full-owning groups

完全拥有组是用户一次迭代多个组件的最快工具。它直接迭代所有组件，而无需间接调用。这种类型的组或多或少地表现得好似用户正在依次访问一堆打包的组件数组，这些组件的排序完全相同，没有跳转或分支。

完全拥有组的创建方式为：

```c++
auto group = registry.group<position, velocity>();
```

还支持按组件过滤实体：

```c++
auto group = registry.group<position, velocity>(entt::exclude<renderable>);
```

创建后，组将获得模板参数列表中指定的所有组件的所有权，并根据需要排列其池。创建组后，将不再允许对拥有的组件进行排序。但是，完全拥有组可以通过其 sort 成员函数进行分类。对完全拥有组进行排序会影响其所有实例。

### Partial-owning groups

对于拥有组，部分拥有组的工作方式与完全拥有组类似，但是依赖于间接获取其他组拥有的组件。这不像完全拥有组那么快，但是当只有一个或两个可用组件需要检索时 (最常见的情况) ，它已经比视图快得多。在最坏的情况下，它也不会比视图慢。

创建部分拥有组：

```c++
auto group = registry.group<position>(entt::get<velocity>);
```

还支持按组件过滤实体：

```c++
auto group = registry.group<position>(entt::get<velocity>, entt::exclude<renderable>);
```

创建后，组将获得模板参数列表中指定的所有组件的所有权，并根据需要排列其池。通过 `entt::get` 提供的类型的所有权不会传递给该组。

创建组后，将不再允许对拥有的组件进行排序。但是，部分拥有组可以通过其 sort 成员函数进行分类 对部分拥有组进行排序会影响其所有实例。

### Non-owning groups

非拥有组通常足够快，可以肯定比视图快，并且适合大多数情况。但是，它们需要自定义数据结构才能正常工作，并且会增加内存消耗。根据经验，如果可能，用户应避免使用非拥有组。

创建非拥有组的方式如下：

```c++
auto group = registry.group<>(entt::get<position, velocity>);
```

还支持按组件过滤实体：

```c++
auto group = registry.group<>(entt::get<position, velocity>, entt::exclude<renderable>);
```

在这种情况下，该组不会获得任何类型的组件的所有权。因此，这种类型的组通常是总体表现最差的组，但也是唯一可以在任何情况下都能使用的组，以略微提高性能。

非拥有组可以通过其 sort 成员函数进行分类。对非拥有组进行排序会影响其所有实例。

### 嵌套组 (Nested groups)

一种组件类型不能被两个或多个冲突 (conflicting) 组所拥有，例如：

- `registry.group<transform, sprite>()`
- `registry.group<transform, rotation>()`

但是，属于同一家族的组可以拥有同一类型，这也被称为嵌套组，例如：

- `registry.group<sprite, transform>()`
- `registry.group<sprite, transform, rotation>()`

它允许在更多的组件组合上提高性能。

两个嵌套组至少拥有一种组件类型，其中一个组所涉及的组件类型会完全包含在另一个组的类型列表中。

因此，用于定义两个或更多组是否嵌套的规则可以概括为：

- 其中一个组中的组件类型要涉及另一个组中的一个或多个的组件类型，无论它们是拥有 (owned)，观察 (observed) 还是排除 (excluded)。
- 限制性最强的组所拥有的组件类型列表，需要与其他组所拥有的组件类型列表相同，或者是完全包含其他组件类型。这也适用于观察和排除的组件列表。

这意味着嵌套组通过以新组件的形式添加更多条件来扩展它们的父组。

正如前面所提到的，组件不一定都是拥有的 ( *owned* )，这样两个组就可以被认为是嵌套的。下列定义完全有效:

- `registry.group<sprite>(entt::get<renderable>)`
- `registry.group<sprite, transform>(entt::get<renderable>)`
- `registry.group<sprite, transform>(entt::get<renderable, rotation>)`

排除列表也会有一定的影响。在定义嵌套组时，被排除的组件类型 T 被视为观察类型 not_T。因此，考虑以下两个定义:

- `registry.group<sprite, transform>()`
- `registry.group<sprite, transform>(entt::exclude<rotation>)`

它们被视为用户定义了以下组:

- `group<sprite, transform>()`
- `group<sprite, transform>(entt::get<not_rotation>)`

其中 not_rotation 是一个空标记，仅当没有 rotation 时才存在。

因此，要定义一个比现有组更严格的新组，只需获取后者的组件类型列表，并通过添加新的组件类型来扩展它。

反之亦然。要定义一个更大的组，仅需采用一个现有组并从中删除约束即可，无论它们以何种形式表示。

请注意，一个组所涉及的组件类型越多，限制就越大。

尽管嵌套组具有极大的灵活性，它允许独立使用拥有，观察或排除的组件类型，但此工具的真正优势在于可以定义更多拥有相同组件的组，从而在更多情况下提供最佳性能。

实际上，给定组所涉及的组件类型的列表，所拥有的组件数量越多，组本身的性能就越高。

附带说明一下，在定义嵌套分组时，不再能对所有分组进行排序。这是因为限制最大的组与限制较小的组共享其元素，并且命令后者将使前者无效。

但是，给定一个嵌套组的族，仍然可以对限制性最强的组进行排序。为了避免用户必须记住其组中限制最大的组，注册表类提供了 sortable 成员函数，用来确定是否可以对组进行排序。

## 类型：const 和 non-const 及两者之间的类型

在构造视图和组时，注册表类提供了两个重载：const 版本和非 const 版本。前者仅接受 const 类型作为模板参数，后者则接受 const 和非 const 类型。

这意味着可以从 const 注册表构造视图和组，并将视图的常量传播到所涉及的类型。举个例子：

```c++
entt::view<const position, const velocity> view = std::as_const(registry).view<const position, const velocity>();
```

考虑将以下定义用于非常量视图：

```c++
entt::view<position, const velocity> view = registry.view<position, const velocity>();
```

在上面的示例中，视图可用于访问只读或可写的 position 组件，而 velocity 组件在所有情况下均为只读。

同样，这些语句都是有效的：

```c++
position &pos = view.get<position>(entity);
const position &cpos = view.get<const position>(entity);
const velocity &cpos = view.get<const velocity>(entity);
std::tuple<position &, const velocity &> tup = view.get<position, const velocity>(entity);
std::tuple<const position &, const velocity &> ctup = view.get<const position, const velocity>(entity);
```

相反，下面会导致编译错误：

```c++
velocity &cpos = view.get<velocity>(entity);
std::tuple<position &, velocity &> tup = view.get<position, velocity>(entity);
std::tuple<const position &, velocity &> ctup = view.get<const position, velocity>(entity);
```

each 函数也将常量传递给它的返回值:

```c++
view.each([](auto entity, position &pos, const velocity &vel) {
    // ...
});
```

调用者仍然可以通过 const 引用来引用 position 组件，因为幸运的是该语言已经允许了该规则。

相同的概念也适用于组。

## Give me everything

视图和组是整个实体列表上的小窗口。根据其组件过滤实体来工作。

在某些情况下，可能需要迭代所有仍在使用的实体，而不管它们的组件是什么。注册表提供了一个特定的成员函数来执行此操作：

```c++
registry.each([](auto entity) {
    // ...
});
```

它将所有仍在使用的实体返回给调用者。

根据经验，如果目标是迭代具有确定组件集的实体，请考虑使用视图或组。这些工具通常比将这个函数与一系列自定义测试结合起来要快得多。

在所有其他情况下，这是正确的方法。

还有另一个成员函数可用于检索 orphans。orphans 实体是一个仍在使用且没有分配组件的实体。

函数的签名与 each 相同:

```c++
registry.orphans([](auto entity) {
    // ...
});
```

要测试单个实体的 *orphanity* ，请改用成员函数 orphan。它接受有效的实体标识符作为参数，并在实体为 *orphanity* 的情况下返回 true，否则返回 false。

通常，所有这些函数都可能导致性能下降。

由于对每个实体执行某些检查，因此每个处理速度都很慢。类似的原因，orphans 的速度甚至会更慢。请勿经常使用这两个函数，以免造成性能下降的风险。

## 什么允许什么不允许

现有的大多数 ECS 不允许在迭代过程中创建销毁实体和组件。

EnTT 可以部分解决此问题，但有一些限制：

- 在大多数情况下，在迭代过程中允许创建实体和组件。
- 迭代期间允许销毁当前实体或其组件。对所有其他实体，则不允许销毁或删除其组件，这可能导致未定义的行为。

在这些情况下，迭代器不会失效。需要明确的是，这并不意味着引用也将继续有效。

考虑以下示例：

```c++
registry.view<position>([&](const auto entity, auto &pos) {
    registry.emplace<position>(registry.create(), 0., 0.);
    pos.x = 0.; // warning: dangling pointer
});
```

Each 函数不会中断 (因为迭代器不会失效) ，但是对引用没有保证。使用通用 range-for 循环，来直接从视图获取组件，或者将组件的创建移到函数的末尾，以避免出现悬浮指针。

相反，如果一个实体被修改或销毁，并且它既不是迭代器当前返回的实体，也不是新创建的实体，那么迭代器就会失效，后续行为也是未定义的。

为了解决这个问题，可以采取以下方法:

- 先存储要删除的实体和组件，并在迭代结束后执行删除操作。
- 使用适当的标记组件标记实体和组件，表明它们需要被清除，然后第二次执行迭代，逐个清除它们。

此功能的显着副作用是，在大多数情况下，所需分配的数量会进一步减少。

### 更好的性能，更多的约束

组是视图的（快得多）替代方法。但是，性能越高，对允许和不允许的条件的约束就越大。

特别是，在极少数情况下，组会增加在迭代过程中创建组件的限制。它发生在非常特殊的情况下。考虑到组的性质和范围，它可能不会碰到，但无论如何需要知道有这个情况。

首先，必须指出的是，在迭代组时创建组件根本不是问题，而且可以像处理视图一样随意进行。对于适用上述规则的部件和实体的销毁也是如此。

但当在组外迭代组拥有的给定组件时，就会出现额外的限制。在这种情况下，添加属于该组的组件可能会使迭代器无效。但对组件和实体的销毁没有进一步的限制。

幸运的是，这样的情况几乎从来没有发生过，只会在一些条件下才会发生。 尤其是：

- 使用单组件视图迭代属于组的组件类型，并向实体添加了能将它添加到组中所需的所有组件，可能会使迭代器无效。
- 使用多组件视图迭代属于组的组件类型，并向实体添加了能将其添加到组中所需的所有组件，会使迭代器无效，除非用户指定另一种类型的组件来引导视图的迭代顺序 (在这种情况下，前者被视为自由类型，不受限制的影响)。

换句话说，只要将类型视为自由类型（例如，具有多组件视图以及部分或非所有者组的示例）或使用其自己的组进行迭代，就不会存在该限制，但是，如果将类型用作迭代规则的主要类型，则可能出现这种情况。

这是因为组拥有其组件池，并在内部组织数据以最大限度地提高性能。因此，只有在将拥有的组件作为其组的一部分或作为具有多个组件视图和组的自由类型进行迭代时，才可以保证所拥有组件的完全一致性。

# 6. 空类型优化

空类型 T 使得 `std::is_empty_v<T>` 返回 true。它们也是可以进行空基优化（EBO）的类型。

EnTT 以一种特殊的方式处理这些类型，在性能和内存使用方面进行优化。这里的影响值得一提。

当检测到空类型时，无论如何都不会实例化它。因此，只有分配给它实体才能使用。

没有可以迭代空类型的方法。视图和组永远不会返回空类型的实例（例如，在调用 each 时），并且某些函数（例如 try_get 或对组件列表的原始访问）不可用。 最后，sort 功能仅接受需要返回实体而不是组件的回调：

```c++
registry.sort<empty_type>([](const entt::entity lhs, const entt::entity rhs) {
    return entt::registry::entity(lhs) < entt::registry::entity(rhs);
});
```

另一方面，迭代速度更快，因为仅考虑分配了类型的实体。而且使用的内存更少，主要是因为无论组件被分配给多少实体，都不存在该组件的任何实例。

一般而言，该库提供的特性均不受影响，但是对于那些需要返回实际实例的特性会受到影响。

可以通过定义 ENTT_NO_ETO 宏来禁用此优化。在这种情况下，无论如何，空类型都将像其他所有类型一样对待。

# 7.  多线程

通常，整个注册表不是线程安全的。出于多种原因，线程安全并不是用户需要的。例如为了性能。

视图、组以及因此被 EnTT 所采用的方法是这个规则的最大例外。视图、组和迭代器本身通常不是线程安全的。因此，用户不应该尝试迭代一组组件并且并发地修改它。然而：

- 当一个线程在迭代拥有 X 组件的实体时，或是从一组实体中指定并删除组件时，另一个线程就可以安全地对组件 Y 和 Z 做同样的事情，一切都会像魔法一样运行。举一个简单的例子，用户可以自由地执行渲染系统并迭代可渲染的实体，同时在一个单独的线程上并发地更新一个物理组件。
- 同样，只要没有同时分配或删除组件，就可以通过多个线程来迭代一组组件。换句话说，假设的运动系统可以启动多个线程，每个线程都将访问包含其实体的速度和位置信息的组件。

这种实体组件系统既可以用于单线程应用程序中，也可以与异步对象或多线程一起使用。此外，典型的 ECS 基于线程的模型并不需要一个完全线程安全的注册表来工作。实际上，在使用大多数常见模型时，用户可以通过注册表实现目标。

最后，可以通过一些编译时定义对 EnTT 进行配置，使其某些部分隐式地变为线程安全的，粗略地说，只有那些真正有意义且不能逆转的部分。

特别是，当在不同线程中使用引用类型索引生成器对象的多个实例（例如注册表类）时，定义 ENTT_USE_ATOMIC 可能会很有用。

有关更多信息，请参见相关文档。

## 迭代器

对于视图和组返回的迭代器，需要特别提及。在大多数情况下，它们满足随机访问迭代器的要求，在所有情况下，它们至少满足双向迭代器的要求。

换句话说，它们适合与标准库的并行算法一起使用。例如，这种迭代器可以与 `std::for_each` 和 `std::execution::par` 结合使用，以使访问并行化，从而更新视图或组返回的组件，只要遵守前面讨论的约束:

```c++
auto view = registry.view<position, const velocity>();

std::for_each(std::execution::par_unseq, view.begin(), view.end(), [&view](auto entity) {
    // ...
});
```

这可以极大地增加吞吐量。

不幸的是，由于当前标准修订版的局限性，并行的 `std::for_each` 仅接受前向迭代器。这意味着库提供的默认迭代器无法将代理对象作为引用返回，而必须返回实际的引用类型。

将来这种情况可能会改变，默认情况下迭代器肯定会返回实体和对其组件的引用列表。Multi-pass 保证在任何情况下都不会中断，性能甚至应该从中进一步受益。