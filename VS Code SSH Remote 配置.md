# VS Code SSH Remote 配置

## 系统环境

Win11 23H2 + 银河麒麟 V10 SP1 Vmware 虚拟机

需要提前安装 OpenSSH 服务端，用于提供 SSH 服务端守护进程 `sshd`，监听连接请求：

```bash
sudo apt update
sudo apt install openssh-server
```

建议提前配置好静态IP：

```bash
ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.171.171  netmask 255.255.255.0  broadcast 192.168.171.255
        inet6 fe80::3c45:ab16:8302:592f  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:e1:91:d8  txqueuelen 1000  (以太网)
        RX packets 3538  bytes 3403836 (3.4 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2668  bytes 293887 (293.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

route -n
内核 IP 路由表
目标            网关            子网掩码        标志  跃点   引用  使用 接口
0.0.0.0         192.168.171.2   0.0.0.0         UG    100    0        0 ens33
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 ens33
192.168.171.0   0.0.0.0         255.255.255.0   U     100    0        0 ens33
```

可参照上述命令结果填写网络配置，静态 IP 填写 192.168.171.171（或同网段其他IP），子网掩码填 255.255.255.0，默认网关填 192.168.171.2，DNS 添加  114.114.114.114（中国电信DNS） 和 8.8.8.8（Google DNS）。

## 连接配置

vs code 报错 Failed to set up socket for dynamic port forward to remote port

```bash
sudo tail -n 100 /var/log/auth.log
```

日志输出如下：

```text
Mar 28 16:24:40 dev-pc sshd[6218]: Received disconnect from 192.168.171.1 port 51686:11: disconnected by user
Mar 28 16:24:40 dev-pc sshd[6218]: Disconnected from user dev 192.168.171.1 port 51686
Mar 28 16:24:40 dev-pc sshd[6204]: pam_unix(sshd:session): session closed for user dev
Mar 28 16:24:40 dev-pc systemd-logind[951]: Session 7 logged out. Waiting for processes to exit.
Mar 28 16:24:52 dev-pc sshd[6485]: Accepted password for dev from 192.168.171.1 port 52468 ssh2
Mar 28 16:24:52 dev-pc sshd[6485]: pam_unix(sshd:session): session opened for user dev by (uid=0)
Mar 28 16:24:52 dev-pc systemd-logind[951]: New session 8 of user dev.
Mar 28 16:24:53 dev-pc sshd[6498]: Received request from 192.168.171.1 port 52468 to connect to host 127.0.0.1 port 36045, but the request was denied.
Mar 28 16:24:53 dev-pc sshd[6498]: Received request from 192.168.171.1 port 52468 to connect to host 127.0.0.1 port 36045, but the request was denied.
Mar 28 16:25:15 dev-pc sshd[6498]: Received disconnect from 192.168.171.1 port 52468:11: disconnected by user
Mar 28 16:25:15 dev-pc sshd[6498]: Disconnected from user dev 192.168.171.1 port 52468
Mar 28 16:25:15 dev-pc sshd[6485]: pam_unix(sshd:session): session closed for user dev
```

解决办法：

```bash
sudo vim /etc/ssh/sshd_config

# 编辑设置：
AllowTcpForwarding yes
PermitOpen any

sudo systemctl restart sshd
```

参考：<https://zhuanlan.zhihu.com/p/720810569>

### 免密配置

使用命令 ssh-keygen 生成新的密钥对。你可以选择在生成密钥对时为其指定不同的文件名。请注意，-f 后的 id_rsa_linux 和 id_rsa_windows 只是示例文件名，你可以根据需要选择其他文件名，也可以不指定文件名或其他参数。

```bash
 ssh-keygen

 # 可根据需要指定参数
 ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa_linux
 ssh-keygen -t rsa -b 2048 -f C:\Users\YourUsername\.ssh\id_rsa_windows
```

输入命令后一路回车

系统会在你指定的路径（本例子为 C:\Users\YourUsername\.ssh）下生成两个文件，分别是 id_rsa_windows.pub 和 id_rsa_windows，前者为生成的公钥，后者为私钥 。

将生成的公钥（ id_rsa_windows.pub 的内容）添加到你远程服务器的 ~/.ssh/authorized_keys 文件中，以允许连接。

将添加公钥到远程服务器后，最后一步便是配置你的主机。

打开你的 SSH 客户端（本机）配置文件（也就是前面生成的 config 文件，一般在 C:\Users\YourUsername\\.ssh\config），添加配置（IdentityFile 私钥文件路径），以指定使用哪个私钥文件。

参考：<https://zhuanlan.zhihu.com/p/667236864>

## 其他注意事项

不同主机复用同一 IP 时，需要在 Windows 系统中涉及清理 `known_hosts`文件（存储已连接服务器的公钥记录）。
