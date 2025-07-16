# Qt 5.14.2 + VS 2022 开发环境配置

## 使用 QtCreator 开发

由于当前版本的 QtCreator Kits 编译器最高只支持 VS2017

所以安装 VS 2022 时需要勾选 “MSVC v141 VS 2017 C++ x86/x64 生成工具”

并需要额外安装调试器（`cdb.exe`），`cdb.exe` 是 Windows SDK 或 WDK（Windows Driver Kit）的一部分，通常随 SDK 的 Debugging Tools for Windows 组件安装。（实测 VS 2022 17.14.3 离线安装包中的 Windows SDK 中 不包含 X64 Debuggers And Tools，但 VS 2022 15.9.21 的 Win10 SDK 10.0.17763 包含了）

以下是单独安装方法：

下载 Windows SDK ISO ： <https://developer.microsoft.com/zh-cn/windows/downloads/windows-sdk/>

加载 ISO，安装包位置在 Installers\X64 Debuggers And Tools-x64_en-us.msi，安装即可

配置编译器和 Kits：

编译器路径大致为：C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\cdb.exe 或 C:\Program Files\Debugging Tools for Windows (x64)\cdb.exe

调试器路径大致为：C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat 并设置好 ABI

## 使用 VS 2022 开发

安装 Qt VS Tools 扩展，并在扩展中配置好 Version 即可
