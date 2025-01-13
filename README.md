# IBEvent

A lightweight, event-driven framework for Interactive Brokers API that makes it easy to handle real-time market data.

[中文文档](README_zh.md)

## Features

- Simple and intuitive event-based API
- Support for both synchronous and asynchronous handlers
- Priority-based event handling
- Automatic data source binding
- Built on top of [ib_async](https://github.com/erdewit/ib_async)

## Installation

```bash
pip install ibevent
```

## Quick Start

Here's a simple example that subscribes to USD/JPY forex data:

```python
from ib_async import IB
from ibevent.events import IBEventType
from ib_async.contract import Forex

# Create IB connection
ib = IB()
ib.connect('127.0.0.1', 7497, clientId=1)

# Create USD/JPY contract
usd_jpy = Forex('USDJPY')

# Register bar update handler
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

# Run until interrupted
try:
    ib.run()
except KeyboardInterrupt:
    ib.disconnect()
    print("\nShutting down...")
```

## Event Types

IBEvent supports all IB API events through the `IBEventType` enum:

- `BAR_UPDATE`: Real-time bar data updates
- `ERROR`: Error events from IB API
- `CONNECTED`: Connection established events
- `DISCONNECTED`: Connection lost events

## Event Handlers

### Basic Handler

```python
@ib.events.register(IBEventType.BAR_UPDATE)
def handle_bar(ib, bars, has_new_bar):
    if has_new_bar:
        print(bars[-1])
```

### Priority-based Handler

```python
@ib.events.register(IBEventType.BAR_UPDATE, priority=10)
def high_priority_handler(ib, bars, has_new_bar):
    # This handler runs first
    pass
```

### Async Handler

```python
@ib.events.register(IBEventType.BAR_UPDATE)
async def handle_bar(ib, bars, has_new_bar):
    if has_new_bar:
        await process_bar(bars[-1])
```

### Data Source Binding

```python
@ib.events.register(
    IBEventType.BAR_UPDATE,
    bind_to=ib.reqRealTimeBars(contract)
)
def handle_bar(ib, bars, has_new_bar):
    pass
```

## Examples

Check out the [examples](examples/) directory for more:

- [Basic Usage](examples/basic_usage.py): Simple real-time bar data subscription
- [Multiple Streams](examples/multiple_streams.py): Handling multiple data streams with priorities
- [Async Handlers](examples/async_handlers.py): Using async/await with event handlers

## Requirements

- Python 3.7+
- ib_async
- Interactive Brokers TWS or IB Gateway

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.