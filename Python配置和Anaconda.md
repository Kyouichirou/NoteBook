# Python配置和Anaconda

## python

### 查看安装的路径

注意安装多个不同版本的python的混用的情况, 如再bat文件调用python, 为不同版本的python安装库.

```bash
# 当前使用的python, 进入python环境下
python
```

```python
import os
import sys
os.path.dirname(sys.executable)
# exit()
```

```bash
# 等同于上述的python语句
python -c "import os, sys; print(os.path.dirname(sys.executable))"
```

[cmd where](https://learn.microsoft.com/zh-cn/windows-server/administration/windows-commands/where)

```bash
where python
# 这是cmd的where命令, powershell不支持
# 列出所有的python版本
# 注意windows10下, AppData\Local\Microsoft\WindowsApps目录下的python.exe
```

语法

```
where [/r <Dir>] [/q] [/f] [/t] [$<ENV>:|<Path>:]<Pattern>[ ...]
```

参数

| 参数                                       | 说明                                                         |
| :----------------------------------------- | :----------------------------------------------------------- |
| /r < 目录>                                 | 指示从指定目录开始的递归搜索。                               |
| /q                                         | 返回退出代码 (**0** 表示成功， **1** 表示失败) ，而不显示匹配文件的列表。 |
| /f                                         | 用引号显示 **where** 命令的结果。                            |
| /t                                         | 显示文件大小以及每个匹配文件的上次修改日期和时间。           |
| [$<ENV > \| < Path > ：] < Pattern > [...] | 指定要匹配的文件的搜索模式。 至少需要一个模式，并且模式可以包含通配符 (***** 和 *****) 。 默认 **情况下，搜索** 当前目录和 PATH 环境变量中指定的路径。 你可以通过使用格式 $*ENV*：*pattern* (来指定要搜索的其他路径，其中 *ENV* 是包含一个或多个路径) 或使用格式 *路径*：*pattern* (其中 *path* 是你要在其) 搜索的目录路径的现有环境变量。 这些可选格式不应与 **/r** 命令行选项一起使用。 |
| /?                                         | 在命令提示符下显示帮助。                                     |

### 版本

```bash
python --version
# 等价
python -V
```

## conda

[`anaconda`](https://www.anaconda.com/)国内镜像站点, [阿里云赞助](http://mirrors.aliyun.com/anaconda/archive/?spm=a2c6h.13651104.0.0.fd0e62ceC0wkcY)

这里conda主要用于管理环境, 包管理还是使用`pip`

```bash
# 查看版本
conda --version
# 查看配置
conda config --show
```

### 查看配置

```bash
conda config --show
```

### 谨慎使用update

```bash
# 该命令会导致系列问题
# 对于整体升级python或者anaconda都需要谨慎
# 可以再虚拟环境执行, 或者是VMware这些完全分离的环境操作
conda update python 
```

### 创建虚拟环境

[document](https://docs.conda.io/projects/conda/en/latest/commands/create.html)

```bash
# 查看有哪些python版本可用
conda search "^python$"

conda create -n pack_env python=3.11

# 参数
usage: conda create [-h] [--clone ENV] (-n ENVIRONMENT | -p PATH) [-c CHANNEL]
                    [--use-local] [--override-channels]
                    [--repodata-fn REPODATA_FNS] [--strict-channel-priority]
                    [--no-channel-priority] [--no-deps | --only-deps]
                    [--no-pin] [--copy] [-C] [-k] [--offline] [-d] [--json]
                    [-q] [-v] [-y] [--download-only] [--show-channel-urls]
                    [--file FILE] [--no-default-packages]
                    [--solver {classic} | --experimental-solver {classic}]
                    [--dev]
                    [package_spec [package_spec ...]]
# 参数
# -n, name, 环境名称
# python=3.11, 指定安装的python版本
# --clone
# Create a new environment as a copy of an existing local environment.
# --copy
# Install all packages using copies instead of hard- or soft-linking.
# -p 路径, 指定虚拟环境存储的位置
```

这里需要注意`clone`和`copy`之间的[差异](https://stackoverflow.com/questions/61261827/conda-create-clone-v-s-copying-the-environment-directly).

> clone, Create a new environment as a copy of an existing local environment.

> copy, Install all packages using copies instead of hard- or soft-linking.

还需要进一步控制虚拟环境安装的内容, 参照[conda create 怎么创建纯净的 Python3.6 环境？](https://segmentfault.com/q/1010000011446212)

```bash
# 激活指定的环境
conda activate env_name (or env_path)
# 取消激活
conda deactivate env_name(optional)
```

### 查看虚拟环境

查看所有的虚拟环境

```bash
conda info -e
# 等价
conda env list
```

### 删除环境

```bash
conda remove -n env_name --all
```

### 导出/恢复环境

```bash
# 导出
conda env export > env.yml
# 恢复
conda env create -n revtest -f=/tmp/env.yml
```

### 变更国内镜像站点

Windows 用户无法直接创建名为 `.condarc` 的文件, 可先执行 `conda config --set show_channel_urls yes` 生成该文件之后再修改.

```bash
channels:
  - defaults
show_channel_urls: true
default_channels:
  - http://mirrors.aliyun.com/anaconda/pkgs/main
  - http://mirrors.aliyun.com/anaconda/pkgs/r
  - http://mirrors.aliyun.com/anaconda/pkgs/msys2
custom_channels:
  conda-forge: http://mirrors.aliyun.com/anaconda/cloud
  msys2: http://mirrors.aliyun.com/anaconda/cloud
  bioconda: http://mirrors.aliyun.com/anaconda/cloud
  menpo: http://mirrors.aliyun.com/anaconda/cloud
  pytorch: http://mirrors.aliyun.com/anaconda/cloud
  simpleitk: http://mirrors.aliyun.com/anaconda/cloud
```

> 注：由于更新过快难以同步, 阿里云不同步`pytorch-nightly`, `pytorch-nightly-cpu`,` ignite-nightly`这三个包.

**注意这个镜像站点的同步问题.**

## 变更pip安装地址为国内的镜像源

这里需要注意的是, 国内的镜像站, 未必完全和pip源完全同步, 部分下载量很小的包, 也许没有被同步到国内的镜像

### 永久变更

这里使用的是华为的镜像站点

```bash
pip config set global.index-url https://repo.huaweicloud.com/repository/pypi/simple
```

### 暂时变更

这里使用的是pip源站点

```bash
pip install streamlit-tree-select -i https://pypi.Python.org/simple/
```

## 包管理

### 执行方式的差异

[-m参数的用途](https://stackoverflow.com/questions/60782785/python3-m-pip-install-vs-pip3-install?noredirect=1)

> 如果去看源码的话, 你会发现 pip 作为执行文件的入口点是 pip._internal.main. 
>
> 另一方面, pip 作为模块运行时入口是 _main.py, 而该模块也只是调用 pip.internal.main. 
>
> 所以两种方式本质上是一样的, 需要注意的问题前面也有人提到过了, 如果系统中同时存在多个 python 解释器, 最好检查一下 python 和 pip 是不是来自同一个版本. 

这里主要涉及

- 环境存在多个版本的python
- 没配置python环境变量
- 潜在的其他安全因素?

```bash
python -m pip package_name

pip install pagekage_name
```

*注: 暂未发现直接使用pip install的实际使用产生的问题.*

其他参考, 作者强烈建议使用 [`-m`](https://snarky.ca/why-you-should-use-python-m-pip/)

### 查看安装包的信息

```bash
pip show -f package_name
# 等价
pip show package_name
# 导出所有安装的包
pip list
pip list > list.txt
pip freeze
```

假如没有安装指定的包, 将出现警告信息

> WARNING: Package(s) not found: emoji

### 安装包

```bash	
# 常用命令
pip install package_name

# 指定安装版本
pip install -v numpy==1.13.1
# 等价
pip install robotframework==2.8.7
# 最小版本
pip install 'SomePackage>=1.0.4'

# 指定安装来源
pip install streamlit-tree-select -i https://pypi.Python.org/simple/

# 从版本文件安装
# cd 命令到改文件下
# windows10 shift + 鼠标右键 - 再当前目录下打开终端
pip install pycryptodome-3.6.5-cp37-cp37m-win_amd64.whl
# 等价
pip install <目录>/<文件名>
pip install --use-wheel --no-index --find-links=wheelhouse/ <包名>

# 从github安装(不建议, 稳定性极差)
pip install git + https://github.com/test/test.git
```

### 卸载

```bash
pip uninstall package_name
```

### 升级包

```bash
# 列出所有可以升级的包
pip list -o
# 查看过期的包
pip list --outdated
# 等价
pip list --outd

# 升级指定的包
pip install --upgrade package_name
pip install -U package_name

# 升级所有(不建议), 下载过程漫长(潜在错误)
# 先安装这个库
pip3 install pip-review
# 再执行
pip-review --local --interactive
```

### 导出包列表

例如再虚拟环境下配置好了一个项目, 需要迁移需要的包

```bash
# 导出对应的包
pip freeze > requirements.txt
# 文件将包含名称和对应的版本号
# 导入
pip install -r requirements.txt
```

## 其他-关于镜像站点

大部分的搜索引擎结果都是指向一些国内大学的站点(如清华的镜像站点), 这些大学的站点一般主要面向校园内, 外部访问, 速度不快, 而且这些站点的维护状态也不如人意, 如很多被推荐的清华镜像站点(有一段时间暂停服务), 建议将选择国内巨型公司的站点, 如阿里云, 华为云, 这些站点的稳定性更高, 速度快.

pip官方的地址: https://pypi.Python.org/simple/ (镜像站点并不一定靠谱, 备用)
