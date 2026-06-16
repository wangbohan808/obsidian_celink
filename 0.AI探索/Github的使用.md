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
systemctl --user disable --now watt-toolkit
systemctl --user start watt-toolkit

systemctl --user status watt-toolkit
systemctl --user restart watt-toolkit
```



# 📅2026-06-10

## 复现watt

```bash
sudo chown -R $USER:$USER /opt/watt
```




## system进行虚拟机加速

**配置好watt的system加速模式；**

```bash
# 清除旧代理
git config --global --unset http.https://github.com.proxy 2>/dev/null
git config --global --unset https.https://github.com.proxy 2>/dev/null

# 配置 HTTP 代理
git config --global http.https://github.com.proxy http://127.0.0.1:26561
git config --global https.https://github.com.proxy http://127.0.0.1:26561
```


```bash
# 关闭 Git SSL 证书校验（规避 Watt 自建证书报错）
git config --global http.sslVerify false
```


```bash
#!/bin/bash
set -euo pipefail

# -------------------------- 配置区（按你实际情况改）-------------------------
WATT_PORT="26561"
WATT_PROTO="http"  # 改为 http
WATT_IP="127.0.0.1"
# -----------------------------------------------------------------------------

echo "===== Watt Toolkit + Git 代理一键修复脚本 ====="

# 1. 清除旧的 Git 代理
echo "[1/5] 清除旧的 Git 代理配置..."
git config --global --unset http.https://github.com.proxy 2>/dev/null || true
git config --global --unset https.https://github.com.proxy 2>/dev/null || true

# 2. 配置正确的 GitHub 代理
echo "[2/5] 设置 Git 代理：$WATT_PROTO://$WATT_IP:$WATT_PORT"
git config --global http.https://github.com.proxy "$WATT_PROTO://$WATT_IP:$WATT_PORT"
git config --global https.https://github.com.proxy "$WATT_PROTO://$WATT_IP:$WATT_PORT"

# 3. 重置 /etc/hosts 并开放权限
echo "[3/5] 重置 /etc/hosts 并开放权限..."
sudo tee /etc/hosts >/dev/null <<'EOF'
127.0.0.1   localhost
::1         localhost
EOF
sudo chmod 666 /etc/hosts

# 4. 防火墙放行端口
echo "[4/5] 放行端口 $WATT_PORT..."
sudo ufw allow "$WATT_PORT"/tcp 2>/dev/null || true
sudo ufw reload 2>/dev/null || true

# 5. 测试连通性
echo "[5/5] 测试代理连通性（curl）..."
if [ "$WATT_PROTO" = "socks5" ]; then
    curl -v --socks5 "$WATT_IP:$WATT_PORT" --connect-timeout 5 https://github.com
else
    curl -v --proxy "$WATT_IP:$WATT_PORT" --connect-timeout 5 https://github.com
fi

echo
echo "✅ 脚本执行完毕！"

```

- **Linux 下配合 Watt Toolkit（System 系统代理模式）专门给 GitHub 配置 Git 代理的一键运维脚本**，作用：批量清理旧配置、对齐 Watt 代理参数、修复 hosts / 权限、放行防火墙、自动连通性检测，一次性解决代理不生效、端口拦截、配置错乱问题。



**删除上述代理：**

```bash
# 删除 github 专属 http/https 代理 
git config --global --unset http.https://github.com.proxy 
git config --global --unset https.https://github.com.proxy

# 查看当前 Git 代理配置，无输出就是已清空
git config --global --get http.https://github.com.proxy
```

## 终端输出暂停复制

**Ctrl+S 暂停 → 鼠标选中 → Ctrl+Shift+C 复制 → Ctrl+Q 继续**。



## 配置证书

### Ubuntu 22.04 + Watt Toolkit 证书完整配置总结
目标：**系统/终端(curl/Git) + Firefox 浏览器 全量信任 Watt 自签证书**，不再关闭 SSL 校验，兼顾安全与代理正常使用。

#### 一、前置信息
证书原始路径（你的环境）：
`/home/wbh808/.local/share/Steam++/Plugins/Accelerator/SteamTools.Certificate.cer`

#### 二、终端/系统证书配置（curl、Git、系统工具）
##### 1. 创建系统证书目录（Ubuntu22.04 必做）
```bash
sudo mkdir -p /usr/local/share/ca-certificates/
```

##### 2. 复制证书并设置权限
```bash
# 复制证书，后缀改为 .crt
sudo cp /home/wbh808/.local/share/Steam++/Plugins/Accelerator/SteamTools.Certificate.cer /usr/local/share/ca-certificates/steamtools.crt
# 设置标准证书权限
sudo chmod 644 /usr/local/share/ca-certificates/steamtools.crt
```

##### 3. 更新系统证书缓存
```bash
sudo update-ca-certificates
```

##### 4. 验证系统证书
```bash
ls /usr/local/share/ca-certificates/steamtools.crt
```
能列出文件即为复制成功。

##### 5. 配置 Git 信任证书（替代关闭 SSL 校验）
```bash
# 取消之前不安全的全局关闭校验
git config --global --unset http.sslVerify
# 指定 Git 使用 Watt 证书
git config --global http.sslCAInfo /usr/local/share/ca-certificates/steamtools.crt
```

##### 6. 功能测试
```bash
# curl 测试（无需 -k 忽略证书）
curl --proxy http://127.0.0.1:26561 https://github.com

