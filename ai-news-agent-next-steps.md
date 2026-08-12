# AI News Agent · 接下来要开发的工作清单

> 更新日期：2026-07-17
> 现状：配置页前端已做好，Dify 侧对应的处理逻辑还没搭，核心业务逻辑（打分、归类）尚未开始
> 使用方式：按优先级从上往下推进，🔴 标记的是阻塞其他工作的关键依赖，建议最先启动

---

## 阶段 0 · 外部依赖（进行中）

- [x] **申请部门公共邮箱 / 共享邮箱账号** —— ✅ 已拿到（`M-BDX-RT-CN.Communication@cn.bosch.com`，可直接用账号密码登录）
- [ ] 🟡 **MailGraph 插件绑定共享邮箱** —— 进行中，遇到新问题：
  - 用共享邮箱账号重新走了 MailGraph 授权流程，Dify 端 Test Run 显示 `SUCCESS`
  - 但 **实际邮件没有送达**收件邮箱（`ying.feng3@cn.bosch.com` 收件箱、垃圾邮件都还没确认查过）
  - **暂停排查，先跳过**——不是彻底放弃，是先去做其他不受阻塞的事，回头再啃。真正定位问题需要查两件事：① 共享邮箱自己的"已发送邮件"和"草稿箱"里有没有这封信（判断是插件只创建了草稿没真发送，还是真发了但卡在传输层）；② 如果已发送里有记录，需要找 IT 用 Message Trace 查邮件路由
- [ ] 确认部门 Dify 是否支持**原生 Schedule Trigger**（我们自己的项目验证过是有的，但需求文档写"没有"，需要在部门 Dify 实例上实测一次）
- [ ] 确认部门 Dify **是否真的部署在内网**（能不能访问内网页面/Docupedia），这是内网数据源能否做的硬前提
- [ ] 确认内网 AI 新闻页面、Docupedia 的具体 URL，以及要用什么方式认证（Cookie/Token/API）

---

## 阶段 1 · 把配置页跑通 ✅ 已完整闭环（2026-07-31 验证成功）

配置页从"只能读"升级到"能读能写、且写入结果真实可靠"，前端 → Cloudflare Worker → Dify Workflow C → GitHub 写入 → 前端核实，整条链路完整跑通。

### 1.1 Cloudflare Worker：配置代理 ✅

- [x] 新建 Worker `ai-news-agent-config-proxy`，CORS 代理 + 转发逻辑跟 `dify-webhook-proxy` 一致
- [x] **踩坑**：第一次搭好后，`DIFY_CONFIG_WEBHOOK_URL` 长期停留在占位符 `https://REPLACE-ME-IN-STEP-1.2`，导致请求根本没有到达 Dify，前端却收到一句看似正常的响应——这是最早误导排查方向的一个坑，回头查 Worker 源码才发现

### 1.2 Dify Workflow C：配置更新 ✅

节点结构：Webhook Trigger → Code（校验密钥 + 决定文件路径）→ IF/ELSE → True 分支（HTTP GET 读当前文件+SHA → Code 提取 SHA → HTTP PUT 写回 GitHub）/ False 分支（Template 返回错误信息）

- [x] 管理密钥存成 Dify 环境变量 `ADMIN_SECRET`，不硬编码在 Code 节点里
- [x] GitHub PAT 也存成环境变量 `GITHUB_PAT`（避免像最初那样明文写在 HTTP 节点 Header 里、随 DSL 导出而泄露）
- [x] **踩坑 1**：GitHub API 的 URL 一开始少了一个斜杠（`contents{{#file_path#}}` 应为 `contents/{{#file_path#}}`），导致 404
- [x] **踩坑 2（关键，公司 Dify 特有）**：公司这套 Dify（`agents.bosch-genai.com`）的 Webhook Trigger 节点，**HTTP 响应内容是请求一到达就立刻返回的固定文案**，跟后续节点是否真正执行成功完全无关——这是官方文档确认的设计（"When your workflow is successfully triggered... a default 200 OK response is sent back"），不是配置遗漏。不管密钥对错、写入成功与否，外部调用方收到的响应永远是同一句话。**这意味着前端不能依赖这个 HTTP 响应判断成功与否**，必须改用"提交后重新拉取配置文件自行比对"的方式确认（见 1.4）
- [x] **踩坑 3**：GitHub PAT 生成时，Permissions 列表里**忘记勾选 "Contents" 这一项**（只勾了 Actions + 必选的 Metadata），导致所有写入请求都报 403 "Resource not accessible by personal access token"。教训：生成 Fine-grained PAT 时，用 Permissions 面板的搜索框主动搜 "Contents" 并勾选，不要以为默认就有

