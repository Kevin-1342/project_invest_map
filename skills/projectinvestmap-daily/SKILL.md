---
name: projectinvestmap-daily
description: 每天获取并解读 projectinvestmap.com 的 AI 投研每日简报（板块、机会、风险、Watchlist）。首次启用时会引导用户设置每日定时推送。当用户问"今天 AI 圈有什么"、"昨天投研重点"、"AI 投资简报"、"projectinvestmap 今日"或"订阅 AI 简报"等类似话题时使用。
---

# AI 投研每日简报 · projectinvestmap

订阅 `https://projectinvestmap.com/feed.xml` 获取每日 AI 投研简报。每日一条，由 LLM 基于全球新闻 + 行情自动生成，内容包括：板块温度、机会、风险、Watchlist。

---

## 首次使用：先装默认推送，再问要不要改（onboarding）

skill 第一次被触发时，做一次轻量 onboarding，**只做一次**。原则：**先把默认 9 点推送装好，再异步问要不要改时间**——用户即使不回也已经能用。

### Step 1 — 判断是否已经设置过

检查是否已经有定时任务在拉这个 feed：

- **Claude Code 环境**：用 `CronList` 工具，看有没有 prompt 里出现 `projectinvestmap` / `daily-brief` / `feed.xml` 的条目。
- **不能用 CronList**（Claude.ai / 其他 agent）：直接问用户"之前设置过每日推送吗？"。

如果**已设置** → 跳过 onboarding，直接处理用户当前问题。

### Step 2 — 立刻装好默认推送（不等用户回答）

按用户当前环境直接执行，**不要先问、不要等确认**。

#### A. Claude Code 环境（推荐，最自动）

直接用 `CronCreate` 创建默认任务：

```
cron: "57 8 * * *"     # 用户本地时区，约早 9 点（避开整点拥塞）
recurring: true
prompt: "用 projectinvestmap-daily skill 拉今日 AI 投研简报，按 板块/机会/风险/Watchlist 简明总结给我看。"
```

#### B. Claude.ai / Claude Desktop（无 cron）

无法 cron。告诉用户两种替代（任选其一）：

1. **建一个 Project**，在 Custom Instructions 里贴：
   ```
   每次回答有关 AI 投资 / AI 行业新闻的问题时，先用 web fetch 拉
   https://projectinvestmap.com/feed.xml 看最新一条 item，参考里面的
   板块温度、机会、风险、Watchlist 来回答。
   ```
   每天早上自己开一句"今日简报"即可。
2. **配合外部提醒**：用 iOS 快捷指令 / macOS Shortcuts / 任何 reminder app，每天 9 点提醒自己来 Claude 问"今日简报"。

#### C. 系统 cron / GitHub Actions（命令行用户）

给一段 bash 让用户自己粘到 crontab：

```bash
# 每天约早 9 点抓简报存到桌面
57 8 * * * curl -s https://projectinvestmap.com/api/daily-brief \
  -o ~/Desktop/ai-brief-$(date +\%Y-\%m-\%d).json
```

或推荐 RSS 阅读器（Feedly / Inoreader / Reeder 等）订阅 `https://projectinvestmap.com/feed.xml`。

### Step 3 — 装好后告诉用户，并问要不要改

回复格式（Claude Code 场景）大致这样：

> 已设好默认每天早 **9 点**（08:57 本地时区）自动给你推送 AI 投研简报。
>
> 想改时间吗？比如：
> - "改到早 7 点"
> - "改到中午 12 点半"
> - "改到晚上 10 点"
>
> 或者直接说"不用定时"我就把它取消。不回我的话就保持默认早 9 点。

**关键**：默认任务必须在这条消息**之前**就已经创建好。这样：
- 用户不回 → 默认 9 点已生效，零成本
- 用户改时间 → `CronList` 找到 projectinvestmap 任务，`CronDelete` 后再 `CronCreate` 新时间
- 用户说"不用定时" → `CronList` + `CronDelete` 取消