# Git 测试
git clone https://github.com/git/git.git
```
正常访问/拉取、无 SSL 报错即终端配置完成。

---

#### 三、Firefox 浏览器证书配置
Firefox 使用独立证书库，不共用系统证书，需手动导入：
##### 方式1：文件选择器显示隐藏文件（直接导入）
1. 打开 Firefox → 右上角菜单 → **设置** → 隐私与安全
2. 下滑至「安全」区域 → 点击 **查看证书**
3. 切换到 **证书颁发机构** → 点击 **导入**
4. 文件选择窗口按 `Ctrl+H`，显示隐藏文件夹
5. 依次进入：`主目录 → .local → share → Steam++ → Plugins → Accelerator`
6. 选中 `SteamTools.Certificate.cer`，确认导入
7. 勾选：**信任此 CA 以标识网站** → 确定
8. 重启 Firefox

##### 方式2：简化操作（先拷贝证书到桌面，好找）
```bash
cp ~/.local/share/Steam++/Plugins/Accelerator/SteamTools.Certificate.cer ~/Desktop/
```
再按上述步骤，导入桌面的证书文件即可。

---

#### 四、整体校验标准
1. Watt 保持：**System 系统代理模式 + HTTP 协议 + 26561 端口 + 关闭二级代理**
2. 终端 `curl/git` 走代理，无 SSL 证书错误
3. Firefox 访问任意网站，不再弹出安全/证书告警
4. 全程**未关闭 SSL 证书校验**，安全性正常

---

#### 五、后续维护 & 卸载清理
1. 证书为**一次性配置**，重启系统/终端不失效，无需重复操作
2. 若后续卸载 Watt，移除证书恢复环境：
```bash
sudo rm /usr/local/share/ca-certificates/steamtools.crt
sudo update-ca-certificates
git config --global --unset http.sslCAInfo
```
3. Firefox：进入「查看证书 → 证书颁发机构」，删除对应 Watt 根证书即可。





# 📅2026-06-11

## 一些知识

github查看添加的远程仓库：

```bash
git remote -v
```









## 编译的时候个别包下载失败


结合你**仅配置 Git 代理、未开启系统全局代理**的实际场景，精简优化工作流，剔除无效步骤，同时保留核心要点。

### Buildroot 包下载失败 最终精简工作流
适用场景：仅配置 Git 专属代理，**无系统全局 http/https 代理**

#### 一、前置说明
你当前代理只作用于 Git，`wget/curl/Buildroot` 不会走代理，**无需执行 `unset 全局代理`**，该步骤直接跳过。

#### 二、标准执行流程（按顺序）
##### 1. 分析报错日志，提取关键信息
从报错行拿到 3 个关键内容：
1. 编译对象：`host-包名-版本`（例：`host-make-4.3`）
2. 目标文件名+完整后缀（例：`make-4.3.tar.lz`/`make-4.3.tar.gz`）
3. 编译临时目录：`buildroot/output/rockchip_rk3588/build/`

##### 2. 进入 Buildroot 本地缓存目录
Buildroot 固定缓存路径：`buildroot/dl/包名/`
```bash
cd ~/rk3588_sdk/buildroot/dl/包名
```

##### 3. 手动下载对应源码包
优先使用 `curl` 伪装浏览器（规避站点防盗链 403），按优先级选择：
###### 首选：GNU 官方源（http 协议，兼容性最好）
```bash
curl -L -A "Mozilla/5.0" http://ftp.gnu.org/gnu/包名/完整文件名 -o 完整文件名
```

###### 备选：国内镜像（阿里/清华）
```bash
curl -L -A "Mozilla/5.0" https://mirrors.aliyun.com/gnu/包名/完整文件名 -o 完整文件名
```

###### 兜底：跨设备传输
网络完全受限，用手机/电脑下载文件，通过 `scp/U盘` 传到当前目录。

> 重点：日志出现几种压缩格式（`.tar.gz`/`.tar.lz`），就下载对应所有格式文件。

##### 4. 清理编译残留（**必做，核心步骤**）
下载完成后，删除失败产生的临时目录、`.stamp` 标记文件，避免旧状态干扰：
```bash
cd ~/rk3588_sdk
rm -rf buildroot/output/rockchip_rk3588/build/host-包名-版本*
```

##### 5. 重新编译
```bash
# 编译整个 SDK
./build.sh

