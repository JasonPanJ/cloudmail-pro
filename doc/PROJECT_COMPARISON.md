# CloudMail 项目对比报告

更新时间：2026-07-17

## 对比对象

- [maillab/cloud-mail](https://github.com/maillab/cloud-mail)：官方上游项目。
- [JasonPanJ/cloud-mail](https://github.com/JasonPanJ/cloud-mail)：从官方项目 Fork、已加入 CC/BCC 并完成生产部署验证的版本。
- [zhuanwan0905/cloudmail-plus](https://github.com/zhuanwan0905/cloudmail-plus)：基于 Cloud Mail 代码快照增加 API Key 与 `/v1` Open API 的修改版。

三个项目均使用 MIT 许可证。`JasonPanJ/cloud-mail` 保留可追溯的上游 Git 历史；`cloudmail-plus` 是独立仓库，其 8 个提交主要由一次完整文件上传和后续修改组成，与上游没有共同 Git 祖先。

## 功能差异

| 能力 | maillab/cloud-mail | JasonPanJ/cloud-mail | cloudmail-plus |
| --- | --- | --- | --- |
| Cloudflare Workers + Hono + Vue 3 | 支持 | 支持 | 支持 |
| D1 / KV / R2 / Workers AI | 支持 | 支持；D1/KV 已配置并验证 | 支持；配置写死为作者账户资源 |
| Resend 与 Cloudflare Email Sending | 支持 | 支持 | 基于较旧快照支持 |
| To 群发 | 支持 | 支持 | 支持 |
| CC / BCC 写信、草稿、详情 | 不支持 | 完整支持 | 不支持 |
| CC / BCC 后端发送 | 不支持 | Cloudflare 与 Resend 均支持 | 不支持 |
| 内部 BCC 隐私处理 | 不支持 | 支持 | 不支持 |
| API 批量用户与邮件查询 | 支持原有开放 API | 支持原有开放 API | 支持原有 API，并新增 API Key `/v1` API |
| API Key 管理界面 | 不支持 | 不支持 | 支持，但实现存在安全与迁移问题 |
| 自动建库/升级 | 支持上游初始化逻辑 | 支持；已验证现有 D1 数据 | 未创建 `api_key` 表，干净部署会失败 |
| 上游最新修复 | 是 | 是，并保留自有功能 | 否；回退了部分最新修复 |
| 生产验证 | 官方演示与社区使用 | 本项目 Worker 实测通过 | 仓库未提供可复现验证证据 |

## 各项目优缺点

### maillab/cloud-mail

优点：

- 社区规模最大、更新活跃，问题与部署资料最丰富。
- 功能完整，覆盖收发件、附件、后台权限、邮件推送、Workers AI、分析和个性化。
- 上游最新版本修复了空值判断、OAuth 状态映射和顺序查询性能问题。

缺点：

- 写信界面与发送服务没有完整 CC/BCC 支持。
- 默认 `wrangler.toml` 的 D1/KV 是占位配置，首次部署仍需正确注入资源和秘密变量。
- 通用项目更新较快，自定义功能在 Sync Fork 时需要持续做兼容验证。

### JasonPanJ/cloud-mail

优点：

- 完整继承当前上游，同时增加端到端 CC/BCC。
- 对收件人做 To > CC > BCC 优先级去重，并统一执行权限、配额、统计和数量限制。
- 内部投递副本隐藏 BCC 列表，避免隐私泄露。
- 复用现有 D1/KV 和云端变量，已经完成构建、上传、100% 流量部署与线上健康检查。

缺点：

- 仍是官方项目的 Fork，GitHub 的 Sync Fork 容易让自定义变更的归属和发布状态变得不直观。
- `wrangler.toml` 中的 D1/KV ID 面向当前 Cloudflare 账户，不适合作为他人一键部署模板。
- GitHub Actions 仍需要仓库所有者配置长期有效的 Cloudflare API Token 才能自动部署。

### zhuanwan0905/cloudmail-plus

优点：

- 新增 API Key 管理页面和较完整的 `/v1` 邮箱、邮件 API 文档。
- API Key 绑定用户，主要邮件读取路径包含用户范围过滤，自动化方向清晰。
- API 文档包含 cURL 与 Python 示例，便于外部系统接入。

缺点与风险：

- 初始化代码没有创建 `api_key` 表，无法在干净 D1 上直接工作。
- API Key 列表和删除服务没有按当前用户隔离；任意已登录用户可能查看或删除其他用户的 Key。
- API Key 以明文存储并在列表接口返回，不符合生产密钥管理的最小暴露原则。
- 创建邮箱没有复用完整的账户数量、角色权限和资源配额校验。
- `wrangler.toml` 公开硬编码了个人域名、D1/KV ID、管理员地址和 JWT 密钥；该 JWT 密钥应立即轮换。
- 修改版没有 CC/BCC，且回退了上游对空值、OAuth 状态和并发查询的修复。
- 仓库历史不可直接追踪到上游提交，后续合并与安全审计成本更高。

## CloudMail Pro 的基线决策

CloudMail Pro 使用 `JasonPanJ/cloud-mail` 已验证代码树作为首个稳定基线：

1. 保留 `maillab/cloud-mail` 最新功能与修复。
2. 保留完整 CC/BCC、收件人去重和 BCC 隐私保护。
3. 保留已经验证的 D1/KV 绑定及 `keep_vars = true` 部署行为。
4. 不直接合并 `cloudmail-plus` 的 API Key 实现，避免把已识别的权限、密钥和数据库迁移问题带入生产。
5. 将安全版 API Key/Open API 作为后续独立功能开发：需要 Key 哈希存储、一次性显示、用户/管理员范围隔离、权限与配额复用、D1 迁移、撤销审计、速率限制和自动化测试。

## 版本依据

- 官方上游基线：`a6b66fc`。
- CC/BCC 功能提交：`40318d0`。
- D1/KV 配置提交：`d29d502`。
- 重建时 `JasonPanJ/cloud-mail/main`：`dcb8add`，其代码树与生产验证提交完全一致。
- 已验证 Worker 版本：`fcc450be-0cfb-42e0-951a-743ea9b3324d`。
