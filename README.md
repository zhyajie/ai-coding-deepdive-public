# Claude Code 技术分享

> AI-Assisted Development — 从安装到实战

---

## 分享目标

通过本次交流，大家将能够：

1. 在 Linux 环境下通过 AMD LLM Gateway 安装并配置 Claude Code
2. 理解 Skills 和 MCP 的概念，并掌握常用 Skills 的使用方法
3. 通过一个真实的 Feature 开发案例，体验 Claude Code 驱动的完整开发流程
4. 通过一个跨平台精度 Debug 案例，掌握 Claude Code 辅助系统性排查的方法

---

## 内容大纲

### Part 1：安装与配置

> 文档：[01_installation_guide.md](./01_installation_guide.md)

- 安装 Node.js 和 Claude Code
- 配置 AMD LLM Gateway 连接
- 绕过 Anthropic OAuth 登录
- 环境变量与权限设置
- 首次启动验证

### Part 2：Skills & MCP

> 文档：[02_skills_and_mcp.md](./02_skills_and_mcp.md)

- **什么是 Skill？** — Claude Code 的可复用指令模板，通过 `/command` 快速触发预定义的工作流
- **什么是 MCP？** — Model Context Protocol，让 Claude 连接外部工具和服务的标准化协议
- 已安装 Skills 一览与使用方法
- 已配置 MCP Server 说明

### Part 3：Feature 开发实战

> 文档：[03_feature_dev_demo.md](./03_feature_dev_demo.md)

- 项目背景：SGLang Qwen 多模态模型 Profiling 功能
- 完整开发流程演示：需求分析 → 代码开发 → 测试验证 → PR 准备
- 可直接复制使用的 Prompt

### Part 4：Debug 精度问题实战

> 文档：[04_debug_precision_demo.md](./04_debug_precision_demo.md)

- 跨平台精度问题排查（H20 vs MI308）
- 三轮 Session 逐步缩小范围：算子层 → 模块层 → 模块内精细定位
- 多 Session 间通过总结文档传递上下文
- 可直接复制使用的 Prompt

### Part 5：使用心得与真实案例

> 文档：[05_best_practices.md](./05_best_practices.md)

- 使用心得：小任务拆解、上下文管理、限制发散
- 真实案例：SGLang 社区 PR 及技术报告

---

## 快速参考

### 常用命令

| 命令 | 说明 |
|------|------|
| `claude` | 启动 Claude Code（已通过 alias 绑定 opus 模型） |
| `claude --model claude-sonnet-4` | 使用 Sonnet 模型启动 |
| `/review` | 审查当前 git diff |
| `/fix <描述>` | 诊断并修复 Bug |
| `/explain <文件>` | 解释代码逻辑 |
| `/test <文件>` | 生成测试用例 |
| `/perf <文件>` | 性能分析 |
| `Ctrl+C` | 中断当前生成 |
| `/clear` | 清空对话历史 |
| `/compact` | 压缩对话上下文 |

### 使用技巧

1. **给出明确的上下文**：告诉 Claude 项目背景、文件位置、期望行为
2. **分步迭代**：复杂任务拆分成多步，逐步引导
3. **利用 Plan Mode**：对于复杂任务，先让 Claude 制定计划再执行
4. **善用 Skills**：`/review`、`/fix`、`/test` 等内置 Skill 大幅提升效率
5. **检查输出**：始终审查 Claude 的代码修改，确认符合预期
