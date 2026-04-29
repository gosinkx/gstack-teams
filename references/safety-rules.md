# gstack Agent Teams — 安全规范（所有角色必须遵守）

> **违反本文件的规则是导致子 agent 假死的首要原因。每个角色在调用任何工具前必须阅读本文件。**

---

## 规则 1：所有终端命令必须设置 timeout

**强制执行。** 每次调用 `Bash` 或 `PowerShell` 工具时，必须设置 `timeout` 参数。

### 超时参考表

| 命令类型 | 建议 timeout (ms) | 建议 timeout (秒) | 备注 |
|----------|-------------------|-------------------|------|
| 文件读取（cat/head/tail） | 30000 | 30秒 | |
| 文件列表（ls/find） | 30000 | 30秒 | |
| 文件操作（cp/mv/rm） | 30000 | 30秒 | |
| 代码搜索（grep/ag/rg） | 60000 | 60秒 | 大仓库适当增加 |
| git 本地操作（status/diff/log） | 60000 | 60秒 | |
| git 远程操作（fetch/pull/push） | 120000 | 120秒 | |
| 包安装（npm install/pip install） | 300000 | 5分钟 | 网络依赖 |
| 构建/编译（npm run build/pytest） | 600000 | 10分钟 | 大型项目适当增加 |
| 启动开发服务器 | 使用 run_in_background | — | 见规则 4 |

### 正确示例

```
✅ Bash(
    command="npm install",
    timeout=300000,
    description="安装 npm 依赖"
)

✅ PowerShell(
    command="pip install -r requirements.txt",
    timeout=300000,
    description="安装 Python 依赖"
)
```

### 错误示例（禁止）

```
❌ Bash(command="npm install", description="安装依赖")
   # 没有 timeout —— 网络慢时会无限等待！

❌ PowerShell(command="git push", description="推送代码")
   # 没有 timeout —— 远程服务器无响应时会卡死！
```

---

## 规则 2：禁止交互式命令

**强制执行。** 不得执行任何可能进入交互模式（等待用户 stdin 输入）的命令。

### 危险命令及安全替代

| 危险命令 | 安全替代 |
|----------|----------|
| `npm install` | `npm install --yes` 或先设置环境变量 `CI=true` |
| `git push` | 确保已配置 credential helper；如不行则 `git config --global credential.helper store` |
| `ssh user@host` | `ssh -o BatchMode=yes -o ConnectTimeout=10 user@host` |
| `python`（无 `-c` 参数） | 使用 `python -c "..."` 或将代码写入文件后执行 `python file.py` |
| `node`（无脚本参数） | 使用 `node -e "..."` 或将代码写入文件后执行 |
| 任何可能 prompt 的命令 | 预先加 `-y` / `--yes` / `--no-input` / `--non-interactive` 等参数 |

### 检测交互式命令的方法

在执行命令前，问自己：**这个命令在某种情况下会等待用户输入吗？**
- 如果是 → 加上非交互参数，或换一种方式执行
- 如果不确定 → 假设它会，加上超时和 non-interactive 参数

---

## 规则 3：必须定期上报状态

**强制执行。** 每完成 2-3 个工具调用，必须通过 `SendMessage` 向 Lead 汇报一次进度。

### 状态上报格式

```
SendMessage(
    type="message",
    recipient="main",   # 或具体的 Leader agent 名称
    content="进度更新：刚完成 [具体操作]，下一步是 [下一步操作]，预计还需 X 步。",
    summary="进度：刚完成XXX"
)
```

### 上报频率

| 任务类型 | 上报频率 |
|----------|----------|
| 快速任务（< 5 个工具调用） | 任务完成时汇报一次即可 |
| 中等任务（5-15 个工具调用） | 每完成 3 个工具调用汇报一次 |
| 长时间任务（> 15 个工具调用） | 每完成 2 个工具调用汇报一次 |

---

## 规则 4：长时间任务使用 run_in_background

**强制执行。** 以下类型的任务应使用 `run_in_background=true`：

- 启动开发服务器（等待端口就绪后即返回）
- 运行完整测试套件（预计 > 5 分钟）
- 大型构建任务（预计 > 5 分钟）
- 下载大文件

### 正确示例

```
✅ Bash(
    command="npm run build",
    timeout=600000,
    run_in_background=true,
    description="构建项目（后台运行）"
)
# 然后通过 BashOutput 获取结果
```

---

## 规则 5：响应健康检查

当收到来自 Lead 的 `HEALTH_CHECK` 消息时，**必须立即**回复当前状态。

### 健康检查消息格式

Lead 发送的消息内容示例：
```
HEALTH_CHECK：请回复当前状态（正在执行第X步，最后操作为...）
```

### 响应格式

```
SendMessage(
    type="message",
    recipient="main",
    content="HEALTH_CHECK 响应：当前正在执行 [当前操作]，已完成的步骤：[步骤1, 步骤2]，预计还需 X 步完成。",
    summary="Health check 响应"
)
```

**重要：** 收到 `HEALTH_CHECK` 后必须在 **30 秒内**回复，否则会被判定为假死。

---

## 规则 6：遇到阻塞立即上报，不要等待

**强制执行。** 遇到以下情况时，**不要**等待或重试，立即通过 `SendMessage` 上报给 Lead：

- 命令执行失败（且不是预期内的失败）
- 找不到预期的文件或目录
- API 调用返回错误
- 权限不足（Permission denied）
- 网络连接失败（且重试 2 次后仍失败）

### 错误上报格式

```
SendMessage(
    type="message",
    recipient="main",
    content="阻塞上报：在 [操作步骤] 遇到错误：[错误信息]。已尝试 [解决方法]，仍未解决。请指示下一步操作。",
    summary="阻塞：[简短描述问题]"
)
```

---

## 规则 7：工具调用错误的处理

当工具调用返回错误时：

1. **读取错误信息**，判断是否为预期内的错误（如测试用例失败是正常的）
2. **如果是预期内的错误** → 记录结果，继续下一步
3. **如果是未预期的错误** → 立即上报（规则 6）
4. **不要无限重试** —— 重试超过 2 次仍失败 → 上报

---

## 安全检查清单（每次调用工具前快速确认）

- [ ] Bash/PowerShell 调用是否设置了 `timeout`？
- [ ] 命令是否可能进入交互模式？如果是，是否已加非交互参数？
- [ ] 如果是长时间任务，是否应该使用 `run_in_background`？
- [ ] 上次状态上报距今是否超过 3 个工具调用？如果是，先上报再继续？
- [ ] 如果工具调用失败，是否有处理计划（上报 or 重试）？

---

*安全规范 v1.0 — 2026-04-29*
*防止假死，从每次工具调用做起。*