### 1.3 初始化配置文件 ✅

- [x] `config/` 目录下 4 个 JSON 文件已创建（recipients / schedule / sources / weights），当前仍是占位数据，正式上线前需要换成真实内容

### 1.4 端到端测试 ✅

- [x] 密钥错误时，Dify 内部逻辑正确走 False 分支拒绝（用 Dify 的 **Workflow Logs**（不是编辑器里的 "Show Run History"）确认过 TRACING 里 CODE→IF/ELSE→TEMPLATE 全部正确执行）
- [x] **踩坑 4**：查运行记录时，一开始找错了地方——编辑器画布里 "Show Run History" 这个按钮，只显示在编辑器内手动 Test Run 产生的记录；**真实的外部 Webhook 调用记录，在应用左侧独立的 "Logs" 菜单里**（跟 "Orchestrate" "API Access" "Monitoring" 平级），两者是完全不同的两个列表，不要弄混
- [x] 密钥正确时，GitHub 上对应文件确实被更新，commit 历史里出现新记录
- [x] **踩坑 5**：前端"核实是否生效"这一步，最初写的比对函数有 bug——用 `JSON.stringify(obj, keys.sort())` 这种"数组形式 replacer"来试图排序 key，但这个 replacer 会**递归应用到所有嵌套层级**，导致 `entities`/`topics` 这些嵌套数组里的 `keyword`/`boost` 字段（不在最外层 key 名单里）被直接过滤掉，实际比较的是"剥空内容的空壳"，只要最外层字段名对得上就会误判为相等。**已修复为真正递归排序所有层级 key 再比较**（见 `admin.html` 里的 `canonicalize()` 函数）
- [x] **踩坑 6**：即使 Dify 和 GitHub 两端都已证实写入成功，前端有时依然会显示"未能确认"——原因是 `raw.githubusercontent.com` 的 CDN 缓存刷新有延迟，哪怕 URL 加了 cache-busting 查询参数，有时仍需要一二十秒才能反映最新内容。**已将核实逻辑的总等待时间从原来的 ~10 秒拉长到 ~25 秒（4 次重试，间隔递增），并加了 `cache: 'no-store'`**
- [x] 排查过程中给前端加了 `console.log` 调试输出（提交的 payload、每次拉取到的远端内容、每次比对结果），这套日志在最终定位问题时起了关键作用，建议保留，方便以后再出问题时快速自查

### 给后续维护者的关键提示

如果之后接手的人（人类或 AI agent）在这个 Workflow 上遇到"响应内容看起来不对但功能其实是好的"这类困惑，直接参考上面 6 个踩坑记录，不要重新从头排查——这些都是这次真实验证过的结论，不是猜测。

## 阶段 2 · 核心新业务逻辑（工作量最大的部分）

这是整个新需求里真正的技术难点，之前的项目完全没有涉及。

### 2.1 权重打分引擎 ✅ 已完成（独立测试 Workflow 中验证通过）

- [x] 新建 Code 节点，读取 `weights.json` 配置
- [x] 实现多维度打分：重点实体关键词、主题领域关键词、业务场景关键词、时效性加分、内网来源加成、负向过滤减分
- [x] 用 5 条手写测试数据验证，6 个维度全部命中过至少一次，分数计算跟手算完全吻合
- [ ] 时效性的"1 天内"和"3 天内"两档还没被测试数据覆盖到（测试数据最新的是 4 天前），建议后续用真实数据跑起来后留意一下这两档是否正常触发
- [ ] 具体分值目前是占位初始值，后续跟业务方一起调优（产品决策，不是技术问题）

### 2.2 板块归类逻辑 ✅ 已完成（独立测试 Workflow 中验证通过）

