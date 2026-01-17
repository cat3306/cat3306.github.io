# Termux与 frp 实现内网穿透


<!--more-->
# Termux与 frp 实现内网穿透

## 第一步：在 VPS 上安装和配置 FRP 服务端
### 1. 在 VPS 上安装 FRP 服务端
``` bash
# SSH 连接到你的 VPS
ssh root@your-vps-ip

# 下载 FRP 最新版本
cd /opt
wget https://github.com/fatedier/frp/releases/download/v0.52.3/frp_0.52.3_linux_amd64.tar.gz
tar -xzf frp_0.52.3_linux_amd64.tar.gz
cd frp_0.52.3_linux_amd64

# 复制服务端文件
cp frps /usr/local/bin/
mkdir -p /etc/frp
```
### 2. 创建 VPS 服务端配置文件
``` bash
nano /etc/frp/frps.toml
```

``` toml
# FRP 服务端配置
[common]
bind_port = 7000
bind_addr = "0.0.0.0"
token = "your_secure_token_here_123456"

# 管理后台
[webServer]
addr = "0.0.0.0"
port = 7500
user = "admin"
password = "admin123"

# 日志配置
[log]
to = "/var/log/frps.log"
level = "info"
max_days = 3

# 允许端口范围
allow_ports = "2000-3000,3001,3003,4000-50000"

# 心跳配置
heartbeat_timeout = 90
```

### 3. 创建系统服务
``` bash
nano /etc/systemd/system/frps.service
```
``` ini
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=root
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.toml
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=5s

[Install]
WantedBy=multi-user.target
```
### 4. 启动并设置开机自启
``` bash
# 重载系统服务
systemctl daemon-reload

# 启动服务
systemctl start frps

# 设置开机自启
systemctl enable frps

# 查看状态
systemctl status frps
```
### 5. 配置防火墙（如果有）
``` bash
# UFW 防火墙
ufw allow 7000
ufw allow 7500
ufw allow 2222  # SSH 穿透端口

# iptables 防火墙
iptables -A INPUT -p tcp --dport 7000 -j ACCEPT
iptables -A INPUT -p tcp --dport 7500 -j ACCEPT
iptables -A INPUT -p tcp --dport 2222 -j ACCEPT

# 保存规则（CentOS/RHEL）
service iptables save
```
## 第二步：在 Termux 中安装和配置 FRP 客户端
### 1. 安装 FRP 客户端

``` bash
# 更新 Termux
pkg update

# 下载 FRP ARM64 版本
cd ~/downloads || mkdir ~/downloads && cd ~/downloads
wget https://github.com/fatedier/frp/releases/download/v0.52.3/frp_0.52.3_linux_arm64.tar.gz

# 解压安装
tar -xzf frp_0.52.3_linux_arm64.tar.gz
cd frp_0.52.3_linux_arm64
cp frpc $PREFIX/bin/

# 验证安装
frpc --version
```

### 2. 创建客户端配置文件
``` bash
mkdir -p $PREFIX/etc/frp
nano $PREFIX/etc/frp/frpc.toml
```
配置内容：
``` toml
serverAddr = "1.2.3.4"
serverPort = 7000
auth.method = "token"
auth.token = "your_token"

# MySQL 主库
[[proxies]]
name = "mysql_master"
type = "tcp"
localIP = "127.0.0.1"
localPort = 3306
remotePort = 3306

# MySQL 从库
[[proxies]]
name = "mysql_slave"
type = "tcp"
localIP = "127.0.0.1"
localPort = 3307
remotePort = 3307

# Redis 集群
[[proxies]]
name = "redis_node1"
type = "tcp"
localIP = "127.0.0.1"
localPort = 6379
remotePort = 6379

[[proxies]]
name = "redis_node2"
type = "tcp"
localIP = "127.0.0.1"
localPort = 6380
remotePort = 6380
```
重要： 将 `YOUR_VPS_IP` 替换为你的 VPS 实际 IP 地址，`token` 必须与服务端配置一致。

