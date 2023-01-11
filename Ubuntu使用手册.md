# Ubuntu使用手册

## 一. 概要

### 1.1 虚拟机安装

`os version: Ubuntu 22.04.1 LTS`

虚拟机安装`Ubuntu`非常简单, 过程略过.

但是注意安装过程, 不建议勾选在线安装软件, 这个过程太慢.

### 1.2 变更软件下载源

![2023-01-08 10 42 35.png](https://img1.imgtp.com/2023/01/08/usgRSJer.png)

不要使用`Ubuntu`默认的中国服务器, 或者是其选的所谓最优服务器(有时选取的服务器在境外, 更慢), 手动选择`阿里云`的服务器即可, 速度非常快.

### 1.3  snap

```bash
(base) skywalker@skywalker-virtual-machine:~$ snap list
名称                         版本                修订版本   追踪               发布者         注记
bare                       1.0               5      latest/stable    canonical✓  base
core20                     20221212          1778   latest/stable    canonical✓  base
firefox                    108.0.2-1         2263   latest/stable/…  mozilla✓    -
gnome-3-38-2004            0+git.6f39565     119    latest/stable/…  canonical✓  -
gtk-common-themes          0.1-81-g442e511   1535   latest/stable/…  canonical✓  -
snap-store                 41.3-60-gfe4703a  582    latest/stable/…  canonical✓  -
snapd                      2.57.6            17883  latest/stable    canonical✓  snapd
snapd-desktop-integration  0.1               49     latest/stable/…  canonical✓  -
```

> 通过Snap可以安装众多的软件包. 需要注意的是，snap是一种全新的软件包管理方式，它类似一个容器拥有一个应用程序所有的文件和库，各个应用程序之间完全独立。所以使用snap包的好处就是它解决了应用程序之间的依赖问题，使应用程序之间更容易管理。但是由此带来的问题就是它占用更多的磁盘空间.

- [snap相关简介](https://blog.csdn.net/icanflyingg/article/details/122943909)
- [Ubuntu-snap详情页](https://cn.ubuntu.com/blog/what-is-snap-application)

### 1.4 下载工具

考虑到`wget`的下载速度太慢, 这里使用`axel`来替代`wget`.

- 安装 `axel`

  ```bash
  apt-get install axel
  ```

- 语法

  axel [options] url1 [url2] [url…]
  选项
  –max-speed=x , -s x 最高速度 x
  –num-connections=x , -n x 连接数 x
  –output=f , -o f 下载为本地文件 f
  –search[=x] , -S [x] 搜索镜像
  –header=x , -H x 添加头文件字符串x（指定 HTTP header）
  –user-agent=x , -U x 设置用户代理（指定 HTTP user agent）
  –no-proxy ， -N 不使用代理服务器
  –quiet ， -q 静默模式
  –verbose ，-v 更多状态信息
  –alternate ， -a Alternate progress indicator
  –help ，-h 帮助
  –version ，-V 版本信息

  实例：
  如指定10个线程，文件存储/tmp/目录下

  ```bash
  axel -n 10 -o /tmp/ https://openslr.magicdatatech.com/resources/12/train-other-500.tar.gz
  # 不要下载这个文件, 巨大无比
  # Ctrl + C取消下载
  ```

### 1.5 文件共享

Windows10上非常简易的文件共享的实现.

> Samba是在Linux系统上实现的SMB（Server Messages Block，信息服务块）协议的一款免费软件。它实现在局域网内共享文件和打印机，是一个客户机/服务器型协议。客户机通过SMB协议访问服务器上的共享文件系统。可以实现Windows系统访问Linux系统上的共享资源

```bash
sudo apt-get install samba

# 安装好之后, 检查
samba --version

# 创建一个需要共享的文件夹
mkdir samba_share

# 授予足够高的权限
sudo chmod 777 samba_share

# 添加用户-Ubuntu的用户
sudo smbpasswd -a skywalker

# 打开samba配置文件
sudo vi /etc/samba/smb.conf
# 注意这个操作

# 担心损坏或者操作失当可以提前备份一份
sudo cp /etc/samba/smb.conf /etc/samba/smb_backup.conf

# 将光标执行最后一行
G

<< config
    [share]
    comment = Share folder

    # 共享目录，这个写的是你刚刚创建的samba共享目录
    path = /home/skywalker/samba_share
    create mask = 0700
    directory mask = 0700

    # 用于登录的账户
    valid users = skywalker
    force user = skywalker
    force group = skywalker

    # 是否公开分享
    public = yes

    # 是否允许来宾用户访问
    guest ok = yes

    # 是否只读
    read only = no

    # 是否可被浏览
    browseable = yes
    available = yes
    writable = yes
config
:wq
# 保存

# 重启服务
sudo service smbd restart
```

在Windows下

`Run`(运行)下输入

```bash
\\192.168.2.102
```

即可实现文件的共享

### 1.6 包格式

- deb

  deb, 源自` Debian`, `Ubuntu`的默认格式.

  ```bash
  # 分析包其依赖项
  # sudo apt install gdebi
  sudo dpkg -i package_name
  ```

- rpm

  > Ubuntu的软件包格式为deb
  > 而RPM格式的包归属于红帽子Red Hat

  需要将rpm的格式包转换为deb的格式包

  ```bash
  sudo apt-get install alien
  
  # 转换包
  sudo alien xxxx.rpm
  # 然后安装
  sudo dpkg -i xxxx.deb
  ```

- sh

  ```bash
  bash Anaconda3-2022.10-Linux-x86_64.sh
  ```

- bin

  假设该安装包位于/home/a1eafall/a.bin，则在终端下执行

  ```
  $cd /home/a1eafall
  ```

  ```bash
  # 赋予权限
  $sudo chmod u+x a.bin
  
  $sudo ./a.bin
  ```

- 压缩文件

  常见格式`.tar.gz,tar.xz`.

  这种格式一般会采用源代码编译安装，或是解压完直接就可以运行的方式，可以通过查看目录内是否有源代码或是configure文件来确实是不是源代码.
  编译安装：使用cd命令进入解压目录.

  ```
  $./configure                       //配置
  $make                              //编译
  $make install                      //安装
  ```

  如果在解压目录下发现/bin/a.sh，就可以直接运行了。

  ```bash
  $./a.sh
  ```

### 1.7 bashrc文件[介绍](https://blog.csdn.net/m0_52650517/article/details/119716929)

> .bashrc文件是一个存在于ubuntu系统内，普通用户目录(/home/dong)或root用户目录(/root)下的隐藏文件。Linux系统中很多shell，包括bash，sh，zs，dash和korn等，不管哪种shell都会有一个.bashrc的隐藏文件，它就相当于shell的配置文件。.bashrc文件在每次打开新的终端时，都要被读取。简单的ls命令不会显示该文件，需要使用指令ls-al进行查看。这个文件主要保存一些终端配置和环境变量，例如：别名alias、路径path等。
> 修改/root路径下的.bashrc文件将会应用到整个系统，属于系统级的配置，而修改用户目录(/home/dong)下的.bashrc则只是限制在用户应用上，属于用户级设置。两者在应用范围上有所区别，建议如需修改的话，修改用户目录下的.bashrc，这样既无需root权限，也不会影响其他用户.
>
> .bash_history：记录之前输入的命令。
> .bash_logoutt：当你退出shell时执行的命令.
> .profile：当你登入shell时执行的命令。一般会在.profile文件中显式调用.bashrc，启动bash时首先会去读取/.profile文件，这样/.bashrc也就得到执行了，你的个性化设置也就生效了.

### 1.8 压缩/解压

| 参数 | 含义                                                |
| ---- | --------------------------------------------------- |
| tar  | Linux压缩/解压缩工具                                |
| -z   | 代表gzip，使用gzip工具进行压缩或解压                |
| -x   | 代表extract，解压文件（压缩文件是-c）               |
| -v   | 代表verbose，显示解压过程（文件列表）               |
| -f   | 代表file，指定要解压的文件名（or 要压缩成的文件名） |

```bash
tar -xvf zip_pack

# 将文件解压到指定的文件夹
tar -xvf zip_pack -C /test_foler
```

### 1.9 权限

> **Linux file ownership**
>
> In Linux, there are three types of owners: `user`, `group`, and `others` .
>
> - Linux User
>
> A user is the default owner and creator of the file. So this user is called owner as well.
>
> - Linux Group
>
> A user-group is a collection of users. Users that belonging to a group will have the same Linux group permissions to access a file/ folder.
>
> You can use groups to assign permissions in a bulk instead of assigning them individually. A user can belong to more than one group as well.
>
> - Other
>
> Any users that are not part of the user or group classes belong to this class.

> **Linux File Permissions**
>
> File permissions fall in three categories: `read`, `write`, and `execute`.
>
> - Read permission
>
> For regular files, read permissions allow users to open and read the file only. Users can't modify the file.
>
> Similarly for directories, read permissions allow the listing of directory content without any modification in the directory.
>
> - Write permission
>
> When files have write permissions, the user can modify (edit, delete) the file and save it.
>
> For folders, write permissions enable a user to modify its contents (create, delete, and rename the files inside it), and modify the contents of files that the user has write permissions to.
>
> - Execute permission
>
> For files, execute permissions allows the user to run an executable script. For directories, the user can access them, and access details about files in the directory.
>
> Below is the symbolic representation of permissions to user, group, and others.

![image-163.png](https://img1.imgtp.com/2023/01/11/DT8l4cQ0.png)

![image-164.png](https://img1.imgtp.com/2023/01/11/Q5kFxAC7.png)

```bash
sudo chmod -R 755 /var/run/mysqld
```



![image-165.png](https://img1.imgtp.com/2023/01/11/WBmy676s.png)

![image-157.png](https://img1.imgtp.com/2023/01/11/PhtOi7Vp.png)

```bash
ls -l
```

![image-158.png](https://img1.imgtp.com/2023/01/11/ONMGNODR.png)

> In the output above, `d` represents a directory and`-` represents a regular file.
>
> d代表目录, `-`代表常规文件

#### 1.9.1 chown

`chown`，change owner, 更改文件的所属用户及组.

3, 4列, 文件所属用户及用户组.

```bash
$ ls -lah .
total 1.2M
drwxr-xr-x 11 shanyue shanyue 4.0K Jun 22 18:42 .
drwxr-xr-x  5 root    root    4.0K Jun 24 11:06 ..
drwxr-xr-x  2 shanyue shanyue 4.0K Jun 10 15:45 .circleci
drwxr-xr-x  2 shanyue shanyue 4.0K Jun 10 15:45 .codesandbox
-rw-r--r--  1 shanyue shanyue  294 May 22  2021 .editorconfig
-rw-r--r--  1 shanyue shanyue  759 Jun 10 15:45 .eslintignore
-rw-r--r--  1 shanyue shanyue 8.4K Jun 10 15:45 .eslintrc.js
drwxr-xr-x  7 shanyue shanyue 4.0K Jun 14 19:06 .git
-rw-r--r--  1 shanyue shanyue   12 May 22  2021 .gitattributes
复制代码
```

通过 `chown -R`，可一并将子文件所属用户及用户组进行修改.

```bash
# 将 . 文件夹下当前目录的用户及用户组设为 shanyue
# -R：遍历子文件修改
$ chown -R shanyue:shanyue .
```

#### 1.9.2 chmod

```bash
chmod permissions filename
```

`mode` 指 `linux` 中对某个文件的访问权限. 通过 `stat` 可获取某个文件的 `mode`.

```bash
# -c：--format
# %a：获得数字的 mode
$ stat -c %a README.md
644

# %A：获得可读化的 mode
$ stat -c %A README.md 
-rw-r--r--
```

- r: 可读，二进制为 100，也就是 4
- w: 可写，二进制为 010，也就是 2
- x: 可执行，二进制为 001，也就是 1

而 `linux`为多用户系统，我们可对用户进行以下分类。

- user, 文件当前用户
- group, 文件当前用户所属组
- other, 其它用户

再回到刚才的 `644` 所代表的的释义

```bash
# rw-：当前用户可写可读，110
# r--：当前用户组可读，010
# r--：其它用户可读，010
# 所以加起来就是 644
-rw-r--r--
```

而通过 `chmod` 即可修改用户的权限.

```bash
$ chmod 777 yarn.lock
复制代码
```

另外也可以以可读化形式添加权限，如下所示：

```bash
# u: user
# g: group
# o: other
# a: all
# +-=: 增加减少复制
# perms: 权限
$ chmod [ugoa...][[+-=][perms...]...]

# 为 yarn.lock 文件的用户所有者添加可读权限
$ chmod u+r yarn.lock
```

- [参考链接](https://juejin.cn/post/7113919813529845768)
- [Linux chmod and chown – How to Change File Permissions and Ownership in Linux (freecodecamp.org)](https://www.freecodecamp.org/news/linux-chmod-chown-change-file-permissions/)

---

## 二. Shell

```bash
# 查看用户登录
w
# 清空终端的显示内容
clear
# 类似于powershell的 cls

# 查看服务运行的状态
service --status-all
 [ - ]  apparmor
 [ ? ]  apport
 [ - ]  console-setup.sh
 [ + ]  cron
 [ - ]  dbus
 [ ? ]  hwclock.sh
 [ + ]  irqbalance
 [ - ]  keyboard-setup.sh
 [ ? ]  kmod
 [ - ]  mysql
 [ ? ]  plymouth
 [ ? ]  plymouth-log
 [ - ]  procps
 [ - ]  rsync
 [ - ]  screen-cleanup
 [ + ]  udev
 [ - ]  ufw
 [ - ]  unattended-upgrades
 [ - ]  uuidd
# 三种状态
+: the service is running
-: the service is not running
?: the service state cannot be determined (for some reason).

# 清空回收站
sudo rm -rf ~/.local/share/Trash/*
```

### 1.1 文件管理

```bash
(base) skywalker@skywalker-virtual-machine:~$ mkdir python_test
(base) skywalker@skywalker-virtual-machine:~$ touch python_test/hello.txt
(base) skywalker@skywalker-virtual-machine:~$ cd python_test
(base) skywalker@skywalker-virtual-machine:~/python_test$ ls
hello.txt
(base) skywalker@skywalker-virtual-machine:~/python_test$ cd
# cd 返回
(base) skywalker@skywalker-virtual-machine:~$ 
```

| 命令                | 说明                 |
| ------------------- | -------------------- |
| touch文件名         | 创建指定文件         |
| mkdir 目录名        | 创建目录(文件夹)     |
| rm 文件名或者目录名 | 删除指定文件或者目录 |
| rmdir 目录名        | 删除空目录           |

```bash
# 删除文件
rm(remove)指令用于删除目录或文件：
语法：           rm [-dfirv][--help][--version][文件或目录...]
补充说明：    执行rm指令可删除文件或目录，如欲删除目录必须加上参数”-r”，否则预设仅会删除文件。 
参数：
                     -d或–directory 　直接把欲删除的目录的硬连接数据删成0，删除该目录。 
                     -f或–force 　强制删除文件或目录。 
                     -i或–interactive 　删除既有文件或目录之前先询问用户。 
                     -r或-R或–recursive 　递归处理，将指定目录下的所有文件及子目录一并处理。 
                     -v或–verbose 　显示指令执行过程。

例如：

删除文件夹：
                     rm -rf folder_name
                     将会删除code目录以及其下所有文件、文件夹。（-r递归）
删除文件：
                     rm -f  filename/path
```

#### 1.1.1 查看文档内容

cat命令：

```bash
(base) skywalker@skywalker-virtual-machine:~$ cat python_test/test.py
import os

print(os.path.exists('/home/skywalker/python_test/hello.txt'))
```

- cat test.txt : 查看test.txt内容
- cat -n test.txt，查看linux.txt文件的内容，并且由1开始对所有输出行进行编号。（包括空白行）
- cat -b linux.txt ，用法和 -n 差不多，但是不对空白行编号.
- cat -s linux.txt，当遇到有连续两行或两行以上的空白行，就代换为一行的空白行。
- cat -e linux.txt，在输出内容的每一行后面加一个$符号。（包括空白行）
- cat linux1.txt linux2.txt，同时显示f1.txt和f2.txt文件内容，注意文件名之间以空格分隔，而不是逗号。
- cat -n linux1.txt>linux2.txt，对linux1.txt文件中每一行加上行号后然后写入到linux2.txt中，会覆盖原来的内容，文件不存在则创建它。
- cat -n linux1.txt>>linux2.txt，对f1.txt文件中每一行加上行号后然后追加到f2.txt中去，不会覆盖原来的内容，文件不存在则创建它。

tail命令：

- tail -f filename 监视filename文件的尾部内容 (默认10行，相当于增加参数 -n 10), 刷新显示在屏幕上. 退出，按下CTRL+C.

#### 1.1.2 换行符

| 系统       | 中文描述 | 英文描述                      | 简写 | 转义符 |
| ---------- | -------- | ----------------------------- | ---- | ------ |
| Windows    | 回车换行 | Carriage Return and Line Feed | CRLF | \n\r   |
| Unix/Linux | 换行     | Line Feed                     | LF   | \n     |
| Mac OS     | 回车     | Carriage Return               | CR   | \r     |

- file, 查看换行符

```
file [选项][参数]
```

- -b：列出辨识结果时，不显示文件名称；
- -c：详细显示指令执行过程，便于排错或分析程序执行的情形；
- -f<名称文件>：指定名称文件，其内容有一个或多个文件名称时，让file依序辨识这些文件，格式为每列一个文件名称；
- -L：直接显示符号连接所指向的文件类别；
- -m<魔法数字文件>：指定魔法数字文件；
- -v：显示版本信息；
- -z：尝试去解读压缩文件的内容。

```bash
# windows下创建的文件
(base) skywalker@skywalker-virtual-machine:~$ file 'samba_share/new 1.txt'
samba_share/new 1.txt: Unicode text, UTF-8 text, with CRLF line terminators

# 原生
(base) skywalker@skywalker-virtual-machine:~$ file python_test/hello.txt
python_test/hello.txt: Unicode text, UTF-8 text
# -b
(base) skywalker@skywalker-virtual-machine:~$ file -b python_test/hello.txt
Unicode text, UTF-8 text
(base) skywalker@skywalker-virtual-machine:~$ file -b 'samba_share/new 1.txt'
Unicode text, UTF-8 text, with CRLF line terminators
```

在Windows的将换行符转为Linux下的, 使用`notepad++`

![2023-01-10 11 04 08.png](https://img1.imgtp.com/2023/01/10/kalmVTfY.png)

![2023-01-10 11 04 29.png](https://img1.imgtp.com/2023/01/10/79PPB8xT.png)

在Ubuntu下

```bash
vim test_a.txt
:set ff?
# 如果出现fileforma＝dos 表示是Windows上的换行符.
# 设置为unix
:set fileformat=unix
# 保存
:wq!
file test_a.txt
```

或者使用`dos2unix`(第三方)进行处理.

#### 1.1.3 编辑内容

- 基于vi

  ![078207F0-B204-4464-AAEF-982F45EDDAE9.jpg](https://img1.imgtp.com/2023/01/09/fjyFzP0A.jpg)

- 基于nano

  ![2023-01-09 11 39 52.png](https://img1.imgtp.com/2023/01/09/XH8uLY5s.png)

在终端中修改文件内容还是比较麻烦的, 只作为应急之用.

```bash
vi python_test/hello.txt
# 按下 i/insert, 进入编辑模式
# 按下esc, 退出编辑
:w, 保存文件但不退出vi　　 
:w file, 将修改另外保存到file中，不退出vi
:w!, 强制保存，不推出vi
:wq, 保存文件并退出vi
:wq!, 强制保存文件，并退出vi
:q, 不保存文件，退出vi
:q!, 不保存文件，强制退出vi
:e!, 放弃所有修改，从上次保存文件开始再编辑
```

以下的这些命令都只能在命令模式下使用，所以首先需要按下 `Esc` 进入命令模式，如果你正处于插入模式.

注意vi/vim区分`大小写`.

### 1.2 vim/vi

![20190401142818701.png](https://img1.imgtp.com/2023/01/08/8xsFF3r2.png)

vi/vim 按键说明

除了上面简易范例的 i, Esc, :wq 之外，其实 vim 还有非常多的按键可以使用。

#### 1.2.1 光标移动/复制粘贴/搜索替换

| 移动光标的方法                                               | 说明                                                         |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| h 或 向左箭头键(←)                                           | 光标向左移动一个字符                                         |
| j 或 向下箭头键(↓)                                           | 光标向下移动一个字符                                         |
| k 或 向上箭头键(↑)                                           | 光标向上移动一个字符                                         |
| l 或 向右箭头键(→)                                           | 光标向右移动一个字符                                         |
| 如果你将右手放在键盘上的话，你会发现 hjkl 是排列在一起的，因此可以使用这四个按钮来移动光标。 如果想要进行多次移动的话，例如向下移动 30 行，可以使用 "30j" 或 "30↓" 的组合按键， 亦即加上想要进行的次数(数字)后，按下动作即可！ |                                                              |
| [Ctrl] + [f]                                                 | 屏幕『向下』移动一页，相当于 [Page Down]按键 (常用)          |
| [Ctrl] + [b]                                                 | 屏幕『向上』移动一页，相当于 [Page Up] 按键 (常用)           |
| [Ctrl] + [d]                                                 | 屏幕『向下』移动半页                                         |
| [Ctrl] + [u]                                                 | 屏幕『向上』移动半页                                         |
| +                                                            | 光标移动到非空格符的下一行                                   |
| -                                                            | 光标移动到非空格符的上一行                                   |
| n<space>                                                     | 那个 n 表示『数字』，例如 20 。按下数字后再按空格键，光标会向右移动这一行的 n 个字符。例如 20<space> 则光标会向后面移动 20 个字符距离。 |
| 0 或功能键[Home]                                             | 这是数字『 0 』：移动到这一行的最前面字符处 (常用)           |
| $ 或功能键[End]                                              | 移动到这一行的最后面字符处(常用)                             |
| H                                                            | 光标移动到这个屏幕的最上方那一行的第一个字符                 |
| M                                                            | 光标移动到这个屏幕的中央那一行的第一个字符                   |
| L                                                            | 光标移动到这个屏幕的最下方那一行的第一个字符                 |
| G                                                            | 移动到这个档案的最后一行(常用)                               |
| nG                                                           | n 为数字。移动到这个档案的第 n 行。例如 20G 则会移动到这个档案的第 20 行(可配合 :set nu) |
| gg                                                           | 移动到这个档案的第一行，相当于 1G 啊！ (常用)                |
| n<Enter>                                                     | n 为数字。光标向下移动 n 行(常用)                            |
| 搜索替换                                                     |                                                              |
| /word                                                        | 向光标之下寻找一个名称为 word 的字符串。例如要在档案内搜寻 vbird 这个字符串，就输入 /vbird 即可！ (常用) |
| ?word                                                        | 向光标之上寻找一个字符串名称为 word 的字符串。               |
| n                                                            | 这个 n 是英文按键。代表重复前一个搜寻的动作。举例来说， 如果刚刚我们执行 /vbird 去向下搜寻 vbird 这个字符串，则按下 n 后，会向下继续搜寻下一个名称为 vbird 的字符串。如果是执行 ?vbird 的话，那么按下 n 则会向上继续搜寻名称为 vbird 的字符串！ |
| N                                                            | 这个 N 是英文按键。与 n 刚好相反，为『反向』进行前一个搜寻动作。 例如 /vbird 后，按下 N 则表示『向上』搜寻 vbird 。 |
| 使用 /word 配合 n 及 N 是非常有帮助的！可以让你重复的找到一些你搜寻的关键词！ |                                                              |
| :n1,n2s/word1/word2/g                                        | n1 与 n2 为数字。在第 n1 与 n2 行之间寻找 word1 这个字符串，并将该字符串取代为 word2 ！举例来说，在 100 到 200 行之间搜寻 vbird 并取代为 VBIRD 则： 『:100,200s/vbird/VBIRD/g』。(常用) |
| **:1,$s/word1/word2/g** 或 **:%s/word1/word2/g**             | 从第一行到最后一行寻找 word1 字符串，并将该字符串取代为 word2 ！(常用) |
| **:1,$s/word1/word2/gc** 或 **:%s/word1/word2/gc**           | 从第一行到最后一行寻找 word1 字符串，并将该字符串取代为 word2 ！且在取代前显示提示字符给用户确认 (confirm) 是否需要取代！(常用) |
| 删除、复制与贴上                                             |                                                              |
| x, X                                                         | 在一行字当中，x 为向后删除一个字符 (相当于 [del] 按键)， X 为向前删除一个字符(相当于 [backspace] 亦即是退格键) (常用) |
| nx                                                           | n 为数字，连续向后删除 n 个字符。举例来说，我要连续删除 10 个字符， 『10x』。 |
| dd                                                           | 剪切游标所在的那一整行(常用)，用 p/P 可以粘贴。              |
| ndd                                                          | n 为数字。剪切光标所在的向下 n 行，例如 20dd 则是剪切 20 行(常用)，用 p/P 可以粘贴。 |
| d1G                                                          | 删除光标所在到第一行的所有数据                               |
| dG                                                           | 删除光标所在到最后一行的所有数据                             |
| d$                                                           | 删除游标所在处，到该行的最后一个字符                         |
| d0                                                           | 那个是数字的 0 ，删除游标所在处，到该行的最前面一个字符      |
| yy                                                           | 复制游标所在的那一行(常用)                                   |
| nyy                                                          | n 为数字。复制光标所在的向下 n 行，例如 20yy 则是复制 20 行(常用) |
| y1G                                                          | 复制游标所在行到第一行的所有数据                             |
| yG                                                           | 复制游标所在行到最后一行的所有数据                           |
| y0                                                           | 复制光标所在的那个字符到该行行首的所有数据                   |
| y$                                                           | 复制光标所在的那个字符到该行行尾的所有数据                   |
| p, P                                                         | p 为将已复制的数据在光标下一行贴上，P 则为贴在游标上一行！ 举例来说，我目前光标在第 20 行，且已经复制了 10 行数据。则按下 p 后， 那 10 行数据会贴在原本的 20 行之后，亦即由 21 行开始贴。但如果是按下 P 呢？ 那么原本的第 20 行会被推到变成 30 行。 (常用) |
| J                                                            | 将光标所在行与下一行的数据结合成同一行                       |
| c                                                            | 重复删除多个数据，例如向下删除 10 行，[ 10cj ]               |
| u                                                            | 复原前一个动作。(常用)                                       |
| [Ctrl]+r                                                     | 重做上一个动作。(常用)                                       |
| 这个 u 与 [Ctrl]+r 是很常用的指令！一个是复原，另一个则是重做一次～ 利用这两个功能按键，你的编辑，嘿嘿！很快乐的啦！ |                                                              |
| .                                                            | 不要怀疑！这就是**小数点***！意思是重复前一个动作的意思。 如果你想要重复删除、重复贴上等等动作，按下小数点『.』就好了！ (常用) |

#### 1.2.2 编辑模式

| 进入输入或取代的编辑模式                                     |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| i, I                                                         | 进入输入模式(Insert mode)： i 为『从目前光标所在处输入』， I 为『在目前所在行的第一个非空格符处开始输入』。 (常用) |
| a, A                                                         | 进入输入模式(Insert mode)： a 为『从目前光标所在的下一个字符处开始输入』， A 为『从光标所在行的最后一个字符处开始输入』。(常用) |
| o, O                                                         | 进入输入模式(Insert mode)： 这是英文字母 o 的大小写。o 为在目前光标所在的下一行处输入新的一行； O 为在目前光标所在的上一行处输入新的一行！(常用) |
| r, R                                                         | 进入取代模式(Replace mode)： r 只会取代光标所在的那一个字符一次；R会一直取代光标所在的文字，直到按下 ESC 为止；(常用) |
| 上面这些按键中，在 vi 画面的左下角处会出现『--INSERT--』或『--REPLACE--』的字样。 由名称就知道该动作了吧！！特别注意的是，我们上面也提过了，你想要在档案里面输入字符时， 一定要在左下角处看到 INSERT 或 REPLACE 才能输入喔！ |                                                              |
| [Esc]                                                        | 退出编辑模式，回到一般模式中(常用)                           |

#### 1.2.3 指令行模式

| 指令行的储存、离开等指令                                     |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| :w                                                           | 将编辑的数据写入硬盘档案中(常用)                             |
| :w!                                                          | 若文件属性为『只读』时，强制写入该档案。不过，到底能不能写入， 还是跟你对该档案的档案权限有关啊！ |
| :q                                                           | 离开 vi (常用)                                               |
| :q!                                                          | 若曾修改过档案，又不想储存，使用 ! 为强制离开不储存档案。    |
| 注意一下啊，那个惊叹号 (!) 在 vi 当中，常常具有『强制』的意思～ |                                                              |
| :wq                                                          | 储存后离开，若为 :wq! 则为强制储存后离开 (常用)              |
| ZZ                                                           | 这是大写的 Z 喔！如果修改过，保存当前文件，然后退出！效果等同于(保存并退出) |
| ZQ                                                           | 不保存，强制退出。效果等同于 **:q!**。                       |
| :w [filename]                                                | 将编辑的数据储存成另一个档案（类似另存新档）                 |
| :r [filename]                                                | 在编辑的数据中，读入另一个档案的数据。亦即将 『filename』 这个档案内容加到游标所在行后面 |
| :n1,n2 w [filename]                                          | 将 n1 到 n2 的内容储存成 filename 这个档案。                 |
| :! command                                                   | 暂时离开 vi 到指令行模式下执行 command 的显示结果！例如 『:! ls /home』即可在 vi 当中察看 /home 底下以 ls 输出的档案信息！ |
| vim 环境的变更                                               |                                                              |
| :set nu                                                      | 显示行号，设定之后，会在每一行的前缀显示该行的行号           |
| :set nonu                                                    | 与 set nu 相反，为取消行号！                                 |

特别注意，在 vi/vim 中，数字是很有意义的！数字通常代表重复做几次的意思！ 也有可能是代表去到第几个什么什么的意思。

举例来说，要删除 50 行，则是用 『50dd』 对吧！ 数字加在动作之前，如我要向下移动 20 行呢？那就是『20j』或者是『20↓』即可。

- [内容连接](https://www.runoob.com/linux/linux-vim.html)

### 2. sudo

>  `super user do`

sudo, 允许普通用户使用超级用户权限执行某些命令. 和`Windows`以`管理员`命令执行`cmd/powershell`类似, sudo之后, 将触发密码输入的要求.

```bash
# 提权操作
sudo doing something

# 暂时切换到root
sudo su

-------------------------------------
su

su为switch user，即切换用户的简写。

格式为两种：

su -l USERNAME（-l为login，即登陆的简写）

su USERNAME

如果不指定USERNAME（用户名），默认即为root，所以切换到root的身份的命令即为：su -root或su -，su root 或su。

su USERNAME，与su - USERNAME的不同之处如下：

su - USERNAME切换用户后，同时切换到新用户的工作环境中。

su USERNAME切换用户后，不改变原用户的工作目录，及其他环境变量目录。



su -

su -，su -l或su --login 命令改变身份时，也同时变更工作目录，以及HOME，SHELL，USER，LOGNAME。此外，也会变更PATH变量。用su -命令则默认转换成成root用户了。

而不带参数的“su命令”不会改变当前工作目录以及HOME,SHELL,USER,LOGNAME。只是拥有了root的权限而已。

注意：su -使用root的密码,而sudo su使用用户密码
```

### 3. apt

apt, advanced package tool, 高级包工具, 属于`debian`系`linux`发行版的关键组成部分之一.

```bash
# 查看安装列表
apt list
# 非常长

# 获取包更新列表
sudo apt-get update
# 对比本地, 可更新的
sudo apt-get upgrade

sudo apt-get upgrade package_name

# 列出可更新的包
apt list --upgradeable

sudo apt-get dist-upgrad package_name
与apt-get upgrade命令相似，apt-get dist-upgrade也会升级软件包。除此之外，它还使用最新版本的软件包来处理依赖关系的更改。它可以智能地解决程序包依赖关系之间的冲突，并根据需要尝试以不重要的程序为代价升级最重要的程序包。与apt-get upgrade命令不同，apt-get dist-upgrade是主动式的，它会安装新软件包或自行删除现有软件包以完成升级。

# 检索
apt search package_name

# 安装
sudo apt install package_name
# 多个
sudo apt install <package_1> <package_2> <package_3>

# 安装, 如果存在则不升级
sudo apt install package_name --no-upgrade

# 升级
sudo apt install package_name --only-upgrade

sudo apt install package_name >= version_number

# 查看安装包的信息
apt show package_name

# 移除
apt remove package_name

# 移除软件包及配置文件
sudo apt purge package_name

# 清理旧版本的软件缓存
sudo apt-get autoclean

清理所有软件缓存
sudo apt-get clean
```

### 4. dpkg

```bash
# 查看已经安装的包
dpkg -l

# -s 查看安装包
alex@DESKTOP-F6VO5U4:/mnt/c/Users/Lian$ dpkg -s net-tools
Package: net-tools
Status: install ok installed
Priority: important
Section: net
Installed-Size: 800
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Architecture: amd64
Multi-Arch: foreign
Version: 1.60+git20181103.0eebece-1ubuntu5
Depends: libc6 (>= 2.34), libselinux1 (>= 3.1~)
Description: NET-3 networking toolkit
 This package includes the important tools for controlling the network
 subsystem of the Linux kernel.  This includes arp, ifconfig, netstat,
 rarp, nameif and route.  Additionally, this package contains utilities
 relating to particular network hardware types (plipconfig, slattach,
 mii-tool) and advanced aspects of IP configuration (iptunnel, ipmaddr).
 .
 In the upstream package 'hostname' and friends are included. Those are
 not installed by this package, since there is a special "hostname*.deb".
Homepage: http://sourceforge.net/projects/net-tools/
Original-Maintainer: net-tools Team <team+net-tools@tracker.debian.org>  

dpkg -L net-tools
# 查看包所涉及的文件

系统安装软件一般在/usr/share，可执行的文件在/usr/bin，配置文件可能安装到了/etc下等。
文档一般在 /usr/share
可执行文件 /usr/bin
配置文件 /etc
lib文件 /usr/lib

dpkg -c filename
# 查看包内容

dpkg -I deb_packagename
# 查看包信息

# 安装
dpkg -i package_name

dpkg -r package_name
# 卸载包

# --purge
dpkg -P package_name
# 卸载软件包的同时删除其配置文件的命令
```

- [更多介绍](https://www.golinuxcloud.com/dpkg-command-in-linux/)

---

## 三. Python

`Ubuntu`目前内置的python版本已经升级到`python3.x`, `python2`已经被移除, 但是`python`的关联的关键词还是`python3`.

```bash
# 无内置的pip, 需要手动安装
sudo apt install python3-pip
```

### 3.1 anaconda

```bash
# 下载安装包, 打开官网下载页, 找到linux即可
# 过慢, 使用阿里镜像
axel -n 10 https://mirrors.aliyun.com/anaconda/archive/Anaconda3-2022.10-Linux-x86_64.sh
wget https://repo.anaconda.com/archive/Anaconda3-2022.10-Linux-x86_64.sh

# 下载好之后
bash Anaconda3-2022.10-Linux-x86_64.sh
# 会列出软件的协议
yes 
# 即可, 后面的操作

# 注意执行
--------------------------------------------------------------------
Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
[no] >>>
# 这里可以选择自动初始化
# 假如没有自动初始化

# 打开环境变量配置文件

sudo vim /etc/profile

# 在最后添加
export PATH=/home/alex/anaconda3/bin:$PATH

# 保存
# 重新载入环境变量, 即可取代原有的python
source /etc/profile
----------------------------------------------------------------------------
# 添加conda到环境变量
source ~/.bashrc

# 安装之后原有的python将被覆盖
python --version
python3 --version
pip list
# 都将指向 anaconda的python
```

```bash
# 暂时移除anaconda路径
echo $PATH
#此时会打印出若干路径, 例如: 
# /home/xxx/anaconda3/bin:/usr/xxx/bin:/usr/xxx/local/bin

# 此时就需要暂时去除PATH中的anaconda环境
PATH=/usr/xxx/bin:/usr/xxx/local/bin
# 再次查看PATH

/usr/xxx/bin:/usr/xxx/local/bin
# 此时, 在此Terminal中PATH暂时去除了Anaconda环境路径, 运行CMake则可以解决冲突. 
```

变更`pip`的镜像源

```bash
pip config set global.index-url https://repo.huaweicloud.com/repository/pypi/simple
```



卸载anaconda

```bash
conda install anaconda-clean
# y
```

---

## 四. 数据库

### 4.1 MongoDB

- version: 6.x

```bash
# 导入公钥
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
# 注意这种方式是不安全, 将被废弃
# Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
# https://stackoverflow.com/questions/68992799/warning-apt-key-is-deprecated-manage-keyring-files-in-trusted-gpg-d-instead?noredirect=1

echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

sudo apt-get update

# 安装mongodb
sudo apt-get install -y mongodb-org

下列软件包有未满足的依赖关系：
 mongodb-org-mongos : 依赖: libssl1.1 (>= 1.1.1) 但无法安装它
 mongodb-org-server : 依赖: libssl1.1 (>= 1.1.1) 但无法安装它
E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系。
# 缺乏依赖的libssl库
```

地址见, [libssl index](http://security.ubuntu.com/ubuntu/pool/main/o/openssl/)

```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1-1ubuntu2.1~18.04.20_amd64.deb
# 直接下载, 速度奇慢无比, 可以使用其他的工具下载
sudo dpkg -i libssl1.1_1.1.1-1ubuntu2.1~18.04.20_amd64.deb
# 安装好之后
# 重新安装mysql
sudo apt install -y mongodb-org
```

```bash
# 查看mongodb的状态
sudo systemctl status mongod
○ mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: https://docs.mongodb.org/manual
# 显然安装好之后, mongodb并没有直接启动

# 启动mongodb
sudo systemctl start mongod

# 查看状态
sudo netstat -anp | grep mongod

# 将mongodb添加到自启动项
sudo systemctl enable mongod
```

![2023-01-08 15 01 36.png](https://img1.imgtp.com/2023/01/08/YrNM24qh.png)

```bash
# 卸载mongodb

# 停止服务
sudo service mongod stop
# 删除
sudo apt purge mongodb-org*

# 删除日志文件
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb
```

### 4.2 MySQL

- version: 8.x

- 首先在MySQL的官网上下载一个[配置文件](https://dev.mysql.com/downloads/repo/apt/)

  ```bash
  wget https://dev.mysql.com/get/mysql-apt-config_0.8.24-1_all.deb
  # 下载好之后
  sudo dpkg -i mysql-apt-config_0.8.24-1_all.deb #[这里根据下载好的文件名进行修改]
  ```

- 刷新一下软件仓库列表

  ```bash
  sudo apt-get update
  ```

- 安装MySQL

  ```bash
  sudo apt-get install mysql-server
  ```

  过程会出现密码输入, 组件配置, 密码的安全模式(一路next, 选择推荐的配置即可)

- 安装好之后的检查

  ```bash
  # 查看 mysql的连接情况
  sudo netstat -anp | grep mysql
  
  # 查看端口开放
  netstat -ano
  
  # 动态的状态获取
  sudo service mysql status
  
  # 停止MySQL服务
  sudo service mysql stop
  
  # 启动MySQL服务
  sudo service mysql start
  
  # 重启服务
  sudo service mysql restart
  
  # 禁止mysql自启动, 开机
  sudo systemctl disable mysql
  
  # 允许
  sudo systemctl enable mysql
  ```

  ![2023-01-08 10 40 45.png](https://img1.imgtp.com/2023/01/08/EKUZs1Mu.png)

- 卸载MySQL

  ```bash
  # remove命令依然还会残存配置等文件
  sudo apt remove package1 package2
  
  # 注意这是完全清除, 包括配置文件
  
  sudo apt purge mysql-*
  
  sudo rm -rf /etc/mysql/ /var/lib/mysql
  
  # 删除所有不需要的包
  sudo apt autoremove
  
  sudo apt autoclean
  ```

---

## 五. SSH

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

---

## 六. [WSL2](https://learn.microsoft.com/zh-cn/windows/wsl/)

`Windows Subsystem for Linux (WSL)`, 可以简单理解为`Windows`下的一个容器, `Linux`在这个容器中运行.

但是和传统的虚拟机相比, 这个容器并不是完全和主机分离, 这个容器中的`Linux`理论上可以获得更好的性能体验和更小的资源占用.

| 功能                                           | WSL 1 | WSL 2 |
| :--------------------------------------------- | :---- | :---- |
| Windows 和 Linux 之间的集成                    | ✅     | ✅     |
| 启动时间短                                     | ✅     | ✅     |
| 与传统虚拟机相比，占用的资源量少               | ✅     | ✅     |
| 可以与当前版本的 VMware 和 VirtualBox 一起运行 | ✅     | ✅     |
| 托管 VM                                        | ❌     | ✅     |
| 完整的 Linux 内核                              | ❌     | ✅     |
| 完全的系统调用兼容性                           | ❌     | ✅     |
| 跨 OS 文件系统的性能                           | ✅     | ❌     |

系统的要求, 同时使用前, 需要启用`CPU`的`虚拟化`, 在主板上.

> To update to WSL 2, you must be running Windows 10...
>
> - For x64 systems: **Version 1903** or later, with **Build 18362** or later.
> - For ARM64 systems: **Version 2004** or later, with **Build 19041** or later.

`wsl`上可以做什么:

*注: 和`wsl1`相比, `wls2`是完整运行`Linux`内核.*

> - [在 Microsoft Store](https://aka.ms/wslstore) 中选择你偏好的 GNU/Linux 分发版。
> - 运行常用的命令行软件工具（例如 `grep`、`sed`、`awk`）或其他 ELF-64 二进制文件。
> - 运行 Bash shell 脚本和 GNU/Linux 命令行应用程序，包括：
>   - 工具：vim、emacs、tmux
>   - 语言：[NodeJS](https://learn.microsoft.com/zh-cn/windows/nodejs/setup-on-wsl2)、Javascript、[Python](https://learn.microsoft.com/zh-cn/windows/python/web-frameworks)、Ruby、C/C++、C# 与 F#、Rust、Go 等
>   - 服务：SSHD、[MySQL](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-database)、Apache、lighttpd、[MongoDB](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-database)、[PostgreSQL](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-database)。
> - 使用自己的 GNU/Linux 分发包管理器安装其他软件。
> - 使用类似于 Unix 的命令行 shell 调用 Windows 应用程序。
> - 在 Windows 上调用 GNU/Linux 应用程序。
> - 运行直接集成到 Windows 桌面的 [GNU/Linux 图形应用程序](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/gui-apps)
> - [将 GPU 加速](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/gpu-compute)用于机器学习、数据科学场景等

### 6.1 安装

`Powershell`, `管理员`身份下

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

上述两条命令执行完成之后, `重启`.

下载一个[wsl update](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi), 安装.

指定`wsl`的版本.

```powershell
wsl --set-default-version 2
```

打开微软商店, 检索`Linux/Ubuntu`, 挑选一个版本, 下载安装, 然后运行即可.

![2023-01-09 10 41 37.png](https://img1.imgtp.com/2023/01/09/cJtX4sGL.png)

首次运行, 需要设置`账户名`和`密码`.

### 6.2 [使用](https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands)

```bash
wsl --version

@Lian ➜ ~ ( base 3.9.12) 101ms wsl --status
Default Distribution: Ubuntu-22.04
Default Version: 2

Windows Subsystem for Linux was last updated on 1/9/2023
The Windows Subsystem for Linux kernel can be manually updated with 'wsl --update', but automatic updates cannot occur due to your system settings.
To receive automatic kernel updates, please enable the Windows Update setting: 'Receive updates for other Microsoft
products when you update Windows'.
For more information please visit https://aka.ms/wsl2kernel.

Kernel version: 5.10.16

@Lian ➜ ~ ( base 3.9.12)  wslconfig /list
Windows Subsystem for Linux Distributions:
Ubuntu-22.04 (Default)


# 查看运行的linux版本
@Lian ➜ ~ ( base 3.9.12) 779ms wsl --list --verbose
  NAME            STATE           VERSION
* Ubuntu-22.04    Running         2

# 指定默认的发行版本
wsl --set-default <Distribution Name>

# 指定运行那个版本
wsl --distribution <Distribution Name> --user <User Name>

# 更新wsl
wsl --update

# 更改默认的发行版本的账户
<DistributionName> config --default-user <Username>

# 关闭指定的发行版本
wsl --terminate <Distribution Name>

wsl --shutdown
```

### 6.3 变更镜像源

源地址的下载速度非常慢.

```bash
# 备份, 防止手抖
sudo cp /etc/apt/sources.list /etc/apt/sources.list.back

sudo nano /etc/apt/sources.list

Ctrl + K, 删除掉原有的文件内容

# 数据来源, 阿里云开发社区, 注意这里的url是不是可靠的.

deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

复制上述内容

Ctrl + O保存 =>> enter
Ctrl + X退出

# 查看修改的内容
cat /etc/apt/sources.list 


# 刷新一下
sudo apt-get update

sudo apt-get upgrade
```

### 6.4 打开

- 在`windows`搜索中输入`Ubuntu`直接启动

- 在`Terminal`中直接打开`WSL`

  ![2023-01-09 11 37 09.png](https://img1.imgtp.com/2023/01/09/BNyGqeWm.png)

  ![2023-01-09 11 37 30.png](https://img1.imgtp.com/2023/01/09/dAzNVqpF.png)

- 在`Terminal`直接输入`wsl`

### 6.5 mysql

在`wsl`上配置`MySQL`远比在虚拟机上**麻烦**.

先按照上述在虚拟机的方式, 在`wsl`配置`MySQL`, 但是安装之后出现各种问题. 暂时转到微软提供的[文档](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-database#install-mysql)上来.

很快就遇到麻烦, 直接安装`MySQL`, 出现依赖问题.(`Ubuntu`的版本为`22.04`)

```bash
alex@DESKTOP-F6VO5U4:/mnt/c/Users/Lian$ sudo apt install mysql-server

[sudo] password for alex:
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 libevent-pthreads-2.1-7 : Depends: libevent-core-2.1-7 (= 2.1.11-stable-1) but 2.1.12-stable-1build3 is to be installed
E: Unable to correct problems, you have held broken packages.
```

由于`libevent-core-2.1-7`的版本(`2.1.12-stable-1build3`)过新, 需要`降低版本`.

```bash
# 安装aptitude, 可以将对应的包的版本降低
# 必须
sudo apt-get install aptitude

# 包的名称就是如此libevent-core-2.1-7, 不要去掉数字
$ 必须
sudo aptitude install libevent-core-2.1-7=2.1.11-stable-1
# =2.1.11-stable-1, 不允许存在空格
```

```bash
# 再次安装
# 必须
sudo apt install mysql-server
```

安装过程和上面的APT的配置包安装的方式不一样, 这个过程是没有配置界面的.

```bash
cat /var/log/mysql/error.log
# 查看日志

# 可以看到, root账户的创建是没有
2023-01-09T06:48:33.198137Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.

# 注意这个文件
alex@DESKTOP-F6VO5U4:/mnt/c/Users/Lian$ sudo cat /etc/mysql/debian.cnf

# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = Jg0RQV8AakMJKisS
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = Jg0RQV8AakMJKisS
socket   = /var/run/mysqld/mysqld.sock

# 安装完成之后, MySQL并没有启动
sudo service mysql start/stop
# 会出现以下的警告
# su: warning: cannot change directory to /nonexistent: No such file or directory

# 关闭掉服务
sudo service mysql stop

# 关键的一步
# 必须
sudo usermod -d /var/lib/mysql/ mysql
# 这一步消除了上面的警告

sudo service mysql start

mysql -uroot -p

mysql -udebian-sys-maint -p

# 以上方式, 均无法正常登录MySQL
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2),  /13

# 是可以正常登录进去的
sudo mysql
```

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```bash
# 必须
alex@DESKTOP-F6VO5U4:/mnt/c/Users/Lian$ sudo cat /etc/mysql/mysql.conf.d/mysqld.cnf
# 将文件的内容[mysqld] 下的 sock, pid等注释取消掉
# 端口从3306改成3307, 以作为和主机上的MySQL作为区分

[sudo] password for alex:
#
# The MySQL database server configuration file.
#
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/s:erver-system-variables.html

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

[mysqld]
#
# * Basic Settings
#
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket  = /var/run/mysqld/mysqld.sock
port            = 3307
datadir = /var/lib/mysql

# If MySQL is running as a replication slave, this should be
# changed. Ref https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_tmpdir
# tmpdir                = /tmp
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 127.0.0.1
mysqlx-bind-address     = 127.0.0.1
#
# * Fine Tuning
#
key_buffer_size         = 16M
# max_allowed_packet    = 64M
# thread_stack          = 256K

# thread_cache_size       = -1

# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover-options  = BACKUP

# max_connections        = 151

# table_open_cache       = 4000

#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
#
# Log all queries
# Be aware that this log type is a performance killer.
# general_log_file        = /var/log/mysql/query.log
# general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Here you can see queries with especially long duration
# slow_query_log                = 1
# slow_query_log_file   = /var/log/mysql/mysql-slow.log
# long_query_time = 2
# log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
# server-id             = 1
# log_bin                       = /var/log/mysql/mysql-bin.log
# binlog_expire_logs_seconds    = 2592000
max_binlog_size   = 100M
# binlog_do_db          = include_database_name
# binlog_ignore_db      = include_database_name
```

```bash
# 必须
sudo mkdir -p /var/run/mysqld
# 必须
sudo chmod -R 755 /var/run/mysqld
# sudo chown mysql /var/run/mysqld/
sudo service mysql start

# sudo chown mysql /var/run/mysqld/mysqld.sock

# 必须
sudo mysql
# 为root账号添加密码
 ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
# 再次登录
mysql -uroot -p

# 默认状态下, MySQL并没有开机自启动

# 如果还是出现问题
# ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
# 备用
-----------------------
sudo service mysql stop
ln -s /var/run/mysqld/mysqld.sock
sudo service mysql start
----------------------------------

# 查看端口开放情况
show global variables like 'port';
```

![2023-01-09 16 17 25.png](https://img1.imgtp.com/2023/01/09/bDYnCYF4.png)

![2023-01-09 16 18 21.png](https://img1.imgtp.com/2023/01/09/dpRPhizQ.png)

至此, 整个配置过程完成, 由于缺乏文档, 以及搜索得到的结果, 绝大部分都不靠谱, 花了不少时间在各种测试上.

如果将`Ubuntu`玩崩了, 可以在`application`下, 找到`Ubuntu`, 高级设置, 重置即可.

![2023-01-09 16 20 22.png](https://img1.imgtp.com/2023/01/09/8ZmkpO2P.png)

注意`Ubuntu`将被初始化, 所有的东西都将抹掉, 包括账号和密码.

### 6.6 nodejs

微软官方文档提供的方式, 需要文件都需要从`GitHub`中下载, 没有梯子, 无法下载.

```bash
# 直接安装可以运行
sudo apt-get install nodejs

# 版本已经老旧, 新版本lts, 已经更新到18
alex@DESKTOP-F6VO5U4:~$ node --version
v10.19.0
```

直接下载官方网站的[linux_nodejs_tar](https://nodejs.org/dist/v18.13.0/node-v18.13.0-linux-x64.tar.xz), 解压后添加环境变量的方式, 并无法运行.

```bash
export PATH=/home/alex/linux_nodejs/bin:$PATH
```

这里改成从`gitee`提供的[镜像地址](https://gitee.com/mirrors/nvm)

```bash
# clone 
git clone https://gitee.com/mirrors/nvm
# 进入目录查看
cd nvm
# 查看文件
ls
# 执行脚本
bash install.sh
```

安装好脚本, 注意需要**`重启`**终端.

```bash
alx@DESKTOP-F6VO5U4:~$ nvm --version
0.39.3
```

这里显示`nvm`的版本, 即证明`nvm`正常的安装好了.

```bash
alx@DESKTOP-F6VO5U4:~$ nvm install --lts
Installing latest LTS version.
Downloading and installing node v18.13.0...
Downloading https://nodejs.org/dist/v18.13.0/node-v18.13.0-linux-x64.tar.xz...
################################################################################################################# 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v18.13.0 (npm v8.19.3)
Creating default alias: default -> lts/* (-> v18.13.0)
alx@DESKTOP-F6VO5U4:~$ node --version
v18.13.0
alx@DESKTOP-F6VO5U4:~$ ls
nvm
alx@DESKTOP-F6VO5U4:~$ npm --version
8.19.3
alx@DESKTOP-F6VO5U4:~$
```

![2023-01-11 13 49 22.png](https://img1.imgtp.com/2023/01/11/qX8lmZtp.png)

```bash
sudo vim /.bashrc
# 设置国内镜像
export NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node

echo $NVM_NODEJS_ORG_MIRROR

# node_mirror: https://npm.taobao.org/mirrors/node/
# npm_mirror: https://npm.taobao.org/mirrors/npm/

# 设置npm镜像
npm config set registry https://registry.npm.taobao.org
npm config set disturl https://npm.taobao.org/dist
npm config set electron_mirror https://npm.taobao.org/mirrors/electron/
npm config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/
npm config set phantomjs_cdnurl https://npm.taobao.org/mirrors/phantomjs/

# 查看
npm get registry
```



---

## 七. Pycharm

- 专业版要求, [Buy PyCharm Professional: Pricing and Licensing, Discounts - JetBrains Toolbox Subscription](https://www.jetbrains.com/pycharm/buy/)

在`pycharm`中使用`wsl(Ubuntu)`中的`python`作为解释器, 这里需要注意, `pycharm`要实现这个功能, 必须是`专业版`(`professional`)才支持, 社区版是不支持加载远端解释器的功能的.

这里的实现并不需要在`Ubuntu`中安装`SSH`.

假如没有专业版, 也能和`wsl`中的`python`实现交互.

![2023-01-09 17 02 19.png](https://img1.imgtp.com/2023/01/10/1JCngG5P.png)

在终端(`Terminal`)中, 可以看到`wsl`下的`python`

作为解释器, 添加到`pycharm`.

首先需要获得专业版的使用, 由于专业版不能直接试用, 需要注册账号才能试用, 颇为麻烦, 以下为通过非常规手段获取专业版的使用.

其中关键在于一个大神分享的一个组件[`ja-netfilter pycharm2022`](https://cn.bing.com/search?q=ja-netfilter+pycharm2022&qs=UT&pq=ja-netfilter+pycha&sk=CT1&sc=7-18&cvid=71A996F3C940482DA9D30815AF52D43C&setmkt=zh-cn&FORM=QBRE&sp=2)

- 下载好[`ja-netfilter`](https://gitee.com/ja-netfilter/ja-netfilter), 将其解压出来, 放置到一个固定的位置(路径不要存在`空格`, `中文`)

- 安装好`pycharm`

- 打开`pycharm`

  关键就在于这一步, 打开`pycharm`并无法直接进入界面进行操作, 必须选择一种激活方式.

  选择激活码的方式(*这里不提供激活码*), 激活之后进入界面, help => 编辑此文件.

  ![2023-01-10 11 39 37.png](https://img1.imgtp.com/2023/01/10/jBcrgRn9.png)

  注意这里的路径只支持`\\`, 或者`/`, 假如这里的路径出现问题, 将导致`pycharm`无法打开.

  ```bash
  Roaming\JetBrains\PyCharm2022.3
  ```

  找到`pycharm64.exe.vmoptions`文件, 文本方式打开, 修改路径即可.

  `bin`目录下, 执行`pycharm.bat`文件将看到

  ![2023-01-10 11 48 47.png](https://img1.imgtp.com/2023/01/10/rziftxEx.png)

- *注意, 以上方式仅作为交流学习用途, 请勿用于商业/或在商业环境中使用破解版本的`pycharm`作为`IED`工具使用, 尊重版权.*

![2023-01-10 11 54 47.png](https://img1.imgtp.com/2023/01/10/RhS9ML3u.png)

---

## 八. 优化占用

![2023-01-09 21 42 59.png](https://img1.imgtp.com/2023/01/10/QKETJ75v.png)

在使用`wsl`一段时间, 特别是安装大型的包之后, 内存剧烈膨胀.

```bash
sudo crontab -e -u root
# -e就是用当前登录用户的角色去执行
# sudo crontab 任务自动执行
```

| Element | Linux Name | Meaning                                                      |
| ------- | ---------- | ------------------------------------------------------------ |
| Daemon  | ‘crond’    | Pronounced “demon” or “day-mon”. These are Linux background system processes. |
| Table   | ‘crontab’  | You write rows to this table when entering a crontab command. Each ‘*’ asterisk represents a segment of time and a corresponding column in each row. |
| Job     | Cron Job   | The specific task to be performed described in a row, paired with its designated time id |

- [crontab介绍](https://linuxhandbook.com/crontab/)

```bash
运行上述的crontab命令之后, 将会出现一个执行引擎的选择
选择: nano

# 在文件的底部添加下列内容
# /45表示45分钟执行一次
# 相关设置见上面的介绍
*/45 * * * * sync; echo 3 > /proc/sys/vm/drop_caches; touch /root/drop_caches_last_run
```

```bash
# 允许启动cron服务而无需输入root密码
sudo nano ~/.bashrc
[ -z "$(ps -ef | grep cron | grep -v grep)" ] && sudo /etc/init.d/cron start &> /dev/null
```

```bash
sudo visudo
# sudo：可以让普通用户拥有root权限去执行命令，sudo的配置文件是/etc/sudoers。
# visudo：通过visudo编辑/etc/sudoers，可以检查语法。
# 打开文件
# 结尾添加一行
%sudo ALL=NOPASSWD: /etc/init.d/cron start
```

- [visudo参考链接](https://www.jianshu.com/p/d3e2c2c613a7)

配置`wsl`文件, 该文件的路径为`%UserProfile%`, 保存文件名为`.wslconfig`, 限制对于硬件的使用.

```bash
[wsl2]
memory=4GB
swap=8GB
localhostForwarding=true
```

全部设置完成, 关闭`wsl`, 重新启动服务.

```bash
wsl --shutdown

# 查看自动任务运行的情况
sudo stat -c '%y' /root/drop_caches_last_run

alex@DESKTOP-F6VO5U4:/mnt/c/Users/Lian$ sudo stat -c '%y' /root/drop_caches_last_run
2023-01-10 10:15:01.486487600 +0800
```

- [参考链接-wsl导致vmmem占用高解决办法](https://zhuanlan.zhihu.com/p/166102340)

---

## 九. 问题

- 按照上述的`apt`方式安装`MySQL`

  ```bash
  sudo service mysql status
  # 正常这个状态会显示出来
  
  # 直接显示没有安装
  alex@DESKTOP-F6VO5U4:~$ rpm -q mysql
  package mysql is not installed
  
  # 但是在dpkg -l下是可以看到MySQL已经被安装
  
  alex@DESKTOP-F6VO5U4:~$ sudo mysql
  [sudo] password for alex:
  ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
  
  # 服务无法识别
  sudo service mysql start
  
  # 和权限有关?
  chmod -R 770 /var/lib/mysql
  # 暂未测试
  # mysql: unrecognized service
  ```

- `MySQL`添加了自启动, 但是还是开机没有自启动.(可能是微软为了加速`wsl`启动`Ubuntu`的考虑, 将绝大部分的服务都禁止自启动?)

- 大量的依赖包, 需要更低版本的支持, 就算是已经安装有.

- 一些服务找不到?
