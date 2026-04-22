学习日期：2026-04-21 星期二
主题关键字：
关键待办：

---

###### 将发行版安装到指定目录

1. 在指定位置建立文件夹
```bash
E:\WSL\Images
E:\WSL\Ubuntu1804
```

2. 切换到指定目录使用指令下载镜像
```bash
# 切换到镜像目录
cd E:\WSL\Images
# 下载官方Ubuntu18.04安装包
curl.exe -L -o Ubuntu1804.appx https://aka.ms/wsl-ubuntu-1804
```


```bash
Rename-Item E:\WSL\Images\ubuntu1804.appx ubuntu1804.zip
Expand-Archive E:\WSL\Images\ubuntu1804.zip E:\WSL\Images\ubuntu1804
```

```powershell
wsl --import Ubuntu-18.04 E:\WSL\Ubuntu1804 E:\WSL\Images\ubuntu1804\install.tar.gz --version 2
```

- `Ubuntu-18.04` = 你给系统起的名字
- `E:\WSL\Ubuntu1804` = **安装到 E 盘这个位置**
- `...\install.tar.gz` = 你刚才解压出来的系统文件
- `--version 2` = 强制使用 WSL2





**指令快速下载**

```bash
# 安装 Ubuntu 22.04 到 E:\WSL\Ubuntu2204
wsl --install -d Ubuntu-22.04 --location E:\WSL\Ubuntu2204 --web-download

# 安装 Ubuntu 20.04
wsl --install -d Ubuntu-20.04 --location E:\WSL\Ubuntu2004 --web-download

# 安装 Ubuntu 24.04
wsl --install -d Ubuntu-24.04 --location E:\WSL\Ubuntu2404 --web-download
```



###### 仅关闭指定的版本

```bash
wsl -t Ubuntu-22.04
```


###### 打开指定的wsl版本

```bash
wsl -l -v

wsl -d Ubuntu-18.04
```




###### 切换登陆的用户

```bash
# 新建用户（把 username 换成你想要的名字，如 linux）
adduser username

# 加入 sudo 组（能执行管理员命令）
usermod -aG sudo username
```

```bash
# 切换到普通用户（把 username 换成你的用户名）
su - username
```