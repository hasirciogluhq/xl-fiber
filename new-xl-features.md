# New xl / xlc features (xlc 1.0.0-rc.11, std 1.4.1)

Verified in this project. Short notes only.

## Syntax
- `local x := expr`: type inferred.
- `local (a, b) = (x, y)`: tuple destructuring (declaration only; `(a, b) = …` reassignment is not allowed).
- `for i in 0..n`, `0..=n`; `for x, i in arr` (x is a pointer into the storage); `for x in [1, 2, 3]`.
- `i++`, `+=`: statements only (`a[i++]` does not parse).
- `b'x'`: byte literal (u8).
- No `;`: statements end at the line break.
- `from` is a keyword: don't use it as a parameter name.
- `cond ? a : b`, `do { } while c`, labeled loops, `defer`, `try { } catch e { }`.

## Types
- `class` with `fn init(self, …)` (constructor, `Name(args)`) and `fn drop(self)` (RAII, move-only).
- `class D : B`, `virtual` / `override`, `interface` + `impl I for T`.
- `impl<T> Type<T> { fn m<F>(self: *Type<T>, f: F) }`: methods added outside the class, generic methods included.
- `enum E { A(i64), B }` payload enums; `switch v { case E.A(x) { } }`.
- `Option<T>` / `Result<T, E>` in `std/collections`; `expr?` propagates `None` / `Err`.
- `new T(args)`, `new(ptr) T(args)`, `delete p`.
- `const`, `define NAME = v`, `define m(x: expr) { }` macros, `staticAssert(cond, "msg")`.

## Closures
- `|x: T| expr`, `|| { … }`: capture **by reference**. Inside a loop every closure shares the loop variable.
- `move || { … }`: capture by value. Pass shared state as pointers: `local (rt, out) = (&ctx, &buf[0])`.
- Taken as a generic `F` and called as `f()`; moved with `__builtin_write(p as *F, f)` / `__builtin_read(p as *F)`.
- `local g: fn(*void) = generic<F>`: function pointer to a generic instance.
- A closure that captures another generic closure and calls it fails (`__closureN has no method f`).
- A closure made inside another closure captures the outer closure's environment. If it outlives the outer one
  (posted, spawned), it dangles: use `local p := &x` then `move || { *p … }`.
- **Bug:** a generic function that takes a closure fails (`__closureN has no field x`) when it is reached through a
  module named `index`; importing the defining module directly works. That's why the tests import `../runtime` etc.

## Compile-time and low level
- `#if os == "macos"` / `arch == "aarch64"`, `#elif`, `#else`, `#error "msg"`.
- `__builtin_asm("tmpl", in("x0") a, out(reg) b, inout("x1") c, clobber("x9"), clobber("v8"), clobber("memory"), options("nomem"))`.
  The x86 template is Intel syntax. No string concatenation in the template: use one literal with `\n`.
- `__builtin_atomicLoad/Store/Cas/Add/Xchg(p, …, MemoryOrder.acquire)`.
- `sizeOf(T)`, `alignOf(T)`, `syscall(nr, …)` (returns -errno).
- **std bugs:** `sysPipe(fds, true)` leaves macOS pipes blocking, and `sysPipe` returns -1 under Windows/Wine (a direct
  `CreatePipe` works). Use xl-fiber's `openPipe(fds)`.
- Windows `sysRead` / `sysWrite` map every error to -1 (no errno). The std poller (WSAPoll) watches sockets only and returns
  at once on an empty set.
- `@library("kernel32") declare fn …` works in packages too, not just std.
- Win64 asm: `qword ptr gs:[8]` reaches the TIB.

## Tooling
- `xl check --target x86_64-linux|x86_64-apple-darwin|x86_64-pc-windows-msvc file.xl`: cross type-check.
- Windows builds need the full triple `x86_64-pc-windows-msvc`. Short `x86_64-windows` type-checks but fails to link.
- `xl build --target x86_64-apple-darwin -O2 -o out file.xl`: x86_64 binaries run on Apple silicon via Rosetta.
- `xl test [pattern] [-O2]`: runs `test/*.test.xl` in parallel.
