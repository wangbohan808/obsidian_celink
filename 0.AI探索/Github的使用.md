# 📅2026-06-08

## ssh下载比https下载稳定

**HTTPS 走 443 端口、依赖复杂的 TLS/SSL 和代理环境，极易被防火墙 / 杀毒 / 代理 / DNS 拦截；SSH 走 22 端口、加密更底层、链路更短、认证独立，所以在同一网络下经常一次就成功。**




# 📅2026-06-09

## Watt Toolkit

微软应用商店不需要连接VPN可以直接进入下载；

`127.0.0.0/8`表示一个网段，只有前八位确定的一个网段

**如果是在Linux上使用，需要完善证书写入的权限问题**

```bash
wangbohan808@wangbohan:~$ sudo chmod a+w /etc/hosts
```

### 两种代理监听方式

代理监听为什么快：
- VIP通道人少不堵
- VIP通道手续少，没有各种官方通道的手续检查

```text
电脑 → 国内默认路由 → 国际出口（拥堵/丢包）→ GitHub

电脑 → 127.0.0.1:26561 → 代理客户端 → 优选专线/轻量国际出口 → GitHub
```
- 不论是否开启代理，PC对应的网关是不会发生变化的
- 既然最初使用浏览器访问github,浏览器就必须要打开、占用一个端口
- 不同的软件有不同的网络跳转链路，watt相较于默认的会更优


同一个网址会有不同的物理服务器；
使用watt开启host模式会选择更优的服务器



## 程序后台运行

```bash
wangbohan808@wangbohan:~$ /opt/watt/Steam++.sh &
[1] 65490
wangbohan808@wangbohan:~$ 符号链接 /opt/watt/Steam++ 已存在
文件具有执行权限。
证书 'SteamTools' 存在。
info: KestrelServerOptEx[0]
      Listened https://0.0.0.0:443, HTTPS reverse proxy service startup completed.
info: KestrelServerOptEx[0]
      Listened http://0.0.0.0:80, HTTP reverse proxy service startup completed.
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://0.0.0.0:443
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://0.0.0.0:80
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /home/wangbohan808/.local/share/Steam++/Plugins/Accelerator/Yarp

wangbohan808@wangbohan:~$ 
wangbohan808@wangbohan:~$ ps
    PID TTY          TIME CMD
  65477 pts/0    00:00:00 bash
  65490 pts/0    00:00:00 Steam++.sh
  65492 pts/0    00:00:05 Steam++
  65525 pts/0    00:00:00 Steam++.Acceler
  65583 pts/0    00:00:00 ps
wangbohan808@wangbohan:~$ 
```

- `&`：让命令在后台运行，立刻返回终端

```bash
wangbohan808@wangbohan:~$ nohup /opt/watt/Steam++.sh > ~/watt.log 2>&1 &
[1] 65703
wangbohan808@wangbohan:~$ 
wangbohan808@wangbohan:~$ 
wangbohan808@wangbohan:~$ ps
    PID TTY          TIME CMD
  65693 pts/0    00:00:00 bash
  65703 pts/0    00:00:00 Steam++.sh
  65705 pts/0    00:00:06 Steam++
  65733 pts/0    00:00:00 Steam++.Acceler
  65859 pts/0    00:00:00 ps
wangbohan808@wangbohan:~$ 
```

- `nohup`：让程序脱离终端运行，关掉终端也不会被杀死
- `> ~/watt.log 2>&1`：把日志输出到 `~/watt.log` 文件里，不会在终端刷屏
- `&`：让命令在后台运行，立刻返回终端

```bash
wangbohan808@wangbohan:~$ pkill -f "Steam++"
[1]+  Terminated              nohup /opt/watt/Steam++.sh > ~/watt.log 2>&1
wangbohan808@wangbohan:~$ ps
    PID TTY          TIME CMD
  65693 pts/0    00:00:00 bash
  65898 pts/0    00:00:00 ps
```

- `pkill -f "Steam++"`批量删除进程，关闭名称包含“Steam++”的进程


服务：Linux 下常说的**服务**，就是被 `systemd` 统一管控、长期在后台运行的进程

```bash
wangbohan808@wangbohan:~$ sudo vim /etc/systemd/system/watt-toolkit.service

[Unit]
Description=Watt Toolkit (Steam++)
After=network.target

[Service]
Type=simple
User=wangbohan808
ExecStart=/opt/watt/Steam++.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target


# 重载服务配置
sudo systemctl daemon-reload

# 设置开机自启
sudo systemctl enable watt-toolkit.service

# 立即启动服务
sudo systemctl start watt-toolkit.service
```


```bash
# 查看状态
sudo systemctl status watt-toolkit.service

# 停止服务（退出程序）
sudo systemctl stop watt-toolkit.service

# 重启服务
sudo systemctl restart watt-toolkit.service
```


上述配置服务文件后不可访问github，需要关闭系统服务，转而启用用户服务；需要进行如下修改：

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/watt-toolkit.service
```

```bash
[Unit]
Description=Watt Toolkit (Steam++)
# 关键：等图形会话、网络都好再启动
After=graphical-session.target network-online.target

[Service]
Type=simple
# 不用写 User=，用户服务默认就是你当前用户
Environment=DISPLAY=:0
Environment=XAUTHORITY=%h/.Xauthority
WorkingDirectory=%h
ExecStart=/opt/watt/Steam++.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable watt-toolkit
systemctl --user start watt-toolkit

systemctl --user status watt-toolkit
systemctl --user restart watt-toolkit
```