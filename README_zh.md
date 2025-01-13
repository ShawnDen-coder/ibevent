# IBEvent

IBEvent 是一个轻量级的事件驱动框架，专为 Interactive Brokers API 设计，让实时市场数据处理变得简单直观。

[English](README.md)

## 特性

- 简单直观的事件驱动 API
- 同时支持同步和异步处理器
- 基于优先级的事件处理机制
- 自动的数据源绑定
- 基于 [ib_async](https://github.com/erdewit/ib_async) 构建

## 安装

```bash
pip install ibevent
```

## 快速开始

这是一个订阅 USD/JPY 外汇数据的简单示例：

```python
from ib_async import IB
from ibevent.events import IBEventType
from ib_async.contract import Forex

# 创建 IB 连接
ib = IB()
ib.connect('127.0.0.1', 7497, clientId=1)

# 创建 USD/JPY 合约
usd_jpy = Forex('USDJPY')

# 注册 bar 数据更新处理器
@ib.events.register(
    IBEventType.BAR_UPDATE,
    bind_to=ib.reqRealTimeBars(
        usd_jpy,
        barSize=5,
        whatToShow='MIDPOINT'
    )
)
def handle_bar(ib, bars, has_new_bar):
    if has_new_bar:
        print(bars[-1])

# 运行直到被中断
try:
    ib.run()
except KeyboardInterrupt:
    ib.disconnect()
    print("\n正在关闭...")
```

## 事件类型

IBEvent 通过 `IBEventType` 枚举支持所有 IB API 事件：

- `BAR_UPDATE`：实时 bar 数据更新
- `ERROR`：来自 IB API 的错误事件
- `CONNECTED`：连接建立事件
- `DISCONNECTED`：连接断开事件

## 事件处理器

### 基础处理器

```python
@ib.events.register(IBEventType.BAR_UPDATE)
def handle_bar(ib, bars, has_new_bar):
    if has_new_bar:
        print(bars[-1])
```

### 优先级处理器

```python
@ib.events.register(IBEventType.BAR_UPDATE, priority=10)
def high_priority_handler(ib, bars, has_new_bar):
    # 这个处理器会首先运行
    pass
```

### 异步处理器

```python
@ib.events.register(IBEventType.BAR_UPDATE)
async def handle_bar(ib, bars, has_new_bar):
    if has_new_bar:
        await process_bar(bars[-1])
```

### 数据源绑定

```python
@ib.events.register(
    IBEventType.BAR_UPDATE,
    bind_to=ib.reqRealTimeBars(contract)
)
def handle_bar(ib, bars, has_new_bar):
    pass
```

## 示例

查看 [examples](examples/) 目录获取更多示例：

- [基础用法](examples/basic_usage.py)：简单的实时 bar 数据订阅
- [多数据流](examples/multiple_streams.py)：使用优先级处理多个数据流
- [异步处理器](examples/async_handlers.py)：在事件处理器中使用 async/await

## 系统要求

- Python 3.7+
- ib_async
- Interactive Brokers TWS 或 IB Gateway

## 贡献

欢迎提交 Pull Request 来改进这个项目！

## 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件
