---
slug: "rust-kata-number-guessing-game"
title: "Rust 卡塔：猜数字游戏"
date: 2023-11-28T07:23:22+08:00
author:
  name: 水王
tags:
  - Rust
categories:
  - rust-kata
draft: false
comment: true
description: Rust 编程练习：实现猜数字游戏。
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

## 测试先行
优雅的代码肯定也是易于测试的，反之，开始编码之前先思考测试场景是从功能层面对问题进行分解，有助于写出职责清晰，松耦合的代码。Rust 测试编写可参考文档[[2]]。

抽取猜数字游戏的两个主要功能：游戏创建（`new_game`）以及数字猜测（`do_guess`），单元测试分为基本功能用例和异常场景用例两组，可以先实现基本功能用例再实现异常场景用例：

```rust
#[cfg(test)]
mod tests {
    use crate::{do_guess, new_game};

    // start of basic use cases
    #[test]
    fn new_game_ok() {
        let (min, max, secret_number) = new_game("1 100");
        assert_eq!(min, 1);
        assert_eq!(max, 100);
        assert!(secret_number >= 1 && secret_number <= 100);
    }

    #[test]
    fn do_guess_bingo() {
        let guess_result = do_guess(1, 100, 50, "50");
        assert_eq!(guess_result, Ok((0, 50, 50)));
    }
    // end of basic use cases

    // start of exception use cases
    #[test]
    fn do_guess_wrong() {
        let guess_result = do_guess(1, 100, 37, "50");
        assert_eq!(guess_result, Ok((1, 1, 49)));

        let guess_result = do_guess(1, 49, 37, "25");
        assert_eq!(guess_result, Ok((-1, 26, 49)));
    }

    #[test]
    fn do_guess_failed() {
        let guess_result = do_guess(1, 100, 37, "abc");
        assert_eq!(guess_result.unwrap_err(), "Please input a integer number");

        let guess_result = do_guess(25, 50, 37, "51");
        assert_eq!(guess_result.unwrap_err(), "Number should between 25 and 50");
    }
    // end of exception use cases
}

/// 创建新的猜数字游戏，返回游戏范围以及范围内的随机数字。
///
/// # Arguments
///
/// * `game_str` - 游戏创建字符串，空格分割的数字起始和结束范围。
///
/// # Examples
/// ```
/// let (min, max, secret) = new_game("1 100")
/// ```
fn new_game(game_str: &str) -> (u32, u32, u32) {
    (1, 100, 50)
}

/// 进行一轮猜数字游戏，返回猜测结果以及根据结果调整过后的数字范围。
///
/// 功能包含对输入进行校验。
///
/// # Arguments
///
/// * `min` - 数字范围：最小数字
/// * `max` - 数字范围：最大数字
/// * `secret_number` - 猜数字游戏答案
/// * `guess_str` - 本轮游戏输入
fn do_guess(min: u32, max: u32, secret_number: u32, guess_str: &str) -> Result<(i8, u32, u32), String> {
    Ok((-1, 25, 49))
}
```

## 代码实现
```rust
use std::cmp::Ordering;
use std::io::{self, Write};

use rand::Rng;

fn main() -> io::Result<()> {
    println!("Let's Play a Number Guessing Game!");

    print!("New Game: ");
    io::stdout().flush().unwrap();

    let mut input_str = String::new();
    io::stdin().read_line(&mut input_str).unwrap();

    // 根据用户输入创建新游戏，返回最小值、最大值和随机数字答案
    let (mut min, mut max, secret_number) = new_game(&input_str);

    // 记录猜的次数
    let mut count: u32 = 0;
    loop {
        print!("Guess a Number between {min} and {max}: ");
        io::stdout().flush().unwrap();

        let mut input_str = String::new();
        io::stdin().read_line(&mut input_str).unwrap();

        let guess_result = do_guess(min, max, secret_number, &input_str);
        match guess_result {
            Ok((result, new_min, new_max)) => {
                print!("You guess {}, ", input_str.trim());
                count += 1;

                min = new_min;
                max = new_max;

                if result == 0 {
                    println!("You win with {count} guesses!");
                    break;
                } else if result == -1 {
                    println!("Too small!");
                } else {
                    println!("Too big!");
                }
            }
            Err(err) => {
                println!("{err}");
                continue;
            }
        };
    }
    Ok(())
}

fn new_game(game_str: &str) -> (u32, u32, u32) {
    let range: Vec<u32> = game_str
        .split_whitespace()
        .map(|s| s.parse().expect("parse error"))
        .collect();

    let min = range[0];
    let max = range[1];
    let secret_number = rand::thread_rng().gen_range(min..=max);

    (min, max, secret_number)
}

