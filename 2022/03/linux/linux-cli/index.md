# linux cli


<!--more-->

## shell

- ```bash
  #查看存在的shell
  cat /etc/shells
  ```

- ```bash
  #查看正在使用的shell
  echo $SHELL
  ```

- ```bash
  #切换shell
  chsh -s /usr/bin/fish
  ```

## adduser

adduser 和 useradd 是一个意思的命令，用于增加一个linux系统的用户账号添加
参数如下：

```
-c comment 指定一段描述，保存在 /etc/passwd 的备注栏
-d 指定用户的主目录，如果目录不存在可以用-m，创建
-g 用户组 指定用户所属的用户组
-s shell，指定登录时用的shell
-u 指定用户的id号
-r 创建系统账号
-m 自动创建用户的家目录
-M 不要自动创建用户的家目录
-e 指定用户的失效时间，日期格式为MM/DD/YY。默认永久有效
-G：指定用户所属的附加群组。
```



```bash
useradd -u 520 -d /usr/myhome  -g users -m  myhome
#创建一个myhome用户uid 520 家目录是/usr/myhome,-m是自动创建家目录
```

## bc 

通用计算器

* "+" 加法

* "-" 减法

* "*" 乘法

* "/" 除法

* "^" 指数

* "%" 余数 

  ```bash
  $ echo "15+5" | bc
  20
  ```

  ```bash
  $ echo "scale=2;(2.777-1.4744)/1" | bc
  1.30 ## scale 设置小数点
  ```

  ```bash
  $ echo "obase=2;ibase=10;111" | bc 
  1101111  #obase=2(输出二进制),ibase=10(输入十进制),110是输入参数
  ```

## cat

命令用于连接文件并打印到标准输出设备上 

```bash
#对文件由1开始编号输出
$ cat -n file_name 
```

```bash
#文件1的类容加上行号输入文件2
$ cat -n file_name_1 > file_name_2 
```

```bash
#对文件由1开始编号输出,空白行不做处理
$ cat -b file_name
```

```bash
#将文件1，文件2(加上行号，空白不加行号)追加到文件3
$ cat -b file_name_1 file_name_2 >> file_name_3 
```

```bash
#清空文件内容
$ cat /dev/null > file_name 
```

## chgrp

 更改文件或者目录的所属群组

```shell
#查看所有群组
$ groups 
```

```shell
#更改文件或者目录群组 并打印执行过程
$ chgrp -v group_name file_name 
```

```shell
#递归更改文件或者目录群组 并打印执行过程
$ chgrp -v -R group_name file_name 
```

## chmod

````
#“a”表示所有用户，“u”表示创建者、“g”表示创建者同组用户、“o”表示其他用户；“+”表示添加权限，“-”表示取消权限；“r”表示读权限、“w”表示写权限、“x”表示写权限。
#当前文件夹下所有的文件追加执行权限，u 表示创建者

