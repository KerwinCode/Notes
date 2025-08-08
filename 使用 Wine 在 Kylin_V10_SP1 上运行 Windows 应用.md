# 使用 Wine 在 Kylin_V10_SP1 上运行 Windows 应用

本文将详细介绍如何在 Kylin V10 SP1 桌面操作系统 (Kylin-Desktop-V10-SP1-2503-HWE-Release-20250430-X86_64) 上安装和配置 Wine，以成功运行 Windows 应用程序。

## 什么是 Wine？

Wine（“Wine Is Not an Emulator” 的递归缩写，即“Wine 不是模拟器”）是一个能够在多种兼容 POSIX 的操作系统（如 Linux、macOS 及 BSD 等）上运行 Windows 应用的兼容层。

与模拟器或虚拟机不同，Wine 不会模拟一个完整的 Windows 操作系统。相反，它将 Windows 的 API 调用实时转换成等效的 POSIX 调用，让 Windows 程序能直接在您的 Linux 系统上运行。这种设计的优点是性能开销极小，可以达到近乎原生的应用性能。

## Wine 的安装

首先，我们需要为系统添加 32 位架构的支持，因为许多 Windows 应用仍然是 32 位的。

### 启用32位架构并安装基础库

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libasound2-plugins:i386 libc6:i386 libncurses5:i386 libstdc++6:i386 libx11-6:i386 libxext6:i386 libxinerama1:i386 libxrandr2:i386 libxrender1:i386 libglu1-mesa:i386 libpulse0:i386 libdbus-1-3:i386
```

### 添加 WineHQ 官方仓库并安装 Wine

使用 Kylin 默认 apt 源安装的 wine 版本较旧，为了获取最新和最稳定的 Wine 版本，推荐使用 WineHQ 官方提供的软件源，"HQ" 是英文 ​Headquarters​（总部）的缩写，WineHQ 是 Wine 项目的官方维护组织及资源集成平台，负责协调开发、发布版本、管理社区基础设施。

```bash
# 下载并添加 WineHQ 的 GPG 密钥
wget -nc https://dl.winehq.org/wine-builds/winehq.key
sudo apt-key add winehq.key

# 添加清华大学的 Wine 构建镜像源（速度更快）
sudo sh -c 'echo "deb https://mirrors.tuna.tsinghua.edu.cn/wine-builds/ubuntu/ focal main" >> /etc/apt/sources.list.d/winehq.list'

# 更新软件列表并安装 winehq-stable
sudo apt update
sudo apt install --install-recommends winehq-stable
```

安装完成后，可以通过以下命令查看 Wine 的版本：

```bash
wine --version
```

如果看到类似 `wine-9.0` 或更高版本的输出，说明 Wine 已经成功安装。

## Wine 的配置与使用

### 首次配置与 winecfg 工具

在首次运行 Wine 时，它会自动在您的主目录下创建一个名为 `.wine` 的隐藏文件夹。这个文件夹就是 Wine 的默认“容器”或“前缀” (Wine Prefix)，用于存放虚拟的 C: 盘以及所有 Windows 应用的配置和数据。

`winecfg` 是 Wine 的主要配置工具，它提供了一个图形化界面，让您可以方便地调整 Wine 的各项设置。您可以随时在终端中运行 `winecfg` 来打开它。

`winecfg` 的主要功能包括：

* **应用程序设置**: 您可以为特定的应用程序设置其模拟的 Windows 版本（例如，让某个旧程序运行在 Windows XP 兼容模式下）。
* **函数库**: 管理 DLL 文件的加载方式，可以设置使用 Wine 内建的 DLL 还是 Windows 原生的 DLL。这对于解决程序兼容性问题至关重要。
* **显示**: 配置虚拟桌面的分辨率、调整窗口管理和渲染相关的设置。
* **驱动器**: 将 Linux 系统中的路径映射为 Wine 环境中的虚拟驱动器盘符（如 D:、E: 等）。

### 使用 Winetricks 简化环境部署

许多 Windows 应用程序需要依赖特定的运行时库（如 .NET Framework、Visual C++ Redistributable）或字体才能正常工作。手动安装这些组件非常繁琐，而 `winetricks` 就是一个能极大简化这个过程的辅助脚本。

#### 安装 Winetricks

```bash
# cabextract 是解压 Windows CAB 文件的必备工具，缺失会导致无法正确安装相关运行库
sudo apt install cabextract -y

# 可代理加速下载，根据实际情况修改 IP
export https_proxy=http://127.0.0.1:7897 http_proxy=http://127.0.0.1:7897 all_proxy=socks5://127.0.0.1:7897

# 下载最新版的 winetricks 脚本
wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
chmod +x winetricks
sudo mv winetricks /usr/bin  # 将其移动到系统路径，使其全局可用
```

#### 使用 Winetricks 安装常用组件

以下命令将安装 .NET 4.8、Visual C++ 2022 运行库以及一些核心和中日韩字体，这些是许多应用的基础依赖。

```bash
# 安装运行时和字体
winetricks dotnet48 vcrun2022 corefonts cjkfonts
```

### 运行 Windows 应用

要运行一个 Windows 的 `.exe` 程序，只需使用 `wine` 命令：

```bash
wine ~/path/to/your/app.exe
```

### 使用独立的 Wine 容器 (Wine Prefix)

直接使用默认的 `~/.wine` 目录虽然方便，但所有应用都装在一起可能会引发冲突。例如，A 应用需要 32 位环境和 .NET 2.0，而 B 应用需要 64 位环境和 .NET 4.8。

为了解决这个问题，Wine 引入了 **Wine Prefix**（可理解为独立的 **Wine 容器**）的概念。您可以为每个（或每类）应用创建一个独立的运行环境，它们之间互不干扰。

`WINEPREFIX` 是一个环境变量，用于指定 Wine 配置目录的路径。

创建和使用新的 Wine 容器：

1. **指定容器路径并初始化**: 下面的命令会创建一个 64 位的 Wine 容器，路径为 `~/.wine-app`。`winecfg` 会被自动调用以完成初始化。

    ```bash
    WINEPREFIX=~/.wine-app WINEARCH=win64 winecfg
    ```

    * `WINEPREFIX=~/.wine-app`: 指定容器的存储位置。
    * `WINEARCH=win64`: 指定这是一个 64 位的环境。如果需要 32 位环境，请使用 `WINEARCH=win32`。**注意：`WINEARCH` 只能在创建容器的第一次指定，之后无法更改。**

2. **在指定容器中安装依赖**: 使用 `winetricks` 时，同样通过 `WINEPREFIX` 环境变量来指定目标容器。

    ```bash
    WINEPREFIX=~/.wine-app winetricks dotnet48
    ```

3. **在指定容器中运行应用**:

    ```bash
    WINEPREFIX=~/.wine-app wine ~/app/app.exe
    ```

使用独立的 Wine 容器是管理复杂 Windows 应用的最佳实践，它能确保每个应用都拥有一个干净、隔离且配置正确的运行环境，同时也方便了应用的迁移和卸载（只需删除对应的容器文件夹即可）。
