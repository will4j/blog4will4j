---
slug: "go-task-the-alternatives-to-gnu-make"
title: "Make 平替：跨平台构建工具 go-task"
date: 2023-12-14T20:16:00+08:00
author:
  name: 水王
tags:
  - Taskfile
categories:
  - tools
draft: false
comment: true
description: 自动化构建工具 go-task 使用介绍。
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

go-task 是用 Go 语言编写的任务执行/构建工具，对比 GNU make，其主要优势在于用户友好、学习成本低的语法规则，以及支持跨平台执行的能力。 使用 Task 编写构建任务，可以极大增强自动化程度，提高开发协作效率。

Task 执行文件为 `Taskfile`，采用 `yaml` 格式，先来看下 cowsay 版本的快速开始 :) ：

```yaml
# Taskfile.yml
version: 3

tasks:
  default:
    desc: "This is the default task"
    silent: true
    cmds:
      - cowsay "Are you ready to go task?"
```

```shell
$ task
 ___________________________
< Are you ready to go task? >
 ---------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

## 工具安装

Task 安装非常方便，参考官方安装指南[[1]]，支持多种包管理工具安装，比如 Mac / Linux 平台的 Homebrew、Tea，Windows 平台的 Chocolatey、Scoop，Node 的 npm，也可以使用安装包、安装脚本或者直接从源码编译安装，任君挑选。以 brew 为例：

```shell
$ brew install go-task
$ task --version
Task version: 3.32.0
```

## 基本使用
### 变量和环境变量
### 工作目录
### 任务依赖
### 命令行参数
### Dry run 调试

## 进阶使用
### 动态变量
### Taskfile 导入
### 平台定制任务

## 参考资料

\[1\]. [go-task 官方安装指南][1]  
\[2\]. [go-task 官方使用指南][2]  

[1]: https://taskfile.dev/installation/
[2]: https://taskfile.dev/usage/
