# PRD 自动生成工作流（双组件架构）

## 概述

这是一个**零 LLM API 费用**的 PRD 工作流方案，由两部分组成：

1. **Skill（技能）**：在 Claude Code 中生成 PRD 内容（利用你已有的 AI 助手，无需额外付费 API）
2. **n8n 工作流（流程编排）**：接收 PRD 内容，自动保存、通知、管理版本

**内容主题可按每次请求动态更改** — 只需在 Claude Code 对话中提供不同的产品信息即可。

---

## 架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                    第一部分：PRD 撰写技能                      │
│                   （Claude Code / 本地 AI）                    │
│                                                             │
│  用户对话 → 结构化 Prompt → 完整 PRD Markdown                │
│           （零额外 API 费用）                                  │
└──────────────────────┬──────────────────────────────────────┘
                       │ POST {topic, content, notifyType}
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   第二部分：PRD 流程编排工作流                   │
│                      （n8n 实例）                              │
│                                                             │
│  Webhook 接收 → 格式化文件名 → 保存文件 → 可选通知(Slack/邮件) │
│                                                             │
│  返回确认：{success, file, path, timestamp}                   │
└─────────────────────────────────────────────────────────────┘
```

### PRD 文档结构

PRD 遵循标准的 **6 章节模板**，覆盖从业务目标到技术实现的完整链路：

| 章节 | 内容 | 适用读者 |
|------|------|---------|
| **1. 概述** | 产品背景、目标用户画像、核心目标、成功指标(KPI) | 所有人 |
| **2. 功能需求** | 用户故事、P0/P1/P2 分级功能列表、Given/When/Then 验收标准 | 产品 + 开发 |
| **3. 非功能需求** | 性能指标(P95 响应时间)、安全合规、兼容性、可扩展性 | 架构 + 运维 |
| **4. 数据模型与接口** | 核心实体定义(ER)、API 接口清单(方法/路径/请求/响应/错误码) | 开发 |
| **5. 发布计划** | 里程碑时间表、版本规划(v0.1/v0.2/v1.0)、依赖关系 | PM + 管理层 |
| **6. 风险评估** | 技术/业务风险矩阵(概率×影响)、缓解措施、负责人 | 管理层 |

功能需求使用 **P0/P1/P2 分级**：
- **P0**（MVP 必达，不超过 5 个）：首次发布必须完成
- **P1**（重要但不阻塞）：下个迭代排期
- **P2**（锦上添花）：未来路线图

验收标准采用 **Given/When/Then** 格式，每个功能至少覆盖正常流程 + 异常流程两个场景，确保需求可测试。

---

### 为什么这样设计？

| 问题 | 解决方案 |
|------|---------|
| 没有免费 LLM API | 使用你已有的 Claude Code / Cursor，无需额外付费 |
| n8n 做生成逻辑太复杂 | n8n 只做擅长的：流程编排、文件管理、通知推送 |
| 需要人工审核 | Claude 对话本身就是「人工+AI协作」，天然适合生成内容 |
| 团队协作需求 | n8n 负责分发、归档、通知，Skill 负责内容质量 |

---

## 快速开始

### 第一步：安装 PRD 撰写 Skill

1. 将 `prd-writer-skill/` 目录复制到你的 Skills 目录：
   - Claude Code: `~/.claude/skills/prd-writer/`
   - 或项目本地: `[project]/.claude/skills/prd-writer/`

2. 在 Claude Code 中输入以下任一方式触发：
   - `/prd-writer`
   - `帮我写一份 PRD`
   - `撰写产品需求文档`

3. 跟随 Claude 的引导，提供产品信息，分章节生成 PRD

### 第二步：导入 n8n 工作流

1. 打开你的 n8n 实例（本地/云端/自托管均可）
2. 进入 **Workflows** → **Add Workflow**
3. 点击右侧菜单（...）→ **Import from File**
4. 选择 `prd-orchestrator-workflow.json`

5. **配置可选通知**（不需要通知则跳过）：
   - **Slack**: 在「发送Slack通知」节点配置 Slack Credential
   - **邮件**: 在「发送邮件通知」节点配置 Gmail Credential

6. 点击右上角 **Activate** 激活工作流

---

## 使用流程

### 场景 1：标准流程（生成 + 保存）

**步骤 1：在 Claude Code 中生成 PRD**

```
用户：/prd-writer
Claude：请提供产品信息...
用户：产品名称：智能投研Agent，目标用户：个人投资者...
Claude：[生成完整 PRD Markdown]
```

**步骤 2：将 PRD 发送到 n8n 保存**

Claude 生成完成后，复制 Markdown 内容，然后通过 curl 发送到 n8n：

```bash
curl -X POST https://your-n8n-instance/webhook/prd-orchestrator \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "智能投研Agent平台",
    "content": "[粘贴完整的 PRD Markdown 内容]",
    "savePath": "/Users/zhangpeifu/prd-reports"
  }'
