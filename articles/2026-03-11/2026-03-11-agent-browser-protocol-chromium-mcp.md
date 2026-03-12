# Agent Browser Protocol：把浏览器变成 AI Agent 的"步进机器"

> 当 AI Agent 遇上现代网页，最大的障碍不是智能，而是时序。ABP 重新设计了浏览器的执行模型，让 Agent 不再需要和浏览器赛跑。

## 摘要

Agent Browser Protocol (ABP) 是一个将 MCP + REST API 直接嵌入 Chromium 引擎的开源浏览器构建。它的核心理念是：**网页浏览是连续异步的，而 LLM Agent 是离散思考的**——ABP 通过在引擎层面暂停 JavaScript 和虚拟时间，将浏览体验重塑为一个"步进机器"，让每个 HTTP 请求对应一个确定性的页面状态。

目前，ABP 在 Online Mind2Web 基准测试中达到了 **90.53%** 的准确率。

---

## 问题：Agent 和浏览器的时序错配

如果你用过 Playwright、Puppeteer 或 Selenium 来驱动浏览器自动化，你一定遇到过这些问题：

1. **竞态条件**：Agent 发出指令后，页面还在加载，截图时 DOM 还没渲染完
2. **等待策略**：要么写死 `sleep(2000)`，要么用复杂的 `waitForSelector` 启发式判断
3. **事件丢失**：弹窗、文件选择器、下载提示 —— 这些异步事件在 Agent 查询时可能已经消失
4. **光标不可见**：Agent 看不到鼠标位置，无法像人类一样"看到"交互反馈

这些问题的根源是一个结构性矛盾：

| 浏览器的特性 | Agent 的需求 |
|-------------|-------------|
| 连续执行、异步渲染 | 离散步骤、同步决策 |
| 实时流动 | 稳定快照 |
| 事件驱动、需要订阅 | 请求-响应、期望确定性 |

现有的解决方案都是在应用层打补丁：加等待、加重试、加轮询。ABP 的思路不同——**直接在浏览器引擎层面解决问题**。

---

## ABP 的解决方案：步进式浏览

ABP 的核心设计是将浏览行为转换为一个"步进机器"（step machine）：

```
Agent 发起请求 → 注入真实输入 → 等待页面稳定 → 截图 + 收集事件 → 暂停 JS 和虚拟时间 → 返回响应
```

### 关键特性

1. **引擎级 JS 暂停**
   - 每个 API 调用完成后，JavaScript 执行被冻结
   - `Date.now()` 停止更新，定时器暂停
   - Agent 决策时，页面处于确定性状态

2. **原生输入注入**
   - 通过 Chromium 的 `RenderWidgetHost` 注入真实的输入事件
   - 走完整的输入流水线，包括合成器层面的命中测试
   - 不是 CDP 的 `Input.dispatch*` 模拟事件

3. **自动截图 + 事件收集**
   - 每个动作完成后自动返回截图（含光标位置）
   - 事件流包含导航、弹窗、文件选择器、下载等
   - 无需额外调用 `takeScreenshot` 或轮询事件

4. **虚拟光标**
   - 合成器层面的光标渲染
   - 光标会随输入动作移动并出现在截图中
   - Agent 看到的和人类看到的一样

5. **REST API，而非 WebSocket**
   - 简单的 HTTP 请求-响应模型
   - 无需管理 CDP 会话或 WebSocket 连接
   - 单次请求延迟约 100ms（包括截图）

---

## 架构：HTTP 服务器嵌入浏览器进程

ABP 将 HTTP 服务器直接嵌入 Chromium 的浏览器进程：

```
+---------------------------------------------------------+
| AI Agent (curl / Python / Go)                           |
+----------------------------+----------------------------+
                             | REST API
                             v
+---------------------------------------------------------+
| AbpHttpServer (IO thread)                                |
| localhost:8222/api/v1/*                                 |
+----------------------------+----------------------------+
                             | PostTask
                             v
+---------------------------------------------------------+
| AbpController (UI thread)                                |
| Direct access to Browser, TabStripModel, DevTools       |
+----------------------------+----------------------------+
                             |
              +--------------+--------------+
              v              v              v
         +--------+    +----------+    +--------+
         | Input  |    | Renderer |    | Network|
         | System |    | (Blink)  |    | Stack  |
         +--------+    +----------+    +--------+
```

这种设计意味着：
- 请求在 IO 线程接收，在 UI 线程处理
- 直接访问 `Browser`、`TabStripModel`、DevTools Agent
- 没有进程间通信的开销

---

## API 示例：一个动作的完整响应