### 3. 启动 Termux 中的 SSH 服务
``` bash
# 安装 OpenSSH
pkg install openssh

# 设置密码（如果还没有）
passwd

# 启动 SSH 服务（默认运行在 8022 端口）
sshd

# 验证 SSH 服务运行
ps aux | grep sshd
netstat -tuln | grep 8022
```
## 4. 启动 FRP 客户端
``` bash
# 前台启动（调试用）
frpc -c $PREFIX/etc/frp/frpc.toml

# 后台启动
nohup frpc -c $PREFIX/etc/frp/frpc.toml > ~/frpc.log 2>&1 &
```
## 第三步：创建管理脚本
``` bash
nano ~/frp-ssh.sh
```
``` bash
#!/data/data/com.termux/files/usr/bin/bash

# FRP SSH 穿透管理脚本

VPS_IP="YOUR_VPS_IP"  # 替换为你的 VPS IP
SSH_PORT=2222         # VPS 上的 SSH 穿透端口
FRPC_CONFIG="$PREFIX/etc/frp/frpc.toml"
FRPC_LOG="$HOME/frpc.log"
FRPC_PID="$HOME/frpc.pid"

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1"; }

# 检查 FRP 客户端是否运行
is_frpc_running() {
    if [[ -f "$FRPC_PID" ]]; then
        local pid=$(cat "$FRPC_PID")
        if kill -0 "$pid" 2>/dev/null; then
            return 0
        else
            rm -f "$FRPC_PID"
        fi
    fi
    return 1
}

# 检查 SSH 服务是否运行
is_sshd_running() {
    pgrep -f "sshd" >/dev/null
}

# 启动 SSH 服务
start_sshd() {
    if is_sshd_running; then
        log_info "SSH 服务已运行"
    else
        log_info "启动 SSH 服务..."
        sshd
        sleep 2
        if is_sshd_running; then
            log_info "SSH 服务启动成功"
        else
            log_error "SSH 服务启动失败"
            return 1
        fi
    fi
}

# 启动 FRP 客户端
start_frpc() {
    if [[ ! -f "$FRPC_CONFIG" ]]; then
        log_error "配置文件不存在: $FRPC_CONFIG"
        log_info "请运行 '$0 config' 创建配置文件"
        return 1
    fi
    
    if is_frpc_running; then
        log_warn "FRP 客户端已运行"
        return 1
    fi
    
    log_info "启动 FRP 客户端..."
    
    # 验证配置文件
    if ! frpc verify -c "$FRPC_CONFIG" >/dev/null 2>&1; then
        log_error "配置文件验证失败"
        frpc verify -c "$FRPC_CONFIG"
        return 1
    fi
    
    # 启动客户端
    nohup frpc -c "$FRPC_CONFIG" > "$FRPC_LOG" 2>&1 &
    local pid=$!
    echo $pid > "$FRPC_PID"
    
    # 等待启动
    sleep 3
    if is_frpc_running; then
        log_info "FRP 客户端启动成功 (PID: $pid)"
        log_info "SSH 穿透地址: $VPS_IP:$SSH_PORT"
    else
        log_error "FRP 客户端启动失败"
        log_error "查看日志: tail -f $FRPC_LOG"
        return 1
    fi
}

# 停止服务
stop_services() {
    # 停止 FRP 客户端
    if is_frpc_running; then
        local pid=$(cat "$FRPC_PID")
        log_info "停止 FRP 客户端 (PID: $pid)..."
        kill "$pid"
        rm -f "$FRPC_PID"
        log_info "FRP 客户端已停止"
    else
        log_warn "FRP 客户端未运行"
    fi
    
    # 停止 SSH 服务
    if is_sshd_running; then
        log_info "停止 SSH 服务..."
        pkill -f "sshd"
        log_info "SSH 服务已停止"
    else
        log_warn "SSH 服务未运行"
    fi
}

# 重启服务
restart_services() {
    log_info "重启 SSH 穿透服务..."
    stop_services
    sleep 2
    start_all
}

# 启动所有服务
start_all() {
    start_sshd
    if [[ $? -eq 0 ]]; then
        start_frpc
    fi
}

# 显示状态
show_status() {
    echo -e "${BLUE}=== SSH 穿透服务状态 ===${NC}"
    
    # SSH 服务状态
    if is_sshd_running; then
        echo -e "${GREEN}✓${NC} SSH 服务: 运行中"
        echo "  本地端口: 8022"
    else
        echo -e "${RED}✗${NC} SSH 服务: 未运行"
    fi
    
    # FRP 客户端状态
    if is_frpc_running; then
        local pid=$(cat "$FRPC_PID")
        echo -e "${GREEN}✓${NC} FRP 客户端: 运行中 (PID: $pid)"
        echo "  穿透地址: $VPS_IP:$SSH_PORT"
        echo "  管理界面: http://127.0.0.1:7400"
    else
        echo -e "${RED}✗${NC} FRP 客户端: 未运行"
    fi
    
    echo ""
    echo -e "${BLUE}=== 连接信息 ===${NC}"
    echo "外网 SSH 连接命令:"
    echo "  ssh -p $SSH_PORT $(whoami)@$VPS_IP"
    echo ""
    echo "本地连接测试:"
    echo "  ssh -p 8022 $(whoami)@127.0.0.1"
}

# 测试连接
test_connection() {
    log_info "测试本地 SSH 连接..."
    if nc -z 127.0.0.1 8022; then
        log_info "本地 SSH 服务正常"
    else
        log_error "本地 SSH 服务不可达"
        return 1
    fi
    
    log_info "测试 VPS 连接..."
    if nc -z "$VPS_IP" 7000; then
        log_info "VPS FRP 服务正常"
    else
        log_error "无法连接到 VPS FRP 服务"
        return 1
    fi
    
    log_info "测试穿透端口..."
    if nc -z "$VPS_IP" "$SSH_PORT"; then
        log_info "SSH 穿透端口正常"
        log_info "可以使用以下命令连接:"
        echo "  ssh -p $SSH_PORT $(whoami)@$VPS_IP"
    else
        log_error "SSH 穿透端口不可达"
        return 1
    fi
}

# 显示日志
show_logs() {
    local lines=${1:-50}
    
    if [[ -f "$FRPC_LOG" ]]; then
        echo -e "${BLUE}=== FRP 客户端日志 (最后 $lines 行) ===${NC}"
        tail -n "$lines" "$FRPC_LOG"
    else
        log_warn "日志文件不存在: $FRPC_LOG"
    fi
}

# 实时监控日志
monitor_logs() {
    if [[ -f "$FRPC_LOG" ]]; then
        echo -e "${BLUE}=== 实时监控 FRP 日志 (Ctrl+C 退出) ===${NC}"
        tail -f "$FRPC_LOG"
    else
        log_warn "日志文件不存在: $FRPC_LOG"
    fi
}

# 生成配置文件
generate_config() {
    if [[ -f "$FRPC_CONFIG" ]]; then
        log_warn "配置文件已存在: $FRPC_CONFIG"
        read -p "是否覆盖? (y/N): " -r
        [[ ! $REPLY =~ ^[Yy]$ ]] && return 1
    fi
    
    # 获取用户输入
    echo -e "${YELLOW}请输入配置信息:${NC}"
    read -p "VPS IP 地址: " vps_ip
    read -p "认证令牌 (与服务端一致): " token
    read -p "SSH 穿透端口 (默认 2222): " ssh_port
    
    ssh_port=${ssh_port:-2222}
    
    # 创建配置目录
    mkdir -p "$(dirname "$FRPC_CONFIG")"
    
    # 生成配置文件
    cat > "$FRPC_CONFIG" << EOF
# FRP 客户端配置
[common]
server_addr = "$vps_ip"
server_port = 7000
token = "$token"

user = "termux_user"

admin_addr = "127.0.0.1"
admin_port = 7400
admin_user = "admin"
admin_pwd = "admin123"

[log]
to = "$FRPC_LOG"
level = "info"
max_days = 3

heartbeat_interval = 30
heartbeat_timeout = 90
pool_count = 1
tcp_mux = true

# SSH 服务代理
[[proxies]]
name = "ssh"
type = "tcp"
local_ip = "127.0.0.1"
local_port = 8022
remote_port = $ssh_port
use_encryption = true
use_compression = true
EOF
    
    # 更新脚本变量
    sed -i "s/YOUR_VPS_IP/$vps_ip/g" "$0"
    sed -i "s/SSH_PORT=2222/SSH_PORT=$ssh_port/g" "$0"
    
    log_info "配置文件已创建: $FRPC_CONFIG"
    log_info "VPS 地址: $vps_ip"
    log_info "SSH 端口: $ssh_port"
}

# 自动重连监控
auto_reconnect() {
    local interval=${1:-30}
    local max_retries=${2:-10}
    local retry_count=0
    
    log_info "启动自动重连监控 (间隔: ${interval}秒)"
    
    while true; do
        if ! is_frpc_running || ! is_sshd_running; then
            retry_count=$((retry_count + 1))
            log_warn "服务异常，尝试重启 (第 $retry_count 次)"
            
            if [[ $retry_count -le $max_retries ]]; then
                restart_services
                sleep 10
            else
                log_error "达到最大重试次数，停止监控"
                break
            fi
        else
            retry_count=0
        fi
        
        sleep "$interval"
    done
}

# 开机自启脚本
setup_autostart() {
    local boot_script="$HOME/.termux/boot/frp-ssh"
    
    mkdir -p "$(dirname "$boot_script")"
    
    cat > "$boot_script" << EOF
#!/data/data/com.termux/files/usr/bin/bash

# 等待网络就绪
sleep 15

# 获取唤醒锁
termux-wake-lock

# 启动 SSH 穿透服务
$0 start

# 启动自动重连监控
nohup $0 auto > /dev/null 2>&1 &
EOF
    
    chmod +x "$boot_script"
    log_info "开机自启动已配置: $boot_script"
}

# 主函数
main() {
    case "$1" in
        start)
            start_all
            ;;
        stop)
            stop_services
            ;;
        restart)
            restart_services
            ;;
        status)
            show_status
            ;;
        test)
            test_connection
            ;;
        logs)
            show_logs "$2"
            ;;
        monitor)
            monitor_logs
            ;;
        config)
            generate_config
            ;;
        auto)
            auto_reconnect "$2" "$3"
            ;;
        setup)
            setup_autostart
            ;;
        *)
            echo -e "${BLUE}FRP SSH 穿透管理工具${NC}"
            echo ""
            echo "用法: $0 <命令> [选项]"
            echo ""
            echo -e "${YELLOW}命令:${NC}"
            echo "  start                启动 SSH 穿透服务"
            echo "  stop                 停止所有服务"
            echo "  restart              重启服务"
            echo "  status               查看服务状态"
            echo "  test                 测试连接"
            echo "  logs [lines]         查看日志"
            echo "  monitor              实时监控日志"
            echo "  config               生成配置文件"
            echo "  auto [interval]      自动重连监控"
            echo "  setup                设置开机自启"
            echo ""
            echo -e "${YELLOW}示例:${NC}"
            echo "  $0 config            # 首次使用，生成配置"
            echo "  $0 start             # 启动服务"
            echo "  $0 status            # 查看状态"
            echo "  $0 test              # 测试连接"
            echo "  $0 auto 30           # 30秒检查间隔自动重连"
            echo "  $0 setup             # 设置开机自启"
            ;;
    esac
}

# 执行主函数
main "$@"
```

