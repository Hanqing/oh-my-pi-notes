# 附录 A　源码导航与阅读路线

> 不要把 17.1.3 当一本从第一页顺读到最后一页的书。先问问题，再沿 composition root、owner、state machine 和边界函数定位。

## A.1 先用责任边界缩小目录

| 想找什么 | 第一站 |
| --- | --- |
| CLI、会话、工具、扩展、TUI mode | [`packages/coding-agent`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src) |
| provider-neutral Agent loop | [`packages/agent`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/agent/src) |
| 模型/provider/协议/鉴权 | [`packages/ai`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/ai/src) |
| 生成模型目录 | [`packages/catalog`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/catalog) |
| 终端渲染 primitives | [`packages/tui`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/tui/src) |
| 跨环境 wire types | [`packages/wire`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/wire/src) |
| Hashline | [`packages/hashline`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/hashline/src) |
| Snapcompact | [`packages/snapcompact`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/snapcompact/src) |
| 本地语义记忆 | [`packages/mnemopi`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/mnemopi/src) |
| 离线统计 | [`packages/stats`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/stats/src) |
| N-API JS loader | [`packages/natives/native`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/natives/native) |
| 原生实现 | [`crates`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/crates) |
| CI、生成、发布 | [`scripts`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/scripts) |

第一原则：先确认职责属于哪个包，再搜索 symbol。直接在整个仓库搜 `prompt` 会得到太多不同层的同名方法。

## A.2 最值得先读的十二个文件

按调用顺序：