# 仅单独编译 Buildroot（按需使用）
cd buildroot && make
```

#### 三、不同报错对应处理
1. **403 Forbidden**
   禁用 `wget`，统一使用 `curl + 浏览器UA` 下载。
2. **404 Not Found**
   严格核对**文件名、版本、后缀**，和日志保持完全一致，补全缺失格式的包。
3. **连接超时**
   切换镜像源，或使用跨设备传包方案。

#### 四、极简速记
读日志 → 提取包名/版本/文件名 → `dl/包名` 目录手动下包 → 清理 `output/build` 残留 → 重新编译。




## repo也需要换源

```bash
# 1. 写入 Shell 环境变量，永久指定 repo 清华源
echo 'export REPO_URL=https://mirrors.tuna.tsinghua.edu.cn/git/git-repo' >> ~/.bashrc
source ~/.bashrc

# 2. 配置 ~/bin 并加入环境变量
mkdir -p ~/bin
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# 3. 从清华下载 repo 启动脚本
curl -sSL https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo
chmod a+x ~/bin/repo

# 4. 制作本地缓存（后续新项目直接拷贝，彻底免排队）
mkdir -p ~/repo-cache
cp ~/bin/repo ~/repo-cache/

# 校验
echo "REPO_URL: $REPO_URL"
which repo
```




## 使用clash进行加速下载

```bash
# 清除旧代理配置
git config --global --unset http.https://github.com.proxy
git config --global --unset http.proxy
git config --global --unset https.proxy
```

Yocto 仓库默认用 `git://` 协议，该端口被拦截，强制转为 `https`：

```bash
git config --global url."https://git.yoctoproject.org/".insteadOf git://git.yoctoproject.org

git config --global url."https://git.openembedded.org/".insteadOf git://git.openembedded.org

# 所有 git:// 协议统一转为 https://
git config --global url."https://".insteadOf git://
```

```bash
# 使用新端口
export HTTP_PROXY=http://127.0.0.1:7897
export HTTPS_PROXY=http://127.0.0.1:7897
export NO_PROXY=localhost,127.0.0.1,::1
```





# 📅2026-06-12

## 不同的包管理器需要各自对应的换源

> 拉取包有不同的包管理器，不同的包管理器都需要设置独立的源替换
> 同一包管理器的不同源仓库只是拉取代码的位置不同、下载的网速不同，实际储存的内容是完全相同的


好的，按照你的要求，我整理了一份常用包管理器更换国内源的速查表。核心就是你说的：**换源只改下载地址，不影响内容，每个包管理器都要单独配置。**

### 📦 常用包管理器换源速查表

| 包管理器 | 适用场景 | 配置文件/命令 | 国内镜像站示例 | 配置命令 (以清华源为例) |
| :--- | :--- | :--- | :--- | :--- |
| **APT** | **Debian/Ubuntu** 系统软件包 | `/etc/apt/sources.list` | `mirrors.tuna.tsinghua.edu.cn` | 手动编辑配置文件替换源地址 |
| **YUM/DNF** | **CentOS/RHEL/Fedora** 系统软件包 | `/etc/yum.repos.d/*.repo` | `mirrors.tuna.tsinghua.edu.cn` | 手动编辑或替换 `.repo` 文件中的源地址 |
| **pip** | **Python** 软件包 | `~/.pip/pip.conf` | `pypi.tuna.tsinghua.edu.cn/simple` | `pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple` |
| **npm** | **Node.js** 软件包 | `~/.npmrc` | `registry.npmmirror.com` | `npm config set registry https://registry.npmmirror.com` |
| **Docker** | **容器镜像** | `/etc/docker/daemon.json` | `docker.mirrors.tuna.tsinghua.edu.cn` | 编辑 `daemon.json` 文件，添加 `registry-mirrors` 参数 |
| **Git** | **代码仓库** | `~/.gitconfig` | `mirrors.tuna.tsinghua.edu.cn/git/...` | `git config --global url."https://新地址".insteadOf "原地址"` |
| **Go** | **Go 语言模块** | 环境变量 `GOPROXY` | `goproxy.cn` | `go env -w GOPROXY=https://goproxy.cn,direct` |

---

### 🚀 核心配置方式概览

为了方便查阅，这些工具的配置方式主要可以分为以下三类：

