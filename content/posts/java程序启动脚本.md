+++
title="java程序启动脚本"
date=2024-12-08

[taxonomies]
categories = ["技术"]
tags = ["Linux", "java", "script"]

[extra]
toc = true
+++

原先自己写的java应用启动脚本，现在用AI优化了一下：

<details open>
<summary><b><code>javaapp-start.sh</code></b></summary>

```bash
#!/usr/bin/env bash

# 脚本基础配置
declare -r SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)
declare -r APP_HOME="$SCRIPT_DIR"
declare -r APP_NAME="javademo-spring"
declare -r JAR_NAME="app.jar"
declare -r JAR_PATH="${APP_HOME}/${JAR_NAME}"
declare -r LOG_DIR="${APP_HOME}/logs"
declare -r GC_LOG_FILE="${LOG_DIR}/${APP_NAME}-gc.log"

# 应用配置
# java 9+
declare -r JVM_OPTS=(
    "-Xms4g"
    "-Xmx4g"
    "-XX:+HeapDumpOnOutOfMemoryError"
    "-XX:HeapDumpPath=${LOG_DIR}"
    "-XX:+UseG1GC"
    # GC日志参数 (Java 9+ 的写法)
    "-Xlog:gc*:file=${GC_LOG_FILE}:time,uptime:filecount=5,filesize=20M"
    # 或者用下面这种更详细的方式
    #"-Xlog:gc*=info:file=${GC_LOG_FILE}:time,uptime,level,tags:filecount=5,filesize=20M"
)
# java 8
# declare -r JVM_OPTS=(
#     "-Xms4g"
#     "-Xmx4g"
#     "-XX:+HeapDumpOnOutOfMemoryError"
#     "-XX:HeapDumpPath=${LOG_DIR}"
#     "-XX:+UseG1GC"
#     "-XX:+PrintGCDetails"
#     "-XX:+PrintGCDateStamps"
#     "-Xloggc:${GC_LOG_FILE}"
#     "-XX:+UseGCLogFileRotation"
#     "-XX:NumberOfGCLogFiles=5"
#     "-XX:GCLogFileSize=20M"
# )

declare -r APP_OPTS=(
    "-Dlog.days=7"
    "-Dlog.dir=${LOG_DIR}"
)

# 日志配置
declare -rA LOG_COLORS=(
    [DEBUG]="\033[36m"
    [INFO]="\033[32m"
    [WARN]="\033[33m"
    [ERROR]="\033[31m"
    [RESET]="\033[0m"
)

declare -A LOG_CONFIG=(
    [enabled]=true
    [dir]="$LOG_DIR"
    [prefix]="startLog"
    [level]="INFO"
    [max_days]=30
    [file]=""
)

# 日志系统初始化
init_logger() {
    local timestamp=$(date '+%Y-%m-%d')
    LOG_CONFIG[file]="${LOG_CONFIG[dir]}/${LOG_CONFIG[prefix]}_${timestamp}.log"

    mkdir -p "${LOG_CONFIG[dir]}" || return 1
    touch "${LOG_CONFIG[file]}" || return 1

    # 清理旧日志
    find "${LOG_CONFIG[dir]}" -name "${LOG_CONFIG[prefix]}_*.log" -type f \
        -mtime +${LOG_CONFIG[max_days]} -delete 2>/dev/null
}

# 日志函数
log() {
    local level=$1 msg=$2 to_file=${3:-true}
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    local log_msg="$timestamp [$level] $msg"

    # 控制台输出
    printf "%b\n" "${LOG_COLORS[$level]}${log_msg}${LOG_COLORS[RESET]}"

    # 文件输出
    ${to_file} && ${LOG_CONFIG[enabled]} && printf "%b\n" "$log_msg" >> "${LOG_CONFIG[file]}"
}

debug() { log "DEBUG" "$1" "${2:-true}"; }
info()  { log "INFO"  "$1" "${2:-true}"; }
warn()  { log "WARN"  "$1" "${2:-true}"; }
error() { log "ERROR" "$1" "${2:-true}"; }

# 进程管理函数
is_running() {
    local show_details=${1:-false}
    if $show_details; then
        jps -l | grep "$JAR_NAME"
    else
        jps -l | grep -q "$JAR_NAME"
    fi
}

check_port() {
    local port=${1:?}
    if timeout 5 bash -c "echo -n > /dev/tcp/127.0.0.1/$port" 2>/dev/null; then
        info "Port $port is accessible" false
        return 0
    else
        error "Port $port is not accessible" false
        return 1
    fi
}

start_app() {
    if ! is_running; then
        info "Starting $APP_NAME..."
        java ${JVM_OPTS[*]} ${APP_OPTS[*]} -jar "$JAR_PATH" >>"${LOG_CONFIG[file]}" &
        sleep 2
        is_running true && info "Started successfully" || error "Failed to start"
    else
        warn "Application is already running"
    fi
}

stop_app() {
    info "Stopping $APP_NAME..."
    if ! is_running; then
        warn "Application is not running"
        return 0
    fi

    pkill -15 -f "$JAR_NAME"

    local timeout=30
    while ((timeout > 0)); do
        is_running || { info "Application stopped gracefully"; return 0; }
        sleep 1
        ((timeout--))
    done

    warn "Force killing the application..."
    pkill -9 -f "$JAR_NAME"
    is_running || info "Application stopped forcefully"
}

monitor_app() {
    local port=$1

    if ! is_running; then
        error "Process not found, starting..."
        start_app
    elif ! check_port "$port"; then
        error "Port $port is not responding, restarting..."
        stop_app && start_app
    else
        info "Application is running normally"
    fi
}

# 主函数
main() {
    init_logger || { error "Failed to initialize logger"; exit 1; }

    case "$1" in
        start)      start_app ;;
        stop)       stop_app ;;
        restart)    stop_app && start_app ;;
        status)     is_running true ;;
        check)      check_port "${2:?Port required}" ;;
        monitor)    monitor_app "${2:?Port required}" ;;
        *)
            echo "Usage: $0 {start|stop|restart|status|check|monitor} [port]"
            exit 1
            ;;
    esac
}

main "$@"
```

</details>


使用：
  1. 将脚本放在jar相同的目录中
  2. 修改`APP_NAME`和`JAR_NAME`常量
  3. 赋予755权限，执行`./javaapp-start.sh start`启动

支持的参数：
```
./javaapp-start.sh start  # 启动
./javaapp-start.sh stop   # 停止
./javaapp-start.sh restart  # 重启
./javaapp-start.sh status  # 查看状态
./javaapp-start.sh check  # 检查端口和进程
./javaapp-start.sh monitor  # 监控程序可以配合crontab来保活
```