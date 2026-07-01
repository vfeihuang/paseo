# 05 - Store 状态管理

## 5.1 架构概览

使用 **Zustand v5** 管理全局状态，配合 `@tanstack/react-query` 管理服务端数据。每个 Store 独立文件，使用 `create()` 或 `createStore()` 创建。

## 5.2 核心 Store

### SessionStore

**文件**: `src/stores/session-store.ts`

**状态**:
```typescript
interface SessionStoreState {
  sessions: Record<string, SessionState>;   // keyed by serverId
}

interface SessionState {
  client: DaemonClient | null;
  serverInfo: ServerInfo | null;
  agents: Record<string, AgentState>;
  workspaces: Record<string, WorkspaceDescriptor>;
  agentDirectory: AgentDirectoryState | null;
}
```

**Action**:
- `initializeSession(serverId, client)` — 初始化新会话
- `clearSession(serverId)` — 清理会话
- `mergeAgents(serverId, agents)` — 合并 Agent 列表
- `mergeWorkspaces(serverId, workspaces)` — 合并工作区列表
- `updateSessionAgent(serverId, agentId, update)` — 更新单个 Agent
- `updateSessionWorkspace(serverId, workspaceId, update)` — 更新单个工作区
- `archiveAgent(serverId, agentId)` — 归档 Agent
- `addMessageToAgent(serverId, agentId, message)` — 添加消息

### SessionStoreHooks

**文件**: `src/stores/session-store-hooks/`

```typescript
useSession(serverId)              → SessionState | null
useAgent(serverId, agentId)       → AgentState | null
useWorkspace(serverId, workspaceId) → WorkspaceDescriptor | null
useWorkspaceExists(serverId, workspaceId) → boolean
useHasHydratedWorkspaces(serverId) → boolean
```

### PanelStore

**文件**: `src/stores/panel-store/`（目录，含 `index.ts`）

**状态**:
```typescript
interface PanelStoreState {
  showMobileAgentList: boolean;         // 移动端 Agent 列表显示
  desktop: {
    agentListOpen: boolean;              // 桌面端 Agent 列表
    fileExplorerOpen: boolean;           // 桌面端文件浏览器
    focusModeEnabled: boolean;           // 专注模式
  };
}
```

**Action**:
- `toggleMobileAgentList()`
- `toggleDesktopAgentList()`
- `openDesktopAgentList()`
- `closeDesktopAgentList()`
- `closeDesktopFileExplorer()`
- `toggleFocusMode()`
- `setFocusMode(v)`

### WorkspaceLayoutStore

**文件**: `src/stores/workspace-layout-store.ts`

**状态**:
```typescript
interface WorkspaceLayoutState {
  layouts: Record<string, WorkspaceLayout>;  // keyed by persistence key
}

interface WorkspaceLayout {
  panes: Pane[];
  tabs: Record<string, WorkspaceTab>;
  activeTabId: string | null;
  focusedPaneId: string | null;
  focusedBrowserId: string | null;
  deck: WorkspaceDeckEntry[];
}
```

**Key Action**:
- `focusPane(key, paneId)`
- `focusTab(key, tabId)`
- `openTab(key, target)`
- `closeTab(key, tabId)`
- `moveTab(key, tabId, targetPaneId)`
- `splitPane(key, paneId, direction)`
- `mergePane(key, paneId)`

### CreateFlowStore

**文件**: `src/stores/create-flow-store.ts`

**状态**:
```typescript
interface CreateFlowStoreState {
  pendingByDraftId: Record<string, PendingCreateEntry>;
}
```

追踪新建工作区的创建流程，用于在创建过程中显示加载状态。

### DraftStore

**文件**: `src/stores/draft-store/`

**状态**:
```typescript
interface DraftStoreState {
  drafts: Record<string, DraftState>;  // keyed by draftKey
}

interface DraftState {
  text: string;
  attachments: ComposerAttachment[];
  cursorPosition: number;
}
```

**Action**:
- `setDraftInput(key, { text, attachments })`
- `clearDraftInput(key, lifecycle)`
- `getDraft(key)`
- `setCursorPosition(key, pos)`

### WorkspaceTabsStore

**文件**: `src/stores/workspace-tabs-store/`

管理工作区内的 Tab 目标定义:
```typescript
type WorkspaceTabTarget =
  | { kind: "draft"; draftId: string; setup?: WorkspaceDraftTabSetup }
  | { kind: "agent"; agentId: string }
  | { kind: "terminal"; terminalId: string }
  | { kind: "file"; path: string; lineStart?: number; lineEnd?: number }
  | { kind: "browser"; browserId: string }
  | { kind: "setup" };
```

### BrowserStore

**文件**: `src/stores/browser-store/`

```typescript
interface BrowserStore {
  browsers: Record<string, BrowserState>;
  createWorkspaceBrowser(workspaceKey) → string;
  // ...
}
```

### ProviderSettingsStore

**文件**: `src/stores/provider-settings-store.ts`

管理 AI provider 设置，包括 API key、端点等。

### ProjectPickerStore

**文件**: `src/stores/project-picker-store.ts`

管理项目选择器的状态，包括搜索、过滤。

### WorkspaceSetupStore

**文件**: `src/stores/workspace-setup-store.ts`

```typescript
shouldShowWorkspaceSetup(serverId, workspaceId) → boolean
```

### WorkspaceDraftSubmissionStore

**文件**: `src/stores/workspace-draft-submission-store.ts`

管理 workspace draft 提交流程的待处理状态。

### LastWorkspaceSelection

**文件**: `src/stores/navigation-active-workspace-store/`

```typescript
interface LastWorkspaceSelection {
  serverId: string;
  workspaceId: string;
}
// useLastWorkspaceSelection()
// navigateToWorkspace(serverId, workspaceId) → boolean
// navigateToLastWorkspace() → boolean
```

### Sidebar 相关 Store

**文件**:
- `src/stores/sidebar-order-store.ts` — 侧边栏顺序
- `src/stores/sidebar-view-store.ts` — 侧边栏视图设置
- `src/stores/sidebar-collapsed-sections-store/`（目录）— 折叠区域

### KeyboardShortcutsStore

**文件**: `src/stores/keyboard-shortcuts-store.ts`

```typescript
interface KeyboardShortcutsStore {
  enabled: boolean;
  isMobile: boolean;
  overrides: ShortcutOverride[];
}
```

## 5.3 React Query 集成

**文件**: `src/query/query-client.ts`

```typescript
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: { staleTime: 30_000, retry: 1 },
  },
});
```

主要用于:
- Agent history 查询
- Provider catalog
- Projects 列表
- Settings 持久化
- Provider usage quota

## 5.4 Store 设计模式

1. **Per-server session**: `sessionStore.sessions[serverId]`
2. **Per-workspace layout**: `workspaceLayoutStore.layouts[persistenceKey]`
3. **Per-draft draft**: `draftStore.drafts[draftKey]`
4. **Context + store**: 部分 context 包装 zustand store（如 VoiceContext）
5. **External store**: 使用 `useSyncExternalStore` 连接非 React 运行时（如 HostRuntime）

## 5.5 Hook 封装模式

```typescript
// 从 store 中订阅部分状态
export function useWorkspace(serverId: string | null, workspaceId: string | null) {
  return useSessionStore((state) => {
    if (!serverId || !workspaceId) return null;
    return state.sessions[serverId]?.workspaces[workspaceId] ?? null;
  });
}
```
