# gstack-teams

> 基于 [gstack](https://github.com/garrytan/gstack) 的 WorkBuddy Agent Teams 多角色调度 Skill

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![WorkBuddy](https://img.shields.io/badge/WorkBuddy-Skill-green.svg)](https://codebuddy.cn)

## 📖 简介

**gstack-teams** 将 gstack 的 9 个专业工程角色适配为 WorkBuddy Agent Teams，让你能够像管理真实的工程团队一样，按开发阶段按需召唤对应角色，形成完整的虚拟工程团队协作工作流。

### 核心特性

- ✨ **9 个专业角色**：覆盖产品、工程、设计、QA、发布全流程
- 🔄 **阶段化调度**：规划期 → 开发期 → 发布期的完整工作流
- 🤝 **多角色协作**：支持串行、并行、按需启动等多种调度模式
- 📋 **标准化产物**：统一的文档规范和交接标准
- 🚀 **自动化流程**：代码审查、测试、发布的端到端自动化

---

## 👥 角色一览

| 角色名 | 职责 | 阶段 | 核心能力 |
|--------|------|------|----------|
| **Product Partner** | 产品方向验证，YC Office Hours 风格 | 规划期 | 需求澄清、价值判断、用户体验 |
| **CEO Reviewer** | 战略审查，11 维度深度评审 | 规划期 | 战略对齐、优先级决策 |
| **Eng Reviewer** | 架构规划，锁定技术方案 | 规划期 | 技术选型、架构设计、方案评审 |
| **Staff Engineer** | 代码审查，双阶段关键/信息性 | 开发期 | 代码质量、最佳实践、性能优化 |
| **Debugger** | 根因调试，铁律不找根因不修复 | 开发期 | Bug 定位、根因分析、修复验证 |
| **Design Reviewer** | 设计质量审查，AI 痕迹检测 | 开发期 | UI/UX 审核、一致性检查 |
| **QA Lead** | 系统化测试，健康评分 | 发布期 | 测试计划、质量评估、健康评分 |
| **Release Engineer** | 全自动发布流程 | 发布期 | 版本管理、发布自动化 |
| **Doc Engineer** | 发布后文档同步 | 发布期 | 文档更新、变更日志 |
| **Eng Manager** | 工程效能回顾分析 | 任意时间 | 效能指标、流程优化、回顾分析 |

---

## 🚀 快速开始

### 安装

#### 方法 1：从 GitHub 安装

```bash
# 1. 克隆仓库
git clone https://github.com/your-username/gstack-teams.git

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
│   └── project-conventions.md
└── gstack-teams.zip
```

### 使用

安装完成后，在 WorkBuddy 对话中使用自然语言触发：

**示例 1：规划新功能**
```
帮我创建规划团队，验证这个新功能的可行性和技术方案
```

**示例 2：代码审查**
```
调用 gstack 角色帮我做 code review，审查当前分支的代码变更
```

**示例 3：发布流程**
```
帮我跑完整发布流程，从测试到文档同步
```

**示例 4：Bug 调试**
```
有个 bug 需要调查，启动 debugger 帮我定位根因
```

---

## 📋 工作流程

### 阶段一：规划期（新功能开始前）

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

**触发词示例**：
- "创建规划团队"
- "规划期团队"
- "验证这个新功能"
- "帮我做产品审查"

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

### 阶段三：发布期（准备上线）

```
准备发布
  ↓
QA Lead（系统化测试 → 健康评分达标）
  ↓
Release Engineer（执行发布 → 推送 PR）
  ↓
Doc Engineer（同步文档 → 变更日志）
  ↓
发布完成
```

**触发词示例**：
- "帮我跑完整发布流程"
- "创建发布团队"
- "启动 QA 测试"
- "准备发布这个版本"

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

## 📂 项目结构

```
gstack-teams/
├── SKILL.md                      # Skill 主定义文件
├── references/
│   ├── roles.md                  # 9 个角色的完整 Prompt 库
│   └── project-conventions.md    # 项目协作规范和路径约定
├── README.md                     # 项目文档（本文件）
├── LICENSE                       # MIT 许可证
└── gstack-teams.zip              # 独立安装包
```

---

## 🔧 角色调度模式

### 串行执行

- **规划期**：Product Partner → CEO Reviewer → Eng Reviewer（严格顺序）
- **发布期**：QA Lead → Release Engineer → Doc Engineer（不可跳过 QA）

### 并行执行

- **开发期**：Staff Engineer + Design Reviewer（同时进行，提高效率）

### 按需启动

- **Debugger**：默认待命，发现 bug 时才启动（不找根因不修复的铁律）

---

## 📚 文档规范

### 标准路径

- 设计文档：`docs/designs/`
- QA 报告：`docs/qa-reports/`
- 架构文档：`docs/architecture/`
- 任务列表：`TODOS.md`
- 回顾分析：`.context/retros/`

### 协作原则

1. **完整性**：每个角色必须完成所有产出物
2. **零静默失败**：遇到问题必须主动汇报，不能静默跳过
3. **证据优先**：所有判断基于数据和证据，而非猜测
4. **最小 diff**：变更要尽可能小，降低风险

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

- GitHub Issues: [提交问题](https://github.com/your-username/gstack-teams/issues)
- Email: your-email@example.com

---

<div align="center">

**⭐ 如果这个项目对你有帮助，请给个 Star！⭐**

Made with ❤️ by WorkBuddy Community

</div>
