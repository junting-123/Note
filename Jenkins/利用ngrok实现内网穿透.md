# Windows/Linux 使用ngrok实现内网穿透

在现代软件开发与运维中，我们经常面临这样一个场景：

- 你的 Web 服务在本地 `localhost:8080` 跑得很欢，但外网同事看不到
- 微信/支付宝开发需要公网回调地址，本地无法接收
- 想给客户演示 Demo，但没有公网服务器

**ngrok 的出现**：提供了一条"隧道"，让内网服务安全地暴露到公网，**零配置、秒级启动**。



## 1.内网穿透概念

定义

内网穿透，也即NAT 穿透，进行 NAT 穿透是为了使具有某一个特定源 IP 地址和源端口号的数据包不被 NAT 设备屏蔽而正确路由到内网主机。简单来说，就是通过一定的技术手段，让位于内网（局域网）的设备能够被公网（互联网）上的设备所访问，实现内外网之间的互联互通。



内网穿透将内部网络的私有 IP 地址转换为公网上的公共 IP 地址，从而隐藏了内部网络的真实结构。然而，这也导致外部网络无法直接访问内部网络中的设备或服务。为了解决这个问题，内网穿透技术通过特定的中间设备及协议，如使用具有公网 IP 的中转服务器，来建立内外网之间的通信隧道。



## 2.应用场景

 1.本地开发与调试

在开发微信公众号、小程序、支付宝生活号或处理第三方支付回调时，服务商要求提供可公网访问的 `Webhook` 回调地址。使用 ngrok 可以瞬间将本地开发环境（如 `localhost:8080`）暴露到公网。



2.远程设备与服务器管理

对于部署在内网的物联网设备（如树莓派）或服务器，可以通过 ngrok 的 TCP 隧道实现安全的远程 SSH 连接，无需配置路由器端口映射。



3.临时共享与服务演示

数据科学家或后端开发在本地启动了一个 API 服务或前端页面，需要快速给团队同事、客户或外部合作伙伴演示时，ngrok 能最快生成可访问的链接，省去了繁琐的部署上线流程。



## 3. Windows使用 ngrok 实现内网穿透步骤

### 3.1 注册 ngrok 账号

ngrok 官方网站：[ngrok: AI & API Gateway | Secure Tunnels & Traffic](https://ngrok.com/)

![image-20260527121432019](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527121433989.png)

完成注册并登录。

### 3.2 下载 ngrok 客户端

进入官网的下载页面：[Get Started with ngrok | Setup Windows](https://dashboard.ngrok.com/get-started/setup/windows)

![image-20260527121828912](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527121830161.png)



### 3.3 配置账户

#### 3.3.1 获取 Authtoken

登录 Ngrok 控制台后，在个人设置或相关页面中找到 “Authtoken”。

注意：这是一个用于验证身份的密钥，Ngrok 通过它识别用户身份，确保合法使用服务。复制该 “Authtoken”，后续配置 Ngrok 时会用到。

![image-20260527122610804](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527122611744.png)





#### 3.3.2 配置 Authtoken

打开命令提示符（CMD），切换到 Ngrok 解压后的目录，然后执行命令 ：

```
ngrok authtoken（Your Authtoken）
```

括号内替换为实际的 Authtoken。执行此命令后，Ngrok 会将 Authtoken 保存到本地配置文件中，以便后续使用时自动验证身份。



#### 3.3.3 启动内网穿透

假设本地运行了一个 Web 服务，监听在 8080 端口，现在要通过 Ngrok 将其暴露到公网。在命令提示符中执行 :

```
ngrok HTTP 8080
```

执行该命令后，Ngrok 会与 Ngrok 服务器建立连接，并为本地的 8080 端口生成一个公网访问地址，类似 “[HTTPS://xxxx.ngrok-free.app](https://link.zhihu.com/?target=HTTPS%3A//xxxx.ngrok-free.app)”（xxxx 为随机生成的字符串）。此时，任何人通过这个公网地址就可以访问本地运行在 8080 端口的服务。



## 4. Linux 使用 ngrok 实现内网穿透步骤

### 4.1 使用官方 ngrok 服务

这里使用Ubuntu 24.04 进行演示，注册 ngrok 账号与如上 windows 一样 

将目录切换至新建的想要存放 ngrock 目录下，通过apt安装 ngrok，命令如下：

```bash
# 下载已编译好的 ngrok 客户端
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-linux-amd64.zip

# 解压
unzip ngrok-stable-linux-amd64.zip

# 移动到系统路径，无论在哪个目录，都能直接运行 ngrok http 8080
sudo mv ngrok /usr/local/bin/

# 配置 token
ngrok authtoken （Your Authtoken）

#  直接映射到 8080 端口
ngrok http 8080
```

![image-20260527134504825](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527134505891.png)

注意：不要随意ctrl+c关闭窗口，否则ngrok会自动退出

### 4.2 浏览器访问公网地址

现在，再用浏览器访问生成的公网地址

![image-20260527134706808](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527134708008.png)

访问成功！



### 4.3 ngrok 保持后台运行

关闭终端后进程不会终止，命令如下：

```bash
# nohup 命令可以让进程忽略挂断信号（SIGHUP）
nohup ngrok http localhost:8080 > ngrok.log 2>&1 &

# 查看运行状态
ps aux | grep ngrok

# 获取公网地址
cat ngrok.log

# 停止 ngrok 进程
pkill ngrok
```

