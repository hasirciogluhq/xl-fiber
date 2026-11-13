# xl-fiber

An asynchronous runtime for [xlang](https://github.com/hasirciogluhq/xlang): fibers, scheduling, timers and async I/O.
It plays the role tokio plays for Rust or boost::asio for C++.

> xl-fiber is an independent third-party library, not affiliated with the official xlang project.

## Features

- **Context switching** in hand-written asm for aarch64, x86_64 System V and Win64.
- **FiberContext**, an `io_context`-style scheduler: `run`, `runOne`, `runFor`, `runUntil`, `poll`, `pollOne`, `stop`, `restart`.
- **Tasks**: `spawn`, `post` and `dispatch`, with `Task.join()` and `Task.done()`.
- **Timers**: `sleepFor` and `sleepUntil`, backed by a min-heap.
- **Async I/O**: `read`, `write`, `waitReadable` and `waitWritable` over kqueue, epoll or WSAPoll.
- **Channels**: a bounded `Channel<T>` with `send`, `recv` and `close`.
- **Pooled stacks** with guard pages, so finished fibers are recycled.

Tested on macOS (arm64, x86_64), Linux (aarch64, x86_64) and Windows x64.

## Install

```sh
xl pkg install github.com/hasirciogluhq/xl-fiber
```

## Example

```xl
from "github.com/hasirciogluhq/xl-fiber/runtime" import FiberContext
from "github.com/hasirciogluhq/xl-fiber/channel" import Channel
from "std/collections/option" import Option

@entry fn main() -> i32 {
    local ctx := FiberContext()
    local ch := Channel<i64>(&ctx, 4)
    local total: i64 = 0
    ctx.spawn(|| {
        for i in 1..=100 { ch.send(i) }
        ch.close()
    })
    ctx.spawn(|| {
        while true {
            switch ch.recv() {
                case Option.Some(v) { total += v }
                case Option.None { break }
            }
        }
    })
    ctx.run()
    return 0
}
```

Import modules such as `runtime` and `channel` directly. Because of a compiler bug, generic functions that take
closures fail when they are reached through `index`.

## Development

```sh
xl test                  # run test/*.test.xl
xl run examples/context.xl
xl check index.xl
```

## Known limitations

- A FiberContext is single-threaded. Cross-thread `post` is not supported yet.
- I/O operations cannot be cancelled and have no timeouts yet.
- On Windows, pipe I/O backs off by 1 ms instead of using IOCP.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Open pull requests against `main`.

## License

[Apache License 2.0](LICENSE)
