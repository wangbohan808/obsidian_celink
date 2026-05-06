# DIY家用智能摄像头项目目标

## 一、项目核心产出

使用ROCK-5C开发板、结合一款可输出YUV文件的摄像头，完成一款可直接家用的DIY版智能摄像头，功能与小米、360、萤石等家用摄像头一致，且在隐私性、可扩展性上更具优势，非课程实验或demo，是能解决实际家用需求的成品设备。

## 二、核心功能目标（可直接落地使用）

1. **实时画面查看**：通过电脑/手机VLC播放器输入指定地址，即可查看1080P清晰实时画面，延迟低（几百毫秒），流畅不卡顿。
    
2. **移动侦测自动录像**：无人时待机，有人、宠物移动时自动启动录像，移动物体离开后自动停止，节省存储空间。
    
3. **视频自动管理**：视频按5分钟/段自动存储，TF卡存满后自动覆盖最早视频，无需手动操作，可直接查找对应日期视频回看。
    
4. **夜间夜视功能**：搭配红外摄像头，夜晚关灯后画面自动切换为黑白模式，无需开灯也能清晰查看。
    
5. **状态可视化提示**：通过开发板LED灯直观判断设备状态——常亮（正在录像）、慢闪（待机监控）、快闪（异常/断连）。
    
6. **隐私安全保障**：所有视频仅存储在自身开发板，不上传任何第三方服务器，不联网也可使用，隐私绝对安全。
    
7. **长期稳定运行**：支持插电长期运行（可连续运行数天、数月），具备看门狗功能，死机后可自动重启，达到成品设备稳定性。
    
8. **可扩展性**：支持后期扩展功能，包括但不限于：加麦克风实现监听、加喇叭实现远程喊话、加按键实现手动录像、加屏幕实现实时画面显示。
    

## 三、技术目标（适配Linux应用/驱动岗需求）

通过项目掌握Linux应用/中间件岗核心必考技能，同时积累驱动交互经验，为后续转驱动岗位奠定基础，具体需掌握：

- V4L2摄像头操作（mmap零拷贝）
    
- RKMPP硬件编码（H.265）
    
- Linux多线程、队列、锁的应用
    
- 内存管理、物理内存ION相关操作
    
- 设备文件（/sys /dev）硬件操作
    
- 网络推流相关技术
    
- 磁盘管理、文件操作
    
- 异常处理、系统监控
    
- 内核日志查看、问题调试能力
    

## 四、项目价值

做出一款能实际用于看家、看宠物、看老人的硬件设备，兼顾实用性与技术深度，助力快速适配Linux应用/中间件岗位，实现向驱动岗位的平滑过渡，提升就业竞争力。

## 五、后续落地支持

提供从第一步到最后一步的完整实现路线，按顺序操作即可，确保半年内可完成成品制作。





# 基于ROCK 5C的本地智能摄像头项目方案（瑞芯微Linux系统岗面试导向）

## 一、项目定位（面试必背，可直接写进简历）

**项目名称：基于 RK3588 的本地智能摄像头（Linux 嵌入式实战项目）**

- 硬件平台：Radxa ROCK 5C（RK3588）
    
- 系统环境：Ubuntu / 瑞莎官方 Debian 镜像
    
- 实现功能：
    
    - V4L2 摄像头采集 + 实时预览
        
    - RKMPP 硬件 H.264/H.265 编码
        
    - 本地循环录像、移动侦测、TF 卡自动管理
        
    - RTSP/HTTP 局域网实时推流，手机/电脑播放
        
    - 系统看门狗、异常重启、状态 LED 指示
        
- 技术栈：C/C++、Linux 系统编程、多线程、V4L2、MMAP、RKMPP、设备树/sysfs 控制、网络编程、Shell 脚本、系统调试。
    
- 亮点：纯自研、不上云、全本地运行、深度贴合瑞芯微平台。
    

## 二、项目整体实施路线（半年节奏，零基础可落地）

### 阶段 0：环境与基础（1～2 周）—— 必做前置准备

目标：实现ROCK 5C正常使用，搭建基础开发环境，避免环境问题阻碍进度。

