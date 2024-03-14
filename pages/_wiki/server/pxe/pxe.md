---
permalink: /wiki/server/pxe/
---

# 南阳理工学院网络启动服务

pxe.nyist.edu.cn 由计算机与信息化处理协会共同维护，旨在为校园网用户提供各种网络启动服务。现在支持引导 Debian、Ubuntu、Arch Linux、CentOS 等常见 Linux/UNIX 发行版安装镜像或 LiveCD，同时还提供 Clonezilla、GParted Live 等实用系统维护工具。

如果遇到使用问题，请邮件联系 cips AT nyist.edu.cn。

## 新版网络启动服务

基于 GRUB 的新版网络启动服务支持**传统 PXE 模式**和**UEFI 模式**的网络启动。代码位于 https://github.com/NYIST-CIPS/netboot

校内 DHCP 服务会自动推送网络启动配置，只要在 BIOS 设置中开启网络启动就可以了。

### 从本地 GRUB2 加载（UEFI）

如果您的机器是 UEFI 模式启动的、UEFI 固件带有网络支持，并已经安装有 GRUB2，则可以在 GRUB 命令行中直接加载网络启动菜单：

```shell
insmod efinet http
net_bootp
configfile (http,cips.nyist.edu.cn)/cips.nyist.edu.cn.ipxe
```

也可以制作一个 EFI 可执行文件，放在 U 盘中方便部署。在 Linux 系统中，把前面的命令行保存到文件 `grub.cfg` ，然后运行命令：

```shell
grub-mkstandalone --compress=xz -O x86_64-efi --locales= --fonts= --themes= -o grub.efi "boot/grub/grub.cfg=./grub.cfg"
```

生成的 `grub.efi` 文件就可以在 UEFI shell 中直接运行，或者放到 FAT 格式的 U 盘中的 `/EFI/BOOT/bootx64.efi` 路径，使 UEFI 固件自动加载它。

还可以接着用下面命令制作一个 FAT 软盘镜像，有些服务器的 IPMI 支持远程加载软盘镜像，这样就可以方便地远程维护服务器了：

```shell
truncate -s 1474560 floppy.img
mformat -f 1440 :: -i floppy.img
mmd '::/EFI' '::/EFI/BOOT' -i floppy.img
mcopy grub.efi '::/EFI/BOOT/bootx64.efi' -i floppy.img
```

### 从 iPXE 加载（传统 PXE）

用 [iPXE](https://ipxe.org/) 脚本或者命令行模式执行以下命令：

```shell
dhcp
set 210:string http://pxe.nyist.edu.cn/boot/tftp/
chain ${210:string}pxelinux.0
```