- [x] 新建 Code 节点，实现"每条资讯归入唯一板块"的规则
- [x] 实现"每个板块/子板块各自独立取 Top N"
- [x] **踩坑记录**：最初"实体关键词优先"的规则会被 Bosch 这类内网资讯里几乎必然出现的词误伤——一条明明该归"产品情报站"的内网资讯，因为提到了"Bosch"被错误分进"商业大事件"。修复方案：内网资讯跳过实体检测，直接从主题/场景关键词开始判断；外网资讯保持"实体 > 主题 > 场景"的原优先级
- [ ] **遗留问题**：拆开内外网规则后，"商业大事件/内部动态"这个板块现在没有任何触发条件了。如果要保留"内部真正重大事件"这个板块，需要业务方定义一套独立的"内部重大事件关键词"，不能沿用"提到 Bosch"这种过于普遍的判断
- [ ] 目前的算法逻辑跑在一个独立的"打分与归类测试" Workflow 里（Start 节点是 User Input，手动粘贴测试数据），还没有搬进正式的 Workflow A——等阶段 3.1.1（Workflow A 抓取环节重构）做完后，需要把这两个 Code 节点原样迁移过去

### 2.3 LLM Prompt 重写 ✅ 已完成（独立测试 Workflow 中验证通过）

- [x] 把 Generate Digest 节点的 Prompt 从"builders 扁平列表"改成"四大板块 + 子板块"的嵌套 JSON 结构
- [x] 输出字段包含 `report_section`（用板块 key 体现）、`weight_score`、板块内条目
- [x] 新增 `weekly_insight` 字段，要求跨板块提炼共同判断，不能只讲某一条
- [x] **踩坑记录（重要，务必留意）**：Thinking Mode 开关**两次设置后都没有真的生效**，LLM 输出仍然带 `<think>...</think>` 包裹。原因未完全定位——可能这次的设置入口点错了地方（部分 Dify 版本这个开关不在节点主面板，在模型名称旁的齿轮图标或"Model Parameters"里，跟 Temperature/Max Tokens 不在同一层级）。**没有花时间死磕这个开关**，改为下游 Code 节点做防御性清理（正则剥离 think 块 + 兜底提取 JSON），不管开关生不生效，流程都能正常跑。后续如果找到真正的开关位置，可以把这层防御性清理当作双保险保留，不用因为开关修好了就删掉这段代码
- [x] 新增 Parse Report Output 节点（防御性清理 + 提取 `sections_json` / `weekly_insight`），这个节点后续渲染邮件模板时会直接用到

### 2.4 邮件模板重写 ✅ 已完成（独立测试 Workflow 中验证通过，视觉效果已截图确认）

- [x] Render Weekly Email 节点：把 `sections_json` + `weekly_insight` 渲染成四板块表格样式的 HTML 邮件
- [x] 邮件主题格式改成 `【AI 情报周报】YYYY-MM-DD`
- [x] 板块顺序用固定数组控制（不依赖 JSON 对象天然顺序），没内容的板块自动跳过不留白
- [x] Footer 固化"仅供内部学习使用"分类标签，不用每次手动加
- [x] 用 Playwright 渲染成图片核对过视觉效果：暖纸色 + 墨蓝配色跟现有产品一致，表格版式跟 Follow Builders 的卡片式布局做了区分，"本周启发"用斜体 + 左边框强调，视觉重量合适

---

## 阶段 2 总结：核心算法链路已在独立测试 Workflow 中完整验证

完整链路（7 个节点，全部在"打分与归类测试" Workflow 里）：

```
User Input(测试数据) → Parse Test Items → Get Weights Config
  → Calculate Scores → Classify Sections → Generate Weekly Report
  → Parse Report Output → Render Weekly Email
```

**下一步不是继续加功能，而是"迁移"**：这 7 个节点目前跑在一个独立测试 Workflow 里，用的是手写测试数据。等阶段 3.1.1（Workflow A 抓取环节重构，从写死节点改成动态遍历 `sources.json`）完成后，需要把这 7 个 Code/LLM 节点原样搬进正式的 Workflow A，接到真实抓取的数据源后面，而不是接 User Input 手写数据。迁移时预计不需要改动节点内部逻辑，只需要重新连线 + 调整输入变量的来源节点。

---

## 阶段 3 · 新数据源与持久化

