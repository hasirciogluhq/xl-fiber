# xl-fiber

An asynchronous runtime library for **xlang**: fibers, scheduling, timers and async I/O. Think of it as
what **tokio** is to Rust, or **boost::asio** is to C++.

## Project rules

1. **Independent project.** xl-fiber is not affiliated with the official xlang project. It is a third-party library.
2. **Act like tokio.** Design the API, structure and conventions the way a mature async runtime crate would
   (tokio / boost::asio): runtime, executor, tasks, channels, timers, I/O reactor.
3. **Library layout.** The package is defined by `xlang.toml` with `index.xl` as the public entry point.
   Keep it a library — runnable code goes in `examples/`, tests go in `test/*.test.xl`.
4. **Colored output uses xl-color.** Use `github.com/hasirciogluhq/xl-color` for all colored terminal output; do not
   hand-write ANSI escape codes. Add dependencies with `xl pkg install github.com/<user>/<repo>` (or `xl pkg add`).
   Package manifests must pin exact release tags, never `latest`.
5. **Edit files with the built-in tools.** Use Read / Edit / Write for every file change, however small. Never edit
   files through Bash, `sed`, `awk`, heredocs or Python scripts.
6. **Modern, compact xlang.** Use classes (`init` / `drop`), closures, `:=`, `for … in`, `b'x'`, `Option` / `?` and
   std types instead of hand-rolled equivalents. New language features worth knowing go in `new-xl-features.md`.

## The xlang toolchain (`../xlang`)

`../xlang` is the main xlang repository (the `xl` binary plus submodules: `xlc`, `xl-std`, `xl-llvm`, `xlang-docs`, …).
Read only what you actually need from it.

- **Do not read the compiler (`xlc`)** unless you need it to understand a compiler error you cannot otherwise resolve.
- **Do not read or modify `xlang-docs`** (or any other part of the xlang repo) for this project.
- For standard-library APIs, look at `../xlang/xl-std` only as needed.

## Commands

```sh
xl run examples/hello.xl   # run an example
xl test                    # build and run test/*.test.xl
xl check index.xl          # type-check the library
xl pkg install             # fetch dependencies listed in xlang.toml
```

Linux: build static test binaries with `xl build --target aarch64-linux|x86_64-linux -o out test/x.test.xl` and run them in
the `default` lima VM (`limactl start default`, `limactl copy`, `limactl shell default …`). x86_64 runs there under
`qemu-x86_64`.

Windows: `xl build --target x86_64-pc-windows-msvc -o out.exe test/x.test.xl`, then `cxrun out.exe` (CrossOver, `cs2`
bottle). `timeout` does not stop Wine: background the run, kill it after a deadline, then `pkill -f out.exe`.
