# 搭建职工信息管理系统



## 1、需求分析

> Flask 是Web 后端框架
>
> Flask 的核心职责是处理 HTTP 请求与响应。负责接收浏览器发来的各种请求（比如查看页面、提交表单），然后调用对应的 Python 函数处理，最后将处理结果（通常是 HTML 页面或 JSON 数据）返回给浏览器。



职工信息管理系统包括以下几个模块：

1、用户登录界面

2、首页菜单

3、信息录入功能

4、信息浏览功能

5、信息查询功能（按照工号和学历查询）

6、信息修改功能（查询后对信息进行修改）

6、信息删除功能（查询后批量删除、单个删除）

7、离职管理（将已离职员工状态改为离职）

8、退出登录



实现对照表

| 页面功能     | 用户操作 (HTTP 请求)                  | Flask 的作用                        | Python 代码                                                  |
| :----------- | :------------------------------------ | :---------------------------------- | :----------------------------------------------------------- |
| **用户登录** | 浏览器 GET `/login`                   | Flask 路由将请求指向 `login()` 函数 | 返回 `login.html` 登录页面                                   |
|              | 浏览器 POST `/login` (提交账号密码)   | Flask 接收表单数据                  | 验证账号密码是否正确，记录登录状态                           |
| **首页菜单** | 浏览器 GET `/`                        | 路由指向 `index()` 函数             | 返回 `index.html` 首页，显示功能菜单                         |
| **录入职工** | 浏览器 GET `/add`                     | 路由指向 `add()` 函数               | 返回 `add.html` 录入表单页面                                 |
|              | 浏览器 POST `/add` (提交表单数据)     | Flask 接收表单数据                  | 将数据插入 SQLite/MySQL 数据库                               |
| **浏览职工** | 浏览器 GET `/list`                    | 路由指向 `list()` 函数              | 从数据库查询所有职工，返回 `list.html` 页面 (表格渲染)       |
| **查询职工** | 浏览器 GET `/search`                  | 路由指向 `search()` 函数            | 接收查询参数 (工号/学历)，从数据库查询，返回 `search.html` 页面 (表格渲染) |
| **修改职工** | 浏览器 GET `/update/<id>`             | 路由指向 `update()` 函数            | 根据 ID 查询职工信息，返回 `update.html` 页面 (表单回显)     |
|              | 浏览器 POST `/update/<id>` (提交修改) | Flask 接收表单数据                  | 更新数据库中对应记录                                         |
| **删除职工** | 浏览器 GET `/delete/<id>`             | 路由指向 `delete()` 函数            | 根据 ID 从数据库中删除记录                                   |
| **离职管理** | 浏览器 POST `/resign`                 | Flask 接收表单数据                  | 根据职工号更新状态为"离职"                                   |
| **退出登录** | 浏览器 GET `/logout`                  | 路由指向 `logout()` 函数            | 清除登录状态，跳转回登录页                                   |



## 2、MVC架构设计

1.技术选型

- Web框架：Flask
- 数据库：SQLite
- 数据库驱动：sqlite3(Python内置)



2.架构设计

MVC (Model-View-Controller)，将应用分为数据、展示和控制三个核心部件，使得业务逻辑、数据访问、界面展示完全解耦。

Model（模型）：数据层，负责与数据库交互，定义数据结构。

View（视图）：展示层，负责呈现用户界面（UI），通常是HTML页面。

Controller（控制）：控制层，接收用户请求，调用Model处理数据，再选择View来展示。

![image-20260630160626706](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260630160628503.png)

用户请求 → Controller (接收请求) → Model (查询数据库) → View (渲染HTML页面) → 用户看到页面



**MVC优势：**

| 优势       | 说明                                                         |
| :--------- | :----------------------------------------------------------- |
| 关注点分离 | 数据、界面、业务逻辑各司其职，代码清晰易读                   |
| 易于维护   | 修改数据库结构只需改 Model，调整页面只需改 View，互不影响    |
| 便于测试   | 可以单独对 Model 层编写单元测试，不依赖 Web 环境             |
| 支持复用   | Model 层可以被不同的 Controller 复用，比如报表导出、API 接口等 |
| 团队协作   | 前端可以专注开发 View（HTML/CSS），后端可以专注 Model 和 Controller |



项目目录结构：

```
employee_system/
├── app.py                 # 应用入口，启动服务器
├── config.py              # 配置文件
├── requirements.txt       # 项目依赖
│
├── models/                # 模型层 - 数据与数据库交互
│   ├── __init__.py
│   ├── employee_dao.py            # 职工数据访问
│   └── user_dao.py                # 用户数据访问
│
├── views/                 # 视图层 - 用户界面
│   ├── __init__.py
│   ├── templates/         # HTML模板文件
│   │   ├── base.html
│   │   ├── index.html
│   │   ├── login.html
│   │   ├── list.html
│   │   ├── add.html
│   │   ├── update.html
│   │   ├── search.html
│   │   └── resign.html
│   └── static/            # 静态资源 (CSS, JS, 图片)
│       └── style.css
│
├── controllers/           # 控制器层 - 路由与业务逻辑
│   ├── __init__.py
│   ├── auth_controller.py # 登录/登出
│   └── employee_controller.py # 职工管理
│
├── services/                      # Service层（业务逻辑）
│   ├── __init__.py
│   ├── employee_service.py        # 职工业务逻辑
│   └── auth_service.py            # 认证业务逻辑
│
├── core/                          # 基础设施层
│   ├── __init__.py
│   ├── database.py                # 数据库连接池 + 初始化
│   ├── response.py                # 统一响应格式
│   └── exceptions.py              # 自定义异常
│
├── middleware/                    # 中间件层
│   ├── __init__.py
│   └── auth_middleware.py         # 登录拦截装饰器
│
└── utils/                 # 工具函数
    ├── __init__.py
    └── validators.py      # 数据校验工具
```



调用链路：

```
用户请求
    ↓
middleware/auth_middleware.py   ← 登录拦截
    ↓
controllers/xxx_controller.py   ← 接收请求，解析参数
    ↓
services/xxx_service.py         ← 核心业务逻辑
    ↓
models/xxx_dao.py               ← 执行 SQL
    ↓
core/database.py                ← 数据库连接池
    ↓
返回响应 (views/templates/*.html)
```



## 3、项目搭建顺序

业务逻辑驱动开发，先专注这个功能做什么，然后逐步完善。







一个功能从后端到前端完整打通后，再写下一个功能。



## 4、用户登录界面

1.用户登录界面分析：

1、登录页面本身（HTML + CSS，用户能看到）

2、后端登录处理（接收账户密码，验证是否正确）

其他文件（数据库连接、DAO、Service）都是为了让“验证账号密码”这一步能工作。



1、构思业务逻辑：





