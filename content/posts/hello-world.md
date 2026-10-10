---
date : '2026-10-09T18:01:41+08:00'
draft : false
title : 'Hello World'
tags: ["测试"]
author: "iNITAL"
cover:
    image: "image-for-test/test1.webp"
    alt: "A sample cover image"
    caption: "~ 八千代镇楼 ~"
---
## Hello World!

## 基础格式

这是一个测试markdown性能的文件。

**粗体**，*斜体*，<u>下划线</u>，~~删除~~，^上标^，~下标~，==高亮==，++插入++

- 无序列表
- ——

1. 有序列表
2. ——

## 表格：

|   A   |   B   |   C   |
| ---- | ---- | ---- |
|   1   |   2   |   3   |

## 代码

~~~cpp
#include <iostream>

int main (){
    std::cout<<"Hello World!\n";
    return 0; 
}
~~~

行内代码：`printf("Hello World!\n");`

## 脚注

脚注[^1]

## 公式
整行公式

$$
E = mc^2
$$

行内公式经过测试似乎不行

## Mermaid图表

暂时不考虑弄。

## 图片

暂时不知道为什么文中图片引用不了。

[^1]: 定义脚注
