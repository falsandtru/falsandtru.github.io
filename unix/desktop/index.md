---
layout: bootstrap
title: Desktop
type: page
nav: pages
---

# Desktop

## CentOS7

### Trackpoint

```
$ xset m 10 2
```

### 日本語キーボード配列

```
$ localectl set-keymap jp106
$ sed -i 's/vconsole.keymap=us/vconsole.keymap=jp106/' /etc/default/grub
$ grub2-mkconfig -o /boot/grub2/grub.cfg
$ reboot
```

### ネットワーク

```
# NICをカスタムNATに差し直す
$ vi /etc/sysconfig/network-scripts/ifcfg-en*
$ systemctl restart network
```

*ifcfg-en*

```
TYPE="Ethernet"
DEVICE=<DEV>
HWADDR=<MAC>
NM_CONTROLLED="yes"
ONBOOT="yes"
BOOTPROTO="static"
IPADDR=192.168.9.100
NETMASK=255.255.255.0
GATEWAY=192.168.9.254
DNS1=192.168.9.254
```

### GUI

```
# http://www.server-world.info/query?os=CentOS_7&p=x
yum -y groups install "GNOME Desktop"
startx
```

### グラフィックドライバ

Download the driver of "NVIDIA-Linux-x86_64-xxx.xx.run"

> http://www.nvidia.co.jp/object/unix-jp.html

Configure

> http://blog.livedoor.jp/rootan2007/archives/52090548.html

```
$ yum install kernel-devel gcc
$ mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img
$ dracut --omit-drivers nouveau /boot/initramfs-$(uname -r).img $(uname -r)
$ vi /etc/modprobe.d/modprobe.conf
> blacklist nouveau
$ vi /etc/modprobe.d/nouveau_blacklist.conf
> blacklist nouveau
$ reboot
$ telinit 3
$ lsmod | grep nouveau # nothing
$ sh Downloads/NVIDIA-Linux-x86_64-xxx.xx.run
```

> http://forums.fedoraforum.org/showthread.php?t=283467

```
$ vi /etc/X11/xorg.conf
```

*xorg.conf*

```
Section "Device"
	Identifier  "Videocard0"
	Driver      "nvidia"
EndSection
```
