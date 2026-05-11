# Git的安装步骤
1. 官网下载最新版本的Git
2. 无脑默认下一步，完成安装
3. 配置git：
```bash
wangb@wangbohan MINGW64 /d/Github_Code
$ git config --global user.name "wangbohan"

wangb@wangbohan MINGW64 /d/Github_Code
$ git config --global user.email "wangbohan808@163.com"
```

4. 配置密钥，与github链接
```bash
ssh-keygen -t rsa -b 4096 -C "wangbohan808@163.com"
```

5. 把生成的.pub文件内容复制到github
6. 启动ssh（启用守护进程，缓存解密密钥）、添加密钥（用于认证前面所有的ssh认证）、验证是否成功、验证github连接
```bash
wangb@wangbohan MINGW64 /d/Github_Code
$ eval "$(ssh-agent -s)"
Agent pid 651

wangb@wangbohan MINGW64 /d/Github_Code
$ ssh-add /c/Users/wangb/.ssh/github_wangbohan808
Identity added: /c/Users/wangb/.ssh/github_wangbohan808 (wangbohan808@163.com)

wangb@wangbohan MINGW64 /d/Github_Code
$ ssh-add -l
4096 SHA256:dr6Z9jlMJssROPdEpA8aUZWov+N1uG1Aj21kLotqEUE wangbohan808@163.com (RSA)

wangb@wangbohan MINGW64 /d/Github_Code
$ ssh -T git@github.com
Hi wangbohan808! You've successfully authenticated, but GitHub does not provide shell access.
```
# 即使使用SOURCETREE操作也一定要挂梯子


> [!error] VPN
> 但是有些软件只有关闭VPN才能使用

# 将电脑上的代码推送到github仓库步骤

- 需要首先安装git：GIT可以使用GUI操作，也可以在使用GIT BASH命令行操作

- 在本地配置git用户信息：
```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

- 进入想要上传的电脑上的代码仓库（这里相当于工作仓库），使用git bash命令行打开
- 在项目目录创建隐藏的.git文件夹（对原本的电脑仓库进行一定的修饰，相当于创建本地仓库）：
```bash
git init
```

.git文件夹存在的意义是什么？

- 查看当前的文件状态（主要显示工作区没有添加到暂存区的文件）：
```bash
git status
```

- 将文件添加到暂存区：
```bash
git add .
```
（.代表添加所有文件，也可以使用git add filename添加指定文件）

- 提交文件到本地仓库：
```bash
git commit -m "描述这次提交"
```

- 在github上创建远程仓库
- 建立好需要刷新一下，防止推送不上去


- github相当于一个网盘，可以根据ssh的地址找到这一个网盘的位置
- 在本地终端添加github远程仓库
```bash
git remote add origin https://github.com/username/repository.git
```
- 验证远程仓库：
```bash
git remote -v
```

- 设置本地代码的默认分支的名称：
```bash
git branch -M main
```
- 推送代码，将本地的main分支与远程的origin分支关联
```bash
git push -u origin main
```

- 不同系统有不同的换行符标准
LINUX使用LF换行符标准，WINDOWS使用CRLF作为换行符
git有自动的换行符转化机制


```bash
$ git remote -v 
origin git@github.com:wangbohan808/RV31_dust_bag.git (fetch) 
origin git@github.com:wangbohan808/RV31_dust_bag.git (push)
```
- origin：远程仓库的默认名
- git@github.com:wangbohan808/RV31_dust_bag.git  :远程仓库的URL（网址），使用ssh协议


# 将github代码拉取到本地仓库

```shell
/* 第一次拉取仓库，直接创建完整文件夹 */
git clone <仓库URL>

/* 更新github的代码到本地仓库 */
git pull origin main
```



# github密钥配置

> [!error] 公钥私钥
> 1. 电脑使用邮箱创建一对公钥与私钥
> 2. 公钥让想要操作的仓库保存配置
> 3. 私钥设置电脑优先使用
> 4. 便可以使用这一个私钥操作（拉取代码、上传代码、修改仓库）配置公钥的仓库
- 远程仓库操作权限：本地电脑与远程仓库的公钥私钥是否配对
- 项目提交：首先要操作的目录与远程仓库对应，提交的账户邮箱是事先进行配置的，提交时会记录邮箱的名称

- 查看当前电脑的git全局配置
- 权限是本地的私钥与仓库的公钥是否匹配的问题

- 检查当前设备配备的密钥信息：
```bash
ls -al ~/.ssh
```

# 创建ssh密钥并进行配置

- 使用ed25519算法，指定生成密钥的路径和名称，并使用邮箱地址添加注释：
```bash
ssh-keygen -t ed25519 -f ~/.ssh/my_new_key -C "your_email@example.com"
```
- 设置正确的文件权限：
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/my_new_key     #私钥必须设置为600
chmod 644 ~/.ssh/my_new_key.pub  #公钥可以设置为644
```

