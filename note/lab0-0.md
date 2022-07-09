#### 前言

一般的应用程序在运行时，需要多层次的环境栈支持

比如在 Linux 中，一个 Rust 版本的 HelloWorld：

```d
fn main() {
    println!("Hello, world!");
}
```

程序通过函数调用，使用语言的标准库，而标准库中根据不同的系统进行系统调用，而针对不同的硬件平台，OS所使用的指令集也不尽相同。

在裸机上开发，我们不能依赖 Rust 的标准库，而是需要根据硬件平台，让 Rust 的编译器生成对应的汇编指令。

#### 平台与目标三元组

编译器在编译、连接的时，需要知道其所在的硬件平台是什么，这样才能生成对应的汇编代码。而**目标三元组**就（Target Triplet）描述了目标平台——也就是程序运行的平台——的CPU指令集、OS类型和标准运行时库

```shell
$ rustc --version --verbose
   rustc 1.61.0-nightly (68369a041 2022-02-22)
   binary: rustc
   commit-hash: 68369a041cea809a87e5bd80701da90e0e0a4799
   commit-date: 2022-02-22
   host: x86_64-unknown-linux-gnu
   release: 1.61.0-nightly
   LLVM version: 14.0.0
```

其中 host 一项表明默认目标平台是 `x86_64-unknown-linux-gnu`， CPU 架构是 x86_64，CPU 厂商是 unknown，操作系统是 Linux，运行时库是 GNU libc

这说明我们的程序是为了运行在 Linux 上，同时使用的库是 GNU libc，对应的汇编指令是基于 x86_64 的

但是如果要让 HelloWorld 运行在裸机上，我们就不能使用 GNU libc，同时要根据不同的平台让编译器生成不同的汇编指令

#### 让程序运行在裸机上

First of all,

> cargo new os

Rust 程序在编译时，默认使用了 Rust 语言的标准库 std，但标准库 std 只能在 OS 存在的情况下才能正常的使用。为了让我们的程序运行在裸机上，我们就不能够使用 Rust 语言的标准库。

我们可以通过注解

```rust
#![no_std]
```

来告诉编译器，在编译的时候不使用 Rust 语言的标准库

同时，我们需要指定编译之后的程序运行在什么样的平台上

我们可以通过在当前项目中建立 `.cargo` 文件夹，再在其中建立 `config.toml` 来指定编译出来的程序所运行的平台（在一个平台上编译出另外一个平台上能运行的程序的过程叫做**交叉编译**）

> [交叉编译](https://baike.baidu.com/item/%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91/10916911) ：在一个平台上生成另一个平台上的可执行代码 

```toml
# os/.cargo/config.toml
[build]
target = "riscv64gc-unknown-none-elf"
```

现在我们的程序看起来是这样的：

```rust
#![no_std] // 通过这个注解，告诉 rust 编译器，我们将使用库 `core`，而不是 `std`
fn main() {
    println!("Hello, world!");
}
```

如果你执行 cargo build，你会发现

> error: cannot find macro `println` in this scope

这说明在库 core 中（对于 core，其中的 API 无需依赖任何操作系统集成或堆分配即可支持），并不存在宏 `println`

宏 `println` 是由 std 提供的，其实现是通过对于 write 的系统调用实现的

我们还会看到：

>  error: `#[panic_handler]` function required, but not found

我们知道，在 std 环境下，如果我们的程序 panic 了，程序会退出，并且打印出出错位置等信息。而所谓的 `panic handler` 其实就是一个在 Rust 程序 panic 时会调用的函数。

由于 std 提供了 `panic handler`，所以我们不需要自己来实现，但是在 core 中，我们需要自行实现我们自己的 `panic handler`

```rust
// os/src/lang_items.rs
use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! { // PanicInfo 的字段全是 !pub 的
    loop {}
}
```

> `#[panic_handler]`必须应用于签名为`fn(&PanicInfo) -> !`的函数，并且这样的函数仅能在一个二进制程序/动态链接库的整个依赖图中仅出现一次。 ---- [《Rust 死灵书》](https://nomicon.purewhite.io/panic-handler.html)

panic 的问题解决了，但是还有一个问题

> error: requires `start` lang_item

编译器提醒我们缺少一个名为 `start` 的语义项。 `start` 语义项代表了标准库 std 在执行应用程序之前需要进行的一些初始化工作。由于我们禁用了标准库，编译器也就找不到这项功能的实现了

在 `main.rs` 的开头加入设置 `#![no_main]` 告诉编译器我们没有一般意义上的 `main` 函数， 并将原来的 `main` 函数删除。这样编译器也就不需要考虑初始化工作了。

```rust
// os/src/main.rs
#![no_std]
#![no_main]

mod lang_items;

// os/src/lang_items.rs
use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}
```

#### 构建程序运行环境

在我们的程序运行前，OS替我们做了一些操作（比如清空堆栈等）

而在裸机平台上根本不存在OS来替我们做这样的工作，所以这些过程需要我们自己来手动处理

[构建用户态执行环境](https://learningos.github.io/rust-based-os-comp2022/chapter1/3mini-rt-usrland.html)

[构建裸机执行环境](https://learningos.github.io/rust-based-os-comp2022/chapter1/4mini-rt-baremetal.html)