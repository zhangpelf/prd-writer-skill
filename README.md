<div align="center">

# 📋 prd-writer-skill

**一句话把需求变成研发可直接开工的 PRD · 零 LLM API 费用**

*Turn a one-liner into a dev-ready PRD in 3 minutes — Zero extra LLM cost*

[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-D97757?logo=anthropic)](./prd-writer-skill/SKILL.md)
[![n8n](https://img.shields.io/badge/n8n-%3E%3D1.50-EA4B71?logo=n8n)](./prd-orchestrator-workflow.json)
[![Zero API Cost](https://img.shields.io/badge/LLM_API-%C2%A50-brightgreen)](#-为什么是双组件)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-orange)](#-贡献)

[快速开始](#-3-分钟快速开始) · [效果演示](#-效果演示) · [为什么不一样](#-为什么不一样) · [精湛细节](#-精湛细节) · [FAQ](#-faq)

> **仓库总览（本文）。** 完整执行规范以 [`prd-writer-skill/SKILL.md`](./prd-writer-skill/SKILL.md) 为准，Skill 使用手册见 [`prd-writer-skill/README.md`](./prd-writer-skill/README.md)。本文与 Skill 文档冲突时，以 Skill 文档为准。

</div>

---

## 痛点 → 方案

| 你现在写 PRD 的痛点 | 这个仓库怎么解决 |
|---|---|
| 和 AI 来回扯 5 轮，出来的还是口号文，研发看完问“到底做啥” | **七大章节强制模板**：版本历史 → 概述 → 功能需求 → 非功能 → 数据模型与接口 → 发布计划 → 风险，缺一不可，直接对齐研发 |
| 改一版需求，旧文档就作废，历史版本找不到 | **Step 0 历史检测 + 全量合并**：自动扫 `docs/`，同域需求全量合并、版本号追加，永远只有一个权威最新版 |
| 写完存在聊天记录里，找不到、发不出 | **本地双路径落盘 + Webhook 推送**：`docs/` 主路径 + `~/Downloads/prd-reports/` 备份，飞书 / 企微 / 钉钉 / 邮件一键通知 |
| n8n 调 LLM 又要配 Key 又要烧钱 | **双组件架构**：Skill 用你已有的 Claude Code 生成（0 额外费用），n8n 只做它擅长的存档、分发、通知 |

## ✨ 为什么不一样

别家“AI 写 PRD”只解决“写出来”，这个仓库解决的是“**写出来、管得住、发得出去、研发认**”。

1. **🔁 改需求不丢历史，全量合并才是真版本管理** — Step 0 三级检索（对话上下文 → `docs/【PRD】*.md` → 备份目录），命中同域自动进合并模式：读全文、按六章各自规则合并、版本表追加 `v1.0 → v1.1`、输出完整可交付全文。严禁只给 diff，跨团队拿到的永远是最新权威版。
2. **🔀 新增 / 迭代两种笔法，自动切换** — 新增场景两列表（原型|需求描述）讲全貌和动线；迭代场景四列表（模块|现状|改动预期|功能描述）讲差异，开头还强制「改动点总览」（新增 / 字段变更 / 删除各几条）。不用你教，Skill 自己判。
3. **✅ 写完就能测** — P0 限 5 个以内防 MVP 膨胀，每个功能至少一组 `Given / When / Then`（正常流 + 异常流：无权限、并发锁、超时、空数据全覆盖）。
4. **🧩 前端零猜测** — 筛选 / 表单必标组件类型（`Select` / `TreeSelect` / `DatePicker` / `Upload` / `Drawer`…），对齐 Ant Design / Element Plus / TDesign，`prd-config.yaml` 一行切换。
5. **🖼️ 图片有落位，不丢图** — 新增图进「原型」列、迭代现状图 / 目标图分列、流程图进 2.2，判不准先进附录并在正文标注。原型截图不会再“仅留占位符”。
6. **🔒 安全评审一次过** — 脱敏显示、审计留存 ≥180 天、HTTPS、敏感词机审 + 人审，基线直接写进模板。
7. **📦 三场景闭环，有回执才算完** — 对话渲染全文 → 双路径落盘 → Webhook 推送，末尾必输出 `✅/⚠️/❌` 回执表。没跑到 Step 8 就是流程没闭环。
8. **💸 0 额外费用** — 生成靠你已有的 Claude Code / OpenCode，n8n 9 节点只做搬运工，不调 LLM、不烧 token、不配 Key 也能跑纯本地模式。

## 🔬 精湛细节（内行看门道）

- **触发词预埋进 frontmatter**：`SKILL.md` 头部 `description` 把“帮我写PRD/生成需求文档…”全埋好，Agent 靠这行自动路由，不用背 `/` 命令。
- **同域误判有刹车**：拿不准是不是同一功能域时，固定话术反问“是在原 PRD 上迭代，还是建独立文档？”，宁可多问一句不瞎合并。
- **六章各有合并算法**：概述取并集、功能追加并重分 P0/P1/P2、接口只追加不删历史、风险追加新项——不是一句“合并一下”糊弄。
- **质量红线写死**：一句话必须“为解决…本次建设…从而实现…”句式；KPI 必须基线→目标→公式→周期四件套；性能禁“流畅/优秀”虚词，P95/P99 写数字。
- **防阻断设计**：本地落盘先于远程推送，Webhook 超时 / 域名挂 / 返回错误只记失败原因，回执标 `⚠️ 降级跳过`；回执路径强制绝对路径、禁 `~`。
- **文件名防炸**：n8n 侧 `replace(/[^a-zA-Z0-9\u4e00-\u9fa5]/g,'_')` + 日期前缀，中文 topic 也能安全落盘；`savePath` 为空自动回退默认目录。
- **三通道原生载荷**：飞书交互卡片 / 企微 Markdown / 钉钉 Markdown 各给一份可直接贴的 JSON，不用你去翻机器人文档。
- **文档分层不打架**：根 README 是门面、Skill README 是手册、`SKILL.md`（522 行）是唯一真源，冲突以 SKILL 为准。
- **git 够狠**：旧直调 LLM 的 workflow 在 `08c7323` 直接删掉，不留两套方案让人纠结。

## 🚀 3 分钟快速开始

### 1️⃣ 安装 Skill（30 秒）

```bash
git clone https://github.com/zhangpelf/prd-writer-skill.git
cp -r prd-writer-skill/prd-writer-skill ~/.claude/skills/prd-writer/
# 项目级隔离则放到 [project]/.claude/skills/prd-writer/
```

### 2️⃣（可选）配推送（30 秒）

```yaml
# prd-writer/config/prd-config.yaml
prd_writer:
  local_output_dir: "{project_root}/docs"
  local_backup_dir: "~/Downloads/prd-reports"
  auto_publish: false        # 改 true 才推送
  webhook_url: ""            # 飞书/企微/钉钉机器人或 n8n 网关
  notify_type: "feishu"      # feishu | wecom | dingtalk | email | none
  design_system: "Ant Design"
```

留空即纯本地模式，不配也能用。

### 3️⃣ 一句话触发（2 分钟出全文）

在 Claude Code / OpenCode / 任何兼容 Skill 的 Agent 里输入：

```
/prd-writer
帮我写一份 PRD，做一个带标签、日历视图、语音速记的灵感小程序
```

你会依次拿到：**对话内全文 → 本地双路径文件 → 推送通知 → 三场景验收回执**。

n8n 侧只需导入一次：n8n → Workflows → Import from File → `prd-orchestrator-workflow.json` → Activate。

## 👀 效果演示

输入：

> 做一个个人待办 + 灵感小程序，支持标签、日历视图、语音速记、Markdown 导出。

输出（节选，完整 7 章见 Skill 模板）：

```markdown
### 版本历史
| v1.0 | 2026-09-15 | AI | 新建 | 初始版本创建 |

# 1. 概述
## 1.1 一句话描述
为解决灵感散落、待办与笔记割裂，本次建设灵感待办小程序，从而实现记录耗时 ≤1.5分钟、周留存 ≥40%。

# 2. 功能需求
#### P0 - MVP（≤5个）
| 功能模块 | 功能点 | 用户价值 | 验收标准 |
|---|---|---|---|
| 速记 | 语音速记转文字 | 作为用户，我需要说话即记，以便于通勤时捕捉灵感 | 见 2.5 用例1 |

## 2.5 验收标准
- Given 已登录… When 点击提交… Then 返回成功提示、防重、写审计日志…
```

迭代时更省事：

> 在刚才的 PRD 基础上，加一个“卡片分享海报”功能。

Skill 自动命中 `docs/【PRD】*.md` → 全量合并输出 `v1.1`，历史一字不丢。

## 🏗️ 工作原理

```text
你的一句话
   │
   ▼
┌─────────────────────────┐
│ Skill（Claude Code 侧） │  零额外 API 费
│ Step0 历史检测 → Step1  │  场景判定 → Step2 补信息
│ → Step3 生成七章 → 对话 │  输出 → 双路径落盘 → 推送
└────────────┬────────────┘
             │ POST { topic, content, notifyType }
             ▼
┌─────────────────────────┐
│ n8n（流转侧，可选）      │
│ Webhook → 文件命名 → 存 │
│ 档 → Slack/邮件/飞书通知 │
└─────────────────────────┘
```

为什么不让 n8n 直接调 LLM？因为你要多付一份 Key 的钱，还要把 prompt 调优再做一遍。Claude Code 本来就是最好的人机协作面，n8n 只干脏活累活。

## 🗂️ 仓库结构

```text
.
├── prd-writer-skill/
│   ├── SKILL.md              # 唯一真源：Step0~Step8 完整执行指令（522 行）
│   ├── README.md             # Skill 使用手册：流水线、载荷、合并策略详解
│   └── config/prd-config.yaml # 65 行：路径 / 推送 / 组件 / 合规全解耦
├── prd-orchestrator-workflow.json  # n8n 编排：9 节点，接收 → 命名 → 落盘 → 通知
└── README.md                 # 本文件：仓库门面
```

## ⚙️ 配置

| 项 | 在哪改 | 默认值 | 说明 |
|---|---|---|---|
| `local_output_dir` | `prd-config.yaml` | `{project_root}/docs` | 主路径，文件名 `【PRD】{需求名}.md` |
| `local_backup_dir` | 同上 | `~/Downloads/prd-reports` | 强制备份，防丢 |
| `auto_publish` | 同上 | `false` | `true` 才触发 Webhook |
| `notify_type` | 同上 | `feishu` | `feishu` / `wecom` / `dingtalk` / `email` / `slack` / `none` |
| `design_system` | 同上 | `Ant Design` | 组件标注规范 |

Webhook 通用载荷：`{ topic, content, savePath, notifyType, notifyTarget }`，各通道原生卡片格式见 [`prd-writer-skill/README.md`](./prd-writer-skill/README.md#webhook-推送载荷)。

## 🆚 和常见方案的差别

| 方案 | 费用 | 版本管理 | 可直接给研发 | 团队分发 |
|---|---|---|---|---|
| 手写 Word | 0，但 1 天起 | 手动 | 看功力 | 手动发 |
| 直接问 ChatGPT | 已付的会员 | 无 | 口号文多 | 复制粘贴 |
| n8n 直调 LLM | 按 token 烧 | 无 | prompt 难调 | 强 |
| **本仓库** | **0 额外** | **自动合并追加** | **模板强制 + GWT** | **n8n 自动** |

## 🗺️ 路线图

- [x] 双场景 + 全量合并 + 三通道推送
- [ ] PDF / Word 一键导出（Pandoc 节点）
- [ ] PRD 完整性自动检查（AI Agent 节点）
- [ ] Git 版本 diff 报告

## ❓ FAQ

**推送失败会丢文档吗？** 不会。本地落盘先于远程推送，失败只在回执里标 `⚠️ 降级跳过`。

**不装 n8n 能用吗？** 能。默认纯本地模式，Skill 自己就能写完 + 存好。

**支持哪些 Agent？** 任何认 `SKILL.md` 的：Claude Code、OpenCode、Antigravity 等。

**为什么合并更新必须全量输出？** 增量 diff 跨团队必出理解偏差，全量才能保证拿到的永远是最新权威版。

**旧版直调 LLM 的 workflow 呢？** 已在 `08c7323` 删除，需要去历史提交里翻。

## 🤝 贡献

欢迎提 Issue / PR：模板错别字、缺的组件类型、新通知通道都可以直接改 `SKILL.md` 后提过来。

## 📄 License

MIT — 拿去商用、改完闭源都行，保留原作者署名即可。

---

<div align="center">

觉得有用请点个 ⭐，这是我持续更新的最大动力。<br>
Built for 产品 + 研发不再为 PRD 扯皮。

</div>