fn do_guess(min: u32, max: u32, secret_number: u32, guess_str: &str) -> Result<(i8, u32, u32), String> {
    match guess_str.trim().parse::<u32>() {
        Ok(_num) => {
            if _num < min || _num > max {
                return Err(format!("Number should between {min} and {max}"));
            }
            // 比对结果，调整数字范围
            match _num.cmp(&secret_number) {
                Ordering::Less => Ok((-1, _num + 1, max)),
                Ordering::Greater => Ok((1, min, _num - 1)),
                Ordering::Equal => Ok((0, _num, _num)),
            }
        }
        Err(_) => return Err(format!("Please input a integer number"))
    }
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
Guess a Number between 1 and 9: 5
You guess 5, Too small!
Guess a Number between 6 and 9: 8
You guess 8, You win with 3 guesses!
```

## Rust 知识点
### Cargo
Cargo [[3]]是 Rust 项目的编译构建和依赖管理工具，对应配置文件 Cargo.toml [[4]]，可通过 Cargo 命令创建项目：
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
默认情况下，Rust 项目只能有一个 `main` 函数作为执行入口（如 `src/main.rs`），通过 Cargo bin 可额外设置。项目依赖声明在 dependencies 配置下，可通过 crates.io [[5]]搜索三方依赖。

rust-kata 配置猜数字游戏入口，添加随机数库依赖后配置如下：
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
Rust 标准库[[6]]中包含控制台输出函数 `print` 和 `println`，两者均支持 Rust 字符串格式化语法[[7]]。

基于性能考虑，`print` 输出会先放到行缓冲区，不会立即打印到控制台，可通过 `io::stdout().flush()` 手动触发打印：

```rust
// 立即输出到控制台，结束后另起一行
println!("Let's Play a Number Guessing Game!");
// 不换行，不会立即输出到控制台，需要手动 flush，或等待下一次 println，或者等待程序运行结束
print!("Guess a Number between {min} and {max}: ");
io::stdout().flush().unwrap();
```

### 变量可变性
Rust 通过 `let` 关键字声明变量，默认不可修改，支持变量修改需要通过 `mut` 关键字声明。

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

### 函数声明与调用
Rust 通过 `fn` 关键字声明函数[[8]]，支持多返回值，可省略 `return` 语句，以最后一行的变量或者表达式作为函数返回值：
```rust
fn upper_and_lower(str: &str) -> (String, String) {
    let upper = src.to_uppercase();
    let lower = src.to_lowercase();
    (upper, lower)
}

fn main() {
    let (upper, lower) = upper_and_lower("Rust");
    println!("{upper} {lower}");
    // RUST rust
}
```

### 变量所有权及其租借
Rust 不使用垃圾收集器，而是引入所有权机制[[9]]来进行内存管理。简单来说，就是保证始终只有一个变量对内存区域具有所有权，所有权变量失效时对应内存即被释放。

#### 所有权
所有权机制主要用于堆内存管理。

对于基础类型变量（如 u32），其长度固定，默认在栈上分配，变量跟随出栈操作释放。栈上变量间传递以复制（Copy）方式进行：
```rust
let x: u32 = 64;
let y = x;
println!("x={x} y={y}");
// 输出 x=64 y=64
```
以上代码中，`let y = x;` 语句实际上复制 `x` 创建了一个新的变量 `y`，`x` 和 `y` 都存储在栈上。

对于复杂类型变量（如字符串、数组），其长度不固定，需要在堆上进行分配。要保证同时只有一个变量对堆内存区域拥有所有权，不可避免会发生所有权变更，rust 对所有权变更的处理策略在其他编程语言的开发者看来可能会比较违反直觉：
```rust
let x = String::from("Rusty");
let y = x;
        - value moved here
println!("x={x} y={y}");
            ^^ value borrowed here after move
```
以上代码中，`let y = x;` 语句执行后，变量 `x` 的所有权移交（Move）给了变量 `y`，区别于复制（Copy）的方式，**所有权移交后变量即视为失效**，除非重新声明使用，后续所有对于变量 `x` 的访问在编译时就会报错。

通过观察变量指针的内存地址可以更好的理解所有权移交：
```rust
let mut x = String::from("Rusty");
println!("x={:p} *x={:p}", &x, &*x);
// 输出 x=0x16f0aa940 *x=0x600001e4c040
let y = x;
println!("y={:p} *y={:p}", &y, &*y);
// 输出 y=0x16f0aa9c0 *y=0x600001e4c040
x = String::from("Rusty");
println!("x={:p} *x={:p}", &x, &*x);
// 输出 x=0x16f0aa940 *x=0x600001e4c050
```
以上代码中，`&x` 表示变量在栈上的地址，`&*x` 表示变量指向的堆内存地址。可以看到，所有权变更后，变量 `y` 指向的堆内存地址 `&*y` 与之前的 `&*x` 相同，说明变量 `y` 确实接管了变量 `x` 对堆内存的所有权。对变量 `x` 重新分配后，其栈上内存地址不变，即栈上变量 `&x` 被复用，但是指向的堆内存地址已经发生了变更。

#### 所有权租借
如果使用所有权变更机制进行函数传参，函数调用后实参变量即失效，将导致后续对该变量的访问报错，这种情况就需要用到所有权租借机制，租借需要依赖变量引用（使用 `&` 操作符）：
```rust
fn main() {
    let x = String::from("Rusty");
    let length = len(&x);
    println!("{x} length={length}");
    // 输出 Rusty length=5
}

fn len(str: &String) -> usize {
    str.len()
}
```
以上代码 `len` 函数调用传递的是变量引用 `&x`，没有发生所有权变更，因此在后续 `println` 语句中仍然可以访问变量 `x`。引用也可以对变量进行修改，但是需要通过 `&mut` 显式声明。

### 从控制台获取用户输入
获取控制台输入[[10]]使用可变引用传递的方式，将用户输入保存到字符串变量：
```rust
let mut input_str = String::new();
io::stdin().read_line(&mut input_str).expect("failed to read line");
```

### 字符串切分和转换
Rust 提供多种字符串 `split` 方式[[11]]，`split` 返回的是一个迭代器对象，可根据需要再次进行映射或过滤处理，猜数字游戏实现使用的是按空白字符切分（`split_whitespace`）:
```rust
let x = String::from("a b c");
let y = x.split_whitespace();
println!("{:?}", y.collect::<Vec<_>>());
// 输出 ["a", "b", "c"]
```

使用 `parse` 方法[[12]]可对字符串进行类型转换：
```rust
let four: u32 = "4".parse().unwrap();
assert_eq!(4, four);
```

### Match 流程控制
Rust 通过 Match [[13]]达到类似 `swtich` 的效果，但是 Match 功能更加强大。可以处理表达式，也可以处理函数返回值：
```rust
let x = String::from("123");
let result = match x.parse::<u32>() {
    Ok(num) => num,
    Err(_) => panic!("can't parse to integer"),
};
```
match 要求分支完备，上面的代码如果没有异常 `Err(_)` 处理分支，编译时会报错。

### 异常处理
Rust 将异常分为不可恢复异常（`panic`）和可恢复异常，后者需要主动处理或者向上传递。通常以 `Result` [[14]]作为结果包装容器：
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
除了用 `match` 显式处理异常外，`Result` 有两种用 `panic` 处理异常的快捷方式，`unwrap` 和 `expect`：
```rust
let x = String::from("123");
// unwrap 在异常时直接 panic，使用默认错误信息
let result = x.parse::<u32>().unwrap();
// expect 也在异常时直接 panic，但使用自定义错误信息
let result = x.parse::<u32>().expect("can't parse to integer");
```
一般在生产环境推荐使用 `expect` 以提供更准确的上下文信息。

## 参考资料
\[1\]. [Programming a Guessing Game. ch02,《Rust 编程语言》][1]  
\[2\]. [How to Write Tests. ch11.1,《Rust 编程语言》][2]  
\[3\]. [Hello Cargo. ch01.3,《Rust 编程语言》][3]  
\[4\]. [The Manifest Format.《Cargo 手册》][4]  
\[5\]. [Rust 社区 crate 仓库][5]  
\[6\]. [Rust 标准库，宏目录][6]  
\[7\]. [Rust 标准模块：format!][7]  
\[8\]. [Functions ch03.3,《Rust 编程语言》][8]  
\[9\]. [What Is Ownership? ch04.1,《Rust 编程语言》][9]  
\[10\]. [Rust 标准库：标准输入输出][10]  
\[11\]. [Rust 标准模块：str split_whitespace 方法][11]  
\[12\]. [Rust 标准模块：str parse 方法][12]  
\[13\]. [Match 流程控制. ch06.2,《Rust 编程语言》][13]  
\[14\]. [可恢复异常 Result. ch09.2,《Rust 编程语言》][14]  

[1]:https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html
[2]:https://doc.rust-lang.org/book/ch11-01-writing-tests.html
[3]:https://doc.rust-lang.org/book/ch01-03-hello-cargo.html
[4]:https://doc.rust-lang.org/cargo/reference/manifest.html
[5]:https://crates.io/
[6]:https://doc.rust-lang.org/std/index.html#macros
[7]:https://doc.rust-lang.org/std/fmt/index.html
[8]:https://doc.rust-lang.org/book/ch03-03-how-functions-work.html
[9]:https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
[10]:https://doc.rust-lang.org/std/io/index.html#standard-input-and-output
[11]:https://doc.rust-lang.org/stable/std/primitive.str.html#method.split_whitespace
[12]:https://doc.rust-lang.org/stable/std/primitive.str.html#method.parse
[13]:https://doc.rust-lang.org/book/ch06-02-match.html
[14]:https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
