#!/bin/sh

# 设置日志级别
export LOG_LEVEL=${LOG_LEVEL:-info}

# 检查必要的环境变量
if [ -z "$TAILSCALE_AUTHKEY" ]; then
    echo "[ERROR] TAILSCALE_AUTHKEY is not set"
    exit 1
fi

# 日志函数
log() {
    local level="$1"
    local message="$2"
    
    if [ "$LOG_LEVEL" = "debug" ] || [ "$level" = "error" ] || [ "$level" = "info" ]; then
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message"
    fi
}

log "info" "Starting Tailscale VPN container"
log "info" "Hostname: $TAILSCALE_HOSTNAME"

# 启动Tailscale守护进程
log "info" "Starting tailscaled daemon..."
./tailscaled --state=/var/lib/tailscale/tailscaled.state --socket=/var/run/tailscale/tailscaled.sock --tun=userspace-networking --socks5-server=localhost:1055 --outbound-http-proxy-listen=localhost:1055 &

# 存储tailscaled的PID
tailscaled_pid=$!

# 设置信号处理
trap "log 'info' 'Stopping Tailscale VPN container'; kill -TERM $tailscaled_pid; wait $tailscaled_pid; ./tailscaled --cleanup; exit 0" SIGTERM SIGINT

# 连接到Tailscale网络
log "info" "Connecting to Tailscale network..."
until ./tailscale up --authkey=${TAILSCALE_AUTHKEY} --hostname=${TAILSCALE_HOSTNAME} --advertise-exit-node ${TAILSCALE_ADDITIONAL_ARGS}
do
    log "error" "Failed to connect to Tailscale, retrying in 1 second..."
    sleep 1
done

log "info" "Successfully connected to Tailscale network"

# 获取并显示Tailscale状态
if ./tailscale status > /dev/null 2>&1; then
    log "info" "Tailscale status summary:" 
    ./tailscale status | head -5
else
    log "error" "Failed to retrieve Tailscale status"
fi

# 等待守护进程结束
wait $tailscaled_pid
