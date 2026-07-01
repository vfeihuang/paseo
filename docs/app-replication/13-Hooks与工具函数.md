# 13 - Hooks 与工具函数

## 13.1 核心 Hooks

### 数据获取类

| Hook | 文件 | 用途 |
|------|------|------|
| `useAgentHistory` | `hooks/use-agent-history.ts` | 分页加载 Agent 历史列表 |
| `useAgentHistoryQuery` | `hooks/use-agent-history-query.ts` | React Query 封装的 Agent 历史 |
| `useProviderSnapshot` | `hooks/use-providers-snapshot.ts` | Provider 快照查询 |
| `useProjects` | `hooks/use-projects.ts` | 项目列表查询 |
| `useOpenProject` | `hooks/use-open-project.ts` | 打开/创建项目 |
| `useWorkspace` | `stores/session-store-hooks` | 获取工作区数据 |

### Agent 类

| Hook | 文件 | 用途 |
|------|------|------|
| `useAgentFormState` | `hooks/use-agent-form-state.ts` | Agent 表单状态（provider/mode/model） |
| `useAgentInitialization` | `hooks/use-agent-initialization.ts` | Agent 初始化流程 |
| `useAgentCommandsQuery` | `hooks/use-agent-commands-query.ts` | Agent 命令查询 |
| `useArchiveAgent` | `hooks/use-archive-agent.ts` | 归档 Agent |
| `useAgentAttention` | `utils/agent-attention.ts` | Agent 注意力管理 |
| `useAgentAttentionClear` | `hooks/use-agent-attention-clear.ts` | 清除注意力 |
| `useLoadOlderAgentHistory` | `hooks/use-load-older-agent-history.ts` | 加载更早历史 |

### Sidebar/Workspace 类

| Hook | 文件 | 用途 |
|------|------|------|
| `useSidebarWorkspacesList` | `hooks/use-sidebar-workspaces-list.ts` | 侧边栏工作区列表 |
| `useSidebarWorkspacesViewModel` | `hooks/sidebar-workspaces-view-model.ts` | 侧边栏 ViewModel |
| `useSidebarStatusViewModel` | `hooks/sidebar-status-view-model.ts` | 侧边栏状态 VM |
| `useSidebarShortcutModel` | `hooks/use-sidebar-shortcut-model.ts` | 侧边栏快捷键模型 |
| `useStatusModeWorkspaces` | `hooks/use-status-mode-workspaces.ts` | 状态模式工作区 |
| `useActiveWorktreeNewAction` | `hooks/use-active-worktree-new-action.ts` | 活跃 worktree 新操作 |
| `useGlobalNewWorkspaceAction` | `hooks/use-global-new-workspace-action.ts` | 全局新建工作区 |

### 设置类

| Hook | 文件 | 用途 |
|------|------|------|
| `useAppSettings` | `hooks/use-settings/index.ts` | 应用设置 CRUD |
| `useSettings` | `hooks/use-settings/...` | 底层设置访问 |
| `useFormPreferences` | `hooks/use-form-preferences.ts` | 表单偏好（provider/model/mode/isolation） |
| `useKeyboardShiftStyle` | `hooks/use-keyboard-shift-style.ts` | 键盘弹出偏移 |
| `useKeyboardShortcuts` | `hooks/use-keyboard-shortcuts.ts` | 键盘快捷键注册 |
| `usePreferredEditor` | `hooks/use-preferred-editor.ts` | 首选编辑器 |

### 音频/语音类

| Hook | 文件 | 用途 |
|------|------|------|
| `useDictation` | `hooks/use-dictation.ts` | 听写管理 |
| `useDictationAudioSource (native/web)` | `hooks/use-dictation-audio-source.*.ts` | 音频源 |
| `useAudioRecorder (native/web)` | `hooks/use-audio-recorder.*.ts` | 录音器 |
| `useIsDictationReady` | `hooks/use-is-dictation-ready.ts` | 听写就绪判断 |
| `useVoice` | `contexts/voice-context.tsx` | 语音模式 |

### UI 类

| Hook | 文件 | 用途 |
|------|------|------|
| `useColorScheme` | `hooks/use-color-scheme.ts` | 颜色方案 |
| `useContainerWidth` | `hooks/use-container-width.ts` | 容器宽度监听 |
| `useHoverSafeZone` | `hooks/use-hover-safe-zone.ts` | Hover 安全区 |
| `useIsCompactFormFactor` | `constants/layout.ts` | 紧凑布局检测 |
| `useIsLocalDaemon` | `hooks/use-is-local-daemon.ts` | 本地 daemon 检测 |
| `useCompactWebViewportZoomLock` | `hooks/use-compact-web-viewport-zoom-lock.ts` | 紧凑视图缩放锁定 |
| `useWebScrollbarStyle` | `hooks/use-web-scrollbar-style.ts` | Web 滚动条样式 |

