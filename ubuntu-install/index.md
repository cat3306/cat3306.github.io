# ubuntu install


<!--more-->

# 官网

镜像文件 [官网下载](https://cn.ubuntu.com/download)

# windos启动盘

制作启动盘工具 [官网](http://rufus.ie/)  [工具地址](https://github.com/pbatard/rufus)

![](/img/ubuntu/1.png)

一键开始将u盘制作成启动盘

# mac制作启动盘

1. 从官网下载ios镜像

2. .`iso`转换成.`dmg`
   ```bash
   hdiutil convert -format UDRW -o ubuntu.iso ubuntu-22.04.1-live-server-amd64.iso
   ```

3. 插入U盘，并且umount掉

   ```
   $ diskutil list
   /dev/disk2 (external, physical):
      #:                       TYPE NAME                    SIZE       IDENTIFIER
      0:      GUID_partition_scheme                        *31.0 GB    disk2
      1:       Microsoft Basic Data                         1.5 GB     disk2s1
      2:                        EFI ESP                     4.3 MB     disk2s2
      3:       Microsoft Basic Data                         307.2 KB   disk2s3
      4:           Linux Filesystem                         29.5 GB    disk2s4
      
      
   ```

   ```
    diskutil unmountDisk /dev/disk2
   ```

4. dd 写入数据，在写入之前需要把后缀改回来
   ```bash
   mv ubuntu.iso.dmg ubuntu.iso
   ```

   ```bash
   sudo dd if=./ubuntu.iso of=/dev/rdisk4 bs=1m
   ```

   

# ubuntu desktop安装

将启动盘插入电脑，联想电脑 fn+f12 进入bios，用启动盘启动安装系统

系统语言设置英文

## 语言与输入法

![](/img/ubuntu/7.png)

![](/img/ubuntu/8.png)

## 截图工具



```bash
$ sudo apt-get install flameshot
$ flameshot -h
Usage: flameshot [flameshot-options] [arguments]

Per default runs Flameshot in the background and adds a tray icon for configuration.

Options:
  -h, --help     Displays this help
  -v, --version  Displays version information

Arguments:
  gui            Start a manual capture in GUI mode.
  screen         Capture a single screen.
  full           Capture the entire desktop.
  launcher       Open the capture launcher.
  config         Configure flameshot.

```

## 快捷键



![](/img/ubuntu/3.png)

![](/img/ubuntu/4.png)

![](/img/ubuntu/5.png)

软件包下载源

![](/img/ubuntu/6.png)

# 图形界面

关闭

````bash
$ sudo systemctl set-default multi-user

$ sudo reboot
````

打开

```bash
$ sudo systemctl set-default graphical
```

# 设置合上盖子不休眠

```bash
$ sudo vim /etc/systemd/logind.conf


#HandleLidSwitch=suspend 修改成 HandleLidSwitch=ignore
```



# 分区挂载

- 查看磁盘信息

```bash
df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           784M  2.0M  782M   1% /run
/dev/sdb2       439G  7.7G  409G   2% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
/dev/sda1        96M   31M   66M  32% /boot/efi
/dev/sda3       223G  1.6G  222G   1% /root/minio
tmpfs           784M   72K  784M   1% /run/user/127
tmpfs           784M   56K  784M   1% /run/user/0
```

- 格式化硬盘

```bash
mkfs.xfs /dev/sda3
```

- 挂载分区

```bash
mount /dev/sda3 /root/minio
```

- 查看磁盘分区的UUID

```bash
blkid

/dev/sda3: UUID="7b54b8d4-0c82-4459-8b37-c317b2c2cac7" BLOCK_SIZE="512" TYPE="xfs" PARTLABEL="Basic data partition" PARTUUID="c1d4bbc4-11cf-46f5-bac4-105153e22df3"
```

- 修改`fstab`开机自动挂载

```bash
vim /etc/fstab

# 加入下面内容
UUID="7b54b8d4-0c82-4459-8b37-c317b2c2cac7" /root/minio xfs defaults 0 1
```