1. 系统烧录与基础配置
    
    1. 镜像：采用瑞莎官方 Debian/Ubuntu 镜像
        
    2. 烧录工具：Etcher 或 Rufus
        
    3. 基础操作：开机、修改密码、更换软件源、更新系统
        
    4. 网络配置：开启 SSH、固定局域网 IP，方便电脑远程操作
        
2. Linux 基础命令（每日30分钟，必掌握）
    
    1. 文件操作：ls、cd、cp、mv、rm、mkdir、chmod、chown
        
    2. 进程管理：ps、top、htop、kill
        
    3. 网络操作：ifconfig/ip addr、ping、ssh、scp
        
    4. 日志查看：dmesg、journalctl、/var/log
        
3. 编译环境安装 `sudo apt update` `sudo apt install gcc g++ make cmake git build-essential`验证标准：编写简单 hello.c 文件，完成编译并成功运行。
    
4. 确定开发方式
    
    1. 远程连接：Windows/Mac 通过 SSH 连接 ROCK 5C
        
    2. 代码编辑：推荐使用 VSCode 远程 SSH 插件（高效便捷）
        

### 阶段 1：摄像头基础 —— V4L2 采集（2～3 周）

目标：实现摄像头图像采集，完成项目核心入口功能。

1. 摄像头选型（新手避坑）
    
    1. 入门首选：USB 摄像头（1080P，操作最简单）
        
    2. 进阶选择：CSI 摄像头（如 OV13850，贴合嵌入式实战场景）
        
2. 验证摄像头识别 `ls /dev/video*` `v4l2-ctl --list-devices` `v4l2-ctl --list-formats-ext -d /dev/video0`验证标准：能正常查看摄像头支持的分辨率、图像格式（如 YUYV、MJPG）。
    
3. 掌握 V4L2 核心流程（面试必考，需熟记）
    
    1. 打开摄像头设备（/dev/video0）
        
    2. 设置图像格式、宽度、高度、帧率
        
    3. 申请缓冲区，并通过 mmap 映射到用户空间（零拷贝核心）
        
    4. 缓冲区入队 → 启动流采集 → 循环出队取帧 → 入队（持续采集）
        
    5. 停止流采集 → 关闭设备
        
4. 编写第一个实战程序：v4l2_capture.c
    
    1. 功能：打开摄像头 → 采集一帧图像 → 保存为 yuv 文件
        
    2. 验证：使用 YUView 工具在电脑上打开 yuv 文件，确认图像正常。
        

阶段目标达成：掌握 Linux 设备操作 + MMAP 零拷贝核心技术。

### 阶段 2：瑞芯微核心 —— RKMPP 硬件编码（3～4 周）

目标：实现图像硬件编码，贴合瑞芯微平台核心技术，提升项目竞争力（面试重点）。

1. 安装瑞芯微 MPP 库
    
    1. 说明：瑞莎官方镜像一般自带，若未自带则进行源码编译
        
    2. 源码下载：rockchip-linux/mpp（GitHub 仓库）
        
    3. 编译流程：cmake → make → make install
        
    4. 验证：运行 mpp_info 命令，确认库安装成功。
        
2. 理解 MPP 核心作用
    
    1. 功能：将 V4L2 采集的 YUV 图像，通过硬件编码转为 H.264/H.265 码流
        
    2. 优势：硬件编码，CPU 占用率极低，贴合实际产品需求。
        
3. 串联 V4L2 与 MPP 流程
    
    1. 通过 V4L2 采集一帧 YUV 图像
        
    2. 将 YUV 图像送入 MPP 编码器
        
    3. 获取 MPP 编码后的 H.264 帧
        
    4. 将编码帧保存为 .264 文件
        
4. 实现标准视频文件输出
    
    1. 添加文件头，将 .264 文件合成为 .mkv 或 .h264 标准格式
        
    2. 验证：使用 VLC 播放器打开视频文件，确认播放正常。
        

阶段目标达成：掌握瑞芯微平台核心中间件 RKMPP，形成面试核心亮点。

### 阶段 3：本地录像 + 文件管理（2 周）

