# Windows 程序崩溃 Dump 的捕获与调试

## 在程序中设置捕获 MiniDump

```cpp
void CreateDumpFile(EXCEPTION_POINTERS* exceptionInfo) {
    HANDLE hFile = CreateFileW(L"dumpfile.dmp", GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
    if (hFile != INVALID_HANDLE_VALUE) {
        MINIDUMP_EXCEPTION_INFORMATION mdei;
        mdei.ThreadId = GetCurrentThreadId();
        mdei.ExceptionPointers = exceptionInfo;
        mdei.ClientPointers = FALSE;

        MiniDumpWriteDump(GetCurrentProcess(), GetCurrentProcessId(), hFile, MiniDumpWithFullMemory, &mdei, NULL, NULL);
        CloseHandle(hFile);
    }
}

LONG WINAPI ExceptionHandler(EXCEPTION_POINTERS* exceptionInfo) {
    CreateDumpFile(exceptionInfo);
    return EXCEPTION_EXECUTE_HANDLER; // 继续执行默认的崩溃处理
}

int main(){
    SetUnhandledExceptionFilter(ExceptionHandler);
    int* p = nullptr;
    *p = 0;
}
```

还需要注意以下三点：

1. 链接 `dbghelp` 、`wininet` 这两个库

2. 添加选项用以生成符号文件

    ```cmake
    add_compile_options(/Zi)
    add_link_options(/DEBUG)
    ```

3. 保存每个版本对应的 .pdb 文件

## 完整测试代码

```cpp
#include <windows.h>
#include <dbghelp.h>
#include <string>
#include <chrono>
using namespace std::chrono_literals;

inline std::string get_current_date() {
    // 获取当前时间
    auto now = std::chrono::system_clock::now();
    std::time_t now_time = std::chrono::system_clock::to_time_t(now);

    // 格式化时间
    std::tm* tm_ptr = std::localtime(&now_time);
    std::stringstream ss;
    ss << std::put_time(tm_ptr, "%Y-%m-%d");
    return ss.str();
}

void CreateDumpFile(EXCEPTION_POINTERS* exceptionInfo) {
    HANDLE hFile = CreateFile((get_current_date() + ".dmp").c_str(), GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
    if (hFile != INVALID_HANDLE_VALUE) {
        MINIDUMP_EXCEPTION_INFORMATION mdei;
        mdei.ThreadId = GetCurrentThreadId();
        mdei.ExceptionPointers = exceptionInfo;
        mdei.ClientPointers = FALSE;

        MiniDumpWriteDump(GetCurrentProcess(), GetCurrentProcessId(), hFile, MiniDumpWithFullMemory, &mdei, NULL, NULL);
        CloseHandle(hFile);
    }
}

LONG WINAPI ExceptionHandler(EXCEPTION_POINTERS* exceptionInfo) {
    CreateDumpFile(exceptionInfo);
    return EXCEPTION_EXECUTE_HANDLER; // 继续执行默认的崩溃处理
}

int main(){
    SetUnhandledExceptionFilter(ExceptionHandler);
    int* p = nullptr;
    *p = 0;
}
```

```cmake
cmake_minimum_required (VERSION 3.8)

set(CMAKE_CXX_STANDARD 20)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/build/${CMAKE_BUILD_TYPE}/bin)

project ("test_dump")

add_compile_options(/Zi)
add_link_options(/DEBUG)

add_executable (${PROJECT_NAME} "main.cpp")

target_link_libraries(${PROJECT_NAME} PRIVATE
    dbghelp
    wininet
)
```

### 参考资料

<https://www.bilibili.com/video/BV1pnXaYPECx/>

<https://github.com/Mq-b/CppLab/blob/main/md/%E4%BB%8E%E9%9B%B6%E9%85%8D%E7%BD%AEwindows%E7%A8%8B%E5%BA%8F%E5%B4%A9%E6%BA%83Dump%E6%8D%95%E8%8E%B7%E4%B8%8E%E8%B0%83%E8%AF%95.md>
