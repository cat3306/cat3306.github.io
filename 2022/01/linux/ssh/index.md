# sshd


<!--more-->

# 官网

https://www.ssh.com/

## 认识SSH

SSH是 Secure Shell 的缩写，是一种的网络协议

## 应用

SSH 主要用于登录服务器。例如以用户user，登录远程主机host，user是远程主机的用户

````bash
$ ssh user@host
$ ssh root@192.168.0.5
````

如果本地用户名远程的用户名一致，登录可以省略用户名

````bash
$ ssh host
````

SSH 的默认端口是22，上面的命令没有带端口号就是默认会去连接22端口。

````bash
$ ssh -p 2222 user@host
````

SSH 之所以安全的是因为它采用了公私钥加密，非对称加密。如果是密码登录是以下过程

- 远程主机收到用户的登录请求，把自己的公钥发个用户
- 用户使用这个公钥，将登录密码加密后，发送个远程主机
- 远程主机收到加密后的登录密码，用自己的私钥解密，如果密码正确，可登录

整个流程看似安全实际上存在中间人攻击的风险。如果我是一个黑客，我在中间拦截了用户的登录请求，并将我伪造的公钥发给用户。用户将加了密的登录密码发给我，我再用私钥解密，这样我就获取了远程主机的登录密码。所以，HTTPS协议由CA来证明公钥的身份，当然CA也是可以伪装的，这是一个无穷无尽的信任问题。SSH 没有CA机构来认证，都是自己签发。

### 密码登录

如果是第一次登录，系统会这样提示

`The authenticity of host '192.168.1.5 (192.168.1.5)' can't be established.
ECDSA key fingerprint is SHA256:0OFCrK6ijuT+GWZ/qLTfnbww8si6AEEeUhKcXnmE5xU.
Are you sure you want to continue connecting (yes/no/[fingerprint])?`

意思是无法确认主机“192.168.1.5(192.168.1.5)”的真实性。只知道它的公钥指纹是这个`0OFCrK6ijuT+GWZ/qLTfnbww8si6AEEeUhKcXnmE5xU` 意思是需要你自己去核对，是不是你期望的远端公钥
OK确认是你要的公钥，键入yes

`Are you sure you want to continue connecting (yes/no/[fingerprint])? yes`

系统会一个warning ，表示host主机已经被认可

`Warning: Permanently added '192.168.1.5' (ECDSA) to the list of known hosts.`

然后要求输入密码
`Password: (enter password)`

如果密码正确，就登录成功

当远程主机的公钥被接受以后，它就会被保存在文件`~/.ssh/known_hosts`之中。下次再连接这台主机，系统就会认出它的公钥已经保存在本地了，从而跳过警告部分，直接提示输入密码。

每个SSH用户都有自己的known_hosts文件，此外系统也有一个这样的文件，通常是/etc/ssh/ssh_known_hosts，保存一些对所有用户都可信赖的远程主机的公钥。

### 公钥登录

每次使用终端登录服务器输密码，不够丝滑。SSH 支持公钥登录，一劳永逸的操作

```bash
$ ssh-keygen
```

会在~/.ssh/目录下生成 id_rsa.pub和id_rsa。前者是你的公钥，后者是你的私钥。将你的公钥上传到服务器上

在 `~/.ssh/authorized_keys` 末尾添加即可

## 配置

```bash
$ cat /etc/ssh/sshd_config
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem	sftp	/usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#	X11Forwarding no
#	AllowTcpForwarding no
#	PermitTTY no
#	ForceCommand cvs server
```

### 修改端口

```bash
$ sudo vim /etc/ssh/sshd_config
```

![](/img/linux/7.png)

```bash
# ubuntu下重启更新配置
$ systemctl restart sshd
```

### 禁用密码

![](/img/linux/8.png)

### 开启公钥

```bash
 vim /etc/ssh/sshd_config
 PubkeyAuthentication yes
 
 将本地的公钥上传到服务器的这个文件中 ~/.ssh/authorized_keys
```

### root登录

```bash
$ vim  /etc/ssh/sshd_config
# 禁止密码登录
PermitRootLogin prohibit-password

# 开启root密码登录
PermitRootLogin yes
```



### 超时

```bash
# vi /etc/ssh/sshd_config
ClientAliveInterval 5m          # 5 minutes
ClientAliveCountMax 2           # 2 times
```

## 



## 原理