- 使系统默认使用新创建的密钥
```text
Host github-WangBohanhhh
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes
```
> [!NOTE] 具体解释
> **`Host github-WangBohanhhh`**：用于替换git@github.com
> **`HostName github.com`**：指定目标连接的服务器是github；
> **`User git`**：Github要求所有使用git操作使用git用户名；
> **`IdentityFile ~/.ssh/id_rsa`**：明确指定用于认证的私钥路经；
> **`IdentitiesOnly yes`**：强制SSH客户端使用指定密钥；
- **地址解析​**​：`github-WangBohanhhh:WangBohanhhh/repo.git`会被转换为 `git@github.com:WangBohanhhh/repo.git`（依赖 `HostName`配置）

- 为config文件设置正确的权限：
```bash
chmod 644 ~/.ssh/config
```
- 通过命令测试配置是否生效：
```bash
ssh -T git@github.com
```


- 查看全局的用户名和邮箱：
```bash
git config --global user.name
git config --global user.email
```
- 标识提交者的身份信息

- 列出所有的全局配置项：
```bash
git config --global --list
```

# 处理冲突

原则：
1. 首先确保“本地仓库”与“最新的远程仓库”保持同步
2. 接下来把“工作区”修改到“本地仓库”
3. 接下来将“修改后的本地仓库”推送到“远程仓库”

- `git push -u origin main`
> [!error] 当我想要推送代码，远程仓库（origin）的 `main`分支上存在本地 `main`分支所没有的提交历史（commits），因而拒绝我的提交
> ! [rejected]        main -> main (fetch first)
error: failed to push some refs to '链接'
hint: Updates were rejected because the remote contains work that you do
hint: have locally. This is usually caused by another repository pushing to
hint: the same ref. If you want to integrate the remote changes, use
hint: 'git pull' before pushing again.




```bash
1. 拉取远程（pull）的更改，将其下载下来，合并到本地:
   git pull origin main --allow-unrelated-histories
拉取的时候，会将远程仓库相较于本地仓库不同的部分在本地仓库中标记出来；
接着需要在本地仓库中处理冲突
```

```bash
远程仓库与本地仓库不同的部分会使用特殊的符号标记出来：

<<<<<<< HEAD
# 这是你本地仓库的 README.md 内容
你的项目描述...
=======
# 这是远程仓库的 README.md 内容
wangbohan808/VZS-H1-Bastation
>>>>>>> 46d0ddb... (这里是远程提交的哈希值或注释)
```

```bash
2. 在编辑器中编辑文件，解决冲突标记：

- `<<<<<<< HEAD`到 `=======`之间的内容，是你**当前本地分支**​（`HEAD`）的版本。
    
- `=======`到 `>>>>>>> ...`之间的内容，是来自**远程 `origin/main`分支**的版本。
```

```bash
3. 保存修改好的冲突文件，告诉git已经解决了
   git add README.md
   上述操作将解决后的文件放入暂存区，准备提交
```

```bash
4. 冲突已经解决，创建一个合并提交完成整个pull操作
   git commit -m "解决了README的冲突文件"
```

```bash
5. 拉取远程仓库如果有冲突，未冲突的文件会放在暂存区；有冲突的文件需要额外解决，解决后加入暂存区；解决所有的之后，暂存区的文件就可以提交；提交完毕就可以推送：
   git push -u origin main
```

# 创建新的分支

```bash
1. 即使有很多分支，一个项目也只需要一个工作区
```

```bash
2. 确保当前在main分支，并且代码是最新的
   远程仓库切换到主分支：git checkout main
   从远程仓库拉去最新的代码：git pull origin main
```

```bash
3. 基于main分支创建一个独立开发的分支
   git checkout -b feature/intermittent-dust-collection
```

```bash
4. 确认当前工作的分支
   git status
   git branch
```

```bash
5. 本地仓库如果不是最新的，需要先拉取`解决冲突`；
本地仓库与远程仓库保持同步，执行以下操作：
	git add .
	git commit -m "feat:集尘功能修改为`集尘5s、停止2s，循环三次`"
```

```bash
6. 将"本地的新分支"推送到"远程仓库对应的新分支" （可以Tab键进行补充）
   git push -u origin feature/intermittent-dust-collection
```


# GITHUB常用指令

```bash
git branch：我的本地仓库有哪几个分支，我现在在哪一个分支上

git pull origin main：将远程仓库最新的代码同步到本地仓库
相当于：
git fetch：联系远程仓库`origin` --> 获得`main`分支的最新提交历史 --> 更新本地远程跟踪分支`origin/main`
git merge：将本地远程跟踪分支`origin/main`合并到当前所在的分支

回退版本：
git log --oneline：查看历史提交版本
git reset --hard <commit_hash>：精准回退到对应版本

git reset --hard HEAD：清除缓存区与工作区，方便更换分支

git reset：清空暂存区，保留本地仓库以及工作区

推送代码：
git push <远程仓库名> <本地分支名>:<远程分支名>

git checkout：切换`工作区`、`本地仓库的HEAD指针`到对应分支
```


# Pull Request（PR）

- `git pull`是协作开发者申请将代码合并到远程仓库
- `pull request`是项目拥有者同意将代码合并到远程仓库
- 只需要刚刚建立分支的时候初始化一下，后续的提交都是自动化的