目标：实现实用化录像功能，体现工程能力和产品思维。

1. 按时间切片存储视频
    
    1. 切片规则：每 5 分钟生成一个视频文件
        
    2. 文件名规范：video_年月日_时分秒.h264（如 video_20260331_235959.h264）
        
2. 实现 TF 卡自动循环覆盖
    
    1. 通过 Shell 脚本或 C 程序，实时判断 TF 卡使用率
        
    2. 当使用率超过阈值（如 90%），自动删除最早的视频文件，无需手动干预。
        
3. 实现入门版移动侦测
    
    1. 核心方法：帧差法（对比前后两帧图像的像素差异）
        
    2. 逻辑：差异值超过设定阈值 → 启动录像；画面静止 → 停止录像。
        

阶段目标达成：具备基础产品化思维，实现“能用、好用”的录像功能。

### 阶段 4：局域网实时推流（2 周）

目标：实现手机/电脑远程实时查看，完善项目实用价值。

1. 选择最简推流方案
    
    1. 推荐工具：live555 或 easyRTSP（上手简单，适配嵌入式场景）
        
    2. 核心逻辑：将 MPP 输出的 H.265 码流，推送至 RTSP Server
        
2. 生成推流地址 `rtsp://192.168.1.xxx:8554/stream`说明：192.168.1.xxx 为 ROCK 5C 的固定局域网 IP。
    
3. 验证推流功能
    
    1. 在手机/电脑上打开 VLC 播放器
        
    2. 输入推流地址，确认能实时播放摄像头画面，延迟控制在几百毫秒。
        

阶段目标达成：掌握 Linux 网络编程、流媒体协议（RTSP），完善项目远程访问功能。

### 阶段 5：产品化、稳定化（2～3 周）

目标：让设备达到商用级稳定性，贴合实际产品标准，提升面试竞争力。

1. LED 状态指示功能
    
    1. 控制方式：通过 /sys/class/leds 操作硬件 LED
        
    2. 状态定义：常亮（正在录像）、慢闪（待机监控）、快闪（设备异常/断连）
        
2. 看门狗（Watchdog）配置
    
    1. 操作方式：打开 /dev/watchdog 设备文件
        
    2. 核心逻辑：定时向看门狗发送“喂狗”指令；程序卡死时，看门狗自动重启设备。
        
3. 实现开机自启动
    
    1. 方法：编写 systemd 服务脚本
        
    2. 效果：设备开机后，摄像头程序自动运行，无需手动启动。
        
4. 搭建日志系统
    
    1. 功能：将程序运行日志、错误信息输出到指定文件
        
    2. 作用：方便后续调试、定位设备异常问题，体现工程严谨性。
        

阶段目标达成：设备具备商用级稳定性，从“demo 级”提升为“产品级”。

### 阶段 6：扩展升级（可选，冲大厂/高薪加分项）

- 添加红外夜视功能（搭配红外 USB/CSI 摄像头）
    
- 实现音频采集与播放（双向对讲，添加麦克风、喇叭）
    
- 搭建 Web 管理页面（使用 boa / lighttpd 服务器，实现参数配置）
    
- 编写配置文件，支持分辨率、帧率、录像时长等参数可自定义修改。
    

## 三、当前可立即执行的3个步骤（今日可启动）

1. 通过 SSH 连接 ROCK 5C，并完成局域网 IP 固定
    
2. 插入 USB 摄像头，使用 v4l2-ctl 命令验证设备识别成功
    
3. 编译运行最简单的 V4L2 采集示例，保存一帧 YUV 图像并验证。
    

## 四、项目与瑞芯微 Linux 系统岗的适配性（面试核心优势）

### 1. 核心技能匹配（岗位必考）

- Linux 应用编程全套（文件、进程、线程、锁、IPC）
    
- V4L2 视频子系统（嵌入式视频开发核心）
    
- 瑞芯微 RKMPP 硬件编解码（平台核心技术，竞品难以替代）
    
- 设备驱动访问（/dev 设备操作、mmap 零拷贝）
    
- 硬件控制（sysfs、看门狗、LED）
    
