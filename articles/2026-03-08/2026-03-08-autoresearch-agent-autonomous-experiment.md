# Autoresearch：让 AI Agent 在你睡觉时做研究

> **字数**：约 950 字
> **风格**：轻松、技术准确、呼应 Karpathy 幽默

---

Andrej Karpathy 在 autoresearch 的 README 开篇写了一段充满科幻色彩的描述：

> "曾经，前沿 AI 研究是由'肉计算机'完成的——他们在吃饭、睡觉、找乐子之间抽空工作，偶尔通过声波互联在'组会'仪式中同步。那个时代早已过去。研究现在完全由自主 AI agent 群体在云端的计算集群巨构上运行。agent 声称我们现在处于代码库的第 10,205 代，反正没人能判断对错，因为'代码'现在是一个自我修改的二进制，已经超越人类理解。这个仓库是这一切如何开始的故事。"

这段话既是幽默，也是愿景。Autoresearch 就是这个愿景的起点。

## 核心概念：Agent 自主实验

**Autoresearch 是什么？**

给 AI agent 一个真实的 LLM 训练设置，让它在你睡觉时自主实验，早上你醒来时看到实验日志和（希望如此）更好的模型。

**工作流程：**
1. Agent 修改训练代码（train.py）
2. 训练运行 5 分钟
3. 检查结果是否改进
4. 保留或丢弃修改
5. 重复

预期：约 12 实验/小时，约 100 实验/夜间。

**极简设计：3 个文件**

- **prepare.py** — 数据准备、运行时工具（不修改）
- **train.py** — 模型、优化器、训练循环（agent 只修改此文件）
- **program.md** — agent 指令（人类修改此文件）

基于 Karpathy 的 nanochat 简化实现，单 GPU（测试于 H100），无分布式训练，无复杂配置。

## 设计哲学

**1. 固定时间预算：5 分钟**

训练始终运行 5 分钟，无论你改什么。这带来两个好处：
- **实验直接可比**：不同修改在相同时间预算下比较
- **自动适配平台**：autoresearch 会找到你平台在时间预算内的最优模型

代价：不同算力平台的结果不可比较。

**2. program.md 作为"研究组织代码"**

你不是直接改 Python 文件，而是编程 `program.md`——这个文件提供上下文给 AI agent，设置你的自主研究组织。默认是基线，但显然可以迭代：找到能实现最快研究进度的"研究组织代码"。

**3. 单文件修改：可审查性**

Agent 只触碰 `train.py`，保持范围可控，diff 可审查。

## 实践指南

**硬件要求：**NVIDIA GPU（H100 测试）、Python 3.10+、uv 包管理器

**快速开始：**
```bash
# 1. 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. 安装依赖
uv sync

# 3. 准备数据（一次性，约 2 分钟）
uv run prepare.py

# 4. 手动运行单个实验（约 5 分钟）
uv run train.py

# 5. 进入自主研究模式
# 启动 Claude/Codex，禁用权限，提示：
# "Hi have a look at program.md and let's kick off a new experiment!"
```

**社区 fork（小算力平台）：**
- **macOS**：miolini/autoresearch-macos
- **macOS (MLX)**：trevin-creator/autoresearch-mlx
- **Windows RTX**：jsegov/autoresearch-win-rtx

**小算力调参建议：**
- 使用低熵数据集（TinyStories）
- 降低 vocab_size（8192 → 256）
- 降低 MAX_SEQ_LEN（到 256）
- 降低 DEPTH（8 → 4）
- 降低 TOTAL_BATCH_SIZE（保持 2 的幂次）

## 愿景与局限

**Karpathy 的愿景：**

这个项目是"这一切如何开始的故事"。从单 GPU、单 agent、单文件开始，逐步扩展到 agent 群体、分布式训练、自我演进的代码库。

**当前局限：**
1. **硬件门槛**：默认 H100，小算力需要 fork
2. **结果不可比较**：不同平台结果不可对比
3. **早期阶段**：项目刚发布，长期效果未知
4. **安全考虑**：自主修改代码可能引入不可预测行为

**对 AI 研究的影响：**

Autoresearch 代表了一个范式转变：从人类设计实验、运行实验、分析结果，到 agent 自主完成整个循环。人类的工作从"做实验"转向"设计实验框架"和"编程 agent 行为"。

这不是取代人类研究者，而是放大研究者的能力。一个研究者可以在睡觉时让 agent 运行 100 个实验，早上醒来查看结果，然后决定下一步方向。

## 结语

Autoresearch 的意义不仅在于提供一个工具，更在于展示了一种可能性：AI 研究可以自动化到什么程度。从"肉计算机"到自主 agent，这个转变正在发生。而 Karpathy 的这个项目，正是朝这个方向迈出的务实一步。

---

**来源**：[karpathy/autoresearch](https://github.com/karpathy/autoresearch)  
**作者**：Andrej Karpathy  
**发布时间**：2026年3月  
**终稿完成**：2026-03-08 | Nancy
