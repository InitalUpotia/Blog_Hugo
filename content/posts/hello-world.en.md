---
date : '2026-10-09T18:01:41+08:00'
draft : false
title : 'Hello World'
tags: ["test"]
author: "iNITAL"
math: true
cover:
    image: "image-for-test/test1.webp"
    alt: "A sample cover image"
    caption: "~ Obligatory Yachiyo to start ~"
---

## Basic

This is a file to test the fearures of the markdown.

**Bold**，*italic*，~~delete~~，^superscript^，~subscript~，==highlight==，++insert++

- list without order
- ——

1. ordered list
2. ——

## Table

|   A   |   B   |   C   |
| ---- | ---- | ---- |
|   1   |   2   |   3   |

## Code

~~~cpp
#include <iostream>

int main (){
    std::cout<<"Hello World!\n";
    return 0; 
}
~~~

Inline code: `printf("Hello World!\n");`

## Footnote

footnote[^1]

## KaTeX

KateX in full line.

$$
E = mc^2
$$

Inline KaTeX: $A ^ 2 = B ^ 2 + C ^ 2$

## Mermaid graph

```mermaid
flowchart LR
  A[start] --> B{if}
  B -- y --> C[end]
  B -- n --> D[continue]
```

## Image



## 网易云音乐

{{< netease-cloud-music id=22805132 type=song height=80 width=500 title="Masquerade" >}}

## Bilibili

{{< bilibili "BV1uhYueJErx" >}}

[^1]: Define the footnote
