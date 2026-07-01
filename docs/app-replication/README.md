# Paseo 移动端 APP 完整复刻文档索引

> **项目**: Paseo — 语音控制的 AI 编码助手移动端应用
> **框架**: React Native (Expo) + TypeScript
> **版本**: 0.1.102-beta.2
> **包名**: `@getpaseo/app`
> **入口**: `packages/app/index.ts`

---

## 文档目录

| # | 文档 | 说明 |
|---|------|------|
| 1 | [01-架构与工程配置.md](./01-架构与工程配置.md) | 项目结构、技术栈、构建配置、polyfill |
| 2 | [02-主题与样式系统.md](./02-主题与样式系统.md) | 主题色板、语义色、暗色变体、间距/字号/圆角/图标尺寸常量、unistyles、阴影 |
| 3 | [03-路由与导航系统.md](./03-路由与导航系统.md) | Expo Router 路由表、Stack 嵌套结构、Host/Workspace/Agent 路由、布局守卫 |
| 4 | [04-全局Context与Provider.md](./04-全局Context与Provider.md) | Provider 嵌套顺序、Bootstrap/Session/Voice/Toast/Sidebar Context |
| 5 | [05-Store状态管理.md](./05-Store状态管理.md) | Zustand 各 Store 的 state/action 完整定义（session/panel/layout/draft/browser 等） |
| 6 | [06-页面与屏幕.md](./06-页面与屏幕.md) | 所有页面的 UI 布局、交互逻辑、事件处理、状态机 |
| 7 | [07-UI组件库.md](./07-UI组件库.md) | 全部通用组件的 Props、状态、渲染逻辑（Button/Dropdown/Combobox/Tooltip 等） |
| 8 | [08-工作区与Tab系统.md](./08-工作区与Tab系统.md) | Workspace 面板管理、Tab 导航、Pane 布局、拖拽排序、终端集成 |
| 9 | [09-Agent对话流组件.md](./09-Agent对话流组件.md) | AgentStream 模型、消息渲染、ToolCall 展示、Composer 输入组件 |
| 10 | [10-语音与音频引擎.md](./10-语音与音频引擎.md) | 双平台音频引擎、VAD、语音运行时状态机、录音/播放管道 |
| 11 | [11-网络层与WebSocket.md](./11-网络层与WebSocket.md) | HostRuntime、连接管理、WebSocket 生命周期、Agent Directory |
| 12 | [12-设置页面详情.md](./12-设置页面详情.md) | 设置面板所有 section 的完整 UI/逻辑/交互 |
| 13 | [13-Hooks与工具函数.md](./13-Hooks与工具函数.md) | 核心 hooks 签名、工具函数分类、算法说明 |
| 14 | [14-国际化与平台适配.md](./14-国际化与平台适配.md) | i18n 配置、8种语言、isWeb/isNative/isElectron 平台门控 |
| 15 | [15-键盘快捷键系统.md](./15-键盘快捷键系统.md) | 全部 60+ 快捷键绑定、ActionDispatcher、作用域管理 |

---

## 核心架构图

```
RootLayout (GestureHandlerRootView)
  └─ RootProviders (QueryClient / I18n / SafeArea / Keyboard / Portal / BottomSheet)
      └─ RuntimeProviders (HostRuntimeBootstrap / PushNotification / SidebarCallout / Toast / Voice / Settings)
          └─ AppShell (SidebarAnimationProvider / HorizontalScrollProvider)
              ├─ OpenProjectListener
              ├─ AppWithSidebar
              │   ├─ LeftSidebar (compact/desktop)
              │   ├─ CompactExplorerSidebarHost (mobile)
              │   └─ RootStack (Expo Router Stack)
              └─ Global overlays (CommandCenter / ProjectPicker / KeyboardShortcutsDialog / etc.)
```

## 路由层级

```
/                              → 启动重定向（Splash/Index）
/welcome                       → 欢迎页
/settings                      → 设置根（compact 显示列表，desktop 重定向到 general）
/settings/[section]            → 设置 section 页
/settings/projects             → 项目列表
/settings/projects/[key]       → 项目详情
/settings/hosts/[serverId]     → 主机设置
/new                           → 创建工作区
/open-project                  → 打开项目 / 首页
/sessions                      → 历史会话
/pair-scan                     → 扫码配对
/h/[serverId]                  → 主机路由组
  /index                       → 重定向到工作区
  /workspace/[workspaceId]     → 工作区详情（含 Tab/Agent/Terminal/Browser）
  /agent/[agentId]             → 智能体详情
  /sessions                    → 主机级会话列表
  /open-project                → 主机级项目打开
  /settings                    → 主机级设置
```

## 主题系统

6 种主题：`light / dark(默认) / zinc / midnight / claude / ghostty`

- 层级式表面色 (surface0~surface4)
- 语义色 (accent/destructive/success/foreground/muted/border)
- 设计 Token：间距 4px 基准 / 字号 12-34px / 圆角 2-9999px / 图标 12-20px

---

> 每个子文档都包含完整的代码路径引用、接口签名、配置参数和交互逻辑，
> 根据这些文档可以完整复刻移动端 App 的全部页面 UI、交互和功能逻辑。