```

**响应**：
```json
{
  "success": true,
  "message": "PRD 保存成功",
  "file": "prd-2026-07-15-智能投研Agent平台.md",
  "path": "/Users/zhangpeifu/prd-reports/prd-2026-07-15-智能投研Agent平台.md",
  "topic": "智能投研Agent平台",
  "notifyType": "none",
  "timestamp": "2026-07-15T12:34:56.789Z"
}
```

### 场景 2：生成 + 自动通知团队

```bash
curl -X POST https://your-n8n-instance/webhook/prd-orchestrator \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "智能投研Agent平台",
    "content": "[PRD Markdown 内容]",
    "savePath": "/Users/zhangpeifu/prd-reports",
    "notifyType": "slack",
    "notifyTarget": "#product-team"
  }'
```

### 场景 3：集成到自动化脚本

创建一个 `send-prd.sh` 脚本：

```bash
#!/bin/bash

# 读取文件内容
CONTENT=$(cat "$1")
TOPIC=$(basename "$1" .md)

# 发送到 n8n
curl -X POST https://your-n8n-instance/webhook/prd-orchestrator \
  -H "Content-Type: application/json" \
  -d "{
    \"topic\": \"$TOPIC\",
    \"content\": $(echo "$CONTENT" | jq -R -s .),
    \"savePath\": "/Users/zhangpeifu/prd-reports",
    \"notifyType\": "email",
    \"notifyTarget\": "team@company.com"
  }"