**1. 直接修改配置文件**
- **APT、YUM、Docker** 都属于这一类。你需要找到并编辑它们对应的系统配置文件（例如 `sources.list`、`*.repo`、`daemon.json`），将官方源的 URL 替换为镜像站的 URL。

**2. 使用专用命令行配置**
- **pip、npm、Go** 提供了非常便捷的命令行工具来修改配置，无需手动查找和编辑文件，一键切换，最为方便。

**3. 使用 URL 替换**
- **Git** 的配置方式比较特殊，它不是直接换掉地址，而是通过 `git config --global url."新地址".insteadOf "原地址"` 命令，告诉 Git 在访问原地址时自动“跳转”到新地址。

### 💡 几点小建议

1.  **常用镜像站**：国内常用的镜像站包括**清华大学 (`tuna`)、阿里云、中科大 (`ustc`)** 等。通常情况下，选择离你最近的即可。
2.  **临时使用**：大部分包管理器都支持在安装命令后加参数临时指定一个源（如 `pip install -i 镜像地址`），这对于测试或只安装一两个包很方便。
3.  **团队协作**：如果是团队项目或 CI/CD 环境，建议将换源的配置步骤或脚本固化下来，确保所有成员的开发环境保持一致。




## 验证与修复代理遗留

```bash
# 查看当前有哪些Git代理配置
git config --global --list | grep proxy

# 验证环境变量已清除
env | grep -i proxy
```





## 开机自启的一些服务


以下是 Ubuntu 22.04 中**可能造成开机自启执行脚本或服务**的所有常见位置，涵盖系统级、用户级、桌面环境及登录会话等各个层面：

---

### 一、系统级（影响所有用户）

#### 1. Systemd 服务
| 路径 | 说明 |
|------|------|
| `/etc/systemd/system/` | 管理员自定义的 systemd 服务、定时器、挂载等单元 |
| `/lib/systemd/system/` | 软件包安装的系统服务（不建议直接修改） |
| `/usr/lib/systemd/system/` | 同上（部分发行版） |
| `/etc/systemd/user/` | 管理员为所有用户定义的 user 服务（需启用 `linger`） |

**启用命令**：`systemctl enable <service>`

#### 2. SysV init 脚本（兼容性）
| 路径 | 说明 |
|------|------|
| `/etc/init.d/` | System V 风格的启动脚本（仍被 systemd 兼容） |
| `/etc/rc*.d/` | 运行级别目录（通常由 `update-rc.d` 管理） |

#### 3. 启动时执行的脚本文件
| 路径 | 说明 |
|------|------|
| `/etc/rc.local` | 传统用户自定义启动脚本（需可执行，且 systemd 有 `rc-local.service`） |
| `/etc/rc.boot/` | 早期 Ubuntu 使用的目录（现很少用） |

#### 4. 环境变量与 profile 脚本
| 路径 | 说明 |
|------|------|
| `/etc/environment` | 系统全局环境变量（键值对格式，不执行脚本） |
| `/etc/profile` | 登录 shell 执行的全局脚本 |
| `/etc/profile.d/*.sh` | 全局 profile 片段（每个登录 shell 都会执行） |
| `/etc/bash.bashrc` | 交互式 bash shell 的全局配置（非登录 shell 也会执行） |

#### 5. PAM 环境文件
| 路径 | 说明 |
|------|------|
| `/etc/security/pam_env.conf` | PAM 模块 `pam_env` 的设置文件 |
| `/etc/default/locale` | 也被 `pam_env` 读取，常见于设置 `LANG` 等 |

#### 6. 定时任务（可能开机后不久执行）
| 路径 | 说明 |
|------|------|
| `/etc/crontab` | 系统级 crontab（可指定 @reboot 任务） |
| `/etc/cron.d/*` | 系统 cron 任务目录 |
| `/etc/cron.daily/`、`/etc/cron.hourly/` 等 | 通过 run-parts 执行，但不一定是开机执行 |
| `/etc/anacrontab` | 用于防止频繁关机的任务（延迟执行） |

#### 7. 显示管理器（Display Manager）启动脚本
| 路径 | 说明 |
|------|------|
| `/etc/gdm3/Init/Default` | GDM3 初始化脚本（X11 会话启动前） |
| `/etc/gdm3/PostLogin/Default` | 用户登录后、会话启动前执行 |
| `/etc/gdm3/PreSession/Default` | 会话准备阶段 |
| `/etc/lightdm/lightdm.conf` | LightDM 配置文件，可指定 `session-setup-script` |
| `/etc/lightdm/Xsession` | LightDM 使用的 Xsession 脚本 |
| `/etc/X11/Xsession.d/` | Xsession 执行的脚本片段（所有 X 会话都会运行） |
| `/etc/X11/xinit/xinitrc.d/` | startx 或 xinit 时执行的脚本 |

