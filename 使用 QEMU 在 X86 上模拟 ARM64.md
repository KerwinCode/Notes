# 使用 QEMU 在 X86 上模拟 ARM64

QEMU（Quick Emulator）是一款开源的机器模拟器和虚拟化软件，由 Fabrice Bellard 于 2003 年创建。它通过动态二进制转换技术实现跨平台虚拟化，支持 x86、ARM、MIPS 等多种处理器架构。QEMU 既可以作为独立虚拟机运行完整的操作系统，也可与 KVM（基于内核的虚拟机）配合使用实现硬件加速虚拟化，这种组合方案能够提供接近原生性能的虚拟化体验。

Windows 上可以在 WSL2 上使用 KVM 加速 QEMU 模拟，需要内核支持（可能需要重新编译 Linux Kernel），但由于 KVM 本身作为 Linux 内核模块，主要依赖 CPU 的硬件虚拟化扩展（如 Intel VT-x/AMD-V）。这意味着 KVM 的虚拟化能力与物理 CPU 架构强绑定——x86 平台的 KVM 只能虚拟化 x86 客户机，ARM 平台的 KVM 只能虚拟化 ARM 客户机。笔者也对这种方式做了测试，由于不能跨架构模拟，故不再赘述。

本文记录了在 Windows 上使用 QEMU 可以在 X86 上模拟 ARM64 的过程，并测试安装了 Kylin-Desktop-V10-SP1-2503 和 Ubuntu-20.04.4-Server，均可以正常启动和运行。

## 资源准备

- 操作系统镜像：

    Kylin-Desktop-V10-SP1-2503-Release-20250430-ARM64.iso

    [ubuntu-20.04.4-live-server-arm64.iso](https://old-releases.ubuntu.com/releases/20.04/ubuntu-20.04.4-live-server-arm64.iso)

- QEMU 安装包：

    [QEMU Binaries for Windows (64 bit) 20250422](https://qemu.weilnetz.de/w64/)

- ARM 架构的 BIOS 固件：

    [QEMU_EFI.fd](https://releases.linaro.org/components/kernel/uefi-linaro/16.02/release/qemu64/)

    该文件是 ARM64 虚拟机在 QEMU 模拟环境中的引导固件，用于替代传统 BIOS。它负责初始化虚拟硬件并加载操作系统内核，是启动 ARM 架构虚拟机的必备组件。

## 过程记录

### 安装 QEMU

一路 next 即可

### 创建虚拟磁盘

```bat
qemu-img create -f qcow2 "E:\Virtual Machines\qvm\kylin_arm64.qcow2" 150G
# 或者使用 raw 格式
qemu-img create -f raw "E:\Virtual Machines\qvm\kylin_arm64.img" 150G
```

```bat
qemu-img create -f qcow2 "E:\Virtual Machines\qvm\ubuntu_arm64.qcow2" 150G
```

在创建虚拟磁盘时也可以使用 raw 格式，它会一次性分配所有空间，理论上它会有更好的性能，使用 qcow 或者 qcow2 格式的话，空间按需分配，可以支持压缩和加密。

### 安装 ARM64 系统

```bat
qemu-system-aarch64.exe -m 16G -cpu cortex-a72 --accel tcg,thread=multi -smp 8 -M virt -bios "E:\Virtual Machines\qvm\QEMU_EFI.fd" -rtc base=utc -display sdl -device VGA -device nec-usb-xhci -device usb-tablet -device usb-kbd -drive if=virtio,file="E:\Virtual Machines\qvm\kylin_arm64.qcow2",id=hd0,format=qcow2,cache=writethrough,discard=unmap -drive if=none,file="D:\Downloads\Kylin-Desktop-V10-SP1-2503-Release-20250430-ARM64.iso",id=cdrom,media=cdrom -device virtio-scsi-device -device scsi-cd,drive=cdrom
```

```bat
qemu-system-aarch64.exe -m 16G -cpu cortex-a72 --accel tcg,thread=multi -smp 8 -M virt -bios "E:\Virtual Machines\qvm\QEMU_EFI.fd" -rtc base=utc -display sdl -device VGA -device nec-usb-xhci -device usb-tablet -device usb-kbd -drive if=virtio,file="E:\Virtual Machines\qvm\ubuntu_arm64.qcow2",id=hd0,format=qcow2,cache=writethrough,discard=unmap -drive if=none,file="D:\Downloads\ubuntu-20.04.4-live-server-arm64.iso",id=cdrom,media=cdrom -device virtio-scsi-device -device scsi-cd,drive=cdrom
```

有时会遇到键盘输入字母失效，可将`-display sdl`改为`-display gtk`，兼容性更好。

### 启动虚拟机

为了简化虚拟机的启动，可以把下面的命令保存为批处理文件，每次双击运行就可以启动虚拟机。

```bat
@echo off
"C:\Program Files\qemu\qemu-system-aarch64.exe" -m 16G -cpu cortex-a72 --accel tcg,thread=multi -smp 8 -M virt -bios "E:\Virtual Machines\qvm\QEMU_EFI.fd" -rtc base=utc -display sdl -device VGA -device nec-usb-xhci -device usb-tablet -device usb-kbd -drive if=virtio,file="E:\Virtual Machines\qvm\kylin_arm64.qcow2",id=hd0,format=qcow2,cache=writeback,discard=unmap -net user,hostfwd=tcp::2222-:22 -net nic
```

```bat
@echo off
"C:\Program Files\qemu\qemu-system-aarch64.exe" -m 16G -cpu cortex-a72 --accel tcg,thread=multi -smp 8 -M virt -bios "E:\Virtual Machines\qvm\QEMU_EFI.fd" -rtc base=utc -display sdl -device VGA -device nec-usb-xhci -device usb-tablet -device usb-kbd -drive if=virtio,file="E:\Virtual Machines\qvm\ubuntu_arm64.qcow2",id=hd0,format=qcow2,cache=writeback,discard=unmap -net user,hostfwd=tcp::2222-:22 -net nic
```

磁盘参数`cache=writethrough`改为`cache=writeback`，性能会提升；

`-net user,hostfwd=tcp::2222-:22 -net nic`将虚拟机的 22 端口映射到主机的 2222 端口。这里使用了 QEMU 的用户网络模式，另外还可以安装 TAP-Windows Adapter， 配置好后，命令可以改为`-net nic -net tap,ifname=tap0`。

## 参考链接

<https://www.cnblogs.com/mylibs/p/kylin-arm64-with-qemu-on-windows.html>

<https://zhuanlan.zhihu.com/p/683687380>

<https://chlience.com/QemuArm64/>
