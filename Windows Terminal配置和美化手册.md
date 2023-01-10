# Windows Terminal配置和美化手册

[TOC]

![finish](https://p0.meituan.net/dpplatform/278f5da3b9184b8c2e9a16cfde595510400705.png)

## 1. 环境

`OS: Win10, x64, 21H2`

## 2. 配置

### 2.1 注意事项

注意, 整个过程尽量不要使用命令行工具来执行安装程序, 下载文件的速度相当慢, 甚至下载不了.

![winget](https://p0.meituan.net/dpplatform/ef14045f49dd8d8ae71d37fe93590fb522578.png)

如果需要通过命令行安装, 需要注意在某些版本的`win10`, 包括未更新的`21H2`, PowerShell无法调用`winget`命令, 需要将`app installer(应用安装程序)`这个app升级到最新的版本, 才能调用`winget`.

`winget`假如可以正常使用.

![winget](https://p0.meituan.net/dpplatform/a1a30554da6fa8c4cf4a223fc7af9d20258637.png)

![error](https://p1.meituan.net/dpplatform/141c99028602281251feb8a16cbe1fed47836.png)

相关文件由于墙的原因, 访问非常不稳定, 有条件的最好开启翻墙的工具.

`Microsoft Store`的访问也很不稳定, 最好开启传递优化, 这会大幅度提升下载文件的速度.

![optimzer](https://p0.meituan.net/dpplatform/b584796c5b602113175cb04b1e2193f430322.png)

某些版本的`Microsoft Store`搜索不到o`h-my-posh3`, 最好升级到最新版本的`Store`, 但是`store`的更新需要系统升级.

前置工作需要:

- 更新`Windows`版本
- 更新微软商店
- 更新应用安装程序
- 准备爬墙

### 2.2 配置过程

使用的是第三方的客户端, 而不是[Windows Terminal](https://www.microsoft.com/store/productId/9N0DX20HK701), 选用的时[Fluent Terminal](https://www.microsoft.com/store/productId/9P2KRLMFXF9T), 一款扁平化风格的终端管理器.

`Powershell`使用的还是`Windows`内置的`Powershell5.x`(即`Windows Powershell`), 而不是微软推荐的`Powershell 7(Powershell core)`.

安装好`Fluent Terminal`之后

安装美化工具[oh-my-posh3](https://apps.microsoft.com/store/detail/XP8K0HKJFRXGCK)(注意不是`posh2`, 这个项目已经不在维护), 相关的美化主题位于安装目录之下的`oh-my-posh\themes`下, 该文件只需要安装即可.

创建`Powershell`配置文件

```bash
-- 调用notepad/其他的编辑器
notepad $profile
```

注意有时这个命令会出现无法找到文件路径的错误, 只需要在document文件夹下创建`windowspowershell`文件夹即可(不需要区分大小写).

生成一个名为`Microsoft.PowerShell_profile.ps1`的文件

写入, 保存即可.

```bash
oh-my-posh init pwsh --config $env:POSH_THEMES_PATH\material.omp.json | Invoke-Expression
```

`material.omp.json`, 这个文件就是themes名称, 需要修改主题, 只需要变更这个名称即可.

相关主题的[介绍](https://ohmyposh.dev/docs/themes)

由于`Powershell`的安全策略, 默认阻止外部脚本的执行, 需要开启权限.

```bash
-- 允许本地用户执行脚本
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

由于部分的主题使用了一些奇特的字符, 需要安装外部字体来解决乱码的问题.

字体下载[地址](https://www.nerdfonts.com/font-downloads), 推荐字体`DejaVuSansMono`

下载字体后, 安装即可(多个或一个).

![font](https://p0.meituan.net/dpplatform/f78806234cf2477a406c88b62d69fda331888.png)

调整一下`fluent terminal`相关设置, 为`Powershell`添加启动参数.

![adjust](https://p0.meituan.net/dpplatform/cb49a545a0fe49328ed564314928647532632.png)

```bash
-- 加载时, 不加载powershell相关信息(头部信息)
-NoLogo
```

其他设置, 调整字体, 内容在窗体显示的位置等.

![adjust](https://p0.meituan.net/dpplatform/c8079f7b3fb4aec6e8d0ea4c30b45b2127111.png)

### 2.3 SSH连接

```bash
# ubuntu terminal
sudo apt-get install openssh-server

# 确定ssh处于正常服务状态
ps -e|grep ssh
964 ?        00:00:00 sshd

# 如果未安装net-tools
sudo apt install net-tools
-- 查看ip地址
ifconfig -a
inet 192.168.2.102  netmask 255.255.255.0  broadcast 192.168.2.255
```

![2023-01-07 13 05 02.png](https://img1.imgtp.com/2023/01/07/xJaugW2F.png)

*注意: 这里使用的不是`mosh`登录, 需要取消掉`mosh`*

![2023-01-07 13 11 30.png](https://img1.imgtp.com/2023/01/07/38idkS4y.png)

选择创建好的配置, 打开, 输入`Ubuntu`的对应的账号密码, 即可连接上虚拟机中的`Ubuntu`.

![2023-01-07 13 09 46.png](https://img1.imgtp.com/2023/01/07/H3xyzsmF.png)

![2023-01-07 13 17 22.png](https://img1.imgtp.com/2023/01/07/vcRE0ZPf.png)

运行`python`, 注意默认状态下`Ubuntu`的`python`还是需要输入`python3`(同时也没有安装有pip包管理工具, 需要手动安装).

```bash
sudo apt install python3-pip
```

退出远程登录终端的方法:

| 序号  | 方法         |
| ----- | ------------ |
| 方法1 | 直接关闭终端 |
| 方法2 | 输入`logout` |
| 方法3 | 输入`exit`   |
| 方法4 | `Ctrl + D`   |

### 2.4 WSL2

![2023-01-09 11 37 09.png](https://img1.imgtp.com/2023/01/09/BNyGqeWm.png)

![2023-01-09 11 37 30.png](https://img1.imgtp.com/2023/01/09/dAzNVqpF.png)

## 3. Windows Terminal

Microsoft原生 [Windows Terminal](https://www.microsoft.com/store/productId/9N0DX20HK701)

![terminal](https://p0.meituan.net/dpplatform/28d3277ae4e5fab502c8f28923a7d20213216.png)