---

### 二、用户级（仅当前用户）

#### 1. 家目录中的配置文件（登录/交互 shell）
| 路径 | 说明 |
|------|------|
| `~/.profile` | 登录 shell 执行（bash、sh、zsh 等） |
| `~/.bash_profile` | bash 登录 shell 优先于 `~/.profile` |
| `~/.bash_login` | 同上 |
| `~/.bashrc` | 交互式非登录 shell 执行（例如打开终端） |
| `~/.zshrc` | Zsh 配置文件 |
| `~/.zshenv` | Zsh 总是读取的环境文件 |
| `~/.config/environment.d/*.conf` | systemd user 环境变量目录（systemd 用户实例读取） |

#### 2. X 会话与桌面环境自动启动
| 路径 | 说明 |
|------|------|
| `~/.xsession` | X11 会话启动脚本（显示管理器会尝试执行） |
| `~/.xsessionrc` | Debian/Ubuntu 推荐的用户级 Xsession 片段 |
| `~/.xinitrc` | startx 或 xinit 时执行 |
| `~/.xprofile` | 显示管理器启动会话前执行（常用于设置环境变量） |
| `~/.config/autostart/` | **桌面环境自动启动目录**（存放 `.desktop` 文件，符合 [Desktop Entry Specification]） |
| `~/.config/upstart/` | 废弃的 Upstart 用户作业（Ubuntu 14.04 之前） |

#### 3. Systemd 用户服务
| 路径 | 说明 |
|------|------|
| `~/.config/systemd/user/` | 用户级 systemd 服务（需 `systemctl --user enable`） |
| `~/.local/share/systemd/user/` | 备用路径（某些软件安装位置） |

**启用用户服务自启**：`systemctl --user enable <service>`  
**开启用户实例开机自启**：`sudo loginctl enable-linger <username>`

#### 4. 用户定时任务
| 路径 | 说明 |
|------|------|
| `crontab -e` 编辑的当前用户 crontab | 可使用 `@reboot` 字段 |
| `~/.cron*` 目录 | 一般不直接使用 |

#### 5. Shell 别名与函数文件（间接自启）
| 路径 | 说明 |
|------|------|
| `~/.aliases`、`~/.functions` | 常被 `~/.bashrc` 或 `~/.zshrc` source，不属于直接自启但常被引入 |

---

### 三、其他值得注意的位置

- **snap 应用自启**：snap 包通过 `daemon` 或 `autostart` 配置，通常在 `/snap/<snap-name>/current/` 下定义，自启项由 snapd 管理。
- **Flatpak 应用自启**：通过 `~/.config/autostart/` 间接实现，或由 Flatpak 的 `--socket=session-bus` 后台服务机制。
- **dbus 服务激活**：`/usr/share/dbus-1/services/` 及 `~/.local/share/dbus-1/services/` 中的 `.service` 文件可让服务在收到 D-Bus 请求时启动，并非严格开机自启，但常被误解。
- **登录 shell 的 source 链**：例如 `~/.bashrc` 可能 source `~/.bash_aliases`、`~/.local/bin/` 下的脚本等，需要递归追踪。

---

### 快速排查建议（针对你当前代理残留问题）

因为你已经检查了常见 shell 配置和 systemd user 环境，但代理变量仍然存在，建议重点检查：

1. **`/etc/profile.d/`** 中的脚本（可能不是 `.sh` 后缀，用 `grep -r proxy /etc/profile.d/` 无后缀搜索）。
2. **`/etc/X11/Xsession.d/`** 下的文件（这些会在你的显示管理器启动 X 会话前执行，很可能设置代理）。
3. **`~/.xprofile`**（你之前查过但可能由于权限或隐藏字符没输出，建议 `cat -A ~/.xprofile` 查看）。
4. **桌面环境自带的自启动 GUI 应用**（如 Steam++ 或 Clash Verge 可能在 `~/.config/autostart/` 中创建了 `.desktop` 文件，虽然该目录不存在，但可能由它们自己的守护进程通过 D-Bus 导入变量）。
5. **PAM 环境**：检查 `/etc/security/pam_env.conf` 和 `/etc/default/locale`。

---

如果需要**自动清除**这类持久化代理变量，可在 `~/.profile` 末尾添加：
```bash
# 清除已知代理环境变量
unset http_proxy https_proxy HTTP_PROXY HTTPS_PROXY all_proxy ALL_PROXY
```
这会在每次登录 shell 时生效，不影响其他环境需求。




## 镜像与代理同时配置

- **镜像和代理是“接力”关系，不是冲突关系**
- **如果镜像里有该文件，就不会去访问原始 URL，也不会触发代理**。只有当镜像里没有（或下载失败）时，才会回退到原始 URL，此时如果你的网络需要代理才能访问原始 URL


