---
layout: mypost
title: 程序后台运行的几种方法
categories: [Linux]
---

默认情况下，我们自己写的程序在运行时候会把当前 shell 给占据了，这时候想要在执行其他命令比较笨的方法就是再登陆一次创建一个会话；还有时候我们希望程序退出后能继续运行，像 mysql 那样。

下面是我收集整理的程序放到后台运行的几种方式

### &符号

直接在命令后添加一个`&`即可在后台运行

```shell
go run test.go &
```

这时候程序放到后台了，但是程序有输出时还会输出到当前 shell 影响我们的操作。可以把输出重定向到文件中解决这个问题

```shell
go run test.go > test.log 2>&1 &
```

对于后台任务的常见操作方式如下

```shell
# 列出所有后台任务及其jobNumber
# -l 附带 pid
jobs
# 后台任务切换到前台
fg {jobNumber}
# 让后台任务由暂停变为继续运行
# Ctrl+z 后，当前任务会放在后台,运行状态为暂停，需要此命令启动
bg {jobNumber}
```

**注意：** 后台任务的标准 I/O 继承自当前 session，用户退出 session 丢失，程序在进行输出信息会由于找不到 session 可能会挂掉，所以一定要进行输出和错误输出的重定向

**注意：** 如果`shopt | grep huponexit`为 on 的话，当前用户退出，它的后台任务也会收到 SIGHUP 信号导致退出。貌似这个开关默认是 off

**注意：** 如果 huponexit 为 off，用户退出在进来是看不到之前的 jobs 的，想要切换到前台是不行的，只能`ps -aux | grep "go run test.go"`找到 pid 后在`kill pid`杀掉

上面说道 huponexit 如果为 on 时候，用户退出，后台任务也会收到 SIGHUP 引号导致退出。还有一个骚操作做是使用子 shell,然后再在子 shell 中创建后台任务,这样一来就算是 huponexit 为 on 的状态也不会收到 SIGHUP 信号，由于是子 shell 所以 jobs 里面是看不到的

```shell
# 使用()在子shell中创建后台任务
(go run test.go > /dev/null 2>&1 &)
```

### nohup 命令

nohup 的作用是阻止 SIGHUP 信号发到这个进程，同时关闭标准输入，该进程不再能够接收任何输入，即使运行在前台

nohup 命令不会自动把进程变为"后台任务"，所以必须加上&符号

```shell
# 默认输出到nohup.out
nohup command &
# 自定义输出位置
nohup command > myout.file 2>&1 &
```

### setsid 命令

如果我们的进程不属于接受 HUP 信号的终端的子进程，那么自然也就不会受到 HUP 信号的影响了

调用 setsid 函数的进程成为新的会话的领头进程，并与其父进程的会话组和进程组脱离。由于会话对控制终端的独占性，进程同时与控制终端脱离

通过`ps -ef`可以看到父进程是 1，属于 init 进程，并不属于当前进程

```
# setsid输出重定向必须指定
setsid go run test.go > /dev/null
```

### disown 命令

当系统的 huponexit 为 on，想让 job 忽略 SIGHUP 信号是就轮到 disown 了；注意的是 disown 操作的是 jobs，操作后该 job 会从当前 jobs 中移除

注意：由于后台任务的标准 I/O 继承自当前 session，如果任务会向控制台输出信息，当用户退出程序可能会出错，所以 job 一定要做输出重定向

```shell
# 移出最近一个正在执行的后台任务
disown
# 移出所有正在执行的后台任务
disown -r
# 移出所有后台任务
disown -a
# 不移出后台任务，但是让它们不会收到SIGHUP信号
disown -h [%jobId]
```

一般使用步骤如下

```shell
go run test.go > /dev/null 2>&1
# 按下ctrl + z
# [1]+  已停止               go run test.go

# 继续运行
bg 1

# 注意有个%
disown -h %1
```

### screen 工具

Screen 是一款由 GNU 计划开发的用于命令行终端切换的自由软件。用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换，可以看作是窗口管理器的命令行界面版本。它提供了统一的管理多个会话的界面和相应的功能。

也就是说 screen 创建了一个新的会话，并时刻记录这会话的输出，同时也能把在当前会话中显示；用户退出并不影响 screen 创建的会话，从而实现后台一直运行

```shell
# 安装
yum install screen

# 列出当前所有session
screen -ls

# 恢复到某个会话
screen -r {sessionName|pid}

# 创建名为{sessionName}的会话，立即进入
screen -S {sessionName}

# 创建名为{sessionName}的会话,后执行一条命令，且不进入
screen -dmS {sessionName} [shell command]

# 创建名为{sessionName}的会话,后执行一条命令，名字自动命名为[pid.]tty.host
screen [shell command]

# 检查目前所有的screen作业，并删除已经无法使用的screen作业
screen -wipe
```

