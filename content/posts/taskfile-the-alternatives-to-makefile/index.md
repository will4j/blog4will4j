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

Task 是用 Go 语言编写的任务执行/构建工具，对比 GNU make，Task 语法规则[[1]]更加简单且语义化，学习成本更低，同时 Task 脚本支持跨平台执行，除了 Linux 和 Mac 外，Windows 系统通过使用 GitBash 也能完全兼容执行。使用 Task 编写构建任务、自动化脚本，可以极大提高开发协作效率。

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
Task 的核心是任务，对应 `cmds` 参数，支持多种格式[[4]]，最常用的是执行 shell 命令或者执行其他 task。

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

### 任务依赖
除了可以使用 `cmds.task` 直接调用执行其他任务外，还可以通过 `deps` 声明任务依赖，在当前任务开始前，所有依赖会先执行完成，多个依赖项并行执行。

如下所示：

```yaml
version: '3'

tasks:
  default:
    deps: [before_1, before_2]
    cmds:
      - echo "after"

  before_1:
    cmds:
      - echo "before 1"

  before_2:
    cmds:
      - echo "before 2"
```
```shell
$ task
before 2
before 1
after
```

### 环境变量
通过设置环境变量可以控制命令行为（比如调整 pypi 镜像）。 使用 `env` 设置全局环境变量，使用 `Task.env` 设置任务局部环境变量，环境变量使用 `$` 符号访问。

如下示例中，`install` 任务识别全局环境变量使用清华源加速，`install-test` 任务识别单独配置环境变量使用阿里云源加速：

```yaml
version: '3'

env:
  PIP_INDEX_URL: https://pypi.tuna.tsinghua.edu.cn/simple

tasks:
  install:
    desc: install requirements.txt
    cmds:
      - pip install -r requirements.txt

  install-test:
    desc: install test packages
    env:
      PIP_INDEX_URL: https://mirrors.aliyun.com/pypi/simple
    cmds:
      - cmd: echo using index $PIP_INDEX_URL
        silent: true
      - pip install pytest pytest-cov
```

### 变量
变量在 `Taskfile` 脚本中使用，可以增强任务灵活度，方便任务复用。使用 `vars` 设置全局变量，使用 `Task.vars` 设置任务局部变量，变量使用 go 模板如 `{{.VAR_NAME}}` 访问。

变量取值按优先级从高到低依次为：

1. `Task.vars` 中定义的任务局部变量值
2. 被其他任务直接调用时在 `Command.vars` 中定义的变量值
3. 通过命令行参数传入的变量值
4. 通过 `includes` 导入的外部 `Taskfile` 中定义的全局变量值
5. 当前 `Taskfile` 中定义的全局变量值
6. 通过 `includes` 导入的外部 `Taskfile` 中定义的全局环境变量值
7. 当前 `Taskfile` 中定义的全局环境变量值
8. 通过命令行参数传入的环境变量值
9. 系统环境变量值

通过下面的示例来演示变量取值，任务 `echo` 在控制台输出变量 `MESSAGE` 的值：

```yaml
# Taskfile.yml
version: '3'

includes:
  docs: Taskfile1.yml

vars:
  MESSAGE: "v5"

env:
  MESSAGE: "v7"

tasks:
  do-echo:
    internal: true
    vars:
      MESSAGE: "v1"
    cmds:
      - echo {{.MESSAGE}}

  echo:
    desc: echo message
    cmds:
      - task: do-echo
        vars:
          MESSAGE: "v2"
```

```yaml
# Taskfile1.yml
version: '3'

vars:
  MESSAGE: "v4"

env:
  MESSAGE: "v6"
```

根据顺序执行，`{{.MESSAGE}}` 取值按照优先级从高到低依次为 `v1` 到 `v9`，其中 `v3` 是命令行参数传入的变量值，`v8` 是命令行参数传入的环境变量值，`v9` 是系统环境变量值：

```shell
$ export MESSAGE=v9
$ MESSAGE=v8 task echo MESSAGE=v3
v1
# 注释掉 v1 后执行
$ MESSAGE=v8 task echo MESSAGE=v3
v2
# 注释掉 v2 后执行
$ MESSAGE=v8 task echo MESSAGE=v3
v3
$ MESSAGE=v8 task echo
v4
# 依次注释掉 v4，v5，v6 后执行
$ MESSAGE=v8 task echo
# 结果依次为 v5，v6，v7
# 注释掉 v7 后执行
$ MESSAGE=v8 task echo
v8
$ task echo
v9
```

### 工作目录
默认情况下，所有任务命令会在 `Taskfile` 所在目录执行，可以通过 `dir` 指定命令执行目录，比如下面的任务会在 `client` 目录执行客户端构建：

```yaml
version: '3'

tasks:
  client:build:
    desc: build client dist file
    dir: client
    cmds:
      - yarn
      - yarn build
```

### 引用其他 Taskfile
Taskfile 引用可以在多种场景中发挥作用，比方说引入通用工具类，或者根据功能不同将任务进行分组等。

可以像如下示例中的 `docs` 一样直接使用路径进行引入，也可以像 `dev` 一样定义参数引入，需要注意的是，引入后的所有命令默认都在当前 `Taskfile` 目录下执行，除非通过 `dir` 参数修改引入脚本的执行目录。

```yaml
# Taskfile.yml
version: '3'

includes:
  docs: docs/Taskfile.yml
  dev:
    taskfile: dev/Taskfile.yml
    vars:
      REQ_FILE: requirements-dev.txt
```
```yaml
# dev/Taskfile.yml
version: '3'

vars:
  REQ_FILE: requirements.txt

tasks:
  build:
    desc: build dev environment
    cmds:
      - echo "build with {{.REQ_FILE}}"
```
```yaml
# docs/Taskfile.yml
version: '3'

tasks:
  build:
    desc: build docs
    cmds:
      - echo "build docs"
```

```shell
$ task --list
task: Available tasks for this project:
* dev:build:        build dev environment
* docs:build:       build docs
```

### 任务调试

## 进阶使用
### 动态变量
可以通过 shell 脚本在运行时动态设置变量的值。比如下面的任务会获取当前 Git 提交哈希值设置变量 `IMAGE_TAG`：

```yaml
version: '3'

vars:
  IMAGE_TAG:
    sh: git rev-parse --short HEAD
```

### 非必要不执行
对于生成式任务或者执行成本较大的任务，
### go 模板引擎
### 循环执行

## 参考资料

\[1\]. [Taskfile 语法规则][1]  
\[2\]. [Task 官方安装文档][2]  
\[3\]. [Task 官方使用指南][3]  
\[4\]. [Task 命令参数][4]  

[1]: https://taskfile.dev/api/#taskfile-schema
[2]: https://taskfile.dev/installation/
[3]: https://taskfile.dev/usage/
[4]: https://taskfile.dev/api/#command