### 3.1 内网数据源接入（依赖阶段 0 的确认结果）

- [ ] 确认内网页面是否有 API（优先），还是只能爬 HTML
- [ ] 如果只有网页，建一个独立的"内网采集服务"（跑在内网机器上的小脚本），负责"访问页面 → 解析 → 输出干净 JSON"，Dify 只调用这个服务，不直接处理脏数据
- [ ] 新增 HTTP 节点接入这个内网数据源（或采集服务的接口）
- [ ] 内网内容按之前确认的"选择 A"方案处理：**只进邮件，不上网页**
- [ ] 🔴 **内网认证信息（Cookie/Token）绝不能存进 `config/sources.json`**——这个文件是通过 GitHub 公开 raw 读取的，任何人都能看到内容。认证信息必须单独存成 Dify 环境变量，在 Workflow 内部使用

### 3.1.1 Workflow A 的抓取环节需要重构 ✅ 全流程端到端已跑通（真实数据）

新建了独立 Workflow「AI News Agent - 抓取与生成」（没有动现有的 Follow Builders Workflow A，互不影响）：

- [x] 新增 HTTP 节点：读取 `config/sources.json`
- [x] 新增 Code 节点：过滤出 `enabled: true` 的来源
- [x] 新增 Iteration 节点：遍历每个启用的来源，`OUTPUT VARIABLES` 必须显式绑定内部节点要收集的字段（**踩坑**：这是必填项，不填 Iteration 不会往外传任何数据；`FLATTEN OUTPUT` 需要保持开启，否则输出是"数组套数组"而不是摊平的大数组）
- [x] Iteration 内部按 `parser_type` 分支处理：
  - `follow_builders_x` → 从嵌套的 builders/tweets 结构里摊平成独立条目，复用了 Follow Builders 项目原有的质量过滤（点赞≥5、文本≥30字）
  - `follow_builders_podcast` → 从播客列表结构提取，transcript 截到 300 字摘要
  - `generic_json_array` / `rss` / `html_page` → 代码已埋好分支，后两者尚未实现真实解析逻辑
- [x] 用真实数据跑通：26 条资讯抓取成功（25 条 X 推文 + 1 条播客）
- [x] **关键设计更正**：`source_type` 从"协议类型"改成"解析方式"命名，`config/sources.json` 和 `admin.html` 配置页已同步更新
- [x] **补齐 Get Weights Config 节点**，把之前独立测试 Workflow 里验证过的 5 个节点（Calculate Scores → Classify Sections → Generate Weekly Report → Parse Report Output → Render Weekly Email）接到真实抓取链路后面
- [x] **全流程端到端 Test Run 成功**：从 Schedule Trigger 到 Render Weekly Email，11 个节点全部绿勾，用真实的 26 条数据（而非手写测试数据）跑完整条链路
- [x] **踩坑记录**：从测试 Workflow 复制 LLM 节点配置到新 Workflow 时，`classified_json` 变量被错误地粘贴进了 Dify LLM 节点的 **CONTEXT** 区域（知识库/检索增强用的功能区），而不是 User Prompt 正文里，导致报错 `Invalid variable` 并卡住不动。**教训**：跨 Workflow 复制 LLM 节点配置时，CONTEXT 和 PROMPT 是两个独立区域，容易粘错位置，需要额外检查
- [ ] follow-builders 上游仓库只保留最近一天快照，真实数据的 `published_at` 全部是当天日期，"时效性打分"的 1天/3天/7天分档区分度不高——记录待观察，暂不处理
- [ ] 待查看真实数据下的板块分布情况（是否有板块完全空、是否有板块超过 Top N 需要截断验证）✅ **已用真实数据验证**：`产品情报站/主流模型` 触发了 Top N=5 截断，`商业大事件/内部动态`、`产品情报站/内部产品`、`AI应用与开发资源` 三个板块全空（符合预期，因为当前启用的数据源全部是外网、且没有条目只命中场景关键词）
- [x] **算法局限已修复并验证**：针对"纯关键词匹配分不清'提到了关键词'和'这是真的大事'"的问题，加了两层防御：
  1. **Calculate Scores 里的规则式初筛**：只命中 1 个关键词、且文本里没有"发布/推出/宣布/更新"等事件动词的资讯，判定为 `is_substantive: false`，分数打三折（不是直接排除，留有余地）
  2. **Generate Weekly Report 的 Prompt 追加兜底指令**：即使分数不低，如果内容空洞（员工个人感想、寒暄），LLM 生成简报时会主动跳过，不强行凑数
  - 用"OpenAI 员工说工作开心"（应被过滤）+ "Aaron Levie 多模型观点"（内容有价值但没有事件动词，不该被误伤）两个边界案例验证，结果符合预期：前者从 30 分降到 9 分且最终简报里消失，后者原样保留在"产品情报站/主流模型"板块
