---
title: "Linux常用命令"
description: "Linux常用命令"
date: 2023-03-23T12:18:40+08:00
draft: false
tags: ["linux"]
categories: ["linux"]
type: "post"
bg_image: "images/backgrounds/page-title.webp"
image: "images/blog/Tuxmea.webp"
---

## 1. 统计访问 Nginx 次数最多的前十个 IP

```bash

awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr -k1 | head -n 10
```

## 2. 查看命令是否存在

```bash

command -v <command_type>

$ command -v foo >/dev/null 2>&1 || { echo >&2 "I require foo but it's not installed.  Aborting."; exit 1; }
$ type foo >/dev/null 2>&1 || { echo >&2 "I require foo but it's not installed.  Aborting."; exit 1; }
$ hash foo 2>/dev/null || { echo >&2 "I require foo but it's not installed.  Aborting."; exit 1; }
```

示例

```bash

if ! command -v <the_command> &> /dev/null
then
  echo "<the_command> could not be found"
  exit
fi


function command_exist() {
    if command -v $1 >/dev/null 2>&1; then
        echo "check <$1> pass"
    else
        echo "command <$1> does not exists"
        exit 1
    fi
}
```

## 3. 网络命令

### 3.1. ip 命令使用

```bash

# 显示网络接口信息
ip link show
ip link list

# 显示更加详细的设备信息
ip -s link list

# 显示系统路由信息
ip route show
ip route list

# 显示邻居表
ip neigh list

# 获取主机所有网络接口
ip link | grep -E '^[0-9]' | awk -F: '{print $2}'
```

### 3.2. netstat 命令

```bash

# 查看路由信息
netstat -rn

# 显示UDP端口号的使用情况
netstat -apu

# 显示TCP端口号的使用情况
netstat -apt

# 显示网卡列表
netstat -i

# 显示组播组的关系
netstat -g

# 显示网络统计信息
netstat -s
```

### 3.3. ss 命令检查连接

```bash

# 找出打开套接字/端口应用程序
ss -lp | grep 3306

# 显示所有状态为Established的HTTP连接
ss -o state established '( dport = :http or sport = :http )'
```

### 3.4. iptables 命令

```bash

# 查看防火墙
iptables -L INPUT --line-numbers

# 删除iptables规则
iptables -D INPUT 7
```

## 4. 文件存储命令

```bash

# 会列出当前目录下每个文件的大小，同时也会给出当前目录下所有文件大小总和
ls -lht

# 列出当前文件夹下所有文件对应的大小
# 把*替换为具体的文件名，会给出具体文件的大小
du -sh *
```

## 4. curl 命令

```bash

# 取得HTTP返回的状态码
curl -I -m 10 -o /dev/null -s -w %{http_code} -X GET xxxx.com


# 统计url的请求耗时
curl -o /dev/null -s -w "time_connect: %{time_connect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\n" "http://www.kklinux.com"
```
