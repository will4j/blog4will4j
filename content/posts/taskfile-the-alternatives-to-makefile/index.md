---
slug: "taskfile-the-alternatives-to-makefile"
title: "Makefile 平替：跨平台构建脚本 Taskfile"
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
keywords: ["Taskfile"]
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

Task 是用 Go 语言编写的任务执行/构建工具，对比 GNU make，Task 支持跨平台执行，语法规则[[1]]更加简单且语义化，因而学习成本也更低。使用 Task 编写构建任务、自动化脚本，可以极大提高开发协作效率。

Task 执行文件为 `Taskfile`，采用 `yaml` 格式，下面以 cowsay 为例进行快速开始演示：

```yaml
# Taskfile.yml
version: '3'

tasks:
  default:
    desc: "This is the default task"
    silent: true
    cmds:
      - cowsay "Are you ready to go task ?"
```

```shell
$ task
 ___________________________
< Are you ready to go task ? >
 ---------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

`default` 任务非必须，是 Task 不带任务参数执行时的默认选项，可以通过 `task --list` 或者 `task -l` 查看所有可执行任务。

## 工具安装

Task 安装非常方便，参考官方安装文档[[2]]，支持多种包管理工具，比如 Mac / Linux 平台的 Homebrew、Tea，Windows 平台的 Chocolatey、Scoop，Node 的 npm，也可以使用安装包、安装脚本或者直接从源码编译安装。以 brew 为例：

```shell
$ brew install go-task
$ task --version
Task version: 3.32.0
```

## 基本使用
Task 有非常完善的使用文档[[3]]，可以先快速浏览，然后在实际使用时作为手册查阅。

### 命令参数
`cmds` 支持多种参数格式[[4]]，最常用的是执行 shell 命令或者执行其他 task。

```yaml
version: '3'

tasks:
  example:
    desc: example task
    cmds:
      # 执行 shell 命令
      - cmd: echo "hello world"
      # 直接输入字符串与 cmd 等价
      - echo "hello world"
      # 多行 shell 脚本
      - |
        cat << EOF > output.txt
        hello world
        EOF
      # 执行其它 task
      - task: another
  
  another:
    desc: another example task
    cmds:
      - cat output.txt
```

### 变量和环境变量

```yaml
# Taskfile.yml
version: '3'

vars:
  PIP_REQ_FILE: requirements.txt

env:
  PIP_INDEX_URL: https://pypi.tuna.tsinghua.edu.cn/simple

tasks:
  pip-install:
    cmds:
      - pip install -r {{.PIP_REQ_FILE}}
```
### 工作目录
### 任务依赖
### 命令行参数
### Dry run 调试

## 进阶使用
### 动态变量
### Taskfile 导入
### 平台定制任务

## 参考资料

\[1\]. [Taskfile 语法规则][1]  
\[2\]. [Task 官方安装文档][2]  
\[3\]. [Task 官方使用指南][3]  
\[4\]. [Task 命令参数][4]  

[1]: https://taskfile.dev/api/#taskfile-schema
[2]: https://taskfile.dev/installation/
[3]: https://taskfile.dev/usage/
[4]: https://taskfile.dev/api/#command
