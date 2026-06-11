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