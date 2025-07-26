+++
date = '2025-07-26T21:02:38+08:00'
draft = false
title = 'Rust学习—— Hell World'
categories = ['学习']
tags = ['Rust']
+++

执行下面的这段命令安装：

```bash
curl --proto '=https' --tlsv1.2 https://sh.restup.rs -sSf | sh
```

执行上面这段命令会安装下面的组件：

- cargo
- clippy
- rust-docs
- rust-std
- rustc

如果是 MacOS 的话没有C编译器的情况下可以执行下面的命令安装：

```bash
xcode-select --install
```

验证是否安装成功：

```bash
rustc --version
```
如果安装成功会显示下面的内容：

```
rustc 1.88.0 (6b00bc388 2025-06-23)
```

输出的内容是最新的版本号，括号里面对应的是提交Hash值和日期。如果想要更新 Rust 的话执行下面的命令

```bash
rustup update
```

如果要卸载的话可以执行

```bash
rustup self uninstall
```

上面安装过程中已经安装了离线文档，在开发过程中我们可以快速的查看对应的文档，执行下面的命令

```bash
rustup doc
```

### Hello World

下面我们来编写学习每一门语言必写的 Hello World, 创建 main.rs 文件

```rust
fn main() {
    println!("Hello World!");
}
```

然后执行下面的命令进行编译

```bash
rustc main.rs
```
执行完上面的命令会生成一个可执行的二进制文件——main，然后我们运行它
```bash
./main
```
最终输出：**Hello World!**


