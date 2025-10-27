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

如果是 Windows，Powershell 可以使用 `Get-Command`、`where.exe`：

```powershell
PS C:\Users\Kerwin> Get-Command cmake

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Application     cmake.exe                                          3.25.2.0   D:\CLion 2023.1.4\bin\cmake\win\x64\bin\cmake.exe

PS C:\Users\Kerwin> where.exe cmake
D:\CLion 2023.1.4\bin\cmake\win\x64\bin\cmake.exe
```

> CMD 终端直接使用 **`where`** 即可。powershell 终端要强调 `.exe` 是因为 where 这个名字被用作另一个东西的别名了。

## `apt` 与 `apt-get`

`apt` 和 `apt-get` 是 Debian 及其衍生发行版（如 Ubuntu）中用于管理软件包的命令行工具。`apt-get` 发布于1998年，而 `apt` 发布于2014年，是 `apt-get` 的一个更新、更友好的版本，旨在改善交互式用户体验。

以下是 `apt` 和 `apt-get` 之间的主要区别：

| 特性 | `apt` | `apt-get` |
| --- | --- | --- |
| **目标用户** | 最终用户（交互式使用） | 脚本和后端工具 |
| **功能** | 整合了 `apt-get` 和 `apt-cache` 的常用功能 | 更底层，专注于包的安装、升级和移除 |
| **用户体验** | 更友好，提供进度条、彩色输出等 | 输出更偏向机器可读，适合脚本解析 |
| **稳定性** | 输出格式可能会在不同版本间变化 | 命令和输出格式保持向后兼容，适合脚本使用 |


下面是它们常用命令的对比整理:


| 功能描述 | 新的 `apt` 命令 | 旧的 `apt-get` / `apt-cache` 命令 | 备注 |
| --- | --- | --- | --- |
| **更新软件包列表** | `sudo apt update` | `sudo apt-get update` | 功能相同，但 `apt` 会额外提示有多少个包可以升级。 |
| **升级已安装的包** | `sudo apt upgrade` | `sudo apt-get upgrade` | `apt upgrade` 会在需要时**安装新依赖**，行为更智能。 `apt-get upgrade` 则不会。 |
| **完整升级系统** | `sudo apt full-upgrade` | `sudo apt-get dist-upgrade` | 功能基本相同，处理依赖关系变化，可能会卸载旧包。`apt` 只是换了个更清晰的名称。 |
| **安装软件包** | `sudo apt install <包名>` | `sudo apt-get install <包名>` | 功能相同，但 `apt` 会提供进度条。 |
| **卸载软件包** | `sudo apt remove <包名>` | `sudo apt-get remove <包名>` | 功能相同，只移除软件包，保留配置文件。 |
| **彻底卸载软件包** | `sudo apt purge <包名>` | `sudo apt-get purge <包名>` | 功能相同，同时移除软件包和其配置文件。 |
| **搜索软件包** | `apt search <关键词>` | `apt-cache search <关键词>` | `apt` 整合了 `apt-cache` 的功能，无需切换命令。 |
| **查看软件包信息** | `apt show <包名>` | `apt-cache show <包名>` | `apt` 整合了 `apt-cache` 的功能，输出的信息对用户更友好，隐藏了一些技术细节。 |
| **列出软件包** | `apt list` | `dpkg -l` | `apt` 提供了更强大和易用的列表功能。 |
| **列出可升级的包** | `apt list --upgradable` | `apt-get upgrade -s` (模拟) | `apt` 直接提供了专门的命令，非常方便。 |
| **清理不再需要的包** | `sudo apt autoremove` | `sudo apt-get autoremove` | 功能相同，用于移除自动安装且不再被依赖的包。 |
| **编辑源列表** | `apt edit-sources` | `(手动编辑文件)` | `apt` 提供了一个便捷的命令来直接编辑 `/etc/apt/sources.list` 文件。 |

其他相关命令：

- `dpkg -L <包名>` 查看一个包安装的所有文件和目录
- `which <命令>` 快速查找可执行命令的路径
- `whereis <命令>` 查找命令、源码和手册页的路径
- `dpkg -S <路径>` 根据文件路径反查所属的软件包

总而言之，对于日常在终端的手动操作，使用 `apt` 会更简单、更直观。而 `apt-get` 因为其稳定性和向后兼容性，仍然是编写自动化脚本时的首选。

## 去除 UTF-8 BOM 头

```bash
# 递归查找并去除当前目录及子目录下所有文件的BOM
grep -rlI $'\xEF\xBB\xBF' . | xargs sed -i 's/\xEF\xBB\xBF//'
```

- 关键参数：

    - `grep -rlI`：递归查找包含 BOM 的文件（`-r`递归，`-l`仅输出文件名，`-I`忽略二进制文件）

    - `sed -i`：直接修改文件内容
- macOS 兼容：需先安装 GNU sed：`brew install gnu-sed`，再用 `gsed` 替代 `sed`

## 修复失效的软链接

如果软链接的目标文件仍然存在，只是路径发生了变化（例如整个目录被移动），可以尝试批量更新它们指向的目标路径。

可以使用 find 与 sed 进行批量路径替换

适用于：失效软链接的旧目标路径有统一的、可批量替换的前缀。


```bash
find . -maxdepth 1 -type l -exec sh -c 'target=$(readlink "$1"); if [ ! -e "$1" ]; then new_target=$(echo "$target" | sed "s|^/old/path|/new/path|"); ln -sfn "$new_target" "$1"; echo "已修复: $1"; fi' sh {} \;
```

命令解释：

`target=$(readlink "$1")`：获取当前软链接原本指向的目标路径。