- 系统调试、日志分析、性能优化、稳定性保障
    
- 完整项目工程化思维（从需求到成品的全流程实现）
    

### 2. 瑞芯微面试高频问题（项目可直接覆盖）

- 你做过瑞芯微平台相关项目吗？（项目基于 RK3588，直接贴合）
    
- 用过瑞芯微 MPP 库吗？具体怎么实现编码？（阶段 2 完整覆盖）
    
- V4L2 采集的核心流程是什么？（阶段 1 重点掌握，可流畅背诵）
    
- 如何实现 Linux 下的零拷贝？（MPP + MMAP 组合实现，有实战代码）
    
- 多线程同步怎么实现？（项目中采集、编码、推流多线程协作，有实战经验）
    
- 如何保证嵌入式设备长时间稳定运行？（看门狗、日志、自启动，全流程覆盖）
    

## 五、项目辅助支持（全程可落地）

按阶段提供以下支持，确保零基础可跟随执行：

1. 每阶段详细操作命令（可直接复制执行）
    
2. 每阶段可直接编译运行的实战代码
    
3. 代码逐行讲解（拆解核心逻辑，补齐知识缺口）
    
4. 报错解决方案（针对 ROCK 5C 平台常见问题，直接给出排查方法）
    
5. 简历项目描述优化 + 面试高频问题标准答案整理




# 探索


echo $PATH
- echo：打印、输出后面的内容到屏幕上
- PATH：Linux系统环境变量，输入命令后，会从其中的目录依次查找可执行程序
```bash
radxa@rock-5c:/$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```


which ls
- which：查找命令的可执行文件的路径
```bash
radxa@rock-5c:/$ which ls
/usr/bin/ls
```

dpkg -l | grep net-tools
- dpkg -l ：查询所有已安装的包（每一个包包含五列，进行输出）
- grep net-tools：输入一个文本，输出包含关键字的对应行

export PATH=原有路径:新路径1:新路径2
- 添加export，使得这一个修改对整个终端环境有效，不添加只对这一个命令有效
```bash
radxa@rock-5c:/$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
radxa@rock-5c:/$ export PATH=$PATH:/sbin/:/usr/bin
radxa@rock-5c:/$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/sbin/:/usr/bin
```



## 查询当前卡发版 网络相关概念

> [!NOTE] 网络的基础概念
> MAC(Media Access Control)：每块网卡出场烧死，不随网络IP变化；作用是在局域网内找到具体的设备
> 网卡：插网线/连WiFi的硬件接口
> IP地址：网络中定位设备、发送数据、可以变化
> 网关：访问外部世界、连接互联网的数据出口，一般是路由器的IP；没有网关，只能在同局域网的设备聊天，不能上网
> DNS：将域名翻译为IP地址
> 路由：去外网->走网卡、网关；同一个局域网内的，走局域网路由



输出当下设备的路由表：
```bash
radxa@rock-5c:/$ ip route show
```

```bash
default via 192.168.70.254 dev wlan0 proto static metric 600 
192.168.70.0/24 dev wlan0 proto kernel scope link src 192.168.70.108 metric 600
```
- 去外网 → 走网关 192.168.70.254 → WiFi 出口
- dev wlan0：走WiFi网卡
- proto static:网关是管理员静态写死的


- 192.168.70.0/24  ： 目标网段，IPV4一共有32位，`/24`代表前24位（前三段）是固定的，最后一段（8位，0~255）是可变的，0 是网段，255 是广播，都不能用
- dev wlan0：还是走 WiFi网卡
- scope link：局域网路由，不需要经过网关，直接通信
- src 192.168.70.108：开发板自己的 IP 地址


查看本地所有网卡的所有信息：
```bash
ip addr show
```


```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 inet 127.0.0.1/8 scope host lo
```
- **lo** = loopback（本地回环）:虚拟网卡，**自己跟自己通信**用的
- 访问 `127.0.0.1` = 访问开发板自己