```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global --unset http.https://github.com.proxy
```
- 清除git代理防止干扰

```bash
git config --global url."https://".insteadOf git://
# 因为只配置http代理，所以首先需要转化一下

# HTTP/HTTPS 代理
export HTTP_PROXY=http://127.0.0.1:7897
export HTTPS_PROXY=http://127.0.0.1:7897
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897

# SOCKS5 代理（作为备用/特殊应用）
export ALL_PROXY=socks5://127.0.0.1:7897
export all_proxy=socks5://127.0.0.1:7897

# 直连例外
export NO_PROXY="localhost,127.0.0.1,::1,*.tuna.tsinghua.edu.cn,*.ustc.edu.cn,*.aliyuncs.com,*.aliyun.com"
export no_proxy="localhost,127.0.0.1,::1,*.tuna.tsinghua.edu.cn,*.ustc.edu.cn,*.aliyuncs.com,*.aliyun.com"
```
- 配置`http https`走代理的端口
- 强调访问清华源以及本机的`http https`不会走代理



```bash
mkdir -p ~/.bitbake
cat > ~/.bitbake/site.conf <<'EOF'
# 全局镜像配置（适用于所有 Yocto 构建）
BB_GENERAL_MIRROR = "http://mirrors.ustc.edu.cn/yocto/sources/ http://mirrors.tuna.tsinghua.edu.cn/yocto/sources/"
EOF
```
- 为`bitback`创建国内镜像源



- 使用clash直连是不行的，需要选择一个稳定的节点，不然无法正常访问下载
- clash混合端口 `7897` 同时支持 HTTP/SOCKS5


```bash
top
```
- 查看CPU在各个进程的资源占用


```bash
#!/bin/bash

# ==================== 0. 清空所有现有配置 ====================
# 清空 Git 全局配置中的代理和 URL 重写规则
git config --global --unset-all http.proxy 2>/dev/null
git config --global --unset-all https.proxy 2>/dev/null
git config --global --unset-all url."https://".insteadOf 2>/dev/null
git config --global --unset-all url."https://github.com/".insteadOf 2>/dev/null
git config --global --unset-all url."git@github.com:".insteadOf 2>/dev/null

# 清空所有代理相关的环境变量（当前 shell）
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy
unset ALL_PROXY all_proxy
unset NO_PROXY no_proxy

echo "✅ 已清空所有现有配置"

# ==================== 1. 配置 Git 协议转换 ====================
# 将所有 git:// 协议转换为 https://（让 Clash 可以代理）
git config --global url."https://".insteadOf git://

echo "✅ Git 协议转换配置完成：git:// → https://"

# ==================== 2. 配置环境变量代理 ====================
# HTTP/HTTPS 代理（大小写都保留）
export HTTP_PROXY=http://127.0.0.1:7897
export HTTPS_PROXY=http://127.0.0.1:7897
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897

# SOCKS5 代理（某些应用需要）
export ALL_PROXY=socks5://127.0.0.1:7897
export all_proxy=socks5://127.0.0.1:7897

# 直连例外（对 wget/curl 有效）
export NO_PROXY="localhost,127.0.0.1,::1,*.tuna.tsinghua.edu.cn,*.ustc.edu.cn,*.aliyuncs.com,*.aliyun.com"
export no_proxy="localhost,127.0.0.1,::1,*.tuna.tsinghua.edu.cn,*.ustc.edu.cn,*.aliyuncs.com,*.aliyun.com"

echo "✅ 环境变量代理配置完成"

# ==================== 3. 配置 bitbake 国内镜像 ====================
mkdir -p ~/.bitbake
cat > ~/.bitbake/site.conf <<'EOF'
# 全局镜像配置（适用于所有 Yocto 构建）
BB_GENERAL_MIRROR = "http://mirrors.ustc.edu.cn/yocto/sources/ http://mirrors.tuna.tsinghua.edu.cn/yocto/sources/"

# 可选：配置代理（如果镜像都没有才走代理）
# BB_FETCH_PROXY = "http://127.0.0.1:7897"
EOF

echo "✅ bitbake 镜像配置完成"

# ==================== 4. 验证配置 ====================
echo ""
echo "==================== 验证结果 ===================="
echo ""

echo "=== 1. Git 配置 ==="
git config --global --list | grep -E "url|proxy" || echo "无代理配置（正确）"
echo ""

echo "=== 2. 代理环境变量（当前 shell）==="
env | grep -i proxy | grep -v "unset" || echo "❌ 代理未设置"
echo ""

echo "=== 3. bitbake 镜像配置 ==="
cat ~/.bitbake/site.conf
echo ""

echo "=== 4. Clash 连接测试 ==="
if curl -s -o /dev/null -w "%{http_code}" --proxy http://127.0.0.1:7897 https://www.google.com 2>/dev/null | grep -q "200"; then
    echo "✅ Clash 代理连接正常"
else
    echo "⚠️  Clash 代理连接失败，请确认 Clash 是否运行在端口 7897"
fi
echo ""

echo "=== 5. Git 协议转换测试 ==="
echo "测试命令: git clone git://github.com/git/git.git"
echo "实际会转换为: https://github.com/git/git.git"
echo ""

echo "==================== 配置完成 ===================="
echo "💡 当前 shell 已配置代理，退出后失效"
echo "   如需重新配置，请重新执行此脚本"
```