1. [`packages/coding-agent/src/cli.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/cli.ts) — 轻入口与 lazy dispatch；
2. [`packages/coding-agent/src/main.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/main.ts) — root command 与 mode 分流；
3. [`packages/coding-agent/src/sdk.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/sdk.ts) — composition root；
4. [`packages/coding-agent/src/session/agent-session.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/agent-session.ts) — 产品编排；
5. [`packages/agent/src/agent.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/agent/src/agent.ts) — Agent state façade；
6. [`packages/agent/src/agent-loop.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/agent/src/agent-loop.ts) — 真实循环；
7. [`packages/ai/src/stream.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/ai/src/stream.ts) — provider dispatch；
8. [`packages/coding-agent/src/session/session-tools.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-tools.ts) — 工具注册与包装；
9. [`packages/coding-agent/src/session/session-manager.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-manager.ts) — JSONL/tree；
10. [`packages/coding-agent/src/session/session-maintenance.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-maintenance.ts) — 压缩/维护；
11. [`packages/coding-agent/src/modes/interactive-mode.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/modes/interactive-mode.ts) — TUI 产品层；
12. [`packages/natives/native/loader-state.js`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/natives/native/loader-state.js) — 原生部署边界。

不要把第 4、9、10 个巨型文件从头逐行读。先用下面的问题表定位 public method/owner，再向两侧展开。

## A.3 按问题找启动链

| 问题 | 搜索符号 | 入口 |
| --- | --- | --- |
| profile 为什么要提前解析 | `extractProfileFlags` | [`profile-bootstrap.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/cli/profile-bootstrap.ts) |
| `.env` 谁覆盖谁 | `homeEnv`、`projectEnv` | [`utils/env.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/utils/src/env.ts) |
| argv 怎样变 session options | `runRootCommand` | [`main.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/main.ts) |
| interactive/print/RPC/ACP 在哪分流 | `runInteractiveMode`、`runPrintMode` | `main.ts` 与 [`modes`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/modes) |
| worker 隐藏入口 | `__omp_worker_` | `cli.ts`、stats/coding-agent worker source |
| 启动阶段哪里慢 | `logger.time` | `main.ts`、`sdk.ts` |
| 独立二进制怎样编译 | `compileCodingAgent` | [`compile-binary.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/scripts/compile-binary.ts) |

## A.4 按问题找配置

| 问题 | 搜索符号 | 入口 |
| --- | --- | --- |
| 有效值优先级 | `#rebuildMerged` | [`config/settings.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/config/settings.ts) |
| 某 setting 默认值 | setting path 字符串 | [`settings-schema.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/config/settings-schema.ts) |
| project setting 来自哪里 | `settingsCapability` | [`capability/settings.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/capability/settings.ts) |
| `--config` 为什么 hard fail | `#loadOverlayYaml` | `settings.ts` |
| 旧字段怎样迁移 | `#migrateRawSettings` | `settings.ts` |
| provider setting 怎样生效 | `initializeWithSettings` | [`config.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/config.ts) |
| model role 怎样解析 | `resolveModelRoleValue` | [`config/model-roles.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/config/model-roles.ts) |

调试设置时先回答：schema default、global、project、overlay、runtime 中哪一层拥有这个 path。只打印 `settings.get()` 看不到来源。

## A.5 按问题找会话装配

在 [`createAgentSession()`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/sdk.ts) 中优先搜索这些锚点：

| 锚点 | 解释 |
| --- | --- |
| `discoverAuthStorage` | 鉴权与 ModelRegistry 所有权 |
| `workspaceTreePromise` | 启动扫描 deadline |
| `discoverContextFiles` | AGENTS/上下文发现 |
| `loadSecrets` | provider boundary 前置 |
| `getSessionBranch` | resume 与 interrupted tail |
| `resolveAllowedModels` | 可选模型与 role |
| `discoverTtsrRules` | rules/TTSR |
| `createToolSession` / tool creation | 工具依赖注入 |
| `new Agent` | 纯 Agent state 建立 |
| `new AgentSession` | 产品编排对象建立 |
| `AgentRegistry` | main/child identity |
| `catch`/teardown | 构造中途失败的资源回收 |

阅读时画出“谁创建、谁拥有、谁 dispose”。composition root 最大的价值不是调用很多 constructor，而是集中处理半构造失败。

## A.6 按问题找 Agent Loop

| 问题 | 符号 | 文件 |
| --- | --- | --- |
| 什么时候判 busy | `AgentBusyError`、`#state.isStreaming` | [`agent.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/agent/src/agent.ts) |
| prompt 怎样进 loop | `prompt` → `#runLoop` | `agent.ts` |
| loop 的 while 在哪 | `runLoopBody` | [`agent-loop.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/agent/src/agent-loop.ts) |
| stream 事件怎样发 | `streamAssistantResponse` | `agent-loop.ts` |
| tool calls 怎样执行 | `executeToolCalls` 附近 | `agent-loop.ts` |
| steering 何时 poll | `getSteeringMessages` | `agent-loop.ts` / `agent.ts` |
| follow-up 何时 drain | `getFollowUpMessages` | `agent-loop.ts` / `agent.ts` |
| abort 怎样传播 | `AbortController` | `agent.ts`、tool context、provider stream |
| telemetry span 在哪 | `startInvokeAgentSpan` 等 | [`telemetry.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/agent/src/telemetry.ts) |

想理解行为时先看 event/stop condition，再看具体 provider。很多“模型又调用一次”的现象来自 loop 合法推进，而不是 provider adapter 重试。

## A.7 按问题找模型与 provider

| 问题 | 入口 |
| --- | --- |
| 模型静态数据 | [`packages/catalog/src/models.json`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/catalog/src/models.json) |
| provider 注册与特性 | [`packages/ai/src/registry/registry.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/ai/src/registry/registry.ts) |
| stream dispatcher | [`packages/ai/src/stream.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/ai/src/stream.ts) |
| 统一类型 | [`packages/ai/src/types.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/ai/src/types.ts) |
| provider 实现 | [`packages/ai/src/providers`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/ai/src/providers) |
| coding-agent model resolution | [`config/model-resolver.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/config/model-resolver.ts) |
| model registry/auth | [`config/model-registry.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/config/model-registry.ts) |
| auth broker | [`packages/ai/src/auth-broker`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/ai/src/auth-broker) |
| provider boundary | [`session-provider-boundary.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-provider-boundary.ts) |

Provider 调试路线：`Model` 字段 → registry api kind → stream dispatcher → adapter payload transform → wire event parser → normalized `AssistantMessage`。

## A.8 按问题找消息转换

| 问题 | 文件/符号 |
| --- | --- |
| AgentMessage/custom message 结构 | [`session/messages.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/messages.ts) |
| 转 LLM message | `convertToLlm` in `messages.ts` |
| provider payload transform | [`providers/transform-messages.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/ai/src/providers/transform-messages.ts) |
| tool call/result 修复 | `repair`/pairing symbols in agent/ai message code |
| thinking signature/payload | provider adapter + session persistence sanitizer |
| secret 出站/入站 | `SessionProviderBoundary`、[`secrets/obfuscator.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/secrets/obfuscator.ts) |
| 图片 fallback | [`utils/image-loading.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/utils/image-loading.ts) |

排查跨 provider replay 时比较三份数据：session entry、统一 AgentMessage、最终 provider payload。只看其中一份无法判断字段在哪一层丢失。

## A.9 按问题找工具系统

| 问题 | 入口 |
| --- | --- |
| 所有内建工具注册 | [`tools/index.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/index.ts) |
| session 级包装/active set | [`session/session-tools.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-tools.ts) |
| ToolSession contract | [`tools/index.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/index.ts) 中的 `ToolSession` |
| approval 算法 | [`tools/approval.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/approval.ts) |
| 调度/资源冲突 | agent-core tool scheduler 相关 symbol |
| 输出 spill/artifact | [`tools/output-meta.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/output-meta.ts) |
| xdev | [`tools/resolve.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/resolve.ts) 与 internal URL `xd://` |
| plan guard | [`tools/plan-mode-guard.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/plan-mode-guard.ts) |
| ACP permission | [`session/acp-permission-gate.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/acp-permission-gate.ts) |

具体工具直接从 `tools/<name>.ts` 读，但必须同时检查注册时的 wrapper；只读 implementation 容易漏掉 approval、TTSR、extension hook 和 output spill。

## A.10 编码工具阅读路线

### Read/Edit/Write

1. [`tools/read.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/read.ts)；
2. [`packages/hashline`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/hashline/src)；
3. [`edit/index.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/edit/index.ts)；
4. [`tools/write.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/write.ts)；
5. path/internal URL/snapshot/mutation version helpers。

### Bash/Eval

1. [`tools/bash.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/bash.ts)；
2. [`session/eval-runner.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/eval-runner.ts)；
3. [`crates/pi-natives/src/shell.rs`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/crates/pi-natives/src/shell.rs)；
4. [`crates/pi-shell/src/shell.rs`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/crates/pi-shell/src/shell.rs)；
5. process/cancel/minimizer。

### LSP/DAP/AST

- [`lsp`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/lsp)；
- [`dap`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/dap)；
- [`tools/ast-edit.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/ast-edit.ts)；
- [`crates/pi-ast`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/crates/pi-ast/src)；
- [`crates/pi-natives/src/ast.rs`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/crates/pi-natives/src/ast.rs)。

## A.11 按问题找上下文与能力发现

| 问题 | 入口 |
| --- | --- |
| discovery 总框架 | [`discovery`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/discovery) |
| capability 定义 | [`capability`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/capability) |
| context files | [`sdk.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/sdk.ts) 中的 `discoverContextFiles()` |
| system prompt 拼接 | [`system-prompt.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/system-prompt.ts) |
| skills | [`extensibility/skills.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/extensibility/skills.ts) |
| slash commands/templates | `extensibility/slash-commands.ts`、`config/prompt-templates.ts` |
| TTSR | [`ttsr-coordinator.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/ttsr-coordinator.ts) 与 export manager |
| workspace tree | `buildWorkspaceTree` in SDK/context utilities |
| `@file` | `extractFileMentions` / `generateFileMentionMessages` |

发现问题要分三步：候选是否扫描到、priority/dedupe 是否留下、最终是否注入 prompt/tool registry。

## A.12 按问题找 Session 与树历史

| 问题 | 搜索符号 | 文件 |
| --- | --- | --- |
| 创建/打开 session | `SessionManager.create/open/resume` | [`session-manager.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-manager.ts) |
| JSONL 读取与 blob | `loadSession` | [`session-loader.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-loader.ts) |
| append serialization | `appendEntry` / write chain | `session-manager.ts` |
| 当前 branch | `getBranch` | `session-manager.ts` |
| 有效 model/thinking state | `buildSessionContext` | session context builder |
| title slot | [`session-title-slot.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-title-slot.ts) |
| persistence sanitize | [`session-persistence.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-persistence.ts) |
| atomic indexed storage | [`indexed-session-storage.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/indexed-session-storage.ts) |
| fork/resume listing | [`session-listing.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-listing.ts) |

永远不要假设 JSONL 最后一行就是 current leaf；先读 header/parentId/tree selection。

## A.13 按问题找压缩与恢复

| 问题 | 入口 |
| --- | --- |
| 阈值与 orchestrator | [`session-maintenance.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-maintenance.ts) |
| compaction 公共类型 | [`packages/agent/src/compaction`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/agent/src/compaction) |
| Snapcompact inline | [`snapcompact-inline.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/snapcompact-inline.ts) |
| recovery/retry | [`turn-recovery.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/turn-recovery.ts) |
| stream guards | [`stream-guards.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/stream-guards.ts) |
| provider remote compaction | provider-specific compaction module |

压缩 bug 先记录触发原因、cut boundary、保留 recent token、tool pairing、previous summary/preserveData，再看 summarizer 文案。

## A.14 按问题找扩展、插件与 MCP

| 问题 | 入口 |
| --- | --- |
| extension load/API | [`extensibility/extensions/loader.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/extensibility/extensions/loader.ts) |
| event dispatch | [`extensions/runner.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/extensibility/extensions/runner.ts) |
| managed timers | [`extensions/managed-timers.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/extensibility/extensions/managed-timers.ts) |
| plugin manifest/install/cache | [`extensibility/plugins`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/extensibility/plugins) |
| MCP config | [`mcp/config.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/mcp/config.ts) |
| MCP lifecycle | [`mcp`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/mcp) manager/client/transport |
| MCP resources/prompts | internal URL `mcp://` 与 bridge |

Extension bug 要查 import-time 与 handler-time 两类；MCP bug 要查 transport、initialize/capabilities、tool adaptation、reconnect ownership 四类。

## A.15 按问题找 TUI

| 层 | 入口 |
| --- | --- |
| 通用 terminal/component/render | [`packages/tui/src`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/tui/src) |
| coding-agent product UI | [`modes/interactive-mode.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/modes/interactive-mode.ts) |
| 输入 | [`modes/controllers/input-controller.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/modes/controllers/input-controller.ts) |
| Agent events | [`modes/controllers/event-controller.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/modes/controllers/event-controller.ts) |
| editor | [`modes/components/custom-editor.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/modes/components/custom-editor.ts) |
| tool cards | `modes/components` 与 generated views |
| native width/key/sixel | `pi-natives` text/keys/sixel exports |

视觉 bug 先判断是 component layout、TUI diff、terminal capability 还是 committed-prefix ownership。不要看到闪烁就直接改 ANSI clear sequence。

## A.16 按问题找多代理与协作

| 问题 | 入口 |
| --- | --- |
| Agent 索引 | [`registry/agent-registry.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/registry/agent-registry.ts) |
| 生命周期/park/revive | `registry`/lifecycle manager |
| task tool | [`task/index.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/task/index.ts) |
| yield | [`tools/yield.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/yield.ts) |
| Hub | [`tools/hub`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/tools/hub) |
| Advisor | [`session/session-advisors.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-advisors.ts) |
| isolation | [`crates/pi-iso`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/crates/pi-iso/src) |
| collab host/guest | [`collab`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/collab) |
| browser client | [`packages/collab-web`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/collab-web/src) |

排查 child 时把 agent ID、task prefix、session file、worktree/isolation root、lifecycle status 一起记录。

## A.17 按问题找 Memory

| 问题 | 入口 |
| --- | --- |
| 统一 backend interface | [`memory-backend/types.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/memory-backend/types.ts) |
| backend resolution | [`memory-backend/resolve.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/memory-backend/resolve.ts) |
| session transition | [`session/session-memory.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/session/session-memory.ts) |
| local rollout pipeline | [`memories`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/memories) |
| Hindsight | [`hindsight`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/hindsight) |
| Mnemopi adapter | [`mnemopi`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/coding-agent/src/mnemopi) |
| Mnemopi engine | [`packages/mnemopi/src`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/mnemopi/src) |
| AutoLearn | [`autolearn/controller.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/autolearn/controller.ts) |

Memory bug 要分别验证 scope、recall query、injection、retain queue、consolidation 和 child alias；“没记住”可能发生在任何一段。

## A.18 按问题找 Native

| 问题 | 入口 |
| --- | --- |
| JS 导出 | [`packages/natives/native/index.js`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/natives/native/index.js) |
| platform/ISA/load | [`loader-state.js`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/natives/native/loader-state.js) |
| N-API façade | [`crates/pi-natives/src/lib.rs`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/crates/pi-natives/src/lib.rs) |
| shell/process | [`crates/pi-shell/src`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/crates/pi-shell/src) |
| AST | [`crates/pi-ast/src`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/crates/pi-ast/src) |
| isolation | [`crates/pi-iso/src`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/crates/pi-iso/src) |
| traversal | [`crates/pi-walker/src`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/crates/pi-walker/src) |
| native package generation | [`packages/natives/scripts`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/natives/scripts) |

先判断问题在“没找到 addon”“版本/ISA 不匹配”“N-API 参数/async”“真正 Rust 算法/OS”哪一层。

## A.19 按问题找安全边界

| 问题 | 入口 |
| --- | --- |
| tool tier/policy | [`tools/approval.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/approval.ts) |
| critical bash | [`tools/bash.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/tools/bash.ts) |
| secrets loading/key | [`secrets/index.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/secrets/index.ts) |
| secrets transform | [`secrets/obfuscator.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/secrets/obfuscator.ts) |
| local URL containment | [`internal-urls/local-protocol.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/internal-urls/local-protocol.ts) |
| plan guard | `tools/plan-mode-guard.ts` |
| collab encryption | [`collab/crypto.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/collab/crypto.ts) |
| auth broker | `packages/ai/src/auth-broker` |

每次安全审查写清“受保护的资源、攻击者、入口、检查点、失败模式、剩余风险”。只说“有 validation”不够。

## A.20 按问题找观测、Stats 与发布

| 问题 | 入口 |
| --- | --- |
| Agent span 语义 | [`packages/agent/src/telemetry.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/agent/src/telemetry.ts) |
| OTLP exporter | [`telemetry-export.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/coding-agent/src/telemetry-export.ts) |
| Session stats parser | [`packages/stats/src/parser.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/stats/src/parser.ts) |
| DB/backfill | [`packages/stats/src/db.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/stats/src/db.ts) |
| worker sync | [`packages/stats/src/aggregator.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/stats/src/aggregator.ts) |
| dashboard server | [`packages/stats/src/server.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/stats/src/server.ts) |
| TS test matrix | [`scripts/ci-test-ts.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/scripts/ci-test-ts.ts) |
| Rust task | [`scripts/run-rs-task.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/scripts/run-rs-task.ts) |
| release commit/tag | [`scripts/release.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/scripts/release.ts) |
| binary matrix | [`scripts/ci-release-build-binaries.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/scripts/ci-release-build-binaries.ts) |
| package publish | [`scripts/ci-release-publish.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/scripts/ci-release-publish.ts) |

## A.21 三条推荐阅读路线

### 路线 1：第一次理解架构

```text
README/package.json
→ cli.ts
→ main.ts::runRootCommand
→ sdk.ts::createAgentSession
→ agent-session.ts::prompt
→ agent.ts::prompt/#runLoop
→ agent-loop.ts::runLoopBody
→ session-manager.ts::append/buildContext
```

### 路线 2：新增一个工具

```text
AgentTool type
→ tools/index.ts registration
→ 一个简单 read tool
→ tools/approval.ts
→ ToolSession
→ session-tools wrappers
→ agent-loop tool execution
→ output-meta/artifact
→ tool tests + session replay tests
```

### 路线 3：新增一个 provider

```text
ai/types.ts Model/API
→ registry/registry.ts
→ 现有相似 adapter
→ stream.ts dispatch
→ transform-messages/dialect
→ catalog generator/model metadata
→ auth resolver
→ provider tests + cross-provider replay + compaction tests
```

## A.22 修改前的“邻接面”检查

| 你改了 | 还应检查 |
| --- | --- |
| Agent event | AgentSession listener、TUI、RPC/ACP、extension、telemetry |
| message type | persistence、provider convert、compaction、stats、collab wire |
| tool schema/name | registry、approval、MCP/xdev、TTSR、generated tool views |
| session entry | loader/migration、branch context、stats parser、collab snapshot |
| setting | schema、migration、UI、hook、project discovery、docs |
| provider payload | retry/replay、secret boundary、telemetry content、tests |
| native export | Rust symbol、napi declaration、generated JS、loader sentinel、leaf build |
| memory backend | prompt、tools、transition/dispose、child alias、compaction |
| TUI render contract | resize、scrollback、images、headless tests、terminal restore |

这张表能防止“本文件测试通过，但另一个消费者仍按旧合同工作”。

## A.23 搜索技巧

推荐从 symbol 精确搜索：

```bash
rg -n "export async function createAgentSession" packages/coding-agent/src
rg -n "async prompt\(" packages/coding-agent/src/session/agent-session.ts
rg -n "runLoopBody" packages/agent/src
rg -n "tools\.approvalMode" packages/coding-agent/src
rg -n "type: \"compaction\"|compaction" packages/coding-agent/src/session
rg -n "__piNativesV" crates packages/natives
```

再用 `rg --files <dir> | sort` 建子目录地图。先按目录缩小，避免生成模型表、嵌入前端资产和 fixture 淹没结果。

## A.24 容易浪费时间的阅读陷阱

1. **从巨型 AgentSession 第一行开始顺读**：先找 public method，再追 helper owner。
2. **把 generated file 当手写源**：models、native exports、collab views、embedded assets 要找生成脚本。
3. **只读 interface 不读调用者**：真正不变量常在 owner 的 lifecycle/recovery 中。
4. **只读 happy path**：重点看 abort、catch、finally、resume、migration。
5. **把 README 数字当注册表事实**：产品表述与当前快照统计口径不同。
6. **把 local artifact sandbox 当 OS sandbox**：术语不等于威胁模型。
7. **忽略 legacy shim**：先确认当前主路径是否仍使用，不要从兼容层推断新架构。
8. **用总行数判断复杂度**：生成数据/测试 fixture 会扭曲。
9. **只看源码运行，不看发布拓扑**：bundle/compiled/native leaf/worker 的入口不同。
10. **看到 Promise 就假定可取消**：继续追 AbortSignal 到线程/进程实际停止点。

## A.25 一个通用调试模板

对任何跨层 bug，按下面记录：

```text
1. 输入：用户/模型/配置/外部服务给了什么？
2. identity：session、entry、toolCall、agent、provider、span ID 是什么？
3. owner：哪个对象拥有当前状态和 dispose？
4. transition：预期从什么状态到什么状态？
5. boundary：在哪做 schema/approval/transform/persist？
6. evidence：event、JSONL、payload、tool result、span 哪份证据最接近事实？
7. race：abort/reload/resume/late callback 是否跨 generation？
8. recovery：进程此刻退出，下次如何判断 terminal？
9. consumers：还有哪些 UI/RPC/stats/collab 依赖同一合同？
10. test：最小单元测试与跨层回归测试各是什么？
```

这个模板比“加日志看看”更快定位 ownership 错误。

## A.26 最短源码路线

只有一小时：

1. `package.json` 与根 README；
2. `main.ts::runRootCommand`；
3. `sdk.ts::createAgentSession` 的大段标题和关键 await；
4. `agent-session.ts::prompt/#promptWithMessage`；
5. `agent.ts::prompt/#runLoop`；
6. `agent-loop.ts::runLoopBody`；
7. `session-manager.ts::getBranch/buildSessionContext`；
8. 本教程第 20 章再走一遍。

准备修改状态机前，继续读附录 B。

---

[返回教程首页](../README.md) · [上一章：一次请求的完整旅程](./第20章-一次请求的完整旅程.md) · [附录 B：术语、状态机与不变量速查](./附录B-术语状态机与不变量速查.md)