```bash
2: end1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000 link/ether 42:a9:86:c3:dc:eb brd ff:ff:ff:ff:ff:ff
```
- end1:有线网卡（类似 eth0）
- NO-CARRIER：没有插网线
- state DOWN：物理层没连接，**网卡没工作**

```bash
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000 link/ether 88:00:33:77:a6:d2 brd ff:ff:ff:ff:ff:ff inet 192.168.70.108/24 brd 192.168.70.255 scope global noprefixroute wlan0
```
- wlan0：WiFi 无线网卡
- <UP,LOWER_UP>：软件已开启、WiFi 已连接成功
- link/ether 88:00:33:77:a6:d2：WiFi 网卡的 **MAC 地址**（硬件身份证）
- inet 192.168.70.108/24：本机 IP 地址
- brd 192.168.70.255：广播地址


## 串口连接wifi

```bash
radxa@rock-5c:~$ sudo nmcli device wifi connect "ce-link-guest" password "RD.saodiji98"
```

## 给开发板的某一个WiFi设置静态IP：

- 需要使用AI整理格式再复制
```bash
sudo nmcli connection modify "ce-link-guest" \ ipv4.method manual \ ipv4.addresses 192.168.70.108/24 \ ipv4.gateway 192.168.70.254 \ ipv4.dns "223.5.5.5,114.114.114.114"
```
- 重新激活连接生效（指令在串口执行，使用ssh会断开连接），不执行当下还是原本的IP地址：
```bash
sudo nmcli connection up "ce-link-guest"
```



屏幕分辨率：物理的屏幕中小灯泡的数量
视频分辨率：传输原始视频数据的像素点的数量分布；2K 视频在 1080p 屏幕上：会被缩小，无法用到全部 2K 像素，画质上限由屏幕决定；480p 视频在 1080p 屏幕上：会被拉伸，**多个屏幕物理像素共同显示同一个视频像素**，所以画面模糊



# 探索流程

## 获取一帧数据

- 书写一个文件获取一帧数据，前面的初始化（打开摄像头等配置操作）可以正常完成，但是最后获取图像、打印图像格式的步骤却始终无法成功--因为自己书写的程序打印不够完善，无法定位到最终的问题在哪里


### 失败反思

- 自己书写的程序打印不够完善，无法定位到最终的问题在哪里

**可能原因：**

1. 另一个进程可能正在使用代码中调用的摄像头设备
2. 设置的分辨率，设备可能不支持
3. 缺少V4L2_CAP_STREAMING检查，可能使得程序躲过了报错打印，成功通过初始化检查
4. 可能是当下的错误是正常的数据无帧，因而会循环等待，而不是打印报错
```c
        if (ioctl(fd, VIDIOC_DQBUF, &buf) < 0) {
            if (errno == EAGAIN) continue; // 无数据，重试
            perror("出队失败");
            break;
        }
```


> [!NOTE] continue语法的使用
> - 作用：跳过本次循环剩余语句，直接进入下一次循环。- 范围：只影响当前最内层循环（for/while/do-while），不跳出循环。
> - 区别于 break：break 直接结束整个循环；continue 只是跳过当前这次，继续下一次


### 问题的排查与解决

#### 排查进程占用

##### 摄像头编号发生变化

解决：给摄像头创建一个固定的别名

1. 管道输出摄像头的唯一ID
```bash
radxa@rock-5c:~$ udevadm info --name=/dev/video0 | grep -E "ID_VENDOR_ID|ID_MODEL_ID|ID_SERIAL_SHORT"

E: ID_MODEL_ID=5858
E: ID_SERIAL_SHORT=200901010001
E: ID_VENDOR_ID=0bda
```
- 厂商 ID：`0bda`
- 型号 ID：`5858`
- 序列号：`200901010001`

2. 向文件中写入，创建永久绑定规则
```bash
sudo tee /etc/udev/rules.d/99-usb-camera-fixed.rules <<-'EOF' SUBSYSTEM=="video4linux", ENV{ID_VENDOR_ID}=="0bda", ENV{ID_MODEL_ID}=="5858", ENV{ID_SERIAL_SHORT}=="200901010001", SYMLINK+="JR_camera", MODE="0666" 
EOF
```

