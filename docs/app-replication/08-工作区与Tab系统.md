# 08 - 工作区与 Tab 系统

## 8.1 架构概览

工作区(Workspace) 是 Paseo 的核心工作单元。每个工作区包含多个 **Tab**，Tab 类型包括 Agent / Terminal / File / Browser / Draft / Setup。桌面端支持多面板 **Pane** 分割。

## 8.2 WorkspaceTab 目标类型

**文件**: `src/stores/workspace-tabs-store/`

```typescript
type WorkspaceTabTarget =
  | { kind: "draft"; draftId: string; setup?: WorkspaceDraftTabSetup }
  | { kind: "agent"; agentId: string }
  | { kind: "terminal"; terminalId: string }
  | { kind: "file"; path: string; lineStart?: number; lineEnd?: number }
  | { kind: "browser"; browserId: string }
  | { kind: "setup" };

interface WorkspaceTab {
  id: string;
  target: WorkspaceTabTarget;
  createdAt: number;
}

interface WorkspaceDraftTabSetup {
  provider: AgentProvider;
  cwd: string;
  modeId: string | null;
  model: string | null;
  thinkingOptionId: string | null;
  featureValues: Record<string, unknown>;
}
```

## 8.3 WorkspaceLayout 定义

**文件**: `src/stores/workspace-layout-store.ts`

```typescript
interface WorkspaceLayout {
  panes: Pane[];                           // 面板列表
  tabs: Record<string, WorkspaceTab>;      // Tab 映射
  activeTabId: string | null;
  focusedPaneId: string | null;
  focusedBrowserId: string | null;
  deck: WorkspaceDeckEntry[];              // 桌面端 deck
}

interface Pane {
  id: string;
  tabIds: string[];                         // 该面板中的 Tab ID 列表（有序）
}

interface WorkspaceDeckEntry {
  workspaceId: string;
  serverId: string;
}
```

## 8.4 面板布局系统

**文件**: `src/components/split-container.tsx`

桌面端使用 `SplitContainer` 实现多面板布局。工作流：

1. `WorkspaceLayoutStore` 管理 `panes` 和 `tabs`
2. `SplitContainer` 根据 `panes` 渲染多个区域
3. 每个 Pane 内部渲染 Tab 列表，通过 `WorkspacePaneContent` 渲染内容
4. 支持拖拽调整面板大小
5. 支持 Tab 拖拽跨面板移动

**Pane 状态转换**:
```typescript
function deriveWorkspacePaneState(layout, tabId) → {
  paneId: string | null;
  focused: boolean;
  tabIds: string[];
  activeTabId: string | null;
}
```

## 8.5 Tab 切换器（Mobile）

**文件**: `src/screens/workspace/workspace-screen.tsx` — `MobileWorkspaceTabSwitcher`

- Pressable trigger 显示当前 Tab 图标 + 名称 + `ChevronDown`
- 点击打开 Combobox 列出所有 Tab
- 每个 Tab 选项显示图标 + 标题 + 右键菜单(Ellipsis)
- 右键菜单操作: Copy resume command / Copy ID / Copy path / Reload / Rename / Close
- 关闭支持 bulk close 逻辑（agent type 有关闭策略）

## 8.6 Tab 菜单构建

**文件**: `src/screens/workspace/workspace-tab-menu.ts`

```typescript
function buildWorkspaceTabMenuEntries({
  surface: "desktop" | "mobile",
  tab,
  index,
  tabCount,
  onCopyResumeCommand,
  onCopyAgentId,
  onCopyFilePath,
  onReloadAgent,
  onRenameTab,
  onCloseTab,
  onCloseTabsBefore,
  onCloseTabsAfter,
  onCloseOtherTabs,
  labels,
}): WorkspaceTabMenuEntry[]
```

Desktop 模式下支持 `closeLeft` / `closeRight`（相对于选项卡位置）。

## 8.7 Tab 展示

**文件**: `src/screens/workspace/workspace-tab-presentation.tsx`

```typescript
interface WorkspaceTabPresentation {
  label: string;
  icon: ComponentType;           // Tab 图标
  iconColor: string;
  iconSize: number;
  titleState: "ready" | "loading";
}

// 通过 WorkspaceTabPresentationResolver 根据 target 解析展示信息
// WorkspaceTabIcon — 渲染图标
// WorkspaceTabOptionRow — 渲染一行（用于 Combobox）
```

