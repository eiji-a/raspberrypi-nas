# Raspberry Pi NAS計画

## Ubuntuパッケージの導入

* アップデート

```
$ sudo apt update
  :
$ sudo apt upgrade
  :
```

* Samba関連

```
$ sudo apt install samba
  :
$ samba -V
Version 4.13.17-Ubuntu
$ sudo systemctl status smbd
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-01-09 04:23:05 UTC; 54s ago
  :
```

* exFAT関連

```
$ sudo apt -y install exfat-fuse exfat-utils
  :
```

## 設定

* Sambaユーザ作成

```
$ sudo useradd samba
$ sudo pdbedit -a samba
new password:               <- 'samba'
retype new password:        <- 'samba'
  :
$ sudo pdbedit -L
samba:1001:
```

* 設定ファイル

/etc/samba/smb.conf を編集する。[global]の追記と共有フォルダの設定を追記する。

```
  :
[global]
  :
max protocol = SMB2
  :
## 最後に追加

[ushare]
comment = USB-HV
path = /mnt/hvd1
writable = yes
browseable = yes
valid user = samba
create mode = 0777
directory mode = 0777
unix extensions = no
```

## 外付けDiskのマウント

* マウント先ディレクトリの作成

```
$ cd /mnt
$ sudo mkdir hvd1
$ sudo chown samba hvd1
```

* 外付けディスクの確認

```
$ sudo fdisk -l
  :
Disk /dev/sda: 7.28 TiB, 8001563222016 bytes, 15628053168 sectors
Disk model: Disk            
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0BDC1331-5858-9090-8081-828310111213

Device     Start         End     Sectors  Size Type
/dev/sda1   2048 15628048946 15628046899  7.3T Microsoft basic data
```

'/dev/sda1'が対象のパーティションとわかる。

* マウント

```
$ sudo mount /dev/sda1 /mnt/hvd1/ --type=exfat --options=rw
FUSE exfat 1.3.0
```

## 外付けディスクの自動マウントおよびSambaの自動起動


* Samba自動起動設定

```
$ sudo systemctl enable smbd
Synchronizing state of smbd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable smbd
```

* 自動マウント設定

blkidにてUUIDを確認する。

```
$ sudo blkid
  :
/dev/sda1: LABEL="HV01" UUID="6383-22AB" TYPE="exfat" PARTLABEL="Basic data partition" PARTUUID="22f13fb7-0c7f-11ed-be90-c19766c72038"
```

/etc/fstab に追記する。

```
  :
UUID=6383-22AB /mnt/hvd1 exfat 	defaults	0	0
```

---
