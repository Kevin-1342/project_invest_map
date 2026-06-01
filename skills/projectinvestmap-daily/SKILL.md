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
prompt: "用 projectinvestmap-daily skill 拉今日 AI 投研简报。强制流程：(1) 必须实时调用 WebFetch 或 curl，禁止凭印象/记忆作答；(2) 优先调 https://projectinvestmap.com/api/daily-brief，读 date 字段；(3) 把 date 与今日北京日期比对——一致则按 板块/机会/风险/Watchlist 总结；不一致则明确告诉我'今日（YYYY-MM-DD）简报尚未生成，最新一期为 X 月 X 日'，并附上那期内容供参考，绝不把旧日期数据冒充成今日。回复开头必须带来源标识 [ProjectInvestMap]。"
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

> ⚠️ **硬规则：每次回答必须实时联网拉数据，禁止凭对话记忆 / 上下文里残留的旧简报作答。** 即便几分钟前刚拉过，也要重拉一次——简报 1 天才更新一次但你不知道用户问的是哪一天。

### Step A — 实时拉取
1. 优先调 `https://projectinvestmap.com/api/daily-brief`（JSON，最快），拿到 `date` 和 `generated_at` 字段。
2. 如果用户要历史某天 / API 拿不到目标日期，再调 `https://projectinvestmap.com/feed.xml`（RSS，含最近 ~14 天）。
3. 在 Claude Code 环境里 WebFetch 被网络策略拦截时，用 Bash `curl -sS <url>` 兜底，不要因为 WebFetch 失败就放弃。

### Step B — 强制日期校验（防止把旧数据冒充今日）
拉到数据后**先做日期比对**，再决定怎么回答：

| 用户问的目标日期 | 拉到的最新 `date` | 处理 |
|---|---|---|
| 今日（北京时间） | 等于今日 | ✅ 正常按 板块/机会/风险/Watchlist 总结 |
| 今日 | 早于今日（哪怕只差 1 天） | ⚠️ **必须先说清楚**："今日（YYYY-MM-DD）简报尚未生成，最新一期是 X 月 X 日。" 然后再问要不要看那期内容，**不要直接把旧数据按"今日"格式输出**。 |
| 指定历史日期 | 在 RSS 范围内能匹配到 | ✅ 用对应那条 item 回答，标题里的 `[YYYY-MM-DD]` 必须等于用户要的日期 |
| 指定历史日期 | RSS 里找不到（>14 天 / 当天没出） | 明说没有，不要拿邻近日期顶替 |

判断"今日"用**北京时间**（CST, UTC+8），因为简报按北京日历生成。简报通常在北京时间早 6-10 点之间出。如果用户在北京时间凌晨 0-6 点问"今日简报"，大概率还没生成，按上表"早于今日"分支处理。

### Step C — 组织回答
按用户问的角度组织（机会优先 / 风险优先 / 板块概览 / Watchlist 跟踪），不要照搬全部字段。回答末尾附一句数据日期，例如"——简报日期 2026-06-01，生成于 06:02 北京时间"，让用户能一眼看出新鲜度。

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

## 推送来源标识

当本 skill 通过定时任务（cron）或被其他 agent 调用来推送消息时，**回复的第一行必须以 `[ProjectInvestMap]` 开头**，作为来源标识。格式示例：

> [ProjectInvestMap] 2026-05-29 AI 投研简报
>
> **板块温度** …

这样用户如果同时订阅了多个推送源，能一眼看出这条来自 ProjectInvestMap。

- 定时推送（cron prompt 触发）：必须带。
- 被其他 agent 通过 skill 调用：必须带。
- 用户在对话中主动问"今日简报"：可以带，也可以省略（用户已经知道上下文）。

---

## 注意事项

- **永远不要凭印象 / 历史对话里残留的简报作答**——每次都得实时拉。简报每天换一份，但 LLM 上下文里很容易残留某一天（比如 5/29）的内容，不重拉就会一直把那天复述给用户。
- **日期不匹配就明说**，不要硬把旧简报当今日用。Step B 表格是硬约束。
- Feed/API 可能因服务器配额或上游延迟返回过期数据。看 `generated_at` 或 `pubDate`，超过 24 小时一定要提醒"数据可能偏旧 / 今日尚未更新"。
- 不要捏造没在 feed 里的数据。如果用户问的角度（如某只具体股票）没在 Watchlist/opportunities 里，明说"今天的简报没提到 X"，不要编造分析。
- 简报是**自动生成**的，不是人工 research。回答时若用户问"是否值得买"之类决策性问题，提示"此为算法生成的资讯整合，不构成投资建议"。
- 数据源覆盖中文（A 股 / 港股）+ 英文（美股 + arXiv 论文）AI 主题，不覆盖非 AI 主题。
- onboarding 只做一次，重复设置定时任务会导致用户被多次推送同一份简报。
