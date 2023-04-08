ECS 是游戏开发中一种比较热门的架构模式，虽然很早就被提出，但被大家所熟识并引起热烈讨论，还是源于GDC2017，《守望先锋》针对它们的 ECS 架构进行的一次技术分享。针对 FPS，MOBA 这类的竞技游戏，ECS 架构有着得天独厚的优势。

## 为什么需要 ECS

在游戏开发时，如果以面向对象的思想对游戏对象进行抽象，那么可以实现一个基类GameObject。之后包括敌机、自机、子弹在内的所有对象都继承这个基类并进行实现。

于是很容易设计出这样一个基类：

```cpp
class GameObject {
private:
    // 基础属性
    int m_Position;  // 位置
    int m_Rotation;  // 方向
    int m_Scale;     // 缩放

    // 碰撞属性
    int m_Size;      // 大小

public:
    voidUpdate();
    voidRender();
};
```

之后，继承这个基类，分别实现GameBullet、GameEnemy……

这样带来的问题就是**游戏对象的逻辑往往在超类中被定死了，而一旦需要对逻辑作出修改，要么重写实现，要么继承基类进行覆盖**。此外，在C++中使用对象池优化时就会造成灾难性的后果——一种类型一个池（尽管可以用通用的内存分配器，但是这样还要考虑内存碎片等杂七杂八的问题）。

对于传统的设计思路，在游戏开发上就会导致**“类灾难”**。于是 ECS 架构被提了出来，用于解决继承带来的问题。

**使用继承去表述游戏对象和逻辑会造成逻辑混杂、维护扩展困难的问题。既然继承出了问题，我们就用组合来解决吧。**于是在2002年的Game Dungeon Siege上，ECS 架构被提了出来。



## **何为 ECS 架构**

***ECS***，即 Entity-Component-System（实体-组件-系统） 的缩写。其模式遵循**组合优于继承**原则，**游戏内的每一个基本单元都是一个实体，每个实体又由一个或多个组件构成，每个组件仅仅包含代表其特性的数据（即在组件中没有任何方法）**，例如：移动相关的组件`MoveComponent`包含速度、位置、朝向等属性，一旦一个实体拥有了`MoveComponent`组件便可以认为它拥有了移动的能力，**系统便是来处理拥有一个或多个相同组件的实体集合的工具，其只拥有行为（即在系统中没有任何数据）**，在这个例子中，处理移动的系统仅仅关心拥有移动能力的实体，它会遍历所有拥有`MoveComponent`组件的实体，并根据相关的数据（速度、位置、朝向等），更新实体的位置。

实体与组件是一个一对多的关系，实体拥有怎样的能力，完全是取决于其拥有哪些组件，通过动态添加或删除组件，可以在（游戏）运行时改变实体的行为。

简言之，**实例就是一个游戏对象实体，一个实体拥有众多的组件，而游戏系统则负责依据组件对实例做出更新。**

再比如上面的例子，如果对象A需要实现碰撞和渲染，那么我们就给它加一个碰撞组件和一个渲染组件；如果对象B只需要渲染不需要碰撞，那么我们就给它加一个渲染组件即可。而在游戏循环中，每一个系统都会遍历一次对象，当渲染系统发现对象持有一个渲染组件时，就会根据渲染组件的数据来执行相应的渲染过程。同样的碰撞系统也是如此。

也就是说游戏对象需要什么就会给自己加一个组件。而系统会依据游戏对象增加了哪些组件来做出行为。换言之实体只需要持有必要的数据，由系统负责逻辑就行了。这也就是ECS模式能和数据驱动很好结合的一个原因。



## **ECS 基本结构**

一个使用 ECS 架构开发的游戏基本结构如下图所示：