- 选择`基础分支`和`比较分支`，PR描述的就是"比较分支"相较于”基础分支“的变化


# 多人协作提交代码

```bash
1. 首先将本地仓库远程分支更新到与远程仓库版本一致；为了更加干净处理这一操作，先将`工作区`与`暂存区`储存起来，防止干扰：
   git stash
```

```bash
2. 将`远程仓库与本地远程仓库对应的分支`拉取到`本地远程仓库`产生一个更新的版本（如果之前使用git commit指令但是没有使用git push，可能需要修改冲突）：
   git pull origin main
修改冲突后保证了本地仓库与工作区的版本相较于远程仓库更新，并且生成一个新的版本号（commit hash）
```

> [!error] 解决冲突
> 1. 可以理解为发生冲突后：`未冲突的部分`保存在暂存区的一块区域；`发生冲突的文件`保存在暂存区的另外一个部分；工作区解决冲突后可以将`解决完毕的发生冲突的的文件`添加回`未冲突的部分`；所有的文件解决完毕可以提交到本地仓库（及对应的远程分支）
> 2. 对应的指令就是`git add .`和`git commit -m "提交备注"`

```bash
3. 前面处理后，已经确保了本仓库与工作区全部是最新的版本，接下来将原本的工作区应用回来、处理冲突，就可以保证工作区是更新的版本，就可以安全推送到远程仓库：
   git stash pop
```
# 工作区代码恢复/切换分支

```bash
1.将暂存区的修改撤回到工作区：git reset HEAD .
2.丢弃工作区所有修改：git checkout .
```


# 本地切换远程分支

```bash
eval "$(ssh-agent -s)"
ssh-add /c/Users/wangb/.ssh/github_wangbohan808

git fetch origin            # 把远程所有分支信息拿回来
git switch 远程分支名        # 自动创建同名本地分支并跟踪
```

# GIT重复输入ssh指令问题

## 修改shell配置文件

```bash
# 创建或编辑 .bashrc文件
code ~/.bashrc

# 启动 ssh-agent（如果没运行）--临时脚本，单密钥管理，不够健壮
if [ -z "$SSH_AUTH_SOCK" ]; then
  eval "$(ssh-agent -s)" > /dev/null
  ssh-add /c/Users/wangb/.ssh/github_wangbohan808
fi
```


## 添加密钥配置文件

1. 在`.ssh`文件夹下创建一个无后缀的`config`文件

```config
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_wangbohan808
    IdentitiesOnly yes
```

2. 波浪号指代用户主目录
> [!tip] config文件说明
> 1. 给配置起别名
> 2. 指定服务器
> 3. 选择用户名
> 4. 使用密钥，自动输入密码
> 5. 这个HOST与指定的密钥绑死

3. github会寻找加载HostName为github.com的配置

# 删除本地仓库和远程仓库

```bash
git fetch origin --prune

git branch -D feature/time-sharing

git push origin --delete feature/time-sharing
```

# 切换分支

## 有未跟踪的文件修改

1. 强制删除切换
```bash
git checkout -f main
```


# 修改分支名称

```bash
1.切换到非目标分支

2.重命名本地分支
git branch -m feature/immediate-dust-clloction feature/immediate-dust-collction

3.删除远程仓库中旧分支
git push origin --delete feature/immediate-dust-clloction   （注意是旧分支名）

4.推送新的本地分支到远程
git push origin feature/immediate-dust-collction   (新分支名) 
```


# 依据分支创建分支

```bash
1.本地仓库创建新分支（新分支的命名是有要求的）
git checkout -b small-bag-immediate-dust-collection small-bag

2.将新分支推送到远程分支（新分支与远程分支对应关联，`git pull`、`git push`默认在关联分支进行）
git push -u origin small-bag-immediate-dust-collection
```


# 修改代码文件的跟踪状态

```bash
git add .
git add -u .
git commit -m "chore: 更新.gitignore，忽略MDK-ARM输出目录"

git push
```

# 比较具体修改

```bash
1.查看文件工作区版本与暂存区（或上一次提交版本）的差异
git diff projects/n32g003_EVAL/examples/Cortex-M0/SysTick/MDK-ARM/CM0_SysTick_Proj.uvguix.wangb

2.查看暂存区与最新一次提交的区别
git diff --staged projects/n32g003_EVAL/examples/Cortex-M0/SysTick/MDK-ARM/CM0_SysTick_Proj.uvguix.wangb

3.查看文件工作区与最新一次提交的所有更改
git diff HEAD projects/n32g003_EVAL/examples/Cortex-M0/SysTick/MDK-ARM/CM0_SysTick_Proj.uvguix.wangb
```

# 查看历史提交

```bash
1. 显示所有提交记录，包括提交哈希值、作者、日期和提交信息，按时间倒序排列：
   git log

2. 以单行形式显示提交记录，只显示提交哈希值的前7位和提交信息：
   git log --oneline
```

# 查看当前的git配置

```bash
git config --list

git config --local --list

cat .git/config

git config user.name

git config user.email 
```

