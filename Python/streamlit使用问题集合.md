# streamlit使用问题集合

## 前言

[streamlit](https://pypi.org/project/streamlit/), 快速web的创建库, 用于快速搭建数据展示平台或者时简单交互的web端口(如数据查询, 动态更新展示等).

## 问题

```bash
conda --version
python --version
```

使用环境: anaconda, conda版本`4.10.3`, 内置的python`3.9.7`

```bash
-- 可以预先变更pip的安装源
-- 查看镜像源
pip config list
pip install streamlit
```

安装好之后, 进行测试

```bash
streamlit hello
```

无法正常打开连接, 出现问题

![error](https://p0.meituan.net/csc/4561e6b05323ebef6f46bc54e869dbf435439.png)

检索相关的报错信息后, 发现问题的[解决的方法](https://discuss.streamlit.io/t/streamlit-hello-typeerror-protocols-cannot-be-instantiated/29947/6), 根据该反馈, 因该是python的版本问题, streamlit在python3.9.7版本下回出现错误.

尝试升级anaconda的版本, 顺便测试一下python新版本(官方宣称3.10.xpython号称速度大幅度提升)是否速度更快.

```bash
-- 各种搜索引擎检索到的各种升级命令需要注意, 大部分都是不靠谱的, 不要强行升级, 会引发很多问题
conda update python
```

但是, 新的问题出现, 这条命名并无法直接升级python, 应该是和各组件兼容出现问题.

![error](https://p0.meituan.net/dpplatform/1dde3d2290d1236e6da9143456b003f137178.png)

只能使用虚拟环境来进行测试

[conda create doc](https://docs.conda.io/projects/conda/en/latest/commands/create.html)

```text
-- conda create + 支持的参数
usage: conda create [-h] [--clone ENV] (-n ENVIRONMENT | -p PATH) [-c CHANNEL]
                    [--use-local] [--override-channels]
                    [--repodata-fn REPODATA_FNS] [--strict-channel-priority]
                    [--no-channel-priority] [--no-deps | --only-deps]
                    [--no-pin] [--copy] [-C] [-k] [--offline] [-d] [--json]
                    [-q] [-v] [-y] [--download-only] [--show-channel-urls]
                    [--file FILE] [--no-default-packages]
                    [--solver {classic,libmamba,libmamba-draft} | --experimental-solver {classic,libmamba,libmamba-draft}]
                    [--dev]
                    [package_spec [package_spec ...]]
-p, --prefix
Full path to environment location (i.e. prefix).
```

```bash
-n, name, name of env
conda create -n web_env python=3.10
-- 查看创建的虚拟环境
conda env list
```

创建好了之后, 激活环境又出现问题, 在powershell下, 并无法通过命令行激活虚拟环境.

*注: 建议在不要所有的任务都在实际的环境中执行, 虚拟环境可以用于测试和封装(避免封装的包体积过大)*

```bash
conda activate web_env
-- 取消当前激活的环境
conda deactivate web_env
```

执行环境是powershell, 但是打开anaconda navigator, 打开虚拟环境(cmd)

安装streamlit好, 测试`stream hello`, 顺利进入测试页面, 印证了论坛反馈的问题, streamlit在python3.9.7环境下出现bug.

![home](https://p0.meituan.net/dpplatform/305ce8b5f0e62f97650fe37b2871d26250947.png)

回到前面的powershell无法激活虚拟环境的问题, 这和powershell的安全策略有关.

```bash
-- 注意这种操作会产生潜在的安全风险
Set-ExecutionPolicy RemoteSigned
-- 或者, 执行权限局限于当前用户
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
-- 查看当前的策略列表
Get-ExecutionPolicy -List
```

执行选择"Y", 注意这一步还不能执行conda切换, 还需要执行终端的初始化

> anaconda 4.6+ 在powerShell中用activate激活环境, 需要初始化环境

```bash
conda init powershell
```

![activate](https://p0.meituan.net/csc/6ee66e8a2b4bf6f0b3f42378f0899cef7353.png)

注意PS前面出现了`(base)`的前缀, 而且powershell之后的加载会默认加载base环境.

![base](https://p0.meituan.net/dpplatform/77678ecbff53dcca308d682a2caa981716554.png)

```bash
conda config --show
```

查看conda的配置, 可以看到自动激活base, 处于激活状态.

取消掉自动激活base环境

```bash
conda config --set auto_activate_base False
```

## streamlit使用

关键点

![restart](https://p0.meituan.net/dpplatform/cf7e0caada5925ab4f016556ecc61d5f100088.png)

streamlit执行逻辑是, 每次产生交互都是从新执行一遍代码, 加入页面渲染一次就需要执行复杂的代码, 需要注意缓存和session的使用.

页面刷新, 需要前后保持变量的值传递, 需要使用session_state来存储值, 另外需要注意装饰器 `@st.cache`的使用.

```python
import streamlit as st
import mysql.connector as conn

# 例如数据库, 日志等不能多次运行, 需要保持其状态

@st.cache
def get_database(configs):
    return conn.connect(**configs)
```

streamlit,不一定需要使用st.write()在文档添加内容, 可以类似于Jupiter在执行之后直接将结果渲染出来(也就是所谓的magic方法).

