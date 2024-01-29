# mac command


<!--more-->

# sshd

启动

```shell
$ sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```

停止

```shell
$ sudo launchctl unload -w /System/Library/LaunchDaemons/ssh.plist
```

查看是否启动成功

```bash
$ sudo launchctl list | grep ssh
说明成功
	# 0	com.openssh.sshd
```

# 配置vim 语法高亮

```bash
cp /usr/share/vim/vimrc ~/.vimrc

vim ~/.vimrc
##添加以下内容
syntax on
set nu!
set autoindent
```


