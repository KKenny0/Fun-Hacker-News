# Claude Code Extended Thinking：透明的幻觉

2026-06-23

## 核心发现

Patrick McCanna 在博客中揭露了一个重要事实：Claude Code 的 Extended Thinking 输出中的 "thinking blocks" 并非真实的思考文本，而是经过加密处理的签名。这意味着开发者无法通过本地日志审查 AI Agent 的实际推理过程。

## 关键问题

1. **加密存储**
   - Claude Code 将每次会话记录到磁盘，包括 "thinking blocks" —— 模型的推理过程
   - 但这些推理内容被加密成一个 600 字符的签名
   - Anthropic 持有解密密钥，用户的机器无法获得

2. **摘要替代**
   - API 返回的是推理过程的摘要，而非完整的推理文本
   - 要获取完整的思考输出需要企业协议
   - Ctrl+O 的 "extended-thinking" 输出是 Fable/Opus 思考的摘要，并非实际驱动模型行为的思考过程

3. **不可审计性**
   - 本地存储的推理日志无法访问
   - 即使通过技术手段抓取输入、输出和操作，也无法获得实际的推理过程
   - 对于需要审计 Agent 行为的场景（如安全审查、合规要求），这是一个重大缺陷

## 文档问题

McCanna 指出，Anthropic 的文档描述过于间接：
- 文档中提到 "extended thinking returns a summary of Claude's full thinking process"
- 但如果不仔细阅读，很容易错过这一关键信息
- 这种表述方式可能导致开发者误解为可以获得完整的思考过程

## 真实损失

这就像是把 JPEG 保存为 BMP，然后编辑 BMP 文件并声称它仍然是 JPEG —— 转换过程造成了数据损失。

对于 AI Agent 来说，推理过程的透明性至关重要。如果无法审计 Agent 是如何做出决策的，那么在很多场景下（如金融、医疗、安全）就无法满足合规要求。

## 启示

这一发现提醒我们：
1. 对于 AI 工具的"透明性"声称需要保持警惕
2. 不要想当然地认为本地存储的数据就是可读的
3. 在选择 AI 开发工具时，需要仔细了解数据访问权限和限制
4. 对于需要审计能力的场景，可能需要寻找替代方案或自定义实现

## 对 Agent 开发的影响

对于依赖 Claude Code 进行 Agent 开发的团队，这意味着：
- 无法通过本地日志进行问题诊断
- 无法审计 Agent 的决策过程
- 在合规要求严格的领域可能面临审查困难
- 需要额外的工具来捕获和记录推理过程

开源模型的进步速度需要加快，以提供真正透明的推理能力。

---

**来源：**
- https://patrickmccanna.net/the-text-in-claude-codes-extended-thinking-output-is-not-authentic/
- Hacker News Rank #2, Score: 264