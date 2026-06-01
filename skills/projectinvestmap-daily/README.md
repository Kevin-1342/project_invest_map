# projectinvestmap-daily · Claude skill

让 Claude 每天帮你看 [projectinvestmap.com](https://projectinvestmap.com) 的 AI 投研每日简报。

## 这是什么

`projectinvestmap.com` 每天自动生成一份 AI 投研简报，覆盖：

- **板块温度**：大模型基座、存储/HBM、光通信、机器人、数据中心等的当日强弱
- **机会**：投资机会 + 标的票池 + 时间维度
- **风险**：当日值得警惕的风险点
- **Watchlist**：值得跟踪的关键事件/公司

数据通过标准 **RSS 2.0** 暴露在 `https://projectinvestmap.com/feed.xml`，免费、无需登录。

## 安装（Claude Code）

一行命令安装：

```bash
mkdir -p ~/.claude/skills && \
  curl -fsSL https://raw.githubusercontent.com/Kevin-1342/project_invest_map/main/skills/projectinvestmap-daily/SKILL.md \
    -o ~/.claude/skills/projectinvestmap-daily/SKILL.md --create-dirs
```

或者克隆后 copy：

```bash
git clone https://github.com/Kevin-1342/project_invest_map.git
cp -r project_invest_map/skills/projectinvestmap-daily ~/.claude/skills/
```

然后重启 Claude Code（或开新会话）。之后只要问 Claude "今天 AI 投研简报"、"projectinvestmap 今日"、"昨天 AI 圈有什么"等，就会自动调用这个 skill。

### 首次启用：装好默认 9 点推送，再问你要不要改

第一次触发 skill 时，Claude 会**先**用 `CronCreate` 把每日推送装好（默认早 **9 点**，08:57 本地时区，避开整点），然后告诉你：

> 已设好默认每天早 9 点自动给你推送 AI 投研简报。想改时间吗？比如"改到早 7 点"、"改到晚上 10 点"。或者说"不用定时"我就取消。不回我的话保持默认早 9 点。

你的回应决定下一步：

| 你说什么 | Claude 怎么做 |
|---|---|
| 不回 / 跳过 / 转去问别的 | 保持默认早 9 点 |
| `改到 X 点` / `早 7 点半` / `晚上 10 点` | 找到刚才的任务删掉，用新时间重建（仍避开整点，如 7:27 / 21:53） |
| `9 点就行` / `默认就行` | 不动，回个"好" |
| `不用定时` / `取消` | 删掉刚才的任务 |

之后随时跟 Claude 说一句 `把 AI 简报推送时间改成 X 点` 都能改。

设好后，Claude 每天到点会自动拉简报，按板块/机会/风险/Watchlist 给你总结。

> 如果你用的不是 Claude Code 而是 Claude.ai，没有 cron 能力，Claude 会改用 Project + Custom Instructions 的方式（见下一节）。

## 安装（Claude.ai / Claude Project）

不需要 skill 文件，直接：

1. 新建一个 Project
2. 在 Custom Instructions 里贴：
   ```
   每次回答有关 AI 投资 / AI 行业新闻的问题时，先用 web fetch 拉
   https://projectinvestmap.com/feed.xml 看最新一条 item，参考里面的
   板块温度、机会、风险、Watchlist 来回答。
   ```
3. 之后问"今日 AI 简报"即可

## 安装（其他 AI agent / RSS 阅读器）

直接订阅 feed URL：

```
https://projectinvestmap.com/feed.xml
```

支持的客户端：Feedly、Inoreader、NetNewsWire、Reeder、Thunderbird、任何兼容 RSS 2.0 的工具。

## 通用订阅 Prompt（复制即用）

如果你用的是任意一个能联网的 AI agent（Claude Code cron / Claude.ai Project Custom Instructions / ChatGPT GPTs / Gemini Gem / Cursor / 其他自建 agent），把下面这段**整体复制**贴到它的系统指令、Custom Instructions、或定时任务的 prompt 里就能用——无需改任何字段：

```
你是 AI 投研每日简报订阅助手。每次被触发时，按下面流程给我推送 projectinvestmap.com 的当日 AI 投研简报：

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

末尾附一句数据日期，例如"——简报日期 2026-06-01，生成于 06:02 北京时间"。

[免责]
若我问"是否值得买"之类决策性问题，提示"此为算法生成的资讯整合，不构成投资建议"。不要捏造没在 feed 里的数据，没提到就直说没提到。
```

> 用法举例：
> - **Claude Code cron**：`CronCreate` 时把这段贴进 `prompt`，cron 设 `57 8 * * *`（约早 9 点）。
> - **Claude.ai Project**：建 Project，把这段贴进 Custom Instructions，每天早上自己开一句"今日简报"。
> - **ChatGPT GPTs / Gemini Gem**：建一个 GPT/Gem，把这段贴进 Instructions，每天问一次。
> - **任意定时任务系统（n8n / Zapier / cron + curl + LLM API）**：把这段当作 system prompt，每天定时触发即可。

## 自动化：每天定时拿简报

- **Claude Code 用户**：装完 skill 第一次触发时已经自动设好了（见上面"首次启用"段），无需额外操作。
- **不用 Claude 的命令行用户**：直接 crontab 一行：

  ```bash
  # 每天早 9 点存最新简报到桌面
  57 8 * * * curl -s https://projectinvestmap.com/api/daily-brief \
    -o ~/Desktop/ai-brief-$(date +\%Y-\%m-\%d).json
  ```

## 数据接口

| 端点 | 类型 | 用途 |
|---|---|---|
| `/feed.xml` | RSS 2.0 | 最近 14 天的简报，给 RSS reader / AI agent 订阅 |
| `/api/daily-brief` | JSON | 仅最新一天，结构化数据，字段见 SKILL.md |

## 免责声明

- 简报由 LLM 自动生成，不是人工 research
- 不构成投资建议
- 数据源覆盖 AI 主题（中美港 + arXiv），不覆盖非 AI 主题
