---
id: vGpPOrqIHmVLZbUOzLnbr
title: Logging
desc: ''
updated: 1641530301747
created: 1641530294507
---

[The Python Standard Library - logging](https://docs.python.org/3/library/logging.html#logging.getLogger)

[Logging HOWTO](https://docs.python.org/3/howto/logging.html#logging-basic-tutorial)

[Logging Cookbook](https://docs.python.org/3/howto/logging-cookbook.html#logging-cookbook)

## Concept

### 什么情况该使用日志？

任务 | 合适的工具 |
---------|----------|
在命令行脚本或程序中显示平常的输出 | `print()` |
在程序正常运行过程中报告事件发生 | `logging.info()` 或者 用于诊断的详细输出 `logging.debug()` |
发出有关特定运行时事件的警告 | 如果问题可以避免，并且可以修改客户端应用从而剔除该告警时应该使用标准库中的 `warnings.warn()`; 如果客户端应用不能处理该情况， 该事件仍然需要被记录时应该使用 `logging.warning()` |
报告有关特定运行时事件的错误 | 抛出异常 |
报告抑制了有关特定运行时事件的错误,未抛出异常 (比如, 处理某长期运行服务的错误) | `logging.error()`, `logging.exception()` 或 `logging.critical()` |

### 日志级别

Level Name | Level Num | 何时使用
:-------:|:--------:|---------
DEBUG | 10 | 详细信息，一般用于诊断问题
INFO | 20 | 确认程序按预期运行
WARNING | 30 | 软件仍按预期工作，但是告警出现了或将要出现一些非预期的问题
ERROR | 40 | 处理更严重的问题，软件的部分功能无法正常工作
CRITICAL | 50 | 严重的错误, 程序可能无法继续继续运行

### 默认日志级别

默认日志级别是 `WARNING`,
这意味默认情况下只有级别高于或等于 `WARNING` 级别的操作才会被处理。

![[python.libs.logging#basicconfigkwargs---none,1:#*]]

可以通过 `Logger` 对象的 `setLevel(level)` 方法设置。

### Logger 层级

所有 Logger 对象组成一个以 root Logger 为根的树状结构，
Logger 对象的 `parent` 属性指向父节点。

Logger 对象的名称是以 `.` 分隔的层次结构。

比如：

- 命名为 `foo.bar.baz` 的 Logger 对象是 `foo.bar` 的后继
- 而命名为 `foo.bar` 和 `foo.bam` 的 Logger 对象又是 `foo` 的后继
- 所有其他节点都是 root Logger 的子孙节点

### effective level

如果一个 Logger 对象没有显式设置 level, 会逐层向上，
直到找到一个显式设置的 level 作为有效 level。

如果直到 root Logger 还没有找到显式设置的 level,
则使用 root Logger 的 level 默认 WARNING 作为有效 level。

## 核心类型和方法

### LogRecord

代表一个日志记录， 其属性包含需要被记录的事件信息。 

LogRecord 对象一般由 Logger 对象在调用 `log()` 方法时自动生成， 
也可以手动调用 `Logger.makeRecord()` 方法生成。

LogRecord 主要属性：

- name, 生成该 LogRcord 对象的 Logger 名称
- level, 日志级别如 `DEBUG`, `INFO`, `WARNING`, `ERROR`。
在 logging 模块中这些都是整数常量。
- pathname, 调用日志所在源文件的完全路径。
- lineno, 调用日志所在源文件的行号。
- msg, 描述事件的字符串或字符串占位符。
- args, msg 中占位符对应的数据。
- exc_info, exception 信息。
- func, 调用日志所在的函数名称。
- sinfo, 栈信息。

LogRecord 主要方法:

- getMessage() -> str

### Logger

暴露给用户的使用接口。

代表日志管理器，把一组 Filter, Handler 和 Formatter 关联在一起。

一个 Logger 可以添加多个 Handler，
每个 Handler 又可以设置不同的 Formatter 和输出目的地。

Logger 主要方法:

- setLevel(level:int)
- addHandler(handler:Handler)
- addFilter(filter:Filter)
- getEffectiveLevel()

### RootLogger

特殊的 Logger， 类似单例，
由 Manager 对象保证程序生命周期中始终有且只有一个的 Logger 对象。

### Manager

管理 Logger 层级结构

实现很简单， 最主要的属性是 `loggerDict`, 用于记录所有 Logger 对象，
字典类型， key 为 Logger 的 name, value 为 Logger 对象。

![[python.libs.logging#getloggernameoptionalstr---logger,1:#*]]

Manager 对象通常也只有一个。

### Handler

用于规定如何将日志记录输出到合适的目的。

logging 模块实现了多种类型的 Handler, 
支持把日志输出到 std流， 文件， 内存, TCP, UDP, SMTP, HTTP 和消息队列等。

也支持常用的日志文件 Rotate 处理。

最常用的 Handler:

- StreamHandler, root Logger 默认的 Handler， 输出到 `stdout` 或 `stderr`
- FileHandler, 输出到文件。

Handler 主要方法:

- setFormatter(fmt)
- addFilter(filter:Filter)
- emit(record:LogRecord)

### Filter

可以被 Handler 和 Logger 使用， 用于实现基于 Logger 名的过滤。

所有匹配的 Logger 及其子孙 Logger 都被允许。例如：

`Filter('A.B')`, 允许 'A.B', 'A.B.C', 'A.B.C.D', 'A.B.D', 'A.B.XX.XX' 等。
但 ’A.BB', 'A.C', 'B.A' 等则不被允许。

如果参数为空， 即 `Filter()`, 表示允许所有。

Filter 主要方法:

- filter(record:LogRecord)

### Formatter

将 LogRecord 对象转化为最终的输出形式，通常是转化成字符串。

Formmater 主要参数：

- fmt, 日志 msg 格式化字符串模板。
    默认值：
    ```
    "%(levelname)s:%(name)s:%(message)s"
    ```
- datefmt, 日志时间格式化字符串模板。
    默认值：
    ```
    default_time_format = '%Y-%m-%d %H:%M:%S'
    default_msec_format = '%s,%03d'
    datefmt='%(default_time_format),%(default_msec_format)'
    ```
- style, 模板风格，支持三种模板风格，默认为 `%`
    - `%`, PercentStyle
    - `{`, StrFormatStyle
    - `$`, StringTemplateStyle

Formatter 主要方法:

- format(record:LogRecord)
- formatTime(record:LogRecord, datefmt:str=None)

### 日志数据流

- **简化版 logging flow**
```mermaid
graph LR;
    A["在代码中调用日志，
    <br>如 logger.info(...)"]-->B[生成 LogRecord 对象];
    B-->C[Handler]-->D[Formmater]-->E[输出到指定位置];
```

- **完整版 logging flow**
![完整版 logging flow](https://docs.python.org/3/_images/logging_flow.png)

### getLogger(name:Optional[str]) -> Logger

使用 getLogger(name:Optional[str])  方法时，
先在 Manager 对象的 `loggerDict` 属性中根据 name 查找，
如果没有找到则新建 Logger 对象，并记录在 `loggerDict` 中。

当 name 为空时， 返回 [[RootLogger|python.libs.logging#RootLogger]]。

### basicConfig(**kwargs) -> None

可以通过 `basicConfig(**kwargs)` 的 `level` 关键字参数设置 root Logger 的日志级别。

需要注意， `basicConfig(**kwargs)` 默认只有在 root Logger 没有任何 handler (也就是第一次初始化) 的时候才会更新 root Logger 的设置。

如果希望使用 `basicConfig(**kwargs)` 更新已存在的 root Logger 配置，
需要设置关键字参数 `force=True`。

## Usage

### 基本使用

```python
# logging to file
import logging
logging.basicConfig(
    filename='example.log',
    encoding='utf-8',
    level=logging.DEBUG,
    format='[%(asctime)s - %(name)s - %(levelname)s] %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S')
logging.debug('This message should go to the log file')
logging.info('So should this')
logging.warning('And this, too')
logging.error('And non-ASCII stuff, too, like Øresund and Malmö')
```

### Logging 至多目的地

```python
import logging

# set up logging to file - see previous section for more details
logging.basicConfig(level=logging.DEBUG,
                    format='%(asctime)s %(name)-12s %(levelname)-8s %(message)s',
                    datefmt='%m-%d %H:%M',
                    filename='/temp/myapp.log',
                    filemode='w')
# define a Handler which writes INFO messages or higher to the sys.stderr
console = logging.StreamHandler()
console.setLevel(logging.INFO)
# set a format which is simpler for console use
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
# tell the handler to use this format
console.setFormatter(formatter)
# add the handler to the root logger
logging.getLogger('').addHandler(console)

# Now, we can log to the root logger, or any other logger. First the root...
logging.info('Jackdaws love my big sphinx of quartz.')

# Now, define a couple of other loggers which might represent areas in your
# application:

logger1 = logging.getLogger('myapp.area1')
logger2 = logging.getLogger('myapp.area2')

logger1.debug('Quick zephyrs blow, vexing daft Jim.')
logger1.info('How quickly daft jumping zebras vex.')
logger2.warning('Jail zesty vixen who grabbed pay from quack.')
logger2.error('The five boxing wizards jump quickly.')
```

### 配置文件

logging 模块支持配置文件

- `logging.config.fileConfig()`
- `logging.config.dictConfig()`8421