`if [ ! -e "$1" ]; then`：判断当前软链接是否失效。

`new_target=$(echo "$target" | sed "s|^/old/path|/new/path|")`：使用 sed命令将旧路径前缀`/old/path`替换为新前缀`/new/path`。

`ln -sfn "$new_target" "$1"`：使用`ln -sfn`命令强制 （`-f`）重新创建同名软链接，指向新的目标路径（`$new_target`），`-n`选项用于处理目标是软链接的情况。

使用时请注意将示例中的`/old/path`和`/new/path`替换为你实际的旧路径前缀和新路径前缀。

## 安装和配置 MySQL

安装和配置 MySQL 的详细步骤如下：

1. 更新系统包索引

```bash
sudo apt update
sudo apt upgrade -y
```

2. 安装 MySQL Server

```bash
# 安装 MySQL 服务器（默认为 8.0 版本）
sudo apt install mysql-server -y
```

3. 启动 MySQL 服务

```bash
# 启动 MySQL 服务
sudo systemctl start mysql

# 设置开机自启
sudo systemctl enable mysql

# 检查服务状态
sudo systemctl status mysql
```

4. 运行安全配置脚本

```bash
# 运行安全安装向导
sudo mysql_secure_installation
```

5. 配置 root 用户密码

```bash
# 登录 MySQL
sudo mysql

# 在 MySQL 中执行
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';
FLUSH PRIVILEGES;
EXIT;
```

6. 基本配置调整

编辑 MySQL 配置文件：

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

常用配置项：

```ini
[mysqld]
# 绑定地址（允许远程访问改为 0.0.0.0）
bind-address = 127.0.0.1

# 字符集设置
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# 最大连接数
max_connections = 100

# 启用二进制日志（主从复制需要）
# server-id = 1
# log_bin = /var/log/mysql/mysql-bin.log
```

7. 重启 MySQL 服务

```bash
sudo systemctl restart mysql
```

8. 常用管理命令

```bash
# 启动服务
sudo systemctl start mysql

# 停止服务
sudo systemctl stop mysql

# 重启服务
sudo systemctl restart mysql

# 查看服务状态
sudo systemctl status mysql

# 查看日志
sudo tail -f /var/log/mysql/error.log
```

9. 创建新用户和数据库

```sql
-- 创建数据库
CREATE DATABASE mydatabase;

-- 创建用户
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'password';

-- 授予权限
GRANT ALL PRIVILEGES ON mydatabase.* TO 'myuser'@'localhost';

-- 刷新权限
FLUSH PRIVILEGES;
```

10. 防火墙配置（如果需要远程访问）

```bash
# 开放 MySQL 端口（默认 3306），有一定安全风险
sudo ufw allow 3306/tcp
sudo ufw reload
```

11. 备份和恢复

```bash
# 备份数据库
mysqldump -u username -p database_name > backup.sql

# 恢复数据库
mysql -u username -p database_name < backup.sql
```

12. 忘记 root 密码处理方法

```bash
# 停止 MySQL 服务
sudo systemctl stop mysql

# 创建临时启动配置
sudo mkdir -p /var/run/mysqld
sudo chown mysql:mysql /var/run/mysqld

# 跳过权限检查启动 MySQL
sudo mysqld_safe --skip-grant-tables --skip-networking &

# 无密码登录 MySQL
mysql -u root

# 在 MySQL 中执行（重要：8.0+专用语法）
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY '你的新密码';
UNINSTALL COMPONENT "file://component_validate_password";  # 如果密码策略报错则先执行这行
ALTER USER 'root'@'localhost' IDENTIFIED BY '你的新密码';  # 再次执行
EXIT;

# 关闭临时实例
sudo mysqladmin -u root -p shutdown

# 正常启动服务
sudo systemctl start mysql
```

13. 卸载 MySQL 服务

```bash
# 检查安装包
dpkg -l | grep mysql
# 卸载 MySQL 服务
sudo apt purge mysql-server mysql-server-8.0 mysql-server-core-8.0 
```

## 开启 FTP 服务

安装 vsftpd，并开启服务：

```bash
sudo apt update
sudo apt install vsftpd
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
```

修改 vsftpd 配置：

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
sudo vim /etc/vsftpd.conf # 禁止匿名登录
anonymous_enable=NO
# 允许本地用户登录
local_enable=YES
# 启用写入权限
write_enable=YES
# 将用户限制在其家目录中（增强安全）
chroot_local_user=YES
# 防止中文乱码
utf8_filesystem=YES
```

重启 vsftpd 服务：

```bash
sudo systemctl restart vsftpd
```

## 离线安装 deb 包及依赖

以下以 C++ 开发环境配置为情景介绍离线安装 deb 包及依赖的方法：

需要准备两个系统版本号完全一致的 Linux 主机，一台可以连接互联网，用于下载 deb 包和依赖，另一台为离线安装的主机。

首先，在可以连接互联网的机器上运行以下命令：

```bash
apt download $(apt-cache depends --recurse --no-recommends --no-suggests --no-conflicts --no-breaks --no-replaces --no-enhances build-essential gdb ninja-build valgrind git vsfptd openssh-server pkg-config perl python3 | grep "^\w" | sort -u)

apt download $(apt-cache depends --recurse --no-recommends --no-suggests --no-conflicts --no-breaks --no-replaces --no-enhances libboost-all-dev libopencv-dev qtbase5-dev qt6-base-dev
libssl-dev libcurl4-openssl-dev zlib1g-dev | grep "^\w" | sort -u)
```

deb 包会下载到当前目录下，打包传输到离线的主机上，并运行以下命令安装：

```bash
sudo dkpg -i *.deb
```