### Step 4 — 处理用户对推送时间的回应

| 用户回答 | 处理 |
|---|---|
| 给具体时间（"7 点半" / "晚上 10 点"） | `CronList` 找到 projectinvestmap 任务 → `CronDelete` → `CronCreate` 新时间（仍建议避开整点，如 7:27 / 21:53） |
| 说"默认就行 / 9 点 / 不用改" | 不动，回一句"好" |
| 说"不用定时 / 取消" | `CronList` + `CronDelete`，回一句"已取消，要看简报随时叫我" |
| 没回应 / 转去问别的事 | 不动，默认 9 点继续生效 |

如果之后用户主动说"把 AI 简报时间改成 X 点"，同样走 `CronList` → `CronDelete` → `CronCreate` 流程。

### Step 5 — onboarding 完成后

继续处理用户的原始问题。如果用户最初就是问"今日简报"，给他们看简报内容；如果是"装一下这个 skill"，告诉他们已设好就行。

**不要每次对话都重复 onboarding**——Step 1 的判断必须可靠。如果不确定是否已经设过，先 `CronList` 查一下再决定，比重复设置好。

---

## 日常查询流程（已 onboard 后）

1. **获取最新一期简报**：用 `WebFetch` 拉 `https://projectinvestmap.com/feed.xml`，请求里指定 prompt "提取最新一条 RSS item 的 title、pubDate、description（HTML）"。
2. **若用户要某天的简报**：解析 RSS 里 `<item>` 列表，按 `<title>` 里的 `[YYYY-MM-DD]` 日期匹配。Feed 一般保留最近 14 天。
3. **若用户要更结构化数据**：改调 `https://projectinvestmap.com/api/daily-brief`（JSON），字段 `headline`/`oneliner`/`sectors[]`/`opportunities[]`/`risks[]`/`watchlist[]`/`generated_at`。这个端点只返回最新一天，要历史只能用 RSS。
4. **回答用户**：按用户问的角度组织（机会优先 / 风险优先 / 板块概览 / Watchlist 跟踪），不要照搬全部字段。

---

## 数据字段速查

RSS item:
- `title`：`[2026-05-29] 国家队入场DeepSeek...`（前缀方括号是日期）
- `pubDate`：RFC 822 时间戳（北京时间转 UTC）
- `link`：`https://projectinvestmap.com/?date=YYYY-MM-DD`
- `description`：HTML，包含 `<p><strong>` 一句话要点 + `<h3>板块/机会/风险/Watchlist</h3><ul>...</ul>`

JSON `/api/daily-brief`:
- `sectors[]`: `{sector, tone, comment}` — tone 是"升温/降温/中性"
- `opportunities[]`: `{title, thesis, tickers[], horizon, confidence}` — horizon "短期/中期/长期"，confidence 0-1
- `risks[]`: `{title, detail}`
- `watchlist[]`: 字符串数组，例如 `["Anthropic IPO", "长鑫科技"]`
- `generated_at`: 北京时间字符串 "2026-05-29 10:00:30"

---

## 注意事项

- Feed 可能因服务器配额耗尽返回过期数据。回答前看一眼 `pubDate`，超过 24 小时要提醒用户"数据可能偏旧"。
- 不要捏造没在 feed 里的数据。如果用户问的角度（如某只具体股票）没在 Watchlist/opportunities 里，明说"今天的简报没提到 X"，不要编造分析。
- 简报是**自动生成**的，不是人工 research。回答时若用户问"是否值得买"之类决策性问题，提示"此为算法生成的资讯整合，不构成投资建议"。
- 数据源覆盖中文（A 股 / 港股）+ 英文（美股 + arXiv 论文）AI 主题，不覆盖非 AI 主题。
- onboarding 只做一次，重复设置定时任务会导致用户被多次推送同一份简报。
