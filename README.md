# project_invest_map

[![website](https://img.shields.io/badge/web-projectinvestmap.com-1f6feb)](https://projectinvestmap.com)
[![rss](https://img.shields.io/badge/RSS-feed.xml-orange)](https://projectinvestmap.com/feed.xml)
[![license](https://img.shields.io/badge/license-MIT-green)](LICENSE)

> **AI Invest Map · 全球 AI 投研每日简报**
> 一张地图看全球 AI 公司、IPO、前沿论文与每日资金流向。每日由 LLM 自动生成投研简报，覆盖板块温度、机会、风险、Watchlist。

🌐 网站： **https://projectinvestmap.com**
📡 RSS：  **https://projectinvestmap.com/feed.xml**

本仓库**只**托管供他人订阅 / 集成的工具（Claude skill 等），不包含网站源码。

---

## 这个项目是什么

[projectinvestmap.com](https://projectinvestmap.com) 是一个聚合全球 AI 投研信息的网站：

- **地图视图**：全球 AI 公司、IPO 排队企业、独角兽按地理分布
- **每日简报**：每天一份 LLM 生成的 AI 投研简报，覆盖板块温度、机会、风险、Watchlist
- **行情面板**：A 股 / 港股 / 美股 AI 概念实时涨幅榜与主力资金流向
- **新闻与论文**：自动采集中英文 AI 新闻 + arXiv 前沿论文，去重、翻译、归类
- **数据更新频率**：行情每 30 分钟、新闻每小时、简报每日生成

数据源覆盖中文（A 股、港股、东方财富）+ 英文（美股、Hugging Face、arXiv）AI 主题。

---

## 为什么订阅

**给你的 AI agent 省 token、省架构。**

每天的投研简报由服务端 LLM 完成全部信息采集、去重、翻译、归类、生成——你的 agent 只需要拉一条 RSS 或调一次 API，直接拿到结构化结果，不用自己跑一遍同样的流程。

| 自己搭 | 订阅本项目 |
|--------|-----------|
| 每天消耗数万 token 抓取 + 摘要 + 分类 | 一次 API 调用，几百 token 读结果 |
| 需要维护多数据源爬虫 + prompt 链 | 零基础设施，开箱即用 |
| 数据源变动需要自己修 | 服务端持续维护，订阅方无感 |

适合场景：
- 个人 AI agent 每日晨报流程——直接消费结果，省掉信息收集环节
- 团队内部 bot 推送——一行 curl 接入，不用重复造轮子
- 研究者快速了解 AI 行业动态——RSS 阅读器订阅即可

> ⚠️ **注意：** 本项目为个人研究项目，订阅完全免费。所有内容均由 AI 自动生成，不具备任何投资参考价值，请勿据此做出投资决策。

---

## ⭐ 怎么用 · 复制下面的 Prompt 即可订阅 ⭐

### 🚀 一键订阅 — 复制下面这段 Prompt 到你的 AI agent 即可 🌟

> [!TIP]
> ### ✨ 想让 AI 每天早上自动推送 AI 投研简报？
> **复制下面整段 prompt → 贴到任何能联网的 AI agent 的系统指令里 → 完成订阅。**
>
> 🕘 默认每天北京时间早 9 点推送 · 🛠️ 不需要改字段 · 💻 不需要写代码

支持环境：Claude Code cron prompt、Claude.ai Project Custom Instructions、ChatGPT GPTs Instructions、Gemini Gem、Cursor、n8n / Zapier 工作流、自建 LLM API 应用。

📋 **复制下面 ⬇️ 整段 prompt：**

```
你是 AI 投研每日简报订阅助手。每次被触发时，按下面流程给我推送 projectinvestmap.com 的当日 AI 投研简报：

[首次启用：装好默认推送，再异步问要不要改]
你第一次被启用时，先帮我把每日定时推送装好，**默认北京时间早晨 9 点**（推荐避开整点拥塞，用 08:57 / 9:03 之类）。能用 cron / scheduler / 定时任务工具的环境（如 Claude Code 的 CronCreate、n8n 定时节点、系统 crontab）就直接创建定时任务；不能 cron 的环境（Claude.ai Project / GPTs / Gemini Gem 等），告诉我用外部 reminder（iOS 快捷指令 / macOS Shortcuts / 任意闹钟）每天早 9 点提醒自己来问"今日简报"。
装好之后告诉我："已默认每天北京时间早 9 点推送。想换时间随时说一声（比如'改到早 7 点'/'改到晚上 10 点'），不回我就保持 9 点。" 我**不回复你也要继续按 9 点推送**，回复了再按我说的时间改（cron 任务先删旧的再建新的）。

[推送渠道：确保我真的能收到]
推送必须发到我**当前正在和你对话的这个渠道**，或者我平时最常用、肯定会看到的渠道（例如这次对话所在的 Claude Code / Claude.ai 会话、ChatGPT 对话、Telegram / Slack / 微信 / 邮箱等——取决于你被部署在哪儿）。**不要**把简报只写进一个我不会主动打开的本地文件、临时缓存或后台日志里就当推送完成。如果你所在的环境无法主动推送（比如纯被动响应的 Project / GPT），第一次启用时就要明确告诉我："我这个环境没法主动推送，需要你每天早上来问我一句'今日简报'，或配合外部 reminder（iOS 快捷指令 / 闹钟 / cron + curl 等）触发我。"——并给出具体可粘贴的 reminder 配置示例。宁可让我手动触发，也不要假装推送了实际我收不到。

[强制规则]
1. 必须实时联网拉数据，禁止凭对话记忆 / 上下文里残留的旧简报作答。即便几分钟前刚拉过，也要重新发请求。
2. 优先调 https://projectinvestmap.com/api/daily-brief（JSON），读取顶层 `date` 和 `generated_at` 字段。
3. 如果上面这个 API 拉不到 / 我要历史某天，再调 https://projectinvestmap.com/feed.xml（RSS 2.0，含最近约 14 天）。

[日期校验：防止把旧数据冒充今日]
拿到数据后，先把数据里的 `date`（或 RSS 标题里 `[YYYY-MM-DD]` 前缀）跟"今日北京时间（UTC+8）"比对：
- 一致 → 正常按 板块/机会/风险/Watchlist 总结。
- 数据日期早于今日（哪怕只差 1 天）→ 必须先明确告诉我："今日（YYYY-MM-DD）简报尚未生成，最新一期为 X 月 X 日。" 然后再问要不要看那期内容，绝不把旧日期数据按"今日"格式硬塞给我。
- 简报通常在北京时间早 6-10 点之间出。北京时间 0-6 点之前，大概率今日还没生成，按"早于今日"处理。

[输出格式]
回复第一行带来源标识 `来源：projectinvestmap.com`（保留为可点击的纯域名，便于我打开网页看更多信息）。然后按下面四个区块组织（用户问的角度是哪个就突出哪个，不要照搬全部字段）：
- 板块温度（升温/降温/中性 + 一句点评）
- 机会（标题、逻辑、相关 ticker、时间维度、置信度）
- 风险（标题 + 细节）
- Watchlist（关键事件 / 公司清单）

末尾附一句数据日期，例如"——简报日期 2026-06-05，生成于 06:02 北京时间"。

[免责]
若我问"是否值得买"之类决策性问题，提示"此为算法生成的资讯整合，不构成投资建议"。不要捏造没在 feed 里的数据，没提到就直说没提到。
```

> 这段 prompt 自带"强制实时拉取 + 日期校验"硬规则，能避免 LLM 把上下文里残留的旧简报当成今日内容复述。

### 2. 直接打开网站

访问 [projectinvestmap.com](https://projectinvestmap.com)。

### 3. 订阅 RSS（推荐给 AI agent / 阅读器）

把这个 URL 贴到任何 RSS 阅读器或 AI agent 工作流里：

```
https://projectinvestmap.com/feed.xml
```

每天一条 item，含完整的板块温度、机会、风险、Watchlist。保留最近 14 天历史。

兼容客户端：Feedly、Inoreader、NetNewsWire、Reeder、Thunderbird、ChatGPT/Claude RSS 工具，等等。

### 4. 在 Claude 里用本仓库的 skill

让 Claude（Code 或 Claude.ai）每天自动帮你读简报。详见 [`skills/projectinvestmap-daily/`](./skills/projectinvestmap-daily/README.md)。

一行装到 Claude Code：

```bash
git clone https://github.com/Kevin-1342/project_invest_map.git
cp -r project_invest_map/skills/projectinvestmap-daily ~/.claude/skills/
```

下次开新会话，问"今天 AI 投研简报"即可触发。

### 5. 用 JSON API（适合自己写脚本）

```bash
curl -s https://projectinvestmap.com/api/daily-brief | jq
```

返回字段：`headline` / `oneliner` / `sectors[]` / `opportunities[]` / `risks[]` / `watchlist[]` / `generated_at`。字段说明见 skill 文档。

仅返回最新一天。要历史用 RSS。

---

## 仓库内容

```
project_invest_map/
├── LICENSE
├── README.md                              # 本文件
└── skills/
    └── projectinvestmap-daily/
        ├── SKILL.md                       # Claude skill 定义
        └── README.md                      # 安装与使用说明
```

未来可能会加更多 skill / 集成示例。

---

## 免责声明

> **本项目为个人研究项目，订阅完全免费。所有内容均由 AI 自动生成，不具备任何投资参考价值。**

- 网站简报由 LLM 自动生成，**不是人工 research**，也未经任何专业人士审核
- **不构成投资建议**——请勿据此做出任何投资决策
- 作者不对因使用本项目内容产生的任何损失承担责任
- 数据源覆盖 AI 主题，不覆盖非 AI 主题
- 服务可能因第三方数据源限额导致部分字段过期，请关注 `pubDate` / `generated_at`

---

## License

[MIT](LICENSE)
