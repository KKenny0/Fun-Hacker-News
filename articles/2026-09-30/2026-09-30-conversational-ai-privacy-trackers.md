# 你的 AI 对话，正在被广告追踪器围观——一篇 406 分 HN 论文说了什么

> 日期：2026-09-30 | 来源：Hacker News（406 分，128 评论）
> 论文：*Prompt like a Butterfly, Sting like a Tracker: A Privacy Analysis of Web and Mobile Conversational AI Agents*（Jorge García Herrero 等，2026-09-16）
> 论文原文：https://jorgegarciaherrero.com/wp-content/interactivos/20260916-Prompt-like-a-butterfly-sting-like-a-tracker-(clean).pdf
> HN 讨论：https://news.ycombinator.com/item?id=49890226

## 一句话结论

主流对话式 AI 服务不只是"可能收集数据"——它们把对话标题、对话链接、甚至用户提示词和聊天截图，连同持久性用户标识符，一起发给了嵌入在自家客户端里的第三方广告追踪网络。而且你点"拒绝非必要 Cookie"基本没用。

## 研究做了什么

研究团队对 9 个主流对话式 AI 服务做了系统性隐私审计：ChatGPT、Claude、Grok、DeepSeek、Perplexity、Gemini、MS Copilot、Mistral、Meta AI。覆盖全部 9 个服务的 Web 端和其中 8 个的 Android 端。

方法上三层结合：Web 端用 Chrome DevTools 协议抓取完整网络流量；Android 端在定制 AOSP 系统上做 mitmproxy 动态分析加 Androguard 静态分析；交互设计采用"角色扮演审计"——模拟用户输入含敏感医疗信息的提示词，观察系统的数据处理行为。同时测试了三种 Cookie 同意状态和访客/免费/付费三种订阅层级。

这个设计的价值在于：它看的不是隐私政策怎么写，而是数据实际流向哪里。

## 核心发现：一张泄露图谱

先给总体数字：**9 个服务共集成 124 个第三方域名，归属于 44 个组织，其中 34 个是广告与追踪服务（ATS）。每一个被测服务都集成了至少一个 ATS。**

具体到厂商，格局大致是：

| 服务 | 泄露内容 | 接收方 |
|------|---------|--------|
| Grok (xAI) | 对话 URL、标题、提示词、**聊天截图** | 同时发给 7 个 ATS：Google Ads、DoubleClick、Meta Pixel、TikTok、Twitter Analytics 等 |
| Claude | 对话 ID、URL、用户邮箱及哈希 | Datadog、Intercom |
| ChatGPT | 全局唯一对话 ID | Datadog |
| Perplexity | 用户提示词、邮箱哈希、广告 ID | Wingify、Singular |
| Gemini | 对话标题 | Google Analytics |
| Mistral | 标题、用户 ID、邮箱 | Intercom |

Grok 是全场最严重的：它是唯一泄露截图的服务，且免费和付费用户的对话永久链接默认公开可访问。论文原话："Grok takes the most permissive stance... free and premium tier conversation permalinks are accessible by default."

## 两个值得单独说的发现

**第一，同意机制形同虚设。** 即使用户明确拒绝非必要 Cookie，仍有 44.4%（4/9）的服务继续允许第三方收集数据。部分服务还采用服务端追踪（sGTM），数据从服务器对服务器转发，浏览器端的拦截插件根本看不到。

**第二，"分享链接"是放大器。** 团队做了金丝雀令牌实验：在 Grok 上分享一个对话后，该链接被来自 14 个国家、48 个自治系统的 IP 访问了 70 次——其中 65.7% 的访问来自美国，尽管对话发生在欧盟。一个"分享给朋友"的动作，实际效果是对整个追踪生态开放了你的完整对话。

## 为什么这比传统网页追踪更严重

论文提出了一个关键概念：**对话衍生制品（Conversation-Derived Artifacts）**。

传统追踪看的是你的点击流——你浏览了什么页面。而对话式 AI 泄露的是高度敏感的自由文本意图：医疗焦虑、财务困境、内心独白。更糟的是，AI 自动生成的对话标题本身就是内容摘要——你还没点开分享，标题已经把你的问题告诉第三方了。

叠加身份标识（邮箱哈希、账户 ID），跨平台画像的精度远超 Cookie 时代。用论文的话说：对话式 AI 引入了一个新的隐私攻击面——"provider-generated conversational artifacts become subject to tracking"。

## HN 社区在吵什么

讨论区的核心分歧是"事故还是故意"。有人认为这是 AI 公司运营能力不足的意外产物；也有人指出 OpenAI 大量高管来自 Meta，他们太清楚把用户数据漏给广告网络（尤其是竞对的广告网络）的代价，更符合商业逻辑的是把数据留在自己手里自建广告业务。这个争论没有定论，但指向同一个事实：无论动机如何，泄露正在发生。

## 边界与局限

三点必须交代：测试时间窗口为 2026 年 5 月前后、地点在欧盟，服务条款和技术实践变化很快；只能观测网络传输层，看不到服务端内部数据处理（比如是否进入训练集）；未覆盖企业版、桌面客户端和 API 调用场景。个别 SDK 兼具归因、支付等功能，观察到连接不等于在所有条件下都用于广告目的。

## 给普通用户的可执行建议

1. 不用"分享对话"功能，尤其不要用 Grok 和 Perplexity 的分享链接
2. 拒绝非必要 Cookie——作用有限，但仍是零成本动作
3. 高敏感话题（医疗、法律、财务）考虑本地模型或至少换用隐私姿态更好的服务
4. 记住一个判断标准：你对 AI 说的每一句话，都应该按"可能会被我不知道的第三方看到"来预期

---

*分析基于论文全文与 HN 讨论区（2026-09-30 抓取）。泄露行为以论文测量时点为准，各厂商可能已调整。*
