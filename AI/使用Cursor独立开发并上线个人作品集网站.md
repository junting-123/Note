### 使用Cursor+Pencil独立设计个人作品集网站（附实操图）



Cursor IDE 是一款基于 VS Code 内核构建的独立 AI 原生代码编辑器，并非依附于其他软件的插件，它内置强大的 AI 模型，支持对话式编程，新手也能轻松驾驭。

Pencil 是一款可嵌入 Cursor IDE 的设计插件，并非独立应用，它能通过自然语言指令生成结构化设计稿，且设计参数可直接同步给 [Agent](https://www.uisdc.com/tag/agent) 生成代码，实现设计与开发的无缝衔接。



## 1.准备工作

1.安装 Cursor IDE

安装地址：[Cursor: The best coding agent](https://cursor.com/cn)

![image-20260527220248364](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527220249379.png)



2.安装Pencil插件

1. 打开 Cursor IDE，使用快捷键 Ctrl+Shift+X（Windows）/Cmd+Shift+X（macOS），或点击左侧活动栏的“拼图”图标，打开插件市场面板。
2. 在插件市场的搜索框中，输入“Pencil”（注意首字母大写，避免搜索到无关插件），找到官方发布的 Pencil 插件（可参考下面配图对应的图表进行查找）。
3. 点击插件卡片上的“Install”按钮，等待 1-2 分钟，插件会自动下载并安装，安装完成后，左侧边栏会出现一个“铅笔”图标，说明安装成功。
4. 重启 Cursor IDE（可选）：部分情况下，插件需要重启才能生效，若左侧未出现铅笔图标，关闭 Cursor IDE 再重新打开即可。

![image-20260527220407055](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527220407923.png)





## 2. 用agent功能自动生成Pencil页面

> 安装好插件后，我们就可以在 Cursor IDE 内，用自然语言指令和 Cursor 的 Agent 进行对话 帮我们生成页面设计稿，这一步是整个流程的核心，重点关注指令的描述，描述越详细，生成的设计稿越符合预期。
>
> 

### 2.1 在 Pencil 插件中新建文件

①打开 Cursor IDE，点击左侧边栏的“铅笔”图标（Pencil 插件入口），进入插件界面。进入的时候会提示需要登录，可以去 pencil（ https://www.pencil.dev/ ）的官网进行注册，使用注册好的账号进行登录。

![image-20260527221221168](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527221222293.png)

注册完成后显示如下：

![image-20260527221455547](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527221456640.png)



② 点击插件界面中的“New .pen file”，创建一个新的设计画布（.pen 是 Pencil 设计文件的专属格式，可直接存储在代码仓库中，支持版本控制），此时会弹出一个无限画布，初始包含一个白色矩形（可删除）。

![image-20260527221726141](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527221727118.png)



③ 给设计文件命名：点击画布顶部的“Untitled.pen”，修改为容易识别的名称（如“test.pen”），方便后续在 Cursor IDE 中查找和管理。



### 2.2 用自然语言生成设计稿

> 我们需要在 Cursor 中，用文字描述想要的页面效果，就会自动在 Pencil 对应的画布上生成对应的设计元素，无需任何设计基础。

① 在 Cursor 的 New chat 对话框中，输入你的页面需求（描述越详细，生成效果越好，请选择 gpt 或 claude 模型，目前 pencil 仅支持这两类模型），示例指令（可直接复制修改）：

```

```



② 输入完成后，点击输入框右侧的“ 发送按钮 ↑，等待 30 秒-1分钟，会自动在画布上绘制出完整的设计稿。

![image-20260527222934649](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260527222935849.png)