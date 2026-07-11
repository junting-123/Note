

# Comfyui Docker 镜像部署到本地

> **核心思想**：把 ComfyUI 的代码和模型文件分开存放。
>
> - **代码（几十 MB）** → 打包进 Docker 镜像
> - **模型和数据（几十 GB）** → 放在宿主机，运行时挂载



## 1. 虚拟环境下载并运行 ComfyUI

官方安装教程：[如何在不同系统上手动安装 ComfyUI - ComfyUI 图形用户界面](https://docs.comfy.ac.cn/installation/manual_install)

由于后续要将comfyui docker到云端，所以我们不需要采用Conda 创建新环境，直接用pyhton venv方式。

1.打开git bash中，执行：

```
# 在你想要的位置创建目录，例如 F 盘
mkdir F:\stat_comfyui
cd F:\stat_comfyui
```

执行后，当前目录下会出现一个 `comfyui-env` 文件夹，这就是你的独立 Python 环境。



2.创建 Python 虚拟环境

用 `venv` 模块创建虚拟环境，环境名字可以叫 `comfyui-env`

```
python -m venv comfyui-env
```



3.激活环境

```
. comfyui-env/Scripts/activate
```

激活成功后，命令行前会出现 (comfyui-env)



4.验证是否激活成功

```
# 查看当前使用的 Python 路径（应该在虚拟环境目录下）
which python
# 应该显示: /f/stat_comfyui/comfyui-env/Scripts/python

# 查看 pip 路径
which pip
# 应该显示: /f/stat_comfyui/comfyui-env/Scripts/pip
```



5.安装comfyui

```
# 克隆 ComfyUI 代码仓库
git clone git@github.com:comfyanonymous/ComfyUI.git

# 安装GPU依赖PyTorch（此处针对Nvidia）
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# 安装 comfyui 依赖项
cd comfyui
pip install -r requirements.txt

启动 comfyui
cd comfyui
python main.py
```

访问 `http://localhost:8188`



## 2. 准备模型及个人数据目录

将comfyui涉及个人模型、数据的文件分和py程序隔离

| 类型                 | 文件夹                | 策略           |
| :------------------- | :-------------------- | :------------- |
| **数据（大、动态）** | models, output, input | 挂载 ✅         |
| **配置（个性化）**   | user                  | 挂载 ✅         |
| **扩展（可选）**     | custom_nodes          | 挂载（或打包） |
| **代码（固定）**     | 其他 py 文件          | 打包进镜像     |



在 `F:\start_comfyui\start_comfyui` 下创建目录结构

```
cd F:\start_comfyui\start_comfyui

# 创建模型、输出、输入目录
mkdir models, output, input，user，custom_nodes
```

把你现有的模型文件（`.safetensors`、`.ckpt` 等）复制到 `F:\start_comfyui\models`，并保持子目录结构。



## 3. 构建镜像

进入你的 comfyui 根目录 `F:\start_comfyui\comfyui`，新建一个无后缀的文件 dockerfile

```
F:\start_comfyui\
├── comfyui\           		# 代码（打包进镜像，不需要挂载）
│   ├── Dockerfile
│   ├── main.py
│   └── ...
├── comfyui_data\           # 数据目录（模型、输出、用户数据）
│   ├── models\             # 模型文件
│   ├── output\             # 生成图片
│   ├── input\              # 输入图片
│   ├── user\               # 工作流、配置
│   └── custom_nodes\       # 插件
```

dockerfile：

```dockerfile
# 使用官方 PyTorch 镜像，通过加速器下载
FROM pytorch/pytorch:2.5.1-cuda12.4-cudnn9-runtime

WORKDIR /app

# 使用清华 apt 源
RUN sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list

RUN apt-get update && apt-get install -y git && rm -rf /var/lib/apt/lists/*

# 使用清华 pip 源
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8188

CMD ["python", "main.py", "--listen", "0.0.0.0", "--port", "8188"]
```



 构建镜像：

```bash
cd F:\start_comfyui\comfyui
docker build -t comfyui .
#                  ↑
#               镜像名字
```



查看镜像：

```bash
docker images
```



## 4. 本地测试挂载

用 `-v` 参数把宿主机的 `models`、`output`、`input` 目录挂载到容器内：

```bash
docker run -d --gpus all -p 8188:8188 \
  -v F:/start_comfyui/comfyui_data/models:/app/models \
  -v F:/start_comfyui/comfyui_data/output:/app/output \
  -v F:/start_comfyui/comfyui_data/input:/app/input \
  -v F:/start_comfyui/comfyui_data/user:/app/user \
  -v F:/start_comfyui/comfyui_data/custom_nodes:/app/custom_nodes \
  --name comfyui comfyui:latest
#           ↑      ↑
#         容器名字  镜像名字
```

这是最重要的部分，把 Windows 上的文件夹“映射”到容器内部，实现数据持久化。

| Windows 路径（主机）                   | 容器内路径            | 用途                                                         |
| :------------------------------------- | :-------------------- | :----------------------------------------------------------- |
| `F:\start_comfyui\comfyui_data\models` | `/app/comfyui/models` | **模型文件目录**，存放 Checkpoint、LoRA、VAE、ControlNet 等模型。容器会从这里读取模型，你下载的新模型也放在这里 |
| `F:\start_comfyui\comfyui_data\output` | `/app/comfyui/output` | **输出目录**，生成的图片会保存到这里，容器重启后不会丢失     |
| `F:\start_comfyui\comfyui_data\input`  | `/app/comfyui/input`  | **输入目录**，你可以把需要处理的图片放在这里供工作流使用     |

**验证**：

- 访问 `http://localhost:8188`
- 加载工作流，选择模型
- 如果能正常生成图片，说明挂载成功

**清理测试容器**：

```
docker stop comfyui-test
docker rm comfyui-test
```



## 5. 镜像上传阿里云ACR

1.开通服务

这里阿里云容器镜像服务（ACR）：推荐访问 [https://cr.console.aliyun.com](https://cr.console.aliyun.com/) 开通个人版（免费）。



2.创建本地空间

命名为：`comfyui`

创建后有相关拉取推送操作指南。



3.登录阿里云 Docker 仓库

```bash
docker login --username=你的阿里云账号 registry.cn-hangzhou.aliyuncs.com
```



4. 给本地镜像打标签

```bash
docker tag [ImageId] crpi-97a0t3x6hs9hfupr.cn-chengdu.personal.cr.aliyuncs.com/xxxxxxx/comfyui_xxxxx:[镜像版本号]
```



5.推送镜像

```bash
docker push registry.cn-hangzhou.aliyuncs.com/xxxx/comfyui:latest
```

请根据实际镜像信息替换示例中的参数。

进入仓库详情页面后，在左侧导航栏中找到并点击“**镜像版本**”，可以查看到已成功推送的镜像列表。



## 6. 最快捷comfyui 部署云端

> 最快方案 = **云服务器直接运行 ComfyUI + SCP 一次性上传 + Syncthing做增量同步（可选）**

1.云服务器装comfyui

```
# 安装comfyui
git clone https://github.com/comfyanonymous/ComfyUI.git

# 进入目录
cd ComfyUI
python -m venv venv
source venv/Scripts/activate

# 直接安装依赖
pip install -r requirements.txt
```



2.Syncthing 云端自动同步本地目录

> Syncthing = 两台电脑之间“自动同步文件夹”的工具

你只需要做一次配置，之后：

- 本地改文件 → 云服务器自动同步
- 新增模型 → 自动上传
- 删除文件 → 自动同步删除

```
本地电脑（F盘模型库）
        ⇅ 自动同步
云服务器（ComfyUI模型目录）
```