### 文件类

| Hook | 文件 | 用途 |
|------|------|------|
| `useImageAttachmentPicker` | `hooks/use-image-attachment-picker.ts` | 图片附件选择 |
| `useFilePicker` | `hooks/use-file-picker.ts` | 文件选择器 |
| `useFileExplorerActions` | `hooks/use-file-explorer-actions.ts` | 文件浏览器操作 |
| `useOpenProjectPicker` | `hooks/use-open-project-picker.ts` | 项目选择器打开 |

### 键盘类

| Hook | 文件 | 用途 |
|------|------|------|
| `useKeyboardActionHandler` | `hooks/use-keyboard-action-handler.ts` | 键盘动作处理器 |
| `useKeyboardShiftStyle` | `hooks/use-keyboard-shift-style.ts` | 键盘弹出偏移 |
| `useKeyboardShortcutOverrides` | `hooks/use-keyboard-shortcut-overrides.ts` | 快捷键覆盖 |
| `useIosHardwareKeyboardSubmit` | `hooks/use-ios-hardware-keyboard-submit.ts` | iOS 物理键盘提交 |

## 13.2 工具函数

**文件**: `src/utils/`

### 路由与导航

| 函数 | 用途 |
|------|------|
| `buildOpenProjectRoute()` | 构建打开项目路由 |
| `buildSettingsSectionRoute(section)` | 构建设置 section 路由 |
| `buildSettingsHostRoute(serverId)` | 构建主机设置路由 |
| `buildSettingsHostSectionRoute(sid, sec)` | 构建主机 section 路由 |
| `buildHostRootRoute(serverId)` | 构建主机根路由 |
| `buildHostAgentDetailRoute(sid, aid)` | 构建 Agent 详情路由 |
| `buildNotificationRoute(data)` | 构建通知路由 |
| `navigateToAgent({ serverId, agentId, currentPathname, pin })` | 导航到 Agent |
| `navigateToWorkspace / navigateToLastWorkspace` | 导航到工作区 |
| `navigateToPreparedWorkspaceTab({ ... })` | 导航到准备好 Tab 的工作区 |
| `parseHostAgentRouteFromPathname` | 从路径解析主机+Agent |
| `parseServerIdFromPathname` | 从路径解析主机 ID |
| `parseWorkspaceOpenIntent` | 解析工作区打开意图 |

### Git

| 函数 | 用途 |
|------|------|
| `diffHighlight(text, lang)` | Diff 代码高亮 |
| `diffLayout` | Diff 布局计算 |
| `diffRendering` | Diff 渲染 |
| `branchSuggestions` | 分支建议 |
| `githubRefs` | GitHub 引用解析 |

### 其他

| 函数 | 用途 |
|------|------|
| `confirmDialog({ title, message, confirmLabel, cancelLabel })` | 确认弹窗 |
| `copyToClipboard(text)` | 复制到剪贴板 |
| `encodeImages(images)` | 图片编码 |
| `formatShortcut(keys)` | 格式化快捷键显示 |
| `openExternalUrl(url)` | 打开外部 URL |
| `openServiceUrl(url, behavior)` | 按设置打开服务 URL |
| `projectDisplayName(name)` | 项目显示名格式化 |
| `projectIconColor(key)` | 项目图标颜色生成 |
| `shortenPath(path, maxLen)` | 路径缩写 |
| `splitMarkdownBlocks(text)` | 分割 Markdown 块 |
| `statusDotColor(status)` | 状态点颜色 |
| `time.format(ms)` | 时间格式化 |
| `toXtermTheme(theme)` | unistyles 主题转 xterm 主题 |
| `workspaceIdentity` | 工作区身份标识 |
| `extractToolCallFilePath` | 提取 Tool 调用文件路径 |
| `fileMentionAutocomplete` | 文件 @ 引用补全 |
| `agentWorkingDirectorySuggestions` | Agent 工作目录建议 |

## 13.3 算法说明

### 侧边栏手势仲裁

**文件**: `src/utils/sidebar-gesture-arbitration.ts`

```typescript
function canOpenLeftSidebarGesture(mobilePanelState, translateX, windowWidth): boolean;
```

根据侧边栏状态和水平滚动位置判断是否允许手势打开侧边栏。

### 状态点颜色算法

**文件**: `src/utils/status-dot-color.ts`

```typescript
function statusDotColor(status: AgentStatus): string;
// 根据 Agent 状态返回对应语义色
```

### 消息高度估算

**文件**: `src/utils/assistant-message-height-estimate.ts`

```typescript
function estimateAssistantMessageHeight(text: string, width: number): number;
```

用于虚拟滚动列表的 item 尺寸估算。

### 消息压缩

**文件**: `src/components/message-compaction-label.ts`

将连续的代码块、工具调用等自动折叠为可展开的摘要标签，减少视觉噪音。
