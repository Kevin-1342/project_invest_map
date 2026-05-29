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

## 自动化：每天定时拿简报

**Claude Code**：跟 Claude 说一句

> 每天早上 9 点用 projectinvestmap-daily skill 总结今日简报，写到 `~/Desktop/ai-daily-{date}.md`

Claude 会自己用 `CronCreate` 设定时任务。

**curl + cron**：

```bash
# 每天早 9 点存最新简报
0 9 * * * curl -s https://projectinvestmap.com/api/daily-brief > ~/ai-brief-$(date +\%Y-\%m-\%d).json
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