![img](https://pic1.zhimg.com/80/v2-04e15b14964c9b61bffdfad42e907ffc_720w.webp)



先有一个World，它是系统和实体的集合，而实体就是一个ID，这个ID对应了组件的集合。组件用来存储游戏状态并且没有任何行为，系统拥有处理实体的行为但是没有状态。



## **详解实体、组件与系统**

### 1. 实体

实体只是一个概念上的定义，指的是**存在你游戏世界中的一个独特物体，是一系列组件的集合**。为了方便区分不同的实体，在代码层面上一般用一个ID来进行表示。所有组成这个实体的组件将会被这个ID标记，从而明确哪些组件属于该实体。由于其是一系列组件的集合，因此完全**可以在运行时动态地为实体增加一个新的组件或是将组件从实体中移除**。比如，玩家实体因为某些原因（可能陷入昏迷）而丧失了移动能力，只需简单地将移动组件从该实体身上移除，便可以达到无法移动的效果了。

**样例**：

- Player(Position, Sprite, Velocity, Health)
- Enemy(Position, Sprite, Velocity, Health, AI)
- Tree(Position, Sprite)

*注：括号前为实体名，括号内为该实体拥有的组件*

### 2. 组件

**一个组件是一堆数据的集合**，可以使用C语言中的结构体来进行实现。它**没有方法，即不存在任何的行为，只用来存储状态**。一个经典的实现是：**每一个组件都继承（或实现）同一个基类（或接口），通过这样的方法，我们能够非常方便地在运行时动态添加、识别、移除组件。**每一个组件的意义在于**描述实体的某一个特性**。例如，`PositionComponent`（位置组件），其拥有`x`、`y`两个数据，用来描述实体的位置信息，拥有`PositionComponent`的实体便可以说在游戏世界中拥有了一席之地。**当组件们单独存在的时候，实际上是没有什么意义的，但是当多个组件通过系统的方式组织在一起，才能发挥出真正的力量。**同时，我们还可以用空组件（不含任何数据的组件）对实体进行标记，从而在运行时动态地识别它。如，`EnemyComponent`这个组件可以不含有任何数据，拥有该组件的实体被标记为“敌人”。

根据实际开发需求，这里还会存在一种特殊的组件，名为 **Singleton Component （单例组件）**，顾名思义，单例组件在一个上下文中有且只有一个。

**样例**：

- PositionComponent(x, y)
- VelocityComponent(x, y)
- HealthComponent(value)
- PlayerComponent()
- EnemyComponent()

*注：括号前为组件名，括号内为该组件拥有的数据*

### 3. 系统

理解了实体和组件便会发现，至此还未曾提到过游戏逻辑相关的话题。**系统便是ECS架构中用来处理游戏逻辑的部分。**何为系统，**一个系统就是对拥有一个或多个相同组件的实体集合进行操作的工具，它只有行为，没有状态，即不应该存放任何数据。**举个例子，游戏中玩家要操作对应的角色进行移动，由上面两部分可知，角色是一个实体，其拥有位置和速度组件，那么怎么根据实体拥有的速度去刷新其位置呢，`MoveSystem`（移动系统）登场，它可以得到所有拥有位置和速度组件的实体集合，**遍历这个集合**，根据每一个实体拥有的速度值和物理引擎去计算该实体应该所处的位置，并刷新该实体位置组件的值，至此，完成了玩家操控的角色移动了。

注意，我强调了移动系统可以得到**所有**拥有位置和速度组件的实体集合，因为一个实体**同时**拥有位置和速度组件，我们便认为该实体拥有移动的能力，因此移动系统可以去刷新每一个符合要求的实体的位置。这样做的好处在于，当我们玩家操控的角色因为某种原因不能移动时，我们只需要将速度组件从该实体中移除，移动系统就得不到角色的引用了，同样的，如果我们希望游戏场景中的某一个物件动起来，只需要为其添加一个速度组件就万事大吉。

**一个系统关心实体拥有哪些组件是由我们决定的，通过一些手段，我们可以在系统中很快地得到对应实体集合。**

上文提到的 **Singleton Component （单例组件）** ，明白了系统的概念更容易说明，还是玩家操作角色的例子，该实体速度组件的值从何而来，一般情况下是根据玩家的操作输入去赋予对应的数值。这里就涉及到一个新组件`InputComponent`（输入组件）和一个新系统`ChangePlayerVelocitySystem`（改变玩家速度系统），改变玩家速度系统会根据输入组件的值去改变玩家速度，假设还有一个系统`FireSystem`（开火系统），它会根据玩家是否按下开火键进行开火操作，那么就有 2 个系统同时依赖输入组件，真实游戏情况可能比这还要复杂，有无数个系统都要依赖于输入组件，同时拥有输入组件的实体在游戏中仅仅需要有一个，每帧去刷新它的值就可以了，这时很容易让人想到单例模式（便捷地访问、只有一个引用），同样的，**单例组件也是指整个游戏世界中有且只有一个实体拥有该组件，并且希望各系统能够便捷的访问到它**，经过一些处理，在任何系统中都能通过类似`world->GetSingletonInput()`的方法来获得该组件引用。

系统这里比较麻烦，还存在一个常见问题：由于代码逻辑分布于各个系统中，各个系统之间为了解耦又不能互相访问，那么如果有**多个系统希望运行同样的逻辑**，该如何解决，总不能把代码复制 N 份，放到各个系统之中。**UtilityFunction**（实用函数） 便是用来解决这一问题的，它将被多个系统调用的方法单独提取出来，放到统一的地方，各个系统通过 UtilityFunction 调用想执行的方法，同系统一样， UtilityFunction 中**不能存放状态**，它应该是**拥有各个方法的纯净集合**。

**样例**：

- MoveSystem(Position, Velocity)
- RenderSystem(Position, Sprite)

*注：括号前为系统名，括号内为该系统关心的组件集合*



## 没有什么是完美的

虽然 ECS 架构可以让维护和扩展的成本降低——必要的时候你只要给对象增加组件、为游戏逻辑增加系统就可以扩展了。这种设计降低了耦合，也提高了可复用性。

但是很明显的，ECS 带来了两个缺陷：

### 1. 数据共享

比如渲染组件需要对象的位置信息、碰撞组件也要对象的位置信息，那么我们怎么解决这里的数据共享问题？

一种解决方法是**把位置信息作为组件抽象出来**，但是这样带来的问题就是效率会降低，在处理渲染和碰撞的时候都会再去存取一次位置信息。

另一种解决方法是**使用引用（比如使用C++中的智能指针）在构建对象的时候分配同一位置对象，然后传递引用给其他组件**。

### 2. 遍历次数

当游戏中数量巨大并且系统数量也增加时，很明显整个算法的复杂度将不低于O(n*m)，遍历对象的次数将变得巨大。

比如碰撞检测系统若是两两对象进行逻辑处理那么速度上的损失将是十分巨大的。

这里的一种解决方法是**分集合处理**，不再赘述。



## EnTT

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

`entt::system<void(entt::registry&)>`是一个函数对象类型，它表示一个可以接受 EnTT 实体注册表作为参数的系统。其中，`void`表示该系统不返回任何值。在使用时，可以将一个函数指针或者lambda表达式转换为这个类型，并传递给 EnTT 的`basic_registry::on_update`函数，使其在每个帧更新时被调用。用于注册一个每帧更新时调用`update_position`函数的系统。这样，在每个帧更新时，`update_position`函数将被调用，并遍历所有具有`Position`组件的实体，更新它们的位置。

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

