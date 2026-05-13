# Needle：用 2600 万参数重定义边缘 AI

## 从 Gemini 到 26M 的蒸馏奇迹

Cactus Compute 团队最近在 Hacker News 上发布了 Needle —— 一个将 Gemini 3.1 的工具调用能力蒸馏到仅 2600 万参数的模型。更令人惊叹的是，这个模型在 16 个 TPU v6e 上训练了 200B tokens 只用了 27 小时，后训练在 2B tokens 的单次函数调用数据集上只用了 45 分钟。

 Needle 采用了他们自研的「简单注意力网络」（Simple Attention Networks, SAN）架构，在 Cactus 平台上可实现 6000 tokens/sec 的预填充速度和 1200 的解码速度。权重和训练数据集完全开源。

## 颠覆性的架构决策：完全移除 FFN

传统 Transformer 模型的参数中，约 2/3 都被前馈网络（FFN）占据。但 Needle 团队做了一个大胆的决策：**完全移除 FFN**。

他们的理由很清晰：

**Softmax 本身就是非线性的。** 标准注意力机制中的 `softmax(QK^T/sqrt(d)) * V` 是一个数据依赖的非线性混合操作。对于工具调用这种「查询匹配工具」的任务，注意力机制本身就是正确的原语。

**工具调用本质是检索与组装。** 匹配查询到工具名、提取参数值、组装 JSON —— 这三个步骤都是输入与输出之间的对齐与复制，这正是交叉注意力擅长的事情。没有任何一个步骤需要逐位置的特征变换（FFN 的主要功能）。

**小模型上 FFN 参数浪费。** 对于 5000 万参数以下的模型执行结构化任务，更多注意力层（更深的交叉注意力 = 更好的查询工具对齐）比 FFN 更有价值。

**更少参数 = 更快推理。** FFN 占用了最大的 GEMM/GEMV 维度 —— 移除它们使得每层参数减少了约 2/3，直接降低了在边缘设备上占主导地位的内存带宽瓶颈。

## 编码器-解码器架构的理性选择

Needle 采用了编码器-解码器（Encoder-Decoder）架构，而非流行的因果解码器（Causal Decoder），理由同样充分：

**双向编码。** 工具是结构化对象 —— 双向编码器能一次性看到完整的定义。因果模型只能从左到右读取，必须从部分上下文中推断结构。

**KV 缓存中没有输入 token。** 编码器-解码器使用固定大小的编码器表示进行交叉注意力，而不是在每个生成步骤时重新关注整个输入。

**多头设计的自然契合。** 编码器输出同时馈送解码器（生成）和对比头（工具检索），实现了清晰的分离。

## 关键技术创新

### 门控残差（Gated Residuals）

没有 FFN 就意味着没有逐位置的非线性重写，这使得残差连接的设计变得至关重要。他们提出了门控残差：

`x = x + sigmoid(gate) * Attn(Norm(x))`

每个子层都有一个可学习的标量门，初始化为 0（sigmoid(0) = 0.5）。训练从半强度的残差开始，模型可以学习增强有用的层（g -> 1）或抑制无用的层（g -> 0），同时不会丢失梯度流。

### ZCRMSNorm

与标准 RMSNorm（`x * gamma / RMS(x)`，gamma 初始化为 1）不同，ZCRMSNorm 使用 `x * (1 + gamma) / RMS(x)`，其中 gamma 初始化为 0。

初始化时，ZCRMSNorm 本质上是一个恒等缩放。这与门控残差搭配：整个块开始时就是一个阻尼恒等 + 阻尼归一化注意力。没有任何组件以强学习偏差启动。

### INT4 量化感知训练（QAT）作为正则化

每 100 步进行一次「假量化」：权重被分组量化到 INT4（对称，group_size=32），然后使用直通估计器（STE）进行前向传播。

量化噪声充当了一种权重噪声正则化，阻止模型过度依赖精确的权重值。对于没有 FFN 的小模型（参数更少，过拟合风险更高），这特别有益。

更重要的是，模型训练时使用的是它将在推理时看到的相同量化，因此不存在后训练量化差距。

## 性能表现

Needle 在单次函数调用任务上击败了 FunctionGemma-270m、Qwen-0.6B、Granite-350m、LFM2.5-350m 等模型。

但他们也很诚实：这些模型在对话设置中表现更好，因为它们有更多的范围/容量。此外，小模型可能比较挑剔（finicky）。

## 实用性与部署

Needle 提供了完整的开箱即用体验：

```bash
git clone https://github.com/cactus-compute/needle.git
cd needle && source ./setup
needle playground
```

这会在 http://127.0.0.1:7860 打开一个 Web UI，你可以在其中测试并微调你自己的工具。权重会自动下载。

Python 使用也很简单：

```python
from needle import SimpleAttentionNetwork, load_checkpoint, generate, get_tokenizer

params, config = load_checkpoint("checkpoints/needle.pkl")
model = SimpleAttentionNetwork(config)
tokenizer = get_tokenizer()

result = generate(
    model, params, tokenizer,
    query="What's the weather in San Francisco?",
    tools='[{"name":"get_weather","parameters":{"location":"string"}}]',
    stream=False,
)
print(result)
# [{"name":"get_weather","arguments":{"location":"San Francisco"}}]
```

## 潜在影响与局限

Needle 展示了一个重要方向：**专门为特定任务优化的极小模型**。对于工具调用这种结构化任务，移除 FFN、使用编码器-解码器、采用门控残差等设计决策都是高度合理的。

这对于边缘 AI（手机、手表、眼镜等）来说可能是一个游戏规则改变者。这些设备通常算力受限，但又需要快速响应。

但我们也需要保持清醒：

1. **任务特异性。** Needle 在工具调用上很强，但这并不意味着它在其他任务上同样出色。
2. **模型稳定性。** 小模型可能比较挑剔，需要仔细的微调。
3. **生态系统。** 实际部署还需要考虑工具定义、错误处理、安全验证等工程问题。

## 结语

Needle 展示了深度学习模型架构设计中「少即是多」的魅力。通过深入理解任务的本质（检索与组装），他们移除了不必要的组件（FFN），优化了关键部分（注意力深度、残差设计），最终实现了一个既小巧又高效的模型。

这种「为任务量身定制架构」的思维模式，可能是未来边缘 AI 发展的一个重要方向。与其追求通用的「越大越好」，不如深入理解具体任务需求，设计恰到好处的模型架构。

---

*相关链接：*
- GitHub: https://github.com/cactus-compute/needle
- Hugging Face: https://huggingface.co/Cactus-Compute/needle
