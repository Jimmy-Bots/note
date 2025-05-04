---
date: 2025-05-04
authors: 
  - Jimmy
categories:
  - Python
  - Tech
---
# uv 初体验

[astral-sh/uv](https://github.com/astral-sh/uv) 这个工具也已经火了一段时间了，碍于 lab 的服务器之前被强制要求断网，就没去尝试，最近在自己 Mac 上想着试试，~~毕竟 Rust 的速度有谁不馋。~~

`uv` 的速度和轻量化（相对 conda 来说）确实让人很心动，还有和 Python 原版虚拟环境的高度兼容，所以就在此记录一下体验过程，爽一爽（

![速度比较](https://raw.githubusercontent.com/astral-sh/uv/e2d105d045a18528cfe1f89a319a62b21fe2dae5/assets/svg/Benchmark-Dark.svg)

<!-- more -->

## Installation

`uv` 的安装方式很多样，可以独立安装，也可以利用 `Pypi` 进行安装，由于我还并不想完全摒弃 `Conda` 环境，先打算混用着（因为它还可以支持 Conda 安装

```bash
conda install uv -c conda-forge
```

方便起见，我是直接安装在了 base 环境下，安装很快一下子就安装好了。

## Basic Usage

一些基础操作比如创建环境，安装包之类的

### Venv Creation

首先可以利用如下命令提前安装好不同版本的`Python`，然后在创建环境时指定版本号。

```bash
uv python install 3.10 3.11 3.12
```

然后是环境创建，可以指定环境目录名字，默认为`.venv/`，还可以指定`Python`版本。

```bash
uv venv [--PYTHON,-P 3.x.x] [PATH]
```

这样就能创建一个原生的`Python Venv`虚拟环境，但是可以利用`uv`进行管理控制。

### Mirrors and Packages

`uv`可以指定镜像源，我为了图省力方便，就直接全局修改了，其全局配置文件在`~/.uv/config.toml`，可以直接修改。

```toml
[[index]]
url = "https://mirrors.zju.edu.cn/pypi/web/simple"
default = true  
```

当然还可以通过环境变量、`pyproject.toml`进行配置，这里就不过多赘述了，具体可以看[官方文档](https://docs.astral.sh/uv/configuration/files/)

换完镜像源后，下一步就可以开始安装包了，安装方式也是非常简单的，和之前使用`pip`安装基本没有区别。

```bash
uv pip install PACKAGE
```

然后是关于环境的导出和导入，关于导出也很方便

```bash
uv pip freeze > requirements.txt
```

导入的话，也能够根据`requirements.txt`安装精确版本

```bash
uv sync
```

以上这些内容，对于基本使用场景下应该都能满足了。

## Advanced Usage

当然`uv`的功能不仅仅止步于上面这些基础操作，它的功能还是相对非常丰富的，使用起来还是非常灵活的。

### Project Management

不过对于一些项目相关的东西这里暂且不太涉及，因为我也还用不到，但是代替`Poetry`是完全没有问题的，能够非常好得管理好项目依赖的。简单说一下，非常简洁，三行解决项目运行（xs

```bash
uv init
uv sync
uv run
```

上面可以看到三行命令足以说明依赖管理的简洁 ~~（不愧是 rust 写的，cargo 也非常好用~~

### uv Tool

tool 是指一些提供命令行接口的 Python Packages，简单来说就是一些全局可用的 CLI 工具，如果不好好管理容易产生各种依赖问题，之前有`pipx`来解决这个问题，`uv`也提供了相关命令解决。

它提供了`uv tool run`和`uvx`（简写）来即时运行所需要的工具，会创建一个临时隔离缓存环境来运行这些工具，并且全局可用，不需要在各个环境中重复安装，通过单独管理，很好的解决了他们与各个环境间可能存在的冲突问题。

## Summary

总体来说是一款性能超模的`Python`项目环境依赖管理工具，毕竟也才发布几个月，属于是未来可期，当下还是不能完全替代其他的工具，比如`Conda`还能管理一些`C/C++`库的依赖。不过问题不大，共用就好了hhhhh