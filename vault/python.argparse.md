---
id: DspWknndqtb020XWKSNrW
title: Argparse
desc: ''
updated: 1641530382427
created: 1641530349904
---

[Argparse Tutorial](https://docs.python.org/3/howto/argparse.html)

[The Python Standard Library - argparse](https://docs.python.org/3/library/argparse.html#module-argparse)

## Concept

- **command**
- **sub-command**
- **positional argument**
    - 必选参数， 如果 miss 会报错, 指令无法运行
    - 通过参数的相对顺序区分
- **optional argument**
    - 可选参数
    - 一般不区分相对顺序
    - `short` 和 `long` 两种风格, 例如 `-h`, `--help`
    - 可以是开关，也可以传入 字符串、数据 等值
- **helper docs**
    - 命令描述说明
    - 参数说明
    - argparse 自动生成
    - `-h, --help` 打印

## argparse 核心类型和方法

- ArgumentParser 对象, 构造参数解析器对象
- add_argument(), 添加参数规则, 定义如何解析一项参数
- parse_args(), 解析参数

### add_argument() 

- `name or flags`, 参数名. 例如, `foo` or `-f`, `--foo`
- `action`, 指定如何处理参数。
    - `'store'`, action 的默认值, 存储参数到变量。
    - `'store_const'`, 和 `const` 一起使用，存储指定常量。
    - `'store_true'` 和 `'store_false'`, 是特殊形式的 `'store_const'`。
    - `'append'`, 参数 append 到一个 list 中。 实现多次使用相同的参数。
    - `'append_const'`,和 `const` 一起使用。
    - `'count'`, 参数没出现一次 +1， 默认值 None。 通常需要使用 `default` 指定默认值为 0.
    - `'help'`, 打印完整的 help doc.
    - `'version'`, 和 `version` 关键字参数一起使用, 打印命令 version 信息。
    - `'append'`, 参数 extend 到一个 list 中。 实现多次使用相同的参数。
    - 传入的 action 必须是 argparse.Action 类型，支持自定义。
- `nargs`, 定义参数对应的值的个数。
    - `N(整数)`, N 个
    - `?`, 1 个或 0 个
    - `*`, 任意个
    - `+`, 1 个或多个, 至少 1 个
- `const`, 常量。 与 `action='store_const'`, `action='append_const'` 或 `nargs='?'` 一起使用。
- `default`, 设置参数默认值。
- `type`, 指定参数数据类型，默认为字符串。
- `choices`, 指定参数备选值范围。
- `required`, 将 optional argument 设置为必选。
- `help`, 参数的帮助说明。
- `metavar`, 参数在 help doc 中的显示占位符。
- `dest`, 存储参数值的容器名。对应的在 parse_args() 方法返回的 Namespace 对象的属性名称。

## Usage

### Positional arguments

```python
# prog.py
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("square", help="display a square of a given number",
                    type=int)
args = parser.parse_args()
print(args.square**2)

# $ python3 prog.py 4
# 16
# $ python3 prog.py four
# usage: prog.py [-h] square
# prog.py: error: argument square: invalid int value: 'four'
```

### Optional arguments

```python
# prog.py
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("-v", "--verbose", help="increase output verbosity",
                    action="store_true")
args = parser.parse_args()
if args.verbose:
    print("verbosity turned on")

# $ python3 prog.py -v
# verbosity turned on
# $ python3 prog.py --help
# usage: prog.py [-h] [-v]
# 
# options:
#   -h, --help     show this help message and exit
#   -v, --verbose  increase output verbosity
```

### Combining Positional and Optional arguments

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("square", type=int,
                    help="display a square of a given number")
parser.add_argument("-v", "--verbosity", action="count", default=0,
                    help="increase output verbosity")
args = parser.parse_args()
answer = args.square**2
if args.verbosity >= 2:
    print(f"the square of {args.square} equals {answer}")
elif args.verbosity >= 1:
    print(f"{args.square}^2 == {answer}")
else:
    print(answer)
```

### Confilicting options 互斥参数

有的参数是互斥关系，比如 `--quiet` 和 `--verbose`, 需要使用 `add_mutually_exclusive_group()` 处理。 在 helper 文档中 `[-v | -q]`
表示互斥关系。

```python
# prog.py
import argparse

parser = argparse.ArgumentParser(description="calculate X to the power of Y")
group = parser.add_mutually_exclusive_group()
group.add_argument("-v", "--verbose", action="store_true")
group.add_argument("-q", "--quiet", action="store_true")
parser.add_argument("x", type=int, help="the base")
parser.add_argument("y", type=int, help="the exponent")
args = parser.parse_args()
answer = args.x**args.y

if args.quiet:
    print(answer)
elif args.verbose:
    print("{} to the power {} equals {}".format(args.x, args.y, answer))
else:
    print("{}^{} == {}".format(args.x, args.y, answer))

# $ python3 prog.py --help
# usage: prog.py [-h] [-v | -q] x y
# 
# calculate X to the power of Y
# 
# positional arguments:
#   x              the base
#   y              the exponent
# 
# options:
#   -h, --help     show this help message and exit
#   -v, --verbose
#   -q, --quiet
```

