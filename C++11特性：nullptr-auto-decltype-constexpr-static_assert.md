# C++11 特性：nullptr / auto / decltype / constexpr / static_assert

## nullptr-空指针

C++98/03 标准中，将一个指针初始化为空指针的方式有 2 种：

```c++
int *p = 0;
int *p = NULL;
```

可以看到，我们可以将指针明确指向 0（0x0000 0000）这个内存空间。一方面，明确指针的指向可以避免其成为野指针；另一方面，大多数操作系统都不允许用户对地址为 0 的内存空间执行写操作，若用户在程序中尝试修改其内容，则程序运行会直接报错。

相比第一种方式，我们更习惯将指针初始化为 NULL。值得一提的是，NULL 并不是 C++ 的关键字，它是 C++ 为我们事先定义好的一个宏，并且它的值往往就是字面量 0**（#define NULL 0）**。

C++ 中将 NULL 定义为字面常量 0，虽然能满足大部分场景的需要，但个别情况下，它会导致程序的运行和我们的预期不符。例如：

```c++
#include <iostream>
using namespace std;

void isnull(void *c){
    cout << "void*c" << endl;
}
void isnull(int n){
    cout << "int n" << endl;
}
int main() {
    isnull(0);
    isnull(NULL);
    return 0;
}
```

程序执行结果为：

```text
int n
int n
```

对于 isnull(0) 来说，显然它真正调用的是参数为整形的 isnull() 函数；而对于 isnull(NULL)，我们期望它实际调用的是参数为 void \*c 的 isnull() 函数，但观察程序的执行结果不难看出，并不符合我们的预期。

C++ 98/03 标准中，如果我们想令 isnull(NULL) 实际调用的是 isnull(void\* c)，就需要对 NULL（或者 0）进行强制类型转换：

```c++
isnull( (void*)NULL );
isnull( (void*)0 );
```

如此，才会成功调用我们预期的函数。

由于 C++ 98 标准使用期间，NULL 已经得到了广泛的应用，出于兼容性的考虑，C++11 标准并没有对 NULL 的宏定义做任何修改。为了修正 C++ 存在的这一 BUG，C++ 标准委员会最终决定另其炉灶，在 C++11 标准中引入一个新关键字，即 nullptr 。

**nullptr 是 nullptr_t 类型的右值常量，专用于初始化空类型指针。**nullptr_t 是 C++11 新增加的数据类型，可称为“指针空值类型”。也就是说，nullpter 仅是该类型的一个实例对象（已经定义好，可以直接使用），如果需要我们完全定义出多个同 nullptr 完全一样的实例对象。

值得一提的是，**nullptr 可以被隐式转换成任意的指针类型**。举个例子：

```c++
int * a1 = nullptr;
char * a2 = nullptr;
double * a3 = nullptr;
```

显然，不同类型的指针变量都可以使用 nullptr 来初始化，编译器分别将 nullptr 隐式转换成 int*、char* 以及 double 等的指针类型。

另外，通过将指针初始化为 nullptr，可以很好地解决 NULL 遗留的问题（**NULL 表示空指针在 C++中具有二义性**），由于 nullptr 无法隐式转换为整形，而可以隐式匹配指针类型，因此执行结果和我们的预期相符。

**总之，在 C++11 标准下，最好使用 nullptr，同时尽量避免使用 NULL 。**

## auto - 自动类型推导

编程时常常需要把表达式的值赋给变量，这就要求在声明变量的时候清楚地知道表达式的类型。然而要做到这一点并非那么容易，有时甚至根本做不到。为了解决这个问题，c++11 新标准引入了 auto 类型说明符，用它就能让编译器替我们去分析表达式所属的类型。和原来那些只对应一种特定类型的说明符（比如 double）不同，**auto 让编译器通过初始值来推算变量的类型**。显然，**auto 定义的变量必须有初始值**：

```c++
// 由val1和val2相加的结果可以推断出item的类型
auto item = va11 + va12; // item初始化为val1和val2相加的结果
```

使用 auto 也能在一条语句中声明多个变量。因为一条声明语句只能有一个基本数据类型，所以该语句中所有变量的初始基本数据类型都必须一样：

```c++
auto i = 0, *p = &i;    // 正确：i是整数，p是整型指针
auto sz = 0, pi = 3.14; // 错误：sz和pi的类型不一致
```

### 复合类型、常量和 auto

