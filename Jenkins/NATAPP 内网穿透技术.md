# 0成本在 Linux 上实现 NATAPP 内网穿透技术

> Natapp 启动后会在本地生成一个**固定的、可预测的公网地址**（例如 `http://jenkins-test.natappfree.cc`），你可以将这个地址直接配置到你的 Webhook 里，实现全自动触发。
>

- linux操作系统：ubuntu 24.04

  

## 1.natapp官网注册账户

登录官网：[NATAPP-内网穿透 基于ngrok的免费企业级国内高速内网映射工具](https://natapp.cn/)



1. 购买一个**免费隧道**（价格显示为0元）。购买时会要求你填写本地服务的端口信息。
   - **隧道协议**：选择 `Web` 或 `HTTP`。
   - **本地端口**：填写你的 Jenkins 端口，通常是 `8080`。

![image-20260602141840300](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260602141841568.png)

注意：涉及到实名认证



2. 查看已购买的隧道

![image-20260602142137614](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260602142139384.png)

点击配置查看并显示authtoken



## 2.Linux上安装与启动

Natapp 提供了非常方便的一键安装脚本，无需手动下载解压。

在你的 Linux 虚拟机终端中，执行以下命令：

```bash
# 使用一键安装脚本，将命令中的 <你的token> 替换为你在官网「我的隧道」里看到的 authtoken
curl -fsSL "https://natapp.cn/get.sh?authtoken=<你的token>" | sh
```



脚本执行完毕后，会提示你 `run_natapp.sh` 脚本的位置。直接运行它即可启动隧道

```
# 进入脚本所在目录（脚本输出会提示具体路径，通常为 /opt/natapp）
cd /opt/natapp
./run_natapp.sh
```

启动成功后，终端会显示一个公网地址（如 `http://jenkins-test.natappfree.cc`），这意味着你的 Jenkins 已经成功穿透到公网了。



## 3.服务后台运行

为了不让 SSH 会话关闭后隧道就断开，你需要让 Natapp 在后台运行。

在你完成安装的目录（`/opt/natapp`）下，执行以下命令：

```bash
# 使用 nohup 将 natapp 放到后台执行
nohup ./natapp -authtoken=<你的token> > natapp.log 2>&1 &
```

这样一来，你的 Jenkins 就能通过一个固定的公网地址，无人值守地接收来自 GitHub 的 Webhook 触发了。



## 4.配置 GitHub Webhook

1. 打开你的 GitHub 仓库，进入 **Settings** -> **Webhooks**。

2. 点击 **Add webhook**。

3. 在 **Payload URL** 中填入 Natapp 生成的公网地址，**并加上完整的 Jenkins Webhook 路径**。格式如下：
   `http://your-domain.natappfree.cc/github-webhook/`
   **(注意：路径末尾的 `/` 可能很重要，建议保留。)**

4. **Content type** 选择 `application/json`。

5. **SSL verification**：选Disable (not recommended)，允许使用HTTP协议。

   

   保存即可。之后每次 `git push`，GitHub 就会自动通过这个固定地址触发你的 Jenkins 构建了。