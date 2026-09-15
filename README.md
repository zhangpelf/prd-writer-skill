<div align="center">

# 📋 prd-writer-skill

**一句话把需求变成研发可直接开工的 PRD · 零 LLM API 费用**

*Turn a one-liner into a dev-ready PRD in 3 minutes — Zero extra LLM cost*

[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-D97757?logo=anthropic)](./prd-writer-skill/SKILL.md)
[![n8n](https://img.shields.io/badge/n8n-%3E%3D1.50-EA4B71?logo=n8n)](./prd-orchestrator-workflow.json)
[![Zero API Cost](https://img.shields.io/badge/LLM_API-%C2%A50-brightgreen)](#-为什么是双组件)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-orange)](#-贡献)

[快速开始](#-3-分钟快速开始) · [效果演示](#-效果演示) · [工作原理](#-工作原理) · [配置](#️-配置) · [FAQ](#-faq)

> **仓库总览（本文）。** 完整执行规范以 [`prd-writer-skill/SKILL.md`](./prd-writer-skill/SKILL.md) 为准，Skill 使用手册见 [`prd-writer-skill/README.md`](./prd-writer-skill/README.md)。本文与 Skill 文档冲突时，以 Skill 文档为准。

</div>

---

## 痛点 → 方案

| 你现在写 PRD 的痛点 | 这个仓库怎么解决 |
|---|---|
| 和 AI 来回扯 5 轮，出来的还是口号文，研发看完问“到底做啥” | **七大章节强制模板**：版本历史 → 概述 → 功能需求 → 非功能 → 数据模型与接口 → 发布计划 → 风险，缺一不可，直接对齐研发 |
| 改一版需求，旧文档就作废，历史版本找不到 | **Step 0 历史检测 + 合并更新**：自动扫 `docs/`，同域需求全量合并、版本号追加，永远只有一个权威最新版 |
| 写完存在聊天记录里，找不到、发不出 | **本地双路径落盘 + Webhook 推送**：`docs/` 主路径 + `~/Downloads/prd-reports/` 备份，飞书 / 企微 / 钉钉 / 邮件一键通知 |
| n8n 调 LLM 又要配 Key 又要烧钱 | **双组件架构**：Skill 用你已有的 Claude Code 生成（0 额外费用），n8n 只做它擅长的存档、分发、通知 |

## ✨ 核心能力

- **🔀 双场景自适应** — 新增场景用两列表（原型 | 需求描述）讲全貌；迭代场景用四列表（模块 | 现状 | 改动预期 | 功能描述）讲差异，进 Skill 自动判定
- **🔁 合并更新不丢历史** — 严禁只输出 diff，永远输出完整可交付全文，版本表追加 `v1.0 → v1.1`
- **✅ 可验收** — P0/P1/P2 分级（P0 ≤ 5 个），每个功能至少一组 `Given / When / Then`（正常流 + 异常流）
- **🧩 前端可直接开工** — 筛选 / 表单必标组件类型（`Select` / `TreeSelect` / `DatePicker` / `Upload`…），对齐 Ant Design / Element Plus
- **🔒 安全基线内置** — 脱敏显示、审计日志 ≥180 天、HTTPS、敏感词审核，安全评审不用返工
- **📦 开箱即用** — 1 个 `SKILL.md` + 1 个 `config` + 1 个 `n8n json`，无构建、无依赖

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
│   ├── SKILL.md              # 唯一真源：Step0~Step8 完整执行指令（517 行）
│   ├── README.md             # Skill 使用手册：流水线、载荷、合并策略详解
│   └── config/prd-config.yaml
├── prd-orchestrator-workflow.json  # n8n 编排：接收 → 命名 → 落盘 → 通知
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

- [x] v2.1 — 双场景 + 合并更新 + 三通道推送
- [ ] v1.1 — PDF / Word 一键导出（Pandoc 节点）
- [ ] v1.2 — PRD 完整性自动检查（AI Agent 节点）
- [ ] v1.3 — Git 版本 diff 报告

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