- [x] **调试踩坑记录（重要，以后会再遇到）**：Dify 单节点测试时，如果上游节点存在历史缓存（哪怕是很久以前跑的），会自动复用那份缓存喂给下游节点测试，不会提示"这是旧数据"。排查这次问题时，一度误以为是"变量丢失"或"LLM 记忆泄露"，反复检查了 Prompt 配置，最后才定位到真正原因：测试用的是几天前遗留的旧缓存。**教训**：改了上游节点代码后，先重新跑一次上游单节点测试产生新缓存，再测下游节点，不要想当然认为下游会自动用最新逻辑的结果

---

### 3.2 博客 Feed 补充

- [ ] follow-builders 仓库里还有一个 `feed-blogs.json` 之前没用上，评估是否要接入"产品情报站"板块

### 3.3 报告持久化存储

- [ ] 新增 HTTP 节点（跟 Write to GitHub 同一套机制），把每周生成的报告写入 `reports/YYYY-MM-DD.json` 和 `reports/YYYY-MM-DD.md`
- [ ] 这是为"后续多端交付"打基础，MVP 阶段先把写入逻辑跑通即可

---

## 里程碑：核心生成链路已经用真实数据完整跑通

从"读取配置的信息来源"到"生成可发送的周报 HTML"，全流程 11 个节点、真实的 26 条资讯数据，端到端验证成功。这是整个 AI News Agent 项目里工作量最大、风险最高的部分——现在已经从"理论上可行"变成"真实跑得通"。

剩下要做的主要是：外部依赖的推进（阶段 0：邮件账号审批）、配置页写入功能的联通（阶段 1：卡在 Cloudflare 仪表盘）、以及把这条已经跑通的生成链路正式接上定时发送和收件人逻辑（阶段 4）。核心算法和数据链路已经不是风险项了。

---

## 🎉 里程碑：端到端全链路第一次完整跑通（2026-08-12）

「AI News Agent - 抓取与生成」这个 Workflow，从 Schedule Trigger 到真实邮件送达收件人，**全部节点验证成功，两个真实收件人（`ying.feng3@cn.bosch.com`、`felix.zhang@cn.bosch.com`）都确认收到了周报邮件**。完整链路：

```
Schedule Trigger → Get Sources Config → Filter Enabled Sources
  → Iteration「Fetch Each Source」（抓取 + 按来源解析）
  → Get Weights Config → Calculate Scores → Classify Sections
  → Generate Weekly Report（LLM）→ Parse Report Output → Render Weekly Email
  → Get Recipients Config → Parse Recipients
  → Iteration「Send to Each Recipient」（Extract Recipient Email → Send Email via MailGraph）
```

### 休假前遗留的 4 个 bug，回来后全部修复验证完毕

- [x] `Get Recipients Config` 的 URL 从空值改成正确的 GitHub raw 地址
- [x] `Iteration 2` 的 `iterator_selector` 绑定到 `Parse Recipients / recipients_list`
- [x] `Extract Recipient Email` 的输入变量名从 `arg1` 改成 `recipient_item`
- [x] `Extract Recipient Email` 的代码语言从 Python3 改成 JavaScript

### 修复过程中新发现的坑（记录下来，以后可能再踩到）

