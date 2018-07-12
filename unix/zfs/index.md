---
layout: bootstrap
title: ZFS
type: page
nav: pages
---

# ZFS

## Installation

```
$ sudo yum update
$ sudo yum install kernel-devel
$ sudo reboot
$ sudo yum install epel-release
$ sudo yum install dkms
$ sudo yum localinstall --nogpgcheck https://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-2.noarch.rpm
$ sudo yum localinstall --nogpgcheck http://archive.zfsonlinux.org/epel/zfs-release.el7.noarch.rpm
$ sudo yum install zfs
$ sudo zpool status
# 正常にインストールされなかったら実行
$ sudo dkms install -m spl -v x.x.x
$ sudo yum remove dkms zfs
$ sudo yum install dkms zfs
# カーネルアップデートでエラーになったとき
$ dkms status
$ sudo dkms remove zfs/x.x.x --all
$ sudo dkms --force install zfs/x.x.x
$ sudo yum install zfs
$ sudo modprobe zfs
```

## Configure

```
# メモリ使用量上限
$ su
$ echo "options zfs zfs_arc_max=34359738368" >> /etc/modprobe.d/zfs.conf
$ echo "34359738368" > /sys/module/zfs/parameters/zfs_arc_max
$ cat /proc/spl/kstat/zfs/arcstats | grep c_max
# チューニング
$ echo "options zfs zfs_resilver_delay=0" >> /etc/modprobe.d/zfs.conf
$ echo "0" > /sys/module/zfs/parameters/zfs_resilver_delay # default 2
$ echo "options zfs zfs_scrub_delay=1" >> /etc/modprobe.d/zfs.conf
$ echo "1" > /sys/module/zfs/parameters/zfs_scrub_delay # default 4
$ exit
# ZIL, L2ARC用パーティションを作成
$ sudo parted --list
$ sudo fdisk
> p
> n
> p
>
> t
>
> 8e
> w
# 暗号化
$ sudo yum install cryptsetup
$ sudo cryptsetup --hash=sha512 --cipher=aes-xts-plain64 --key-size=256 luksFormat /dev/sdb
$ sudo cryptsetup --hash=sha512 --cipher=aes-xts-plain64 --key-size=256 luksFormat /dev/sdc
$ sudo cryptsetup --hash=sha512 --cipher=aes-xts-plain64 --key-size=256 luksFormat /dev/sdd
$ sudo cryptsetup luksOpen /dev/sdb zfs-vol-1
$ sudo cryptsetup luksOpen /dev/sdc zfs-vol-2
$ sudo cryptsetup luksOpen /dev/sdd zfs-vol-3
$ sudo cryptsetup --hash=sha512 --cipher=aes-xts-plain64 --key-size=256 luksFormat /dev/sda3
$ sudo cryptsetup --hash=sha512 --cipher=aes-xts-plain64 --key-size=256 luksFormat /dev/sda4
$ sudo cryptsetup luksOpen /dev/sda3 zfs-log
$ sudo cryptsetup luksOpen /dev/sda4 zfs-cache
$ sudo ls /dev/mapper/
# プールを作成
$ sudo zpool create -f -o ashift=12 -m <mountpoint> <zpool> raidz2 zfs-vol-{1,2,3}
$ sudo zpool add <zpool> log zfs-log
$ sudo zpool add <zpool> cache zfs-cache
$ sudo zpool set listsnapshots=on <zpool>
$ sudo zpool status
#$ sudo zfs set recordsize=128k <zpool>
#$ sudo zfs set dedup=on <zpool>
#$ sudo zfs set snapdir=visible <zpool>
$ sudo zfs set compression=lz4 <zpool>
$ sudo zfs set atime=off <zpool>
$ sudo zfs set relatime=on <zpool>
$ sudo zfs set xattr=sa <zpool>
$ sudo zfs get all
$ sudo zdb -S <zpool>
$ sudo zpool iostat -v <zpool>
$ sudo reboot
$ sudo zfs list
$ sudo zfs mount -a
```

## Status

```
$ sudo zpool status
$ sudo zpool list
$ sudo zpool iostat -v
$ sudo zpool iostat -v 2
$ sudo smartctl -a /dev/sda
$ sudo zfs list -r -t filesystem <zpool>
$ sudo zfs list -r -t snapshot -o name <zpool>
$ sudo zfs list -r -t all -o space <zpool>
$ sudo zfs get used | grep -v @
$ sudo zfs get usedbysnapshots | grep -v @
$ sudo zfs get available | grep -v @
$ sudo zfs get compressratio | grep -v @
$ sudo zdb -S <zpool>
$ sudo zdb -DD <zpool>
```

## Import

```
$ sudo zpool import
$ sudo zpool import <zpool>
```

```
$ sudo zpool import -D
$ sudo zpool import -F
```

## Export

```
$ sudo zpool export <zpool>
```

## Snapshot

```
$ sudo zfs snapshot <zsnapshot>
$ sudo zfs snapshot -r <zsnapshot>
```

## Rollback

```
$ sudo zfs rollback <zsnapshot>
```

## Cleanup

```
$ sudo zfs list -r -t snapshot -o name <zfilesystem> | grep <zvolume>@ | head -n -70 | xargs -r -n 1 sudo zfs destroy
```

## Branch

```
$ sudo zfs clone <zsnapshot> <zvolume>/branch/snap
```

## Backup

```
$ sudo zfs send -R <zsnapshot> | ssh root@localhost zfs recv -v -d <zfilesystem>
$ sudo zfs send -R -i <zsnapshot> <zsnapshot> | ssh root@localhost zfs recv -v -d <zfilesystem>
```

```
$ sudo zfs send -R <zsnapshot> | sudo tee /mnt/backup/<zfilesystem>.snap.zfs.img > /dev/null
$ sudo zfs send -R <zsnapshot> | gzip -c | sudo tee /mnt/backup/<zfilesystem>.snap.zfs.img.gz > /dev/null
```

```
$ sudo zfs list -r -t snapshot -o name <zfilesystem> | grep @ | tail -n 1 | xargs -n 1 zfs send -R | sudo tee /mnt/backup/<zfilesystem>.zfs.img > /dev/null
```

## Restore

```
$ ssh root@localhost zfs send -R <zsnapshot> > zfs recv -v -d <zfilesystem>
```

```
$ sudo cat /mnt/backup/<zfilesystem>.snap.zfs.img | sudo zfs recv -v -d <zfilesystem>
$ sudo gunzip -c /mnt/backup/<zfilesystem>.snap.zfs.img.gz | sudo zfs recv -v -d <zfilesystem>
```

## Auto snapshot

https://github.com/zfsonlinux/zfs-auto-snapshot

## Schedule scrub

```
# アクセスのないプール以外では不要
$ sudo crontab -e
> 0 1 * * * zpool scrub <zpool>
```

## Recovery

```
$ sudo zdb -cc -b -m -e <zpool>
```

```
$ sudo zdb -AAA -e <zpool>
```