```bash
wbh808@wbh-robot:~/yocto-rockchip-sdk/build/conf$ cat local.conf
#include include/common.conf
#include include/demo.conf

MACHINE = "rockchip-rk3588s-rock-5c"
BBMASK += ".*/recipes-browser/chromium"
wbh808@wbh-robot:~/yocto-rockchip-sdk/build/conf$ 
```
- 以上对于这一个文件的操作实际产生的影响以及含义不是很清楚

```bash
# 如果你确认 include/common.conf 和 include/demo.conf 不存在或不需要，可以保持注释
# 如果它们存在且包含必要配置（如镜像、分发版），请取消注释
include include/common.conf
include include/demo.conf

# 指定目标机器（Rockchip RK3588S 基于 Radxa ROCK 5C）
MACHINE = "rockchip-rk3588s-rock-5c"

# 屏蔽 Chromium 浏览器相关的所有 recipe（避免下载数 GB 源码导致构建失败）
#BBMASK += ".*/recipes-browser/chromium"

# ========== 以下为新增的推荐配置（解决网络问题） ==========
# 使用国内镜像加速源码下载（以清华源为例，可根据实际选择）
PREMIRRORS_prepend = "\
    git://.*/.* http://mirrors.tuna.tsinghua.edu.cn/git/yocto/ \n \
    https://.*/.* http://mirrors.tuna.tsinghua.edu.cn/yoctoproject/ \n \
    ftp://.*/.* http://mirrors.tuna.tsinghua.edu.cn/yoctoproject/ \n \
"

# 启用自己的镜像机制
INHERIT += "own-mirrors"

# 设置镜像主 URL
SOURCE_MIRROR_URL = "http://mirrors.tuna.tsinghua.edu/yoctoproject/"

# 提高 wget 重试次数和超时时间（应对网络不稳定）
FETCHCMD_wget = "/usr/bin/env wget -t 10 -T 180"

# 限制并发任务数，避免过多网络连接同时失败（可选）
#BB_NUMBER_THREADS = "4"
#PARALLEL_MAKE = "-j 4"
```
- 使用通配替换首先使用国内镜像源

- 配置文件路径如下：
```bash
wbh808@wbh-robot:~/yocto-rockchip-sdk/build/conf$ cat local.conf
#include include/common.conf
#include include/demo.conf

MACHINE = "rockchip-rk3588s-rock-5c"
BBMASK += ".*/recipes-browser/chromium"
```




## 代理与实际软件源原理


```text
[Yocto构建] → 决定“请求哪个URL” (镜像地址 vs 默认地址)
     ↓
[系统网络] → 决定“该URL的流量走哪条出口” (代理 vs 直连)
     ↓
[最终速度] = URL响应速度 + 出口路径质量
```


### 🔗 三者如何协同？一个具体例子说明

假设你在 Yocto 中编译一个需要从 `github.com` 下载源码的包：

|配置情况|Yocto 请求的 URL|Clash 规则对 `github.com` 的策略|实际出口|最终速度|
|---|---|---|---|---|
|无镜像 + 无代理|`https://github.com/...`|无代理（直连）|本地直连 github|慢（跨国）|
|无镜像 + 开启代理（规则模式）|`https://github.com/...`|匹配到 `GEOIP,CN` 不包含，走 `PROXY`|代理服务器|取决于代理质量|
|**配置清华源镜像** + 无代理|`https://mirrors.tuna.tsinghua.edu.cn/...`|无代理（直连）|本地直连清华|**快** ✅|
|**配置清华源镜像** + 开启代理（规则模式）|`https://mirrors.tuna.tsinghua.edu.cn/...`|匹配到 `DOMAIN-SUFFIX,tuna.tsinghua.edu.cn,DIRECT`|本地直连清华|**快** ✅ (且未消耗代理流量)|
|**配置清华源镜像** + 开启代理（全局模式）|`https://mirrors.tuna.tsinghua.edu.cn/...`|无例外，全部 `PROXY`|代理访问清华|慢（绕路） ❌|






