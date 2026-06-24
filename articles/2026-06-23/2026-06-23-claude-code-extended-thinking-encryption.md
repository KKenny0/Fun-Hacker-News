# Claude Code 的「Extended Thinking」并非真思考：加密机制与企业墙

## 背景

Patrick McCanna 在使用 Claude Code 时发现，本地的会话日志中包含"thinking blocks"（模型的自身推理），但这些 blocks 只有一个 600 字符的签名，无法读取实际内容。进一步查阅官方文档后，他揭示了 Anthropic 的 Extended Thinking 功能并非透明。

## 核心发现

### 1. 加密存储，密钥在云端

根据 Anthropic 官方文档，Claude 会将推理过程加密成一个 signature：
- **加密后存储**：本地日志中的 thinking blocks 仅包含加密后的签名
- **密钥不在本地**：解密密钥由 Anthropic 持有，本地设备无法获取
- **API 返回摘要**：API 返回的是推理摘要，而非完整思考过程

### 2. 需要企业协议才能获取完整思考

要获得完整的 Extended Thinking 输出，需要：
- 与 Anthropic 签署企业协议
- 付费获取完整思考日志
- 普通用户和免费用户无法访问

### 3. Ctrl+O 输出也非原始思考

文档明确说明：通过 Ctrl+O 调用的"Extended Thinking"输出是 Fable/Opus 模型思考的摘要，并非实际驱动模型决策的完整思考。这类似于将 BMP 保存为 JPEG 再编辑，存在信息损失。

## 实际影响

### 对用户的误导

1. **可审计性承诺需谨慎**：如果向任何人承诺可审计的推理日志，需要明确这些日志在本地不可读
2. **本地日志非推理记录**：即使用户能抓取输入、输出和动作，也无法获得实际推理过程
3. **文档表述不够直接**：文档说"extended thinking returns a summary of Claude's full thinking process"，需要仔细阅读才能理解这不是完整思考

### 对开发和调试的影响

对于依赖推理过程进行调试的开发者来说：
- 无法通过本地日志重现模型决策过程
- 需要额外抓取手段，但依然无法获得实际推理
- 仅 API 返回的摘要不足以进行深入调试

## 引用与验证

- **官方文档**：<https://platform.claude.com/docs/en/build-with-claude/extended-thinking>
- **Matt Green 的分析**：Patrick McCanna 引用了密码学专家 Matt Green 对签名块的观察，提供了更详细的技术视角
- **作者实测**：Patrick McCanna 通过实际检查本地日志和文档阅读，验证了加密机制的存在

## 边界与不确定性

### 未验证的部分

1. **企业协议的具体条款**：文章未提供企业协议的具体内容和价格
2. **摘要的质量**：未评估摘要对完整思考的覆盖率
3. **其他模型的行为**：未比较其他 AI 编码助手的类似机制

### 已知限制

- 文章基于个人经验和文档阅读，未获得 Anthropic 官方确认
- 未提供加密算法的详细技术分析
- 未测试企业协议用户的实际体验

## 结论与建议

### 对用户

如果需要可靠的推理记录：
- 不要依赖本地日志作为审计依据
- 在承诺可审计性前，明确获取完整思考的流程和成本
- 考虑使用其他支持透明推理记录的工具

### 对开发者

如果依赖推理过程进行调试：
- 明确当前 Claude Code 的 Extended Thinking 功能不支持完整的推理记录
- 设计工作流时考虑这一限制
- 探索其他方案，如捕获 API 交互和中间状态

### 对 AI 工具设计者

Anthropic 的这种设计反映了 AI 透明度和商业利益之间的权衡：
- **保护推理过程**：防止模型推理过程被轻易复制或逆向工程
- **差异化服务**：将完整思考记录作为企业级功能
- **用户教育**：需要更清晰地传达功能边界，避免误解

## 后续关注

- 企业协议的具体条款和价格
- 其他 AI 编码助手是否采用类似机制
- 社区对此类加密机制的反馈和替代方案

---

**来源**：Hacker News #48630535，原文：<https://patrickmccanna.net/the-text-in-claude-codes-extended-thinking-output-is-not-authentic/>  
**作者**：Patrick McCanna  
**发布时间**：2026-06-22  
**HN Score**：264 | **Comments**：185