## 8.8 WorkspaceHeaderSource

**文件**: `src/screens/workspace/workspace-header-source.ts`

```typescript
function resolveWorkspaceHeaderRenderState({
  workspace,
  checkoutState,
}): WorkspaceHeaderRenderState {
  // 解析: title, subtitle, shouldShowSubtitle, isGitCheckout, currentBranchName
}
```

标题来源:
- `workspace.descriptor.label` → 手动设置名称
- `workspace.workspaceDirectory` → 自动从路径取最后一段
- subtitle → 项目名或分支名

## 8.9 分支切换（BranchSwitcher）

**文件**: `src/components/branch-switcher.tsx`

**Props**:
```typescript
interface BranchSwitcherProps {
  currentBranchName: string | null;
  serverId: string;
  workspaceId: string;
  workspaceDirectory: string | null;
  isGitCheckout: boolean;
  testID?: string;
}
```

**UI**:
- 圆角 badge trigger: `GitBranch` 图标(14px) + 分支名 + `ChevronDown` 图标
- hover 时背景变 surface2
- 弹出 Combobox: 分支列表，每项 GitBranch 图标 + 分支名
- 调用 `useBranchSwitcher` hook 管理分支列表加载、创建和切换

## 8.11 终端集成

**文件**:
- `src/screens/workspace/terminals/use-workspace-terminals.ts` — 终端生命周期管理
- `src/terminal/` — 终端模拟器实现

**使用流程**:
```typescript
const { terminals, createTerminal, destroyTerminal } = useWorkspaceTerminals(serverId, workspaceId);
```

终端支持 `xterm.js`（Web）和原生实现（Native）。

## 8.12 文件打开

**文件**: `src/screens/workspace/workspace-file-open-command.ts`

```typescript
function handleOpenFile(serverId, workspaceId, filePath, lineStart?, lineEnd?) {
  // 创建或跳转到 file tab
}
```

## 8.13 Bulk Close

**文件**: `src/screens/workspace/workspace-bulk-close.ts`

```typescript
// classifyBulkClosableTabs — 分类可关闭的 Tab
// closeBulkWorkspaceTabs — 执行批量关闭
// buildBulkCloseConfirmationMessage — 构建确认消息
```

## 8.14 Workspace Route State

**文件**: `src/screens/workspace/workspace-route-state.ts`

```typescript
type WorkspaceRouteState =
  | { kind: "loading" }
  | { kind: "host-unreachable" }
  | { kind: "host-mismatch" }
  | { kind: "restoring" }
  | { kind: "missing-workspace" }
  | { kind: "ready" };
```

`resolveWorkspaceRouteState` 根据连接状态和 workspace 存在性决定显示内容。

## 8.15 Deck 系统（桌面端）

**文件**: `src/screens/workspace/workspace-deck-retention.ts`

桌面端支持多工作区 Deck（类似浏览器标签页），每个 Deck entry 包含 `workspaceId` + `serverId`。

```typescript
function workspaceDeckRetention({ layout, isFocused }) → void
// 非聚焦时自动移除不可见 workspace 的 tab 占用
```

## 8.16 Workspace 工具栏按钮

### WorkspaceScriptsButton
- **文件**: `src/screens/workspace/workspace-scripts-button.tsx`
- 在工作区 header 中显示 scripts 快捷按钮
- 使用 `Tooltip` 包裹，可配置 `hideLabels` 和 `presentation`("ghost"/"button")
- 交互: `onPress` → 执行脚本 RPC → 脚本完成后显示 toast "Script finished"
- 由 `WorkspaceScreen` 根据工作区 header 状态决定是否渲染

### WorkspaceOpenInEditorButton
- **文件**: `src/screens/workspace/workspace-open-in-editor-button.tsx`
- 在 explorer 工具栏中显示，打开当前文件到外部编辑器
- 交互: `onPress` → `usePreferredEditor` 获取偏好编辑器 → `openInEditor(filePath)` → 打开外部编辑器
- disabled 条件: 无偏好编辑器配置时

## 8.17 Workspace Desktop 面板