## 第四步：配置和启动服务
### 1. 生成配置文件
``` bash 
# 运行配置向导
~/frp-ssh.sh config

# 按提示输入：
# VPS IP 地址: 你的VPS IP
# 认证令牌: your_secure_token_here_123456 (与服务端一致)
# SSH 穿透端口: 2222 (默认)
```
### 2. 启动服务
``` bash
# 启动 SSH 穿透服务
~/frp-ssh.sh start

# 查看状态
~/frp-ssh.sh status

# 测试连接
~/frp-ssh.sh test
```
### 3. 从外网连接测试
``` bash
# 从外网连接 (替换为你的实际 VPS IP 和用户名)
ssh -p 2222 your_termux_username@your_vps_ip

# 查看 Termux 用户名
whoami
```

## 第五步：设置开机自启和监控
### 1. 设置开机自启
``` bash 
# 安装 Termux:Boot (从 F-Droid)
# 然后运行：
~/frp-ssh.sh setup
```

### 2. 启动自动重连监控
``` bash
# 30秒检查一次，最多重试10次
~/frp-ssh.sh auto 30 10 &

# 后台运行
nohup ~/frp-ssh.sh auto 30 10 > ~/frp-auto.log 2>&1 &
```

## 故障排除
### 1. 常见问题检查
``` bash
# 查看 Termux 服务状态
~/frp-ssh.sh status

# 查看详细日志
~/frp-ssh.sh logs 100

# 实时监控日志
~/frp-ssh.sh monitor

# 测试各个环节的连接
~/frp-ssh.sh test
```
### 2. VPS 服务端检查
``` bash
# 在 VPS 上检查服务状态
systemctl status frps

# 查看服务端日志
tail -f /var/log/frps.log

# 检查端口监听
netstat -tuln | grep -E ':(7000|7500|2222)'

# 重启服务端
systemctl restart frps
```
### 3. 网络连通性检查
``` bash
# 在 Termux 中测试到 VPS 的连接
ping your_vps_ip
nc -zv your_vps_ip 7000

# 在外网测试穿透端口
nc -zv your_vps_ip 2222
```

## 安全建议
### 1. 修改默认端口和密码
``` bash
# 修改 SSH 端口和加强认证
nano ~/.ssh/authorized_keys

# 设置强密码
passwd

# 禁用密码登录，只使用密钥认证（可选）
```
### 2. 配置防火墙规则
``` bash
# 在 VPS 上只开放必要端口
ufw --force reset
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow 7000  # FRP 控制端口
ufw allow 2222  # SSH 穿透端口
ufw --force enable
```
### 3. 监控访问日志
``` bash
# 监控 SSH 连接
tail -f /var/log/auth.log | grep ssh

# 监控 FRP 连接
~/frp-ssh.sh monitor
```
