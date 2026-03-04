# Part 5：使用心得与真实案例

---

## 使用心得

### 1. 小任务拆解，成功率更高

Claude Code 处理小而明确的任务时成功率显著更高。面对大型任务，建议拆解成多个小任务逐步完成，而不是一次性给出一个庞大的需求。每个小任务都有清晰的输入、输出和验证标准，Claude 更容易准确理解和执行。

### 2. 上下文管理：及时总结，适时开启新会话

Claude Code 的上下文窗口为 1M tokens，超出后会自动压缩上下文（compact），压缩后模型效果会有折扣。建议：

- 当上下文接近上限时，让 Claude 总结当前进展和结论，保存为 markdown 文档
- 剔除无用的探索方向和冗余信息
- 开启一个全新会话，用 `@总结文档.md` 引入之前的上下文，继续工作

这样既保持了连续性，又确保 Claude 始终在最佳状态下工作。

### 3. 限制发散，用文档引导方向

大模型容易思路发散，可能走入错误的排查方向或尝试绕过问题。错误的探索还会污染上下文，影响后续判断。应对策略：

- 多用文档总结已确认的正确排查思路
- 在 Prompt 中明确限制排查方向（如"严格按照我的排查思路运行"）
- 发现模型走偏时，果断开启新 Session，用总结文档重新引导

---

## 真实案例

以下是使用 Claude Code 完成的实际开源贡献，均已提交到 SGLang 社区：

### 案例一：Diffusion Profiling 支持

> PR：[[Diffusion] [Profiling] Add end-to-end profiling support for diffusion serving pipeline](https://github.com/sgl-project/sglang/pull/18367)

为 SGLang 的 Diffusion 推理管线添加端到端 Profiling 支持。通过环境变量配置，可同时采集 HTTP Server 和 GPU Worker 的 `torch.profiler` trace，与 LLM 的 Profiling API 保持一致。

**对应本次分享**：[Part 3 — Feature 开发实战](./03_feature_dev_demo.md)

### 案例二：Ulysses SP 精度修复

> PR：[[Diffusion] Fix Ulysses SP for QwenImageEdit: shard text/cond/noisy tokens correctly](https://github.com/sgl-project/sglang/pull/18516)

修复 QwenImageEdit 在 `ulysses_degree >= 2` 时输出损坏的问题。根因是只有 noisy latent tokens 被正确分片，而 condition image tokens、text embeddings 及对应的 RoPE、modulate_index 未被分片，导致 attention 和 modulation 的 shape 不匹配。

**对应本次分享**：[Part 4 — Debug 精度问题实战](./04_debug_precision_demo.md)

### 案例三：Attention Variants 技术报告

> 报告：[Attention Variants in Multimodal Diffusion Models](https://github.com/zhyajie/sglang/blob/attention_report/attention_variants_report.md)

使用 Claude Code 辅助完成的技术调研报告，系统梳理了 STA、SageAttention 等注意力加速方案在视频/图像 Diffusion 模型中的原理、Benchmark 数据和硬件适配分析。