编译器推断出来的 auto 类型有时候和初始值的类型并不完全一样，编译器会适当地改变结果类型使其更符合初始化规则。

首先，正如我们所熟知的，使用引用其实是使用引用的对象，特别是当引用被用作初始值时，真正参与初始化的其实是引用对象的值。此时**编译器以引用对象的类型作为 auto 的类型**：

```c++
int i = 0, &r = i;
auto a = r;            // a是一个整数（r是i的别名，而i是一个整数）
```

其次，**auto 一般会忽略掉顶层 const**（顶层 const 表示指针本身是个常量），同时底层 const（底层 const 表示指针所指的对象是一个常量）则会保留下来，比如当初始值是一个指向常量的指针时：

```c++
const int ci = i, &cr = ci;
auto b = ci;  // b 是一个整数（ci的顶层const特性被忽略掉了）
auto c = cr;  // c 是一个整数（cr是ci的别名，ci本身是一个顶层const）
auto d = &i;  // d 是一个整型指针（整数的地址就是指向整数的指针）
auto e = &ci; // e 是一个指向整数常量的指针（对常量对象取地址是一种底层const）
```

**如果希望推断出的 auto 类型是一个顶层 const，需要明确指出**：

```c++
const auto f = ci; // ci的推演类型是int，f是const int
```

还可以将引用的类型设为 auto，此时原来的初始化规则仍然适用：

```c++
auto &g = ci;   // g是一个整型常量引用，绑定到ci
auto &h = 42;   // 错误：不能为非常量引用绑定字面值
const auto &j = 42; // 正确：可以为常量引用绑定字面值
```

**设置一个类型为 auto 的引用时，初始值中的顶层常量属性仍然保留。**和往常一样，如果我们给初始值绑定一个引用（用常量引用初始化常量），则此时的常量就不是顶层常量了（因为用于声明引用的 const 都是底层 const）。

要在一条语句中定义多个变量，切记，**符号 & 和 \* 只从属于某个声明符，而非基本数据类型的一部分**，因此初始值必须是同一种类型：

```c++
auto k = ci, &l = i; // k是整数，1是整型引用
auto &m = ci, *p = &ci; // m是对整型常量的引用，p是指向整型常量的指针
auto &n = i, *p2 = &ci; // 错误：i的类型是int而&ci的类型是const int
```

## decltype - 编译期表达式类型推导

有时会遇到这种情况：**希望从表达式的类型推断出要定义的变量的类型，但是不想用该表达式的值初始化变量**。为了满足这一要求，C++11 新标准引入了第二种类型说明符 decltype，它的作用是选择并返回操作数的数据类型。在此过程中，编译器分析表达式并得到它的类型，却不实际计算表达式的值：

```c++
decltype(f()) sum = x; // sum的类型就是函数f的返回类型
```

编译器并不实际调用函数 f，而是使用当调用发生时 f 的返回值类型作为 sum 的类型。换句话说，编译器为 sum 指定的类型是什么呢？就是假如 f 被调用的话将会返回的那个类型。

decltype 处理顶层 const 和引用的方式与 auto 有些许不同。**如果 decltype 使用的表达式是一个变量，则 decltype 返回该变量的类型（包括顶层 const 和引用在内）**：

```c++
const int ci = 0, &cj = ci;
decltype(ci) x = 0; // x的类型是const int
decltype(cj) y = x; // y的类型是const int&, y绑定到变量x
decltype(cj) z;  // 错误：z是一个引用，必须初始化
```

因为 cj 是一个引用，decltype(cj) 的结果就是引用类型，因此作为引用的 z 必须被初始化。

需要指出的是，**引用从来都作为其所指对象的同义词出现，只有用在 decltype 处是一个例外**。

### decltype 和引用

如果 decltype 使用的表达式不是一个变量，则 decltype 返回表达式结果对应的类型。有些表达式将向 decltype 返回一个引用类型。一般来说当这种情况发生时，意味着该表达式的结果对象能作为一条赋值语句的左值：

```c++
// decltype的结果可以是引用类型
int i = 42, *p = &i, &r = i;
decltype(r + 0) b; // 正确：加法的结果是int，因此b是一个（未初始化的）int
decltype(*p) c;    // 错误：c是int&，必须初始化
```

因为 r 是一个引用，因此的结果是引用类型。如果想让结果类型是 r 所指的类型，可以把 r 作为表达式的一部分，如 r+0，显然这个表达式的结果将是一个具体值而非一个引用。

