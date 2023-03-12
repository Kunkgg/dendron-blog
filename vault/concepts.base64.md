---
id: 6yjrthbsilq6k6kza17yyv9
title: Base64
desc: ''
updated: 1678632729832
created: 1678631596779
---

## Base64

### 为什要有 Base64 ?

参考:

- https://developer.mozilla.org/en-US/docs/Glossary/Base64
- https://www.quora.com/What-is-base64-encoding/answer/Rajesh-Prajapati-71
- https://en.wikipedia.org/wiki/Base64

觉得 MDN 介绍的理由比较容易理解.

Base64 是一种把 `二进制转换为字符串` 的编码, 目地是为了在为字符串设计的通信机制中传递二进制文件.
比如, http 中可以把图片以 data="<base64编码>" 的形式传递.
或者邮件的附件.

### Base64 的编码规则

base64 编码规则比较容易理解, 把 bit 序列按每 6 bits 为一组划分, 有 pow(2, 6) = 64 种值.
数字 0 - 63 对应一个一个字符. 字符的范围是 '[A-Za-Z0-9+/]'.
在编码的最后可能需要用 `=` 作为对齐字符, 凑够 24 bits.

### 工具

在 Python 中可以使用 `base64` 标准库模块处理编解码, 例如:

```python
import base64
In [20]: base64.b64encode(b'Man')
Out[20]: b'TWFu'
In [21]: base64.b64encode(b'a')
Out[21]: b'YQ=='
```