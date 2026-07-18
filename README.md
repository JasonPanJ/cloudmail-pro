<p align="center">
    <img src="doc/demo/logo.png" width="80px" />
    <h1 align="center">CloudMail Pro</h1>
    <p align="center">经过生产验证的 Cloudflare 邮箱服务，支持 To、CC、BCC、附件收发与完整后台管理</p>
    <p align="center">
        简体中文 | <a href="/README-en.md" style="margin-left: 5px">English </a>
    </p>
    <p align="center">
        <a href="https://github.com/JasonPanJ/cloudmail-pro/blob/main/LICENSE" target="_blank" >
            <img src="https://img.shields.io/badge/license-MIT-green" />
        </a>    
        <a href="https://github.com/JasonPanJ/cloudmail-pro/issues" >
            <img src="https://img.shields.io/github/issues/JasonPanJ/cloudmail-pro" alt="issues" />
        </a>  
        <a href="https://github.com/JasonPanJ/cloudmail-pro/stargazers" target="_blank">
            <img src="https://img.shields.io/github/stars/JasonPanJ/cloudmail-pro" alt="stargazers" />
        </a>
    </p>
    <p align="center">
        <a href="https://trendshift.io/repositories/20459" target="_blank" >
            <img src="https://trendshift.io/api/badge/repositories/20459" alt="trendshift" >
        </a>
    </p>
</p>


## 项目简介

