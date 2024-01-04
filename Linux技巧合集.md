# 挂载

在linux中，万物皆文件。挂载就是将另一个文件、设备甚至远程主机的某个共享文件设置成为本机的某个文件，这样你在读写这个文件时，就是对那些设备进行操作。

挂载的命令是mount

```
mount [-t vfstype] [-o options] device dir
```

- `[-t vfstype]`：是你想要挂载的类型。常用的有这些

  - 光盘或iso镜像：iso9660
  - FAT16文件系统：msdos
  - FAT32文件系统：vfat
  - NTFS文件系统：ntfs
  - Windows文件网络共享：smbfs
  - UNIX(Linux)文件网络共享：nfs

  通常来说，mount会自动选择正确的类型，因此比较少指定这个参数，就像你在window上插入不同文件系统的u盘也能被正确识别一样。

- `[-o options]`：指定挂载参数，你可以指定多个参数，使用`,`分隔

  - 挂载成硬盘：loop
  - 以只读方式挂载：ro
  - 以读写方式挂载：rw
  - 指定访问文件系统所用字符集：iocharset=XXX，比如iocharset=utf8
  - 如果挂载文件网络系统，还有username，password等参数

- `device`：要挂载的设备，在Linux下，物理硬件通常会被放在/dev目录下

- `dir`：挂载到本地的某个文件，也叫挂载点，挂载点通常放在/mnt目录下。请注意，你可以指定已经存在的某个文件，这种方式挂载后，该文件只能被用于操作挂载的设备，而文件本身的内容你无法操作。取消挂载后，你才能操作原文件。

取消挂载时，你只需要撤销挂载点即可，dir是你创建的挂载点

```
umount dir
```



比如，你需要挂载一个镜像iso文件：

```
mkdir /mnt/vcdrom	# 创建挂载点
mount -o loop -t iso9660 /path/to/file.iso /mnt/vcdrom
```

这样操作/mnt/vcdrom就可以访问光盘镜像文件mydisk.iso里的所有文件了。



挂载U盘等设备，你可以先使用

```
sudo fdisk -l
```

查看已连接的物理设备，插入U盘后，再使用该命令得到U盘的设备地址，比如/dev/sda6

创建挂载点并挂载

```
mkdir /mnt/myusbdisk
mount /dev/sda6 /mnt/myusbdisk
```



挂载Windows文件网络系统

你需要确保windows主机已经开启SMB服务，并且，linux客户端需要安装samba软件

```
mkdir –p /mnt/win_smb
mount -t smbfs -o username=用户名,password=密码 //12.23.34.45/c$ /mnt/win_smb
```



挂载Linux文件网络系统

你需要确保Linux主机已经开启NFS服务

```
mkdir –p /mnt/linux_nfs
mount -t nfs -o rw 12.23.34.45:/path/to/file /mnt/linux_nfs
```





## Linux开启NFS服务

安装NFS服务

```
sudo apt install nfs-kernel-server
```

修改配置文件

```
sudo vim /etc/exports
```

你可以在此添加映射点，格式为

```
/nfs_share  *(insecure,rw,sync,no_root_squash,no_subtree_check)
```

- `/nfs_share`：即为你新建的共享目录

- `*`：可在此处指定某网段内IP可访问，例如：192.168.18.0，或在此处填写设备名表示某设备可访问，不过你需要在hosts里指定设备名称的IP。`*`表示所有IP都可以访问。

- ro                      只读访问

  rw                      读写访问

  sync                    所有数据在请求时写入共享

  async                   NFS在写入数据前可以相应请求

  secure                  NFS通过1024以下的安全TCP/IP端口发送

  insecure                NFS通过1024以上的端口发送

  nolock					不开启NFS锁，某些老设备可能不支持NFS锁

  wdelay                  如果多个用户要写入NFS目录，则归组写入(默认)

  no_wdelay               如果多个用户要写入NFS目录，则立即写入，当使用async时，无需此设置。

  hide                    在NFS共享目录中不共享其子目录

  no_hide                 共享NFS目录的子目录

  subtree_check           如果共享/usr/bin之类的子目录时，强制NFS检查父目录的权限(默认)

  no_subtree_check        和上面相对，不检查父目录权限

  all_squash              共享文件的UID和GID映射匿名用户anonymous，适合公用目录。

  no_all_squash           保留共享文件的UID和GID(默认)

  root_squash             root用户的所有请求映射成如anonymous用户一样的权限(默认)

  no_root_root_squash           root用户具有根目录的完全管理访问权限

  anonuid=xxx             指定NFS服务器/etc/passwd文件中匿名用户的UID
  
  anongid=xxx             指定NFS服务器/etc/passwd文件中匿名用户的GID

使配置生效

```
sudo exportfs -r
```

启动NFS服务器

```
sudo systemctl start nfs-server.service
```

开机自启

```
sudo systemctl enable nfs-server.service
```

查看挂载情况

```
showmount -e localhost
```

此外，你还需修改防火墙。

取消挂载只需要修改配置文件再重新应用即可