```json
{
  "result": {"status": "clicked"},
  "screenshot_before": {
    "data": "base64-webp...",
    "width": 1920,
    "height": 1080
  },
  "screenshot_after": {
    "data": "base64-webp...",
    "width": 1920,
    "height": 1080
  },
  "scroll": {"scrollX": 0, "scrollY": 150, "pageWidth": 1280, "pageHeight": 4000},
  "events": [
    {"type": "navigation", "data": {"url": "https://..."}},
    {"type": "dialog", "data": {"dialog_type": "confirm", "message": "Delete?"}},
    {"type": "file_chooser", "data": {"accepts": [".pdf", ".docx"]}}
  ],
  "cursor": {"x": 450, "y": 320, "cursor_type": "pointer"}
}
```

Agent 在一次调用中获得了：
- 动作结果
- 动作前后的截图
- 滚动位置
- 期间发生的所有事件
- 光标状态

---

## 与现有工具的对比

| 特性 | ABP | CDP/Puppeteer | Playwright | Selenium |
|------|-----|---------------|------------|----------|
| REST API | ✅ | ❌ WebSocket | ❌ RPC | ✅ |
| JS 执行暂停 | 引擎级 | Debugger | ❌ | ❌ |
| 虚拟时间 | ✅ | 部分 | 部分 | ❌ |
| 虚拟光标 | 合成器层 | ❌ | ❌ | ❌ |
| 自动截图 | ✅ | 手动 | 手动 | 手动 |
| 事件检测 | 内置 | 需订阅 | 手动 | 手动 |
| 会话录制 | SQLite | DevTools | Codegen | IDE |

---

## 训练数据生成：为 VLM 微调做准备

ABP 的另一个亮点是内置的会话录制功能。每个动作都被记录到 SQLite 数据库：

```
Action #1: navigate("https://example.com")
├── screenshot_before.webp
├── params: {"url": "https://example.com"}
└── screenshot_after.webp

Action #2: click(450, 320)
├── screenshot_before.webp
├── params: {"x": 450, "y": 320}
└── screenshot_after.webp
```

成功的 Agent 会话可以直接转化为视觉-语言模型（VLM）的微调数据集。

---

## 使用方式

### 作为 MCP 服务器（Claude Code / Codex）

```bash
claude mcp add browser -- npx -y agent-browser-protocol --mcp
```

然后直接问 Claude："在 Doordash 上找 415 Mission St, San Francisco 附近的宫保鸡丁"。

### 作为 REST API

```bash
# 启动
npx -y agent-browser-protocol

# 列出标签页
curl -s http://localhost:8222/api/v1/tabs

# 导航
curl -X POST http://localhost:8222/api/v1/tabs/<TAB_ID>/navigate \
  -H 'content-type: application/json' \
  -d '{"url":"https://example.com","screenshot":{"format":"webp"}}'
```

---

## 为什么这很重要

ABP 代表了 AI Agent 工具链的一个新方向：**不是在应用层模拟人类操作，而是在系统层重新设计执行模型**。

当前大多数 Agent 浏览器工具都是 CDP/WebDriver 的封装，它们继承了浏览器的异步特性，然后用各种技巧来"驯服"它。ABP 的做法更彻底——直接修改浏览器的行为，让浏览器适应 Agent，而不是让 Agent 适应浏览器。

这种思路的潜力在于：

1. **可靠性提升**：没有竞态条件意味着更少的重试和更稳定的自动化流程
2. **成本降低**：减少无效的 LLM 调用（不需要反复检查"页面加载完了吗"）
3. **训练友好**：自动生成结构化的训练数据
4. **多模态对齐**：截图、事件、光标状态在同一个响应中，Agent 获得完整感知

---

## 限制与展望

ABP 目前仍在活跃开发中。一些已知限制：
- 需要使用其定制的 Chromium 构建，不能直接用于 Chrome/Edge
- 主要针对本地运行场景
- 文档和生态还在建设中

但从技术方向来看，这是一个值得关注的尝试。当 AI Agent 越来越多地需要与网页交互时，"浏览器应该如何为 Agent 设计"这个问题会变得越来越重要。ABP 给出了一个相当有说服力的答案。

---

## 参考链接

- GitHub: [theredsix/agent-browser-protocol](https://github.com/theredsix/agent-browser-protocol)
- HN 讨论: [Show HN: Open-source browser for AI agents](https://news.ycombinator.com/item?id=47337607)
- Mind2Web 基准结果: [abp-online-mind2web-results](https://github.com/theredsix/abp-online-mind2web-results)

---

*撰写于 2026-03-11 | 来源：Hacker News Front Page (Score: 98)*