- [x] **踩坑**：把 Code 节点的语言从 Python3 切到 JavaScript 时，Dify **把代码内容重置成了默认占位模板**（`function main({arg1, arg2}) { return {result: arg1+arg2} }`），把之前写好的业务逻辑覆盖掉了，OUTPUT VARIABLES 也跟着变成了默认的 `result` 字段。**教训**：切换 Code 节点语言前，先把代码内容复制保存到别处，切换后重新粘贴回去，不要假设内容会保留
- [x] **踩坑**：`Parse Recipients` 节点的输入变量名一度被留空（不是 `recipients_raw`），导致 `JSON.parse(undefined)` 报错。同一类"变量名对不上"的问题这次出现在了第三个不同的节点上，说明这类 bug 是这套 Dify 平台交互设计下容易反复出现的通病，改节点配置后建议养成习惯：改完变量绑定，退出面板重新进来看一眼，确认真的保存对了
- [x] **踩坑**：判断"节点到底成不成功"时，一开始看错了字段——`PROCESS DATA` 经常是空的 `{}`，这不代表节点失败，**真正该看的是 STATUS 和 OUTPUT/INPUT 区域**，不要被 PROCESS DATA 的空值误导
- [x] Iteration 节点外层显示 SUCCESS，不代表内部子节点都成功——`Continue on Error` 会让内部失败被"吞掉"，外层照样绿勾。**必须点开 Iteration 内部展开的子流程，才能看到每次循环真实的执行状态**

---

## 阶段 4 · 推送节奏与收件人模型调整

### 4.1 频率切换

- [ ] Schedule Trigger 从每日改成每周五（`send_day: 4`），具体时间从 `schedule.json` 读取（而不是写死在节点配置里）
- [ ] 如果阶段 0 确认部门 Dify 没有原生 Schedule Trigger，改用 Power Automate 的 **Recurrence 触发器**做外部定时器，调用 Dify 的 Workflow API

### 4.2 收件人数据源切换

- [ ] 决定：Workflow A 的 Filter Subscribers 节点，是继续读 `subscribers.json`（自助订阅模式），还是切换成读配置页管理的 `recipients.json`（静态配置模式），还是两个并行
- [ ] 如果切换，Filter 节点的过滤条件要跟着改（从"daily/weekly 偏好"改成"是否在 recipients.json 里"）

---

## 阶段 6 · 每周汇总存档机制（进行中，2026-08-12 开始搭建）

### 需求背景

之前每次生成的周报，内容其实只是"抓取那一刻能看到的最新数据"——因为上游 `follow-builders` 仓库的 `feed-x.json`/`feed-podcasts.json` 每天会被覆盖，不保留历史。要做到真正的"周汇总"，需要自己每天存档，发送那天合并过去 7 天的存档。

### 整体设计

```
Schedule Trigger（后续要改成每天触发，不是只在周五）
  ↓
Get Sources Config → Filter Enabled Sources → Iteration「Fetch Each Source」
  ↓（拿到今天抓到的原始资讯）
【新增，搭建中】存档今天的数据到 GitHub
  ↓
【待搭建】判断今天是不是发送日（读 schedule.json 的 send_day）
  ↓
IF/ELSE
  ├─ 是发送日 →【待搭建】合并过去 7 天的存档，去重
  │              ↓
  │            Get Weights Config → ... → Send Email via MailGraph（原有流程接上合并后的数据）
  └─ 不是 → 到这里结束，今天只存档
```

### 已搭建的存档部分（6 个新节点，插在 Iteration 之后、Get Weights Config 之前）

```
Prepare Archive Path（Code 10）
   ↓
Check Archive Exists（HTTP Request 5，GET）
   ↓
Prepare Archive SHA（Code 11）
   ↓
Prepare Archive Content（Code 12）
   ↓
Prepare Archive Request Body（Code 13）
   ↓
Write Archive To GitHub（HTTP Request 6，PUT）
```

存档路径规则：`archive/raw-items/YYYY-MM-DD.json`，每天一个文件，内容是 `{date, item_count, items}`。

#### 各节点代码（已验证正确的部分）

**Prepare Archive Path**：
```javascript
function main() {
  const now = new Date();
  const dateStr = now.getFullYear() + '-' +
    String(now.getMonth() + 1).padStart(2, '0') + '-' +
    String(now.getDate()).padStart(2, '0');
  return {
    today_date: dateStr,
    archive_path: `archive/raw-items/${dateStr}.json`
  };
}
```
输出变量：`today_date`（String）、`archive_path`（String）—— **两个都要声明，漏一个会报 "Not all output parameters are validated"**