另一方面，**如果表达式的内容是解引用操作，则 decltype 将得到引用类型**。正如我们所熟悉的那样，解引用指针可以得到指针所指的对象，而且还能给这个对象赋值。因此，decltype(\*p)的结果类型就是 int&，而非 int。

decltype 和 auto 的另一处重要区别是，decltype 的结果类型与表达式形式密切相关。有一种情况需要特别注意：对于 decltype 所用的表达式来说，如果变量名加上了一对括号，则得到的类型与不加括号时会有不同。**如果 decltype 使用的是一个不加括号的变量得到的结果就是该变量的类型：如果给变量加上了一层或多层括号，编译器就会把它当成是一个表达式。**变量是一种可以作为赋值语句左值的特殊表达式，所以这样的 decltype 就会得到引用类型：

```c++
// decltype的表达式如果是加上了括号的变量，结果将是引用
decltype((i)) d; // 错误：d是int&，必须初始化
decltype(i) e;   // 正确：e是一个（未初始化的）int
```

**切记：decltype（(variable)）（注意是双层括号）的结果永远是引用，而 decltype(variable)结果只有当 variable 本身就是一个引用时才是引用。**

### decltype(auto)

**decltype(auto)相当于直接对表达式应用 decltype**，这样就不用把后面的表达式写两遍了（可以把 decltype(auto)中的 auto 理解成一个占位符）。

比如

```cpp
int a = 1;
decltype(auto) b = a;
decltype(auto) c = (a);
```

可以看成

```cpp
int a = 1;
decltype(a) b = a;
decltype((a)) c = (a);

int b = a;
int & c = (a);
```

