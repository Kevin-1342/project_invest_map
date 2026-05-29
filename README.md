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

## 怎么用

### 1. 直接打开网站

访问 [projectinvestmap.com](https://projectinvestmap.com)。

### 2. 订阅 RSS（推荐给 AI agent / 阅读器）

把这个 URL 贴到任何 RSS 阅读器或 AI agent 工作流里：

```
https://projectinvestmap.com/feed.xml
```

每天一条 item，含完整的板块温度、机会、风险、Watchlist。保留最近 14 天历史。

兼容客户端：Feedly、Inoreader、NetNewsWire、Reeder、Thunderbird、ChatGPT/Claude RSS 工具，等等。

### 3. 在 Claude 里用本仓库的 skill

让 Claude（Code 或 Claude.ai）每天自动帮你读简报。详见 [`skills/projectinvestmap-daily/`](./skills/projectinvestmap-daily/README.md)。

一行装到 Claude Code：

```bash
git clone https://github.com/Kevin-1342/project_invest_map.git
cp -r project_invest_map/skills/projectinvestmap-daily ~/.claude/skills/
```

下次开新会话，问"今天 AI 投研简报"即可触发。

### 4. 用 JSON API（适合自己写脚本）

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

- 网站简报由 LLM 自动生成，**不是人工 research**
- **不构成投资建议**
- 数据源覆盖 AI 主题，不覆盖非 AI 主题
- 服务可能因第三方数据源限额导致部分字段过期，请关注 `pubDate` / `generated_at`

---

## License

[MIT](LICENSE)
