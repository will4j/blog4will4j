---
slug: "rust-kata-number-guessing-game"
title: "Rust 卡塔：猜数字游戏"
date: 2023-11-23T07:23:22+08:00
author:
  name:
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

## 问题定义

## 代码实现
```rust
use std::io::{self, Write};

fn main() -> io::Result<()> {
    println!("Guessing a random number from range!");
    print!("New game: ");
    io::stdout().flush()?;

    let mut range_str = String::new();
    io::stdin().read_line(&mut range_str)?;

    let range: Vec<u32> = range_str
        .split_whitespace()
        .map(|s| s.parse().expect("parse error"))
        .collect();
    println!("Guess a Number range {:?}:", range);
    Ok(())
}
```

## 语法解释

## 参考资料

[1]:https://doc.rust-lang.org/std/macro.print.html
[2]:https://doc.rust-lang.org/std/io/index.html#standard-input-and-output
[3]:https://doc.rust-lang.org/std/str/struct.Split.html
[4]:https://doc.rust-lang.org/std/fmt/
