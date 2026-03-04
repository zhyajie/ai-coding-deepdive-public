# Part 4：使用 Claude Code Debug 精度问题

## 背景

在多平台部署多模态模型时，不同硬件平台（如 NVIDIA H20 vs AMD MI308）可能产生不同的推理结果。本章演示如何使用 Claude Code 系统性地排查跨平台精度问题，从算子层面逐步缩小范围，最终定位到具体模块。

---

## 问题描述

### 输入图片

| 输入图片 1 | 输入图片 2 |
|-----------|-----------|
| <img src="qwen_image_edit_input_21.png" width="300"> | <img src="qwen_image_edit_input_22.png" width="300"> |

### Prompt

```json
{
  "file_name": ["qwen_image_edit_input_21.png", "qwen_image_edit_input_22.png"],
  "caption": "将两个人合成一张合影，让他们并肩站立，面带微笑，背景为山景，保持自然的光线和透视关系。"
}
```

### 结果对比

| H20（正确） | MI308（精度异常） |
|-------------|-----------------|
| <img src="H20_result.png" width="300"> | <img src="Mi308_result.png" width="300"> |

> H20 生成结果正常，MI308 存在明显的精度问题。

### 运行环境

- 服务端启动脚本：`launch_serving.sh`
- 客户端测试脚本：`bench_client.sh`

---

## Debug 流程概览

```
Session 1: 算子层排查
  └─ 替换 aiter 算子 → 替换 torch 算子 → 排除算子问题

Session 2: 模块层定位
  └─ H20 dump 各模块输出 → MI308 逐模块加载 → 定位问题模块

Session 3: 模块内精细定位
  └─ 对问题模块编写单测 → 逐层对比 → 定位根因
```

---

## Session 1：算子层排查

### 目标

排除底层算子实现（aiter / torch 自定义算子）导致精度问题的可能性。

### Prompt（可直接复制）

```
现在有一个精度问题，图片分别是 qwen_image_edit_input_21.png，
qwen_image_edit_input_22.png。

{"file_name": ["qwen_image_edit_input_21.png","qwen_image_edit_input_22.png"],
 "caption": "将两个人合成一张合影，让他们并肩站立，面带微笑，背景为山景，
 保持自然的光线和透视关系。"}

在 H20 和 MI308 会生成不同的图片结果，其中 MI308 有精度问题。
H20_result.png  Mi308_result.png

@launch_serving.sh @bench_client.sh 这个分别是我的服务启动脚本和客户端。

我的排查思路如下，严格按照我的排查思路运行：

1. 替换 MI308 上所有的 aiter 实现换成非 aiter 的实现，实现完成后帮我
   总结成一个 markdown 文档，然后自动运行服务端和客户端，观察输出结果
   是否正常。

2. 如果替换完所有的 aiter 算子依然异常，替换所有的 moe rms rope
   attention 都换成 torch 的实现，然后自动运行服务端和客户端，观察输出
   结果是否正常。
```

### 预期交互过程

1. Claude 阅读代码，找到所有 aiter 算子的调用点
2. 逐一替换为非 aiter 实现，生成总结文档
3. 自动启动服务端和客户端，检查结果
4. 若仍异常，继续替换 moe/rms/rope/attention 为 torch 原生实现
5. 再次运行验证

### Session 1 结束

经过多轮交互，确认算子层面无问题后，让 Claude 输出总结：

```
把当前所有的排查过程和结论写一个总结文档，markdown 格式保存。
```

> Claude 会生成类似 `session1_summary.md` 的文档，记录排查步骤和结论。

---

## Session 2：模块层定位

### 目标

Qwen Image Edit 模型由多个模块组成（Qwen2.5-VL → Qwen2.5 LLM → DIT → VAE），通过 H20 dump + MI308 逐模块加载的方式，定位具体是哪个模块导致精度差异。

### Prompt（可直接复制）

