# 知遇：从本地原型到多人可用的青年协作平台

## 项目链接

- 源码：[qinglian-connect](https://github.com/Zhang21901/qinglian-connect)
- 在线产品：[知遇](https://zhang21901.github.io/qinglian-connect/)

## 项目目标

知遇是面向青年社群的协作匹配平台。成员填写学校、专业、经历、技能、协作角色和可投入时间；平台提供成员目录、协作意向、邀请码和 AI 匹配，帮助成员找到合适的合作伙伴或可参与的项目。

## 值得优化的过程

### 1. 在开始前先定义“上线”

“本地可以打开”不等于“可以交给其他人使用”。项目一开始就应明确是否需要公网访问、用户注册、持久化数据、多人并发、管理员权限、预算限制和域名。

知遇最终采用了静态前端与云后端组合：GitHub Pages 托管页面，Supabase 提供认证、PostgreSQL、Storage 和 Edge Functions，Gemini 提供 AI 匹配能力。

### 2. 在开发前建立部署清单

实际发布过程涉及临时隧道警告页、邮件发送频率、Supabase 登录状态、Edge Function 配置和浏览器缓存。应在开发前确定一条完整的发布链路，并标记每项外部依赖的配置者和免费额度边界。

推荐把发布链路写成：

```text
GitHub Pages -> Supabase Auth / Database / Storage -> Edge Functions -> Gemini API
```

### 3. 数据库变更使用迁移文件

通过 Dashboard 直接执行 SQL 能快速修复线上问题，但容易造成“本地 schema”和线上状态不一致。后续应为每次结构变更创建按日期命名的 migration 文件，并通过固定流程执行。

知遇中已经沉淀了完整 schema：[supabase/schema.sql](https://github.com/Zhang21901/qinglian-connect/blob/main/supabase/schema.sql)。后续可从单一 schema 演进为版本化 migrations。

### 4. 云函数应自动部署

手动在 Supabase Dashboard 粘贴 Edge Function 时，曾发生旧代码与新代码拼接，造成函数启动失败，浏览器表现为 Edge Function 请求发送失败。

后续应使用 Supabase CLI 或 GitHub Actions 自动部署函数，避免人工复制粘贴。知遇的 AI 函数源码位于：[supabase/functions/ai-match/index.ts](https://github.com/Zhang21901/qinglian-connect/blob/main/supabase/functions/ai-match/index.ts)。

### 5. 功能验收要包含完整用户路径

以头像功能为例，不能只确认“上传控件出现”，还应覆盖：

- 合法 JPG、PNG、WebP 上传成功
- 超过 5MB 或非图片文件被拒绝
- 成员目录和协作意向能显示头像
- 用户只能写入自己的 Storage 目录
- 修改后头像缓存更新
- 手机端布局可用

## 可复用的工程经验

### 静态站点也可以是完整产品

前端使用静态站点不代表只有演示效果。通过托管平台和 BaaS，可以实现真实的认证、数据、权限、文件上传和服务端 AI 调用。

### 围绕成员身份设计数据模型

成员资料、协作意向、邀请码、每日 AI 用量和头像都应绑定到统一的用户 ID。这样能清晰实现权限控制、数据展示和后续扩展。

### AI 密钥必须放在服务端

知遇将 Gemini API Key 保存在 Supabase Edge Function Secret 中，而不是浏览器脚本中。AI 调用同时实施每日配额、失败退款、模型重试和错误提示。

### AI 输出需要为真实界面设计

Gemini 的思考过程曾挤占输出 token，导致回答在句中截断。修复方式包括降低思考预算、提高输出上限、控制回答长度，并要求固定结构和结束标记。

### 公开资料和私密信息分层

联系方式不应默认进入成员目录或 AI 上下文。知遇仅在成员主动选择公开时展示联系方式，AI 匹配只读取公开资料摘要。

### 缓存是发布的一部分

静态页面发布后，用户可能仍加载旧脚本。前端资源应使用版本参数或哈希文件名，例如：

```html
<script src="app.js?v=release-id"></script>
```

## 调试方法

当用户反馈“打不开”“看不到”“请求失败”时，应优先在线上真实复现，再根据浏览器网络、控制台、云函数日志和数据库响应定位问题，而不是仅凭代码推测。

知遇曾通过浏览器网络记录定位到 CORS 预检返回 `503 BOOT_ERROR`，最终确认是 Edge Function 的部署内容错误，而不是 Gemini Key 或用户登录状态。

## 可复用的项目启动提示词

```text
我要做一个面向真实用户使用的 Web 产品，不是本地演示。

请先完成：
1. 根据我的目标列出必须确认的产品需求。
2. 给出免费优先的技术架构、部署方式、账号体系、数据模型和成本边界。
3. 明确哪些配置必须由我在第三方平台完成。
4. 实现时同时完成前端、数据库、鉴权、权限策略、错误提示和线上部署。
5. 每次修改后都验证线上版本，不只验证本地。
6. 数据库结构变化必须写入 migration 文件；云函数避免手动粘贴，优先自动部署。
7. 每个用户功能给出可验证的验收场景。
8. 最后输出公开网址、管理员操作入口、部署说明和后续维护清单。
```

## 下一步改进

1. 为头像上传、AI 匹配和协作意向添加自动化回归测试。
2. 将 `schema.sql` 拆分为可追踪的数据库迁移文件。
3. 用 GitHub Actions 自动部署 Supabase Edge Functions。
4. 增加协作意向筛选、状态管理和成员资料详情页。
5. 增加管理员成员管理与内容治理能力。
