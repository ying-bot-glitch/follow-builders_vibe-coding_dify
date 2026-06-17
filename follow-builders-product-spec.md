# Follow Builders — 产品框架 & UI 设计规范

> 本文档涵盖完整产品链路：前端页面结构、后端 Dify Workflow、邮件设计、UI 规范。供前端开发、Dify 配置、Vibe Coding 参考。

---

## 一、产品概述

**产品名**：Follow Builders
**核心定位**：追踪 26 位真正在构建 AI 产品的人（研究员、创始人、工程师），每天提炼他们的最新洞察，通过网站展示 + Outlook 邮件推送给订阅者。
**技术栈**：Vibe Coding（前端）+ Dify（AI Workflow + 邮件推送）
**数据来源**：[follow-builders](https://github.com/zarazhangrui/follow-builders) 仓库的公开 JSON feed，每日自动更新，无需 API Key

---

## 二、前端页面结构

### 页面总览

| 页面 | 路由 | 说明 |
|---|---|---|
| 日报首页 | `/` | 每日内容 feed，核心展示页 |
| 订阅页 | `/subscribe` | 填写邮箱 + 偏好，提交触发 Dify Webhook |

### 导航栏

```
follow builders                    日报  [订阅日报 →]
```

- `follow builders`：全小写 logo，左侧，Inter 500，Ink 色
- `日报`：文字链接，跳转 `/`
- `订阅日报 →`：唯一转化按钮，跳转 `/subscribe`，Accent 背景，`border-radius: 4px`
- 高度：52px，底边 `1px solid Border`，不吸顶

> 导航只设一个转化入口，去掉冗余的"订阅"文字链接——按钮已足够显眼，两个入口指向同一页面没有意义。

---

### 页面1：日报首页 `/`

#### Hero 区域

```
Follow builders,          ← Newsreader italic，36px，Ink
not influencers.

追踪 26 位真正在构建 AI   ← Inter，15px，Muted，行高 1.7
产品的人——研究员、创始
人、工程师——看他们在想
什么、做什么。

[订阅日报]  查看往期 →    ← 主按钮 Accent；次按钮纯文字链接
```

- 背景：Paper（`#F8F5EF`）
- 标题左对齐，`not influencers.` 刻意换行，形成节奏停顿

#### 筛选栏

```
全部  ·  X / Twitter  ·  播客  ·  博客
```

- 纯文字 + `·` 分隔，不用 pill 按钮
- 选中态：Accent 色 2px 下划线 + 字体 500 weight
- 悬停：文字色过渡到 Accent，`200ms ease`

#### 内容 Feed（左侧时间线 + 卡片列表）

**时间线（签名元素）**

```
┌──────┬─────────────────────────────────────┐
│ JUN  │                                     │
│      │  播客卡片                           │
│  17  │  推文卡片                           │
│      │  推文卡片                           │
│──────│─────────────────────────────────────│
│ JUN  │                                     │
│      │  推文卡片                           │
│  16  │  ...                                │
└──────┴─────────────────────────────────────┘
```

- 左栏宽度：48px 固定
- 月份缩写：JetBrains Mono 11px，旋转 -90°，Muted 色
- 日期数字：JetBrains Mono 18px，500 weight，Ink 色
- 左栏与内容区之间：`1px solid Border` 竖线

**推文卡片**

```
┌────────────────────────────────────────────┐
│ [AV]  Name · Role                   [TAG] │
│                                            │
│  正文摘要，2-3 句，14px Inter，行高 1.65  │
│  重点词用 500 weight，克制使用             │
│                                            │
│  原文链接 ↗                     6 小时前  │
└────────────────────────────────────────────┘
```

- Avatar：28px 圆形，无边框，背景用 Accent light，首字母缩写
- 时间戳：JetBrains Mono，11px，Muted
- 卡片间距：`gap: 1px`（极细缝隙，出版物密度感）
- hover：背景 `#FFF → #FAFAF8`，`120ms ease`，无投影

**播客卡片**

```
┌────────────────────────────────────────────┐
│  Latent Space                      [播客] │
│  ─────────────────────────────────────    │
│  "Why Agents Keep Failing"                │  ← Newsreader italic，16px
│                                            │
│  · 工具选择准确率超过15个后骤降至60%       │
│  · Eval 驱动开发正在取代直觉迭代          │
│  · 2026年 agent 框架将整合为 3-4 个赢家   │
│                                            │
│  收听原集 ↗                     8 小时前  │
└────────────────────────────────────────────┘
```

- 播客标题用 Newsreader italic，与推文卡片形成视觉层级区分
- Bullet 用 `·` 而非 `-`

---

### 页面2：订阅页 `/subscribe`

```
┌──────────────────────────────────────────────┐
│                                              │
│   每天 2 分钟，                              │  ← Newsreader italic，28px
│   追上 AI 最前线                             │
│                                              │
│   订阅后每天早 9 点收到邮件，AI 提炼         │
│   26 位顶级 builders 的最新洞察。            │
│                                              │
├──────────────────────────────────────────────┤
│  邮箱地址                                    │
│  ┌────────────────────────────────────────┐  │
│  │  you@company.com                       │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  关注话题（可多选）                          │
│  [AI 产品] [创业融资] [技术研究] [开源代码]  │
│                                              │
│  推送频率                                    │
│  [每日 ●]  [每周]                           │
│                                              │
│  [     免费订阅，无需信用卡 →     ]          │
│                                              │
└──────────────────────────────────────────────┘
```

- 表单最大宽度 `480px`，页面居中
- 话题：toggle chip，选中时 Accent light 背景 + Accent 文字 + Accent 边框
- 频率：segmented control，二选一
- 提交后：触发 Dify Webhook（见第三节），将邮箱 + 偏好写入 Dify 变量

---

## 三、后端 Dify Workflow

### 数据来源（无需 API Key）

| Feed 文件 | URL | 内容 |
|---|---|---|
| `feed-x.json` | `https://raw.githubusercontent.com/zarazhangrui/follow-builders/main/feed-x.json` | 26 位 builders 的推文，含 name / bio / text / url / likes |
| `feed-podcasts.json` | `https://raw.githubusercontent.com/zarazhangrui/follow-builders/main/feed-podcasts.json` | 播客最新集，含 transcript |
| `feed-blogs.json` | `https://raw.githubusercontent.com/zarazhangrui/follow-builders/main/feed-blogs.json` | Anthropic 官博文章 |

### Workflow A：每日内容生成（定时触发）

```
[定时触发器] 每天 09:00 Asia/Shanghai
    ↓
[HTTP 节点 A] GET feed-x.json
[HTTP 节点 B] GET feed-podcasts.json
    ↓
[代码节点] 裁剪数据
  · 推文：取前 10 位 builder，每人最多 3 条，过滤 likes < 5 的低质量推文
  · 播客：取最新 1 集，transcript 截断至 3000 字
  · 输出：builders_json, podcast_json
    ↓
[LLM 节点] 生成中文摘要
  · 模型：claude-sonnet-4-6 或 gpt-4o-mini
  · 输出：每位 builder 2-3 句摘要 + 播客 3 条 bullet + 今日一句话 insight
    ↓
        ├──→ [代码节点] 格式化为 JSON → 存入 Dify 变量 daily_digest
        │    （前端轮询此变量渲染页面）
        │
        └──→ [代码节点] 渲染 HTML 邮件模板
                  ↓
             [HTTP 节点] 读取订阅者列表（Dify 变量 subscribers）
                  ↓
             [循环节点] 逐一发送 Outlook 邮件
```

### Workflow B：订阅者注册（Webhook 触发）

```
[Webhook 触发] 前端表单提交
  · 参数：email, topics[], frequency
    ↓
[代码节点] 校验邮箱格式
    ↓
[Dify 变量节点] 追加到 subscribers 列表
  · 存储结构：[{email, topics, frequency, created_at}, ...]
    ↓
[HTTP 节点] 发送欢迎邮件（复用邮件模板，内容为当日最新摘要）
```

### Dify 变量说明

| 变量名 | 类型 | 内容 |
|---|---|---|
| `subscribers` | Array | 订阅者列表，含 email / topics / frequency |
| `daily_digest` | Object | 当日生成的结构化摘要，前端读取展示 |

> demo 阶段订阅者数据存 Dify 变量，够用。正式上线再迁移数据库。

### LLM 节点 Prompt

**System**
```
你是一位 AI 行业读物的编辑，负责把顶尖 builders 的推文和播客内容提炼成简洁的中文日报。
风格：克制、有洞察、像报纸文化版的语气，不用网络流行语，不过度感叹。
每条摘要控制在 2-3 句话。
```

**User**
```
今天是 {{sys.current_datetime}}。

以下是 AI builders 的最新推文：
{{builders_json}}

以下是最新播客：
{{podcast_json}}

请输出 JSON 格式：
{
  "date": "YYYY-MM-DD",
  "builders": [
    {
      "name": "...",
      "role": "...",
      "summary": "2-3句中文摘要",
      "url": "原文链接"
    }
  ],
  "podcast": {
    "show": "节目名",
    "title": "集标题",
    "bullets": ["要点1", "要点2", "要点3"],
    "url": "链接"
  },
  "insight": "今日一句话，点出今天内容的共同主题或最值得记住的判断。"
}
```

---

## 四、邮件设计

### 邮件结构

```
┌─────────────────────────────────────────────┐
│                                             │
│  follow builders          2026年6月17日     │  ← Header，Paper 底色
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│  PODCAST                                    │
│  Latent Space                               │
│  "Why Agents Keep Failing"                  │  ← 播客标题，serif
│                                             │
│  · 工具选择准确率超过15个后骤降至60%         │
│  · Eval 驱动正在取代直觉迭代                │
│  · 2026年框架将整合为 3-4 个赢家            │
│                                  收听原集 → │
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│  Aaron Levie · Box CEO                      │
│  大型企业正在把 token 预算当成新的运营成本  │
│  管理……                        查看原推 →  │
│                                             │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │
│                                             │
│  Zara Zhang · Builder                       │
│  用 Realtime 2 API 做了 YouTube 实时副驾   │
│  浏览器插件……                  查看原推 →  │
│                                             │
├─────────────────────────────────────────────┤
│                                             │
│  今日 insight                               │  ← 墨蓝左边框，Newsreader italic
│  "AI 的竞争正在从模型能力转移到             │
│   谁能把 token 花在刀刃上。"               │
│                                             │
├─────────────────────────────────────────────┤
│  退订  ·  修改偏好  ·  followbuilders.com   │  ← Footer，Muted 色小字
└─────────────────────────────────────────────┘
```

### 邮件样式规范

| 属性 | 值 |
|---|---|
| 背景色 | `#F8F5EF`（与网站 Paper 色一致） |
| 内容区最大宽度 | `600px`，居中 |
| 正文字体 | Georgia / Times New Roman（系统 serif 降级，邮件客户端不加载 web font） |
| 正文字号 | 15px，行高 1.7 |
| 链接色 | `#3B5BA5` |
| 分割线 | `1px solid #E2E0DA` |
| 编码 | 内联 CSS，无 JavaScript，兼容 Outlook 2016+ |

### 今日 insight 样式

```html
<div style="border-left: 3px solid #3B5BA5; padding: 12px 16px; margin: 24px 0;">
  <p style="font-style: italic; font-size: 15px; color: #1A1A1A; margin: 0;">
    "AI 的竞争正在从模型能力转移到谁能把 token 花在刀刃上。"
  </p>
</div>
```

### Footer

```
退订  ·  修改偏好  ·  followbuilders.com
```

- 字号：12px，颜色 Muted（`#6B6B6B`）
- `退订` 链接触发 Dify Webhook，从 subscribers 变量中移除该邮箱

---

## 五、UI 设计规范

### 设计方向

**核心气质**：人文报刊 / 文学副刊。不是科技产品，而是一份有编辑立场的读物——像《纽约书评》的版式克制、老派报纸文化副刊的排印讲究。参考 The Browser、Longreads 的阅读密度，配合暖纸色与斜体衬线，让人感觉"这里有人在选"。

**避免**：科技蓝 + 卡片堆砌的 SaaS 模板感；过度圆角；"AI 感"配色（霓虹紫 + 黑底）；任何让页面看起来像 dashboard 的元素。

**签名元素**：左侧 48px 竖排日期时间线，贯穿整个 feed 区域，视觉上像一份带时间戳的记者手账。

---

### 色彩

| 名称 | Hex | 用途 |
|---|---|---|
| Ink | `#1A1A1A` | 主文字、logo |
| Paper | `#F8F5EF` | 页面底色（旧纸暖色） |
| Surface | `#FFFFFF` | 卡片、输入框背景 |
| Accent | `#3B5BA5` | 主按钮、链接、选中态（收敛墨蓝） |
| Accent light | `#EBF0FA` | 标签背景、hover 填充 |
| Muted | `#6B6B6B` | 次要文字、时间戳、role 标签 |
| Border | `#E2E0DA` | 卡片边框、分割线（暖灰） |
| Tag X | `#FFF3E0` / `#7C4F00` | X/Twitter 内容标签 |
| Tag Pod | `#E8F5E9` / `#1B5E20` | 播客内容标签 |
| Tag Blog | `#EEF2FF` / `#1E3A8A` | 博客内容标签 |

---

### 字体

| 角色 | 字体 | 规格 | 用途 |
|---|---|---|---|
| Display | **Newsreader** *italic* | 32–40px | Hero 标题、章节标题、播客标题 |
| Body | **Inter** | 14px / 400 | 正文内容、UI 文字 |
| Mono | **JetBrains Mono** | 11–12px / 400 | 时间戳、统计数字、来源标签 |

> 三种字体均可从 Google Fonts 免费获取。Newsreader 斜体带明显手稿感，用在标题上立刻建立"这是一份读物"的感知；Inter 保持理性；Mono 强化时效性感知。

---

### 标签系统

| 内容类型 | 背景 | 文字 | 字体 |
|---|---|---|---|
| X / Twitter | `#FFF3E0` | `#7C4F00` | Mono 11px |
| 播客 | `#E8F5E9` | `#1B5E20` | Mono 11px |
| 博客 | `#EEF2FF` | `#1E3A8A` | Mono 11px |

- `border-radius: 3px`（接近方形，有出版物标注感）
- 位置：始终在卡片右上角

---

### 微交互

| 元素 | 交互 |
|---|---|
| 内容卡片 hover | 背景 `#FFF → #FAFAF8`，`120ms ease` |
| 筛选项 | 下划线从左向右展开，`200ms ease` |
| 订阅按钮 | `scale(0.98)` on active，无弹跳 |
| 页面加载 | 卡片从下 `8px` 淡入，错位 `50ms` delay（首屏限 3 张） |

> 时间线不做动画，它是静态的视觉锚点。

---

### 响应式

| 断点 | 变化 |
|---|---|
| `< 768px` | 左侧时间线隐藏，日期改为卡片内 mono 文字 |
| `< 480px` | Hero 标题缩小至 28px；筛选栏横向滚动 |
| 订阅页 `< 480px` | 话题 chip 改为 2 列 grid |

---

## 六、Vibe Coding Prompt

将以下内容作为完整 prompt 发给 Lovable / Cursor / v0：

```
Build a bilingual AI builders digest website with two pages.

PAGES:
- / (daily feed): hero + filter bar + content cards + left date timeline
- /subscribe: email subscription form

NAVIGATION (single bar, no sticky):
  left: "follow builders" (lowercase, Inter 500)
  right: "日报" text link | "订阅日报 →" button (Accent bg, radius 4px)
  Only ONE subscribe entry point — no duplicate text link.

DESIGN TOKENS:
  --color-paper: #F8F5EF      /* page background, warm newsprint */
  --color-surface: #FFFFFF    /* cards only */
  --color-ink: #1A1A1A        /* primary text */
  --color-accent: #3B5BA5     /* muted ink-blue — restrained, not electric */
  --color-accent-light: #EBF0FA
  --color-muted: #6B6B6B
  --color-border: #E2E0DA

TYPOGRAPHY:
  Display: Newsreader (Google Fonts), italic only — hero h1, podcast titles
  Body: Inter, 14px/400, line-height 1.65
  Mono: JetBrains Mono, 11-12px — timestamps, labels, tags

SIGNATURE LAYOUT — LEFT DATE TIMELINE:
  48px fixed column, left of feed area
  Contains: month abbreviation (11px mono, rotated -90°) + day number (18px mono 500)
  Separated from cards by 1px vertical rule in --color-border
  Groups cards by date. Hidden on mobile < 768px (show date inside card instead)

CARDS:
  gap: 1px between cards (not 12px) — dense editorial feel
  border: 1px solid --color-border, no shadow
  hover: background #FFF → #FAFAF8, 120ms ease
  Podcast card: title in Newsreader italic 16px, bullet points with · character
  Tweet card: 28px avatar circle, name + role, 2-3 line summary, mono timestamp

FILTER BAR:
  Plain text: 全部 · X / Twitter · 播客 · 博客
  Active state: 2px accent underline + weight 500. NO pill buttons.

SUBSCRIBE PAGE:
  Max width 480px centered
  Fields: email input, topic multi-select chips (AI产品/创业融资/技术研究/开源代码),
          frequency segmented control (每日/每周)
  Submit button full-width, label: "免费订阅，无需信用卡 →"
  On submit: POST to Dify webhook endpoint

AVOID: purple, rounded pills everywhere, white page background,
       emoji in UI, dashboard-style cards with colored headers, shadows.
```

---

## 七、文件说明

- 本文档为产品完整规范，前端 / Dify / 设计三端共用
- 字体均从 Google Fonts 免费获取：[Newsreader](https://fonts.google.com/specimen/Newsreader)、[Inter](https://fonts.google.com/specimen/Inter)、[JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono)
- 色值可直接用于 Tailwind `theme.extend.colors` 或 CSS variables
- Dify Workflow 无需任何外部 API Key（feed 数据公开，邮件发送用 Outlook SMTP）
- 签名元素（左侧时间线）是实现优先级最高的视觉特征，其余可适当简化