**Check Archive Exists**（HTTP GET）：
```
URL: https://api.github.com/repos/ying-bot-glitch/follow-builders_vibe-coding_dify/contents/{{#prepare_archive_path.archive_path#}}
Headers: Authorization: Bearer {{#env.GITHUB_PAT#}}, Accept: application/vnd.github.v3+json
```

**Prepare Archive SHA**：
```javascript
function main({ check_status, check_body }) {
  if (check_status === 200) {
    try {
      const data = JSON.parse(check_body);
      return { file_exists: true, existing_sha: data.sha || '' };
    } catch (e) {
      return { file_exists: false, existing_sha: '' };
    }
  }
  return { file_exists: false, existing_sha: '' };
}
```
输出变量：`file_exists`（Boolean）、`existing_sha`（String）—— **同样要两个都声明**
输入变量：`check_status` ← Check Archive Exists / status_code，`check_body` ← Check Archive Exists / body

**Prepare Archive Content**：
```javascript
function main({ items, today_date }) {
  const archiveData = { date: today_date, item_count: items.length, items: items };
  const jsonStr = JSON.stringify(archiveData, null, 2);
  return { archive_content_base64: Buffer.from(jsonStr).toString('base64') };
}
```
输入变量：`items` ← Iteration「Fetch Each Source」的汇总输出，`today_date` ← Prepare Archive Path / today_date
输出变量：`archive_content_base64`（String）

**Prepare Archive Request Body**：
```javascript
function main({ file_exists, existing_sha, content_base64, today_date }) {
  const body = { message: `Archive raw items for ${today_date}`, content: content_base64, branch: 'main' };
  if (file_exists && existing_sha) { body.sha = existing_sha; }
  return { request_body_json: JSON.stringify(body) };
}
```
4 个输入变量分别来自 3 个不同节点，容易搞混，注意对照：
- `file_exists` ← Prepare Archive SHA / file_exists
- `existing_sha` ← Prepare Archive SHA / existing_sha
- `content_base64` ← Prepare Archive Content / archive_content_base64
- `today_date` ← Prepare Archive Path / today_date

**Write Archive To GitHub**（HTTP PUT）：
```
URL: https://api.github.com/repos/ying-bot-glitch/follow-builders_vibe-coding_dify/contents/{{#prepare_archive_path.archive_path#}}
Headers: 同 Check Archive Exists
Body 类型: raw（或者只放一个变量引用的 JSON 类型）
Body 内容: {{#prepare_archive_request_body.request_body_json#}}
```

### 这次搭建过程中踩的坑（都是之前反复出现过的同一类问题，再次印证要每加一个节点就单独测）

- Code 节点 `return` 了几个字段，OUTPUT VARIABLES 就要声明几个，少一个就报 "Not all output parameters are validated"——这次在 `Prepare Archive Path` 和 `Prepare Archive SHA` 上各犯了一次
- `Prepare Archive Request Body` 的 4 个输入变量，一开始有一个的 `value_selector` 是空的（只是加了变量名，没有真正点选来源），导致"变量选择器里找不到"——其实变量能被找到，是那一行本身没绑定过，不是真的丢失
- `Check Archive Exists` 和 `Write Archive To GitHub` 的 URL，`contents` 后面一开始都少了斜杠（`contentsarchive/...` 而不是 `contents/archive/...`）——跟很久以前配置更新 Workflow 里踩过的一模一样的坑

### ⏳ 当前卡住的地方（未完成，下次接着改）

**`Check Archive Exists` 节点的 ERROR HANDLING 设置是 `None`**——这导致 404（文件不存在，这是完全正常预期的情况）被当成"节点执行失败"，直接把整个 Workflow 终止了，根本走不到后面的 `Prepare Archive SHA` 去正确处理这个 404。

**已经确认的解决方法（三个选项里选哪个）**：这套 Dify 的 ERROR HANDLING 有三个选项：
- `None`（当前值，出错直接停）
- `Default Value`（出错时给固定默认输出，然后继续往下走）✅ **要选这个**
- `Fail Branch`（出错走专门的异常分支，这次用不上）

**下次接上时的第一件事**：把 `Check Archive Exists` 的 ERROR HANDLING 改成 `Default Value`，配置默认输出：
| 字段 | 默认值 |
|---|---|
| `status_code` | `404` |
| `body` | `{}` |

