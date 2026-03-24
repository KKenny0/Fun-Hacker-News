# cq：给 AI Agent 们建一个 Stack Overflow

> Mozilla AI 推出的开源项目，让 AI 编程代理们可以共享知识、避免重复踩坑

## 摘要

当 ChatGPT 和 Claude 们"吃掉"了 Stack Overflow 的知识库后，人类开发者不再去那里提问了。但问题来了：AI agents 自己也会遇到同样的坑，而且它们在各自的代码库里反复踩同样的坑，浪费大量的 tokens 和算力。Mozilla AI 的解决方案是 **cq** —— 一个给 AI agents 用的"Stack Overflow"，让代理们可以查询过去的经验、贡献新知识、形成一个共享的知识公地。

---

## 背景：Stack Overflow 的兴衰

2008 年，Stack Overflow 诞生。到 2014 年，它每月有超过 20 万个问题，成为程序员们不可或缺的知识库。但转折点出现在 2022 年底 ChatGPT 发布后——2025 年 12 月，Stack Overflow 的问题数暴跌至 **3,862 个**，回到了它刚上线时的水平。

讽刺的是：LLM 训练时"吃掉"了 Stack Overflow 的知识库，然后又反过来"掏空"了这个社区。作者用了一个很有意思的词：**matriphagy**（母食），蜘蛛幼崽吃掉母亲来获得营养——Stack Overflow 的知识库养育了 LLM，然后 LLM 又把 Stack Overflow 给掏空了。

但更讽刺的是：现在 **AI agents 也需要自己的 Stack Overflow 了**。

---

## 问题：AI Agents 在"孤岛"里重复踩坑

如果你用过 Claude Code、Cursor 这些 AI 编程工具，可能会发现一个现象：同样的错误，agent 会反复犯。

比如：
- 第一次集成 Stripe API 时，不知道 Stripe 对 rate-limited 请求会返回 200 状态码但 body 里是错误信息
- 配置 CI/CD 时，不知道某个框架的特定坑
- 读了一堆文档、写了一堆代码、触发 CI 失败、诊断问题、然后重来

**每个 agent 都在各自的代码库里独立地撞墙，每次都要消耗 tokens 和算力。**

这就是 cq 要解决的问题。

---

## 解决方案：cq —— Agents 的知识公地

cq 的名字来自 **colloquy**（对话），在无线电通信里 CQ 是"呼叫所有台站"的意思。它的核心理念是：

> **让 agents 在开始工作前，先查询一个共享的知识库，看看其他 agents 有没有遇到过类似的问题。**

### 工作流程

1. **查询阶段**：agent 在处理不熟悉的任务前（比如 API 集成、CI/CD 配置、新框架），先查询 cq commons
2. **获取经验**：如果其他 agent 已经学到了某些知识（比如 Stripe 返回 200 但 body 是错误），你的 agent 就知道了，不用再踩坑
3. **贡献阶段**：当你的 agent 发现了新的知识，可以提交回去
4. **信任机制**：其他 agents 可以确认哪些知识有效，哪些已经过时——知识通过使用获得信任，而不是靠权威

### 核心价值

- **节省 tokens 和算力**：不用每个 agent 都从头摸索
- **跨团队的知识共享**：不同团队、不同代码库的 agents 可以共享经验
- **信任机制**：经过多个 agents 在多个代码库验证的知识，比单个模型的猜测更可信

---

## 信任问题：为什么这很重要？

Stack Overflow 2025 年的调查显示：
- **84% 的开发者**正在使用或计划使用 AI 工具
- 但 **46% 的人不信任 AI 输出的准确性**，比前一年的 31% 还在上升

也就是说：**工程师们在用 AI，但心里没底。**

cq 的信任机制可以缓解这个问题：经过多个 agents 验证的知识，比单个模型的"最佳猜测"更可信。这类似于 Stack Overflow 上高票回答比随便一个回答更可信的逻辑。

---

## 当前进展：开源 PoC 已可用

cq 目前处于早期阶段，但已经有了一个可用的 PoC：

- **插件**：支持 Claude Code 和 OpenCode
- **MCP 服务器**：管理本地知识库
- **团队 API**：支持组织内共享
- **UI 界面**：支持"人工审核"环节
- **容器化部署**：可以快速启动整套系统

Mozilla AI 正在自己内部 dogfooding（吃自己的狗粮），在实际项目中积累知识单元，找出真正的痛点。

项目是 **完全开源** 的，代码在 [GitHub](https://github.com/mozilla-ai/cq) 上，还有一个详细的设计提案。

---

## 为什么这值得关注？

### 1. 解决了真实痛点

AI agents 浪费 tokens 和算力在重复错误上，这是每个用 AI 编程的人都能感受到的问题。cq 提供了一个系统性的解决方案。

### 2. 开放 vs 封闭

如果每个 AI 公司都搞自己的"agent 记忆系统"，那就会形成一个个封闭的孤岛。Mozilla AI 的做法是建立一个开放的标准和公地，这符合开源和开放生态的理念。

### 3. 历史的回响

作者在文章开头提到"历史总是重复"——技术在循环。现在 AI agents 需要自己的 Stack Overflow，这是技术演进的一个有趣节点。

### 4. 社区驱动

这不是大公司关起门来搞的东西，而是开源的、寻求社区反馈的项目。这种模式更有可能形成真正的标准。

---

## 我的看法

cq 解决的是一个真实存在的问题，而且时机很对——Andrew Ng 最近也在问"是否应该有一个 AI agents 的 Stack Overflow"。Mozilla AI 的答案是：**是的，我们正在建，来加入我们。**

这个项目的成功与否，取决于能否形成足够大的网络效应：参与的 agents 越多，知识库越有价值；知识库越有价值，越多的 agents 会参与。这和 Stack Overflow 的增长逻辑一样。

如果你在用 AI 编程工具，或者自己在开发 agents，值得去看看 [cq 的 GitHub 仓库](https://github.com/mozilla-ai/cq) 和 [设计提案](https://github.com/mozilla-ai/cq/blob/main/docs/CQ-Proposal.md)，看看这个方向是否符合你的需求，或者你能贡献什么。

---

## 来源

- [cq: Stack Overflow for Agents - Mozilla AI Blog](https://blog.mozilla.ai/cq-stack-overflow-for-agents/)
- [cq GitHub 仓库](https://github.com/mozilla-ai/cq)
- [Stack Overflow 2025 Developer Survey](https://survey.stackoverflow.co/2025)
- [Andrew Ng 关于 Agent Stack Overflow 的讨论](https://www.deeplearning.ai/the-batch/issue-344/)

---

*本文基于 Hacker News 热门帖子 [Show HN: Cq – Stack Overflow for AI coding agents](https://blog.mozilla.ai/cq-stack-overflow-for-agents/) (178 分) 撰写*
