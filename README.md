# gstack-teams

> 基于 [gstack](https://github.com/garrytan/gstack) 的 WorkBuddy Agent Teams 多角色调度 Skill v2.1

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![WorkBuddy](https://img.shields.io/badge/WorkBuddy-Skill-green.svg)](https://codebuddy.cn)
[![Version](https://img.shields.io/badge/version-v2.1-orange.svg)](#)

## 📖 简介

**gstack-teams** 将 gstack 的 14 个专业工程角色适配为 WorkBuddy Agent Teams，让你能够像管理真实的工程团队一样，按开发阶段按需召唤对应角色，形成完整的虚拟工程团队协作工作流。

### 核心特性

- ✨ **14 个专业角色**：覆盖产品、工程、设计、安全、QA、发布、监控全流程
- 🔄 **阶段化调度**：规划期 → 开发期 → 发布期前 → 发布期 → 发布期后的完整工作流
- 🛡️ **安全机制**：防假死设计（max_turns + timeout + 看门狗模式）
- 🤖 **自动审查流水线**：Autoplan 按 6 条决策原则自动串联审查
- 🤝 **多角色协作**：支持串行、并行、按需启动等多种调度模式
- 📋 **标准化产物**：统一的文档规范和交接标准
- 🚀 **自动化流程**：代码审查、安全审计、测试、发布、监控的端到端自动化

---

## 👥 角色一览（v2.1）

### 规划期角色

| 角色名 | 职责 | 阶段 | 建议 max_turns |
|--------|------|------|----------------|
| **Product Partner** | 产品方向验证，YC Office Hours 风格 | 规划期 | 15 |
| **CEO Reviewer** | 战略审查，11 维度深度评审 | 规划期 | 20 |
| **Eng Reviewer** | 架构规划，锁定技术方案 | 规划期 | 15 |
| **Autoplan** | 自动审查流水线，6 条决策原则 | 规划期 | 25 |

### 开发期角色

| 角色名 | 职责 | 阶段 | 建议 max_turns |
|--------|------|------|----------------|
| **Staff Engineer** | 代码审查，双阶段关键/信息性 | 开发期 | 15 |
| **Debugger** | 根因调试，铁律不找根因不修复 | 开发期 | 25 |
| **Design Reviewer** | 设计质量审查，AI 痕迹检测 | 开发期 | 10 |

### 发布期前角色

| 角色名 | 职责 | 阶段 | 建议 max_turns |
|--------|------|------|----------------|
| **CSO** | 首席安全官，OWASP + STRIDE 审计 | 发布期前 | 20 |

### 发布期角色

| 角色名 | 职责 | 阶段 | 建议 max_turns |
|--------|------|------|----------------|
| **QA Lead** | 系统化测试，健康评分 | 发布期 | 30 |
| **Ship** | 创建 PR，测试，推送 | 发布期 | 20 |
| **Land-and-Deploy** | 合并 PR，部署，上线验证 | 发布期 | 25 |
| **Doc Engineer** | 发布后文档同步 | 发布期 | 15 |

### 发布期后角色

| 角色名 | 职责 | 阶段 | 建议 max_turns |
|--------|------|------|----------------|
| **Canary** | 部署后可视监控，基线对比+控制台错误 | 发布期后 | 10 |

### 任意时间角色

| 角色名 | 职责 | 阶段 | 建议 max_turns |
|--------|------|------|----------------|
| **Eng Manager** | 工程效能回顾分析 | 任意时间 | 15 |

---

## 🚀 快速开始

### 安装

#### 方法 1：从 GitHub 安装

```bash
# 1. 克隆仓库
git clone https://github.com/gosinkx/gstack-teams.git

# 2. 复制到 WorkBuddy skills 目录
# Windows:
Copy-Item -Path "gstack-teams\*" -Destination "$env:USERPROFILE\.workbuddy\skills\gstack-teams" -Recurse

# macOS/Linux:
cp -r gstack-teams/* ~/.workbuddy/skills/gstack-teams/
```

#### 方法 2：使用 ZIP 安装包

下载 `gstack-teams.zip`，解压后复制到 WorkBuddy skills 目录：

```
~/.workbuddy/skills/gstack-teams/
├── SKILL.md
├── references/
│   ├── roles.md
│   ├── safety-rules.md
│   ├── failure-log.md
│   ├── cso.md
│   ├── autoplan.md
│   ├── land-and-deploy.md
│   ├── canary.md
│   └── project-conventions.md
└── README.md
```

### 使用

安装完成后，在 WorkBuddy 对话中使用自然语言触发：

**示例 1：规划新功能**
```
帮我创建规划团队，验证这个新功能的可行性和技术方案
```

**示例 2：自动审查流水线**
```
帮我启动自动审查流水线，按 6 条决策原则自动执行所有审查
```

**示例 3：代码审查**
```
调用 gstack 角色帮我做 code review，审查当前分支的代码变更
```

**示例 4：安全审计**
```
启动 CSO 进行发布前安全审计，执行 OWASP + STRIDE 扫描
```

**示例 5：发布流程**
```
帮我跑完整发布流程，从测试到部署监控
```

**示例 6：部署后监控**
```
部署完成了，启动 Canary 监控线上状态
```

---

## 📋 工作流程

### 完整工作流图

```
新功能需求
    │
[product-partner] → 设计文档
    │
[ceo-reviewer] ──→ 战略审查
    │
[eng-reviewer] ──→ 架构规划
    │
[autoplan] ──────→ 自动审查流水线（可选）
    │（开始编码）
[staff-engineer] ─┐
[design-reviewer] ─┤ 并行审查
[debugger] (待命) ─┘
    │（代码就绪）
[qa-lead] ───────→ 质量测试（必须通过，max_turns=30）
    │
[cso] ──────────→ 安全审计（必须在 Ship 之前完成）
    │
[ship] ─────────→ 创建 PR，推送远程（max_turns=20）
    │
[land-and-deploy] ─→ 合并 + 部署 + 验证（max_turns=25）
    │
[doc-engineer] ──→ 文档同步（max_turns=15）
    │
[canary] ────────→ 部署后监控（max_turns=10）
    │
[eng-manager] ───→ 每周回顾（可选，max_turns=15）
```

### 阶段一：规划期（新功能开始前）

**选项 A：手动审查（逐项确认）**
```
用户需求
  ↓
Product Partner（需求澄清、设计文档）
  ↓ @Lead
CEO Reviewer（战略审查、决策）
  ↓ @Lead
Eng Reviewer（架构规划、技术方案）
  ↓
汇总三方意见 → 决策是否开始编码
```

**选项 B：自动审查流水线（Autoplan，无中间确认）**
```
用户需求
  ↓
Autoplan（自动串联 CEO → Design → Eng Review）
  ↓
6 条决策原则自动处理中间问题
  ↓
最终门控推荐 → 用户确认
```

**触发词示例**：
- "创建规划团队"
- "规划期团队"
- "验证这个新功能"
- "帮我做产品审查"
- "启动自动审查流水线"

### 阶段二：开发期（代码开发中）

```
代码变更完成
  ↓
Staff Engineer（代码审查）
  ↓
Design Reviewer（设计审查）
  ↓（并行）
如有 bug → Debugger（根因调试）
  ↓
汇总审查意见 → 决策是否合并
```

**触发词示例**：
- "帮我做 code review（多角色）"
- "调用开发团队"
- "审查当前分支"
- "调试这个 bug"

### 阶段三：发布期前（安全审计）

**⚠️ 必须在 QA 之前完成，高风险问题未修复禁止进入发布期**

```
代码就绪
  ↓
CSO（15 阶段 OWASP + STRIDE 安全审计）
  ↓
门控结论：APPROVED / APPROVED_WITH_RISKS / BLOCKED
```

**阻断条件**：
- 发现高风险问题（OWASP Top 10 相关）
- 依赖项有已知 CVE（High/Critical）
- 代码中检测到硬编码密钥

**触发词示例**：
- "启动 CSO 安全审计"
- "执行发布前安全审查"
- "OWASP 扫描"

### 阶段四：发布期（准备上线 + 合并部署）

**选项 A：分步执行（推荐，便于控制）**
```
准备发布
  ↓
QA Lead（系统化测试 → 健康评分达标）
  ↓
Ship（执行发布流程 → 创建 PR，推送远程）
  ↓
Land-and-Deploy（合并 PR → 部署 → 上线验证）
  ↓
Doc Engineer（同步文档 → 变更日志）
  ↓
触发 Canary 监控
```

**选项 B：全自动（无中间确认）**
```
QA Lead → Ship → Land-and-Deploy → Doc Engineer → Canary
（任一步骤失败则中止并 @Lead）
```

**触发词示例**：
- "帮我跑完整发布流程"
- "创建发布团队"
- "启动 QA 测试"
- "合并部署这个版本"

### 阶段五：发布期后（部署后监控）

```
部署完成且 Land-and-Deploy 验证通过
  ↓
Canary（基线对比 + 控制台错误检测）
  ↓
判定结论：🟢通过 / 🟡警告 / 🔴异常
```

**监控时长**：
- 小型修复（Hotfix）：15 分钟
- 常规发布：30 分钟
- 大型功能发布：60 分钟

**触发词示例**：
- "启动 Canary 监控"
- "部署后检查"
- "线上状态监控"

### 额外功能：工程回顾

```
任意时间
  ↓
Eng Manager（工程效能回顾分析）
  ↓
效能报告、优化建议
```

**触发词示例**：
- "做工程回顾"
- "分析工程效能"
- "工程管理回顾"

---

## 🛡️ 安全机制（v2.0+ 新增）

### 防假死设计

**问题**：子 agent 执行终端命令时容易假死，主 agent 无限等待

**解决方案**：

#### 规则 1：强制设置 `max_turns`

每次调用 Agent 工具必须设置 `max_turns`（见角色表建议值）：

```python
Agent(
    description="启动 QA 测试",
    prompt="...",
    max_turns=30   # ← 必须设置
)
```

#### 规则 2：终端命令强制设置 `timeout`

| 命令类型 | 建议 timeout | 说明 |
|----------|--------------|------|
| 文件操作 | 30000ms | 30秒 |
| 包安装 | 300000ms | 5分钟 |
| 构建/编译 | 600000ms | 10分钟 |
| 测试套件 | 300000ms | 5分钟 |
| git 操作 | 60000ms | 1分钟 |
| git push/fetch | 120000ms | 2分钟 |

#### 规则 3：禁止交互式命令

```python
# ✅ 正确：
Bash(command="npm install --yes", timeout=300000)

# ❌ 错误：
Bash(command="npm install")  # 可能等待用户输入！
```

#### 规则 4：子 agent 定期上报状态

每个子 agent 每完成 2-3 个工具调用，通过 `SendMessage` 向 Lead 汇报进度。

#### 规则 5：长时间任务使用 `run_in_background`

预计运行时间超过 5 分钟的任务，使用 `run_in_background=true`。

### 看门狗模式（Watchdog）

主 agent 启动子 agent 后，启动看门狗监控：

```
1. 启动子 agent（设置 max_turns）
   │
2. 主 agent 进入等待状态
   │
3. 【看门狗】每 60 秒检查一次：
   │   - 收到状态上报 → 正常，继续等待
   │   - 超过 180 秒无消息 → 发送健康检查
   │   - 2 次健康检查无响应 → 判定假死
   │
4. 【恢复】判定假死后：
   ├── 发送 shutdown_request
   ├── 等待 30 秒确认关闭
   └── 向用户报告：哪个 agent 假死、可能原因、建议操作
```

---

## 📂 项目结构

```
gstack-teams/
├── SKILL.md                      # Skill 主定义文件（v2.1）
├── references/
│   ├── roles.md                  # 14 个角色的完整 Prompt 库
│   ├── safety-rules.md           # 安全规范（防假死）
│   ├── failure-log.md            # 假死事件记录模板
│   ├── cso.md                    # CSO 安全审计流程（15 阶段）
│   ├── autoplan.md               # 自动审查流水线（6 条决策原则）
│   ├── land-and-deploy.md        # 合并部署验证流程
│   ├── canary.md                 # 部署后监控流程
│   └── project-conventions.md    # 项目协作规范
├── README.md                     # 项目文档（本文件）
├── LICENSE                       # MIT 许可证
└── gstack-teams.zip              # 独立安装包
```

---

## 🔧 角色调度模式

### 串行执行

- **规划期**：Product Partner → CEO Reviewer → Eng Reviewer（严格顺序）
- **发布期**：QA Lead → Ship → Land-and-Deploy → Doc Engineer（不可跳过 QA）
- **发布期前**：CSO 必须在 Ship 之前完成

### 并行执行

- **开发期**：Staff Engineer + Design Reviewer（同时进行，提高效率）

### 按需启动

- **Debugger**：默认待命，发现 bug 时才启动（不找根因不修复的铁律）
- **Canary**：部署完成后启动，监控线上状态

---

## 📚 文档规范

### 标准路径

- 设计文档：`docs/designs/`
- QA 报告：`docs/qa-reports/`
- 架构文档：`docs/architecture/`
- 安全审计报告：`SECURITY-AUDIT.md`
- 部署监控报告：`CANARY-REPORT.md`
- 任务列表：`TODOS.md`
- 回顾分析：`.context/retros/`

### 协作原则

1. **完整性**：每个角色必须完成所有产出物
2. **零静默失败**：遇到问题必须主动汇报，不能静默跳过
3. **证据优先**：所有判断基于数据和证据，而非猜测
4. **最小 diff**：变更要尽可能小，降低风险

---

## 🆕 更新日志

### v2.1（2026-04-30）

- ✨ **新增 4 个角色**：CSO、Canary、Autoplan、Land-and-Deploy
- 🔄 **新增 2 个阶段**：发布期前（安全审计）、发布期后（部署监控）
- 🛡️ **安全机制**：7 条安全规则、看门狗模式、假死恢复流程
- 📋 **新增参考文件**：cso.md、autoplan.md、land-and-deploy.md、canary.md、safety-rules.md、failure-log.md

### v2.0（2026-04-29）

- 🛡️ **安全机制**：防假死设计（max_turns + timeout）
- 📝 **所有角色**：新增 STATUS_REPORTING 和 SAFETY RULES 章节
- 📊 **角色表**：新增建议 max_turns 列

### v1.0

- 🎉 初始版本，9 个基础角色

---

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

### 如何贡献

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

---

## 📄 许可证

本项目采用 [MIT 许可证](LICENSE)。

---

## 🙏 致谢

- [garrytan/gstack](https://github.com/garrytan/gstack) - 原始角色设计灵感来源
- [WorkBuddy](https://codebuddy.cn) - 强大的 AI 协作平台

---

## 📮 联系方式

- GitHub Issues: [提交问题](https://github.com/gosinkx/gstack-teams/issues)

---

<div align="center">

**⭐ 如果这个项目对你有帮助，请给个 Star！⭐**

Made with ❤️ by WorkBuddy Community

</div>
