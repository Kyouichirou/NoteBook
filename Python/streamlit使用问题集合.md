# streamlit使用问题集合

[streamlit](https://pypi.org/project/streamlit/), 快速web的创建库.

问题:

```base
pip install streamlit
```

使用的anaconda, conda版本4.10.3, 内置的python3.9.7

安装好之后, 进行测试

```base
streamlit hello
```

无法正常打开连接, 出现问题

![error](https://p0.meituan.net/csc/4561e6b05323ebef6f46bc54e869dbf435439.png)

问题的[解决](https://discuss.streamlit.io/t/streamlit-hello-typeerror-protocols-cannot-be-instantiated/29947/6), 根据反馈, 因该是python的版本问题, streamlit在python3.9.7版本下回出现错误.

尝试升级anaconda的版本, 顺便测试一下python新版本(3.10.x号称速度大幅度提升)是否速度更快.

```base
-- 各种升级命令需要注意
conda update python
```

但是, 新的问题出现, 这条命名并无法直接升级python, 应该是和各组件兼容出现问题.

![error](https://p0.meituan.net/dpplatform/1dde3d2290d1236e6da9143456b003f137178.png)

只能使用虚拟环境来进行测试

```base
conda create -n web_env python=3.10
```

创建好了之后, 激活环境又出现问题, 在powershell下, 并无法通过命令行激活虚拟环境.

```base
conda activate web_env
```

执行环境是powershell, 但是打开anaconda navigator, 打开虚拟环境(cmd)

安装streamlit, 测试stream hello, 顺利进入测试页面, 印证了论坛反馈的问题, streamlit在python3.9.7环境下出现bug.

![home](https://p0.meituan.net/dpplatform/305ce8b5f0e62f97650fe37b2871d26250947.png)

```base
-- 注意这种操作会产生潜在的安全风险
Set-ExecutionPolicy RemoteSigned
```

执行选择"Y", 注意这一步还不能执行conda切换, 还需要执行终端的初始化

> anaconda 4.6+ 在powerShell 中 用activate 激活环境 

```base
conda init powershell
```

![activate](https://p0.meituan.net/csc/6ee66e8a2b4bf6f0b3f42378f0899cef7353.png)

