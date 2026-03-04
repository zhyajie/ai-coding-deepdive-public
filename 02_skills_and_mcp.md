# Part 2：Skills & MCP

---

## 核心概念

### 什么是 Skill？

**Skill** 是 Claude Code 的可复用指令模板 —— 一组预定义的 Prompt + 工作流，封装成斜杠命令（如 `/review`、`/fix`）。

**类比**：Skill 就是 Claude 的 SOP 手册，告诉他做代码审查该检查哪些项、修 Bug 该按什么步骤排查。

```
/review                      # 审查当前 git diff
/fix 登录页面点击无反应        # 修复指定 Bug
/explain src/server.py       # 解释代码逻辑
```

### 什么是 MCP？

**MCP（Model Context Protocol）** 是 Anthropic 推出的开放协议，让 Claude 能连接和使用外部工具与服务（访问网页、查询数据库、调用 API 等）。

**类比**：Skill 是"做事的方法论"，MCP 是"连接外部世界的桥梁"。

| 维度 | Skill | MCP |
|------|-------|-----|
| 本质 | 预定义的 Prompt 模板 | 外部工具连接协议 |
| 作用 | 规范 Claude 的工作流程 | 扩展 Claude 的能力边界 |
| 使用方式 | `/command` 斜杠命令触发 | Claude 自动调用已注册的工具 |
| 举例 | `/review` 代码审查 | `fetch` 抓取网页内容 |

---

## 已安装 Skills 一览

> 绝大部分 Skill 通过 Anthropic 官方 Skills 仓库一键安装：
>
> ```
> /install anthropics/skills
> ```
>
> 仓库地址：https://github.com/anthropics/skills

### 开发工具类

| 命令 | 功能 | 用法 |
|------|------|------|
| `/fix` | Bug 诊断与修复 | `/fix <bug 描述>` |
| `/explain` | 代码逻辑解释 | `/explain <文件路径>` |
| `/refactor` | 代码重构 | `/refactor <文件路径>` |
| `/review` | 代码审查（基于 git diff） | `/review` |
| `/perf` | 性能分析与优化 | `/perf <文件路径>` |
| `/test` | 生成测试用例 | `/test <文件路径>` |
| `/security` | 安全审计 | `/security <文件或项目>` |
| `/migrate` | 技术/依赖迁移升级 | `/migrate <技术名>` |

### 文档类

| 命令 | 功能 | 用法 |
|------|------|------|
| `/doc` | 生成/更新代码文档 | `/doc <文件路径>` |
| `/doc-coauthoring` | 引导式协作撰写文档 | 告诉 Claude 你想写文档即可 |
| `/git-summary` | Git 活动摘要 | `/git-summary` |

### 文件处理类（自动触发）

| Skill | 触发条件 | 能力 |
|-------|----------|------|
| PDF | 提及 `.pdf` 文件 | 读取、合并、拆分、OCR、加密等 |
| DOCX | 提及 `.docx` 文件 | 创建/编辑 Word 文档 |
| PPTX | 提及幻灯片/演示文稿 | 创建/编辑幻灯片 |
| XLSX | 涉及 `.xlsx`/`.csv` 文件 | 电子表格、公式、图表 |

### 前端与设计类（自动触发）

| Skill | 功能 |
|-------|------|
| Frontend Design | 创建生产级前端界面 |
| Web Artifacts Builder | 复杂前端（React + Tailwind + shadcn/ui） |
| Webapp Testing | Playwright 驱动的 Web 应用测试 |
| Canvas Design | 视觉设计（PNG/PDF） |
| Algorithmic Art | p5.js 算法艺术 |
| Theme Factory | 10 种预设主题，可自定义 |

### MCP & Skill 开发类

| Skill | 功能 |
|-------|------|
| MCP Builder | 构建 MCP Server（Python/Node.js） |
| Skill Creator | 创建/优化/评估 Skill |

---

## 手动安装的 Skills

以下 Skill 需要单独安装：

### Find Skills

```bash
# 安装
npx skills add vercel-labs/skills -g -y
```

- **功能**：从开源生态搜索和安装新 Skill
- **浏览**：https://skills.sh/

### Code Simplifier

```bash
# 在 Claude Code 中执行
/install anthropics/claude-plugins-official
# 或手动下载放到 ~/.claude/skills/code-simplifier.md
```

- **功能**：自动简化和优化代码，只改写法不改行为

### Ralph Wiggum — 迭代开发循环

```bash
# 在 Claude Code 中执行
/install anthropics/claude-code
# 或手动下载放到 ~/.claude/skills/ralph-wiggum/
```

- **功能**：Claude 反复执行任务直到满足完成条件
- **命令**：`/ralph-loop "任务描述" --max-iterations 20 --completion-promise "DONE"`

### Superpowers 插件

```bash
# 在 Claude Code 中执行
/install anthropics/claude-plugins-official
```

| Skill | 说明 |
|-------|------|
| `brainstorming` | 开发前的需求探索和设计 |
| `writing-plans` | 编写多步实现计划 |
| `test-driven-development` | 测试驱动开发 |
| `systematic-debugging` | 系统性调试方法论 |
| `verification-before-completion` | 提交前验证结果正确 |
| `dispatching-parallel-agents` | 并行分发任务给多个 Agent |
| `requesting-code-review` | 完成后请求代码审查 |

---

## 已配置的 MCP Server

### Fetch — 网页内容抓取

配置文件 `~/.claude/.mcp.json`：

```json
{
  "mcpServers": {
    "fetch": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-fetch@latest"]
    }
  }
}
```

让 Claude 能访问和抓取网页内容，用于查阅文档、获取 API 参考等。

---

## Skill 安装方式汇总

| 方式 | 命令 | 说明 |
|------|------|------|
| 官方 Skills 仓库 | `/install anthropics/skills` | 一键安装所有官方 Skill |
| 官方 Plugins | `/install anthropics/claude-plugins-official` | Superpowers 等插件 |
| skills.sh 生态 | `npx skills add <package> -g -y` | 社区第三方 Skill |
| 手动安装 | 将 `.md` 文件放到 `~/.claude/commands/` | 自定义 Skill |
