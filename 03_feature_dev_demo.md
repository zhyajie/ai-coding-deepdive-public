# Part 3：Feature 开发实战 Demo

## 项目背景

**项目**：SGLang — 大规模语言模型和多模态模型的高效推理框架

**问题**：当前 SGLang 的 `qwen_image_edit` 多模态模型（Qwen-Image-Edit）缺乏在线服务的 Profiling 功能。LLM 模型已经支持在线 Profiling，但多模态模型（特别是图像编辑模型）还不具备该能力。

**目标**：为 Qwen 多模态图像编辑模型添加 Profiling 支持，实现与 LLM 模型一致的 Profiling 体验 —— 通过环境变量配置，服务端自动收集 Profiling 数据并保存到指定目录。

---

## 开发环境

### 服务端启动脚本

```bash
export SGLANG_TORCH_PROFILER_DIR=./sglang_qwen_profiling
export SGLANG_PROFILE_WITH_STACK=1
export SGLANG_PROFILE_RECORD_SHAPES=1
sglang serve  \
        --model-path /mnt/raid0/pretrained_model/Qwen-Image-Edit-2511 \
        --num-gpus 1 \
        --ulysses-degree 1 \
        --tp-size 1 \
        --image-encoder-precision bf16 \
        --vae-precision bf16 \
        --host 0.0.0.0 \
        --port 40000 \
        --text-encoder-cpu-offload False \
        --vae-cpu-offload False \
        --image-encoder-cpu-offload False \
        --use-fsdp-inference False \
        --dit-cpu-offload False \
        2>&1 | tee qwen_trace.log
```

### 客户端 Benchmark 脚本

```bash
python3 -m sglang.multimodal_gen.benchmarks.bench_serving \
    --backend sglang-image --task image-to-image \
    --dataset vbench \
    --dataset-path /home/yajizhan/dev/benchmark_data_oneImage/ \
    --max-concurrency 1 --num-prompts 2 \
    --num-inference-steps 8 --guidance-scale 1 \
    --warmup-requests 1 \
    --profile
```

### 预期结果

客户端发送带 `--profile` 的 Benchmark 请求后，服务端在 `./sglang_qwen_profiling` 目录下自动保存 PyTorch Profiler 的 Profiling 文件（`.json` trace 文件），可通过 Chrome `chrome://tracing` 或 TensorBoard 查看。

---

## Demo 流程

### Step 1: 给出开发 Prompt

> 以下 Prompt 可在演示时直接复制粘贴到 Claude Code 中使用。

```
当前 SGLang 的 Qwen Image Edit 多模态模型缺少在线服务的 Profiling 功能。
我要实现的功能类似于 LLM 模型已有的 Profiling 支持。

具体需求：
1. 服务端通过环境变量启用 Profiling：
   - SGLANG_TORCH_PROFILER_DIR: 指定 profiling 文件保存目录
   - SGLANG_PROFILE_WITH_STACK: 是否记录调用栈
   - SGLANG_PROFILE_RECORD_SHAPES: 是否记录 tensor shapes
2. 客户端 bench_serving 脚本通过 --profile 参数触发 profiling
3. Profiling 数据自动保存到 SGLANG_TORCH_PROFILER_DIR 指定的目录

服务端启动命令：
export PYTHONPATH=/home/yajizhan/qwen_code/sglang_main/python
export SGLANG_TORCH_PROFILER_DIR=./sglang_qwen_profiling
export SGLANG_PROFILE_WITH_STACK=1
export SGLANG_PROFILE_RECORD_SHAPES=1
sglang serve \
    --model-path /mnt/raid0/pretrained_model/Qwen-Image-Edit-2511 \
    --num-gpus 2 --ulysses-degree 2 --tp-size 1 \
    --image-encoder-precision bf16 --vae-precision bf16 \
    --host 0.0.0.0 --port 40000 \
    --text-encoder-cpu-offload False --vae-cpu-offload False \
    --image-encoder-cpu-offload False --use-fsdp-inference False \
    --dit-cpu-offload False \
    2>&1 | tee qwen_trace.log

客户端启动命令：
export PYTHONPATH=/home/yajizhan/qwen_code/sglang_main/python
python3 -m sglang.multimodal_gen.benchmarks.bench_serving \
    --backend sglang-image --task image-to-image \
    --dataset vbench \
    --dataset-path /home/yajizhan/dev/benchmark_data_oneImage/ \
    --max-concurrency 1 --num-prompts 2 \
    --num-inference-steps 8 --guidance-scale 1 \
    --warmup-requests 1 \
    --profile

请先分析相关代码，了解 LLM 模型的 Profiling 实现方式，然后和我确认需求细节，
再开始开发。开发完成后，请自动运行服务端和客户端脚本，确认 profiling 文件
已成功生成在 sglang_qwen_profiling 目录中。最后帮我准备一份全英文的 PR 描述，
我要去社区提 PR。
```

### Step 2: Claude 分析代码并确认需求

Claude 会：
1. 阅读 LLM 模型中 Profiling 的实现代码，理解现有架构
2. 阅读多模态模型的推理路径，找到需要插入 Profiling 的位置
3. 与你确认需求细节（profiling 粒度、数据格式等）

### Step 3: Claude 实现功能

Claude 会：
1. 在多模态模型的推理路径中集成 PyTorch Profiler
2. 复用 LLM 模型已有的 Profiling 基础设施（环境变量、文件保存逻辑等）
3. 确保 `--profile` 参数在客户端和服务端正确传递

### Step 4: 自动测试验证

Claude 会：
1. 启动服务端（带 Profiling 环境变量）
2. 运行客户端 Benchmark（带 `--profile` 参数）
3. 检查 `sglang_qwen_profiling` 目录中是否生成了 Profiling 文件
4. 确认文件格式正确

### Step 5: 准备 PR

Claude 会生成全英文的 PR 描述，包含：
- 变更摘要
- 实现细节
- 测试方法
- 相关 Issue（如有）

---

## 交流要点

### 关键观察点

1. **代码理解能力**：Claude 如何快速理解大型代码库的架构
2. **需求澄清**：Claude 在开发前主动确认需求细节，而非盲目编码
3. **代码复用**：Claude 如何发现并复用已有的 Profiling 基础设施
4. **端到端验证**：Claude 不只写代码，还会运行测试确认功能可用
5. **PR 准备**：Claude 自动生成社区级别的 PR 描述

### Claude Code 的核心价值

| 传统开发 | Claude Code 辅助开发 |
|----------|---------------------|
| 手动浏览代码库理解架构 | Claude 自动分析代码结构和依赖 |
| 在不同文件间来回查找参考实现 | Claude 并行阅读多个相关文件 |
| 手动编写代码、测试、调试 | Claude 一站式完成开发-测试-验证 |
| 手动撰写 PR 描述 | Claude 自动生成规范的 PR 描述 |
| 需要熟悉整个代码库 | Claude 即时学习并理解代码上下文 |
