+++
title="我的linux命令帮助手册"
date=2018-08-20

[taxonomies]
categories = ["Linux"]
tags = ["Linux", "command"]

[extra]
toc = true
+++

# 命令手册

## 1. 网络相关命令



### curl

#### 学习资料

[卷曲食谱 (catonmat.net)](https://catonmat.net/cookbooks/curl)

#### 忽略https证书和进行密码认证

```bash
curl -k -u admin:123 https://xxx
```

#### 将res结果作为另外一个curl的参数

```bash
curl -s --compressed --http2 -X GET \
        -H "Host:smp-api.iyouke.com" \
        -H "authorization:bearer5378fb9bd91687516570322" \
        -H "charset:utf-8" \
        -H "xy-extra-frame.html" \
        -H "accept:*/*" $url | \
        curl -s -u yanghe:8m. -d @- http://localhost:8081/moto
```
#### 连接网站的耗时
```bash
curl -o /dev/null -s -w time_namelookup:"\t"%{time_namelookup}"\n"time_connect:"\t\t"%{time_connect}"\n"time_pretransfer:"\t"%{time_pretransfer}"\n"time_starttransfer:"\t"%{time_starttransfer}"\n"time_total:"\t\t"%{time_total}"\n"time_redirect:"\t\t"%{time_redirect}"\n" --dns-servers 1.1.1.1 https://www.baidu.com
```

#### 测试端口连通性
```bash
echo  | curl --connect-timeout 3 -m 3 telnet://www.baidu.com:443
```

#### curl debug

```bash
# -vvv也行
curl -s --trace - --dns-servers 1.1.1.1 https://www.baidu.com 
```

### wget

#### 断点续传+后台下载

```shell
wget -b -c https://xxx
```

下载这个站点目录下的全部rpm

```shell
wget -r -np -L -nv -x -N mirrors.cmecloud.cn/bclinux/el8.2/BaseOS/x86_64/os/Packages/
```
- 参考
  - <https://www.cnblogs.com/librarookie/p/14660645.html>

### netstat

#### 统计TCP连接状态

```shell
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

### ss

#### 指定端口统计连接数

```bash
ss -ta src :6000
```

### lsof

#### 程序打开文件数

```bash
lsof -c nginx  | wc -l
```

```bash
lsof | grep java |grep IPv | wc -l
lsof | grep java |grep :6379 | wc -l
```


### telnet 
#### 批量测试端口连通性
```bash
bash -c 'for i in {109..228}; do echo -n "$i:";grep -q "Connect" &> /dev/null <<< `echo | timeout 3s telnet 192.168.168.$i 7000`; echo $?;done
```



## 2. 系统相关命令

### top

#### 实时降序CPU或内存占用

>top命令：在终端中运行top命令，然后按下"Shift + M"键，按内存使用量进行排序。这将显示当前运行的所有进程按照内存使用量从高到低的列表。您可以查看RES（Resident Set Size）列，它表示进程实际占用的物理内存。
```sh
Shift + M 按mem降序
Shift + C 按CPU降序
```

显示内存、swap占用百分比并显示进度条
```sh
top 按下然后按m
```

显示cpu占用百分比并显示进度条
```sh
top 按下然后按t
```

### ps

#### 查看占用CPU最多的进程

```bash
ps aux --sort=-%cpu | head
```

#### 查看占用内存最多的进程

```bash
ps aux --sort=-%mem | head
ps aux --sort -rss
```

#### 过滤指定程序导致标题遗漏问题

使用多重匹配，多匹配一次`PID`就可以解决

```bash
ps axw -o pid,ppid,user,%cpu,vsz,wchan,command | egrep '(nginx|PID)'
```

#### 查看指定进程的CPU&MEM使用率

```bash
ps -p $(pgrep -f nginx -o) -o pid,user,%cpu,%mem,cmd
```

### pidstat

#### 查看指定进程CPU内存占用

**可以优先使用ps**

```bash
./pidstat -ru -p $(pgrep -o nginx) -h --human 1 -l
```

#### 查看指定进程IO占用

```bash
./pidstat -d -p $(pgrep -o nginx) -h --human 1 -l
```

### systemd

```shell
# 列出正在运行的 Unit
$ systemctl list-units
# 重新加载一个服务的配置文件
$ sudo systemctl reload apache.service
# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload
# 显示某个 Unit 的所有底层参a数
$ systemctl show httpd.service

# 查看指定进程的日志
$ sudo journalctl _PID=1
# 查看某个路径的脚本的日志
$ sudo journalctl /usr/bin/bash
# 查看指定用户的日志
$ sudo journalctl _UID=33 --since today
# 查看某个 Unit 的日志
$ sudo journalctl -u nginx.service
$ sudo journalctl -u nginx.service --since today
# 实时滚动显示某个 Unit 的最新日志
$ sudo journalctl -u nginx.service -f
```

### iptables

```shell
# 清空自定义链、清空规则、清空规则计数器
iptables -X
iptables -F
iptables -Z

# 指定多个端口
iptables -A INPUT -s 172.16.0.0/16 -d 172.16.100.10 -p tcp -m multiport --dports 20:22,80 -j ACCEPT


iptables -A INPUT -s 172.16.0.0/16 -d 172.16.100.10 -p tcp -m multiport --dports 20:22,80 -j ACCEPT


# iprange扩展
# [!] --src-range from[-to] 源IP地址范围
# [!] --dst-range from[-to] 目标IP地址范围
iptabels -A INPUT -d 172.16.1.100 -p tcp --dport 80 -m iprange --src-range 172.16.1.5-192.16.1.10 -j DROP

# mac 扩展
# mac 模块可以指明源MAC地址,，适用于：PREROUTING, FORWARD，INPUT chains
# [!] --mac-source xx:xx:xx:xx:xx
iptables -A INPUT -s 172.16.0.100 -m mac --mac-source 
00:50:56:12:34:56 -j ACCEPT


# 持久保存
# centos6:
service iptables save
# centos7、8
iptables-save >/PATH/TO/SOME_RULES_FILE

# 加载规则 centos7：
iptables-restore < /PATH/FROM/SOME_RULES_FILE
-n, -noflush: 不清除原有规则
-t, -test: 仅生成规则集，但不提交

# centos6
#会自动从/etc/sysconfig/iptables 重新载入规则
service  iptables  restart
```

#### 防火墙规则示例

```shell
# ssh 端口重要
iptables -A INPUT -p tcp -m state --state=NEW,ESTABLISHED --dport 5151 -j ACCEPT

# 服务相关
# ng注意端口
iptables -A INPUT -p tcp -m state --state=NEW,ESTABLISHED --dport 6100 -j ACCEPT

# 其它规则
iptables -A INPUT -m state --state=INVALID -j DROP
iptables -A INPUT -i lo  -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -m iprange --src-range 192.168.12.89-192.168.12.90 -j ACCEPT
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 10050 -j ACCEPT

# END
iptables -A INPUT -p all -s any/0 -j DROP
```



```shell
# ssh 端口重要
iptables -A INPUT -p tcp -m state --state=NEW,ESTABLISHED --dport 5151 -j ACCEPT


# 服务相关，尽早匹配
# ng注意端口
iptables -A INPUT -p tcp  -m state state=NEW,ESTABLISHED,RELATED --dport 6000 -j ACCEPT


# 其它规则
iptables -A INPUT -m state INVALID -j DROP
iptables -A INPUT -i lo  -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -m iprange --src-range 192.168.12.89-192.168.12.90 -j ACCEPT


# END
iptables -A INPUT -p all -s any/0 -j DROP
```

```shell

# ssh 端口重要
iptables -A INPUT -p tcp -m state --state=NEW,ESTABLISHED --dport 5151 -j ACCEPT


# 服务相关，尽早匹配
# 管理服务对ng放行
iptables -A INPUT -p tcp -s 192.168.12.12/24 -m state state=NEW,ESTABLISHED,RELATED --dport 6200 -j ACCEPT
iptables -A INPUT -p tcp -s 192.168.12.13/24 -m state state=NEW,ESTABLISHED,RELATED --dport 6200 -j ACCEPT
iptables -A INPUT -p tcp -s 192.168.12.61/24 -m state state=NEW,ESTABLISHED,RELATED --dport 6200 -j ACCEPT
iptables -A INPUT -p tcp -s 192.168.12.62/24 -m state state=NEW,ESTABLISHED,RELATED --dport 6200 -j ACCEPT



# 其它规则
iptables -A INPUT -m state INVALID -j DROP
iptables -A INPUT -i lo  -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -m iprange --src-range 192.168.12.89-192.168.12.90 -j ACCEPT

# END
iptables -A INPUT -p all -s any/0 -j DROP
```

```shell
sudo -i

systemctl stop firewalld
systemctl disable firewalld


iptables -X
iptables -F
iptables -Z

netstat -anpt | grep ssh | head

iptables -A INPUT -p tcp -m state --state=NEW,ESTABLISHED --dport 5151 -j ACCEPT
iptables -A INPUT -p tcp -s 192.168.12.12 -m state --state=NEW,ESTABLISHED,RELATED --dport 6200 -j ACCEPT
iptables -A INPUT -p tcp -s 192.168.12.13 -m state --state=NEW,ESTABLISHED,RELATED --dport 6200 -j ACCEPT
iptables -A INPUT -p tcp -s 192.168.12.61 -m state --state=NEW,ESTABLISHED,RELATED --dport 6200 -j ACCEPT
iptables -A INPUT -p tcp -s 192.168.12.62 -m state --state=NEW,ESTABLISHED,RELATED --dport 6200 -j ACCEPT


# 其它规则
iptables -A INPUT -m state --state=INVALID -j DROP
iptables -A INPUT -i lo  -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -m iprange --src-range 192.168.12.89-192.168.12.90 -j ACCEPT
iptables -A INPUT -m iprange --src-range 192.168.12.14-192.168.12.17 -j ACCEPT
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 10050 -j ACCEPT

#iptables -A INPUT -p all -s any/0 -j DROP
```

### yum(推荐使用dnf)

#### yum-config-manager

```bash
yum -y install yum-utils
```

 建议使用阿里云yum源：（推荐）

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### last

#### 重启服务器的时间

```bash
last -x reboot
```

### pmap

#### 当前进程占用内存的总量

~~writeable/private 好像是当前进程占用内存的总量，可能不对~~

```sh
(name=idreader; pmap -d $(pgrep -fn $name))
```

### pgrep

#### grep过滤命令help参数

```bash
pgrep --help1 |& grep -wEc -- '-l|-f'
pgrep --help1 |& grep -wEc -- '-a|-f'

pgrep --help1 | grep -wEc -- '-l|-f'
pgrep --help1 | grep -wEc -- '-a|-f'
```

### sudo

#### 判断用户是否具有sudo权限

```shell
sudo -l -U zhangsan

```

### 添加用户为sudo用户
```shell
sudo usermod -aG wheel yanghe
```
### nmcli

#### 配置IP地址

```shell
nmcli con show
 nmcli con mod enp0s8 ipv4.addresses 192.168.56.119/24 ipv4.gateway 192.168.56.1 ipv4.method manual
 nmcli con up enp0s8
```

## 3. 文件操作相关命令

### dasel

```bash

echo '{"users": [{"name": "Tom"}]}' | dasel put object -p json -t string '.users.[]' name=Frank
./dasel_linux_amd64 delete -f  example.json -m '.(?:host=192.168.168.33)'

./dasel_linux_amd64 -n -f  example.json -m '.(?:host=192.168.168.911)'

 ./dasel_linux_amd64 -n -f  example.json -m '[0].host' 
 # 将删除的内容输出到xxx.json，并不会删除源文件内容，如果不指定-o 则会在源文件上进行修改
./dasel_linux_amd64 delete -f example.json '[0]'  -o xxx.json

```

#### 参考

- [TomWright/dasel: Select, put and delete data from JSON, TOML, YAML, XML and CSV files with a single tool. Supports conversion between formats and can be used as a Go package. (github.com)](https://github.com/TomWright/dasel#one-tool-to-rule-them-all)
- jq表达式在线验证[jq play](https://jqplay.org/)

### tar

#### 通过标准输入进行打包

`ls -f`替换find性能会提高很多

```bash
find $max_dir -name *.json | head -$file_num | tar zcvf $tar_dir/$tar_name -T - --remove-file >>$tar_dir/tar.log
```



### awk

#### 使用正则进行分组

```bash
echo -e "Saturday\ntoday" | awk 'match($1, /S(.*)?t(.*)y/, groups) { print groups[1]; print groups[2]; }'
```

### pcregrep & pcre2grep
专门使用PCRE规范来实现的grep
使用正则表达式：
```shell
pcregrep  -o1 'echo(.*?)d' hello-world.sh
```
>主要原因是grep支持三种正则表达式BRE,ERE,PCRE，而其默认使用的是BRE，但\d是定义在PCRE中的，所以grep默认是不支持\d的。

pcre2grep 有个好用的功能：
```shell
$ echo -e 'name:zhangsan,age:18 \n name:lisi,age:20' | pcre2grep -O '$1 $2' 'name:(\w+),age:(\d+)'
zhangsan 18
lisi 20
```
- 参考
  - https://www.cnblogs.com/codelogs/p/16060372.html


### tail
#### tail -f加管道命令后不能及时输出的问题(IO缓冲

1. 使用stdbuf命令
`tail -fn0 a.log | stdbuf -oL grep "F3 00" | grep SR`
3. 把标准输出重定向到标准错误上，并把错误也给管道（因为stderr是无缓冲的）
`tail -f >&2 |& awk`




## 4. 中间件相关命令

### nginx

#### 通过信号控制nginx

NGINX默认master进程pid文件在~~/usr/local/nginx/logs/nginx.pid~~，我的Mint在`/var/run/nginx.pid`
![[Snipaste_2023-07-12_14-17-01.png]]
主进程支持以下信号：

- TERM,INT ：快速关闭
- QUIT： 优雅的关闭
- HUP ：更改配置，使用新配置启动进程，关闭旧进程
- USR1：重新打开日志文件
- USR2：升级可执行文件
- WINCH：优雅的关闭工作进程，单个工作进程也可以用信号控制，不是必须的
  - TERM, INT：快速关闭
  - QUIT：优雅的关闭
  - 重新打开日志文件
  - WINCH：终止异常调试（需要启用debug_points）

#### 更改nginx配置

```bash
ps axw -o pid,ppid,user,%cpu,vxz,wchan,command | egrep '(nginx|PID)'
kill -HUP 33126
```

PID为33126的进程仍在工作，一段时间后就会退出

#### 日志文件轮询

```bash
mv access.log access_bak.log
kill -USR1 11604
```

#### 动态升级可执行文件

[通过信号控制 nginx - Nginx 教程 (hxstrive.com)](https://www.hxstrive.com/subject/nginx/796.htm)
不如kill掉，替换

### Docker

```shell
docker save -o docker-elk-filebeat.tar docker-elk-filebeat:latest # 导出镜像
 docker load -i docker-elk-filebeat.tar # 导入镜像
docker kill 容器ID或镜像名字 # 删除容器
docker rmi 镜像名称 # 删除镜像

# 一次性删除多个容器实例：
docker rm -f $(docker ps -a -q)
docker rm -a -q | xargs docker rm

docker export 容器ID > 文件名.tar # 导出容器
# 恢复到新的镜像，然后创建新的容器
cat 文件名.tar | docker import - 
docker run -it ...

# 容器打包成镜像
docker commit -m "描述" -a "作者" 容器ID:tag

# 容器卷
docker run -it --privileged=true -v /宿主机目录:/容器内目录:rw 镜像名 (默认是rw)，可以改为ro（read only），容器卷只读，不可写入。宿主机可以写入，同步给容器。

# docker卷继承
docker run -it --privileged=true --volumes-from 父类 --name u2 ubuntu

# 查看所有虚玄镜像
docker image ls -f dangling=true
# 删除所有虚玄镜像
docker image prune

```

Docker挂在主机目录访问如果出现connot open directory ：Permission denied
解决办法：在挂载目录后多加一个--privileged=true

### docker-compose

```shell
docker-compose up # 启动所有docker-compose服务
docker-compose up -d # 启动所有docker-compose服务在后台运行
docker-compose down # 停止并删除容器、网络、卷、镜像
docker-compose exec 容器ID /bin/bash # 进入容器内部
docker-compose ps # 展示当前docker-compose编排过的运行的所有容器
docker-compose top # 展示当前docker-compose编排过的容器进程
docker-compose logs 容器ID # 查看容器输出日志
docker-compose config # 检查配置
docker-compose config -q # 检查配置，有问题才输出
docker-compose restart # 重启服务
docker-compose start # 启动服务
docker-compose stop # 停止服务


## 5. shell相关命令

### shuf

**在使用`shuf`命令时，请注意，它可能无法处理非常大的文件，因为它会将整个文件加载到内存中进行处理。对于大型文件，您可能需要寻找其他工具或方法来进行随机化处理。**

```sh
shuf input.txt

# 覆盖
shuf input.txt -o output.txt

# 随机抽取5行
shuf -n 5 input.txt

# 生成一个随机数
shuf -i 1-100 -n 1

# 从标准输入读取内容
echo -e "line1\nline2\nline3" | shuf

# 生成指定范围小数
min=0; max=1; scale=100; shuf -i "$((min*scale))-$((max*scale))" -n 1 | awk '{ print $1 / 100 }'

```

## 5. shell脚本相关命令
### numfmt

```bash
yanghe@IkunOS:~/coreutils-9.3/src$ ./numfmt --help
用法: ./numfmt [选项]... [数字]...
对 <数字> 进行重新格式化，如果未指定 <数字>，则从标准输入读取数字。

长选项的必选参数对于短选项也是必选的。
      --debug          对无效的输入内容打印警告信息
  -d, --delimiter=X    使用 X 而不是空格作为字段分隔符
      --field=FIELDS   replace the numbers in these input fields (default=1);
                         see FIELDS below
      --format=格式    使用 printf 风格的浮点 <格式>；
                         参见下文 <格式> 部分以了解细节
      --from=单位      使用 <单位> 对输入数字进行自动缩放；默认值为 "none"；
                         参见下文 <单位> 部分
      --from-unit=N    指定输入的单位大小（而非默认值 1）
      --grouping       使用区域设置中定义的方法对数字进行分组，例如 1,000,000
                         （意味着该选项在 C/POSIX 区域设置下无效果）
      --header[=N]     将前 N 行视为头部，原样输出（不进行转换）；
                         如未指定 N，默认值为 1
      --invalid=模式   遇到无效数字时的处理模式： <模式> 可以是：
                         abort（默认）、fail、warn、ignore
      --padding=N      将输出填充至 N 字符长度；若 N 值为正，则使用
                         右对齐；若 N 值为负，则使用左对齐；
                         若输出宽度大于 N 的绝对值，则不填充；
                         默认行为是如果输入含有空格，则自动填充
      --round=方法     缩放时使用指定 <方法> 进行舍入；<方法> 可以是：
                         up、down、from-zero（默认）、towards-zero、nearest
      --suffix=后缀    将 <后缀> 添加到输出数字之后，并允许输入数字中
                         含有该 <后缀>
      --to=单位        使用 <单位> 对输出数字进行自动缩放；参见下文 <单位> 部分
      --to-unit=N      指定输出的单位大小（而非默认值 1）
  -z, --zero-terminated    以 NUL 空字符而非换行符作为行分隔符
      --help        display this help and exit
      --version     output version information and exit

<单位> 选项：
  none       不进行自动缩放；带后缀会触发错误
  auto       接受可选的单字符或双字符后缀：
               1K = 1000,
               1Ki = 1024,
               1M = 1000000,
               1Mi = 1048576,
  si         接受可选的单字符后缀：
               1K = 1000,
               1M = 1000000,
               ...
  iec        接受可选的单字符后缀：
               1K = 1024,
               1M = 1048576,
               ...
  iec-i      接受可选的双字符后缀：
               1Ki = 1024,
               1Mi = 1048576,
               ...

<字段> 支持 cut(1) 风格的字段范围：
  N    第 N 个字段，从 1 开始计数
  N-   从第 N 个字段开始，到行尾
  N-M  第 N 个到第 M 个字段（含第 N 个和第 M 个）
  -M   第 1 个到第 M 个字段（含第 1 个和第 M 个）
  -    所有字段
多个字段或字段范围可以使用逗号分隔

<格式> 必须适合打印一个浮点参数 "%f"。
可选的单引号 (%'f) 将启用 --grouping（如果当前区域设置支持）。
可选的宽度值 (%10f) 会填充输出。可选的以 0 开头 (%010f) 的宽度
会使用 0 对数字进行填充。可选的负值 (%-10f) 会启用左对齐。
可选的精度 (%.1f) 会覆盖由输入所确定的精度。

如果所有输入数字均成功转换，退出状态值将为 0。
默认情况下，./numfmt 将会在第一个转换错误出现时停止并将退出状态设为 2。
如果使用了 --invalid='fail'，程序每次遇到转换错误时都将输出警告，并在
最后将退出状态设为 2。如果使用了 --invalid='warn'，程序每次遇到转换
错误时都输出诊断信息，但最后的退出状态将为 0。如果使用了
--invalid='ignore'，程序在遇到转换错误时将不输出诊断信息，最后的
退出状态也将为 0。

示例：
  $ ./numfmt --to=si 1000
            -> "1.0K"
  $ ./numfmt --to=iec 2048
           -> "2.0K"
  $ ./numfmt --to=iec-i 4096
           -> "4.0Ki"
  $ echo 1K | ./numfmt --from=si
           -> "1000"
  $ echo 1K | ./numfmt --from=iec
           -> "1024"
  $ df -B1 | ./numfmt --header --field 2-4 --to=si
  $ ls -l  | ./numfmt --header --field 5 --to=iec
  $ ls -lh | ./numfmt --header --field 5 --from=iec --padding=10
  $ ls -lh | ./numfmt --header --field 5 --from=iec --format %10f

GNU coreutils 在线帮助：<https://www.gnu.org/software/coreutils/>
请向 <http://translationproject.org/team/zh_CN.html> 报告任何翻译错误
完整文档 <https://www.gnu.org/software/coreutils/numfmt>
或者在本地使用：info '(coreutils) numfmt invocation'

```


### 随机生成密码

```bash
tr -dc 'A-Za-z0-9_!@#$%^&*()-+=' < /dev/urandom | head -c 12
```

### shc 

#### 加密shell脚本

```bash
shc -r -f filename.sh
# 开始静态编译，让可以跨平台运行
cc -Wall -lutil -static -o filename filename.x.c
```


## 6. 包管理与环境管理
### coda

```dos
# 查看conda版本，验证是否安装 
conda -V 

# 更新conda至最新版本
conda update conda 

# 更新所有包
conda update --all 

# 更新指定的包
conda update package_name 

# 创建名为env_name的新环境，并在该环境下安装名为package_name 的包，
# 可以指定新环境的版本号，例如：conda create -n python3 python=python3.7 numpy pandas，
# 创建了python3环境，python版本为3.7，同时还安装了numpy pandas包
conda create -n env_name package_name 

# 切换至env_name环境
conda activate env_name 

#退出环境
conda deactivate 

# 显示所有已经创建的环境
conda info -e  
# 或者使用 
conda env list

# 复制old_env_name为new_env_name
conda create --name new_env_name --clone old_env_name 

# 删除环境
conda remove --name env_name --all 

# 查看所有已经安装的包
conda list 

# 在当前环境中安装包
conda install package_name 

# 在指定环境中安装包
conda install --name env_name package_name 

# 删除指定环境中的包
conda remove -- name env_name package 

# 采用上一步方法删除环境失败时，可采用这种方法
conda env remove -n env_name 


# 删除当前环境中的包
conda remove package 

```







































```

#### yaml 文件级

Docker Compose 的 YAML 文件包含4个一级 key:version、services、networks、volumes。

- version 是必须指定的，位于文件的第一行。它定义了 Compose 文件格式（主要是API）的版本。
- services 用于定义不同的应用服务。
- networks 用于指引 Docker 创建新的网络。
- volumes 用于指引 Docker 来创建新的卷。

## strip — 丢弃对象文件中的符号和其他数据

在嵌入式用的比较多，嵌入式存储空间一般非常小，一般服务器无所谓

```shell
yanghe@IkunOS:/tmp$ file hello_world
hello_world: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=0266e8c8b510581d97cd75551682b839d8c8ba47, for GNU/Linux 3.2.0, not stripped
yanghe@IkunOS:/tmp$ du hello_world
1024 hello_world
yanghe@IkunOS:/tmp$ strip -s hello_world
yanghe@IkunOS:/tmp$ du hello_world
940 hello_world
yanghe@IkunOS:/tmp$ file hello_world
hello_world: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=0266e8c8b510581d97cd75551682b839d8c8ba47, for GNU/Linux 3.2.0, stripped
```

参考：

- [strip命令 | CS笔记 (pynote.net)](https://cs.pynote.net/sf/c/cdm/202110268/)
- [Linux 可执行文件瘦身指令 strip 使用示例 - ENG八戒 - 博客园 (cnblogs.com)](https://www.cnblogs.com/englyf/p/17510353.html)



## 其它
### /proc/pid/status 查看内存使用率

/proc/[PID]/status文件：每个运行的进程都在/proc目录下有一个以进程ID命名的目录。进入特定进程的目录（例如/proc/12345），然后查看status文件。在status文件中，可以找到VmRSS字段，它表示进程实际占用的物理内存。

`cat /proc/12345/status | grep VmRSS`

### shell将stderr输出到stdout

```bash
# |& == 2>&1
```

### python3 开启文件服务器

```bash
python3 -m http.server
```

### 查看进程线程&进程详细信息

```bash
cat /proc/102551/status | more
Name: java
Umask: 0002
State: S (sleeping)
Tgid: 102551
Ngid: 0
Pid: 102551
PPid: 102344
TracerPid: 0
Uid: 1002 1002 1002 1002
Gid: 1002 1002 1002 1002
FDSize: 4096
Groups: 1002 
VmPeak: 23980072 kB
VmSize: 23818032 kB
VmLck:        0 kB
VmPin:        0 kB
VmHWM:  9205188 kB
VmRSS:  9057300 kB
RssAnon:  9049584 kB
RssFile:     7716 kB
RssShmem:        0 kB
VmData: 23656776 kB
VmStk:      132 kB
VmExe:        4 kB
VmLib:    17316 kB
VmPTE:    18692 kB
VmSwap:    25872 kB
Threads: 213
SigQ: 0/127861
SigPnd: 0000000000000000
ShdPnd: 0000000000000000
SigBlk: 0000000000000004
SigIgn: 0000000000000002
SigCgt: 2000000181005ccd
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: 0000001fffffffff
CapAmb: 0000000000000000
Seccomp: 0
Speculation_Store_Bypass: vulnerable
Cpus_allowed: ffffffff,ffffffff,ffffffff,ffffffff,ffffffff
Cpus_allowed_list: 0-159
Mems_allowed: 00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000
000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000
,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00
000000,00000000,00000001
Mems_allowed_list: 0
voluntary_ctxt_switches: 478194
nonvoluntary_ctxt_switches: 170

```
