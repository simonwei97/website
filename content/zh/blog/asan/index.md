---
title: "ASAN内存检测工具"
description: "ASAN内存检测工具"
date: 2023-01-23T12:18:40+08:00
draft: false
tags: ["go"]
categories: ["go"]
type: "post"
bg_image: "images/backgrounds/page-title.webp"
image: "images/blog/golang.jpg"
---

## 1. Address Sanitizer

Address Sanitizer(ASan) 是一个快速的内存错误检测工具，它可以检测以下问题：

-   访问已被释放的内存
-   堆上缓冲区访问溢出
-   栈上缓冲区访问溢出
-   全局缓冲区访问溢出
-   内存泄漏

gcc 4.8 版和 LLVM 3.1 版及以上支持了 [Address Sanitizer(ASan)](https://github.com/google/sanitizers)。

## 2. Go 使用说明

**go 1.18** 支持在 build 时指定 `-asan` 指令，详见 [Go 1.18 Release Notes](https://tip.golang.org/doc/go1.18)。

程序依赖 libasan 动态库。需要下载对应的 libasan 库。如

-   `libasan0`
-   `libasan4`

### 如何使用？

```go
go build -asan xx.go
```

之后运行程序，会显示具体的错误。

更多请参考 [runtime: enable address sanitizer in Go #44853](https://github.com/golang/go/issues/44853)。

### cgo 示例

代码越界访问堆对象错误。

```go
package main

/*
#cgo CFLAGS: -fsanitize=address
#cgo LDFLAGS: -fsanitize=address

#include <stdlib.h>
#include <stdio.h>
int *p;

void test(int *a) {
	// Heap out of bounds.
	int c = a[5];        // BOOM
	// We shouldn't get here; asan should stop us first.
	printf("a[5]=%d\n", c);
}
*/
import "C"

func main() {
	cIntSlice := []C.int{200, 201, 203, 203, 204}
	C.test(&cIntSlice[0])
}
```

### 其他

`gcc/clang` 编译

```bash
CFLAGS='-fsanitize=address' CXXFLAGS='-fsanitize=address' LDFLAGS='-fsanitize=address'
```

## 参考

-   https://github.com/google/sanitizers/wiki/AddressSanitizer
-   https://fuzzing-project.org/tutorial-cflags.html
-   https://aijishu.com/a/1060000000287557
