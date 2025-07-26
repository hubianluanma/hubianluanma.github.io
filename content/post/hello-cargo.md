+++
date = '2025-07-26T21:26:15+08:00'
draft = false
title = 'Rust学习—— Hello Cargo'
categories = ['学习']
tags = ['Rust']

+++

Cargo 是 Rust 的构建系统和包管理器，类似Java中的maven。大多数 Rustaceans 都使用这个工具管理他们的 Rust 项目，因为 Cargo 会为你处理很多任务，例如构建代码、下载代码所依赖的库。

最简单的 Rsut 程序，例如 Hello World 这样的代码，它没有任何依赖项。如果我们使用 Cargo 构建 Hello World 项目，它将仅使用 Cargo 中处理构建代码的部分。当编写的程序变的复杂时，我们将添加依赖项，这时候使用 Cargo 添加依赖项将容易得多。

可以使用下面的命令来查看 Cargo 的版本

```bash
cargo --version
```

如果输出：`command not found` 表明没有正确安装，可以查看 [Rust Install](https://blog.hubianluanma.com/post/rust-install) 文章解决。接下来我们就使用 Cargo 来创建项目。

### 通过 Cargo 创建项目

对比之前我们的 Hello World 代码，来看看使用 Cargo 创建项目有哪些不同，接下来我们进入到我们的 rust 代码空间或者一个目录下（非必须，但我建议创建一个用于存放rust项目的文件夹，方便管理）执行下面的代码

```bash
cargo new hello_cargo
cd hello_cargo
```

`cargo new hello_cargo` 命令会创建一个名为 _hello_cargo_ 的文件夹，并且我们将项目命名为了_hello_cargo_，Cargo 在同名目录中创建相关文件。

然后我们进入到对应文件夹后会看到生成了一个文件：`Cargo.toml` 和一个名为 `src` 的文件夹，其中包含了一个名为 `main.rs` 的文件，里面自动生成了代码：

```rust
fn main() {
    println!("Hello, world!");
}
```

同时它还初始化了一个新的 Git 存储库以及一个 `.gitignore` 文件。

**Cargo.toml**

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

上面是在执行 `cargo new` 时自动生成的 `toml` 文件，文件中的第一行 `[package]` 就是一个 section 的标题，它下面的配置指示的就是当前包的一些相关信息，当我们向包中添加更多信息时，我们将添加其他部分。

接下来的三行设置了 Cargo 编译程序所需的配置信息：名称、版本等信息，最后一行`[dependencies]` 下面是要列出项目等依赖项。

截止到现在，使用 cargo 创建项目和之前普通的 hello world 项目的区别是：

- 会自动创建 Cargo.toml 文件
- main.rs 等代码文件会放在 src 目录下

如果我们想将类似使用普通 hello world 创建的项目改为 cargo 项目的话，只需要相关代码移动到 src 目录下，然后创建适当的 Cargo.toml 文件。获取 Cargo.toml 文件的一个简单方法是运行 `cargo init` 运行它会自动创建 Cargo.toml

### 构建和运行 Cargo 项目

现在我们使用 Cargo 构建和运行 Hello World 程序，在 _hello_cargo_ 文件夹下输入一下命令构建项目：

```bash
cargo build
```

执行完上面的命令后会在目录下生成一个 target 目录，里面的 debug 目录里包含了可运行的同名文件 `hello_cargo` 和一些其他文件。之所以在 debug 目录下是因为默认的构建方式时调试构建，此时我们可以直接运行该文件

```bash
./target/debug/hello_cargo
```

当我们首次运行 `cargo build` 命令后，目录中会生成一个 `Cargo.lock` 文件，它的作用时跟踪项目中依赖的确切版本。

上面我们时在构建完后使用 target下的debug中的可执行文件运行的，但我们可以使用 `cargo run` 编译代码，然后在命令中运行生成的可执行文件。

运行 `cargo run` 时会默认执行 `cargo build` ，当我们的代码没有改动的情况下重新运行 `cargo run` ，此时项目不会重新编译，只有代码发生了变化才会重新出发编译。

当我们不想生成可执行文件，只是想检查一下我们的代码，确保编译不会出错，此时我们可以使用 `cargo check` 命令，它不会生成 可执行文件，执行完后我们可以查看 target/debug 下没有生成对应可执行的文件。

### 构建发布版本

当我们准备好发布项目时，我们可以使用 `cargo build --release` 对其进行优化。此命令将在_target/release_而不是_target/debug_中创建可执行文件。这些优化可以使Rust代码运行的更快，但它会延长程序的编译时间。

### 回顾

- 我们可以只用 `cargo new` 来创建项目
- 使用 `cargo build` 来构建项目，并且可执行文件被生成到了 _target/debug_ 中
- 我们可以使用 `cargo run` 一次性构建和运行程序
- 我们可以使用 `cargo check` 在不生成可执行文件的情况下检查编译项目
- `cargo build` 适用于开发，因为它相对于 `cargo build --release` 更快，而后者适合发布项目，因为它会对 Rust 代码进行优化，使其运行的更快。
