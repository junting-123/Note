# git上传代码到github仓库

> SSH 的全称是 **Secure Shell**（安全外壳协议），它的核心作用就是**通过网络安全地连接到远程计算机**，并在上面执行命令。



## 一.下载git安装

![image-20260518160705396](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260518160708293.png)



## 二、SSH连接github

### 2.1本地生成公钥

1. 在 Git Bash 窗口中粘贴下面的命令（请把其中的邮箱替换成你自己的 GitHub 邮箱），然后按回车。

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```



2. 按三次回车：执行后，命令行会依次询问你三个问题。不需要输入任何内容**，直接按三次回车键即可。

- 第一次问：保存密钥的位置（默认就行）
- 第二次问：设置密码（留空就行）
- 第三次问：再次确认密码（留空就行）



3. 复制公钥：密钥生成后，输入以下命令来复制公钥，它会自动保存到你的剪贴板里。

```bash
cat ~/.ssh/id_ed25519.pub | clip
```



4. 验证是否成功

- 检查文件是否存在

```
ls ~/.ssh/
```

能看到 `id_ed25519` 和 `id_ed25519.pub` 两个文件。



### 2.2 github填写SSH Key

1. 打开github官网-settings

![image-20260518161117590](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260518161118832.png)



2. 依次点击SSH and GPG Keys-New SSH Key

![image-20260518161253041](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260518161254257.png)



3. 填写SSH信息

![image-20260518163814238](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260518163816098.png)



### 2.3 SSH自动使用正确密钥

1. 创建一个配置文件，告诉 SSH 优先使用 `id_ed25519`：

```bash
echo "IdentityFile ~/.ssh/id_ed25519" > ~/.ssh/config
```

2. 测试

```bash
ssh -T git@github.com
```

出现You've successfully authenticated，说明连接成功！

你已经完成了 SSH Key 配置，现在可以免密码操作 GitHub 仓库了



## 三、本地项目推送至github仓库

流程：

![image-20260523215653627](C:/Users/79153/AppData/Roaming/Typora/typora-user-images/image-20260523215653627.png)



完整命令：

```bash
cd /path/to/calc-project #你要git的文件夹

git init  #初始化仓库
git add .  #添加文件到本地仓库
git commit -m "first commit"  #添加文件描述信息

git remote add origin + 远程仓库地址 //链接远程仓库，创建主分支 eg：git remote add origin https://github.com/junting-123/Note.git
git push -u origin main  #把本地仓库的文件推送到远程仓库
git pull origin master #把本地仓库的变化连接到远程仓库主分支
```



流程：`cd 项目文件夹` → 改代码 → `git add .` → `git commit` → `git push`



其他git语句

```
# 添加远程仓库
git remote add origin<远程仓库地址>

# 查看当前分支
git branch

# 查看所有分支（本地+远程）
git branch -a

# 切换分支
git checkout 分支名

# 创建并切换到新分支
git checkout -b 新分支名

# 如果分支来自远程
git checkout -b 本地分支名 origin/远程分支名

# 删除本地分支（已合并的）不能删除当前所在分支 - 需要先切换到其他分支
git branch -d 分支名

# 强制删除本地分支（未合并也删）
git branch -D 分支名
```

