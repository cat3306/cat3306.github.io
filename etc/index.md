# linux etc


<!--more-->

# 环境变量

总的来说，环境变量分为系统级和用户级
可以通过 `set` 来查看系统的环境变量，`source ?` 来更新环境变量(source /etc/profile)

- 系统级环境变量:每个登录到系统的用户都能够读取到的系统环境变量
- 用户级环境变量:每个登录到系统的用户只能读取属于自己的用户级别的环境变量

### 系统级

1. ```bash
   /etc/profile #环境变量配置
   ```

   在系统启动后第一个用户登录时运行，并且会运行 `/etc/profile.d` 所有的环境变量shell，变量将应用于登录到系统的所有的用户，官方有句话

   ```bash
   # It's NOT a good idea to change this file unless you know what you
   # are doing. It's much better to create a custom.sh shell script in
   # /etc/profile.d/ to make custom changes to your environment, as this
   # will prevent the need for merging in future updates.
   ```

   翻译过来就是在说，要设置环境变量，推荐在/etc/profile.d/ 创建一个shell脚本，而不是直接修改/etc/profile

   ```bash
   #shell 脚本设置环境变量待补充
   ```

2. ```bash
   /etc/bashrc #ubuntu和debian 中是 /etc/bash.bashrc
   ```

   在bash shell打开时运行，修改改文件配置的环境变量将会影响所有的用户使用bash shell

   同样的官方提示

   ```bash
   # System wide functions and aliases
   # Environment stuff goes in /etc/profile
   
   # It's NOT a good idea to change this file unless you know what you
   # are doing. It's much better to create a custom.sh shell script in
   # /etc/profile.d/ to make custom changes to your environment, as this
   # will prevent the need for merging in future updates.
   ```

### 用户级

1. ```bash
   ~/.profile #推荐使用
   ```

   当用户登录执行，每个用户都可以使用该文件来配置专属自己的的shell信息

2. ```
   ~/.bashrc 
   ```

   当用户登录时以及每次打开新的shell时改文件被读取

3. ```
   ~/.bash_profile 或 ~./bash_login
   ```

   ubuntu 如果有其中的一个文件存在的话, 当启动的是一个 登录shell时，Bash 会执行该文件而不会执行~/.profile ；
   如果两个文件都存在的话，Bash 将会优先执行~/.bash_profile 而不是~/.bash_login ；
   然而, 默认情况下，这些文件不会影响图形会话

4. ```
   ~/.bash_logout
   ```

   当每次退出系统(退出bash shell)时执行该文件

### 总结

执行顺序是

```
 /etc/profile
 ~/.bash_profile | ~/.bash_login | ~/.profile
 ~/.bashrc
 /etc/bashrc
 ~/.bash_logout
```

# 查看常用的系统配置

## 查看机器配置

- 查看内存容量

  ```bash
  cat /proc/meminfo | grep MemTotal
  ```

- 查看cpu型号

  ```bash
   cat /proc/cpuinfo | grep 'model name' |uniq
  ```

- 查看cpu个数

  ````bash
  cat  /proc/cpuinfo | grep "physical id" | uniq | wc -l
  ````

  ```bash
  # 查看内核/操作系统/CPU信息的linux系统信息  
  uname -a 
  
  # 查看操作系统版本  
  head -n l /etc/issue 
  
  # 查看CPU信息  
  cat /proc/cpuinfo 
  
  # 查看计算机名的linux系统信息命令  
  hostname 
  
  # 列出所有PCI设备   
  lspci -tv 
  
  # 列出所有USB设备的linux系统信息命令  
  lsusb -tv 
  
  # 列出加载的内核模块   
  lsmod 
  
  # 查看环境变量资源  
  env 
  
  # 查看内存使用量和交换区使用量   
  free -m 
  
  # 查看各分区使用情况  
  df -h 
  
  # 查看指定目录的大小   
  du -sh 
  
  # 查看内存总量
  grep MemTotal /proc/meminfo   
  
  # 查看空闲内存量  
  grep MemFree /proc/meminfo  
  
  # 查看系统运行时间、用户数、负载  
  uptime 
  
  # 查看系统负载磁盘和分区   
  cat /proc/loadavg 
  
   # 查看挂接的分区状态  
  mount | column -t
  
  # 查看所有分区  
  fdisk -l  
  
  # 查看所有交换分区  
  swapon -s 
  
  # 查看磁盘参数(仅适用于IDE设备) 
  hdparm -i /dev/hda   
  
  # 查看启动时IDE设备检测状况网络  
  dmesg | grep IDE 
  
  # 查看所有网络接口的属性   
  ifconfig 
  
  # 查看防火墙设置  
  iptables -L 
  
  # 查看路由表   
  route -n 
  
  # 查看所有监听端口  
  netstat -lntp 
  
  # 查看所有已经建立的连接   
  netstat -antp 
  
  # 查看网络统计信息进程  
  netstat -s 
  
  # 查看所有进程   
  ps -ef 
  
  # 实时显示进程状态用户  
  top 
  
  # 查看活动用户   
  w 
  
  # 查看指定用户信息  
  id 
  
  # 查看用户登录日志   
  last 
  
  # 查看系统所有用户 
  cut -d: -f1 /etc/passwd  
  
  # 查看系统所有组   
  cut -d: -f1 /etc/group 
  
  # 查看当前用户的计划任务服务  
  crontab -l 
  
  # 列出所有系统服务   
  chkconfig –list 
  
  # 列出所有启动的系统服务程序  
  chkconfig –list | grep on 
  
   # 查看所有安装的软件包   
  rpm -qa
  
  #查看CPU相关参数的linux系统命令  
  cat /proc/cpuinfo 
  
  #查看linux硬盘和分区信息的系统信息命令   
  cat /proc/partitions
  
  #查看linux系统内存信息的linux系统命令 
  cat /proc/meminfo 
  
  #查看版本，类似uname -r  
  cat /proc/version  
  
  #查看设备io端口  
  cat /proc/ioports 
  
  #查看中断   
  cat /proc/interrupts 
  
  #查看pci设备的信息  
  cat /proc/pci
  
  #查看所有swap分区的信息  
  cat /proc/swaps
  
  fdisk -l | grep Disk
  ```

  # ubuntu server 设置静态IP
  
  `gateway4` has been deprecated, use default routes instead
  
  老版本的gateway4不支持了，采用下面方式
  
  ```yaml
  routes:
        - to: default
          via: 192.168.1.1
  ```
  
  
  
  `vim /etc/netplan/00-installer-config.yaml`
  
  ```yaml
  # This is the network config written by 'subiquity'
  network:
    ethernets:
      enp3s0f1:
        dhcp4: no
        addresses: [192.168.1.7/24]
        routes:
        - to: default
          via: 192.168.1.1
        nameservers:
          addresses: [8.8.8.8,8.8.4.4]
    version: 2
  ```
  
  - addresses 设置静态IP和网段
  - routes 是网关
  - dhcp4: no 不采用动态获取IP


