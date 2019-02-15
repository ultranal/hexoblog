---
layout: post
title: virtualenvwrapper 配合 Visual Studio Code 使用
date: 2019-02-13 22:12:28.323281824 +08:00
categories:
  - python
tags: 
  - virtualenv
  - vscode
---

使用 Git 维护 Python 项目，常常需要使用 `.gitignore` 来忽略 virtualenv 虚拟环境。相比之下，使用 virtualenvwrapper 项目统一管理虚拟环境就方便的多。

# virtualenvwrapper

[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/index.html) 是用来集中管理 virtualenv 环境的解决方案。相比于原生 virtualenv，virtenvwrapper 将所有虚拟环境统一放置在 `~/.virtualenvs` 下，省去了每次找环境的麻烦；同时也避免了将 virtualenv 目录放置在项目目录下、不便管理的问题。

## 安装

使用 pip 安装 virtualenvwrapper 包：
```
$ pip3 install virtualenvwrapper
```

virtualenvwrapper 依赖于环境变量 `WORKON_HOME` 来配置虚拟环境的所在目录，同时用户需要加载 `/usr/local/bin/virtualenvwrapper.sh` 来初始化环境。在 `~/.*shrc` 中新增两行：
```bash
export WORKON_HOME='~/.virtualenvs'
source /usr/local/bin/virtualenvwrapper.sh
```

另外，对于同时使用 Python 2 和 Python 3 的用户，最好是显式地规定使用的 Python 环境：
```bash
export VIRTUALENVWRAPPER_PYTHON='/usr/local/bin/python3'
```

## 使用

使用 `mkvirtualenv` 新建一个虚拟环境：
```
$ mkvirtualenv foo
```

进入虚拟环境使用 `workon` 命令，退出依然使用 `deactivate`：
```
$ workon foo
$ deactivate
```

使用 `rmvirtualenv` 删除虚拟环境：
```
$ rmvirtualenv foo
```

## virtualenvwrapper 配合 Visual Studio Code 使用

参考 [Visual Studio Code 官方文档](https://code.visualstudio.com/docs/python/environments)，可以通过设置 `python.venvPath` 设置 VSCode 搜索 venv 的路径。
打开 VSCode 配置文件，添加一行：
```json
{
    "python.venvPath": "~/.virtualenvs"
}
```


