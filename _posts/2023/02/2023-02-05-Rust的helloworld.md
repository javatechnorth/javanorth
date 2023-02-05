---
layout: post
title:  如何编写Rust的helloword
tagline: by IT可乐
categories: Rust
tags: 
    - IT可乐
---

哈喽，大家好，我是指北君。  
今天我们接着来学习Rust，本篇文章给大家介绍如何编写Rust的HelloWord。
<!--more-->

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-01-01.png)

安装完成 Rust 之后，我们可以编写 Rust 的 Hello Word。这里介绍两种方式，一种是rust原生方式，一种是利用 cargo 工具（重要）

### 1、rustc 方式

#### 1.1 创建项目目录

rust 运行不关心代码存放的目录，我们可以任意选择一个合适的位置，创建一个目录。

比如：我们创建一个目录名称为 rust_helloword

> mkdir rust_helloword



#### 1.2 编写rust程序

rust 的源文件后缀是 .rs 。所以我们在第一个创建的项目目录下，创建一个 main.rs 文件。

然后在 main.rs 文件中写入如下代码：

```rust
fn main(){
    println!("Hello World!");
}
```



#### 1.3 编译并运行 rust 程序

在创建的 main.rs 文件目录下，输入如下命令：

①、编译

> rustc main.rs

执行之后，会在当前目录下生成一个 main 的可执行文件。

PS：windows 是生成 main.exe 可执行文件；Linux/Mac 是生成 main 文件。

②、运行

> ./main

运行之后会在窗口打印出 Hello World！

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-03-01.png)

至此，我们完成了第一个 Rust 程序的编写。



### 2、cargo方式

cargo 英文文档：https://doc.rust-lang.org/cargo/

cargo中文文档：https://cargo.budshome.com/index.html

Cargo 是 Rust 的构建系统和包管理器。大多数 Rustacean（这个词是从甲壳纲动物这个单词Crustacean[[krʌ'steʃən]]，去掉了首字母C，而演变而来的，表示 rust 开发者） 使用 Cargo 来管理 Rust 项目，因为它可以为你处理很多任务，比如构建代码、下载依赖库并编译这些库。

在编写更加复杂的 rust 程序时，会用到很多依赖项，如果使用 Cargo 来启动项目，会简单很多。



#### 2.1 检查cargo安装

注意：在安装 rust 时，我们是安装的 rustup，这会自动安装 Cargo，所以我们这里不介绍如何安装 cargo。

> cargo --version

出现如下界面：

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-03-02.png)

分别是 cargo 【版本号】（【哈希码】 【发布时间】）



#### 2.2 创建项目

输入如下命令：

> cargo new hello_cargo

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-03-03.png)

该命令会自动创建一个 hello_cargo 目录，里面包含两个文件和一个目录：一个 *Cargo.toml* 文件，一个 *src* 目录，以及位于 *src* 目录中的 *main.rs* 文件。

#### 2.3 文件介绍

①、Cargo.toml

```
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"

[dependencies]
```

这是 Cargo的配置文件，使用 TOML (*Tom's Obvious, Minimal Language*)语法

第一行，`[package]`，是一个片段（section）标题，表明下面的语句用来配置一个包。随着我们在这个文件增加更多的信息，还将增加其他片段（section）。

接下来的四行设置了 Cargo 编译程序所需的配置：项目的名称、版本、作者以及要使用的 Rust 版本。Cargo 从环境中获取你的名字和 email 信息，所以如果这些信息不正确，请修改并保存此文件。

最后一行，`[dependencies]`，用于书写项目的依赖包（类似Maven、Gradle里面编写的依赖）。在 Rust 中，代码包被称为 *crates*。这个项目并不需要其他的 crate。



②、src/main.rs

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 为我们生成了一个 “Hello, world!” 程序，rust 原生方式与 Cargo 生成项目的区别是 Cargo 将代码放在 *src* 目录，同时项目根目录包含一个 *Cargo.toml* 配置文件。

Cargo 期望源文件存放在 *src* 目录中。项目根目录只存放 README、license 信息、配置文件和其他跟代码无关的文件。使用 Cargo 帮助你保持项目干净整洁，一切井井有条。

如果没有使用 Cargo 开始项目，比如我们创建的 Hello,world! 项目，可以将其转化为一个 Cargo 项目：将代码放入 *src* 目录，并创建一个合适的 *Cargo.toml* 文件。



#### 2.4 构建并运行项目

①、构建项目

> cargo bulid

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-03-04.png)

这个命令会创建一个可执行文件 *target/debug/hello_cargo* （在 Windows 上是 *target\debug\hello_cargo.exe*），而不是放在当前目录下。

②、运行

执行运行上一步生成的可执行文件即可。

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-03-05.png)

如果一切顺利，终端上应该会打印出 `Hello, world!`。首次运行 `cargo build` 时，也会使 Cargo 在项目根目录创建一个新文件：*Cargo.lock*。这个文件记录项目依赖的实际版本。这个项目并没有依赖，所以其内容比较少。

原则上自己永远也不需要碰这个文件，让 Cargo 处理它就行了。

#### 2.5 cargo run

我们刚刚使用 `cargo build` 构建了项目，并使用 `./target/debug/hello_cargo` 运行了程序，也可以使用 `cargo run` 在一个命令中同时编译并运行生成的可执行文件：

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-03-06.png)

注意这一次并没有出现表明 Cargo 正在编译 `hello_cargo` 的输出。Cargo 发现文件并没有被改变，就直接运行了二进制文件。如果修改了源文件的话，Cargo 会在运行之前重新构建项目，并会出现像这样的输出：

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-03-07.png)



#### 2.6 cargo check

cargo check 该命令可以快速检查代码确保其可以进行编译，但是不产生可执行文件。

![](http://www.javanorth.cn/assets/images/2023/itcoke/rust/rust-03-08.png)

为什么你会不需要可执行文件呢？

通常 `cargo check` 要比 `cargo build` 快得多，因为它省略了生成可执行文件的步骤。如果你在编写代码时持续的进行检查，`cargo check` 会加速开发！为此很多 Rustaceans 编写代码时定期运行 `cargo check` 确保它们可以编译。当准备好使用可执行文件时才运行 `cargo build`。

#### 2.7 发布(release)构建

当项目最终准备好发布时，可以使用 `cargo build --release` 来优化编译项目。这会在 *target/release* 而不是 *target/debug* 下生成可执行文件。这些优化可以让 Rust 代码运行的更快，不过启用这些优化也需要消耗更长的编译时间。这也就是为什么会有两种不同的配置：一种是为了开发，你需要经常快速重新构建；

另一种是为用户构建最终程序，它们不会经常重新构建，并且希望程序运行得越快越好。

如果你在测试代码的运行时间，请确保运行 `cargo build --release` 并使用 *target/release* 下的可执行文件进行测试。



### 3、总结

对于简单项目， Cargo 方式相比比 `rustc` 并没有多大的优势，不过随着开发的深入，项目越来越大，那么其优势也会越来越大。对于拥有多个 crate 的复杂项目，交给 Cargo 来协调构建将简单的多。

所以我们在开发过程中要将 Cargo 当做习惯。