# Open Design：Claude Design 的开源替代品，让设计代理成为你本地的技能引擎

2026年4月17日，Anthropic 发布了 Claude Design，展示了当 LLM 停止写作散文、开始输出设计作品时会发生什么。这个产品迅速走红，但同时也暴露了一个问题：它是闭源的、付费的、仅限云端的，并且锁定在 Anthropic 的模型和技能生态系统中。没有版本控制，没有自托管选项，没有 Vercel 部署，更无法替换为你自己的代理。

现在，开源社区给出了自己的答案 —— **Open Design**。

## 不提供代理，而是让代理更强大

Open Design 的核心理念很简单：**我们不提供代理，你的代理已经足够好了**。这个项目扫描你的 PATH，自动检测已安装的 12 种编码代理 CLI，包括 Claude Code、Codex CLI、Devin for Terminal、Cursor Agent、Gemini CLI、OpenCode、Qwen Code、GitHub Copilot CLI、Hermes、Kimi CLI、Pi 和 Kiro CLI。找到的任何一个都可以成为设计引擎，并且可以在模型选择器中一键切换。

如果没有安装 CLI？Open Design 提供了 OpenAI 兼容的 BYOK（Bring Your Own Key）代理 —— 粘贴任何 OpenAI 兼容的 `baseUrl` + `apiKey` + `model`，Anthropic（通过 OpenAI）、DeepSeek、Groq、MiMo、OpenRouter，或者你自己托管的 vLLM 都能成为引擎。守护进程在边缘阻止了内部 IP/SSRF 攻击。

## 技能是文件，不是插件

Open Design 遵循 Claude Code 的 `SKILL.md` 约定，每个技能就是一个文件夹 —— `SKILL.md` + `assets/` + `references/`。将文件夹放入 `skills/` 目录，重启守护进程，它就会出现在选择器中。

项目内置了 **31 个技能**，分为两种模式：**prototype**（27 个技能）和 **deck**（4 个技能）。Prototype 模式生成单页作品，从杂志落地页到手机屏幕再到 PM 规格文档；Deck 模式则是水平滑动的演示文稿。

技能按场景分组：设计、营销、运营、工程、产品、财务、HR、销售、个人。每个技能都有示例 HTML 文件，无需认证、无需设置，直接打开就能看到代理会输出什么。

## 设计系统是可移植的 Markdown

Open Design 内置了 **129 个设计系统**，包括 2 个手写的启动器 + 70 个产品系统（Linear、Stripe、Vercel、Airbnb、Tesla、Notion、Anthropic、Apple、Cursor、Supabase、Figma、小红书等）+ 57 个来自 awesome-design-skills 的设计技能。

设计系统遵循 `awesome-design-md` 的 9 节 `DESIGN.md` 模式 —— 颜色、排版、间距、布局、组件、动效、声音、品牌、反模式。每个作品都从激活的系统读取。切换系统，下一次渲染就使用新的令牌。

## 交互式问题表单：防止 80% 的重定向

Open Design 的提示栈硬编码了一个 `RULE 1`：每个新的设计简报都以 `<question-form id="discovery">` 开始，而不是代码。表面、受众、语调、品牌背景、规模、约束条件。一个长简报仍然留有设计决策开放 —— 视觉语调、颜色立场、规模 —— 这些正是表单在 30 秒内锁定的东西。

这是从 `huashu-design` 中提炼的 **Junior-Designer 模式**：提前批量提问，尽早展示可见内容（即使是带有灰色块的线框图），让用户廉价地重定向。结合品牌资产协议（定位、下载、`grep` 十六进制、写入 `brand-spec.md`、口语化），这是输出停止感觉像 AI 自由发挥、开始感觉像设计师在绘画前先关注的最大单一原因。

## 守护进程：让代理感觉像在你的笔记本电脑上，因为它确实在

守护进程使用 `cwd` 设置为项目在 `.od/projects/<id>/` 下的作品文件夹来生成 CLI。代理获得 `Read`、`Write`、`Bash`、`WebFetch` —— 针对真实文件系统的真实工具。它可以 `Read` 技能的 `assets/template.html`，`grep` 你的 CSS 中的十六进制值，写入 `brand-spec.md`，放置生成的图片，并在回合结束时生成 `.pptx` / `.zip` / `.pdf` 文件，这些文件作为下载芯片显示在文件工作区中。

会话、对话、消息、选项卡持久保存在本地 SQLite 数据库中 —— 明天打开项目，代理的待办事项卡就在你离开的地方。

## 技术架构

Open Design 的架构分为三层：

**前端**：Next.js 16 App Router + React 18 + TypeScript，可部署到 Vercel

**守护进程**：Node 24 · Express · SSE 流式传输 · `better-sqlite3`

**代理传输**：`child_process.spawn`，为每种 CLI 提供类型化事件解析器

存储使用 `.od/projects/<id>/` 中的纯文件 + `.od/app.sqlite` 中的 SQLite 数据库。预览通过 `srcdoc` 的沙箱 iframe + 每技能 `<artifact>` 解析器实现。导出支持 HTML（内联资源）、PDF（浏览器打印，支持 deck）、PPTX（代理通过技能驱动）、ZIP（归档器）、Markdown。

生命周期由单一入口点控制：`pnpm tools-dev`（start / stop / run / status / logs / inspect / check）。

## 六个核心理念

1. **我们不提供代理。你的已经足够好了。** 守护进程在启动时扫描你的 PATH，找到的任何编码代理都成为候选设计引擎。

2. **技能是文件，不是插件。** 遵循 Claude Code 的 `SKILL.md` 约定，添加技能只需一个文件夹。

3. **设计系统是可移植的 Markdown，而不是主题 JSON。** 9 节 `DESIGN.md` 模式，每个作品都从激活的系统读取。

4. **交互式问题表单防止 80% 的重定向。** 提前批量提问，尽早展示可见内容，让用户廉价地重定向。

5. **守护进程让代理感觉像在你的笔记本电脑上，因为它确实在。** 代理获得针对真实文件系统的真实工具，会话持久保存在本地 SQLite 数据库中。

6. **提示栈是产品。** 每层都是可组合的，每层都是你可以编辑的文件。

## 媒体生成

Open Design 的媒体生成表面与设计循环一同提供。**gpt-image-2**（Azure / OpenAI）用于海报、头像、信息图、插图地图；**Seedance 2.0**（ByteDance）用于电影级 15 秒文本到视频和图像到视频；**HyperFrames**（heygen-com/hyperframes）用于 HTML→MP4 动态图形（产品揭示、动态排版、数据图表、社交叠加、Logo 片尾）。

**93** 个即用型提示模板库 —— 43 个 gpt-image-2 + 39 个 Seedance + 11 个 HyperFrames —— 位于 `prompt-templates/` 下，带有预览缩略图和源归属。与代码相同的聊天表面；输出真实的 `.mp4` / `.png` 芯片到项目工作区。

## 许可证

Open Design 采用 Apache-2.0 许可证，这意味着你可以自由使用、修改和分发。

## 总结

Open Design 不是另一个 AI 设计工具，而是一个让现有 AI 代理变得更好的框架。它通过技能驱动的设计工作流、可移植的设计系统、交互式问题表单和本地守护进程，让代理真正成为你的设计引擎，而不是一个远程的、黑盒的 API。

对于已经在使用 Claude Code、Codex、Cursor 等编码代理的开发者来说，Open Design 提供了一个自然的扩展 —— 你不需要学习新的代理，只需要让你的现有代理变得更擅长设计。

---

**来源**：[Open Design GitHub 仓库](https://github.com/nexu-io/open-design) | HN Score: 165 | Comments: 82