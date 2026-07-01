# 09 - Agent 对话流组件

## 9.1 AgentStream 模型

**文件**: `src/agent-stream/model.ts`

```typescript
interface AgentStreamModel {
  // 消息管理
  messages: StreamMessage[];
  // 流状态
  status: "idle" | "connecting" | "streaming" | "completed" | "error";
  // 工具调用
  toolCalls: ToolCallState[];
  // 消息压缩
  compaction: MessageCompaction;
}
```

## 9.2 AgentStream 视图

**文件**: `src/agent-stream/view.tsx`

**核心组件**:
- `AgentMessage` — 每条消息（用户/Assistant）
- `TurnFooter` — 消息底部时间/耗时
- `LiveElapsed` — 实时耗时计数器（100ms 更新）
- `TurnCopyButton` — 复制消息按钮

**渲染策略**: `src/agent-stream/strategy.ts`

```typescript
// Strategy 根据平台选择不同渲染方式
// Web: 虚拟滚动列表（@tanstack/react-virtual）
// Native: FlatList
```

## 9.3 AgentScreen 状态机

**文件**: `src/hooks/use-agent-screen-state-machine.ts`

```typescript
type AgentScreenState =
  | "loading"
  | "ready"
  | "error"
  | "requires-input"
  | "completed"
  | "archived";

function useAgentScreenStateMachine(agentId, serverId)
  → { state, messages, isLoading, error }
```

## 9.4 Composer 输入组件

**文件**: `src/composer/index.tsx`

**布局**:
```
┌─────────────────────────────────────┐
│ [Controls Left]   [Input]   [Voice] │
│ [Mode Bar]                          │
│ [Attachments Bar]                   │
└─────────────────────────────────────┘
```

**Props**:
```typescript
interface ComposerProps {
  initialServerId: string | null;
  initialValues: CreateAgentInitialValues;
  isVisible: boolean;
  onlineServerIds: string[];
  lockedWorkingDir: string | null;
}
```

**Action 控制**:
- `submit()` — 提交消息
- `startDictation()` — 开始听写
- `startVoiceMode()` — 启动语音模式
- `stopVoiceMode()` — 停止语音模式
- `attachFile()` — 附加文件
- `attachImage()` — 附加图片
- `insertMention(type)` — 插入 @type 引用

## 9.5 Composer Input

**文件**: `src/composer/input/input.tsx`

**功能**:
- 多行文本输入
- 占位符动态更新
- 高度自适应
- 听写模式覆盖层
- @ 快捷引用（文件、Agent、命令）
- 粘贴图片支持

**状态**:
```typescript
// useAgentInputDraft 管理输入草稿
function useAgentInputDraft({ draftKey, composer }) {
  // 返回: { text, setText, attachments, setAttachments, composerState, ... }
}
```

## 9.6 Composer Submit

**文件**: `src/composer/submit.ts`

```typescript
function submitComposerMessage({
  payload,           // { text, attachments, cwd }
  composerState,     // provider, mode, model 等
  composeKey,
}): Promise<SubmitResult>;

type SubmitResult =
  | { kind: "noop" }
  | { kind: "queued" }
  | { kind: "submitted"; agentId: string; workspaceId: string }
  | { kind: "failed"; error: string };
```

## 9.7 Agent Controls

**文件**: `src/composer/agent-controls/`

- `mode-control.tsx` — Agent 模式选择
- provider 选择器
- model 选择器
- thinking 选项

## 9.8 Attachments

**文件**: `src/composer/attachments/`

- 文件附件
- 图片附件
- GitHub PR/Issue 引用
- 附件提交分解: `splitComposerAttachmentsForSubmit(attachments)`

## 9.9 Tool Call 系统

**文件**:
- `src/components/tool-call-details.tsx` — Tool 调用详情
- `src/components/tool-call-sheet.tsx` — Tool 调用 Sheet

**文件**: `src/utils/tool-call-display.ts`

```typescript
// toolCallIconName — 根据 tool 类型返回图标名
// toolCallDetailState — 解析 tool 调用状态
```

**Tool Call 类型**:
- `read_file` / `write_file` / `edit_file`
- `run_terminal_command`
- `web_search` / `web_fetch`
- `list_directory` / `glob`
- `grep`
- `task` / `ask_user`
- `thinking`

## 9.10 消息渲染

**文件**: `src/components/message.tsx`

**用户消息**:
- 左侧头像徽标
- 文字气泡
- 附件缩略图
- 时间戳

**Assistant 消息**:
- Markdown 渲染（markdown-it）
- 代码块高亮
- Tool call 展示
- 思考过程（thinking 块）
- 消息操作: 复制 / 回退(rewind) / 分支(fork)