行1：sudo tee ... <<-'EOF' 
行2：规则内容 
行3：EOF

3. 重载规则
```bash
sudo udevadm control --reload-rules && sudo udevadm trigger
```

4. 真实设备是黄色，软链接是天蓝色
```bash
radxa@rock-5c:~$ ls /dev/video*
/dev/video0  /dev/video1  /dev/video-dec0  /dev/video-enc0
radxa@rock-5c:~$ ls /dev/my_camera
/dev/my_camera
```





# AI应用配置API

**各个AI模型与服务的厂商都有对应的API文档，讲解如何给AI应用配置对应的API**

## Claude配置LongCat

```json
{
	 "env": { 
	 "ANTHROPIC_AUTH_TOKEN": "your_longcat_api_key", "ANTHROPIC_BASE_URL": "https://api.longcat.chat/anthropic", "ANTHROPIC_MODEL": "LongCat-Flash-Chat", "ANTHROPIC_SMALL_FAST_MODEL": "LongCat-Flash-Chat", "ANTHROPIC_DEFAULT_SONNET_MODEL": "LongCat-Flash-Chat", "ANTHROPIC_DEFAULT_OPUS_MODEL": "LongCat-Flash-Chat", "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "60000", "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1 
	 }, 
	 "permissions": { 
	 "allow": [], 
	 "deny": [] 
	 } 
}
```

- 第一行`{`： JSON对象的起始标志
- 内部有`env` 和 `permissions` 两个顶级配置项

- API代理地址：指定使用哪一个AI服务器处理请求
- 认证令牌：使用AI的消费额度




# 新的代码模块

## 一、文件命名

头文件（.h）：全小写 + 下划线，如 camera.h、v4l2_camera.h、ipc_camera.h

源文件（.c）：与头文件同名，全小写 + 下划线，如 v4l2_camera.c、rkmpp_encoder.c、main.c

## 二、变量命名

全局变量：g_前缀 + 小写下划线，如 g_camera_ctx、g_device_state

静态变量：s_前缀 + 小写下划线，如 s_buffer_count、s_initialized

局部变量：小写下划线，如 frame_buffer、ret、fd

常量 / 宏定义：全大写 + 下划线，如 MAX_FRAME_WIDTH、CAMERA_BUFFER_COUNT

布尔变量：is_/has_/enabled_前缀，如 is_capturing、has_motion、watchdog_enabled

指针 / 计数：指针可用 _ptr 后缀，计数用 _count/_size/_len，如 frame_ptr、buffer_count、frame_size

## 三、函数命名

模块接口：模块名_功能名，如 camera_init、camera_start_capture、encoder_encode_frame

通用函数：初始化 xxx_init、清理 xxx_cleanup、启动 xxx_start、停止 xxx_stop、获取 xxx_get_xxx、设置 xxx_set_xxx

线程函数：模块名_thread_func，如 camera_thread_func、encoder_thread_func

内部静态函数：static 修饰 + 小写下划线，如 static int init_buffers ()

## 四、结构体与类型命名

结构体类型：小写下划线 +_t 后缀，如 camera_context_t、camera_config_t、device_state_t

结构体成员：小写下划线，如 width、height、fd、is_initialized、frame_count

回调函数类型：功能名_callback_t，如 frame_callback_t、motion_detected_callback_t

## 五、同步对象命名

互斥锁：xxx_mutex，如 frame_mutex、config_mutex

条件变量：xxx_cond，如 data_ready_cond、buffer_empty_cond

## 六、目录与模块命名

目录名：功能语义化、全小写，如 src/camera、src/encoder、src/streaming、include、utils

## 七、枚举与错误码命名

枚举值：全大写 + 下划线，如 DEVICE_STATE_IDLE、DEVICE_STATE_RECORDING

操作命令：模块_CMD_功能，如 CAMERA_CMD_START、CAMERA_CMD_SET_EXPOSURE

错误码：模块_错误类型，如 CAMERA_ERROR_NONE、CAMERA_ERROR_DEVICE、ENCODER_ERROR_INIT



# 📅2026-05-02

## 开发板的供电规格





