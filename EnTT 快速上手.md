# EnTT 快速上手

EnTT 是一个现代 C++ 实现的 ECS ，它提供了一种轻量级和高效的方式来管理实体和组件。下面是 EnTT 的使用方法：

### 安装

EnTT 可以通过包管理器或从源代码构建来安装。如果使用包管理器，则可以使用以下命令安装 Entt：

```bash
# 使用 vcpkg 安装
vcpkg install entt

# 使用 Conan 安装
conan install entt/3.8.0

# 使用 apt 安装
sudo apt-get install libentt-dev
```

如果从源代码构建，则需要执行以下步骤：

1. 克隆 EnTT 的仓库：

```bash
git clone https://github.com/skypjack/entt.git
```

1. 使用 CMake 构建：

```bash
cd entt
mkdir build
cd build
cmake ..
make
```

### 实体

实体是一个简单的标识符，可以添加、删除、遍历和查询组件。在 EnTT 中，实体由 `entt::entity` 类型表示。

#### 创建实体

要创建一个实体，请使用实体管理器的 `create` 方法：

```c++
entt::registry registry;
entt::entity entity = registry.create();
```

#### 删除实体

要删除一个实体，请使用实体管理器的 `destroy` 方法：

```c++
registry.destroy(entity);
```

#### 查询实体是否存在

要查询一个实体是否存在，请使用实体管理器的 `valid` 方法：

```c++
if (registry.valid(entity)) {
  // entity exists
}
```

#### 查询实体的数量

要查询实体的数量，请使用实体管理器的 `size` 方法：

```c++
std::size_t count = registry.size();
```

#### 遍历所有实体

要遍历所有实体，请使用实体管理器的 `each` 方法：

```c++
registry.each([](auto entity) {
  // do something with entity
});
```

### 组件

组件是与实体相关联的数据。在 EnTT 中，组件可以是任何类型。

#### 添加组件

要添加一个组件，请使用实体管理器的 `emplace` 方法：

```c++
registry.emplace<Position>(entity, 0.0f, 0.0f);
```

#### 删除组件

要删除一个组件，请使用实体管理器的 `remove` 方法：

```c++
registry.remove<Position>(entity);
```

#### 查询实体是否具有组件

要查询一个实体是否具有特定类型的组件，请使用实体管理器的 `has` 方法：

```c++
if (registry.has<Position>(entity)) {
  // entity has Position component
}
```

#### 获取实体的组件

要获取一个实体的组件，请使用实体管理器的 `get` 方法：

```c++
auto& position = registry.get<Position>(entity);
```

#### 遍历具有特定组件的实体

要遍历具有特定类型组件的所有实体，请使用实体管理器的 `view` 方法：

```c++
for (auto entity : registry.view<Position>()) {
  auto& position = registry.get<Position>(entity);
  // do something with position
}
```

#### 遍历具有多个组件的实体

要遍历具有多个类型组件的所有实体，请使用实体管理器的 `view` 方法和 `each` 方法：

```c++
registry.view<Position, Velocity>().each([](auto entity, auto& position, auto& velocity) {
  // do something with position and velocity
});
```

### 系统

系统是用于处理实体和组件的函数。在 EnTT 中，系统由一个或多个函数组成。

#### 创建系统

要创建一个系统，请使用 `entt::system` 类型和一个或多个函数指针：

```c++
void update_position(entt::registry& registry) {
  for (auto entity : registry.view<Position>()) {
    auto& position = registry.get<Position>(entity);
    position.x += 1.0f;
    position.y += 1.0f;
  }
}

entt::system<void(entt::registry&)> update_system = &update_position;

entt::registry registry;
registry.on_update(update_system);
```

`entt::system<void(entt::registry&)>`是一个函数对象类型，它表示一个可以接受 EnTT 实体注册表作为参数的系统。其中，`void`表示该系统不返回任何值。在使用时，可以将一个函数指针或者 lambda 表达式转换为这个类型，并传递给 EnTT 的`basic_registry::on_update`函数，使其在每个帧更新时被调用。用于注册一个每帧更新时调用`update_position`函数的系统。这样，在每个帧更新时，`update_position`函数将被调用，并遍历所有具有`Position`组件的实体，更新它们的位置。

需要注意的是，在使用`entt::system`时，还可以通过`entt::sink`类型将多个系统组合在一起，以便在同一帧更新中一起执行。

#### 系统触发器

系统触发器是在特定条件下触发系统函数的函数。在 EnTT 中，系统触发器由 `entt::dispatcher` 类型表示。

##### 构造函数

要创建一个系统触发器，请使用默认构造函数：

```c++
entt::dispatcher dispatcher;
```

##### 添加系统

要添加一个系统，请使用系统触发器的 `sink` 方法：

```c++
dispatcher.sink<Update>().connect<&update_position>();
```

这将在 `Update` 事件触发时调用 `update_position` 函数。

##### 触发系统

要触发一个系统，请使用系统触发器的 `trigger` 方法：

```c++
dispatcher.trigger<Update>(registry);
```

这将触发所有连接到 `Update` 事件的系统函数。

### 实体标记

