# Browser Coding Agent

一个完全跑在浏览器里的、Claude Code 风格的 Coding Agent。单文件（`index.html`），无后端，无构建步骤。

在线体验：<https://roam.xiaopingfeng.com/browser-coding-agent/>

## 它是什么

- 终端式对话流：`>` 输入、`●` 旁白、`● Tool(args)` / `⎿` 结果、内联 diff。
- Agent 用 LS / Glob / Grep / Read / Write / Edit / MultiEdit 等工具读写工作区；文件、预览、变更在右侧抽屉里。
- 先 Read 才能 Edit；改动前内联确认；Shift+Tab 在 默认 / 自动接受 / plan 之间切换。
- 带页面验证工具（Preview / Click / Press / Type / Inspect / Eval / Console）、代码工具（Run / Test / Check）、WebFetch、TodoWrite、AskUser 和只读子 Agent（Task）。
- 支持 ES 模块（import/export 在 Run、Test、Preview 里共用同一套解析）。
- 斜杠命令、CLAUDE.md、`#` 记忆、`/rewind`、Esc 中断。
- Agent 的改动只进内存，`/save` 才写盘。
- 可选的 `Bash` 工具：用 [WebContainers](https://webcontainers.io/) 在页面里起一个真 shell，能跑 node、npm、`node --test`。它要求页面跨源隔离（`Cross-Origin-Opener-Policy: same-origin` 加 `Cross-Origin-Embedder-Policy: credentialless` 或 `require-corp`），没有这两个响应头时 `Bash` 不会出现，其余功能照常。WebContainers 的运行时由 StackBlitz 托管，用于商业生产环境需要他们的商业许可，原型和 PoC 不需要。

## 模型

在页面里配置，三选一：

| 来源 | 说明 |
|---|---|
| 本地 WebLLM | 浏览器内推理（WebGPU），首次需要下载模型；上下文很短，只给核心工具 |
| Ollama | 默认 `http://localhost:11434/v1`，需要 Ollama 允许该页面的来源 |
| 云端 API | 任何 OpenAI 兼容的 `/chat/completions`（例如智谱 BigModel）。API Key 只存在你浏览器的 localStorage 里 |

动作协议是文本 JSON + 代码块，所以三种模型走同一个循环。

## 本地运行

```bash
python3 -m http.server 8123
# 打开 http://localhost:8123/
```

读写本地文件夹用到 File System Access API（`showDirectoryPicker`），需要 Chromium 系浏览器；其他浏览器可以用页面里的内存工作区。

`python3 -m http.server` 不会发跨源隔离的响应头，所以本地这样跑没有 `Bash`。要试 `Bash`，用能加这两个响应头的服务器，或直接用在线版。

## 已知限制

- 小模型（7B 级）容易伪造观察结果、没验证就说完成；harness 里加了一些兜底（语法检查、重复报错守卫、交互后页面无变化则不许结束），但不能替代更强的模型。
- 本地 WebLLM 的真实推理没有做过完整验证。
- 没有做 MCP、hooks、自动补全和原生 tool-calling。

## 来源

这是 [问野 / Roam](https://roam.xiaopingfeng.com/) 里的一个条目（RM·0009），源码同时在 <https://github.com/fxp/roam>。作者：冯小平（<https://xiaopingfeng.com>）。

同一个 Agent 外壳还有一个接 Wasmer SDK 的版本：<https://roam.xiaopingfeng.com/browser-coding-agent-wasmer/>（RM·0010）。两个运行时的对比，和“什么任务能搬进浏览器标签页”的实测，见 <https://xiaopingfeng.com/deepdive/agent-in-the-browser/>。