chmod u+x ./*
````

![](/img/linux/1.png)

## chown 

chown(change owner) 设置文件或文件夹的所有者和关联组，该命令只能root执行

```bash
#设置所有者为root
$ chown root /var/log/go_log/on.log 
```

```bash
#设置所有者和所属群组为root
$ chown root:root /var/log/go_log/on.log 
```

```bash
#设置/var/log/go_log 下所有子文件和文件夹的所有者为joker，所属的组joker
$ chown -R joker:joker /var/log/go_log  
```

```bash
#设置当前文件下所有子文件和文件夹的所有者为joker，所属的组joker
$ chown -R joker:joker *  
```

## cut 

cut用于截取每行的类容

- -b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
- -c ：以字符为单位进行分割。
- -d ：自定义分隔符，默认为制表符。
- -f ：与-d一起使用，指定显示哪个区域。
- -n ：取消分割多字节字符。仅和 -b 标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的
  范围之内，该字符将被写出；否则，该字符将被排除

配合grep可提取关键字符串，常用于日志提取，列如下面这个日志我想提取user_id:xxx,routeId:xxx

```
2021-04-05 09:08:17.155 INFO 1eb829c [go:4661107/730] ▶ api_server.go:3110 PostSportOss PostSportOss ok,user_id:07f438bb-28b5-4f37-a6e7-8312ff90cabc,ossKey:sport/07f438bb-28b5-4f37-a6e7-8312ff90cabc/2021-04-05T09.08.16/4B3cPIr7b2R3q1iY9z,routeId:074ae03c-95ab-11eb-a48a-0178905c8dc0,userAgent:map[device_type:iphone iner_version:4470 ip: platform:0 platform_version:13.6 platfrom_version: version:9.48.1]
```

可用和grep搭配

```
grep "PostSportOss ok" gps_data_server.log.2021-04-05* | grep "platform:0" | cut -d ',' -f 2,4 

user_id:07f438bb-28b5-4f37-a6e7-8312ff90cabc,routeId:074ae03c-95ab-11eb-a48a-0178905c8dc0
```

cut -d 表示用自定义分割符 `,` -f 表示哪个域，1表示：以`,` 分割的第一个子串,2表示分割的第二个子串，例子中第二个子串是`user_id:07f438bb-28b5-4f37-a6e7-8312ff90cabc` 依次类推

## du 

(全称:disk usage)

- 参数说明

  ```
  -a或-all 显示目录下所有的文件,文件夹
  -h或--human-readable 用k,m,g显示，人类读取友好
  -s或--summarize 仅显示统计
  ```

- 例子

  ```bash
  $ du -a -h | grep /.git\$
  #22M	./chengxiaowei_blog/.git
  
  $ du -s -h
  45M	.
  ```

## find

find 指定目录下查找文件或者文件夹

```bash
$ find path -name "file_name"
```

## rm

```bash
## 直接删除，不需要询问
$ rm -f path or file
```

```bash
## 递归强删
$ rm -rf path or file 
```

```bash
## 找出目录下的所有后缀为.go并强删除
$ find path -name ".go" | xargs rm -rf 
```

```bash
## 过滤 '1.md' 删除所有文件
$ find * | grep -v '1.md' | xargs rm
```



## ssh

```bash
 ## 生成公私钥
$ ssh-keygen -t rsa
```

```bash
ssh -V #查看版本
OpenSSH_6.4p1, OpenSSL 1.0.1e-fips 11 Feb 2013
```

```bash
#链接我那台ubuntu
ssh joker@192.168.0.104 
```

```bash
#指定端口链接
ssh -p 2223 joker@192.168.0.104 
```

```bash
#可以用域名代替ip
vim /etc/hosts
ssh joker
```

修改默认端口

```bash
sudo vim /etc/ssh/sshd_config
```

![](/img/linux/7.png "工作流程")

使用简单防火墙禁用22端口,开放2223

```bash
sudo ufw deny 22
sudo ufw allow 2223
```

禁止密码登录,钥匙登录

```bash
 vim /etc/ssh/sshd_config
 PasswordAuthentication no
 PubkeyAuthentication yes
 
 将本地的公钥上传到服务器的这个文件中 ~/.ssh/authorized_keys
```

流程就是本地ssh服务用`~/.ssh/id_rsa` 私钥将一个随机字符串加密，发到服务器，服务器将你的公钥解密



#### ssh use .pem file

```bash
$ ssh -i ~/Downloads/demo.pem root@192.168.0.100
```



## tar

- 压缩

  ```bash
   touch a.c       
   tar -czvf test.tar.gz a.c   //压缩 a.c文件为test.tar.gz
  ```

- 解压

  ```bash
  tar -xzvf test.tar.gz
  ```

- 参数

  ```bash
  -c或--create 建立新的备份文件
  -z或--gzip 通过gzip指令处理备份文件
  -v或--verbose 显示指令执行过程
  -f或--file 指定备份文件
  -x或--extract或--get 从备份文件还原文件
  ```

## top 命令



用于实时显示各个进程的状态

![](/img/linux/2.png "工作流程")

- 界面

  第一行：

  - 系统时间:16:49:59
  - 运行时间:up 627 days ，22:54 //627天22小时54分
  - 当前登录的用户:1 user //一个用户
  - load avage :负载均衡0.61,0.83,0.95 三个数字分别表示1分钟，5分钟，15分钟的负载情况

  第二行:进程信息

  - Tasks 82 total :总量有82进程在跑
  - 5 running :活跃的有5个
  - 74 sleeping :74个休眠
  - 0 stopped ：0个停止
  - 3 zombie：3个僵尸进程

  第三行:cpu状态

  - 8.1 us (user space) --用户空间cpu的百分比
  - 16.2 sy (sysctl) -- 内核cpu的百分比
  - 0.0 ni ()-- 改变过优先级的进程cpu的百分比
  - 75.8 id (idolt) --空闲cpu的百分比
  - 0.0 wa (wait) --IO 等待cpu的百分比
  - 0.0 hi (Hardware IRQ) --硬中断cpu的百分比
  - 0.0 si (Software Interrupts) --软中断cpu的百分比

  第四行: 内存状态（Kib Mem）

  - 1882232 total: 总计大约1838M内存
  - 303244 free: 大约有296M空闲内存
  - 523444 used : 大约用了511M
  - 1055544 buff/cache: 1030M 缓冲区

  第五行: 交换内存情况(Kib Swap)

  第七行:各个进程的状态监控

  - PID --进程id
  - USER --进程的所有者
  - PR --进程优先级
  - NI -- nice 值。小于0表示高优先级，大于0表示低优先级
  - VIRT-- 进程使用的虚拟进程总量
  - RES --进程使用的，未被换出的物理内存
  - SHR --共享内存
  - S --进程状态.D =不可中断的睡眠状态，R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程
  - %CPU -- cpu占用量
  - %MEM --内存占用量
  - TIME+ -- 使用cpu时间 单位1/100 秒 
  - CMMAND--进程名字

- top 运行中可以通过 top 的内部命令对进程的显示方式进行控制。内部命令如下： s – 改变画面更新频率 l – 关闭或开启第一部分第一行 top 信息的表示 t – 关闭或开启第一部分第二行 Tasks 和第三行 Cpus 信息的表示 m – 关闭或开启第一部分第四行 Mem 和 第五行 Swap 信息的表示 N – 以 PID 的大小的顺序排列表示进程列表 P – 以 CPU 占用率大小的顺序排列进程列表 M – 以内存占用率大小的顺序排列进程列表 h – 显示帮助 n – 设置在进程列表所显示进程的数量 q – 退出 top s – 改变画面更新周期

- 根据第七行的各个字段来排序

  进入top。默认是cpu占用量排序的，怎么改变其他字段排序？

  1. 敲击键盘 "b"(打开或关闭加亮效果)，效果如下,此时高亮的是running的运行状态的进程，可以键入`y` 键关闭或打开运行状态(running)的进程高亮效果:
     
     ![](/img/linux/3.png "工作流程")
     
  2. 键入`x` 键(打开或关闭列的排序高亮效果)此时是按照cpu占用量这个列排序，效果如下 
     ![](/img/linux/4.png "工作流程")
  
  3. 可以键入 `shift + <` 右移动高亮列，`shift+>` 左移动高亮列，效果如下:
  
     ![](/img/linux/5.png "工作流程")
  
     ![](/img/linux/6.png "工作流程")

```
top -p pid #根据pid搜索进程
```

## uniq

Linux uniq 命令用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用。
uniq 可检查文本文件中重复出现的行列。

## vim

1. 命令

   ```bash
   vim test.txt
   :q!
   :q
   :wq
   ```

   ```bash
   :set nu #设置行号
   :set nu! #取消行号
   ```

2. 快捷键

   ```
   gg //连按两下键盘g到文件顶部
   shift+gg //按住shift，连按两次g到文件底部
   0 //键盘上的0键，跳到句首
   shift+$ //跳到句尾
   ```

## wc

wc 用于计算文件的byte，字数，或是列数

- wc -c或--byte或--chars 只显示byte
- wc -l 或--lines 显示行数
- wc --help 在线帮助
- wc --version 显示版本信息

```bash
⋊> ~/Desktop wc -l swim.txt                                                                                         
1613 swim.txt #swim.txt 有1613行
```

```bash
#统计有多少个错误
grep "ERRO" /var/log/go_log/tcp_game.log | wc
```

## grep

```bash
## -C10 匹配行和它的前后10行
grep  -C10 panic userinfo_center.out.2021-11-02.07:28:34

## -B 匹配行和它的前10行
grep  -C10 panic userinfo_center.out.2021-11-02.07:28:34

## -A 匹配行和它的后10行
grep  -A10 panic userinfo_center.out.2021-11-02.07:28:34
```

过滤大于500ms的sql日志

``` bash
grep -E '([5-9][0-9]{2}|[0-9]{4,}).[0-9]{3}ms' ak-user-info-sql_WARN.log



2024-01-21 06:00:12.874 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/mobile_user_speed_time.go:46 SLOW SQL >= 200ms[1765.106ms] [rows:1] 
2024-01-21 06:00:12.880 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[778.238ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 1 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-21 06:00:12.881 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[759.098ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-21 06:00:12.888 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[663.101ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-21 12:54:18.589 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[660.299ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '9788226285568'
2024-01-21 13:24:53.178 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[738.466ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 1 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-21 14:09:49.139 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:101 SLOW SQL >= 200ms[779.674ms] [rows:0] 
2024-01-21 14:50:39.152 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[658.196ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 1 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-21 17:13:51.964 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[693.042ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-21 20:51:53.990 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[526.559ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-21 23:02:29.098 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[672.740ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '5917042364416'
2024-01-22 19:10:34.677 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[593.420ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '8759533686784'
2024-01-23 01:35:27.799 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[547.189ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '0927665164288'
2024-01-23 14:01:51.064 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[626.121ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '7936392409088'
2024-01-23 18:56:20.766 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[590.405ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '4968669655040'
2024-01-23 20:25:42.034 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/orders.go:84 SLOW SQL >= 200ms[760.383ms] [rows:1] SELECT count(*) FROM `yebao_order_prod`.`mobile_orders` WHERE user_id = '0979697274880' AND order_status = 1
2024-01-23 21:28:25.190 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[589.867ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-24 16:22:58.936 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[545.928ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 1 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-25 00:01:05.827 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[653.833ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-25 10:40:55.629 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/mobile_user_speed_time.go:53 SLOW SQL >= 200ms[961.386ms] [rows:0] SELECT expire_time,id FROM `yebao_user_prod`.`mobile_user_speed_time` WHERE user_id = '4189208481792' AND expire_time > now() AND speed_time - use_speed_time >0  ORDER BY expire_time desc LIMIT 1
2024-01-25 20:42:38.322 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[630.686ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '5050252945408'
2024-01-25 20:57:39.592 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[655.196ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '9230559154176'
2024-01-25 20:57:39.592 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[631.024ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-26 16:14:40.695 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[528.645ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '6276873236480'
2024-01-27 00:01:00.589 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/mobile_user_speed_log.go:30 SLOW SQL >= 200ms[574.666ms] [rows:1] INSERT INTO `yebao_user_prod`.`mobile_user_speed_time_log` (`user_id`,`create_time`,`speed_time`,`cost_type`) VALUES ('4450336583680','2024-01-27 00:01:00.014',36059,0)
2024-01-27 01:00:38.586 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[770.286ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-27 22:43:04.530 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[746.597ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
2024-01-28 14:13:30.511 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[686.320ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '7196358758400'
2024-01-28 14:21:02.068 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/user_info.go:78 SLOW SQL >= 200ms[745.693ms] [rows:1] SELECT id,nick_name,user_account,user_status,head_image_url,start_mobile_time FROM `yebao_user_prod`.`user_infos` WHERE user_id = '7496821084160'
2024-01-28 14:39:26.248 WARN /home/cat13/go/src/cloud/AK/ak-user-info/models/free_limit_conf.go:32 SLOW SQL >= 200ms[736.443ms] [rows:1] SELECT * FROM `yebao_game_prod`.`mobile_free_limit_conf` WHERE os = 0 ORDER BY `mobile_free_limit_conf`.`id` DESC LIMIT 1
```

## SS 

```bash
## 进程id pid 端口号 进程名
$ ss -tunlp | grep 6666
LISTEN     0      128                      :::6666                    :::*      users:(("shoe",28547,18))
```

````bash
## 统计链接服务器的链接数，9095为服务器端口
ss | grep 9095 |wc -l
````

## netstat

```bash
#网络监听，使用端口情况
netstat -tulnp
```

- **-t** -显示TCP端口
- **-u** -显示UDP端口
- **-n** -显示数字地址而不是主机
- **-l** - 仅显示监听地址端口
- -p - 显示进程的PID和名称，仅当root 或者sudo用户身份，才会显示信息

## scp

- -1： 强制scp命令使用协议ssh1
- -2： 强制scp命令使用协议ssh2
- -4： 强制scp命令只使用IPv4寻址
- -6： 强制scp命令只使用IPv6寻址
- -B： 使用批处理模式（传输过程中不询问传输口令或短语）
- -C： 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
- -p：保留原文件的修改时间，访问时间和访问权限。
- -q： 不显示传输进度条。
- -r： 递归复制整个目录。
- -v：详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
- -c cipher： 以cipher将数据传输进行加密，这个选项将直接传递给ssh。
- -F ssh_config： 指定一个替代的ssh配置文件，此参数直接传递给ssh。
- -i identity_file： 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
- -l limit： 限定用户所能使用的带宽，以Kbit/s为单位。
- -o ssh_option： 如果习惯于使用ssh_config(5)中的参数传递方式，
- -P port：注意是大写的P, port是指定数据传输用到的端口号
- -S program： 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。

本地-->远端

```bash
$ scp -r ./* joker@192.168.0.102:~/Desktop/oldimg

$ scp test.png joker@192.168.0.102:~/Desktop/test.png

$ scp test.png 192.168.0.102:~/Desktop/oldimg

$ scp test.png 192.168.0.102:~/Desktop/test.png
```

远端-->本地

```bash
$ scp  -r joker@192.168.0.102:~/Desktop/oldimg ~/oldimg

$ scp  joker@192.168.0.102:~/Desktop/test.png ~/test.png

$ scp  -r 192.168.0.102:~/Desktop/oldimg ~/oldimg

$ scp  192.168.0.102:~/Desktop/test.png ~/test.png
```

## sudo

### 免密

```bash
$ sudo chmod 640 /etc/sudoers
$ vim /etc/sudoer
$ echo 'cat13 ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
$ chmod 440 /etc/sudoers

#or

$ cd /etc/sudoers.d/
$ vim cat13
#添加 'cat13 ALL=(ALL) NOPASSWD:ALL'
```

## lsblk

列出块设备信息

```bash
$ lsblk -h
Usage:
 lsblk [options] [<device> ...]

List information about block devices.

Options:
 -D, --discard        print discard capabilities
 -E, --dedup <column> de-duplicate output by <column>
 -I, --include <list> show only devices with specified major numbers
 -J, --json           use JSON output format
 -O, --output-all     output all columns
 -P, --pairs          use key="value" output format
 -S, --scsi           output info about SCSI devices
 -T, --tree[=<column>] use tree format output
 -a, --all            print all devices
 -b, --bytes          print SIZE in bytes rather than in human readable format
 -d, --nodeps         don't print slaves or holders
 -e, --exclude <list> exclude devices by major number (default: RAM disks)
 -f, --fs             output info about filesystems
 -i, --ascii          use ascii characters only
 -l, --list           use list format output
 -M, --merge          group parents of sub-trees (usable for RAIDs, Multi-path)
 -m, --perms          output info about permissions
 -n, --noheadings     don't print headings
 -o, --output <list>  output columns
 -p, --paths          print complete device path
 -r, --raw            use raw output format
 -s, --inverse        inverse dependencies
 -t, --topology       output info about topology
 -w, --width <num>    specifies output width as number of characters
 -x, --sort <column>  sort output by <column>
 -z, --zoned          print zone model
     --sysroot <dir>  use specified directory as system root

 -h, --help           display this help
 -V, --version        display version

Available output columns:
         NAME  device name
        KNAME  internal kernel device name
         PATH  path to the device node
      MAJ:MIN  major:minor device number
      FSAVAIL  filesystem size available
       FSSIZE  filesystem size
       FSTYPE  filesystem type
       FSUSED  filesystem size used
       FSUSE%  filesystem use percentage
      FSROOTS  mounted filesystem roots
        FSVER  filesystem version
   MOUNTPOINT  where the device is mounted
  MOUNTPOINTS  all locations where device is mounted
        LABEL  filesystem LABEL
         UUID  filesystem UUID
       PTUUID  partition table identifier (usually UUID)
       PTTYPE  partition table type
     PARTTYPE  partition type code or UUID
 PARTTYPENAME  partition type name
    PARTLABEL  partition LABEL
     PARTUUID  partition UUID
    PARTFLAGS  partition flags
           RA  read-ahead of the device
           RO  read-only device
           RM  removable device
      HOTPLUG  removable or hotplug device (usb, pcmcia, ...)
        MODEL  device identifier
       SERIAL  disk serial number
         SIZE  size of the device
        STATE  state of the device
        OWNER  user name
        GROUP  group name
         MODE  device node permissions
    ALIGNMENT  alignment offset
       MIN-IO  minimum I/O size
       OPT-IO  optimal I/O size
      PHY-SEC  physical sector size
      LOG-SEC  logical sector size
         ROTA  rotational device
        SCHED  I/O scheduler name
      RQ-SIZE  request queue size
         TYPE  device type
     DISC-ALN  discard alignment offset
    DISC-GRAN  discard granularity
     DISC-MAX  discard max bytes
    DISC-ZERO  discard zeroes data
        WSAME  write same max bytes
          WWN  unique storage identifier
         RAND  adds randomness
       PKNAME  internal parent kernel device name
         HCTL  Host:Channel:Target:Lun for SCSI
         TRAN  device transport type
   SUBSYSTEMS  de-duplicated chain of subsystems
          REV  device revision
       VENDOR  device vendor
        ZONED  zone model
          DAX  dax-capable device

For more details see lsblk(8).
```

```bash
$ lsblk -Jnbo NAME,MAJ:MIN,RM,SIZE,RO,TYPE,MOUNTPOINT,TRAN
```

## apt
全称`Advanced Packaging Tool`,`Debian `和 `Ubuntu `的包管理软件

```

```

## rsync

rsync is an [open source](https://www.opensource.org/) utility that provides fast incremental file transfer

Rsync是一个开源实用程序，提供快速增量文件传输

````bash
#拷贝某个忽略某个文件夹
rsync -rv --exclude=.git ../Joker_null/  ./
````

## crontab 

定时任务

编辑定时任务配置

```bash
crontab -e
```

重载定时任务配置

```bash
service cron reload
```

## IP

{{% figure class="center" src="/img/linux/9.png" title="socket_detail" alt="img" %}}

更改目标地址`112.16.229.104` 走`ens3`网卡

```bash
ip route add 112.16.229.104 via 0.0.0.0 dev ens3
```