实体标记是一个简单的标志，用于标识实体是否具有特定的属性。

#### 添加实体标记

要添加实体标记，请使用实体管理器的 `assign` 方法：

```c++
registry.assign<Tag>(entity);
```

#### 查询实体标记

要查询实体标记，请使用实体管理器的 `has` 方法：

```c++
if (registry.has<Tag>(entity)) {
  // entity has Tag
}
```

#### 遍历具有特定标记的实体

要遍历具有特定标记的所有实体，请使用实体管理器的 `view` 方法：

```c++
for (auto entity : registry.view<Tag>()) {
  // do something with entity
}
```

了解更多关于 EnTT 的信息，请参阅其文档和示例（https://skypjack.github.io/entt/index.html）。

### 使用 EnTT 的一个完整实例

下面是一个使用 EnTT 的例子，其中我们使用 EnTT 来管理一些物体的位置和速度，并且在更新位置时检测它是否超出界限：

```c++
#include <iostream>
#include <entt/entt.hpp>

struct Position { float x,y; };
struct Velocity { float dx,dy; };
```

首先，我们包含 EnTT 的头文件以及标准输入输出头文件，并定义了两个组件 `Position` 和 `Velocity`，分别代表物体的位置和速度。

```c++
void update(entt::registry &registry, float dt) {
    registry.view<Position, Velocity>().each([dt](auto entity, Position &pos, Velocity &vel) {
        pos.x += vel.dx * dt;
        pos.y += vel.dy * dt;

        // Check bounds
        if (pos.x < 0 || pos.x >= 800 || pos.y < 0 || pos.y >= 600) {
            registry.destroy(entity);
        }
    });
}
```

这里我们定义了一个更新函数 `update`，传入一个 registry 对象和一个时间增量 `dt`。在函数内部，我们从对象池中查询带有 `Position` 和 `Velocity` 组件的所有实体，并用 lambda 表达式对它们进行遍历。在遍历过程中，我们先根据它们的速度更新它们的位置，再检查它们是否超过了屏幕边界。如果超界，则使用 `registry.destroy` 函数销毁该实体。

```c++
int main() {
    entt::registry registry;

    // Create entities with Position and Velocity components
    auto entity1 = registry.create();
    registry.emplace<Position>(entity1, 100.f, 100.f);
    registry.emplace<Velocity>(entity1, 50.f, 50.f);

    auto entity2 = registry.create();
    registry.emplace<Position>(entity2, 400.f, 400.f);
    registry.emplace<Velocity>(entity2, -100.f, -100.f);
```

在 `main` 函数中，我们定义了一个 registry 对象并创建两个实体。对于每个实体，我们添加了 `Position` 和 `Velocity` 组件，并传递了相应的参数，即它们的位置和速度。

```c++
// Update entities
    update(registry, 1.f);
```

接着，我们调用 `update` 函数，传入 registry 对象和时间增量 `1.f`。

```c++
// Print entity positions
    registry.view<Position>().each([](auto entity, Position &pos) {
        std::cout << "Entity " << entity << " position: (" << pos.x << ", " << pos.y << ")" << std::endl;
    });
```

最后，我们遍历所有带有 `Position` 组件的实体，并打印它们的位置。

这个例子展示了如何使用 EnTT 管理实体和组件，并实现简单的逻辑处理。在真实的项目中，可以根据需要进一步扩展实体和组件，以及实现更复杂的逻辑处理。

注意，在使用 EnTT 时，需要在代码中包含 `entt/entt.hpp` 头文件，并将 `registry` 对象定义为全局变量或传递给函数。另外，在创建尽可能少的实体时，应该使用对象池，可以避免频繁的内存分配和释放，并重复利用已经存在的实体对象。另外，还可以提高数据的局部性，因为所有实体对象都存储在同一个内存池中。这可以提高缓存命中率，并减少内存访问延迟，从而进一步提高性能。

下面是完整代码：

```c++
#include <iostream>
#include <entt/entt.hpp>

struct Position { float x,y; };
struct Velocity { float dx,dy; };

void update(entt::registry &registry, float dt) {
    registry.view<Position, Velocity>().each([dt](auto entity, Position &pos, Velocity &vel) {
        pos.x += vel.dx * dt;
        pos.y += vel.dy * dt;

        // Check bounds
        if (pos.x < 0 || pos.x >= 800 || pos.y < 0 || pos.y >= 600) {
            registry.destroy(entity);
        }
    });
}

int main() {
    entt::registry registry;

    // Create entities with Position and Velocity components
    auto entity1 = registry.create();
    registry.emplace<Position>(entity1, 100.f, 100.f);
    registry.emplace<Velocity>(entity1, 50.f, 50.f);

    auto entity2 = registry.create();
    registry.emplace<Position>(entity2, 400.f, 400.f);
    registry.emplace<Velocity>(entity2, -100.f, -100.f);

    // Update entities
    update(registry, 1.f);

    // Print entity positions
    registry.view<Position>().each([](auto entity, Position &pos) {
        std::cout << "Entity " << entity << " position: (" << pos.x << ", " << pos.y << ")" << std::endl;
    });

    return 0;
}
```
