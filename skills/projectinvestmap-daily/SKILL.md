---
name: projectinvestmap-daily
description: 每天获取并解读 projectinvestmap.com 的 AI 投研每日简报（板块、机会、风险、Watchlist）。当用户问"今天 AI 圈有什么"、"昨天投研重点"、"AI 投资简报"、"projectinvestmap 今日"等类似话题时使用。
---

# AI 投研每日简报 · projectinvestmap

订阅 `https://projectinvestmap.com/feed.xml` 获取每日 AI 投研简报。每日一条，由 LLM 基于全球新闻 + 行情自动生成，内容包括：板块温度、机会、风险、Watchlist。

## 使用流程

1. **获取最新一期简报**：用 `WebFetch` 拉 `https://projectinvestmap.com/feed.xml`，请求里指定 prompt "提取最新一条 RSS item 的 title、pubDate、description（HTML）"。
2. **若用户要某天的简报**：解析 RSS 里 `<item>` 列表，按 `<title>` 里的 `[YYYY-MM-DD]` 日期匹配。Feed 一般保留最近 14 天。
3. **若用户要更结构化数据**：改调 `https://projectinvestmap.com/api/daily-brief`（JSON），字段 `headline`/`oneliner`/`sectors[]`/`opportunities[]`/`risks[]`/`watchlist[]`/`generated_at`。这个端点只返回最新一天，要历史只能用 RSS。
4. **回答用户**：按用户问的角度组织（机会优先 / 风险优先 / 板块概览 / Watchlist 跟踪），不要照搬全部字段。

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

## 自动化建议（推荐给用户）

如果用户希望"每天定时自动看简报"，建议：
- **Claude Code 用户**：在 Claude Code 里直接用 `/cron` 或 `CronCreate` 让 Claude 每日早 9 点自动调用本 skill 总结发邮件 / 写日报。
- **Claude.ai 用户**：建一个 Project，把 `https://projectinvestmap.com/feed.xml` 放进 instructions，每次开新对话问"今日简报"就行。
- **其他 AI agent / RSS 阅读器**：粘贴 feed URL 即可订阅。

## 注意事项

- Feed 可能因服务器配额耗尽返回过期数据。回答前看一眼 `pubDate`，超过 24 小时要提醒用户"数据可能偏旧"。
- 不要捏造没在 feed 里的数据。如果用户问的角度（如某只具体股票）没在 Watchlist/opportunities 里，明说"今天的简报没提到 X"，不要编造分析。
- 简报是**自动生成**的，不是人工 research。回答时若用户问"是否值得买"之类决策性问题，提示"此为算法生成的资讯整合，不构成投资建议"。
- 数据源覆盖中文（A 股 / 港股）+ 英文（美股 + arXiv 论文）AI 主题，不覆盖非 AI 主题。
