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

## 问题描述
实现一个猜数字游戏：游戏开始前，从玩家输入的数字范围中随机选取一个数字作为答案；每轮游戏根据玩家的输入缩小数字范围，直到用户猜中答案时游戏结束，统计玩家猜的总次数。

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
```text
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
### 控制台输出
### 从控制台获取用户输入
### 函数声明与调用
### 字符串切分和转换
### 变量所有权
### 分支控制
### 异常处理

## 参考资料
\[1\]. [Programming a Guessing Game, ch02 of The Rust Programming Language.][1]

[1]:https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html
[1]:https://doc.rust-lang.org/std/macro.print.html
[2]:https://doc.rust-lang.org/std/io/index.html#standard-input-and-output
[3]:https://doc.rust-lang.org/std/str/struct.Split.html
[4]:https://doc.rust-lang.org/std/fmt/