只需要一个域名，即可在 Cloudflare Workers 上搭建支持多邮箱账户的响应式邮件服务。CloudMail Pro 基于 [maillab/cloud-mail](https://github.com/maillab/cloud-mail) 的最新稳定代码，保留上游能力，并加入经过真实 Worker 部署验证的 CC/BCC 和存储绑定配置。

## Pro 版本特性

- **完整收件人模型**：写信、草稿、邮件详情和发送服务均支持 To、CC、BCC。
- **隐私保护**：内部投递时不会向收件人副本泄露 BCC 地址。
- **一致的收件人校验**：To、CC、BCC 去重并统一参与权限、配额、统计和 50 人上限检查。
- **双发送通道**：Cloudflare Email Sending 与 Resend 均支持 CC/BCC。
- **已验证的 Cloudflare 绑定**：复用现有 D1 与 KV，`keep_vars = true` 保留云端 `admin`、`domain`、`jwt_secret` 等变量。
- **上游兼容**：包含上游最新的空值处理、OAuth 状态和查询性能修复。

生产基线已使用 Worker 版本 `fcc450be-0cfb-42e0-951a-743ea9b3324d` 验证；前端构建、Worker 上传、D1/KV 访问和线上健康检查均通过。

三个相关项目的逐项功能、优缺点和安全性对比见 [项目对比报告](doc/PROJECT_COMPARISON.md)。

## 项目展示

- [在线演示](https://skymail.ink)<br>
- 部署方式：请按照下方图文流程操作。

| ![](/doc/demo/demo1.png) | ![](/doc/demo/demo2.png) |
|-----------------------|-----------------------|
| ![](/doc/demo/demo3.png) | ![](/doc/demo/demo4.png) |

## 图文部署指南

项目内置 GitHub Actions 工作流，可自动安装依赖、构建前端、创建或复用 D1/KV、部署 Cloudflare Worker，并初始化数据库。

### 1. Fork 项目仓库

打开 [JasonPanJ/cloudmail-pro](https://github.com/JasonPanJ/cloudmail-pro)，点击右上角 **Fork**，将项目复制到自己的 GitHub 账户。保留默认分支名称 `main`。

![Fork 项目仓库](mail-vue/public/image/deploy/fork-repository.svg)

### 2. 准备 Cloudflare 权限

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 在账户页面记录 **Account ID**。
3. 创建部署用 API Token，并授予 Workers Scripts、D1、KV 与账户设置相关权限。
4. 准备一个已经接入 Cloudflare 的邮箱域名。

![准备 Cloudflare 权限](mail-vue/public/image/deploy/cloudflare-access.svg)

### 3. 配置 GitHub Actions Secrets

进入 Fork 后的仓库，依次打开 **Settings → Secrets and variables → Actions → New repository secret**，添加以下变量：

| Secret | 是否必填 | 说明 |
| --- | :---: | --- |
| `CLOUDFLARE_API_TOKEN` | 是 | 上一步创建的 Cloudflare API Token |
| `CLOUDFLARE_ACCOUNT_ID` | 是 | Cloudflare Account ID |
| `DOMAIN` | 是 | 邮箱域名 JSON 数组，例如 `["example.com"]` |
| `ADMIN` | 是 | 管理员邮箱，例如 `admin@example.com` |
| `JWT_SECRET` | 是 | 随机长字符串，不要包含 `?`、`%`、`#`、`/`、`\` |
| `CUSTOM_DOMAIN` | 推荐 | Worker 自定义访问域名，例如 `mail.example.com` |
| `NAME` | 否 | Worker、D1 和 KV 名称，默认 `cloud-mail` |
| `D1_DATABASE_ID` | 否 | 留空时工作流自动创建或复用同名 D1 |
| `KV_NAMESPACE_ID` | 否 | 留空时工作流自动创建或复用同名 KV |
| `R2_BUCKET_NAME` | 否 | 需要收发附件时填写已有 R2 Bucket 名称 |

`JWT_SECRET`、API Token 等敏感信息只能放在 Secrets 中，不要写入 README、代码或公开日志。

![配置 GitHub Secrets](mail-vue/public/image/deploy/github-secrets.svg)

### 4. 运行自动部署

1. 打开仓库的 **Actions** 页面。
2. 选择 Cloudflare 部署工作流，点击 **Run workflow**。
3. 等待 **Build and Deploy** 与 **Initialize database** 全部成功。
4. 以后向 `main` 分支推送 `mail-vue` 或 `mail-worker` 修改时，工作流会自动重新部署。

![运行 GitHub Actions 部署](mail-vue/public/image/deploy/run-deployment.svg)

### 5. 配置域名与 Email Routing

1. 在 Cloudflare **Workers & Pages** 中，为 Worker 添加与 `CUSTOM_DOMAIN` 一致的自定义域名。
2. 打开 **Email → Email Routing** 并启用邮件路由。
3. 添加 **Catch-all** 规则，目标选择 **Send to a Worker**，然后选择刚部署的 Worker。
4. 访问自定义域名，测试管理员登录、账户注册与收信。

![配置 Email Routing](mail-vue/public/image/deploy/email-routing.svg)

> 如果数据库初始化步骤失败，请先确认 `CUSTOM_DOMAIN` 已正确解析到 Worker，再访问 `https://你的项目域名/api/init/你的JWT_SECRET` 手动初始化；成功响应为 `success`。




## 功能介绍

- **💰 低成本使用**： 可部署到 Cloudflare Workers 降低服务器成本

- **💻 响应式设计**：响应式布局自动适配PC和大部分手机端浏览器

- **📧 邮件发送**：集成Resend发送邮件，支持群发，内嵌图片和附件发送，发送状态查看

- **🛡️ 管理员功能**：可以对用户，邮件进行管理，RABC权限控制对功能及使用资源限制

- **📦 附件收发**：支持收发附件，使用R2对象存储保存和下载文件

- **🔔 邮件推送**：接收邮件后可以转发到TG机器人或其他服务商邮箱

- **📡 开放API**：支持使用API批量生成用户，多条件查询邮件 

- **🔢 验证码识别**：使用Workers AI，自动识别邮件验证码 

- **📈 数据可视化**：使用ECharts对系统数据详情，用户邮件增长可视化显示

- **🎨 个性化设置**：可以自定义网站标题，登录背景，透明度

- **🤖 人机验证**：集成Turnstile人机验证，防止人机批量注册

- **📜 更多功能**：正在开发中...



## 技术栈

- **平台**：[Cloudflare Workers](https://developers.cloudflare.com/workers/)

- **Web框架**：[Hono](https://hono.dev/)

- **ORM：**[Drizzle](https://orm.drizzle.team/)

- **前端框架**：[Vue3](https://vuejs.org/) 

- **UI框架**：[Element Plus](https://element-plus.org/) 

- **邮件推送：** [Resend](https://resend.com/)

- **缓存**：[Cloudflare KV](https://developers.cloudflare.com/kv/)

- **数据库**：[Cloudflare D1](https://developers.cloudflare.com/d1/)

- **文件存储**：[Cloudflare R2](https://developers.cloudflare.com/r2/)

## 目录结构

```
cloud-mail
├── mail-worker				    # worker后端项目
│   ├── src                  
│   │   ├── api	 			    # api接口层			
│   │   ├── const  			    # 项目常量
│   │   ├── dao                 # 数据访问层
│   │   ├── email			    # 邮件处理接收
│   │   ├── entity			    # 数据库实体
│   │   ├── error			    # 自定义异常
│   │   ├── hono			    # web框架配置、拦截器、全局异常等
│   │   ├── i18n			    # 语言国际化
│   │   ├── init			    # 数据库缓存初始化
│   │   ├── model			    # 响应体数据封装
│   │   ├── security			# 身份权限认证
│   │   ├── service			    # 业务服务层
│   │   ├── template			# 消息模板
│   │   ├── utils			    # 工具类
│   │   └── index.js			# 入口文件
│   ├── pageckge.json			# 项目依赖
│   └── wrangler.toml			# 项目配置
│
├── mail-vue				    # vue前端项目
│   ├── src
│   │   ├── axios 			    # axios配置
│   │   ├── components			# 自定义组件
│   │   ├── echarts			    # echarts组件导入
│   │   ├── i18n			    # 语言国际化
│   │   ├── init			    # 入站初始化
│   │   ├── layout			    # 主体布局组件
│   │   ├── perm			    # 权限认证
│   │   ├── request			    # api接口
│   │   ├── router			    # 路由配置
│   │   ├── store			    # 全局状态管理
│   │   ├── utils			    # 工具类
│   │   ├── views			    # 页面组件
│   │   ├── app.vue			    # 入口组件
│   │   ├── main.js			    # 入口js
│   │   └── style.css			# 全局css
│   ├── package.json			# 项目依赖
└── └── env.release				# 项目配置
```

## 赞助

如果 CloudMail Pro 对你有帮助，欢迎使用微信扫码赞赏，感谢支持项目持续维护。

<p align="center">
  <img width="320px" src="./doc/images/jason-wechat-appreciation.jpg" alt="CloudMail Pro 微信赞赏码">
</p>

## 许可证

本项目采用 [MIT](LICENSE) 许可证	


## 交流

[Telegram](https://t.me/cloud_mail_tg)



