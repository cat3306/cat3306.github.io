# git raw


<!--more-->

### 准备

git，服务器(ubuntu为例)，ssh服务

### 创建一个git用户组和用户

```bash
$ useradd -m git #添加用户，-m创建用户文件夹
$ sudo passwd git #设置密码
$ sudo su git #切换git用户，输入密码
$ chsh -s /bin/bash #指定shell
```

### 收集远端的公钥

把用户的公钥导入
`/home/git/.ssh/authorized_keys`  一行一个

```bash
$ cd /home/git
$ mkdir .ssh && chmod 755 .shh
$ touch authorized_keys
```

### 创建仓库目录

```bash
$ mkdir -p repositories && cd repositories
$ git init --bare test.git
```

### 注意事项

1. ssh 服务需要配置

   ```bash
   $ vim /stc/ssh/sshd_config #or vim /stc/ssh/ssh_config 
   #将权限放开
   RSAAuthentication yes
   PubkeyAuthentication yes
   AuthorizedKeysFile   .ssh/authorized_keys
   ```

2. 出于安全考虑，git用户不允许登录shell，可以编辑`/etc/passwd` 完成

   ```bash
   git: x:1001:1001::/home/git:/bin/bash
    #改成
   git: x:1001:1001::/home/git:/bin/git-shell
   ```

3. 必须创建git用户搭建，并且ssh鉴权在git用户的家目录下
   远端可以拉仓库了

   ```bash
   git@192.168.0.104:/home/git/repositories/test.git
   ```