```

使用：
```bash
chmod +x send-prd.sh
./send-prd.sh ./智能投研Agent平台-prd.md
```

---

## 工作流详解

### 节点说明

| 节点 | 功能 | 配置要点 |
|------|------|---------|
| **接收PRD内容** | Webhook 入口，接收 POST 请求 | 路径：`/webhook/prd-orchestrator` |
| **提取请求参数** | 解析并标准化输入 | 支持字段：topic, content, savePath, notifyType, notifyTarget |
| **保存PRD文件** | 写入 Markdown 文件 | 文件名格式：`prd-YYYY-MM-DD-{topic}.md` |
| **是否通知Slack** | 条件判断 | 当 notifyType=slack 时触发 |
| **是否通知邮件** | 条件判断 | 当 notifyType=email 时触发 |
| **合并通知状态** | 汇总执行状态 | 用于最终响应 |
| **返回结果** | Webhook 响应 | 返回 {success, file, path, topic, timestamp} |

### 请求参数

| 字段 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| `topic` | string | ✅ | PRD 主题/产品名称 | "智能投研Agent平台" |
| `content` | string | ✅ | 完整 PRD Markdown 内容 | "## 1. 概述\n..." |
| `savePath` | string | ❌ | 保存目录（默认当前目录） | "/Users/zhangpeifu/prd-reports" |
| `notifyType` | string | ❌ | 通知类型：`none`/`slack`/`email` | "slack" |
| `notifyTarget` | string | 条件必填 | 通知目标（频道/邮箱） | "#product-team" |

---

## 进阶配置

### 添加更多通知渠道

n8n 支持 400+ 集成，你可以添加：

- **飞书/钉钉**：使用 HTTP Request 节点调用 Webhook
- **企业微信**：使用企业微信节点
- **Notion**：使用 Notion 节点创建页面
- **GitHub**：使用 GitHub 节点创建 Issue 或 Wiki 页面

### 自动备份到云存储

在「保存PRD文件」后添加：
- **Google Drive** 节点：自动上传到云端
- **Dropbox** 节点：同步到 Dropbox
- **S3** 节点：存入对象存储

### 版本管理

在「保存PRD文件」前添加 **Git** 节点：
1. 执行 `git add` 和 `git commit`
2. 提交信息自动包含 topic 和时间戳
3. 推送到远程仓库

---

## 与 Skill 的协作方式

### 方式 1：手动复制粘贴（最简单）

1. 在 Claude Code 中使用 `/prd-writer` 生成 PRD
2. Claude 输出完整 Markdown
3. 你复制内容，通过 curl/脚本发送到 n8n

### 方式 2：Claude 自动发送（推荐）

配置 Claude Code 在生成完成后自动调用 n8n Webhook：

在 `CLAUDE.md` 或 Skill 中增加指令：
```
当 PRD 生成完成后，自动执行以下操作：
1. 将完整 Markdown 内容保存到 /tmp/prd-draft.md
2. 调用脚本：./scripts/send-prd.sh /tmp/prd-draft.md
3. 确认发送成功并告知用户文件路径
```

### 方式 3：批量处理

如果你有多个 PRD 要生成：
1. 在 Claude Code 中一次性生成多个（对话中逐个输出）
2. 使用 n8n 的 **Schedule Trigger** + **Spreadsheet File** 节点批量读取
3. 自动发送到不同团队成员

---

## 文件清单

```
n8n-workflows/
├── prd-writer-skill/
│   └── SKILL.md          # PRD 撰写技能（Claude Code 使用）
├── prd-orchestrator-workflow.json  # n8n 流程编排工作流
└── README.md             # 本文档
```

---

## 常见问题

### Q: 为什么不让 n8n 直接调用 LLM 生成？
A: 这样做需要配置付费 LLM API Key（OpenAI/Anthropic）。本方案利用你已有的 Claude Code 环境，零额外费用。

### Q: Claude Code 生成的 PRD 质量如何？
A: 通过 Skill 中的结构化 Prompt 和模板约束，Claude 可以生成专业级 PRD。关键是提供充分的产品上下文信息。

### Q: 我不想用 curl，有更简单的方式吗？
A: 可以创建一个简单的网页表单（HTML），用 JavaScript 调用 Webhook。或者使用 n8n 的 **Form Trigger** 节点替代 Webhook，直接在浏览器中填写。

### Q: 支持团队协作评审吗？
A: 在 n8n 工作流中「保存PRD文件」后添加 **Wait** 节点，通过 Slack/邮件发送草稿链接，等待 PM 确认后再正式发布。

### Q: 如何追踪已生成的 PRD？
A: n8n 的 **Execution** 页面记录了每次调用的完整日志。你也可以在「保存PRD文件」后添加 **Google Sheets** 节点，自动记录到表格中。

---

## 兼容性

- **n8n**: >= 1.50.0（推荐 1.60+ 或 2.x）
- **Claude Code**: 任何版本
- **操作系统**: macOS / Linux / Windows（自托管 n8n）

---

## 扩展路线图

| 阶段 | 功能 | 实现方式 |
|------|------|---------|
| v1.0 | 基础保存 + 通知 | 当前版本 |
| v1.1 | 多格式输出（PDF/Word） | 添加 **Pandoc** 或 **Gotenberg** 节点 |
| v1.2 | PRD 质量检查 | 添加 **AI Agent** 节点自动检查完整性 |
| v1.3 | 版本对比 | 添加 **Git** 节点，生成 diff 报告 |
| v1.4 | 自动归档 | 按项目/时间自动分类到不同目录 |

---

## 最佳实践

1. **命名规范**：Topic 使用简短中文，文件名会自动转义特殊字符
2. **目录管理**：每月创建新目录（如 `prd-reports/2026-07/`），避免根目录文件过多
3. **通知策略**：仅对重要 PRD 启用通知，避免信息过载
4. **备份习惯**：定期将 `prd-reports/` 目录同步到云存储
5. **版本控制**：对正式版 PRD 提交到 Git，草稿可以不提交

---

## 旧版工作流

如果你仍然希望使用「n8n 直接调用 LLM」的方案（需要 OpenAI API Key），旧版工作流 `prd-generator-workflow.json` 仍然保留在目录中。

---

*版本：2.0 | 更新：2026-07-15 | 架构：双组件（Skill + n8n）*
