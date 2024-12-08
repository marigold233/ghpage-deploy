+++
title="用systemd托管java程序"
date=2024-12-08

[taxonomies]
categories = ["技术"]
tags = ["Linux", "systemd", "java"]

[extra]
toc = true
+++

## 前提

普通用户自定义的systemd unit文件可以放在`~/.config/systemd/user/`下，那么我们需要创建这个文件夹并把单元放到此处。

```bash
mkdir -p ~/.config/systemd/user/ && cd ~/.config/systemd/user/
```

## 方式一 完全用systemd
<details open>
<summary><b><code>javademo-spring.service</code></b></summary>

```ini
[Unit]
Description=Java Demo Spring Application
After=network.target

[Service]
Type=simple
#User=qing
#Group=qing
WorkingDirectory=/home/qing/javademo-spring
ExecStartPre=/bin/mkdir -p /home/qing/javademo-spring/logs
Environment="JAVA_HOME=/opt/jdk-21.0.4"
# JVM 配置
Environment="JAVA_OPTS=-Xms4g -Xmx4g -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/qing/javademo-spring/logs -XX:+UseG1GC -Xlog:gc*:file=/home/qing/javademo-spring/logs/javademo-spring-gc.log:time,uptime:filecount=5,filesize=20M"
# 应用配置
Environment="APP_OPTS=-Dlog.days=7 -Dlog.dir=/home/qing/javademo-spring/logs"
ExecStart=/bin/bash -c "${JAVA_HOME}/bin/java ${JAVA_OPTS} ${APP_OPTS} -jar app.jar"
ExecStop=/bin/kill -s TERM $MAINPID
Restart=on-failure
RestartSec=10
StartLimitInterval=60s
StartLimitBurst=5
LimitNOFILE=65536
StandardOutput=append:/home/qing/javademo-spring/logs/javademo-spring.log
StandardError=append:/home/qing/javademo-spring/logs/javademo-spring.error.log
SuccessExitStatus=0

[Install]
WantedBy=multi-user.target
```

</details>

然后重载systemd：

```bash
systemctl --user daemon-reload
systemctl --user start javademo-spring
```

## 方式二 用启动脚本

这种方式要配合脚本来实现，之前写过**java程序启动脚本**

<details open>
<summary><b><code>javademo-spring-script.service</code></b></summary>

```ini
[Unit]
Description=Java Demo Spring Application
After=network.target

[Service]
Type=forking
#User=qing
#Group=qing
WorkingDirectory=/home/qing/javademo-spring
ExecStart=/bin/bash javaapp-start-v2.sh start
ExecStop=/bin/bash javaapp-start-v2.sh stop
Restart=on-failure
RestartSec=10
StartLimitInterval=60s
StartLimitBurst=5
LimitNOFILE=65536
#StandardOutput=append:/home/qing/javademo-spring/logs/javademo-spring.log
#StandardError=append:/home/qing/javademo-spring/logs/javademo-spring.error.log
SuccessExitStatus=0

[Install]
WantedBy=multi-user.target
```

</details>


## 建议

  在生产环境推荐设置成`Restart=always`，适当调整重启`RestartSec`参数。