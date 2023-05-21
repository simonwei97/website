---
title: "Linux交叉编译解决办法"
date: 2021-07-21T20:34:13+08:00
draft: false
# page title background image
bg_image: "images/backgrounds/page-title.jpg"
# about image
image: "images/blog/x86_arm.jpg"
# meta description
description: "Linux 交叉编译"
# categories
categories: ["Linux"]
# tags
tags: ["Linux", "交叉编译"]
# type
type: "post"

aliases: ""
---

## 1.前提

Linux 物理机内核版本需高于 `4.18` 。如下：

```bash
[root@test ~]$ uname -a
Linux test 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

## 2.操作步骤

### 2.1.安装交叉编译工具， 用于编译 arm64 版本程序

1. 下载[交叉编译工具](https://releases.linaro.org/components/toolchain/binaries/4.9-2017.01/aarch64-linux-gnu/), 这里选择文件 `gcc-linaro-4.9.4-2017.01-x86_64_aarch64-linux-gnu.tar.xz` 下载
2. 新建安装目录 `mkdir -p /usr/local/ARM-toolchain`
3. 将安装包解压到该目录下 `tar -xf gcc-linaro-4.9.4-2017.01-x86_64_aarch64-linux-gnu.tar.xz -C /usr/local/ARM-toolchain/`
4. 修改 `/etc/bash.bashrc` 文件，加入如下配置

```bash
# Add ARM toolschain path
if [ -d /usr/local/ARM-toolchain/gcc-linaro-4.9.4-2017.01-x86_64_aarch64-linux-gnu/bin ] ; then
  PATH=/usr/local/ARM-toolchain/gcc-linaro-4.9.4-2017.01-x86_64_aarch64-linux-gnu/bin:"${PATH}"
fi
```

5. 执行 `source /etc/bash.bashrc`，使配置生效。
6. 执行 `aarch64-linux-gnu-gcc -v` 测试是否可用。
7. 此后在该系统下可指定使用 `aarch64-linux-gnu-gcc` 作为交叉编译器
8. 进入到目标项目目录下，执行命令 `make -xx` 编译 arm 版本程序。

### 2.2.安装 docker 版的 qemu， 用于运行测试或打包镜像

1. 下载安装包： https://github.com/multiarch/qemu-user-static
2. 安装官方文档执行如下操作，安装配置模拟环境：

```bash
uname -m

# 返回 x86_64

docker run --rm -t arm64v8/ubuntu uname -m standard_init_linux.go:211: exec user process caused "exec format error"

docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

docker run --rm -t arm64v8/ubuntu uname -m aarch64
```

3. 上面步骤完成后，即可在该系统下直接运行 arm64 版本的可执行文件。
4. 进入到目标项目目录下，执行命令 `make image -xx` 编译 arm 版本程序。
