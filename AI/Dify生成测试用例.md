# 使用 Dify 生成测试用例并部署飞书

> Dify 是一款开源的大模型应用开发 / 运营平台（LLMOps），零代码 / 低代码快速搭建企业级 AI 应用，支持对话机器人、RAG 知识库、工作流、智能 Agent  等场景。



## 一、Window 上使用 Docker 部署 Dify

安装官方教程：[使用 Docker Compose 部署 Dify - Dify Docs](https://docs.dify.ai/zh/self-host/quick-start/docker-compose)

### 1.1 克隆 Dify 仓库

进入命令行：

```bash
# 切换到目标目录
cd F:\dify

# 克隆 Dify 源代码到本地机器
git clone https://github.com/langgenius/dify.git

# 进入 Dify 源代码中的 docker 目录
cd dify/docker

# 复制必要的环境配置文件
cp .env.example .env

# 启动容器
docker compose up -d

# 验证所有容器是否成功
docker compose ps

# 查看日志（如有问题）
docker compose logs -f
```

应看到类似以下的输出，显示每个容器的状态和启动时间：

![image-20260603225339648](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603225340825.png)



### 1.2 访问 Dify

打开浏览器，访问：**[http://localhost](http://localhost/)**

首次访问需要设置管理员账号：

- 邮箱：输入你的邮箱
- 密码：设置密码

![image-20260603225431859](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603225432853.png)

## 二、搭建测试用例助手 



### 2.1 对话型应用

1. 新建「对话型应用」，登录 Dify 控制台，点击左侧「应用」，右上角「新建应用」。 
2. 选择「对话型应用」，输入应用名称（如「测试用例自动生成助手」）。
3. 点击「创建」 ——> 进入应用编辑页面（核心操作区）。

![image-20260603231158288](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260603231200371.png)



### 2.2 准备大模型

> Ollama 是"模型运行器"。你需要它来运行模型，让 Dify 能调用。



先安装 Ollama 并拉取模型（`ollama pull llama3:8b`），确保 Ollama 服务正常运行（默认端口 `11434`）；

1. 下载安装包：访问 Ollama 官网 [ollama.com](https://ollama.com/)，下载 `OllamaSetup.exe` 安装程序。

```bash
irm https://ollama.com/install.ps1 | iex
```



2. 运行安装：双击下载的 `.exe` 文件，点击 `Install` 即可完成安装。

![image-20260604132239864](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260604132240776.png)



3. 验证安装：安装完成后，打开 命令提示符 (CMD) 或PowerShell，输入以下命令并回车，如果显示版本号（如 `ollama version is 0.5.7`）则说明安装成功：

   ```
   cd F:\dify\ollama
   .\ollama --version
   ```



4. 手动启动服务并指定路径

   ```
   # 1. 先进入你的 ollama 程序目录
   cd F:\dify\ollama
   
   # 2. 关键：在当前窗口强制设置模型存储路径
   $env:OLLAMA_MODELS = "F:\dify\model"
   
   # 3. 在当前窗口启动服务（重要：不要关这个窗口）
   .\ollama serve
   ```

   这时候服务窗口应该会提示类似 `"server listening on 0.0.0.0:11434"` 的信息，这个窗口必须一直开着，不能关。

   保持刚才的服务窗口运行，再新开一个 PowerShell 窗口，执行：

   ```
   cd F:\dify\ollama
   .\ollama pull llama3:8b
   ```

   下载完成后，`F:\dify\model` 目录下应该会自动出现完整的 `blobs`、`manifests`、`models` 子目录，并且里面有大量数据（4-5GB）。



5. 模型配置

   本地 Ollama 拉取模型：

   ```
   # 拉取模型
   ollama pull llama3:8b
   
   # 查看目录是否有内容
   dir F:\dify\model
   
   # 或者通过 ollama 命令查看已下载的模型列表
   cd F:\dify\ollama
   .\ollama list
   ```



### 2.3 配置大模型

Dify 调用大模型

1. 点击顶部「模型配置」→ 左侧「自定义模型」→ 「添加模型」；
2. 选择「Ollama」，填写配置项
   - 模型名称：llama3:8b
   - 基础URL：本地直装 Dify 填 `http://localhost:11434`；Docker 部署 Dify 填 `http://host.docker.internal:11434`
   - 模型类型：对话
   - 模型上下文长度：8192
   - 最大token上限：4096



### 2.4 设计提示词

提示词是生成高质量测试用例的关键，需明确「角色、输出格式、覆盖场景」，操作如下：

1. 回到「提示词编排」页面，清空默认提示词，可参考如下提示词模板：

```
【角色】你是拥有10年软件测试经验的资深测试专家，精通功能测试、接口测试、边界测试、异常测试，熟悉行业通用测试用例规范。
【任务】根据用户提供的「模块名称」和「需求描述」，生成结构化、可直接执行的测试用例。
【输出规则】
1. 严格按照以下Markdown表格格式输出，无任何额外冗余内容：
| 用例ID | 测试模块 | 测试场景 | 详细测试步骤 | 预期结果 | 优先级 | 测试类型 |
|--------|----------|----------|--------------|----------|--------|----------|
| TC-{模块缩写}-001 | 模块名称 | 场景描述 | 步骤1；步骤2；步骤3 | 可验证的结果描述 | 高/中/低 | 功能/接口/边界/异常 |
2. 用例覆盖维度：必须包含正常场景、异常场景、边界场景，核心功能标注「高优先级」；
3. 测试步骤需「可执行」（如“调用接口时传入参数XXX”），预期结果需「可验证」（如“返回200状态码+XXX字段”）；
4. 用例ID格式统一：TC-模块缩写（2-4个字符）-序号（3位）。

用户消息：{{{#sys.query#}}

请从上面的用户消息中提取「模块名称」和「需求描述」，然后生成测试用例，用 Markdown 表格输出。

【开始生成】请严格按上述规则生成测试用例：
```



### 2.5 配置应用交互界面

 点击右上角「功能」→ 「对话开场白」→ 「编写开场白」：


<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260605160629989.png" alt="image-20260605160629062" style="zoom:50%;" />

开场白参考：

```
请填写以下信息：
模块名称：如「用户登录」「支付模块」
需求描述：详细描述功能需求
我将根据你的输入生成结构化、可直接使用的测试用例 ✨
```



### 2.6 测试验证

点击页面右上角「预览」，进入工具交互界面；

测试参考：

```
module_name：文件上传模块

requirement_desc：用户可以在个人中心上传头像，支持JPG、PNG格式，大小不超过5MB。上传成功后，页面应显示新头像预览图。上传非图片格式文件时，提示“仅支持JPG/PNG格式”；上传超过5MB的文件时，提示“文件大小不能超过5MB”；未选择文件直接点击上传时，提示“请先选择文件”。
```

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260605161906629.png" alt="image-20260605161905442" style="zoom:50%;" />

等待几秒，输出测试用例：

| 用例ID    | 测试模块     | 测试场景                | 详细测试步骤                                                 | 预期结果                   | 优先级 | 测试类型 |
| :-------- | :----------- | :---------------------- | :----------------------------------------------------------- | :------------------------- | :----- | :------- |
| TC-FU-001 | 文件上传模块 | 正常上传                | 1. 用户选择 JPG 图片文件 2. 点击上传按钮 3. 等待上传结果     | 上传成功，显示新头像预览图 | 高     | 功能     |
| TC-FU-002 | 文件上传模块 | 异常上传 - 非图片格式   | 1. 用户选择 TXT 文本文件 2. 点击上传按钮 3. 等待上传结果     | 提示“仅支持JPG/PNG格式”    | 中     | 接口     |
| TC-FU-003 | 文件上传模块 | 异常上传 - 文件大小超出 | 1. 用户选择大于5MB 的 JPEG 图片文件 2. 点击上传按钮 3. 等待上传结果 | 提示“文件大小不能超过5MB”  | 中     | 边界     |
| TC-FU-004 | 文件上传模块 | 正常上传 - 未选择文件   | 1. 用户未选择文件 2. 点击上传按钮 3. 等待上传结果            | 提示“请先选择文件”         | 低     | 异常     |
| TC-FU-005 | 文件上传模块 | 正常上传 - 文件大小合法 | 1. 用户选择小于5MB 的 PNG 图片文件 2. 点击上传按钮 3. 等待上传结果 | 上传成功，显示新头像预览图 | 高     | 功能     |



## 三、Dify API 部署到飞书

### 3.1 飞书应用配置

#### 3.1.1 飞书创建应用

用电脑浏览器打开 **飞书开放平台**：https://open.feishu.cn/app

1. **登录**：点击页面右上角的“登录”，用你的飞书账号扫码或输入手机号登录
2. **进入后台**：登录后会自动进入“开发者后台”



在开发者后台页面，找到并点击 **“创建企业自建应用”** 按钮

1. 在弹出的窗口中，填写：
   - **应用名称**：给机器人起个名字，比如“我的测试助手”
   - **应用描述**：简单说明一下用途，比如“用于内部测试和通知”
2. 点击 **“确定创建”**，应用就建好了



#### 3.1.2 飞书获取 App ID 和 App Secret

1. 创建成功后，会自动进入应用详情页。在左侧菜单栏找到 **“添加应用能力”**，点击
2. 在“添加能力”弹窗中，找到并点击 **“机器人”** 旁边的 **“添加”** 按钮

3. 在应用详情页的左侧菜单栏，点击 “凭证与基础信息”

4. 在右侧页面中，可以看到 “App ID” 和 “App Secret”

- App ID：可以直接复制使用
- App Secret：需要点击“点击获取” 进行验证后，才会显示并可以复制

保存好这两个信息，它们就是飞书应用的“账号”和“密码”，LangBot 配置时需要用到



#### 3.1.2 配置权限并发布应用

拿到 ID 和 Secret 后，应用还不能直接使用，必须完成权限配置和发布。

1. 配置消息权限：
   - 点击左侧 “权限管理”
   - 在“权限管理”页面，搜索并添加以下核心权限（点击“开通权限”）：
     - 接收群聊中@机器人消息事件 (`im:message.group_at_msg`)
     - 获取与发送单聊、群组消息 (`im:message`)
     - 读取用户发给机器人的单聊消息 (`im:message.p2p_msg`)
     - 获取用户 user ID (`contact:user.employee_id:readonly`)
   - （可选）其他权限可根据需要添加，以上是机器人收发消息的基础
2. 发布应用：
   - 点击左侧 “版本管理与发布”
   - 点击 “创建版本”，填写版本号（如 `1.0.0`）和更新说明
   - 点击 “发布”，应用状态变为“已上线”后，机器人才能被其他成员找到并使用

![image-20260606002102521](C:/Users/79153/AppData/Roaming/Typora/typora-user-images/image-20260606002102521.png)



添加“接收消息”事件

1. 在应用详情页左侧，点击 “事件与回调”。
2. 在“订阅方式”一栏，选择 “使用长连接接收事件”。
   - （这种模式不需要配置公网地址，最简单）
3. 点击下方的 “添加事件” 按钮。
4. 在弹出的窗口中，搜索并勾选 `im:message.receive_v1` (接收消息)，然后确认添加



### 3.2 Dify 应用发布

1. 打开你的测试用例生成助手（已确认是CHATFLOW类型）
2. 点击右上角"发布"按钮，确保应用已发布

![image-20260605163223541](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260605163224673.png)

3. 获取API访问凭证
   - 进入Chatflow详情页面
   - 点击左侧菜单的"访问API"
   - 复制以下信息：基础URL：如 `https://api.dify.ai/v1`（云服务）或自部署地址；API密钥：格式如 `app-xxxxxx`



### 3.3 通过LangBot完成对接

> LangBot是连接飞书和Dify的关键中间件，负责接收飞书消息并转发给Dify处理。

#### 3.3.1 使用Docker部署LangBot

命令行运行：

```
# 拉取LangBot镜像
docker run -d --name langbot -p 5300:5300 -v ./langbot/data:/app/data docker.langbot.app/langbot-public/rockchin/langbot:latest

# 查看容器状态
docker ps

# 查看日志
docker logs langbot

# 访问
http://localhost:5300
```



如果下载过慢，采用如下方法：

```
git clone https://github.com/langbot-app/LangBot.git
cd LangBot/docker

# 启动服务
docker compose up -d
```

部署完成后，访问 `http://你的服务器IP:5300` 进入LangBot Web管理界面。



#### 3.3.2 配置飞书机器人

①登录LangBot WebUI（默认端口5300）

②左侧菜单点击"机器人" → 点击"+"创建机器人

③填写飞书应用信息：

- 平台选择：飞书
- 应用名称：你创建的应用名称
- App ID：从飞书开放平台获取
- App Secret：从飞书开放平台获取

④勾选"启用飞书流式回复模式"（实现逐字输出效果）

⑤提交保存

![image-20260606001100285](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606001101558.png)



#### 3.3.3 配置Dify对接

②在"AI引擎"部分，运行器选择**Dify服务API**

③填写Dify参数

| 参数     | 说明                | 你的值                                |
| :------- | :------------------ | :------------------------------------ |
| Base URL | Dify API地址        | `https://api.dify.ai/v1` 或自部署地址 |
| API Key  | 从Dify获取的API密钥 | `app-xxxxxxxx`                        |
| 应用类型 | 选择"聊天"          | chat                                  |

<img src="https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606002623397.png" alt="image-20260606002622408" style="zoom:50%;" />



配置完成后，飞书显示如下，说明配置成功

![image-20260606003022459](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606003023455.png)



### 3.4 测试验证

1. 在飞书中搜索你的应用名称
2. 发送测试消息，如："模块名称：用户登录，需求描述：用户输入正确账号密码可登录成功"
3. 机器人应返回生成的测试用例表格

![image-20260606150952390](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606150953978.png)



注意：要同时启动dify，ollama以及Langbot，智能体才生效！