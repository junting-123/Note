# Fiddler抓包工具保姆级使用教程（超详细）

## 一. 下载安装Fiddler

### 1.1 Fiddler的应用场景

对于测试人员，Fiddler 的核心价值在于成为连接客户端与服务器之间的“透视镜”和“操控台”。通过拦截和分析 HTTP/HTTPS 流量，它能帮助测试人员在多个关键测试场景中精准定位问题、模拟异常情况、提升测试效率。



1.抓包分析与问题定位

当被测应用出现 Bug 时，它可以清晰展示客户端发出的请求内容和服务器返回的响应内容，帮助测试人员快速判断问题是前端 Bug、后端 Bug 还是配置问题。

- **核心功能**：完整捕获 HTTP/HTTPS 请求与响应，查看具体的 Header、Cookie、状态码、响应体等详细信息。
- **典型场景**：当页面数据显示错误时，可通过抓包确认是前端显示逻辑错误（请求正确但渲染错误），还是后端返回的数据本身就是错误的。当开发未能提供完整接口文档时，也可通过抓包分析界面操作，逆向获取接口信息



2.接口测试与调试

- **核心功能**：利用 `Composer` 组件，可以手动新建或拖拽历史请求，修改参数后重新发送（Replay），验证接口的逻辑和健壮性。
- **典型场景**：测试登录接口时，可重复发送请求，验证不同参数组合下的响应是否符合预期；进行重复请求测试（如重复提交订单），检查服务端是否有防重处理



3. 模拟请求与响应（Mock）

利用 `AutoResponder` 功能，可以让 Fiddler 自动拦截特定的请求，并返回自定义的数据，无需后端代码即可模拟各种场景。

- **核心功能**：将线上真实的 API 请求指向本地的 Mock 文件（如 JSON 或 XML 数据）。
- **典型场景**：在前端开发而后端接口尚未完成时，可以用 Mock 数据调试前端展示；模拟接口返回特定的错误码（如 404、500）或极长、为空的特殊数据，测试客户端是否能正确处理



4.弱网与异常场景模拟

对于移动端 App 测试，网络状况的多样性是测试重点，Fiddler 可以轻松模拟限速环境。

- **核心功能**：通过 `Rules` -> `Performance` -> `Simulate Modem Speeds` 开启弱网模拟，并可自定义限速规则。
- **典型场景**：测试 App 在地铁、电梯等弱网环境下的加载效果、超时处理机制及重连逻辑。



5. 安全测试辅助

测试人员可以利用 Fiddler 在数据传输过程中进行拦截和篡改，以此验证系统的安全性。

- **核心功能**：设置断点（`bpu` 命令），拦截请求或响应，在发送前修改参数（如价格、ID、用户状态）。
- **典型场景**：在支付接口测试中，拦截服务器返回的支付成功响应，将其中的金额从 `0.01` 改为 `0.00`，观察客户端和后台是否会错误地显示并处理为支付成功



6. 性能分析

Fiddler 提供直观的性能分析工具，帮助定位影响加载速度的瓶颈。

- **核心功能**：`Statistics` 面板提供请求的总耗时、DNS 解析时间、连接建立时间等详细数据；`Timeline` 面板可视化展示请求的并发与顺序关系。
- **典型场景**：分析页面加载慢的原因，是单个大文件耗时过长，还是并发请求数量过多导致阻塞



### 1.2 Fiddler 下载安装

