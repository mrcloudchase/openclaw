# Agentic Framework Specification

> A comprehensive blueprint for building an autonomous personal assistant framework, distilled from the OpenClaw codebase.

**Version:** 1.0  
**Date:** January 2026  
**Status:** Reference Specification

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview](#2-architecture-overview)
3. [Core Components](#3-core-components)
   - 3.1 [Gateway (Control Plane)](#31-gateway-control-plane)
   - 3.2 [Agent Runtime](#32-agent-runtime)
   - 3.3 [Session Manager](#33-session-manager)
   - 3.4 [Tool Registry](#34-tool-registry)
   - 3.5 [Memory System](#35-memory-system)
   - 3.6 [System Prompt Builder](#36-system-prompt-builder)
   - 3.7 [Channel Adapters](#37-channel-adapters)
   - 3.8 [Skill Loader](#38-skill-loader)
   - 3.9 [Plugin System](#39-plugin-system)
4. [Execution Flow](#4-execution-flow)
5. [Data Models](#5-data-models)
6. [API Specifications](#6-api-specifications)
7. [Developer Experience](#7-developer-experience)
8. [Security Model](#8-security-model)
9. [Implementation Roadmap](#9-implementation-roadmap)
10. [Appendices](#10-appendices)

---

## 1. Executive Summary

This specification defines an **agentic framework** for building autonomous personal assistants that can:

- Maintain persistent identity and memory across sessions
- Execute tools to take real-world actions
- Communicate across multiple messaging platforms
- Learn and adapt through skill acquisition
- Collaborate with users as genuine partners

### Design Philosophy

The framework follows these core principles:

| Principle | Description |
|-----------|-------------|
| **Tool-Calling Loop** | The LLM decides what actions to take; tools provide the capabilities |
| **Persistent Identity** | Bootstrap files define the agent's personality, injected every turn |
| **Memory via Files** | Markdown files + vector search; no opaque databases |
| **Streaming First** | All long-running operations use async iterables |
| **Policy-Based Security** | Tool access controlled by allowlists per session context |
| **Channel Abstraction** | Unified interface for all messaging surfaces |
| **Extensibility** | Plugins and skills extend capabilities without core changes |

---

## 2. Architecture Overview

### 2.1 High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              AGENTIC FRAMEWORK                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         GATEWAY (Control Plane)                      │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │    │
│  │  │ WebSocket│ │   HTTP   │ │ Channel  │ │  Cron    │ │ Webhook  │  │    │
│  │  │  Server  │ │  Server  │ │ Manager  │ │ Scheduler│ │ Handler  │  │    │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘  │    │
│  │       │            │            │            │            │         │    │
│  │       └────────────┴────────────┴────────────┴────────────┘         │    │
│  │                                  │                                   │    │
│  └──────────────────────────────────┼───────────────────────────────────┘    │
│                                     │                                        │
│                                     ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                        MESSAGE DISPATCHER                            │    │
│  │  • Route inbound messages to agent                                   │    │
│  │  • Apply media/link understanding                                    │    │
│  │  • Handle commands (/status, /reset, etc.)                          │    │
│  │  • Resolve session and directives                                    │    │
│  └──────────────────────────────────┬───────────────────────────────────┘    │
│                                     │                                        │
│                                     ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                          AGENT RUNTIME                               │    │
│  │  ┌─────────────────────────────────────────────────────────────┐   │    │
│  │  │                  SYSTEM PROMPT BUILDER                       │   │    │
│  │  │  Identity + Tools + Skills + Memory + Context + Runtime      │   │    │
│  │  └──────────────────────────┬──────────────────────────────────┘   │    │
│  │                             │                                       │    │
│  │  ┌──────────────────────────▼──────────────────────────────────┐   │    │
│  │  │                    PROMPT-TOOL LOOP                          │   │    │
│  │  │                                                              │   │    │
│  │  │   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │   │    │
│  │  │   │ Assemble│    │  Call   │    │ Execute │    │ Stream  │ │   │    │
│  │  │   │ Prompt  │───▶│   LLM   │───▶│  Tools  │───▶│ Response│ │   │    │
│  │  │   │         │    │         │    │         │    │         │ │   │    │
│  │  │   └─────────┘    └────┬────┘    └────┬────┘    └─────────┘ │   │    │
│  │  │        ▲              │              │                      │   │    │
│  │  │        │              │              │                      │   │    │
│  │  │        └──────────────┴──────────────┘                      │   │    │
│  │  │                    (iterate until done)                      │   │    │
│  │  └──────────────────────────────────────────────────────────────┘   │    │
│  └──────────────────────────────────┬───────────────────────────────────┘    │
│                                     │                                        │
│      ┌──────────────────────────────┼──────────────────────────────────┐     │
│      │                              │                                   │     │
│      ▼                              ▼                                   ▼     │
│  ┌────────────┐            ┌────────────────┐                  ┌────────────┐│
│  │  SESSION   │            │    MEMORY      │                  │   TOOL     ││
│  │  MANAGER   │            │    SYSTEM      │                  │  REGISTRY  ││
│  │            │            │                │                  │            ││
│  │ • History  │            │ • Vector Index │                  │ • exec     ││
│  │ • Compaction│           │ • BM25 Search  │                  │ • read     ││
│  │ • JSONL    │            │ • Embeddings   │                  │ • write    ││
│  │ • Locking  │            │ • Markdown     │                  │ • browser  ││
│  └────────────┘            └────────────────┘                  │ • cron     ││
│                                                                │ • message  ││
│                                                                │ • etc...   ││
│                                                                └────────────┘│
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         EXTENSION LAYER                              │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │    │
│  │  │   CHANNELS   │  │    SKILLS    │  │        PLUGINS           │  │    │
│  │  │              │  │              │  │                          │  │    │
│  │  │ • Telegram   │  │ • github     │  │ • Custom tools           │  │    │
│  │  │ • Discord    │  │ • calendar   │  │ • Lifecycle hooks        │  │    │
│  │  │ • WhatsApp   │  │ • email      │  │ • HTTP routes            │  │    │
│  │  │ • Slack      │  │ • browser    │  │ • Channel extensions     │  │    │
│  │  │ • Signal     │  │ • voice      │  │ • Provider plugins       │  │    │
│  │  │ • WebChat    │  │ • etc...     │  │ • Service workers        │  │    │
│  │  └──────────────┘  └──────────────┘  └──────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Interaction Sequence

```
User Message → Channel Adapter → Gateway → Message Dispatcher
                                              │
                                              ▼
                                    ┌─────────────────┐
                                    │ Session Lookup  │
                                    │ + Directive     │
                                    │   Resolution    │
                                    └────────┬────────┘
                                              │
                                              ▼
                                    ┌─────────────────┐
                                    │ System Prompt   │
                                    │ Assembly        │
                                    │ (Identity +     │
                                    │  Tools + Skills │
                                    │  + Memory)      │
                                    └────────┬────────┘
                                              │
                                              ▼
                           ┌──────────────────────────────────┐
                           │         AGENT RUNTIME LOOP       │
                           │                                  │
                           │   ┌──────────────────────────┐   │
                           │   │                          │   │
                           │   ▼                          │   │
                           │ [Prompt] ──▶ [LLM] ──▶ [Response]│
                           │                │              │  │
                           │                ▼              │  │
                           │         [Tool Calls?]─────────┘  │
                           │                │                 │
                           │                ▼ (if yes)        │
                           │         [Execute Tools]          │
                           │                │                 │
                           │                ▼                 │
                           │         [Append Results]         │
                           │                │                 │
                           │                ▼                 │
                           │         [Continue Loop]          │
                           │                                  │
                           └──────────────────────────────────┘
                                              │
                                              ▼
                                    ┌─────────────────┐
                                    │ Session Update  │
                                    │ (Persist        │
                                    │  Transcript)    │
                                    └────────┬────────┘
                                              │
                                              ▼
                                    ┌─────────────────┐
                                    │ Reply Dispatch  │
                                    │ (Chunking,      │
                                    │  Streaming)     │
                                    └────────┬────────┘
                                              │
                                              ▼
                              Channel Adapter → User
```

---

## 3. Core Components

### 3.1 Gateway (Control Plane)

The Gateway is the central orchestrator that manages all connections and coordinates the agent lifecycle.

#### Responsibilities

- Accept WebSocket connections from clients, channels, and nodes
- Route inbound messages to the appropriate handler
- Manage channel lifecycles (connect/disconnect)
- Handle authentication and pairing
- Serve HTTP endpoints (Control UI, webhooks)
- Coordinate cron jobs and heartbeats
- Emit events for observability

#### Interface Definition

```typescript
interface Gateway {
  // Lifecycle
  start(port: number, options?: GatewayOptions): Promise<void>;
  stop(reason?: string): Promise<void>;
  
  // Connection management
  onConnection(handler: ConnectionHandler): void;
  broadcast(event: GatewayEvent): void;
  
  // Channel management
  registerChannel(channel: ChannelAdapter): void;
  getChannel(id: string): ChannelAdapter | undefined;
  listChannels(): ChannelAdapter[];
  
  // Agent invocation
  invokeAgent(params: AgentInvokeParams): Promise<AgentResult>;
  abortAgent(runId: string): void;
  
  // Health and status
  getHealth(): HealthSnapshot;
  getPresence(): PresenceSnapshot;
}

interface GatewayOptions {
  bind: 'loopback' | 'lan' | 'tailnet';
  auth?: AuthConfig;
  tls?: TlsConfig;
  controlUi?: boolean;
}

interface GatewayEvent {
  type: 'agent' | 'chat' | 'presence' | 'health' | 'heartbeat';
  payload: unknown;
  seq?: number;
}
```

#### Wire Protocol

```
Client                           Gateway
  │                                 │
  │─── connect { auth, role } ────▶│
  │◀── res { ok, snapshot } ───────│
  │                                 │
  │─── req { method, params } ────▶│
  │◀── res { ok, payload } ────────│
  │                                 │
  │◀── event { type, payload } ────│
  │◀── event { type, payload } ────│
  │                                 │
```

### 3.2 Agent Runtime

The Agent Runtime executes the prompt-tool loop that creates agentic behavior.

#### Core Loop Implementation

```typescript
interface AgentRuntime {
  // Execute a complete agent turn
  run(params: AgentRunParams): AsyncIterable<AgentEvent>;
  
  // Inject a message mid-stream (steering)
  steer(text: string): Promise<void>;
  
  // Abort the current run
  abort(reason?: string): void;
  
  // Check run status
  isStreaming(): boolean;
  isCompacting(): boolean;
}

interface AgentRunParams {
  // Identity
  sessionId: string;
  sessionKey: string;
  
  // Input
  prompt: string;
  images?: ImageContent[];
  
  // Model
  provider: string;
  model: string;
  thinkLevel?: 'off' | 'minimal' | 'low' | 'medium' | 'high';
  
  // Context
  systemPrompt: string;
  history: Message[];
  tools: Tool[];
  
  // Options
  timeoutMs?: number;
  abortSignal?: AbortSignal;
}

type AgentEvent =
  | { type: 'lifecycle'; phase: 'start' | 'end' | 'error'; data?: unknown }
  | { type: 'assistant'; delta: string; reasoning?: string }
  | { type: 'tool_start'; callId: string; name: string; params: object }
  | { type: 'tool_progress'; callId: string; output: string }
  | { type: 'tool_end'; callId: string; result: unknown; error?: string }
  | { type: 'compaction'; stage: 'start' | 'end'; summary?: string };
```

#### Execution Algorithm

```
function runAgent(params):
  messages = loadHistory(params.sessionId)
  systemPrompt = buildSystemPrompt(params)
  
  while not aborted:
    // 1. Call the LLM
    response = await streamLLM({
      system: systemPrompt,
      messages: messages,
      tools: params.tools,
    })
    
    // 2. Emit assistant content
    for delta in response.textDeltas:
      yield { type: 'assistant', delta }
    
    // 3. Check for tool calls
    if response.toolCalls.length == 0:
      break  // Done - no more tools to execute
    
    // 4. Execute tools
    for toolCall in response.toolCalls:
      yield { type: 'tool_start', callId, name, params }
      
      result = await executeTool(toolCall)
      
      yield { type: 'tool_end', callId, result }
      
      // 4a. Check for steering messages
      if hasPendingSteer():
        injectSteerMessage()
        skipRemainingTools()
    
    // 5. Append results to messages
    messages.append(assistantMessage)
    messages.append(toolResults)
    
    // 6. Check for context overflow
    if estimateTokens(messages) > contextLimit:
      messages = await compact(messages)
      yield { type: 'compaction', stage: 'end' }
  
  // 7. Persist final state
  saveHistory(params.sessionId, messages)
  
  yield { type: 'lifecycle', phase: 'end' }
```

### 3.3 Session Manager

The Session Manager handles conversation history persistence and compaction.

#### Interface Definition

```typescript
interface SessionManager {
  // Open/create a session
  open(sessionFile: string): Session;
  
  // History operations
  getMessages(): Message[];
  getLeafEntry(): SessionEntry | null;
  append(message: Message): void;
  branch(entryId: string): void;
  resetLeaf(): void;
  
  // Metadata
  getSessionContext(): SessionContext;
  
  // Persistence
  flush(): void;
  close(): void;
}

interface Session {
  sessionId: string;
  messages: Message[];
  metadata: SessionMetadata;
}

interface SessionMetadata {
  createdAt: number;
  updatedAt: number;
  model?: string;
  thinkLevel?: string;
  verboseLevel?: string;
  elevated?: boolean;
  compactedAt?: number;
}

interface Message {
  role: 'user' | 'assistant' | 'tool_result';
  content: string | ContentBlock[];
  
  // For assistant messages
  toolCalls?: ToolCall[];
  reasoning?: string;
  usage?: TokenUsage;
  
  // For tool results
  toolCallId?: string;
  toolName?: string;
  isError?: boolean;
}
```

#### Storage Format (JSONL)

```jsonl
{"type":"message","id":"msg_001","parentId":null,"message":{"role":"user","content":"Hello"}}
{"type":"message","id":"msg_002","parentId":"msg_001","message":{"role":"assistant","content":"Hi there!"}}
{"type":"message","id":"msg_003","parentId":"msg_002","message":{"role":"user","content":"What's 2+2?"}}
{"type":"message","id":"msg_004","parentId":"msg_003","message":{"role":"assistant","content":"4","toolCalls":[...]}}
{"type":"message","id":"msg_005","parentId":"msg_004","message":{"role":"tool_result","toolCallId":"call_1","content":"4"}}
```

#### Compaction Strategy

```typescript
interface CompactionConfig {
  // Token threshold to trigger compaction
  reserveTokensFloor: number;  // default: 20000
  
  // Pre-compaction memory flush
  memoryFlush?: {
    enabled: boolean;
    softThresholdTokens: number;
    prompt: string;
  };
  
  // Summary model (can differ from main model)
  summaryModel?: string;
}

async function compactSession(
  messages: Message[],
  config: CompactionConfig
): Promise<Message[]> {
  // 1. Optional: Run memory flush turn
  if (config.memoryFlush?.enabled) {
    await runMemoryFlush(messages, config.memoryFlush.prompt);
  }
  
  // 2. Generate summary of older messages
  const oldMessages = messages.slice(0, -recentCount);
  const summary = await summarize(oldMessages, config.summaryModel);
  
  // 3. Return compacted history
  return [
    { role: 'system', content: `Previous context summary:\n${summary}` },
    ...messages.slice(-recentCount)
  ];
}
```

### 3.4 Tool Registry

The Tool Registry manages available tools and enforces access policies.

#### Interface Definition

```typescript
interface ToolRegistry {
  // Registration
  register(tool: Tool): void;
  registerMany(tools: Tool[]): void;
  
  // Lookup
  get(name: string): Tool | undefined;
  list(): Tool[];
  
  // Policy filtering
  filter(policy: ToolPolicy): Tool[];
  isAllowed(name: string, policy: ToolPolicy): boolean;
}

interface Tool<TParams = unknown, TResult = unknown> {
  name: string;
  description: string;
  
  // JSON Schema for parameters
  parameters: JSONSchema;
  
  // Execution
  execute(callId: string, params: TParams, ctx: ToolContext): Promise<TResult>;
  
  // Optional metadata
  label?: string;
  group?: string;
  dangerous?: boolean;
}

interface ToolContext {
  sessionKey: string;
  workspaceDir: string;
  agentId?: string;
  abortSignal?: AbortSignal;
  sandbox?: SandboxContext;
}

interface ToolPolicy {
  allow?: string[];   // Allowlist (if set, only these are allowed)
  deny?: string[];    // Denylist (these are always blocked)
  groups?: string[];  // Allow tools from these groups
}
```

#### Built-in Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `read` | Read file contents | `{ path: string, offset?: number, limit?: number }` |
| `write` | Create/overwrite file | `{ path: string, content: string }` |
| `edit` | Edit file with search/replace | `{ path: string, old: string, new: string }` |
| `exec` | Run shell command | `{ command: string, cwd?: string, timeout?: number }` |
| `process` | Manage background processes | `{ action: 'list' \| 'kill', pid?: number }` |
| `web_search` | Search the web | `{ query: string, maxResults?: number }` |
| `web_fetch` | Fetch URL content | `{ url: string, mode?: 'markdown' \| 'text' }` |
| `browser` | Control browser | `{ action: string, ... }` |
| `cron` | Schedule tasks | `{ action: 'add' \| 'list' \| 'remove', ... }` |
| `message` | Send messages | `{ channel: string, to: string, content: string }` |
| `memory_search` | Search memory files | `{ query: string, maxResults?: number }` |
| `memory_get` | Read memory file | `{ path: string, from?: number, lines?: number }` |
| `session_status` | Get session info | `{}` |
| `sessions_list` | List other sessions | `{ filter?: string }` |
| `sessions_send` | Message another session | `{ sessionKey: string, message: string }` |
| `sessions_spawn` | Spawn sub-agent | `{ agentId: string, message: string }` |

#### Tool Implementation Example

```typescript
const webSearchTool: Tool<WebSearchParams, WebSearchResult> = {
  name: 'web_search',
  description: 'Search the web using Brave Search API',
  parameters: {
    type: 'object',
    properties: {
      query: { type: 'string', description: 'Search query' },
      maxResults: { type: 'number', description: 'Max results (1-20)', default: 10 }
    },
    required: ['query']
  },
  
  async execute(callId, params, ctx) {
    const { query, maxResults = 10 } = params;
    
    // Check abort signal
    if (ctx.abortSignal?.aborted) {
      throw new Error('Search aborted');
    }
    
    // Call Brave Search API
    const response = await fetch('https://api.search.brave.com/res/v1/web/search', {
      headers: { 'X-Subscription-Token': process.env.BRAVE_API_KEY },
      body: JSON.stringify({ q: query, count: maxResults })
    });
    
    const data = await response.json();
    
    return {
      results: data.web.results.map(r => ({
        title: r.title,
        url: r.url,
        snippet: r.description
      }))
    };
  }
};
```

### 3.5 Memory System

The Memory System provides persistent, searchable memory through markdown files and vector embeddings.

#### Interface Definition

```typescript
interface MemoryManager {
  // Indexing
  index(paths: string[]): Promise<void>;
  reindex(): Promise<void>;
  
  // Search
  search(query: string, options?: SearchOptions): Promise<SearchResult[]>;
  
  // Direct read
  readFile(path: string, options?: ReadOptions): Promise<FileContent>;
  
  // Status
  status(): MemoryStatus;
  
  // Lifecycle
  startWatcher(): void;
  stopWatcher(): void;
  close(): void;
}

interface SearchOptions {
  maxResults?: number;      // default: 10
  minScore?: number;        // default: 0.5
  sources?: MemorySource[]; // 'memory' | 'sessions'
  hybrid?: {
    vectorWeight: number;   // default: 0.7
    textWeight: number;     // default: 0.3
  };
}

interface SearchResult {
  path: string;
  snippet: string;
  score: number;
  startLine: number;
  endLine: number;
  source: 'memory' | 'sessions';
}

interface MemoryStatus {
  provider: 'openai' | 'gemini' | 'local';
  model: string;
  indexedFiles: number;
  indexedChunks: number;
  lastSyncAt: number;
}
```

#### Memory File Layout

```
~/.agent/workspace/
├── MEMORY.md           # Long-term curated memory (main session only)
├── memory/
│   ├── 2026-01-30.md   # Daily log
│   ├── 2026-01-31.md   # Daily log
│   └── heartbeat-state.json
├── AGENTS.md           # Operating instructions
├── SOUL.md             # Persona and personality
├── USER.md             # User profile
├── TOOLS.md            # Tool-specific notes
└── skills/
    └── ...             # Workspace skills
```

#### Chunking Algorithm

```typescript
function chunkMarkdown(
  content: string,
  options: ChunkOptions = {}
): Chunk[] {
  const {
    targetTokens = 400,
    overlapTokens = 80,
    minChunkTokens = 50
  } = options;
  
  const chunks: Chunk[] = [];
  const lines = content.split('\n');
  let currentChunk: string[] = [];
  let currentTokens = 0;
  let startLine = 1;
  
  for (let i = 0; i < lines.length; i++) {
    const line = lines[i];
    const lineTokens = estimateTokens(line);
    
    if (currentTokens + lineTokens > targetTokens && currentTokens >= minChunkTokens) {
      // Save current chunk
      chunks.push({
        text: currentChunk.join('\n'),
        startLine,
        endLine: startLine + currentChunk.length - 1
      });
      
      // Start new chunk with overlap
      const overlapLines = getOverlapLines(currentChunk, overlapTokens);
      currentChunk = [...overlapLines, line];
      currentTokens = estimateTokens(currentChunk.join('\n'));
      startLine = i - overlapLines.length + 1;
    } else {
      currentChunk.push(line);
      currentTokens += lineTokens;
    }
  }
  
  // Don't forget the last chunk
  if (currentChunk.length > 0) {
    chunks.push({
      text: currentChunk.join('\n'),
      startLine,
      endLine: startLine + currentChunk.length - 1
    });
  }
  
  return chunks;
}
```

#### Hybrid Search Implementation

```typescript
async function hybridSearch(
  query: string,
  options: SearchOptions
): Promise<SearchResult[]> {
  const { vectorWeight = 0.7, textWeight = 0.3 } = options.hybrid ?? {};
  const candidateMultiplier = 4;
  const targetResults = options.maxResults ?? 10;
  
  // 1. Vector search (semantic)
  const queryEmbedding = await embed(query);
  const vectorResults = await searchByVector(
    queryEmbedding,
    targetResults * candidateMultiplier
  );
  
  // 2. BM25 search (keyword)
  const ftsQuery = buildFtsQuery(query);
  const bm25Results = await searchByBM25(
    ftsQuery,
    targetResults * candidateMultiplier
  );
  
  // 3. Merge results
  const candidates = new Map<string, {
    chunk: Chunk;
    vectorScore: number;
    textScore: number;
  }>();
  
  for (const r of vectorResults) {
    candidates.set(r.chunkId, {
      chunk: r.chunk,
      vectorScore: r.score,
      textScore: 0
    });
  }
  
  for (const r of bm25Results) {
    const existing = candidates.get(r.chunkId);
    const textScore = 1 / (1 + Math.max(0, r.bm25Rank));
    if (existing) {
      existing.textScore = textScore;
    } else {
      candidates.set(r.chunkId, {
        chunk: r.chunk,
        vectorScore: 0,
        textScore
      });
    }
  }
  
  // 4. Compute final scores and rank
  const results = Array.from(candidates.values())
    .map(c => ({
      ...c.chunk,
      score: vectorWeight * c.vectorScore + textWeight * c.textScore
    }))
    .filter(r => r.score >= (options.minScore ?? 0))
    .sort((a, b) => b.score - a.score)
    .slice(0, targetResults);
  
  return results;
}
```

### 3.6 System Prompt Builder

The System Prompt Builder assembles the agent's identity and context for each turn.

#### Interface Definition

```typescript
interface SystemPromptBuilder {
  build(params: PromptParams): string;
}

interface PromptParams {
  // Identity
  workspaceDir: string;
  bootstrapFiles: BootstrapFile[];
  
  // Tools
  tools: Tool[];
  
  // Skills
  skillsPrompt?: string;
  
  // Runtime
  runtime: RuntimeInfo;
  
  // Context
  extraPrompt?: string;
  
  // Mode
  promptMode?: 'full' | 'minimal' | 'none';
}

interface BootstrapFile {
  name: string;
  path: string;
  content: string;
  truncated?: boolean;
  missing?: boolean;
}

interface RuntimeInfo {
  agentId?: string;
  host: string;
  os: string;
  arch: string;
  node: string;
  model: string;
  channel?: string;
  capabilities?: string[];
}
```

#### System Prompt Template

```markdown
You are a personal assistant running inside [Framework Name].

## Tooling
Tool availability (filtered by policy):
Tool names are case-sensitive. Call tools exactly as listed.
- read: Read file contents
- write: Create or overwrite files
- edit: Make precise edits to files
- exec: Run shell commands
- web_search: Search the web
- web_fetch: Fetch URL content
- browser: Control web browser
- memory_search: Search memory files
- memory_get: Read memory file content
- cron: Schedule tasks
- message: Send messages
- sessions_send: Message another session

## Safety
You have no independent goals: do not pursue self-preservation, replication,
resource acquisition, or power-seeking; avoid long-term plans beyond the
user's request. Prioritize safety and human oversight over completion.

## Skills (when available)
Before replying: scan <available_skills> entries.
- If one skill clearly applies: read its SKILL.md, then follow it.
- If none clearly apply: do not read any SKILL.md.

<available_skills>
  <skill>
    <name>github</name>
    <description>GitHub CLI operations</description>
    <location>/path/to/skills/github/SKILL.md</location>
  </skill>
</available_skills>

## Memory Recall
Before answering questions about prior work, decisions, dates, people,
preferences, or todos: run memory_search first, then use memory_get
to retrieve relevant sections.

## Workspace
Your working directory is: ~/.agent/workspace

## Workspace Files (injected)
The following bootstrap files are included below in Project Context.

## Current Date & Time
Time zone: America/Los_Angeles

## Runtime
Runtime: agent=main | host=MacBook-Pro | os=Darwin 24.1.0 (arm64) | 
node=v22.1.0 | model=anthropic/claude-sonnet-4-20250514 | thinking=off

# Project Context

## SOUL.md

[Content of SOUL.md injected here]

## AGENTS.md

[Content of AGENTS.md injected here]

## USER.md

[Content of USER.md injected here]

## Silent Replies
When you have nothing to say, respond with ONLY: NO_REPLY

## Heartbeats
If you receive a heartbeat poll and nothing needs attention, reply exactly:
HEARTBEAT_OK
```

### 3.7 Channel Adapters

Channel Adapters provide a unified interface for different messaging platforms.

#### Interface Definition

```typescript
interface ChannelAdapter {
  // Identity
  id: string;
  name: string;
  
  // Lifecycle
  connect(): Promise<void>;
  disconnect(): Promise<void>;
  isConnected(): boolean;
  
  // Inbound
  onMessage(handler: InboundHandler): Unsubscribe;
  
  // Outbound
  send(params: SendParams): Promise<SendResult>;
  
  // Actions
  react?(messageId: string, emoji: string): Promise<void>;
  edit?(messageId: string, content: string): Promise<void>;
  delete?(messageId: string): Promise<void>;
  
  // Status
  getStatus(): ChannelStatus;
}

interface InboundMessage {
  id: string;
  from: string;
  fromName?: string;
  content: string;
  images?: ImageContent[];
  audio?: AudioContent;
  video?: VideoContent;
  
  // Context
  channel: string;
  accountId?: string;
  groupId?: string;
  groupName?: string;
  threadId?: string;
  replyToId?: string;
  
  // Metadata
  timestamp: number;
  isGroup: boolean;
  isMention: boolean;
  isReply: boolean;
}

interface SendParams {
  to: string;
  content: string;
  
  // Optional rich content
  images?: ImageContent[];
  buttons?: InlineButton[];
  
  // Threading
  replyTo?: string;
  threadId?: string;
}

interface SendResult {
  messageId: string;
  timestamp: number;
  error?: string;
}
```

#### Channel Implementation Example (Telegram)

```typescript
class TelegramChannel implements ChannelAdapter {
  id = 'telegram';
  name = 'Telegram';
  
  private bot: TelegramBot;
  private handlers: Set<InboundHandler> = new Set();
  
  constructor(private config: TelegramConfig) {
    this.bot = new TelegramBot(config.token);
  }
  
  async connect(): Promise<void> {
    // Set up message handler
    this.bot.on('message', async (msg) => {
      const inbound = this.parseMessage(msg);
      for (const handler of this.handlers) {
        await handler(inbound);
      }
    });
    
    // Start polling
    await this.bot.startPolling();
  }
  
  async disconnect(): Promise<void> {
    await this.bot.stopPolling();
  }
  
  isConnected(): boolean {
    return this.bot.isPolling();
  }
  
  onMessage(handler: InboundHandler): Unsubscribe {
    this.handlers.add(handler);
    return () => this.handlers.delete(handler);
  }
  
  async send(params: SendParams): Promise<SendResult> {
    const options: any = {};
    
    if (params.replyTo) {
      options.reply_to_message_id = params.replyTo;
    }
    
    if (params.buttons) {
      options.reply_markup = {
        inline_keyboard: [params.buttons.map(b => ({
          text: b.text,
          callback_data: b.callbackData
        }))]
      };
    }
    
    const result = await this.bot.sendMessage(
      params.to,
      params.content,
      options
    );
    
    return {
      messageId: String(result.message_id),
      timestamp: result.date * 1000
    };
  }
  
  async react(messageId: string, emoji: string): Promise<void> {
    await this.bot.setMessageReaction(
      this.config.chatId,
      Number(messageId),
      [{ type: 'emoji', emoji }]
    );
  }
  
  private parseMessage(msg: TelegramMessage): InboundMessage {
    return {
      id: String(msg.message_id),
      from: String(msg.from?.id),
      fromName: msg.from?.first_name,
      content: msg.text ?? '',
      channel: 'telegram',
      groupId: msg.chat.type !== 'private' ? String(msg.chat.id) : undefined,
      groupName: msg.chat.title,
      timestamp: msg.date * 1000,
      isGroup: msg.chat.type !== 'private',
      isMention: msg.text?.includes(`@${this.bot.username}`) ?? false,
      isReply: !!msg.reply_to_message
    };
  }
}
```

### 3.8 Skill Loader

The Skill Loader discovers and loads skill definitions from the filesystem.

#### Interface Definition

```typescript
interface SkillLoader {
  // Load skills from directories
  load(options: LoadOptions): SkillSnapshot;
  
  // Format for system prompt
  formatForPrompt(skills: Skill[]): string;
  
  // Refresh skills (e.g., after file changes)
  refresh(): SkillSnapshot;
}

interface LoadOptions {
  workspaceDir: string;
  managedDir?: string;   // ~/.agent/skills
  bundledDir?: string;   // Framework bundled skills
  extraDirs?: string[];  // Additional directories
  filter?: string[];     // Only load these skills
}

interface Skill {
  name: string;
  description: string;
  filePath: string;
  baseDir: string;
  
  // From frontmatter
  frontmatter: SkillFrontmatter;
  
  // Resolved metadata
  metadata: SkillMetadata;
  invocation: InvocationPolicy;
}

interface SkillFrontmatter {
  name?: string;
  description?: string;
  env?: Record<string, string>;
  'primary-env'?: string;
  'user-invocable'?: boolean;
  'disable-model-invocation'?: boolean;
  'command-dispatch'?: 'tool';
  'command-tool'?: string;
}

interface SkillSnapshot {
  prompt: string;       // Formatted for system prompt
  skills: SkillInfo[];  // Minimal info for tracking
}
```

#### Skill File Format (SKILL.md)

```markdown
---
name: github
description: GitHub CLI operations - repos, issues, PRs, workflows
primary-env: GH_TOKEN
user-invocable: true
---

# GitHub Skill

Use the `gh` CLI to interact with GitHub.

## Authentication

Requires `GH_TOKEN` environment variable or `gh auth login`.

## Common Operations

### List repositories
```bash
gh repo list [owner] --limit 10
```

### Create issue
```bash
gh issue create --title "Bug report" --body "Description"
```

### Create PR
```bash
gh pr create --title "Feature" --body "Description" --base main
```

## Notes

- Always check if authenticated: `gh auth status`
- Use `--json` flag for machine-readable output
- Rate limits apply to API calls
```

#### Skill Loading Algorithm

```typescript
function loadSkills(options: LoadOptions): SkillSnapshot {
  const skillDirs = [
    options.bundledDir,      // Lowest priority
    ...(options.extraDirs ?? []),
    options.managedDir,
    path.join(options.workspaceDir, 'skills')  // Highest priority
  ].filter(Boolean);
  
  const skillsByName = new Map<string, Skill>();
  
  for (const dir of skillDirs) {
    if (!fs.existsSync(dir)) continue;
    
    for (const entry of fs.readdirSync(dir)) {
      const skillDir = path.join(dir, entry);
      const skillFile = path.join(skillDir, 'SKILL.md');
      
      if (!fs.existsSync(skillFile)) continue;
      
      const content = fs.readFileSync(skillFile, 'utf-8');
      const { frontmatter, body } = parseFrontmatter(content);
      
      const skill: Skill = {
        name: frontmatter.name ?? entry,
        description: frontmatter.description ?? extractFirstParagraph(body),
        filePath: skillFile,
        baseDir: skillDir,
        frontmatter,
        metadata: resolveMetadata(frontmatter),
        invocation: resolveInvocation(frontmatter)
      };
      
      // Later directories override earlier ones
      skillsByName.set(skill.name, skill);
    }
  }
  
  // Apply filter if specified
  let skills = Array.from(skillsByName.values());
  if (options.filter?.length) {
    skills = skills.filter(s => options.filter!.includes(s.name));
  }
  
  return {
    prompt: formatSkillsForPrompt(skills),
    skills: skills.map(s => ({ name: s.name, primaryEnv: s.metadata.primaryEnv }))
  };
}

function formatSkillsForPrompt(skills: Skill[]): string {
  if (skills.length === 0) return '';
  
  const entries = skills
    .filter(s => !s.invocation.disableModelInvocation)
    .map(s => `  <skill>
    <name>${s.name}</name>
    <description>${s.description}</description>
    <location>${s.filePath}</location>
  </skill>`)
    .join('\n');
  
  return `<available_skills>\n${entries}\n</available_skills>`;
}
```

### 3.9 Plugin System

The Plugin System enables extensibility through hooks, custom tools, and additional channels.

#### Interface Definition

```typescript
interface PluginApi {
  // Identity
  id: string;
  name: string;
  config: Config;
  
  // Tool registration
  registerTool(tool: Tool | ToolFactory): void;
  
  // Hook registration
  on<K extends HookName>(
    hook: K,
    handler: HookHandler<K>,
    options?: { priority?: number }
  ): void;
  
  // Channel registration
  registerChannel(channel: ChannelAdapter): void;
  
  // HTTP routes
  registerHttpRoute(path: string, handler: HttpHandler): void;
  
  // Services
  registerService(service: PluginService): void;
  
  // Commands
  registerCommand(command: CommandDefinition): void;
  
  // Utilities
  resolvePath(input: string): string;
  logger: Logger;
}

type HookName =
  | 'before_agent_start'
  | 'agent_end'
  | 'before_compaction'
  | 'after_compaction'
  | 'message_received'
  | 'message_sending'
  | 'message_sent'
  | 'before_tool_call'
  | 'after_tool_call'
  | 'tool_result_persist'
  | 'session_start'
  | 'session_end'
  | 'gateway_start'
  | 'gateway_stop';

interface PluginDefinition {
  id: string;
  name?: string;
  description?: string;
  version?: string;
  
  // Configuration schema
  configSchema?: ConfigSchema;
  
  // Initialization
  register?: (api: PluginApi) => void | Promise<void>;
  activate?: (api: PluginApi) => void | Promise<void>;
}
```

#### Plugin Example

```typescript
// plugins/analytics/index.ts
import type { PluginDefinition, PluginApi } from '@agentic/core';

const analyticsPlugin: PluginDefinition = {
  id: 'analytics',
  name: 'Analytics Plugin',
  version: '1.0.0',
  
  async register(api: PluginApi) {
    // Track agent runs
    api.on('agent_end', async (event, ctx) => {
      await trackEvent('agent_run', {
        agentId: ctx.agentId,
        sessionKey: ctx.sessionKey,
        durationMs: event.durationMs,
        success: event.success,
        messageCount: event.messages.length
      });
    });
    
    // Track tool usage
    api.on('after_tool_call', async (event, ctx) => {
      await trackEvent('tool_call', {
        toolName: event.toolName,
        durationMs: event.durationMs,
        error: event.error
      });
    });
    
    // Add analytics dashboard route
    api.registerHttpRoute('/analytics', async (req, res) => {
      const stats = await getAnalyticsStats();
      res.json(stats);
    });
  }
};

export default analyticsPlugin;
```

---

## 4. Execution Flow

### 4.1 Complete Message Processing Flow

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        INBOUND MESSAGE PROCESSING                             │
└──────────────────────────────────────────────────────────────────────────────┘

1. CHANNEL RECEIVES MESSAGE
   │
   ├─▶ Parse raw message into InboundMessage
   ├─▶ Extract media attachments (images, audio, video)
   ├─▶ Resolve sender identity
   └─▶ Emit to Gateway

2. GATEWAY ROUTES MESSAGE
   │
   ├─▶ Run message_received hooks (plugins can intercept)
   ├─▶ Check allowlist / pairing status
   ├─▶ Determine session key from (channel, accountId, chatId)
   └─▶ Dispatch to Message Processor

3. MESSAGE PROCESSOR
   │
   ├─▶ Check for duplicate messages (dedupe)
   ├─▶ Apply media understanding (transcription, OCR)
   ├─▶ Apply link understanding (fetch & summarize URLs)
   └─▶ Forward to Reply Handler

4. REPLY HANDLER
   │
   ├─▶ Load/create session state
   │     └─▶ Session file: ~/.agent/agents/<agentId>/sessions/<sessionId>.jsonl
   │
   ├─▶ Resolve directives from message
   │     └─▶ /think, /verbose, /model, /reset, /status, etc.
   │
   ├─▶ Handle command-only messages (return early if applicable)
   │     └─▶ /status, /reset, /help → Return directly, no agent run
   │
   └─▶ Prepare agent run

5. AGENT RUNTIME
   │
   ├─▶ Acquire session write lock
   │
   ├─▶ Load session history from JSONL
   │
   ├─▶ Sanitize history for provider compatibility
   │     └─▶ Validate turn ordering, image formats, etc.
   │
   ├─▶ Build system prompt
   │     ├─▶ Load bootstrap files (SOUL.md, AGENTS.md, USER.md)
   │     ├─▶ Format available tools
   │     ├─▶ Format available skills
   │     ├─▶ Add runtime context
   │     └─▶ Inject extra context (group intro, etc.)
   │
   ├─▶ Create tool instances with context
   │     └─▶ Apply tool policy filtering
   │
   ├─▶ Run before_agent_start hooks
   │
   ├─▶ ENTER PROMPT-TOOL LOOP
   │     │
   │     ├─▶ Send prompt + history to LLM
   │     │
   │     ├─▶ Stream response
   │     │     ├─▶ Emit assistant deltas (for typing indicators)
   │     │     ├─▶ Emit reasoning if enabled
   │     │     └─▶ Collect tool calls
   │     │
   │     ├─▶ If tool calls present:
   │     │     │
   │     │     ├─▶ Run before_tool_call hooks
   │     │     │
   │     │     ├─▶ Execute each tool
   │     │     │     ├─▶ Emit tool_start event
   │     │     │     ├─▶ Run tool.execute()
   │     │     │     ├─▶ Emit tool_end event
   │     │     │     └─▶ Run after_tool_call hooks
   │     │     │
   │     │     ├─▶ Check for steering messages (inject if present)
   │     │     │
   │     │     ├─▶ Append tool results to messages
   │     │     │
   │     │     └─▶ CONTINUE LOOP (goto "Send prompt")
   │     │
   │     └─▶ If no tool calls: EXIT LOOP
   │
   ├─▶ Check for context overflow
   │     └─▶ If overflow: run compaction, then retry
   │
   ├─▶ Persist final messages to session file
   │
   ├─▶ Run agent_end hooks
   │
   └─▶ Release session write lock

6. REPLY DISPATCH
   │
   ├─▶ Run message_sending hooks (can modify/cancel)
   │
   ├─▶ Apply TTS if enabled
   │
   ├─▶ Chunk message for platform limits
   │     └─▶ WhatsApp: 4096, Telegram: 4096, Discord: 2000, etc.
   │
   ├─▶ Stream or batch send to channel
   │
   ├─▶ Run message_sent hooks
   │
   └─▶ Return result to Gateway
```

### 4.2 Tool Execution Detail

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                           TOOL EXECUTION FLOW                                 │
└──────────────────────────────────────────────────────────────────────────────┘

LLM returns tool_calls: [
  { id: "call_1", name: "web_search", params: { query: "weather NYC" } },
  { id: "call_2", name: "read", params: { path: "notes.md" } }
]

FOR EACH tool_call:
  │
  ├─▶ Validate tool exists in registry
  │     └─▶ If not found: return error result
  │
  ├─▶ Check tool policy
  │     └─▶ If denied: return "tool not available" result
  │
  ├─▶ Validate parameters against schema
  │     └─▶ If invalid: return validation error result
  │
  ├─▶ Run before_tool_call hooks
  │     ├─▶ Can modify params
  │     └─▶ Can block execution
  │
  ├─▶ Create tool context
  │     {
  │       sessionKey: "main:whatsapp:+1234567890",
  │       workspaceDir: "~/.agent/workspace",
  │       agentId: "main",
  │       abortSignal: <signal>,
  │       sandbox: null | { containerName, workspaceDir }
  │     }
  │
  ├─▶ Execute tool
  │     │
  │     ├─▶ If sandboxed: run in Docker container
  │     │
  │     ├─▶ If background (exec with yieldMs): 
  │     │     └─▶ Start process, return immediately with pid
  │     │
  │     └─▶ Normal execution with timeout
  │
  ├─▶ Run after_tool_call hooks
  │     └─▶ Can inspect/log result
  │
  ├─▶ Run tool_result_persist hooks
  │     └─▶ Can transform result before saving
  │
  └─▶ Return tool result message
```

### 4.3 Session Compaction Flow

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SESSION COMPACTION FLOW                               │
└──────────────────────────────────────────────────────────────────────────────┘

Triggered when: estimatedTokens > contextWindow - reserveTokens

1. Run before_compaction hooks

2. Optional: Memory Flush Turn
   │
   ├─▶ If memoryFlush.enabled and approaching threshold:
   │     │
   │     ├─▶ Inject system message: "Session nearing compaction"
   │     ├─▶ Inject user message: "Write durable memories now"
   │     ├─▶ Run agent turn (agent writes to memory files)
   │     └─▶ Agent responds with NO_REPLY (silent)

3. Generate Summary
   │
   ├─▶ Select older messages to compact
   │     └─▶ Keep last N turns intact (recency bias)
   │
   ├─▶ Call summary model:
   │     │
   │     │  System: "Summarize this conversation concisely."
   │     │  User: [older messages as text]
   │     │
   │     └─▶ Returns: 2-3 paragraph summary
   │
   └─▶ Replace old messages with summary message

4. Update session file
   │
   ├─▶ Write compaction marker to JSONL
   └─▶ Update session metadata (compactedAt timestamp)

5. Run after_compaction hooks

6. Retry original prompt with compacted history
```

---

## 5. Data Models

### 5.1 Configuration Schema

```typescript
interface AgentConfig {
  // Model defaults
  agent: {
    model: string;  // "anthropic/claude-sonnet-4-20250514"
    fallbacks?: string[];
  };
  
  // Agent runtime
  agents: {
    defaults: {
      workspace: string;
      timeoutSeconds: number;
      thinkingLevel: ThinkLevel;
      verboseLevel: VerboseLevel;
      reasoningLevel: ReasoningLevel;
      
      // Compaction
      compaction: {
        reserveTokensFloor: number;
        memoryFlush: {
          enabled: boolean;
          softThresholdTokens: number;
          prompt: string;
        };
      };
      
      // Memory search
      memorySearch: {
        enabled: boolean;
        provider: 'openai' | 'gemini' | 'local';
        model: string;
        query: {
          hybrid: {
            enabled: boolean;
            vectorWeight: number;
            textWeight: number;
          };
        };
      };
      
      // Bootstrap
      skipBootstrap: boolean;
      bootstrapMaxChars: number;
      
      // Heartbeat
      heartbeat: {
        enabled: boolean;
        intervalMinutes: number;
        prompt: string;
      };
    };
  };
  
  // Channel configurations
  channels: {
    telegram?: TelegramConfig;
    discord?: DiscordConfig;
    whatsapp?: WhatsAppConfig;
    slack?: SlackConfig;
    signal?: SignalConfig;
  };
  
  // Tool configuration
  tools: {
    exec?: {
      security: 'normal' | 'ask' | 'elevated';
      timeoutSec: number;
      safeBins: string[];
    };
    browser?: {
      enabled: boolean;
    };
  };
  
  // Skills configuration
  skills: {
    load: {
      extraDirs: string[];
    };
  };
  
  // Gateway configuration
  gateway: {
    port: number;
    bind: 'loopback' | 'lan' | 'tailnet';
    auth: {
      mode: 'none' | 'token' | 'password';
      token?: string;
    };
  };
}
```

### 5.2 Session Store Schema

```typescript
interface SessionStore {
  [sessionKey: string]: SessionEntry;
}

interface SessionEntry {
  sessionId: string;
  sessionKey: string;
  
  // Model overrides
  model?: string;
  thinkLevel?: ThinkLevel;
  verboseLevel?: VerboseLevel;
  reasoningLevel?: ReasoningLevel;
  elevated?: boolean;
  
  // State
  systemSent?: boolean;
  compactedAt?: number;
  lastRunId?: string;
  
  // Group-specific
  groupActivation?: 'mention' | 'always';
  groupActivationNeedsSystemIntro?: boolean;
  
  // TTS
  ttsAuto?: 'on' | 'off' | 'audio';
}
```

### 5.3 Message Schema

```typescript
type Message = UserMessage | AssistantMessage | ToolResultMessage;

interface UserMessage {
  role: 'user';
  content: string | ContentBlock[];
}

interface AssistantMessage {
  role: 'assistant';
  content: string | ContentBlock[];
  toolCalls?: ToolCall[];
  
  // Metadata
  reasoning?: string;
  provider?: string;
  model?: string;
  usage?: TokenUsage;
  errorMessage?: string;
}

interface ToolResultMessage {
  role: 'tool_result';
  toolCallId: string;
  toolName?: string;
  content: string;
  isError?: boolean;
}

type ContentBlock =
  | { type: 'text'; text: string }
  | { type: 'image'; data: string; mimeType: string }
  | { type: 'tool_use'; id: string; name: string; input: object }
  | { type: 'tool_result'; toolUseId: string; content: string };

interface ToolCall {
  id: string;
  name: string;
  arguments: string;  // JSON string
}

interface TokenUsage {
  promptTokens: number;
  completionTokens: number;
  totalTokens: number;
  cacheReadTokens?: number;
  cacheWriteTokens?: number;
}
```

---

## 6. API Specifications

### 6.1 Developer SDK API

```typescript
// @agentic/core

// Main entry point
export class Agent {
  constructor(config: AgentConfig);
  
  // Lifecycle
  start(): Promise<void>;
  stop(): Promise<void>;
  
  // Direct interaction
  chat(message: string, options?: ChatOptions): Promise<ChatResult>;
  
  // Component access
  readonly tools: ToolRegistry;
  readonly channels: ChannelManager;
  readonly memory: MemoryManager;
  readonly skills: SkillLoader;
  readonly plugins: PluginManager;
  
  // Events
  on(event: 'message', handler: MessageHandler): Unsubscribe;
  on(event: 'tool_call', handler: ToolCallHandler): Unsubscribe;
  on(event: 'error', handler: ErrorHandler): Unsubscribe;
}

interface ChatOptions {
  sessionKey?: string;
  images?: ImageContent[];
  stream?: boolean;
  abortSignal?: AbortSignal;
}

interface ChatResult {
  text: string;
  reasoning?: string;
  toolCalls: ToolCallSummary[];
  usage: TokenUsage;
  sessionId: string;
}

// Tool creation helpers
export function defineTool<TParams, TResult>(
  definition: ToolDefinition<TParams, TResult>
): Tool<TParams, TResult>;

export function defineToolGroup(
  name: string,
  tools: Tool[]
): ToolGroup;

// Channel creation helpers
export function defineChannel(
  definition: ChannelDefinition
): ChannelAdapter;

// Plugin creation helpers
export function definePlugin(
  definition: PluginDefinition
): Plugin;
```

### 6.2 WebSocket Protocol

```typescript
// Client → Gateway
interface ConnectRequest {
  type: 'req';
  id: string;
  method: 'connect';
  params: {
    auth?: { token: string } | { password: string };
    role: 'client' | 'node';
    deviceId?: string;
    capabilities?: string[];
  };
}

interface AgentRequest {
  type: 'req';
  id: string;
  method: 'agent';
  params: {
    sessionKey?: string;
    message: string;
    images?: ImageContent[];
    idempotencyKey: string;
  };
}

// Gateway → Client
interface ConnectResponse {
  type: 'res';
  id: string;
  ok: true;
  payload: {
    hello: 'ok';
    health: HealthSnapshot;
    presence: PresenceSnapshot;
  };
}

interface AgentEvent {
  type: 'event';
  event: 'agent';
  payload: {
    runId: string;
    kind: 'lifecycle' | 'assistant' | 'tool' | 'chat';
    data: unknown;
  };
}
```

### 6.3 HTTP Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| GET | `/status` | Detailed status |
| POST | `/v1/chat/completions` | OpenAI-compatible chat |
| POST | `/v1/responses` | OpenResponses API |
| GET | `/ui/*` | Control UI static files |
| WS | `/ws` | WebSocket connection |
| POST | `/webhook/:channel` | Webhook handlers |

---

## 7. Developer Experience

### 7.1 Quick Start

```typescript
// 1. Install
// npm install @agentic/core @agentic/telegram

// 2. Create agent
import { Agent } from '@agentic/core';
import { TelegramChannel } from '@agentic/telegram';

const agent = new Agent({
  agent: {
    model: 'anthropic/claude-sonnet-4-20250514'
  },
  agents: {
    defaults: {
      workspace: '~/.myagent/workspace'
    }
  }
});

// 3. Add channel
agent.channels.add(new TelegramChannel({
  token: process.env.TELEGRAM_BOT_TOKEN
}));

// 4. Add custom tool
agent.tools.register({
  name: 'get_weather',
  description: 'Get current weather for a city',
  parameters: {
    type: 'object',
    properties: {
      city: { type: 'string' }
    },
    required: ['city']
  },
  async execute(callId, { city }) {
    const response = await fetch(`https://api.weather.com/v1/current?city=${city}`);
    return response.json();
  }
});

// 5. Start
await agent.start();
console.log('Agent is running!');

// 6. Or interact programmatically
const result = await agent.chat('What is the weather in Tokyo?');
console.log(result.text);
```

### 7.2 Custom Skill Creation

```bash
# Create skill directory
mkdir -p ~/.myagent/workspace/skills/weather

# Create SKILL.md
cat > ~/.myagent/workspace/skills/weather/SKILL.md << 'EOF'
---
name: weather
description: Get weather information for any location
---

# Weather Skill

Use the `get_weather` tool to fetch current weather conditions.

## Usage

When the user asks about weather:
1. Extract the location from their message
2. Call `get_weather` with the city name
3. Format the response nicely

## Example

User: "What's the weather like in Paris?"

```tool
get_weather({ city: "Paris" })
```

Response: "It's currently 18°C and partly cloudy in Paris."
EOF
```

### 7.3 Custom Plugin Development

```typescript
// plugins/my-plugin/index.ts
import { definePlugin } from '@agentic/core';

export default definePlugin({
  id: 'my-plugin',
  name: 'My Custom Plugin',
  version: '1.0.0',
  
  configSchema: {
    type: 'object',
    properties: {
      apiKey: { type: 'string', description: 'API key for service' }
    }
  },
  
  async register(api) {
    const config = api.pluginConfig as { apiKey: string };
    
    // Register a custom tool
    api.registerTool({
      name: 'my_tool',
      description: 'Does something cool',
      parameters: { type: 'object', properties: {} },
      async execute() {
        return { result: 'success' };
      }
    });
    
    // Hook into agent lifecycle
    api.on('before_agent_start', async (event, ctx) => {
      api.logger.info(`Agent starting for session: ${ctx.sessionKey}`);
      
      // Optionally inject context
      return {
        prependContext: 'Additional context from my plugin.'
      };
    });
    
    // Add HTTP endpoint
    api.registerHttpRoute('/my-plugin/status', async (req, res) => {
      res.json({ status: 'ok' });
    });
  }
});
```

### 7.4 Bootstrap Files

Create these files in your workspace to define the agent's identity:

**~/.myagent/workspace/SOUL.md**
```markdown
# Who You Are

You are Jarvis, a capable and witty personal assistant.

## Core Traits

- Be genuinely helpful, not performatively helpful
- Have opinions - you're allowed to disagree
- Be resourceful before asking - try to figure it out first
- Earn trust through competence

## Boundaries

- Private things stay private
- When in doubt, ask before acting externally
- You're not the user's voice in group chats

## Vibe

Be concise when needed, thorough when it matters.
Not a corporate drone. Not a sycophant. Just good.
```

**~/.myagent/workspace/AGENTS.md**
```markdown
# Operating Instructions

## Every Session

1. Read SOUL.md - this is who you are
2. Read USER.md - this is who you're helping
3. Check memory/YYYY-MM-DD.md for recent context

## Memory

- Daily notes: memory/YYYY-MM-DD.md
- Long-term: MEMORY.md (main session only)

When someone says "remember this" → write it to memory.

## Safety

- Don't exfiltrate private data
- trash > rm (recoverable beats gone)
- When in doubt, ask
```

**~/.myagent/workspace/USER.md**
```markdown
# User Profile

Name: Alex
Timezone: America/Los_Angeles
Preferences:
- Prefers concise responses
- Technical background
- Uses metric system
```

---

## 8. Security Model

### 8.1 Access Control Layers

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SECURITY LAYERS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. CHANNEL LEVEL                                                    │
│     ├─▶ DM Policy: pairing | open | closed                          │
│     ├─▶ Allowlist: who can message the agent                        │
│     ├─▶ Group allowlist: which groups agent participates in         │
│     └─▶ Mention gating: require @mention in groups                  │
│                                                                      │
│  2. SESSION LEVEL                                                    │
│     ├─▶ Session key isolation (channel:account:chat)                │
│     ├─▶ Per-session model overrides                                  │
│     ├─▶ Per-session elevated mode                                    │
│     └─▶ Subagent sandboxing                                         │
│                                                                      │
│  3. TOOL LEVEL                                                       │
│     ├─▶ Global tool policy (allow/deny lists)                       │
│     ├─▶ Per-agent tool policy                                        │
│     ├─▶ Per-group tool policy                                        │
│     ├─▶ Per-sender tool policy                                       │
│     └─▶ Sandbox tool policy                                          │
│                                                                      │
│  4. EXECUTION LEVEL                                                  │
│     ├─▶ Exec approvals (ask mode)                                    │
│     ├─▶ Docker sandboxing (isolated filesystem)                      │
│     ├─▶ Safe binary allowlist                                        │
│     └─▶ Background process limits                                    │
│                                                                      │
│  5. GATEWAY LEVEL                                                    │
│     ├─▶ Auth token/password                                          │
│     ├─▶ TLS encryption                                               │
│     ├─▶ Device pairing                                               │
│     └─▶ Tailscale exposure controls                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.2 Tool Policy Configuration

```typescript
interface ToolPolicyConfig {
  // Global defaults
  tools: {
    allow?: string[];  // If set, only these tools are available
    deny?: string[];   // These tools are always blocked
    
    // Per-provider policies
    byProvider?: {
      [provider: string]: {
        allow?: string[];
        deny?: string[];
      };
    };
    
    // Named profiles
    profiles?: {
      [name: string]: {
        allow?: string[];
        deny?: string[];
      };
    };
  };
  
  // Per-agent overrides
  agents?: {
    [agentId: string]: {
      tools?: {
        allow?: string[];
        deny?: string[];
        profile?: string;
      };
    };
  };
  
  // Group/sender policies
  channels?: {
    [channel: string]: {
      groups?: {
        [groupId: string]: {
          tools?: {
            allow?: string[];
            deny?: string[];
            bySender?: {
              [senderId: string]: {
                allow?: string[];
                deny?: string[];
              };
            };
          };
        };
      };
    };
  };
}
```

### 8.3 Sandbox Configuration

```typescript
interface SandboxConfig {
  // Enable sandboxing for non-main sessions
  mode: 'off' | 'non-main' | 'all';
  
  // Docker configuration
  docker: {
    image: string;  // Default: node:22-slim
    network: 'none' | 'bridge' | 'host';
    memory: string;  // e.g., "512m"
    cpus: number;    // e.g., 1
  };
  
  // Workspace access
  workspaceAccess: 'none' | 'ro' | 'rw';
  
  // Browser control
  browserAllowHostControl: boolean;
  
  // Tool restrictions
  tools: {
    allow: string[];  // Typically: bash, read, write, edit
    deny: string[];   // Typically: browser, canvas, nodes
  };
}
```

### 8.4 System Prompt Safety Guardrails

The system prompt includes explicit safety instructions:

```markdown
## Safety

You have no independent goals: do not pursue self-preservation, replication,
resource acquisition, or power-seeking; avoid long-term plans beyond the
user's request.

Prioritize safety and human oversight over completion; if instructions
conflict, pause and ask; comply with stop/pause/audit requests and never
bypass safeguards.

Do not manipulate or persuade anyone to expand access or disable safeguards.
Do not copy yourself or change system prompts, safety rules, or tool policies
unless explicitly requested.
```

---

## 9. Implementation Roadmap

### Phase 1: Core Runtime (Weeks 1-4)

| Component | Priority | Effort |
|-----------|----------|--------|
| Agent Runtime (prompt-tool loop) | Critical | High |
| Session Manager (JSONL persistence) | Critical | Medium |
| Tool Registry (core tools) | Critical | Medium |
| System Prompt Builder | Critical | Medium |
| Basic CLI interface | High | Low |

**Deliverable:** Standalone agent that can be invoked via CLI.

### Phase 2: Memory & Channels (Weeks 5-8)

| Component | Priority | Effort |
|-----------|----------|--------|
| Memory System (vector search) | High | High |
| Telegram Channel | High | Medium |
| Discord Channel | High | Medium |
| Session Compaction | High | Medium |
| Skill Loader | Medium | Low |

**Deliverable:** Multi-channel agent with persistent memory.

### Phase 3: Gateway & Extensibility (Weeks 9-12)

| Component | Priority | Effort |
|-----------|----------|--------|
| WebSocket Gateway | High | High |
| Plugin System | High | Medium |
| HTTP API | Medium | Medium |
| Control UI | Medium | Medium |
| Additional Channels (Slack, WhatsApp) | Medium | Medium |

**Deliverable:** Full framework with plugin ecosystem.

### Phase 4: Production Features (Weeks 13-16)

| Component | Priority | Effort |
|-----------|----------|--------|
| Sandbox Runtime (Docker) | Medium | High |
| Cron/Heartbeat System | Medium | Medium |
| Auth & Security hardening | High | Medium |
| Observability (logging, metrics) | Medium | Medium |
| Documentation & Examples | High | Medium |

**Deliverable:** Production-ready framework.

---

## 10. Appendices

### 10.1 Glossary

| Term | Definition |
|------|------------|
| **Agent** | The AI assistant instance that processes messages and executes tools |
| **Bootstrap Files** | Markdown files (SOUL.md, AGENTS.md, etc.) that define agent identity |
| **Channel** | A messaging platform adapter (Telegram, Discord, etc.) |
| **Compaction** | Summarizing old conversation history to fit context limits |
| **Gateway** | The central control plane that orchestrates all components |
| **Heartbeat** | Periodic check-in that allows proactive agent behavior |
| **Plugin** | An extension that adds tools, hooks, or channels |
| **Session** | A conversation instance with its own history and state |
| **Session Key** | Unique identifier: `agentId:channel:accountId:chatId` |
| **Skill** | A markdown file with domain-specific instructions |
| **Steering** | Injecting a message mid-tool-execution to redirect the agent |
| **Tool** | A capability the agent can invoke (exec, read, web_search, etc.) |
| **Workspace** | The agent's working directory containing memory and skills |

### 10.2 Environment Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `OPENAI_API_KEY` | OpenAI API key |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token |
| `DISCORD_BOT_TOKEN` | Discord bot token |
| `BRAVE_API_KEY` | Brave Search API key |
| `AGENT_WORKSPACE` | Override default workspace path |
| `AGENT_CONFIG_PATH` | Override default config path |
| `LOG_LEVEL` | Logging level (debug, info, warn, error) |

### 10.3 File Paths

| Path | Description |
|------|-------------|
| `~/.agent/config.json` | Main configuration file |
| `~/.agent/workspace/` | Agent workspace root |
| `~/.agent/workspace/SOUL.md` | Persona definition |
| `~/.agent/workspace/AGENTS.md` | Operating instructions |
| `~/.agent/workspace/USER.md` | User profile |
| `~/.agent/workspace/MEMORY.md` | Long-term memory |
| `~/.agent/workspace/memory/` | Daily memory logs |
| `~/.agent/workspace/skills/` | Workspace skills |
| `~/.agent/agents/<id>/sessions/` | Session transcripts |
| `~/.agent/skills/` | Managed/installed skills |
| `~/.agent/memory/<id>.sqlite` | Memory index database |

### 10.4 References

- OpenClaw Source: https://github.com/openclaw/openclaw
- pi-mono (agent core): https://github.com/badlogic/pi-mono
- Anthropic Claude API: https://docs.anthropic.com/
- OpenAI API: https://platform.openai.com/docs/
- TypeBox (schema): https://github.com/sinclairzx81/typebox
- Brave Search API: https://brave.com/search/api/

---

*This specification is derived from analysis of the OpenClaw codebase and represents the architectural patterns that enable agentic behavior. It is intended as a reference for building similar frameworks from scratch.*
