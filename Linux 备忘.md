# Linux 备忘

## 命令行回收站工具 - `Trash-Cli`

可使用使用 apt-get 或 apt 命令来安装 Trash-Cli：

```bash
sudo apt install trash-cli
```

使用 Trash-Cli:

Trash-Cli 提供了下面这些命令：

```bash
trash-put： 删除文件和目录（仅放入回收站中）
trash-list ：列出被删除了的文件和目录
trash-restore：从回收站中恢复文件或目录 trash.
trash-rm：删除回收站中的文件
trash-empty：清空回收站
```

删除超过 X 天的垃圾文件：

```bash
# 删除回收站中超过 30 天的文件
$ trash-empty 30
```

参考：
<https://zhuanlan.zhihu.com/p/44948578>

## 终端使用代理

创建一个别名（alias），在你的 shell 配置文件（通常是.bashrc、.bash_profile、.zshrc）中添加以下内容：

```bash
alias proxy-on='export http_proxy=http://proxyAddress:port; export https_proxy=http://proxyAddress:port'
alias proxy-off='unset http_proxy; unset https_proxy'
# 注: proxyAddress 需要替换为代理服务器IP, port 为服务端口
```

应用配置

```bash
source ~/.zshrc 或 source ~/.bash_profile
```

此后，便能够直接通过运行 proxy-on 来启用代理，运行 proxy-off 来禁用代理。

参考：
<https://blog.csdn.net/zsuroy/article/details/143220595>
<https://zhuanlan.zhihu.com/p/357875811>

## 复制依赖库

```bash
mkdir lib64
ldd xxx | awk '{print $3}' | xargs -i cp -L {} ./lib64/
```

## 查看目录/文件大小 - `du`

```bash
# 显示目录/文件总大小
du -sh /path/to/directory
# -s仅显示总大小，不列出子目录细节
# -h自动转换为KB、MB、GB等易读单位

# 使用--max-depth=N限制遍历层级（如仅显示一级子目录）
du -h --max-depth=1 /path/to/directory

# 排除特定文件/目录
du -sh --exclude='*.log' /var  # 排除所有.log文件

# 统计多个目录/文件并排序
du -sh * | sort -rh  # 查看当前目录下所有子项大小并降序排列

# 符号链接处理
du -Lsh /path/to/symlink  # 计算链接目标的大小
# -L参数统计符号链接指向的实际文件大小，而非链接本身

# df命令快速查看磁盘剩余空间
df -h /path/to/directory  # 显示目录所在分区的总/已用/剩余空间
```

## 查看程序/命令的路径 - `which`

如果我们想要查找一些可以在终端直接使用的程序的路径，比如 `gcc`、`cmake`、`git` 这种，我们可以使用 `which` 命令查看，通常都是在 `/usr/bin/`：

```bash
which cmake
/usr/bin/cmake
```

如果是 windows，powershell 可以使用 `Get-Command`、`where.exe`：

```powershell
PS C:\Users\A1387> Get-Command cmake

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Application     cmake.exe                                          3.25.2.0   D:\CLion 2023.1.4\bin\cmake\win\x64\bin\cmake.exe

PS C:\Users\A1387> where.exe cmake
D:\CLion 2023.1.4\bin\cmake\win\x64\bin\cmake.exe
```

> CMD 终端直接使用 **`where`** 即可。powershell 终端要强调 `.exe` 是因为 where 这个名字被用作另一个东西的别名了。

## Debian 和 Ubuntu 系列 Linux 发行版的包管理工具 - `apt`

`apt` 是一个用于 Debian 和 Ubuntu 系列 Linux 发行版的包管理工具。它提供了简单的方法来安装、更新和删除软件包。基本用法包括：

```bash
sudo apt update              # 更新软件包列表
sudo apt install [package]   # 安装指定的软件包
sudo apt remove [package]    # 卸载指定的软件包
```

安装的库通常位于 `/usr/lib/x86_64-linux-gnu`，头文件位于 `/usr/include`。使用以下命令可以查看某个包的安装位置：

```bash
dpkg -L [package]
```

- 优点：apt 可以自动处理依赖关系，并且从官方源中安装软件相对安全和可靠。

## 去除 UTF-8 BOM 头

```bash
# 递归查找并去除当前目录及子目录下所有文件的BOM
grep -rlI $'\xEF\xBB\xBF' . | xargs sed -i 's/\xEF\xBB\xBF//'
```

- 关键参数：

    - `grep -rlI`：递归查找包含 BOM 的文件（`-r`递归，`-l`仅输出文件名，`-I`忽略二进制文件）

    - `sed -i`：直接修改文件内容

- macOS 兼容：需先安装 GNU sed：`brew install gnu-sed`，再用 `gsed` 替代 `sed`
