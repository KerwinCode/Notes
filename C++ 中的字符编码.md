# C++ 中的字符编码

## 字符集与字符编码

**字符（Character）** ：对文字和符号的总称，例如汉字、拉丁字母、emoji 都是字符。在计算机中，一个字符由 2 部分组成：字符表达的含义（表现）与字符的编码。

**字符编码（ Character Encoding）**：“编码” 这个词经常出现，以下列举出 “编码” 常见的 3 层解释，希望能帮助你在阅读文章时快速理解作者的意思。

- **含义 1 - 作为动词：** 表示把一个字符转换为一个二进制机器数的过程，这个机器数才是字符在计算机中真实存储/传输的格式。例如把 A 转换为 65(ASCII) 的动作，就是一个编码动作；
- **含义 2 - 作为名词：** 表示经过编码动作后得到的那个机器数，对于 A 来说，65(ASCII) 就是 A 的编码(值)，有时会称为编号；
- **含义 3 - 作为名词：** 表示把字符转换为机器数的编码方案，例如 ASCII 编码、GBK 编码、UTF-8 编码。本文下述部分取此意。

**字符集（Character Set）** ：字符与字符编码组成的系统。

## 常见字符编码

### ASCII

上个世纪 60 年代，美国制定了一套字符编码，对英语字符与二进制位之间的关系，做了统一规定。这被称为 ASCII 码，一直沿用至今。

ASCII 码一共规定了 128 个字符的编码，比如空格`SPACE`是 32（二进制`00100000`），大写的字母`A`是 65（二进制`01000001`）。这 128 个符号（包括 32 个不能打印出来的控制符号），只占用了一个字节的后面 7 位，最前面的一位统一规定为`0`。

### ANSI

ANSI 编码是由美国国家标准协会 (ANSI: American National Standards Institute) 制定的标准化的数字或字母标识符。ANSI 编码是一种对 ASCII 码的拓展：ANSI 编码用 `0x00~0x7f`（即十进制下的 0 到 127）范围的 1 个字节来表示 1 个英文字符，超出一个字节的 0x80~0xFFFF 范围来表示其他语言的其他字符。也就是说，ANSI 码仅在前 128（0-127）个与 ASCII 码相同，之后的字符全是某个国家语言的所有字符。值得注意的是，两个字节最多可以存储的字符数目是 2 的 16 次方，即 65536 个字符，这对于一个语言的字符来说，绝对够了。还有 ANSI 编码其实包括很多编码：中国制定了 GB2312 编码，用来把中文编进去另外，日本把日文编到 Shift_JIS 里，韩国把韩文编到 Euc-kr 里，各国有各国的标准。受制于当时的条件，不同语言之间的 ANSI 码之间不能互相转换，这就会导致在多语言混合的文本中会有乱码。

### Unicode

为了解决不同国家 ANSI 编码的冲突问题，Unicode 应运而生：如果有一种编码，将世界上所有的符号都纳入其中。每一个符号都给予一个独一无二的编码，那么乱码问题就会消失。这就是 Unicode，就像它的名字都表示的，这是一种所有符号的编码。

