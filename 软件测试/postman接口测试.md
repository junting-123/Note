# Postman 接口测试

需求：

[登录-开源商城 | B2C商城 | B2B2C商城 | 三级分销 | 免费商城 | 多用户商城 | tpshop｜thinkphp shop｜TPshop 免费开源系统 | 微商城](https://hmshop-test.itheima.net/Home/user/login.html)

获取登录接口信息

postman完成登录正向和负向信息接口测试



## 1、F12进入开发者工具进行抓包

第一步：打开开发者工具并准备抓包

1. 打开浏览器：推荐使用 Chrome 或 Edge 浏览器。
2. 进入登录页面：访问 TP商城的登录页面（ `https://hmshop-test.itheima.net/Home/user/login.html`）。
3. 打开开发者工具：在键盘上按下 `F12` 键。
4. 切换到“网络”(Network)标签页：在开发者工具顶部菜单栏中，点击 `Network`（网络）选项卡。
5. 保留日志（重要）：勾选 `Preserve log`（保留日志）选项，防止页面跳转后请求记录被清空。

![image-20260621233131636](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260621233132928.png)



第二步：抓包登录

​	1. 输入正确的信息

​		用户名：13012345678

​		密码：123456

​		验证码：8888

	2. 点击“登录”按钮
	3. 立即查看Network面板：此时列表中会出现新的请求，其中就会包含 `index.php?m=Home&c=User&a=do_login`

![image-20260621233546121](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260621233716702.png)

![image-20260621234227100](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260621234228596.png)

![image-20260621234448919](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260621234450117.png)

（1）标头页面

登录请求地址：https://hmshop-test.itheima.net/index.php?m=Home&c=User&a=do_login 

请求方法：POST

（2）负载页面

请求体参数：username:13012345678；password:123456；verify_code:8888

（3）响应页面

返回数据：{"status":-1,"msg":"\u8d26\u53f7\u4e0d\u5b58\u5728!"}



## 2、Postman正向接口测试

正向用例（成功登录）

1、打开 Postman 并新建请求

点击 + 或 New → HTTP Request



2、配置请求基本信息

按照你抓包得到的信息填写：

| 配置项             | 填写内容                                                     |
| :----------------- | :----------------------------------------------------------- |
| Method（请求方法） | 下拉选择 `POST`                                              |
| URL（请求地址）    | `https://hmshop-test.itheima.net/index.php?m=Home&c=User&a=do_login` |



3、配置请求体（Body）

因为你抓包看到的是 `username`、`password`、`verify_code` 这种键值对格式，所以：

| 操作                       | 说明                         |
| :------------------------- | :--------------------------- |
| 点击 Body 标签             | 在地址栏下方                 |
| 选择 x-www-form-urlencoded | 这是表单格式，适合键值对参数 |
| 添加三个键值对             | Key 和 Value 分别填写        |

```
Key              Value
─────────────────────────────────
username         13012345678
password         123456
verify_code      8888
```



4、添加断言（script脚本）

点击 Tests 标签（在 Body 旁边），输入以下 JavaScript 代码：

```javascript
// ============================================
// 1. 基础断言：验证HTTP状态码为200
// ============================================
pm.test("HTTP状态码为200", function () {
    pm.response.to.have.status(200);
});

// ============================================
// 2. 格式断言：验证响应体为合法的JSON格式
// ============================================
pm.test("响应体为合法JSON格式", function () {
    // 尝试解析JSON，如果失败会抛出异常
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an('object');
});

// ============================================
// 3. 业务状态码断言：验证status字段等于1
// ============================================
pm.test("业务状态码status = 1", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql(1);
});

// ============================================
// 4. 业务消息断言：验证msg字段内容为"登陆成功"
// ============================================
pm.test("业务消息msg = '登陆成功'", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.msg).to.eql("登陆成功");
});
```



5、点击send发送请求

- 点击右上角蓝色 Send 按钮
- 等待响应返回（通常几百毫秒）



6、查看响应和断言结果

**查看响应体（Response Body）**

在界面下半部分的 Body 标签中，你应该能看到：

```
{"status":1,"msg":"登陆成功"}
```

**查看断言结果（Test Results）**

点击 Test Results 标签（在 Body 旁边），你会看到：

```
✅ HTTP状态码为200
✅ 响应体为合法JSON格式
✅ 业务状态码status = 1
✅ 业务消息msg = '登陆成功'
```



## 3、Postman反向接口测试

反向用例

| 测试场景        | 预期 status | 预期 msg 包含 |
| --------------- | ----------- | ------------- |
| 反向-账号不存在 | -1          | "账号不存在"  |
| 反向-密码错误   | -2          | "密码错误"    |
| 反向-验证码错误 | 0           | "验证码错误"  |

修改配置请求体：

1、账号不存在

![image-20260622120924701](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260622120925916.png)



测试结果：
![image-20260622120945258](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260622120946113.png)



2、密码错误

![image-20260622121004379](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260622121005433.png)

测试结果

![image-20260622121016155](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260622121017042.png)



3、验证码错误

![image-20260622121033971](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260622121055529.png)

测试结果

![image-20260622121045156](https://cdn.jsdelivr.net/gh/junting-123/my-blog-images/img/20260622121051890.png)