```
现在有一个精度问题，图片分别是 qwen_image_edit_input_21.png，
qwen_image_edit_input_22.png。

{"file_name": ["qwen_image_edit_input_21.png","qwen_image_edit_input_22.png"],
 "caption": "将两个人合成一张合影，让他们并肩站立，面带微笑，背景为山景，
 保持自然的光线和透视关系。"}

在 H20 和 MI308 会生成不同的图片结果，其中 MI308 有精度问题。
H20_result.png  Mi308_result.png

@launch_serving.sh @bench_client.sh 这个分别是我的服务启动脚本和客户端。

我已经排查了一段时间 @session1_summary.md ，排除了 torch 算子和 aiter
算子的问题，现在我要进行模块级 debug。

你阅读 sglang 的代码，qwen_image_edit 由 Qwen2.5-VL + Qwen2.5 LLM
+ DIT + VAE 部分组成。

我的思路是这样的，让 H20 dump 每个模块的输出，MI308 逐步加载每个模块
（从后往前加载），就能定位是哪个模块的精度问题。需要你帮我实现以下代码：

1. H20 帮我写段代码，功能如下：通过环境变量 dump_name="xx1,xx2"
   （xx 是各个模块名字）dump 各模块的输出 tensor，以及
   load_name="xx" 能自动 load 已有的 tensor。先和我沟通好细节。

2. MI308 上，拉取 H20 相同的代码。分别去执行 load_name="xx"，
   自动去执行并保存图片。
```

### 预期交互过程

1. Claude 分析模型架构，列出各模块及其输入输出
2. 与你确认 dump/load 的具体细节（tensor 格式、保存路径、命名规则等）
3. 实现 dump/load 代码，支持环境变量控制
4. 在 H20 上 dump 所有模块输出
5. 在 MI308 上从后往前逐模块加载 H20 的 tensor，替换对应模块的输出
6. 每次替换后运行推理，对比结果是否恢复正常
7. 最终确认问题模块（例如：Qwen2.5-VL）

### Session 2 结束

确认问题模块后，让 Claude 输出总结：

```
把当前的 debug 过程和结果总结成 markdown 文档保存。
```

---

## Session 3：模块内精细定位

### 目标

已确认 Qwen2.5-VL 存在精度问题，进一步对该模块内部进行逐层单测，定位到具体的子模块或层。

### Prompt（可直接复制）

```
现在有一个精度问题，图片分别是 qwen_image_edit_input_21.png，
qwen_image_edit_input_22.png。

{"file_name": ["qwen_image_edit_input_21.png","qwen_image_edit_input_22.png"],
 "caption": "将两个人合成一张合影，让他们并肩站立，面带微笑，背景为山景，
 保持自然的光线和透视关系。"}

在 H20 和 MI308 会生成不同的图片结果，其中 MI308 有精度问题。
H20_result.png  Mi308_result.png

@launch_serving.sh @bench_client.sh 这个分别是我的服务启动脚本和客户端。

我已经确认了 Qwen2.5-VL 存在精度问题，帮我查看 transformer 中
Qwen2.5-VL 的代码，输入 shape 保持当前 shape 去详细的单测每个模块。
```

### 预期交互过程

1. Claude 阅读 Qwen2.5-VL 的模型代码（transformer 实现）
2. 分析内部结构（Attention、FFN、Norm、Embedding 等）
3. 基于当前实际的输入 shape，为每个子模块编写单测
4. 在 H20 和 MI308 上分别运行单测，逐层对比输出差异
5. 定位到具体的问题层/算子

---

## 交流要点

### Debug 方法论

本案例展示了一套系统性的跨平台精度 Debug 方法：

```
第一步：排除算子层   （替换 aiter → 替换 torch）
第二步：定位模块层   （dump/load 逐模块替换）
第三步：模块内精细定位（逐层单测对比）
```

**核心思路**：自顶向下、逐层缩小范围。每一步都有明确的验证手段。

### Claude Code 在 Debug 中的价值

| 环节 | 手动 Debug | Claude Code 辅助 |
|------|-----------|-----------------|
| 找到所有 aiter 调用点 | 手动 grep + 逐个确认 | Claude 自动分析代码依赖 |
| 批量替换算子实现 | 手动修改多个文件 | Claude 一次性完成替换 |
| 编写 dump/load 代码 | 手动实现，容易遗漏边界 | Claude 根据模型结构自动生成 |
| 编写逐层单测 | 耗时且需理解每层的 shape | Claude 阅读代码后自动推断 shape |
| 运行验证 | 手动执行、手动检查 | Claude 自动运行并分析结果 |
| 过程记录 | 容易遗忘排查过程 | Claude 自动生成总结文档 |

### 关键技巧

1. **用 `@` 引用文件**：`@launch_serving.sh` 让 Claude 直接读取脚本内容
2. **引用上一轮总结**：`@session1_summary.md` 让新 Session 继承排查上下文
3. **严格指定排查步骤**：避免 Claude 自由发挥，按你的思路执行
4. **每轮结束生成文档**：保留排查记录，也方便跨 Session 传递上下文
5. **图片直接拖入 Claude Code**：Claude 可以看到图片内容，直观对比结果差异
