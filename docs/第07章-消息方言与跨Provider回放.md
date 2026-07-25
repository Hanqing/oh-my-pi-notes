# 第 7 章　消息、方言与跨 Provider 回放

> 多模型适配最难的不是把 `role` 改个名字，而是让带 thinking、tool call、图片、签名和 provider 私有历史的长会话，在切换供应商后仍然合法。

## 7.1 统一消息代数

[`packages/ai/src/types.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/ai/src/types.ts) 定义四种顶层消息：

```ts
type Message =
  | UserMessage
  | DeveloperMessage
  | AssistantMessage
  | ToolResultMessage;
```

内容不是一个字符串，而是 tagged union：

- `text`；
- `thinking` / `redactedThinking`；
- `image`；
- `toolCall`；
- Anthropic fallback/server tool 私有块；
- provider metadata/payload。

统一代数的目标不是抹平差异，而是把差异装进有边界的字段。

## 7.2 四类消息各自承担什么

### UserMessage

除了 content/timestamp，还有：

- `synthetic`：系统注入；
- `steering`：中途转向，发送前强调但不展示；
- `attribution`：用户还是 Agent 发起；
- `providerPayload`：保留原生历史。

### DeveloperMessage

给支持 developer role 的 provider 保留单独语义；不支持时由 adapter 降级或合并。

### AssistantMessage

它既是内容，也是一次 provider 请求的账单与终态：

- api/provider/model；
- responseId/upstreamProvider；
- usage/cost/context snapshot；
- stop reason/details/error；
- duration/ttft；
- retry recovery；
- provider payload。

### ToolResultMessage

用 `toolCallId` 与调用配对，内容支持 text/image，还可带：

- details（本地 TUI/后续逻辑）；
- provider metadata；
- `isError`；
- `prunedAt`；
- `useless`，提示 compaction 可安全省略。

## 7.3 ToolCall 为什么字段这么多

一个统一 `ToolCall` 包含：

- `id/name/arguments`；
- streaming partial JSON；
- Google thought signature；
- harness intent；
- owned dialect 的 raw block；
- OpenAI custom tool 的 `customWireName`；
- computer use 的 provider metadata。

这些字段可以分三层：

| 层 | 字段例子 | 是否跨 provider 保留 |
| --- | --- | --- |
| 语义 | id/name/arguments | 是，但 ID 可能规范化 |
| harness | intent/rawBlock/customWireName | 视目标能力 |
| provider 私有 | thoughtSignature/providerMetadata | 通常只同源回放 |

跨 provider 转换不能无脑 JSON stringify 全部字段。

## 7.4 从产品消息到 provider context 的流水线

```mermaid
flowchart LR
    AM["AgentMessage\n含 custom messages"] --> C["convertToLlm"]
    C --> TC["transformContext"]
    TC --> PC["pi-ai Context"]
    PC --> TP["transformProviderContext\nsecret/image policy"]
    TP --> TM["transformMessages\nrepair + cross-provider"]
    TM --> AD["provider adapter"]
    AD --> W["wire request"]
```

### `convertToLlm`

Agent core 允许 coding-agent 通过 TypeScript declaration merging 扩展 `AgentMessage`。plan、goal、skill、advisory 等 custom message 不能直接送 provider，必须先转换为标准 user/developer/system 语义或过滤。

### `transformContext`

加入 turn 级上下文，例如 steering 包装、extension `context` 事件修改。

### `transformProviderContext`

发生在目标 model 已知之后，可做 secret obfuscation、图像尺寸/数量 clamp 等。

### Provider adapter

最后把统一 message/tool schema 转成 Anthropic blocks、OpenAI input items、Gemini contents、Bedrock Converse 等真实协议。

## 7.5 `transformMessages()` 是历史修复站

[`packages/ai/src/providers/transform-messages.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/ai/src/providers/transform-messages.ts) 处理很多长期会话才会遇到的问题：

- 丢弃空 id/name 的 malformed tool call；
- 同步丢弃匹配的 tool result；
- 规范化目标 provider 不接受的 tool call ID；
- 更新对应 result ID；
- 清除跨模型不再有效的 thought signature；
- 过滤 provider-only content block；
- 修复 assistant/tool-result 交替结构；
- 给缺失结果的调用生成 synthetic result；
- 避免 duplicate result 二次进入 provider；
- 处理 aborted/stale tool result；
- 丢弃没有可行动内容的 assistant turn。

它不是格式化器，而是 transcript consistency pass。

## 7.6 为什么工具 ID 需要重写

不同 API 对 ID 的字符集、长度和前缀要求不同。若 assistant 中的 call ID 被重写，之后所有 `ToolResultMessage.toolCallId` 必须同步变化。

实现维护映射：

```text
original call id  ──normalize──> target id
      │                              │
      └──── matching result ─────────┘
```

如果只改调用不改结果，provider 会认为 result 没有对应 call；如果多个坏 ID 被错误合并，又会出现一个 result 对多个 call。

## 7.7 同 Provider 回放与跨 Provider 回放

### 同源回放

尽可能保留：