官网下载：[[Fiddler下载 - Fiddler抓包工具中文版下载](https://fiddler.hk/download.html)](https://www.telerik.com/fiddler)

选择 FIDDLER TOOLS ——> Fiddler Classic——> Try For Free
进入到页面并填写相关信息。

![image-20260606160739032](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606160740212.png)

下载好后点击exe文件



## 二、Fiddler界面详解

### 2.1 总体布局

![image-20260606161251103](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606161252143.png)

左边web session面板的字段及图标含义如下：

| 字段名           | 含义解释                                                     | 测试用途举例                                                 |
| :--------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **#**            | 请求的唯一ID编号，从1开始递增。                              | 用于确认请求的发送顺序。                                     |
| **Result**       | HTTP响应状态码，如200（成功）、404（未找到）、500（服务器错误）等。 | **快速定位问题**：看到大量4xx或5xx状态码，能立刻判断是客户端请求有误还是服务器出错了。 |
| **Protocol**     | 请求使用的协议，如HTTP、HTTPS或FTP。                         | 确认敏感数据是否使用了HTTPS加密传输。                        |
| **Host**         | 请求发送的目标服务器的主机名或域名，通常包含端口号。         | 查看请求具体发往哪个服务器，帮助分析流量走向。               |
| **URL**          | 请求的资源路径和文件名，GET请求的参数也显示在这里。          | 分析具体调用了哪个API接口，以及传递了哪些参数。              |
| **Body**         | HTTP响应体的字节数大小。                                     | 判断响应数据是否过大，可能导致加载缓慢。                     |
| **Caching**      | 响应头中与缓存相关的字段，如`Cache-Control`或`Expires`。     | 验证缓存策略是否生效，判断资源是否因缓存问题而没有更新。     |
| **Content-Type** | 响应内容的MIME类型，如`text/html`、`application/json`、`image/png`等。 | 确认服务器返回的数据格式是否正确，比如接口是否返回了预期的JSON。 |
| **Process**      | 发起此请求的本地Windows进程名和进程ID（PID），如 `chrome:1234`。 | **精准定位请求来源**：当不确定某个请求是哪个程序发出的时，这个字段能直接告诉你。 |
| **Comments**     | 用户可以给会话添加备注的区域。                               | 对重要的或有问题的请求添加标记，方便后续回顾和团队协作。     |
| **Custom**       | 允许用户通过FiddlerScript脚本自定义的字段。                  | 高级功能，用于添加自定义信息来满足特定测试需求。             |



### 2.2 File菜单



![image-20260606161834080](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606161835178.png)

| 菜单项              | 功能说明                                           | 典型使用场景                                                 |
| :------------------ | :------------------------------------------------- | :----------------------------------------------------------- |
| **Load Archive**    | 加载一个已保存的 `.saz` 文件，会清空当前所有会话。 | 加载之前保存的问题现场，重新分析。                           |
| **Recent Archives** | 快速打开最近加载过的 `.saz` 文件。                 | 快速回顾近期调试过的项目。                                   |
| **Save**            | - **All Sessions**：保存当前所有会话。             | 需要完整保留整个抓包记录时使用。                             |
|                     | - **Selected Sessions**：只保存选中的会话。        | 从大量请求中筛选出关键几条单独保存，便于沟通。               |
| **Capture Traffic** | **开关**：控制 Fiddler 是否抓包（快捷键 `F12`）。  | 排除无关流量干扰，只在你需要的时候开始抓取。                 |
| **New Viewer**      | 打开一个新的 Fiddler 窗口（仅用于查看）。          | 同时对比两份 `.saz` 文件，或在不影响主窗口会话的情况下查看旧日志。 |
| **Exit**            | 关闭 Fiddler。                                     | -                                                            |



### 2.3 Edit 菜单

| 菜单项               | 功能说明                   | 快捷键 (默认)   | 典型使用场景                                                 |
| :------------------- | :------------------------- | :-------------- | :----------------------------------------------------------- |
| **Copy**             | 复制会话信息               | -               | 将选中的 Session 复制到剪贴板，粘贴到文档或聊天工具中。      |
|                      | - **Session**              | `Ctrl + C`      | 复制整个会话的摘要信息（如状态码、协议、主机等）。           |
|                      | - **Headers Only**         | -               | 只复制请求和响应的头部信息，不包含 Body。                    |
|                      | - **Full Session**         | -               | 复制会话的完整内容（头部+Body），可能非常大。                |
|                      | - **Response Body**        | -               | **仅复制响应 Body**。常用于快速保存接口返回的 JSON 数据。    |
| **Remove**           | 删除选中的会话             | `Del`           | 移除面板中选中的 Session，便于清理无关请求，聚焦关键流量。   |
| **Select All**       | 选中所有会话               | `Ctrl + A`      | 配合 `Remove` 或 `Save` 进行批量操作。                       |
| **Find Sessions...** | 按条件搜索会话             | `Ctrl + F`      | **非常常用**。可按 URL、状态码、响应体关键字等条件快速定位特定请求。 |
| **Flag**             | 给选中的会话添加标记       | `F2` (可自定义) | 用颜色高亮标记重要或异常的 Session，方便后续在列表中快速识别。 |
| **Unflag All**       | 清除所有标记               | -               | 批量清除所有高亮标记。                                       |
| **Lock**             | 锁定会话，防止被移除       | -               | 锁定重要的 Session，避免在执行 `Remove` 等操作时被误删。     |
| **Unlock All**       | 解锁所有被锁定的会话       | -               | 批量解锁。                                                   |
| **Go To Session...** | 跳转到指定ID的会话         | `Ctrl + G`      | 输入 Session 的编号（`#` 列的数字），直接跳转到该请求。      |
| **Paste as Session** | 将剪贴板内容粘贴为 Session | -               | 高级功能。将符合格式的文本（如请求报文）粘贴到 Session 列表中，可用于构造或重放请求。 |



### 2.4  rules菜单

| 菜单项                       | 功能说明                                                     | 核心测试场景                                                 |
| :--------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Hide Image Requests**      | 隐藏所有图片请求（如 .jpg, .png），会话列表中不再显示。      | 在分析网页性能时，隐藏图片可以让视图更聚焦于核心的HTML、JS和API请求。 |
| **Hide HTTPS CONNECTs**      | 隐藏用于建立HTTPS隧道的CONNECT请求。                         | 清理视图，让你直接看到经过解密的真实HTTPS请求内容，避免视觉干扰。 |
| **Automatic Authentication** | 自动对需要认证的请求（如NTLM）做出响应，避免反复弹窗。       | 测试需要Windows身份验证的内部网站，让抓包流程更顺畅。        |
| **User-Agents**              | 将浏览器的User-Agent头替换为预设的其他浏览器或设备（如iPhone、iPad）。 | **移动端测试**：在电脑浏览器上测试网页，通过修改UA来模拟手机浏览器，查看服务端返回的页面适配是否正确。 |
| **Performance**              | - **Simulate Modem Speeds**：模拟慢速网络（弱网测试）。 - **Disable Caching**：禁用缓存，强制每次都从服务器拉取最新资源。 | **弱网测试**：测试App或网站在网络差时的表现和超时处理。 **缓存测试**：验证前端资源更新后，用户是否能立即看到新版本。 |
| **Remove All Encodings**     | 移除请求和响应中的压缩头（如gzip），让数据以未压缩的原始形式传输。 | 便于直接在Inspector中**阅读和修改**请求/响应的正文内容。     |



### 2.5 Tools菜单

| 菜单项                          | 功能说明                                                     | 使用场景 / 备注                                              |
| :------------------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Options...**                  | **打开 Fiddler 的核心设置窗口**。可以配置： - **HTTPS**：解密 HTTPS 流量、安装证书。 - **Connections**：设置代理端口（默认8888）、允许远程连接（抓包手机 App）。 - **Gateway**：设置上游代理等。 | 最常用的选项，Fiddler 的大部分关键配置都在这里。             |
| **WinINET Options...**          | 打开 Windows 系统自带的 **Internet 属性**设置窗口。          | 快速修改系统代理设置，功能与在控制面板中找到 Internet 选项相同。 |
| **Clear WinINET Cache**         | **清空 WinINET 缓存**。WinINET 是 Windows 系统中处理 HTTP/HTTPS 请求的底层 API，IE 浏览器和许多其他应用都依赖它。 | 当你怀疑因缓存导致资源未更新时，可快速清除。                 |
| **Clear WinINET Cookies**       | **清空 WinINET 记录的 Cookie**。                             | 用于清理某些顽固的、无法通过浏览器常规方式清除的 Cookie，适合进行干净的会话测试。 |
| **TextWizard...**               | 打开一个文本编码/解码小工具。                                | 非常实用，可快速对 `URL` 编码、`Base64` 编码等进行互转，方便分析接口参数或响应内容。 |
| **Compare Sessions**            | **对比两个 Session**。需先在左侧会话列表选中两个请求，再点击此项。 | 通常需要额外安装插件（如 WinDiff）才能使用，常用于对比两次请求/响应的差异（如参数、Cookie等）。 |
| **New Session Clipboard...**    | 打开一个“会话剪贴板”窗口。                                   | 这是一个临时存放区，你可以把左侧会话列表中选中的会话拖拽进去，方便对特定请求进行单独分析和比较。 |
| **HOSTS...**                    | 打开 **主机重定向工具**。                                    | **非常实用的功能**！可以在 Fiddler 内部配置 IP 和域名的映射关系，**作用类似于修改 Windows 的 hosts 文件**，但无需管理员权限，且可以随时启用/停用，适合在测试环境间快速切换。 |
| **Reset Script**                | 重置 FiddlerScript。                                         | 如果你在 `Rules` -> `Customize Rules...` 中编写或修改了脚本导致异常，可以用此选项恢复到初始状态。 |
| **Sandbox**                     | 打开一个 Fiddler 官方文档页面（http://webdbg.com/sandbox/）。 | 内容较少，实用性不高。                                       |
| **View IE Cache**               | 打开 IE 浏览器缓存文件的存放路径。                           | 用于直接查看系统级别的缓存文件。                             |
| **Win8 Loopback Exemptions...** | 打开 **Win8/Win10 回环豁免** 工具（在工具栏中常被称为 `WinConfig`）。 | **这是抓取“UWP 应用”（如 Windows 应用商店里的 App）或某些现代浏览器（如 Edge）流量的关键入口**。由于这些应用运行在受保护的容器中，默认不走本地代理，需要在这里进行“豁免”才能被 Fiddler 抓到 |



### 2.6 views菜单

| 功能                 | 快捷键     | 说明                                                         |
| :------------------- | :--------- | :----------------------------------------------------------- |
| **Show Toolbar**     | -          | 控制是否显示顶部工具栏。如果不小心隐藏了工具栏，在这里勾选即可恢复。 |
| **Default Layout**   | -          | **默认布局**：左侧是会话列表（Web Sessions），右侧上下分别显示请求（Request）和响应（Response）。 |
| **Stacked Layout**   | -          | **堆叠布局**：会话列表在上方，请求和响应在下方，适合需要查看长内容时使用。 |
| **Wide Layout**      | -          | **宽布局**：会话列表在上方，请求和响应在下方左右分屏显示，充分利用宽屏空间。 |
| **Reset Layout**     | -          | **重置布局**：如果界面拖拽乱了，选择此项可一键恢复默认布局。 |
| **Stay on Top**      | -          | 让 Fiddler 窗口始终保持在最前端，不被其他窗口遮挡。          |
| **Minimize to Tray** | `Ctrl + M` | 最小化 Fiddler 到系统托盘（通知区域），而不是任务栏，让后台抓包不干扰前台操作。 |
| **Refresh**          | `F5`       | 刷新界面，通常用于重绘视图或更新显示                         |

## 三、数据统计面板

### 3.1 Statistics

关于HTTP请求的性能（例如发送/接受字节数，发送/接收时间，还有粗略统计世界各地访问该服务器所花费的时间）以及数据分析。

![image-20260606162939960](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606162940771.png)



### 3.2 Inspectors

用于查看会话的内容，上半部分是请求的内容，下半部分是响应的内容，提供headers、textview、hexview,Raw等多种方式查看单条http请求的请求报文的信息。

![image-20260606163123194](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260606163124521.png)



### 3.3 AutoResponder

Fiddler拦截到的某个网络请求，自动替换为你指定的本地文件或固定响应内容，从而无需修改服务器代码，即可模拟各种返回场景进行测试。



## 四. 实战操作



### 4.1 基本抓包操作

开始/停止抓包：

- 点击File > Capture Traffic或使用快捷键F12
- 状态栏左侧显示"Capturing"表示正在抓包



清空会话列表：

- 点击Edit > Remove > All Sessions或使用Ctrl+X



保存会话数据：

- 点击File > Save > All Sessions 可保存为 SAZ 格式（Fiddler专用格式）
- 也可导出为其他格式如 HTTPArchive 格式



### 4.2 劫持网页实战（autoresponder）

将“baidu”这个关键字跟本地电脑的一张图片绑定了，再访问带有“baidu”关键字的地址，就会被劫持，具体如下：

1. 在 Fiddler 右侧，点击 AutoResponder 标签页。
2. 将左侧选中的那个百度请求，直接用鼠标拖拽到右侧的 AutoResponder 面板中。你会看到面板里自动出现了一条规则。
3. 在面板下方，勾选顶部的三个复选框：
   - Enable Rules：这是“启用规则”的总开关，必须勾上。
   - Unmatched requests passthrough：让没被拦截的请求正常通过，不影响你上其他网。
   - Enable Latency：模拟网络延迟，建议勾上让实验更真实。
4. 在下面的规则列表中，点击你刚拖进来的那条规则。在右侧的 "Rule Editor" 区域，为了匹配所有带“baidu”的地址，输入： `baidu`。
5. 点击下半部分的下拉框，选择 "Find a file..."。
6. 在弹出的文件选择框中，找到并选中你准备的那张本地图片（例如 `F:\test.jpg`），然后点击打开。
7. 点击右侧的 "Save" 按钮

![image-20260607161547534](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607161548405.png)



浏览器显示：

![image-20260607140557550](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607161318598.png)





### 4.2  抓包过滤实战（Filter）

最常用的两种抓包过滤的方式

1. host-请求的目标地址

![image-20260607161739220](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607161740435.png)

show only intranet hosts：只显示内网主机（较少使用）

show only internet hosts：只显示互联网主机

![image-20260607162453834](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607162455186.png)

Hide the following Hosts：表示在输入框中，输入了哪些域名信息，就不过滤，不进行监听。

Show only the following Hosts：标识在输入框中输入了哪些域信息，就只监听这些域名，其他的域名将不进行监听。

Flag the following Hosts：表示在输入框中输入了哪些域名信息，在左侧的session面板中，这些配置的域名在监听到时，会加标识



2. client process-进程过滤

![image-20260607162046951](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607162048838.png)

Show only traffic from：只显示来自后面选择进程的请求

Show only Internet Exporer traffic：只显示来自IE的请求

Hide trafficfrom service host：隐藏来自service host的请求



以只抓包www.baidu.com为例，使用 Fileter，设置参数如下：

![image-20260607161117755](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607161118696.png)

浏览器输入www.baidu.com，在监控面板可以看到只输出此网址的信息：

![image-20260607162856637](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607162857417.png)



### 4.3 弱网测试（Simulate Modem Speeds）

> Fiddler模拟弱网，控制电脑的网速等

fiddler 控制网速

1. 启动 Fiddler，打开菜单栏Rules---Performances---Simulate Modem Speeds这里打开了模拟调节速度

![image-20260607163920246](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607163921662.png)



2. 打开 Fiddler Script，找到以下代码进行修改

```
 if (m_SimulateModem) {
     // Delay sends by 900ms per KB uploaded.
     oSession["request-trickle-delay"] = "900"; 　　# 每上传lKB 数据，延时0.9秒
     // Delay receives by 700ms per KB downloaded.
     oSession["response-trickle-delay"] = "700"; 　　# 每下载lKB 数据，延时0.7秒
```

点击save script

![image-20260607164601245](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607164602043.png)

然后去请求网站，发现请求速度变慢。



### 4.4 接口测试（Composer）

1.compose界面介绍如下：

![image-20260607165603225](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607165604158.png)

我们可以把抓包的数据直接拽到Composer中，获取接口的所有的请求信息。

![image-20260607165841641](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607165842951.png)



**composer 请求 get 方式**

1、直接通过Composer请求 github

2、点击执行后会发现左侧会话列表出现一个请求内容

![image-20260607170645917](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607170646724.png)



3、点击查看请求内容

![image-20260607170934306](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260607170935407.png)

## 五、定位bug

**1、整体定位**

用 Fiddler 调试接口和定位 Bug，主要看以下几个关键位置：

| 列名     | 作用       | 异常标志                    |
| :------- | :--------- | :-------------------------- |
| #        | 请求序号   | -                           |
| Result   | HTTP状态码 | ⚠️ 非200/304/302都可能是问题 |
| Protocol | HTTP/HTTPS | -                           |
| Host     | 目标服务器 | 检查是否发错服务器          |
| URL      | 请求路径   | 检查路径是否正确            |

常见的成功状态码：

| 状态码 | 出现场景                 | 谁背锅             |
| :----- | :----------------------- | :----------------- |
| 200    | 正常访问                 | ✅ 正常             |
| 304    | 刷新页面时常见           | ✅ 正常（性能优化） |
| 404    | 网址输错、接口路径写错   | 前端               |
| 500    | 后端代码报错、数据库挂了 | 后端               |
| 302    | 未登录跳转登录页         | ✅ 正常逻辑         |

快速定位技巧：

- 红色图标：请求失败（网络错误、DNS解析失败）
- 红色字体：返回4xx/5xx错误码



**2、分析bug（inspectors）**

> 选中左侧某个请求后，右侧 **Inspectors** 是你主要分析的地方。

- 前端问题：请求部分

| 标签     | 看什么                                    | 常见Bug                           |
| :------- | :---------------------------------------- | :-------------------------------- |
| Headers  | 请求头（Method、URL、Cookie、User-Agent） | 缺少Token、Cookie没带、Method写错 |
| WebForms | POST表单参数（key-value形式）             | 参数名拼写错误、必填项缺失        |
| JSON     | POST请求的JSON Body                       | JSON格式错误、字段名不匹配        |
| Raw      | 原始请求数据                              | 查看完整请求内容                  |

定位技巧：

前端Bug → 请求参数发错了（比如 age 写成 aeg）



- 后端问题：响应部分

| 标签     | 看什么                                     | 常见Bug                    |
| :------- | :----------------------------------------- | :------------------------- |
| Headers  | 响应头（Content-Type、Set-Cookie、Server） | 返回类型不对、Cookie没设置 |
| JSON     | JSON格式的响应体                           | 后端返回的错误信息都在这里 |
| TextView | 纯文本响应                                 | 查看HTML错误页面           |
| Raw      | 原始响应数据                               | -                          |

定位技巧：

后端Bug → 返回的状态码和响应体告诉你问题在哪
- 500：服务器内部错误（看后端日志）
- 404：路径不存在（检查URL）
- 400：参数错误（看响应体的error字段）
- 401：未授权（检查Token）

