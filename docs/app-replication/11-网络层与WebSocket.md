# 11 - 网络层与 WebSocket

## 11.1 架构总览

网络层由 `runtime/host-runtime.ts` 管理，核心是一个连接状态机 + 主机注册表。每个连接到 Paseo daemon 的实例称为一个 **Host**。

## 11.2 HostRuntime

**文件**: `src/runtime/host-runtime.ts`

**核心 Store**:
```typescript
interface HostRuntimeStore {
  // 主机注册表
  hosts: HostProfile[];
  // 连接状态（keyed by serverId）
  connections: Record<string, HostConnectionState>;
  // Agent 目录
  agentDirectories: Record<string, AgentDirectoryState>;
  // 监听器
  listeners: Set<() => void>;
  hostListListeners: Set<() => void>;
}
```

**HostProfile**:
```typescript
interface HostProfile {
  serverId: string;
  label: string | null;
  connections: HostConnectionProfile[];
}

interface HostConnectionProfile {
  type: "directTcp" | "directSocket" | "directPipe" | "relay";
  endpoint?: string;
  relayEndpoint?: string;
  useTls?: boolean;
  daemonPublicKeyB64?: string;
}
```

## 11.3 连接状态机

```typescript
type ConnectionStatus = "idle" | "connecting" | "online" | "offline" | "error";

interface HostConnectionState {
  status: ConnectionStatus;
  lastError: string | null;
  client: DaemonClient | null;
  generation: number;                 // 防止竞态
}
```

**连接生命周期**:
1. `runProbeCycle(serverId)` — 探测所有可用连接方式
2. `connect(serverId)` — 建立 WebSocket 连接
3. `disconnect(serverId)` — 断开连接
4. 自动重连（指数退避）

## 11.4 DaemonClient（WebSocket 包装）

**文件**: `packages/client/`（monorepo workspace）

```typescript
interface DaemonClient {
  // 连接
  connect(): Promise<void>;
  close(): Promise<void>;
  
  // 消息发送
  send(message: ClientMessage): void;
  sendStream(chunk: Uint8Array): void;
  
  // 事件订阅
  onMessage(handler: (msg: ServerMessage) => void): () => void;
  onStatusChange(handler: (status: ConnectionStatus) => void): () => void;
  
  // 请求-响应
  request<T>(message: ClientMessage, timeout?: number): Promise<T>;
  
  // Agent 流
  createAgentStream(config: AgentStreamConfig): AgentStream;
  
  // Workspace
  createWorkspace(input: CreateWorkspaceInput): Promise<CreateWorkspaceResult>;
  createPaseoWorktree(input: CreatePaseoWorktreeInput): Promise<CreateWorktreeResult>;
  
  // 文件系统
  readFile(path: string): Promise<FileContent>;
  writeFile(path: string, content: string): Promise<void>;
  listDirectory(path: string): Promise<DirectoryEntry[]>;
  
  // Git
  getCheckoutStatus(): Promise<CheckoutStatusPayload>;
  getBranches(): Promise<Branch[]>;
  getDiff(): Promise<DiffResult>;
  
  // 终端
  createTerminal(config: TerminalConfig): Promise<TerminalSession>;
  destroyTerminal(terminalId: string): Promise<void>;
  
  // Provider
  getProviders(): Promise<Provider[]>;
  getProviderUsage(): Promise<ProviderUsage[]>;
  
  // Agent 历史
  getAgentHistory(cursor?: string, limit?: number): Promise<AgentHistoryResult>;
}
```

## 11.5 Agent Directory

**文件**: `src/runtime/host-runtime.ts`

```typescript
type AgentDirectoryStatus =
  | "initial_loading"
  | "revalidating"
  | "ready"
  | "error_not_found"
  | "error_gateway"
  | "error_server";

interface AgentDirectoryState {
  status: AgentDirectoryStatus;
  agents: AgentSummary[];
  pagination: { cursor: string | null; hasMore: boolean };
}
```

**加载策略**:
- 首次连接后延迟加载
- lazy loading + cursor-based 分页
- 订阅变更（长轮询或 WebSocket 推送）
- generation 防竞态

## 11.6 主机发现与配对

**连接配置类型**:
| 类型 | 说明 | 适用场景 |
|------|------|---------|
| `directTcp` | 直连 TCP | 局域网 |
| `directSocket` | WebSocket 直连 | 局域网/本地 |
| `directPipe` | 命名管道 | Windows 本地 |
| `relay` | 中继服务器 | 远程连接 |

**扫码配对流程** (`pair-scan.tsx`):
1. 相机扫描 QR 码 → 提取 `#offer=` URL
2. 解码 Base64 payload → `ConnectionOfferSchema.parse()`
3. 测试连接: `connectToDaemon({ id, type, relayEndpoint, useTls, daemonPublicKeyB64 }, { serverId })`
4. 持久化: `upsertConnectionFromOfferUrl(offerUrl)`

**配对链接解析**:
```typescript
// src/utils/daemon-endpoints.ts
function decodeOfferFragmentPayload(encoded: string): unknown;
function normalizeHostPort(host: string): string;
```

## 11.7 中继协议

**文件**: `packages/protocol/` — `connection-offer.ts`

```typescript
// ConnectionOfferSchema: Zod schema 验证
{
  serverId: string;
  relay: {
    endpoint: string;
    useTls: boolean;
  };
  daemonPublicKeyB64: string;
}
```

## 11.8 HostRuntime Hook API

```typescript
// 在组件中使用
useHosts() → HostProfile[]                          // 所有主机
useHostRuntimeClient(serverId) → DaemonClient | null
useHostRuntimeIsConnected(serverId) → boolean
useHostRuntimeSnapshot(serverId) → HostConnectionState | null
useHostRuntimeConnectionStatuses(ids) → Map<string, ConnectionStatus>
useHostRegistryLoaded() → boolean
useHostRegistryStatus() → AgentDirectoryStatus
useHostMutations() → { upsertConnectionFromOfferUrl, removeHost }
useHostFeature(serverId, feature) → boolean | null
useHostFeatureMap(ids, feature) → Map<string, boolean>
```

## 11.9 Daemon 启动服务

**文件**: `src/runtime/daemon-start-service.ts`

```typescript
interface DaemonStartService {
  start(): void;
  stop(): void;
  isRunning(): boolean;
  getLastError(): string | null;
  subscribe(listener): () => void;
  recordError(message: string): void;
}
```

**启动策略**:
- Desktop: `shouldStartBuiltInDaemon()` → 管理内置 daemon 进程
- Mobile/Web: 连接到外部 daemon

## 11.10 WebSocket 消息流

```
Client → Server:
  { type: "user_message", text, attachments, agentId }
  { type: "dictation_chunk", audio: ArrayBuffer }
  { type: "voice_chunk", audio: ArrayBuffer }
  { type: "cancel" }
  { type: "get_agent_history", cursor?, limit? }
  { type: "create_workspace", source, ... }
  { type: "terminal_input", terminalId, data }

Server → Client:
  { type: "agent_stream", agentId, delta, ... }
  { type: "tool_call", name, args, status, result }
  { type: "agent_status", agentId, status }
  { type: "audio_output", chunks: ArrayBuffer[] }
  { type: "transcript", text, isFinal }
  { type: "error", message }
  { type: "agent_directory_update", agents }
  { type: "workspace_update", workspace }
```
