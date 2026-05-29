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
