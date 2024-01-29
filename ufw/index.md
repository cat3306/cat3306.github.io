# ufw


<!--more-->

ufw (Uncomplicated FireWall) 简单防火墙，是ubunt系统下默认防火墙。比iptables简单
配置文件  `cat /etc/default/ufw`

1. 查看规则

   ```bash
   $ sudo ufw status #简略信息
   $ sudo ufw status verbose#详细信息
   $ sudo ufw status numbered #序号输出
   ```

2. 开放某个端口

   ```bash
   $ sudo ufw allow 80 #开放80
   ```

3. 关闭某个端口

   ```bash
   $ sudo ufw deny 22
   ```

4. 阻止某个ip访问(针对所有的端口)

   ```bash
   $ sudo ufw deny from 208.176.0.50 to any
   ```

5. 阻止某个ip访问(针对一个端口)

   ```bash
   $ sudo ufw deny from 202.54.1.5 to any port 80
   ```

