你现在是一名顶级架构师 + 全栈工程师，请根据以下完整需求，生成一个适用于医院落地使用的**《医生年度报告可视化平台》**。该平台旨在帮助医生、科室主任及医院管理层展示和分析医生的工作绩效、患者数据、诊疗统计等信息。项目包含后端、管理端前端、用户端前端三部分。

你需要生成系统架构、数据库设计、API 设计、目录结构、前端方案与核心代码骨架。
请严格按照以下要求执行。

---

# 📌 **一、技术栈要求**

## ✔ 后端

* Python **3.13.9**
* Flask（现代结构）
* SQLAlchemy
* Flask-Migrate
* Marshmallow
* JWT
* SM4 加密（登录 & 修改密码必须强制）
* Redis（验证码等）
* 响应格式统一：

  ```
  { code, msg, data }
  ```

## ✔ 管理端前端

* React **19**
* Ant Design **v6**
* Redux Toolkit / Zustand
* React Router v7
* i18next 国际化（必须支持 zh-CN/英文，默认 zh-CN）
* 多主题切换（至少三套）
* 响应式布局
* RBAC 权限路由

## ✔ 用户端前端

* React 19
* 可视化库：ECharts / Ant Design Charts
* 可视化模板编辑器
* 模板 CRUD
* 数据导入与解析（Excel/CSV/JSON）
* 响应式布局

---

# 📌 **二、管理端登录方式（两大类）**

管理端**共有两大类登录方式**：

---

## **① 密码登录（必含密码 → SM4 加密）**

密码登录分两种互斥模式，由管理端后台配置决定：

### ✔ A. 账号 + 密码 + 短信验证码

前置要求：

1. 管理端选择短信登录模式
2. 发送短信验证码前必须完成**图片验证码（3种之一）验证**

流程：

```
账号 → 密码（SM4） → 选择图片验证码类型 → 验证通过 → 发送短信验证码 → 登录
```

### ✔ B. 账号 + 密码 + OTP

* OTP = 一次性密码，如 Google Authenticator
* 使用 TOTP（RFC 6238）
* 后端为每个管理员生成 otp_secret

配置示例：

```
LOGIN_METHOD = "sms"  # 或 otp
IMAGE_CAPTCHA_TYPE = "slider"
```

---

## **② 扫码登录**

扫码登录也是管理端的一种独立登录方式：

* 管理端可展示二维码用于扫码
* 可以是医院 App / 第三方 SSO / 微信扫码（由你生成方案）
* 扫码完成后由后端发放 JWT

后台可配置是否开启扫码登录：

```
ENABLE_QR_LOGIN = true
```

---

# 📌 **三、管理端用户字段要求（严格遵守）**

管理端 user 表必须包含以下字段：

| 字段名称            | 说明           |
| --------------- | ------------ |
| id              | 主键           |
| username        | 登录用户名        |
| password        | 密码（SM4 → 哈希） |
| password_salt   | 密码盐          |
| email           | 邮箱           |
| phone           | 手机号          |
| avatar          | 头像           |
| job_number      | 工号           |
| real_name       | 姓名           |
| nickname        | 昵称           |
| latest_login_at | 最近登录时间       |
| otp_secret      | OTP 密钥       |
| status          | 启用状态         |
| created_at      | 创建时间（必须此命名）  |
| updated_at      | 更新时间（必须此命名）  |

---

# 📌 **四、管理端前端显示昵称的规则**

右上角用户显示名称优先级：

```
nickname > real_name > username
```

右上角下拉菜单必须包含：

* 个人资料
* 修改密码
* 退出登录

并且顶部导航必须支持：

* 国际化切换（默认 zh-CN，第二个是英文）
* 网站主题切换（预设多套主题）

---

# 📌 **五、权限体系（RBAC）**

必须包含三张表，并且 **role 和 permission 必须同时拥有 code 与 name 字段**。

### ✔ 角色 role

字段包含：

* id
* code
* name
* description
* created_at
* updated_at

且必须有一个预置角色：

```
code = "super_admin"
```

并且：

* super_admin 不可编辑
* super_admin 不可删除
* super_admin 自动拥有所有权限

### ✔ 权限 permission

字段包含：