### SplitContainer
- **文件**: `src/components/split-container.tsx`
- 桌面端多面板布局容器
- 支持: 拖拽分隔条调整大小、分屏(split right/down)、Tab 拖拽跨面板移动
- 使用 `useWorkspaceLayoutStore` 管理布局状态

### WorkspaceDesktopTabsRow
- **文件**: `src/screens/workspace/workspace-desktop-tabs-row.tsx`
- 桌面端横排 Tab 栏
- 支持: 拖拽排序、右键菜单、hover 关闭按钮

### WorkspacePaneContent
- **文件**: `src/screens/workspace/workspace-pane-content.tsx`
- 渲染 pane 内容的通用组件
- 根据 tab target kind 分发到: Agent/File/Terminal/Browser/Setup/Draft

## 8.18 左侧边栏（LeftSidebar）

**文件**: `src/components/sidebar/`（多文件）

**渲染**: 在 `AppWithSidebar` 中渲染，覆盖所有页面

**桌面端**:
- 固定宽度面板（surfaceSidebar 背景色）
- 显示: Agent 列表 + 工作区列表 + 底部导航图标
- 通过侧边栏呼出 slot（`SidebarCalloutContext`）展示通知/提示
- 快捷键 `Ctrl+B` 切换显隐，`Ctrl+0` 聚焦
- 使用 `SidebarOrderStore` / `SidebarViewStore` 管理排序和视图

**移动端**:
- 全屏覆盖层，从左侧滑入（`SidebarAnimationContext` 控制手势动画）
- 手势滑动打开（`canOpenLeftSidebarGesture` 仲裁: 检查面板状态和水平滚动位置）
- 内容: 与桌面端相同 + 可折叠 section（`SidebarCollapsedSectionsStore`）
- 硬件返回键关闭

**点击交互**:
- **Agent 行**: `onPress(agent)` → `navigateToAgent({ serverId, agentId, currentPathname, pin })` → 导航到对应 Agent 详情
- **工作区行**: `onPress(workspace)` → `navigateToWorkspace(serverId, workspaceId)` → 切换到该工作区
- **底部导航图标**:
  - 搜索图标 → 打开 `CommandCenter`（快捷键 `toggle-command-center`）
  - 设置图标 → `router.navigate(buildSettingsSectionRoute("general"))` → 跳转到设置
- **Ellipsis 菜单（工作区行 trailing）**: `DropdownMenu` → Rename / Close / Archive
- **DiffStat 徽章（工作区行）**: 显示 `workspace.descriptor.diffStat` 变化文件数

**Callout 系统**:
- `useSidebarCalloutContext` 提供 `showCallout` / `dismissCallout`
- 提示类型: WorktreeSetup, Rosetta, Update
- 在侧边栏顶部或内容区域间渲染

## 8.19 Explorer 侧边栏（ExplorerSidebar）

**文件**: `src/components/explorer-sidebar.tsx`

**渲染**: 仅在 `WorkspaceScreen` 桌面端显示，右侧面板

**UI 布局**:
- `ExplorerSidebarAnimationContext` 控制显隐动画
- 切换按钮: `SourceControlPanelIcon` + DiffStat 徽章（`workspace.descriptor.diffStat`）
- 内容: File explorer / Git diff / Pull request 面板

**交互**:
- 快捷键 `Ctrl+Shift+B` 切换，`Ctrl+Shift+0` 聚焦
- 切换按钮（`SourceControlPanelIcon` + DiffStat 徽章）:
  - 点击 → `togglePanel("file-explorer")` → 切换显隐
- 面板切换（File explorer / Git diff / Pull request）:
  - 通过 tab 按钮或 SegmentedControl 切换面板类型
- **点击文件**: `handleOpenFileFromExplorer` → 创建/跳转 file tab（可传 lineStart/lineEnd）
- 移动端: 通过 `CompactExplorerSidebarHost` 管理，硬件返回键关闭
- 状态: `usePanelStore` 中的 `selectIsFileExplorerOpen`

**ExplorerCheckoutContext**:
```typescript
interface ExplorerCheckoutContext {
  serverId: string;
  cwd: string;
  isGit: boolean;
}
```
- 用于文件浏览器的 git 状态显示和分支切换