与 auto 的区别：auto 不一定能推断出原有类型，而 decltype(auto)可以**(实际使用了 decltype 的推断规则）**。

**decltype 会保留 cv 限定符**（const， volatile），而 auto 有可能会去掉 cv 限定符。

```cpp
int i = 0;
const int& a = i;
auto b = a; // b为int
auto &c = a; // c为const int&
decltype(auto) d = a; // d为const int&
```

在编程实践中，如果能确定返回类型，可以直接用 auto 写

```cpp
auto& f(const int& i)
{
    return i;
}
```

泛型编程里一般返回类型未知，decltype(auto)就更好用，比如从容器的一个元素推导返回类型

```cpp
template<typename Cont, typename N>
decltype(auto) f(Cont&& c, N n)
{
    return std::forward<Cont>(c)[n];
}
```

## constexpr - 常量表达式

**常量表达式(constexpression）是指值不会改变并且在编译过程就能得到计算结果的表达式。**显然，字面值属于常量表达式，用常量表达式初始化的 const 对象也是常量表达式。**一个对象（或表达式）是不是常量表达式由它的数据类型和初始值共同决定**，例如：

```c++
const int max_file = 20;   // max_files是常量表达式
const int limit = max_files + 1; // limit就是常量表达式
int staff_size = 27;    // staff_size不是常量表达式
const int sz = get_size();   // sz不是常量表达式
```

尽管 staffsize 的初始值是个字面值常量，但由于它的数据类型只是一个普通 int 而非 const int，所以它不属于常量表达式。另一方面，尽管 sz 本身是一个常量，但它的具体值直到运行时才能获取到，所以也不是常量表达式。

### constexpr 变量

在一个复杂系统中，很难（几乎肯定不能）分辨一个初始值到底是不是常量表达式。当然可以定义一个 const 变量并把它的初始值设为我们认为的某个常量表达式，但在实际使用时，尽管要求如此却常常发现初始值并非常量表达式的情况。可以这么说，在此种情况下，对象的定义和使用根本就是两回事儿。

C++11 新标准规定，允许将变量声明为 constexpr 类型以便由编译器来验证变量的值是否是一个常量表达式。**声明为 constexpr 的变量一定是一个常量，而且必须用常量表达式初始化**：

```c++
constexpr int mf = 20;    // 20是常量表达式
constexpr int limit = mf + 1; // mf+1是常量表达式
constexpr int sz = size();  // 只有当size是一个constexpr函数时,才是一条正确的声明语句
```

尽管不能使用普通函数作为 constexpr 变量的初始值，但是新标准允许定义一种特殊的 constexpr 函数。这种函数应该足够简单以使得编译时就可以计算其结果，这样就能用 constexpr 函数去初始化 constexpr 变量了。

**一般来说，如果你认定变量是一个常量表达式，那就把它声明成 constexpr 类型。**

### 字面值类型

常量表达式的值需要**在编译时就得到计算**，因此对声明 constexpr 时用到的类型必须有所限制。因为这些类型一般比较简单，值也显而易见、容易得到，就把它们称为“字面值类型”（literal type)。

到目前为止接触过的数据类型中，**算术类型、引用和指针都属于字面值类型**。自定义类、库、string 类型则不属于字面值类型，也就不能被定义成 constexpr。

尽管指针和引用都能定义成 constexpr，但它们的初始值却受到严格限制。一个 constexpr 指针的初始值必须是 nullptr 或者 0，或者是存储于某个**固定地址**中的对象。

函数体内定义的变量一般来说并非存放在固定地址中，因此 constexpr 指针不能指向这样的变量。相反的，**定义于所有函数体之外的对象其地址固定不变，能用来初始化 constexpr 指针**。允许函数定义一类有效范围超出函数本身的变量，这类变量和定义在函数体之外的变量一样也有固定地址。因此，constexpr 引用能绑定到这样的变量上，constexpr 指针也能指向这样的变量。

### 指针和 constexpr

必须明确一点，在 constexpr 声明中如果定义了一个指针，**限定符 constexpr 仅对指针有效，与指针所指的对象无关**：

```c++
const int *p = nullptr;  // p是一个指向整型常量的指针
constexpr int *q = nullptr; // q是一个指向整数的常量指针
```

p 和 q 的类型相差甚远，p 是一个指向常量的指针，而 q 是一个常量指针，其中的关腱在于**constexpr 把它所定义的对象置为了顶层 const**。与其他常量指针类似，constexpr 指针既可以指向常量也可以指向一个非常量：

```c++
constexpr int *np = nullptr; // np是一个指向整数的常量指针，其值为空
int j = 0;
const int i = 42;    // i的类型是整型常量
// i和j都必须定义在函数体之外
constexpr const int *p = &i; // p是常量指针．指向整型常量i
constexpr int *p1 = &j;   // pl是常量指针，指向整数j
```

### constexpr 函数

constexpr 函数主要用于**编译期计算**，尽管编译期运算会延长编译时间，但有时候会用它来加快程序的运行速度，提高性能。

```c++
#include <iostream>
using namespace std;

// C++ 98/03
template<int N> struct Factorial
{
    const static int value = N * Factorial<N - 1>::value;
};
template<> struct Factorial<0>
{
    const static int value = 1;
};

// C++ 11
constexpr int factorial(int n)
{
    return n == 0 ? 1 : n * factorial(n - 1);
}

// C++ 14 以后
constexpr int factorial2(int n)
{
    int result = 1;
    for (int i = 1; i <= n; ++i)
        result *= i;
    return result;
}

int main()
{
    static_assert(Factorial<3>::value == 6, "error");
    static_assert(factorial(3) == 6, "error");
    static_assert(factorial2(3) == 6, "error");
    int n = 3;
    cout << factorial(n) << factorial2(n) << endl; //66
}
```

以上代码演示了如何在编译期计算 3 的阶乘。

- 在 C++11 之前，在编译期进行数值计算必须**使用模板元编程**技巧。具体来说我们通常需要定义一个内含编译期常量 value 的类模板（也称作元函数）。这个类模板的定义至少需要分成两部分，分别用于处理一般情况和特殊情况。

    代码示例中 Factorial 元函数的定义分为两部分：

    当模板参数大于 0 时，利用公式 N!=N\*(N-1)! 递归调用自身来计算 value 的值。

    当模板参数为 0 时，将 value 设为 1 这个特殊情况下的值。

- 在 C++11 之后，编译期的数值计算可以通过**使用 constexpr 声明并定义编译期函数**来进行。相对于模板元编程，使用 constexpr 函数更贴近普通的 C++程序，计算过程显得更为直接，意图也更明显。但在 C++11 中 constexpr 函数所受到的限制较多。如 factorial 函数所示，使用 C++11 在编译期计算阶乘仍然需要利用递归技巧。

    > C++11 标准下，constexpr 函数的使用限制：
    >
    > 1. 整个函数的函数体中，除了可以包含 using 指令、typedef 语句以及 static_assert 断言外，只能包含一条 return 返回语句。
    > 2. 该函数必须有返回值，即函数的返回值类型不能是 void。
    > 3. 函数在使用之前，必须有对应的定义语句。我们知道，函数的使用分为“声明”和“定义”两部分，普通的函数调用只需要提前写好该函数的声明部分即可（函数的定义部分可以放在调用位置之后甚至其它文件中），但常量表达式函数在使用前，必须要有该函数的定义。
    > 4. return 返回的表达式必须是常量表达式。

- 在 C++14 之后，**逐步解除了对 constexpr 函数的一些限制**。如 factorial2 函数所示，使用 C++14 在编译期计算阶乘只需利用 for 语句进行常规计算即可。

虽说 constexpr 函数所定义的是编译期的函数，但实际上在运行期 constexpr 函数也能被调用。事实上，如果使用编译期常量参数调用 constexpr 函数，我们就能够在编译期得到运算结果；而如果使用运行期变量参数调用 constexpr 函数，那么在运行期我们同样也能得到运算结果。

最后一行代码`cout << factorial(n) << factorial2(n) << endl;` 所演示的是在运行期使用变量 n 调用 constexpr 函数的结果。

准确的说，constexpr 函数是一种在编译期和运行期都能被调用并执行的函数。出于 constexpr 函数的这个特点，**在 C++11 之后进行数值计算时，无论在编译期还是运行期我们都可以统一用一套代码来实现。编译期和运行期在数值计算这点上得到了部分统一。**

## static_assert - 静态断言

assert 的作用是先计算表达式 expression ，如果其值为假（即为 0），那么它先向 stderr 打印一条出错信息，然后通过调用 abort 来终止程序运行。assert 分为动态断言和静态断言两种。

### 动态断言

assert 是运行期断言，它用来发现运行期间的错误，不能提前到编译期发现错误，既然是运行期检查，对性能当然是有影响的，所以经常在发行版本中，assert 都会被关掉；函数原型如下：

```c++
#include <assert.h>
void assert( int expression );
```

### 静态断言

c++0x 引入了 static_assert 关键字，用来实现编译期间的断言，叫静态断言。使用方法如下：

```c++
static_assert(static_expression, error_message);
```

如果第一个参数常量表达式的值为 false，会产生一条编译错误，错误位置就是该 static_assert 语句所在行，第二个参数就是错误提示字符串。然后通过调用 abort 来终止程序运行。

使用 static_assert，我们可以在编译期间发现更多的错误，用编译器来强制保证一些契约，并帮助我们改善编译信息的可读性，尤其是用于模板的时候。

static_assert 可以用在全局作用域中，命名空间中，类作用域中，函数作用域中，**几乎可以不受限制的使用**。

编译器在遇到一个 static_assert 语句时，通常立刻将其第一个参数作为常量表达式进行演算，但如果该常量表达式依赖于某些模板参数，则延迟到模板实例化时再进行演算，这就让检查模板参数成为了可能。

性能方面，由于是 static_assert 编译期间断言，不生成目标代码，因此 static_assert 不会造成任何运行期性能损失。

简单例子：

```c++
static_assert(sizeof(void *)==4,"64位系统上不支持！");
```

该 static_assert 用来确保编译仅在 32 位的平台上进行，不支持 64 位的平台，该语句可以放在文件的开头处，这样可以尽早检查，以节省失败情况下的编译时间。

```c++
#include <cassert>
#include <cstring>
using namespace std;

template <typename T, typename U> int bit_copy(T& a, U& b){
    assert(sizeof(b) == sizeof(a));
　　 static_assert(sizeof(b) == sizeof(a), "template parameter size no equal!");
    memcpy(&a,&b,sizeof(b));
};

int main()
{
    int aaa = 0x2468;
    double bbb;
    bit_copy(aaa, bbb);
    getchar();
    return 0;
}
```

这里使用 assert 运行时断言，但如果 bit_copy 不被调用，我们将无法触发该断言，实际上正确产生断言的时机应该在模版实例化时，即编译时期。使用 static_assert 替换 assert 再次编译，即可获得一个错误并显示我们指定的错误信息。

**注意：static_assert 的断言表达式的结果必须是在编译时期可以计算的表达式,即必须是常量表达式。如果使用变量，则会导致错误。**

静态断言在编译时进行处理，不会产生任何运行时刻空间和时间上的开销，这就使得它比 assert 宏具有更好的效率。另外比较重要的一个特性是如果断言失败，它会产生有意义且充分的诊断信息，帮助程序员快速解决问题。
