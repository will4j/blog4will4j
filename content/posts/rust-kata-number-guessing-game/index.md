---
slug: "rust-kata-number-guessing-game"
title: "Rust 卡塔：猜数字游戏"
date: 2023-11-23T07:23:22+08:00
author:
  name: 水王
tags:
  - draft
categories:
  - rust-kata
draft: true
comment: true
description:
keywords:
license:
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
---

> **Note:** Rust 卡塔系列旨在通过具体场景的编程练习学习 Rust 编程语言，结尾是相关的 Rust 知识点概要总结，附上参考资料以作扩展阅读。

## 问题描述
实现一个猜数字游戏：游戏开始前，从玩家输入的数字范围（如1到100）中随机选取一个数字作为答案；每轮游戏根据玩家的输入缩小数字范围，直到玩家猜中答案时游戏结束，统计玩家猜的总次数。

> **Note:** Rust 官网电子书《Rust 编程语言》第二章[[1]]也以猜数字游戏作为示例，这个卡塔较之会稍微复杂一些，但用意都在于通过具体场景演示 Rust 基本语法。

## 代码实现
```rust
use std::cmp::Ordering;
use std::io::{self, Write};

use rand::Rng;

fn main() -> io::Result<()> {
    println!("Let's Play a Number Guessing Game!");

    // 根据用户输入创建新游戏，返回最小值、最大值和随机数字答案
    let (mut min, mut max, secret_number) = new_game();

    // 记录猜的次数
    let mut count: u32 = 0;
    loop {
        // 用户输入数字
        let prompt = format!("Guess a Number between {} and {}", min, max);
        let guess = input_int(&prompt);

        print!("You guess {guess}, ");
        count += 1;
        // 比对结果，猜对则游戏结束，否则调整数字范围
        match guess.cmp(&secret_number) {
            Ordering::Less => {
                println!("Too small!");
                min = guess
            }
            Ordering::Greater => {
                println!("Too big!");
                max = guess
            }
            Ordering::Equal => {
                println!("You win with {count} guesses!");
                break;
            }
        }
    }
    Ok(())
}

fn new_game() -> (u32, u32, u32) {
    let mut input_str = String::new();
    input(&mut input_str, "New Game");

    let range: Vec<u32> = input_str
        .split_whitespace()
        .map(|s| s.parse().expect("parse error"))
        .collect();

    let min = range[0];
    let max = range[1];
    let secret_number = rand::thread_rng().gen_range(min..=max);

    (min, max, secret_number)
}

fn input_int(prompt: &str) -> u32 {
    let num;
    loop {
        let mut input_str = String::new();
        input(&mut input_str, prompt);

        match input_str.trim().parse() {
            Ok(_num) => {
                num = _num;
                break;
            }
            Err(_) => {
                println!("please input a integer number.");
                continue;
            }
        }
    }
    num
}

fn input(input_str: &mut String, prompt: &str) {
    print!("{prompt}: ");
    io::stdout().flush().unwrap();

    io::stdin().read_line(input_str).expect("failed to read line");
}
```

## 代码执行
```shell
$ cargo run --bin number-guessing-game
    Finished dev [unoptimized + debuginfo] target(s) in 0.04s
     Running `target/debug/number-guessing-game`
Let's Play a Number Guessing Game!
New Game: 1 20
Guess a Number between 1 and 20: 10
You guess 10, Too big!
Guess a Number between 1 and 10: 5
You guess 5, Too small!
Guess a Number between 5 and 10: 9
You guess 9, You win with 3 guesses!
```

## Rust 知识点
### Cargo
Cargo [[2]]是 Rust 项目的编译构建和依赖管理工具，对应配置文件 Cargo.toml [[3]]，可通过 Cargo 命令创建项目：
```shell
$ cargo new rust-kata
     Created binary (application) `rust-kata` package
$ tree rust-kata
rust-kata
├── Cargo.toml
└── src
    └── main.rs
$ cat rust-kata/Cargo.toml
[package]
name = "rust-kata"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```
默认情况下，Rust 项目只能有一个 `main` 函数作为执行入口（如 `src/main.rs`），通过 Cargo bin 可额外设置。项目依赖声明在 dependencies 配置下，可通过 crates.io [[4]]搜索三方依赖。rust-kata 配置猜数字游戏入口，添加随机数库依赖后配置如下：
```shell
$ cat rust-kata/Cargo.toml
[package]
name = "rust-kata"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
rand = "0.8.5"

[[bin]]
name = "number-guessing-game"
path = "src/bin/number_guesssing_game.rs"
```
程序执行方式：
```shell
# 执行项目主程序 src/main.rs
$ cargo run --bin rust-kata
# 执行猜数字游戏程序
$ cargo run --bin number-guessing-game
```

### 控制台输出
Rust 标准库[[5]]中包含控制台输出函数 `print` 和 `println`，两者均支持 Rust 字符串格式化语法[[6]]。

基于性能考虑，`print` 输出会先放到行缓冲区，不会立即打印到控制台，可通过 `io::stdout().flush()` 手动触发打印：

```rust
// 立即输出到控制台，结束后另起一行
println!("Let's Play a Number Guessing Game!");
// 不换行，不会立即输出到控制台，需要手动 flush，或等待下一次 println，或者等待程序运行结束
print!("Guess a Number between {min} and {max}: ");
io::stdout().flush().unwrap();
```

### 变量可变性
Rust 通过 `let` 声明变量，变量默认不可变，支持修改需要通过 `mut` 关键字声明。

```rust
// x 变量不支持修改
let x = 5;
x = 6;
^^^^^ cannot assign twice to immutable variable
```

```rust
// mut x 变量支持修改
let mut x = 5;
x = 6;
```

### 

### 从控制台获取用户输入
```rust
let mut input_str = String::new();
io::stdin().read_line(&mut input_str).expect("failed to read line");
```

### 函数声明与调用
### 字符串切分和转换
### 分支控制
### 异常处理

## 参考资料
\[1\]. [Programming a Guessing Game. ch02,《Rust 编程语言》][1]  
\[2\]. [Hello Cargo. ch03,《Rust 编程语言》][2]  
\[3\]. [The Manifest Format.《Cargo 手册》][3]  
\[4\]. [Rust 社区 crate 仓库][4]  
\[5\]. [Rust 标准库，宏目录][5]  
\[6\]. [Rust 标准模块：format!][6]  
\[7\]. [Rust 标准模块：format!][7]  

[1]:https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html
[2]:https://doc.rust-lang.org/book/ch01-03-hello-cargo.html
[3]:https://doc.rust-lang.org/cargo/reference/manifest.html
[4]:https://crates.io/
[5]:https://doc.rust-lang.org/std/index.html#macros
[6]:https://doc.rust-lang.org/std/fmt/index.html
[7]:https://doc.rust-lang.org/std/io/index.html#standard-input-and-output
[3]:https://doc.rust-lang.org/std/str/struct.Split.html
[4]:https://doc.rust-lang.org/std/fmt/
[5]:https://docs.rs/rand/0.8.5/rand/trait.Rng.html#method.gen_range
