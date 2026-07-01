# 04 - 全局 Context 与 Provider

## 4.1 Provider 层级总览

```
RootLayout
  └─ GestureHandlerRootView
      └─ RootProviders
          │   QueryClientProvider     ─── @tanstack/react-query
          │   I18nProvider            ─── i18next
          │   SafeAreaProvider        ─── react-native-safe-area-context
          │   KeyboardProvider        ─── react-native-keyboard-controller
          │   KeyboardShiftProvider   ─── useKeyboardShiftStyle
          │   PortalProvider          ─── @gorhom/portal
          │   BottomSheetModalProvider ── @gorhom/bottom-sheet
          └─ RuntimeProviders
              │   HostRuntimeBootstrapProvider  ─── 启动逻辑
              │   PushNotificationRouter        ─── 通知处理（非Provider）
              │   SidebarCalloutProvider        ─── 侧边栏呼出
              │   ToastProvider                 ─── 吐司通知
              └─ ProvidersWrapper
                  │   VoiceProvider             ─── 音频引擎
                  │   DesktopWindowControlsSync ─── 桌面窗口控制
                  │   OfferLinkListener         ─── 配对链接监听
                  │   HostSessionManager        ─── 主机会话管理
                  │   FaviconStatusSync         ─── Favicon
                  └─ AppShell
                      │   SidebarAnimationProvider  ─── 侧边栏动画
                      │   HorizontalScrollProvider  ─── 水平滚动
                      │   OpenProjectListener        ─── 打开项目监听
                      └─ AppWithSidebar
                          │   SidebarAnimationProvider(compact)
                          │   ExplorerSidebarAnimationProvider(compact)
                          └─ RootStack
```

## 4.2 核心 Context 定义

### HostRuntimeBootstrapContext

**文件**: `src/app/_layout.tsx`

```typescript
interface HostRuntimeBootstrapState {
  splashError: string | null;
  retry: () => void;
  hasGivenUpWaitingForHost: boolean;
  storeReady: boolean;
  startupBlocker: StartupBlocker;
}
// 通过 useHostRuntimeBootstrapState() 获取
// 通过 useStoreReady() 获取 storeReady 状态
```

### VoiceContext

**文件**: `src/contexts/voice-context.tsx`

```typescript
// VoiceProvider 提供音频引擎单例
// useVoice()        → 获取 voice runtime 快照 + 控制方法
// useVoiceTelemetry() → 获取 telemetry 数据
// useVoiceAudioEngineOptional() → 获取音频引擎实例（可选）
```

### SidebarAnimationContext

**文件**: `src/contexts/sidebar-animation-context.tsx`

```typescript
// 侧边栏动画共享状态（移动端滑动手势）
// useSidebarAnimation() → translateX, backdropOpacity, windowWidth,
//                         animateToOpen, animateToClose, setOverlayPeek,
//                         isGesturing, mobilePanelState, gestureAnimatingRef, openGestureRef
```

### ExplorerSidebarAnimationContext

**文件**: `src/contexts/explorer-sidebar-animation-context.tsx`

```typescript
// Explorer 侧边栏（右侧）动画状态
```

### HorizontalScrollContext

**文件**: `src/contexts/horizontal-scroll-context.tsx`

```typescript
// 水平滚动状态共享
// useHorizontalScrollOptional() → isAnyScrolledRight（用于手势仲裁）
```

### SidebarCalloutContext

**文件**: `src/contexts/sidebar-callout-context.tsx`

```typescript
// 侧边栏呼出提示管理
// 控制 workspace setup / worktree 等 callout 显示
```

### ToastContext

**文件**: `src/contexts/toast-context.tsx`

```typescript
// useToast() → { show(message), error(message) }
// 全局吐司通知，结合 DownloadToast 组件
```

### SessionContext

**文件**: `src/contexts/session-context.tsx`

```typescript
// SessionProvider(serverId, client) → 为每个主机/daemon 创建会话上下文
// 内部管理会话状态和 service status 跟踪
```

### HostRouteContext

**文件**: `src/navigation/host-route-context.tsx`

```typescript
// HostRouteProvider(serverId) → 为 /h/:serverId 路由组提供主机上下文
// useHostRouteServerId() → 获取当前主机 ID
```

## 4.3 生命周期管理

### 启动引导（HostRuntimeBootstrapProvider）

```typescript
useEffect(() => {
  startHostRuntimeBootstrap({
    store,
    daemonStartService,
    shouldStartDaemon: shouldStartBuiltInDaemon,
    onGateError: (message) => daemonStartService.recordError(message),
  });
}, []);
```

**启动状态机**:
1. 等待 host registry 加载
2. 自动启动内置 daemon（desktop 模式）
3. 5 秒超时后允许用户跳过（`hasGivenUpWaitingForHost`）
4. 根据 `resolveStartupBlocker` 决定是否阻塞

### ProvidersWrapper 主题/外观同步

```typescript
// 主题同步
useEffect(() => {
  if (settings.theme === "auto") UnistylesRuntime.setAdaptiveThemes(true);
  else {
    UnistylesRuntime.setAdaptiveThemes(false);
    UnistylesRuntime.setTheme(THEME_TO_UNISTYLES[settings.theme]);
  }
}, [settings.theme]);

// 外观同步（字体/字号/语法高亮）
useEffect(() => {
  applyAppearance({ uiFontFamily, monoFontFamily, uiFontSize, codeFontSize, syntaxTheme });
}, [settings.uiFontFamily, settings.monoFontFamily, ...]);
```