（不需要精确模拟真实 404 响应，因为 `Prepare Archive SHA` 的代码逻辑是"只要 status 不是 200 就判定文件不存在"，随便给个非 200 的数字即可）

改完后重新触发测试，确认这次能顺利走完 `Check Archive Exists` → `Prepare Archive SHA` → ... → `Write Archive To GitHub`，并且去 GitHub 上确认 `archive/raw-items/2026-08-12.json` 真的出现了。

**⚠️ 注意**：`Write Archive To GitHub` 不要做同样的 Default Value 设置——这是最终写入步骤，真失败了应该让它明确报错，不要悄悄跳过。

### 🐛 本次顺带发现的一个独立新 bug（跟存档无关，还没修）

`Iteration「Fetch Each Source」`报错：

```
The length of output variable `parsed_items` must be less than 30 elements.
```

这次触发抓到的资讯数量超过了 30 条，撞到了 Dify 对某个输出变量的长度限制。推测需要在 `Parse Source Content` 节点（负责把 X feed / 播客 feed 解析成扁平数组的那个节点）末尾加一个数量截断（比如 `.slice(0, N)`），但需要先确认：
- 这个 30 条限制具体是哪个变量触发的（是 Iteration 汇总后的 `parsed_items` 本身，还是内部某个中间变量）
- 截断会不会误伤真正该保留的高质量资讯（截断逻辑最好放在打分之后，只保留分数最高的 N 条，而不是简单粗暴地按抓取顺序砍掉后面的）

这个问题还没开始修，属于新发现，优先级排在"存档功能修完"之后。

### 存档功能修完之后，还要做的（大头工作，尚未开始）

- [ ] Schedule Trigger 改成每天触发（当前是配置成 Daily 测试用，还没有正式改成"每天存档+周五发送"的组合逻辑）
- [ ] 新增"判断今天是不是发送日"的节点（读 `schedule.json` 的 `send_day`）
- [ ] 新增 IF/ELSE：是发送日 → 走生成发送；不是 → 流程到存档为止结束
- [ ] 新增"合并过去 7 天存档"的节点：读 7 个 `archive/raw-items/YYYY-MM-DD.json` 文件（注意有的日期可能没有存档文件，要能容忍缺失，不能整体失败），合并后按 `url` 字段去重，替换掉原来单日的 `items`，喂给 `Get Weights Config` 之后的打分归类流程
- [ ] 合并去重逻辑要考虑：某天 Workflow 没跑成功导致存档缺失，该怎么处理（读不到就跳过那天，不要报错中断）

---

## 阶段 5 · 邮件发送方案转正 ✅ 已完成（提前于原计划）

- [x] 部门共享邮箱账号申请下来，MailGraph 插件绑定成功
- [x] 「AI News Agent - 抓取与生成」的邮件发送节点，直接就是用 MailGraph 搭建的（不是先用 Power Automate 过渡再切换，跳过了这一步）
- [ ] Follow Builders 的 Workflow A/B 仍然停留在 Power Automate（个人账号）/ Resend，还没有切换成同样的 MailGraph 共享邮箱方案——这是个可以顺手做的优化，技术上是把现成的 `Send Email via MailGraph` 节点配置复制过去，替换掉原来的 HTTP 节点

---

## 优先级排序建议（如果要一次只做一件事）

1. **阶段 0 的邮件审批** —— 现在就发起，因为等待时间不可控，越早启动越好
2. **阶段 1 配置页跑通** —— 已经做了前端，趁热把 Dify 侧接上，形成完整闭环
3. **阶段 2 打分与归类** —— 这是整个新需求的核心价值所在，建议投入最多时间打磨
4. 阶段 3、4、5 —— 视阶段 0 的确认结果决定顺序，有些项要等外部依赖才能开始

---

## 一句话总结

**配置页只是"最后一公里"的管理界面，真正决定这个项目能不能立起来的，是阶段 0 的两个外部依赖（邮件共享账号审批、内网访问确认）和阶段 2 的两个核心算法（打分、归类）。建议这周先把审批流程发出去，同时开始写打分引擎的 Code 节点——这两件事互不依赖，可以同时推进。**