Unicode 当然是一个很大的集合，现在的规模可以容纳 100 多万个符号。每个符号的编码都不一样，比如，`U+0639`表示阿拉伯字母`Ain`，`U+0041`表示英语的大写字母`A`，`U+4E25`表示汉字`严`。具体的符号对应表，可以查询[unicode.org](http://www.unicode.org/)，或者专门的[汉字对应表](http://www.chi2ko.com/tool/CJK.htm)。

### UTF-8 / UTF-8 BOM / UTF-16

需要注意的是，Unicode 只是一个符号集，它只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储。UTF-8 就是在互联网上使用最广的一种 Unicode 的实现方式。**这里的关系是，UTF-8 是 Unicode 的实现方式之一。**

UTF-8 最大的一个特点，就是它是一种变长的编码方式。它可以使用 1~4 个字节表示一个符号，根据不同的符号而变化字节长度。

UTF-8 的编码规则很简单，只有二条：

1）对于单字节的符号，字节的第一位设为`0`，后面 7 位为这个符号的 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的。

2）对于`n`字节的符号（`n > 1`），第一个字节的前`n`位都设为`1`，第`n + 1`位设为`0`，后面字节的前两位一律设为`10`。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码。

下表总结了编码规则，字母`x`表示可用编码的位。

> ```text
> Unicode符号范围      |             UTF-8编码方式
>    (十六进制)        |              （二进制）
> --------------------+---------------------------------------------
> 0000 0000-0000 007F | 0xxxxxxx
> 0000 0080-0000 07FF | 110xxxxx 10xxxxxx
> 0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx
> 0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
> ```

跟据上表，解读 UTF-8 编码非常简单。如果一个字节的第一位是`0`，则这个字节单独就是一个字符；如果第一位是`1`，则连续有多少个`1`，就表示当前字符占用多少个字节。

下面，还是以汉字`严`为例，演示如何实现 UTF-8 编码。

`严`的 Unicode 是`4E25`（`100111000100101`），根据上表，可以发现`4E25`处在第三行的范围内（`0000 0800 - 0000 FFFF`），因此`严`的 UTF-8 编码需要三个字节，即格式是`1110xxxx 10xxxxxx 10xxxxxx`。然后，从`严`的最后一个二进制位开始，依次从后向前填入格式中的`x`，多出的位补`0`。这样就得到了，`严`的 UTF-8 编码是`11100100 10111000 10100101`，转换成十六进制就是`E4B8A5`。

**UTF-8 BOM** 指在 UTF-8 文件中放置 BOM，主要是微软的习惯。BOM（byte order mark）是为 UTF-16 和 UTF-32 准备的，用于标记字节序。微软在 UTF-8 中使用 BOM 是因为这样可以把 UTF-8 和 ASCII 等编码明确区分开，但这样的文件在 Windows 之外的操作系统里会带来问题。「UTF-8」和「带 BOM 的 UTF-8」的区别就是有没有 BOM。即文件开头有没有 `U+FEFF`。UTF-8 的网页代码不应使用 BOM，否则常常会出错。

记事本里的 Unicode 和 Unicode (Big Endian) 分别指的是 UTF-16 LE、UTF-16 BE 这两种编码方案。通常 UTF-16 LE 的文本开头会有 0xFF 0xFE 的标识，UTF-16 BE 的文本开头会有 0xFE 0xFF 的标识；UTF-8 没有高低字节谁先谁后的问题（可以认为是强制 BE），所以通常也没有这样的标识。所以这三种 Unicode 方案都是跨平台兼容性非常好的编码方案。如果非得挑其中一个的话，选 UTF-8。因为有些程序员在解析字符串的时候不会考虑编码问题，会按纯 ASCII 的方式来解析字符串。而 **UTF-8 是完美兼容纯 ASCII 的，在这样的程序上解析 UTF-8 编码的字符串是完全无痛的**。对于 UTF-16 而言（无论 BE、LE），按纯 ASCII 解析有时会遇到问题。

### GB2312 / GBK / GB18030

GB2312：这是中国最早制定的汉字编码标准，旨在解决简体中文在计算机系统中的存储和处理问题。它采用了区位码的方式，每个汉字由一个区号和一个位号组成，总共可以表示 6763 个汉字。然而，GB2312 存在明显的局限性，例如缺少许多常用汉字和繁体字，也不支持少数民族文字。

GBK：为了弥补 GB2312 的不足，GBK 在 1995 年发布，扩展了字符集以包含更多的汉字和符号。GBK 不仅包含了 GB2312 的所有字符，还增加了对繁体字、日文假名等的支持。GBK 的编码范围更大，能够表示 21886 个字符。尽管 GBK 解决了部分 GB2312 的问题，但它与 ASCII 编码存在冲突，可能导致数据解析错误。

GB18030：GB18030 是在 GBK 的基础上进一步扩展的编码标准，于 2000 年首次发布，并在后续版本中不断扩充。GB18030 采用了四字节编码结构，能够支持包括 CJK 扩展汉字、藏文、蒙文在内的多种语言文字。GB18030 是目前中国最全面的汉字编码标准，具有强制性地位，广泛应用于国内的政务和金融系统中。

| 特性       | GB2312 (1980)                         | GBK (1995)                                                                                                                                            | GB18030 (2000/2005/2022)                                           |
| ---------- | ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 字符收录量 | 6763 个汉字 + 682 个符号              | 21886 个字符（含 21003 汉字）                                                                                                                         | 87887+字符（覆盖汉字、少数民族文字、表情符号等）                   |
| 编码结构   | 双字节固定长度                        | 双字节固定长度                                                                                                                                        | 变长编码（1/2/4 字节）                                             |
| 字节范围   | 高字节 0xB0-0xF7</br>低字节 0xA1-0xFE | 高字节 0x81-0xFE</br>低字节 0x40-0xFE（剔除 0x7F）                                                                                                    | 单字节 0x00-0x7F</br>双字节同 GBK</br>四字节 0x81-0xFE + 0x30-0x39 |
| 兼容性     | 兼容 ASCII                            | 兼容 GB2312 与 ASCII，但 GBK 低字节范围（0x40-0x7E）与 ASCII 可打印字符（如 \| 0x7C）重叠，会导致冲突，所以 **GBK 环境需规避 0x40-0x7E 符号独立使用** | 完全兼容 GBK/GB2312，支持 Unicode 映射                             |
| 强制效力   | 旧国标（2017 年转为推荐）             | 非国家标准（技术规范）                                                                                                                                | 唯一强制性国家标准                                                 |

## C++ 翻译阶段中的编码转换

根据 cppreference 文档中翻译阶段的：[阶段 5：确定字符串字面量的公共编码](https://cppreference.cn/w/c/language/translation_phases)；我们可以知道：**字符字面量、字符串字面量中的所有字符都要从源字符集转换为执行字符集**。

也就是说编译器在编译代码的时候会先进行一个编码转换，编译与运行过程会涉及：

- 源字符集（即源代码文件的编码方式，但并非代码文件本身的编码）。
- 执行字符集（编译器用来编译代码的字符编码）。
- 终端字符集（终端使用何种编码）。

你可能认为“_源字符集_” 就是我们的文件编码，但是其实并非如此，你只要没有显式设置，那么就是遵循系统的默认区域设置。

> [!TIP]
>
> Linux 可以使用 `locale` 命令查看，Windows （PowerShell）使用 `[System.Text.Encoding]::Default`。
>
> 如果是普通的中国大陆区域的 windows 系统，会看到 “**简体中文（GB2312）**”，设置了全局 UTF-8 则是：**Unicode (UTF-8)**

那么执行字符集呢？还是那句话，只要没有显式设置，就会遵循系统的默认区域设置。

这意味着，若`.\test.cpp`文件编码为 GB18030，系统设置了全局 UTF-8，则在翻译阶段（阶段五）中，**编译器并不会对我们代码中的 GB18030 字符串字面量进行任何转换**。最终，编译出的可执行程序会以 `UTF-8` 形式强行解码 GB18030 编码的字符串，这就导致了输出时出现乱码。

那么如何解决这个问题呢？很简单，我们指明它的**源字符集**为 GB18030 不就是了？

```bash
g++ .\test.cpp -o test -finput-charset=gb18030
```

执行字符集无需设置，按照区域默认，就是 `utf-8`。那么在翻译阶段（阶段五）中，编译器会将我们代码中的 GB18030 字符串字面量转换为 UTF-8。最终编译出的可执行程序就能正常输出。

其实大家还需要考虑终端的编码，即使程序完全正确，终端的编码不对依然可能有乱码，这就是环境问题，而非程序本身了。另外，如果想要显式指明执行字符集，可以使用选项：`-fexec-charset`，也就是：

```bash
g++ .\test.cpp -o test -finput-charset=gb18030 -fexec-charset=utf-8
```

以上提到的选项 gcc 与 clang 都可以使用。

Visual Studio 2015 Update 2 及之后版本分别用 [`/source-charset`](https://learn.microsoft.com/zh-cn/cpp/build/reference/source-charset-set-source-character-set?view=msvc-170) 和 [`/execution-charset`](https://learn.microsoft.com/zh-cn/cpp/build/reference/execution-charset-set-execution-character-set?view=msvc-170) 指定源字符集和执行字符集。并且还增加了一个新的选项：[**`/utf-8`**](https://learn.microsoft.com/zh-cn/cpp/build/reference/utf-8-set-source-and-executable-character-sets-to-utf-8?view=msvc-170)（将源字符集和执行字符集设置为 UTF-8）。更多详情也可以参阅微软文档。

此外，如果你编辑器中你看到的代码没有乱码，但实际编译后的程序会出现了乱码。这也可能是因为编辑器可能会根据系统区域设置使用的编码查看文件，而不是 UTF-8。

在此还需要特别强调两点：

1. **不要以为代码文件是 UTF-8 编码，编译器就自动认为源字符集是 UTF-8，编译器不会自动检测，源字符集和执行字符集缺省时，默认为系统区域设置**。
2. **不要以为代码文件是 UTF-8 编码，就没有非 UTF-8 的字符**。某些 IDE 的编码处理可能会有问题，导致你无法直接察觉文件中的字符集问题。**不要使用记事本编辑代码**，建议使用 **VS Code** 等支持多种字符集的编辑器来进行查看，确保编码一致。

## C++20 对 UTF-8 的支持

C++20 显著增强了对 UTF-8 的原生支持，核心改进围绕**类型安全**、**专用容器**和**跨平台一致性**展开，解决了早期版本中的编码混淆问题。

### 1. `char8_t` 类型

- **作用**：明确标识 UTF-8 编码的字符，本质是无符号 8 位整数 (`unsigned char`)，但编译器赋予其语义标签。
- **类型安全**：严格区分 UTF-8 字符串与其他编码（如 ASCII 或本地编码），避免误用：

    ```cpp
    char8_t c = u8'中';        // C++20：合法，明确 UTF-8
    // char* p = u8"文本";     // C++20 错误！需用 char8_t* 接收
    ```

- **兼容性**：与旧代码交互时需显式转换：

    ```cpp
    const char* legacy_str = "Hello";
    std::u8string new_str = reinterpret_cast<const char8_t*>(legacy_str);  // 需确保原字符串是 UTF-8
    ```

### 2. `std::u8string` 容器

- **专用容器**：基于 `char8_t` 的字符串类，提供与 `std::string` 相同的接口（如 `append()`, `find()`）。

    ```cpp
    #include <string>
    std::u8string utf8_str = u8"你好，世界！";
    utf8_str += u8" 🚀";  // 支持 Unicode 扩展字符（如 Emoji）
    ```

- **注意**：不能直接通过 `std::cout` 输出（需类型转换）：

    ```cpp
    std::cout << reinterpret_cast<const char*>(utf8_str.c_str());  // 强制转换输出
    ```

`std::u8string::size()` 返回的也是字节数而非字符数，需用第三方库（如 ICU）计算字符数

### 3. `u8` 前缀行为变化

- **C++17**：`u8"文本"` 返回 `const char[]`，易与其他编码混淆。
- **C++20**：`u8"文本"` 返回 `const char8_t[]`，编译器强制类型检查。

**兼容性处理**：旧代码迁移时需调整接收类型或使用 `reinterpret_cast`。

## 避免字符乱码的有效实践

字符乱码问题的根源在于编码不一致导致的数据解释错位。当源码文件、终端环境和程序内部对字符的编码规则不统一时，二进制数据会被错误解析，从而产生乱码。统一使用 UTF-8 编码可彻底解决这一问题。

还需注意的是：若源码为 UTF-8，但字符串操作函数（如`strlen`）按字节处理，截断多字节字符（如中文占 3 字节）会导致乱码；C++ 代码中**使用`u8"中文"`强制 UTF-8 字符串**以确保程序内字符为 UTF-8 编码。`u8` 前缀在这种情况下能提供更强的语义保证，并确保即使在编译器设置不明确的情况下，该字符串也一定是 UTF-8，增加了代码的健壮性。

MSVC 下，如果源代码文件使用 UTF-8 BOM 编码，则编译器会通过 BOM 标记（`EF BB BF`）自动识别文件为 UTF-8 编码，无需额外设置源字符集选项，若使用 UTF-8，则需要手动设置源字符集为 UTF-8。此时，若不设置执行字符集，MSVC 默认使用本地代码页（在中文 Windows 系统上通常是 GBK，代码页 936），编译器会将之前解析出的 Unicode 码点转换为 GBK 编码的字节序列，并存入最终生成的可执行文件中。在“源文件 UTF-8 → 编译器 → 执行文件 GBK → GBK 控制台”这个链条下，程序在中文 Windows 上能正常工作，且无需额外设置代码页。但还是更推荐同时显式设置执行字符集为 UTF-8，因为这是将程序行为从“依赖环境”变为“自我定义”，不再受限于操作系统的默认语言环境，可以确保程序在不同语言的 Windows 上都能正常工作。

在跨平台项目中，为了编写可移植性更好、行为更一致的代码，推荐**源代码文件统一使用 UTF-8 编码，不带 BOM**，MSVC 上使用 `/utf-8` 选项（等同于 `/source-charset:utf-8 /execution-charset:utf-8`）。这使得你的代码在不同语言环境的 Windows 系统，以及在 Linux 或 macOS 上都有着相同的行为（字符串在运行时均为 UTF-8 编码）。虽然这意味着在默认的中文 Windows 控制台上运行时需要额外处理（如 `chcp 65001` 或修改 `locale`），但这从长远来看能避免更多因编码不一致导致的潜在问题。

具体地，在 `CMakeLists.txt` 添加编译选项（前面提到过，若不设置，则默认为系统区域设置）：

```cmake
if(MSVC)
    add_compile_options(/utf-8)
    # 等同于以下两行
    add_compile_options(/source-charset:utf-8)
    add_compile_options(/execution-charset:utf-8)
endif()
# 或者
target_compile_options(MyTarget PRIVATE $<$<C_COMPILER_ID:MSVC>:/utf-8>)
```

还需要作以下设置，示例如下：

```cpp
#include <iostream>

#ifdef _WIN32
#include <windows.h>
#include <codecvt>
#endif

void init_global_utf8() {
#ifdef _WIN32
    SetConsoleOutputCP(CP_UTF8);
    SetConsoleCP(CP_UTF8);
 setlocale(LC_ALL, ".utf8");
#pragma warning(push)
#pragma warning(disable: 4996)
    std::wcout.imbue(std::locale(std::wcout.getloc(), new std::codecvt_utf8_utf16<wchar_t>));
#pragma warning(pop)
#endif
}
```

`SetConsoleOutputCP(CP_UTF8); SetConsoleCP(CP_UTF8);`设置代码页为 UTF-8

`setlocale(LC_ALL, ".utf8");` 设置 locale

`wcout.imbue(locale(wcout.getloc(), new codecvt_utf8_utf16<wchar_t>));` 将 wcout 接受的 utf16 编码的 wchar_t 流转换为 utf8 buffer，并需要抑制编译警告。

还可以配置 Git 提交时设置 UTF-8 编码：

```bash
# 1. 全局编码设置
git config --global i18n.commitencoding utf-8     # 提交信息使用 UTF-8 编码
git config --global i18n.logoutputencoding utf-8  # 日志输出使用 UTF-8 编码
git config --global gui.encoding utf-8            # Git GUI 界面使用 UTF-8
git config --global core.quotepath false          # 禁用路径转义，使状态信息正确显示中文文件名

# 2. 换行符规范
git config --global core.autocrlf true            # 提交时换行符转为 LF
git config --global core.safecrlf true            # 拒绝混合换行符的提交，避免冲突

# 3. 系统环境变量（Windows）
#    新建用户变量：变量名 `LESSCHARSET`，值 `utf-8`
#    用于配置 less 分页器工具的环境变量，其核心作用是强制 less 在显示文本内容时使用 UTF-8 编码，避免因终端编码不匹配导致的乱码问题

# 4. 重启终端并验证（Linux）
git config --global --list | grep encoding

# 5. 新项目初始化仓库时启用 UTF-8，已有项目需要手动转换文件编码
git init
echo "* text=auto" > .gitattributes
# .gitattributes 主要用于规范化换行符，确保跨平台协作一致性
# 要强制文件编码为 UTF-8，最佳实践是添加 .editorconfig 文件
```

## 参考资料

<https://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html>

<https://blog.csdn.net/xiangxianghehe/article/details/77574965>

<https://www.zhihu.com/question/20650946/answer/52835091801>

<https://www.zhihu.com/question/20167122/answer/14194448>

<https://zhuanlan.zhihu.com/p/546806312>

<https://github.com/lovepika/thinking_file_encoding_cpp/tree/master>

<https://zhuanlan.zhihu.com/p/717454699>