* id
* code
* name
* type（menu/button/api）
* parent_id（可选）
* created_at
* updated_at

### ✔ 关系表

* role_permission
* user_role

---

# 📌 **六、验证码体系（管理端密码登录中使用）**

管理端的“短信验证码登录”必须依赖图片验证码，支持三种类型：

1. 输入文字验证码
2. 点击顺序验证码
3. 滑块拼图验证码

后台可配置：

```
IMAGE_CAPTCHA_TYPE = "text" / "click" / "slider"
```

### 数据库表 image_captcha

* id
* captcha_type
* captcha_key
* captcha_content（JSON）
* expired_at
* created_at

### 数据库表 otp_sms（短信验证码）

* phone
* code
* used
* expired_at
* created_at

---

# 📌 **七、文件上传系统（管理端使用）**

支持三种存储方式（可扩展）：

* 本地（默认）
* 阿里云 OSS
* 七牛云 Kodo

### uploaded_file 表字段：

* file_name
* file_path
* file_url
* mime_type
* storage_type
* uploader_id
* file_size
* created_at
* updated_at

并需提供：

* 文件管理页面
* 权限访问控制
* 删除文件逻辑


---

# 📌 **八、业务功能模块要求（核心）**

用户端（医生/管理层视角）必须包含以下核心可视化模块：

### ✔ 1. 数据概览仪表板 (Dashboard)
* 年度工作总览（患者总数、诊疗次数、手术数量等）
* 关键指标 KPI 展示
* 趋势变化图表（同比/环比）

### ✔ 2. 患者分析模块
* 患者年龄/性别分布统计
* 复诊率与留存分析
* 病种/诊断分布分析

### ✔ 3. 诊疗统计模块
* 诊疗项目/手术统计
* 手术成功率与并发症分析
* 平均诊疗时长与药物使用统计

### ✔ 4. 绩效与导出
* 个人工作效率与质量评估
* 报告导出（PDF/Excel/图片）
* 数据脱敏处理（导出时自动隐藏敏感信息）

---

# 📌 **九、安全与隐私要求（HIPAA 标准）**

* **数据脱敏**：前端展示和导出时，必须对患者姓名、ID、电话进行脱敏处理（如：张**，138****0000）。
* **访问控制**：严格的科室/个人数据权限隔离。
* **数据加密**：敏感医疗数据在数据库中加密存储。

---

# 📌 **十、审计日志（医院要求）**

### login_log

* user_id
* login_type（password-sms / password-otp / qr）
* success
* ip
* user_agent
* created_at

### operation_log

自动记录：

* user_id
* url
* http_method
* request_params
* response_data
* ip
* created_at

---

# 📌 **十一、字段命名规范（严格遵守）**

凡是时间字段必须使用：

```
created_at
updated_at
```

其他命名一律不得替代。

---

# 📌 **十二、输出要求（你必须输出以下内容）**

请你输出：

---

## ✔（1）整体系统架构图

管理端 / 用户端 / 后端三部分结构。

## ✔（2）后端 Flask 目录结构（模块化拆分）

包含：登录、验证码、文件上传、权限、操作日志、用户管理、模板管理等。

## ✔（3）完整数据库建表 SQL（严格包含 created_at & updated_at）

全部表包括：

* user / role / permission
* user_role / role_permission
* login_log / operation_log
* image_captcha / otp_sms
* uploaded_file
* medical_dataset（上传的医疗数据源表）
* annual_report_template（年度报告模板表）
* report_instance_data（生成的具体报告数据表）

## ✔（4）全部 API 文档（RESTful + code/msg/data）

包括：

* 密码登录（短信 / OTP）
* 扫码登录
* 图片验证码
* 短信验证码
* 模板 CRUD
* 文件上传
* 权限管理
* 日志查询
* 用户管理

## ✔（5）管理端 React19 + Antd v6 工程结构

包含：

* 登录页（支持短信/OTP 两种模式）
* 扫码登录页
* 主布局 & 权限路由
* 主题切换
* 国际化
* 文件管理
* 角色&权限管理
* 用户管理
* 日志页面

## ✔（6）用户端 React19 工程结构

包含可视化模板编辑器、数据导入模块、以及核心业务组件（患者分析、诊疗统计、绩效评估等图表组件）、预览导出功能。