在每个 screen 下，可以通过下面的快捷键来进行管理多个 shell(窗口)。注意按键的触发方式：**ctrl+a 后松开，再按一次命令键**

- `ctrl+a d` 放到后台，切回到用户会话
- `ctrl+a k` 杀死当前的窗口，同时也将杀死这个窗口中正在运行的进程
- `ctrl+a c` 创建新的 shell，多个 shell 通过 ca 0~9,a,n,p 进行切换
- `ctrl+a w` 显示所有窗口列表
- `ctrl+a |` 左右分屏
- `ctrl+a TAB` 在分屏中跳转

### Tmux 工具

和 screen 用法相似，不过更厉害，快捷键更多(⊙﹏⊙)

```shell
# 安装
yum install tmux
# 新建
tmux new -s {sessionName}
# 恢复
tmux a -t {sessionName}
tmux ls
```

基本操作同 screen，注意按键的触发方式：**ctrl+b 后松开，再按一次命令键**。分为三类，分为会话操作，窗口操作，面板操作

- `ctrl+b d` 放到后台，切回到用户会话
- `ctrl+b c` 创建一个 shell，在底部的 tab 中可以看到有几个 shell 在运行
- `ctrl+a 0~9|w` 多个 shell 中切换，
- `ctrl+a %` 左右分屏
- `ctrl+a o` 在分屏中跳转

### 注册为 Systemd 服务

> Systemd 是 Linux 系统工具，用来启动守护进程，已成为大多数发行版的标准配置

所有的服务的配置文件都存放在`/usr/lib/systemd/system/`，设置开机启动后会在`/etc/systemd/system/multi-user.target.wants`符号链接

命令格式`systemctl [OPTIONS...] COMMAND [UNIT...]`：

```bash
# 列出所有unit文件
systemctl list-unit-files
# 当前已加载到内存中的unit
systemctl list-units
# 新增配置文件后用于刷新配置
systemctl daemon-reload
systemctl start app-web1.service
systemctl stop app-web1.service
# 查看所有日志
tail -f /var/log/messages
# 查看指定服务日志
journalctl -u app-web1.service
```

常用配置写法，如下`app-web1.service`

```conf
[Unit]
Description=演示网站1
After=network-online.target mysqld.service
Wants=mysqld.service

[Service]
Type=simple
WorkingDirectory=/root/
# ExecStart必须是绝对路径,否则直接报错
ExecStart=/root/go-live-server -p 8080 -d www
KillMode=process
Restart=on-failure
RestartSec=1min

[Install]
WantedBy=multi-user.target
```

为了简化操作，可以通过一个脚本来管理服务的安装与启动

```bash
#!/bin/bash
sName="app-web1.service"
installPath="/usr/lib/systemd/system/"

echo "管理服务： ${sName}"
echo "命令： [status|start|stop|restart|enable|disable, log, install, uninstall]"

echo -n "请输入操作:"
read input

case $input in
  status|start|stop|restart|enable|disable)
    if [ ! -f "${installPath}${sName}" ];then
      echo "文件不存在: ${installPath}${sName}"
    else
      echo "systemctl $input $sName"
      systemctl $input $sName
    fi
    ;;
  log)
    echo "journalctl -f -u $sName"
    journalctl -f -u $sName
    ;;
  install)
    echo "cp ${sName} ${installPath}"
    cp ${sName} ${installPath}
    echo "systemctl daemon-reload"
    systemctl daemon-reload
    ;;
  uninstall)
    if [ ! -f "${installPath}${sName}" ];then
      echo "文件不存在,无需卸载"
    else
      echo "systemctl stop $sName"
      systemctl stop $sName
      echo "systemctl disable $sName"
      systemctl disable $sName
      echo "rm ${installPath}${sName}"
      rm ${installPath}${sName}
      systemctl daemon-reload
    fi
    ;;
  *)
    echo "不支持的命令: $input"
    ;;
esac
```

### Supervisor 工具

更方便更强大的 Systemd，和 Systemd 相似，都是通过配置文件来管理多个进程

### 参考

[Linux 技巧：让进程在后台运行更可靠的几种方法](https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/)

[tmux 常用命令](https://www.cnblogs.com/lizhang4/p/7325086.html)

[Linux 守护进程的启动方法](http://www.ruanyifeng.com/blog/2016/02/linux-daemon.html)

[Systemd 入门教程：命令篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

[Systemd 入门教程：实战篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)

[systemctl 中文手册](http://www.jinbuguo.com/systemd/systemctl.html)
