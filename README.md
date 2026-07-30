# transfer-rumor-check · 足球转会传闻查证

![License](https://img.shields.io/badge/license-MIT-2ea44f)
![Claude Code Skill](https://img.shields.io/badge/Claude_Code-skill-D97757)
![Codex Skill](https://img.shields.io/badge/Codex-skill-111827)

> **别只搜英语。让 AI 同时检查买卖双方当地信源，再判断一条转会传闻究竟有多真、现在是否还成立。**

一套查证足球转会传闻可信度的 **AI agent skill**。给它一句话——哪怕只是"听说 X 要去 Y 队"——它主动联网建立报道场，沿**两个正交维度**分别下结论：

- **真实性**：这个转会说法有多可能是真的（六档：已官宣 / 极可靠 / 高可信 / 待证实 / 存疑 / 大概率假）
- **时效性**：它现在还成不成立（六状态：新鲜 / 进行中 / 已过时 / 已完成 / 已被推翻 / 窗口已关）

两个维度**分开报，绝不合成一个分**——因为一条消息完全可能"当时千真万确、现在毫无意义"。

核心不是"给个可信度分数"，而是一套**可解释、可迁移的判断方法论**：

- **信源当证据，不当评价对象**——用"谁说的"推断"这话有多可能真"，不给媒体本身打分或画像。
- **信源由系统主动检索**——用户只给球员+球队，skill 自己去查谁在报、报到哪一步。
- **买卖双方所在地/语言都要搜**——只搜一种语言会系统性漏掉离信源最近、最可能唱反调的卖方本地源（本方法论最重要的一条纪律）。
- **机构为主的保守分级 + 来源独立性判定**——"很多家都报了"若都转同一次采访，是伪印证，不算多源。
- **矛盾裁决**——时间不同视为信息演进（新的覆盖旧的）；同时间窗、同等权威仍对立，判为真分歧，真实性封顶"待证实"、两边摊开不选边。

## 30 秒看懂输出

当买方媒体称“交易将成”，卖方本地可靠来源却否认协议时，它不会按报道篇数投票：

| 维度 | 结论 |
|---|---|
| 🎯 **真实性** | **待证实** ⚠️ 信源分歧 |
| ⏱ **时效性** | **进行中**（双方口径仍在对立） |

**本次判定命题**：交易将成——结论只对这个命题负责；“确实在谈”不能证明“必然成交”。

| 立场 | 谁在报 | 口径 |
|---|---|---|
| 偏“将成” | 买方所在地媒体 | 接近达成，但多篇报道可能来自同一次采访 |
| 偏“未成” | 卖方所在地可靠本地源 | 没有协议，谈判阶段与买方报道不符 |

这时系统会识别伪多源、保留真实分歧，并把双方证据透明摊开，而不是替用户强行选边。

## 为什么它是"方法论"而不是"某个模型的私货"

SKILL.md + 数据文件是**纯文本、模型无关**的判断框架。任何能读文件 + 能联网搜索的 AI agent 都能执行。实测把它喂给不同厂商的模型（Claude 与 GPT/Codex），面对同一条传闻，各自独立搜出本地语言一手源、判出一致的结论——价值在方法论本身，不绑定单一平台。

## 仓库结构：一个方法论，两个安装入口

同一套方法论，按两个 agent 平台各打包了一份可直接安装的形态：

```
claude/transfer-rumor-check/    → Claude Code skill
codex/transfer-rumor-check/     → Codex skill
```

两版判断逻辑与数据完全对齐，只是文件组织贴合各自平台约定（见下）。

### 安装：Claude Code

从 GitHub 克隆并安装：

```bash
git clone --depth 1 https://github.com/Wolinaxia/transfer-rumor-check.git
cp -R transfer-rumor-check/claude/transfer-rumor-check ~/.claude/skills/
```

如果已经位于本仓库根目录：

```bash
cp -R claude/transfer-rumor-check ~/.claude/skills/
```

之后随口问"这转会靠谱吗 / 查证一下这条转会消息"即可自动触发（靠 `SKILL.md` frontmatter 的 `description`）。

- `SKILL.md` — 判断逻辑（7 步流程、真实性六档、时效性、矛盾裁决、输出格式）
- `sources.md` — 数据层（信源分档、硬细节、转会窗口、中超/中甲另一套源）
- `test-cases.md` — 10 场景回归测试集（改方法论后逐条跑，防退化）

### 安装：Codex

从 GitHub 克隆并安装：

```bash
git clone --depth 1 https://github.com/Wolinaxia/transfer-rumor-check.git
cp -R transfer-rumor-check/codex/transfer-rumor-check ~/.codex/skills/
```

如果已经位于本仓库根目录：

```bash
cp -R codex/transfer-rumor-check ~/.codex/skills/
```

自然语言触发，或显式 `$transfer-rumor-check`。

- `SKILL.md` — 判断逻辑
- `references/sources.md` · `references/test-cases.md` — 数据层与测试集（Codex 约定放 `references/`）
- `agents/openai.yaml` — Codex UI 展示与显式触发配置

> **前提：联网搜索能力。** 本方法论的报道场交叉验证、买卖双方检索、时效核实全依赖实时搜索。无网环境只能跑离线的一半（机构分档、主张内在逻辑检查、大致窗口），且会显式降级：真实性最高只给"待证实"、时效性标"无法核实"。

## 已知边界

- **采样偏差是首要失败模式**：只搜一种语言会系统性漏掉卖方本地反方。两版 SKILL.md 都有"买卖双方都要搜"的硬门槛专门拦这个。
- **只处理纯文字输入**，不接链接抓取 / 截图识图。
- **只摊信息源、不给核实建议**——透明铺开判断依据，让用户自己判断下一步，不假装能替你核实。
- **`sources.md` 的具体分档是初版**，需按实际媒体生态持续校准（改数据只动 `sources.md`，不碰逻辑）。

## 参与改进

欢迎用一条正在发生、跨语言区或存在信源分歧的真实传闻来压力测试。发现漏搜卖方本地源、来源独立性误判或阶段夸大时，请提交 [Issue](https://github.com/Wolinaxia/transfer-rumor-check/issues)。

如果这套方法对你有帮助，欢迎给仓库一个 Star，让更多球迷和 AI agent 开发者看到它。

## License

MIT