- response/item ID；
- signed thinking；
- OpenAI Responses 原生 history items；
- Anthropic server tool blocks；
- computer use metadata；
- prompt cache 有利的稳定前缀。

### 跨源回放

只保留语义可移植部分：

- 可见 text；
- 必要 thinking 摘要（按目标策略）；
- 标准 tool call/result；
- 图片；
- 合法 user/developer role。

Opaque payload 不是“信息越多越好”。把 Anthropic signature 发给 OpenAI 没意义，把 OpenAI response item 原样发给自定义兼容端还可能触发 schema error。

## 7.8 方言层解决“没有原生工具调用”的模型

[`packages/ai/src/dialect/`](https://github.com/can1357/oh-my-pi/tree/v17.1.3/packages/ai/src/dialect) 包含 Anthropic、DeepSeek、Gemini、Gemma、GLM、Harmony、Hermes、Kimi、Qwen、XML 等定义。

方言负责：

- 把 tool definitions 渲染进 prompt；
- 把历史 call/result 渲染成模型训练过的语法；
- 从流式文本解析新 tool call；

- 渲染 thinking 边界；
- 声明 tool result 应扮演 `tool` 还是 `user` role；
- 提供示例语法。

“owned dialect”表示 harness 拥有这段语法的生成和解析，而不是 provider SDK 返回原生 structured tool call。

## 7.9 原生 function calling 也不等于完全统一

即使 OpenAI/Anthropic/Gemini 都支持工具，它们仍有差异：

- schema strictness；
- `additionalProperties`；
- tool choice 形状；
- parallel tool calls；
- thinking/signature 与 tool use 的排列；
- result role 与 block grouping；
- custom grammar tools；
- hosted/native computer tool。

`Tool` 类型同时支持 Zod、ArkType 和 JSON Schema；进入 wire 前由 schema normalizer 转为目标 provider 允许的子集。

## 7.10 流式 event 与持久化 message 是两种对象

`AssistantMessageEvent` 包含 start/delta/end/terminal。每个事件带当前 partial message，适合 UI。

持久化只应该在明确边界记录稳定消息。否则可能出现：

- 每个 token 一条 JSONL；
- resume 时把多个 partial 当多个 assistant turn；
- toolcall partial JSON 被误执行；
- stats 重复计费。

所以 event 是过程，message 是状态；两者不能用同一个数组代替。

## 7.11 ProviderPayload 是受控的“逃生舱”

统一模型不可避免会漏掉 provider 的特殊历史结构。`providerPayload` 允许保存 opaque 数据，例如 OpenAI Responses history items。

但它有严格使用原则：

- 只由对应 provider adapter 解释；
- 跨 provider 时丢弃或转换；
- 本地 UI 不应依赖它展示基本对话；
- 出错时仍有标准 content/usage/stopReason 可用。

这是“80% 统一 + 20% 可隔离逃生”的设计，而不是让所有上层理解每种 provider JSON。

## 7.12 Append-only context 为什么还要 fingerprint

启用 append-only provider context 时，管理器会跟踪已发送消息的稳定身份/序列化形状。它需要识别：

- 纯追加；
- 旧消息被替换；
- provider payload/tool IDs 变化；
- compaction 后前缀重写。

只有纯追加才能安全复用增量上下文；检测到历史变形就要重建。错误地假设 append-only 会把旧 provider cache 与新 transcript 混合。

## 7.13 协作 wire 消息为何另有一套类型

[`packages/wire/src/index.ts`](https://github.com/can1357/oh-my-pi/blob/v17.1.3/packages/wire/src/index.ts) 复制了浏览器需要的消息子集：text/image/thinking/tool call、session entries 和 events。

它不是 provider wire，也不是完整 AgentMessage：

- 不暴露 provider 私有 payload；
- details 用 `unknown`；
- 未知 entry/event 要宽容跳过；
- `message_update` 明确传完整累积 partial；
- 协议带自己的 `COLLAB_PROTO` 版本。

这建立了 host 到浏览器的最小公开契约。

## 7.14 消息层的五条不变量

1. Tool call/result ID 必须配对。
2. Provider 私有签名只在允许的回放路径保留。
3. 流 partial 不等于持久化 final。
4. 跨 provider 转换优先保留语义，而不是保留全部字段。
5. 错误/中断也是结构化 assistant message，不能只存在日志里。

## 7.15 调试消息兼容问题

建议保存四个快照做 diff：

1. SessionManager 重建出的 AgentMessage；
2. `convertToLlm` 后的标准 Message；
3. `transformMessages` 后的目标 context；
4. provider adapter 实际 wire request。

若只看第 4 步，很难判断字段是在何处丢失；若只看 session JSONL，又看不到目标 provider 的合法性修复。

## 7.16 本章小结

统一消息模型不是最低公分母，而是一个带受控私有字段的语义中间表示。跨 provider 时，`transformMessages()` 保证历史合法；方言层让没有可靠原生工具协议的模型也能进入同一 Agent Loop。

下一章进入 Agent 的“系统调用”边界：[工具注册、调度与审批](./第08章-工具注册调度与审批.md)。
