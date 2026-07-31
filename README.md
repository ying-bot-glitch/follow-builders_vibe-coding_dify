# Follow Builders & AI News Agent

这个仓库承载两个相关但独立的项目。**修改前请先确认自己在改哪一个**，两者共用同一个仓库，但 Workflow、数据、部署环境都是分开的。

---

## 🔗 线上地址

| 内容 | 地址 |
|---|---|
| **主站**（Follow Builders，公开访问） | https://ying-bot-glitch.github.io/follow-builders_vibe-coding_dify/ |
| **管理配置页**（AI News Agent，需要管理密钥） | https://ying-bot-glitch.github.io/follow-builders_vibe-coding_dify/admin.html |

管理配置页的入口藏在主站右下角一个不起眼的"内部管理"文字链接里，不会出现在主导航中，也不建议直接把链接分享给非管理人员。

---

## 📁 两个项目速览

### 1. Follow Builders（个人项目，已上线运行）

AI builders 每日推文 + 播客摘要订阅产品。每天早上 9 点自动生成中文日报，发给订阅者。

- 前端：本仓库根目录的 `index.html`
- 订阅数据：`subscribers.json`
- Dify Workflow：个人 Dify Cloud（dify.ai）上的「Follow Builders Daily Digest」+「Follow Builders Subscribe」
- 邮件发送：Power Automate（个人账号中转，非长期方案）
- 详细文档：`follow-builders-workflows-explained.md`

**这是独立运行的个人项目，不要在改 AI News Agent 时误动它的 Workflow 或数据文件。**

### 2. AI News Agent（部门项目，开发中）

部门级 AI 情报周报系统：抓取内外网 AI 资讯 → 打分 → 归类 → LLM 生成中文简报 → 推送 Outlook。

- 管理配置页：本仓库的 `admin.html`
- 配置文件：`config/` 目录下的 `recipients.json` / `schedule.json` / `sources.json` / `weights.json`
- Dify Workflow：目前在个人 Dify Cloud，**待搬迁到公司 Dify**（`agents.bosch-genai.com`）
- 邮件发送：部门共享邮箱账号，已测试成功
- **交接与开发状态**：见 `ai-news-agent-handoff.md`（完整背景、架构、已完成代码、待办事项）
- 更细粒度的任务清单：`ai-news-agent-next-steps.md`
- 需求匹配分析：`ai-news-agent-requirement-mapping.md`

**如果你是接手这个项目的人，先读 `ai-news-agent-handoff.md`，里面包含所有已验证代码和已知坑的完整记录。**

---

## 🗂️ 仓库文件结构

```
├── index.html                                  Follow Builders 主站
├── admin.html                                  AI News Agent 管理配置页
├── subscribers.json                            Follow Builders 订阅者数据
├── config/
│   ├── recipients.json                         AI News Agent 收件人清单
│   ├── schedule.json                           AI News Agent 推送频率配置
│   ├── sources.json                             AI News Agent 信息来源配置
│   └── weights.json                             AI News Agent 打分权重配置
├── ai-news-agent-handoff.md                     AI News Agent 完整交接文档 ⭐
├── ai-news-agent-next-steps.md                  AI News Agent 任务清单
├── ai-news-agent-requirement-mapping.md         需求差距分析
├── follow-builders-workflows-explained.md       Follow Builders 两个 Workflow 详解
├── follow-builders-progress.md                  Follow Builders Day 1 进度
├── follow-builders-progress-day2.md             Follow Builders Day 2 进度
├── follow-builders-company-dify-migration.md    公司 Dify 迁移验证指南
└── deck-1-product-intro.html / deck-2-tech-deep-dive.html   内部分享用 slides
```

---

## ⚙️ 外部服务依赖

| 服务 | 用途 | 归属项目 |
|---|---|---|
| GitHub Pages | 前端托管 | 两者共用（本仓库） |
| GitHub Contents API | 数据/配置文件的读写 | 两者共用 |
| Cloudflare Worker `dify-webhook-proxy` | Follow Builders 订阅请求的 CORS 代理 | Follow Builders |
| Cloudflare Worker `ai-news-agent-config-proxy` | AI News Agent 配置保存请求的 CORS 代理 | AI News Agent |
| 个人 Dify Cloud（dify.ai） | 当前两个项目的 Workflow 都跑在这里 | 两者共用（AI News Agent 待搬迁） |
| 公司 Dify（`agents.bosch-genai.com`） | 邮件发送测试、AI News Agent 未来的正式运行环境 | AI News Agent |
| Resend / Power Automate / 部门共享邮箱 | 邮件发送（历史上用过三种方案，详见交接文档） | 两者 |

---

## 🚧 当前已知的重要限制

- AI News Agent 的 Dify Workflow 尚未搬迁到公司环境，仍在个人 Dify Cloud 上
- AI News Agent 的 LLM 节点用的是 `deepseek-v4-pro`，公司 Dify 的模型计划里没有这个模型，搬迁时必须一并更换（详见 `ai-news-agent-handoff.md` 第十节）
- 公司 Dify 的 Webhook Trigger 节点，其 HTTP 响应内容是请求一到达就返回的固定文案，**不会**反映后续节点的真实执行结果——`admin.html` 的保存逻辑已经改成"提交后重新拉取配置文件自行比对"的模式来绕过这个限制
- 内网数据源尚未接入，等待内网页面访问方式确认

完整的已知问题和踩坑记录，见 `ai-news-agent-handoff.md` 第十一节。