## 较优代理+换源方案

- 常用的包管理工具换为清华源
- 特殊的构建工具内部方案（其实也是更换包管理工具的软件源）换为清华源
- 将使用ssh协议的内容更换为使用https协议

- 接着就可以使用clash配置全局代理
- 并且设置清华源相关的http协议除外



## linux网络配置相关拓展

```bash
export NO_PROXY=localhost,127.0.0.1,::1
```
- **`127.0.0.1`**：IPv4 回环地址，本机通信
- **`::1`**：IPv6 回环地址，是 `127.0.0.1` 的 IPv6 版本
- 强调本机程序访问本机的各个端口不走代理

```bash
# 没有 no_proxy 时的悲剧
curl http://localhost:3000/api
# → 流量被转发到 Clash:7897
# → Clash 再去外网找 localhost（不存在）
# → 连接失败 ❌

# 有 no_proxy 时
curl http://localhost:3000/api  
# → 检查发现 localhost 在 no_proxy 列表中
# → 直接连接本机 3000 端口
# → 成功访问本地服务 ✅
```


```bash
# ~/.bashrc 示例内容
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
export PATH=$PATH:/home/user/my_scripts
alias ll='ls -al'

# 当你打开新终端时，bash 自动执行这个文件中的命令
# 所以这些环境变量和别名就自动生效了
```
**典型程序对环境变量的支持：**
- **curl/wget**：读取 `http_proxy`、`https_proxy`、`no_proxy`
- **git**：读取 `http.proxy` 配置，但也支持 `http_proxy` 环境变量
- **pip/apt**：也支持这些标准变量
- **容器工具**：docker pull 支持 `HTTP_PROXY` 变量


```bash
# 场景演示
# 1. .bashrc 中有：export http_proxy=http://123.456.7.8:9999

# 2. 打开终端，变量自动加载
echo $http_proxy
# 输出：http://123.456.7.8:9999

# 3. 执行 unset
unset http_proxy
echo $http_proxy
# 输出：（空）

# 4. 但 .bashrc 文件内容没变
cat ~/.bashrc | grep http_proxy
# 输出：export http_proxy=http://123.456.7.8:9999 （仍然存在）

# 5. 开新终端，又会自动加载旧代理（第2步的状态）
```
- `unset`只影响当前`shell`配置



```bash
# 创建函数或别名
alias proxy='export http_proxy=http://127.0.0.1:7897; export https_proxy=http://127.0.0.1:7897'
alias unproxy='unset http_proxy https_proxy'

# 使用时
proxy
curl google.com  # 走代理
unproxy
curl localhost    # 直连
```
- 使用代理：原本`https`访问对应服务器的443端口
- 走代理之后：向`127.0.0.1:7897`传请求，从这个端口接收数据
- 当宿主机传请求后，`127.0.0.1:7897`的代理软件会选择更优线路作为传话筒


```bash
# 查看当前代理
echo $http_proxy $https_proxy

# 测试代理是否生效
curl -I https://www.google.com 2>&1 | grep -i proxy
# 或直接看输出内容

# 测试 no_proxy
curl -v http://localhost:8080 2>&1 | grep -i "no proxy"
```
- `curl -I`访问网页，只获取响应头，从中筛选`grep`需要的内容


```bash
# 执行前，查看全局配置
git config --global --list | grep proxy
http.proxy=http://127.0.0.1:7890  # 旧的代理配置
https.proxy=http://127.0.0.1:7890

# 执行删除命令
git config --global --unset http.proxy
git config --global --unset https.proxy

# 执行后，再次查看
git config --global --list | grep proxy
（没有输出，已被删除）
```
- git具有单独的环境变量

环境变量 http_proxy  <  Git 配置 http.proxy  <  命令行参数
（最低优先级）        （中等优先级）          （最高优先级）

一般情况下，越是精确局部的配置，优先级越高




# 📅2026-06-16

## 新建Github分支

```bash
# 1. 根据当前分支，创建并切换新分支
git switch -c login-page
# 2. 修改代码后暂存提交
git add .
git commit -m "完成登录页面基础结构"
# 3. 推送到远程仓库，建立关联
git push -u origin login-page
```


- 两分支快照完全一致：存在已追踪未提交改动也可直接切换，本地改动保留，无报错；
- 两分支快照不同，且本地改动文件在目标分支存在不同版本 / 目标分支无该文件：普通切换报错终止；`-f` 强制切换会覆盖丢失本地已追踪文件改动，未追踪文件不受影响；
- 仅存在未追踪新文件 /.gitignore 忽略文件：任何分支切换都不会拦截，文件永久保留在工作区。