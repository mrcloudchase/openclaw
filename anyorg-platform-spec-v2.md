# AnyOrg Platform Specification v2

> The universal control plane for hybrid human/agent organizations

**Version:** 2.0  
**Date:** January 2026

---

## Table of Contents

1. [Philosophy & Principles](#1-philosophy--principles)
2. [Architecture Overview](#2-architecture-overview)
3. [Open Agent Framework (Standalone)](#3-open-agent-framework-standalone)
4. [Channel System](#4-channel-system)
5. [Model Gateway (BYOM)](#5-model-gateway-byom)
6. [Bring Your Own Agent (BYOA)](#6-bring-your-own-agent-byoa)
7. [AnyOrg Platform (Control Plane)](#7-anyorg-platform-control-plane)
8. [Access Control](#8-access-control)
9. [Implementation Strategy](#9-implementation-strategy)

---

## 1. Philosophy & Principles

### 1.1 Core Beliefs

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              DESIGN PRINCIPLES                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  1. MEET HUMANS WHERE THEY WORK                                                  │
│     Slack, Discord, Teams, Email, Jira, GitHub, AWS Console...                  │
│     Agents must operate natively in these environments.                         │
│                                                                                  │
│  2. NO LOCK-IN                                                                   │
│     • Bring Your Own Model (any provider, any model)                            │
│     • Bring Your Own Agent (any framework)                                      │
│     • Bring Your Own Channels (any integration)                                 │
│     • Bring Your Own Infrastructure (cloud, on-prem, hybrid)                    │
│                                                                                  │
│  3. OPEN SOURCE THE FRAMEWORK, MONETIZE THE PLATFORM                            │
│     • Agent Framework: MIT licensed, standalone, drives adoption                │
│     • AnyOrg Platform: Paid product for org control, compliance, scale         │
│                                                                                  │
│  4. AGENTS ARE CHANNEL-NATIVE                                                    │
│     An agent in Slack IS a Slack user. An agent in GitHub IS a GitHub user.    │
│     Not a proxy—a native participant.                                           │
│                                                                                  │
│  5. PLATFORM CONTROLS ACCESS, NOT CAPABILITY                                     │
│     AnyOrg doesn't control what agents CAN do (that's the framework).           │
│     AnyOrg controls what agents are ALLOWED to do in this org.                  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 What's Open vs Paid

| Component | Open Source? | What It Does |
|-----------|--------------|--------------|
| **Agent Framework** | ✅ Yes (MIT) | Build agents, define capabilities, run anywhere |
| **Channel Adapters** | ✅ Yes (MIT) | Connect to Slack, GitHub, Jira, etc. |
| **Agent Protocol** | ✅ Yes (MIT) | Standard interface for any agent to connect |
| **Model Gateway Core** | ✅ Yes (MIT) | Unified inference API across providers |
| **AnyOrg Platform** | ❌ No (Paid) | Org control plane, access control, compliance |

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           ANYORG ARCHITECTURE                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│                     WHERE WORK HAPPENS (Channels)                                │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ │
│  │ Slack  │ │Discord │ │ Teams  │ │ Email  │ │ GitHub │ │  Jira  │ │  AWS   │ │
│  └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘ │
│      │          │          │          │          │          │          │       │
│      └──────────┴──────────┴──────────┴────┬─────┴──────────┴──────────┘       │
│                      Channel Adapters      │      (Open Source)                 │
│  ══════════════════════════════════════════╪════════════════════════════════   │
│                                            │                                    │
│                        CONTROL PLANE       │      (AnyOrg Platform - Paid)      │
│  ┌─────────────────────────────────────────┴───────────────────────────────┐   │
│  │                          ANYORG GATEWAY                                  │   │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐           │   │
│  │  │  Identity  │ │   Access   │ │   Model    │ │   Audit    │           │   │
│  │  │  Service   │ │  Control   │ │  Gateway   │ │  Service   │           │   │
│  │  └────────────┘ └────────────┘ └────────────┘ └────────────┘           │   │
│  └─────────────────────────────────────────┬───────────────────────────────┘   │
│                                            │                                    │
│  ══════════════════════════════════════════╪════════════════════════════════   │
│                                            │                                    │
│                          AGENT LAYER       │      (Any Framework)               │
│  ┌─────────────────────────────────────────┴───────────────────────────────┐   │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐       │   │
│  │  │  AnyOrg Agent    │  │  LangChain Agent │  │  Custom Agent    │       │   │
│  │  │  (Our Framework) │  │  (via Protocol)  │  │  (via Protocol)  │       │   │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────┘       │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                            │                                    │
│  ══════════════════════════════════════════╪════════════════════════════════   │
│                                            │                                    │
│                       INFERENCE LAYER      │      (Bring Your Own Model)        │
│  ┌─────────────────────────────────────────┴───────────────────────────────┐   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │   │
│  │  │ Anthropic│ │  OpenAI  │ │  Azure   │ │  Google  │ │  Ollama  │      │   │
│  │  │  Claude  │ │  GPT-4   │ │  OpenAI  │ │  Gemini  │ │ (on-prem)│      │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘      │   │
│  │                    Org brings their own API keys                         │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Open Agent Framework (Standalone)

The **AnyOrg Agent Framework** is fully open source and can run **without** the AnyOrg platform.

### 3.1 Framework Packages

```
@anyorg/agent-core       Core agent runtime, loop, state management
@anyorg/agent-tools      Tool system, execution, sandboxing
@anyorg/agent-memory     Memory, context, retrieval
@anyorg/agent-skills     Skill loading, instructions
@anyorg/agent-channels   Channel adapters (Slack, GitHub, etc.)
@anyorg/agent-models     Model providers, unified API
@anyorg/agent-protocol   Standard protocol for platform integration
```

### 3.2 Standalone Usage (No Platform)

```typescript
// Run agents completely standalone - no platform required

import { Agent } from '@anyorg/agent-core';
import { SlackChannel } from '@anyorg/agent-channels/slack';
import { GitHubTools } from '@anyorg/agent-tools/github';
import { AnthropicProvider } from '@anyorg/agent-models/anthropic';

const agent = new Agent({
  name: 'DevBot',
  
  // Direct model configuration (org's own key)
  model: new AnthropicProvider({
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-sonnet-4-20250514'
  }),
  
  // Tools
  tools: [
    new GitHubTools({ token: process.env.GITHUB_TOKEN })
  ],
  
  // Channels where this agent operates
  channels: [
    new SlackChannel({
      token: process.env.SLACK_BOT_TOKEN,
      appId: process.env.SLACK_APP_ID
    })
  ],
  
  // Identity
  identity: {
    systemPrompt: `You are DevBot, a helpful coding assistant...`,
    skills: ['./skills/coding.md', './skills/review.md']
  }
});

await agent.start();
```

### 3.3 Platform-Connected Usage

```typescript
// Same framework, connected to AnyOrg Platform for org controls

import { Agent } from '@anyorg/agent-core';
import { AnyOrgPlatform } from '@anyorg/agent-protocol';

const agent = new Agent({
  name: 'DevBot',
  
  // Connect to AnyOrg Platform
  platform: new AnyOrgPlatform({
    endpoint: 'https://api.anyorg.io',
    agentToken: process.env.ANYORG_AGENT_TOKEN
  }),
  
  // Platform now provides:
  // - Model access (via Model Gateway with access control)
  // - Tool permissions (what's allowed for this agent)
  // - Channel credentials (which channels this agent can use)
  // - Identity verification
  // - Usage tracking & audit
});

await agent.start();
```

### 3.4 Agent Core Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         AGENT CORE ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   INPUT                      PROCESS                        OUTPUT              │
│                                                                                  │
│  ┌─────────┐            ┌─────────────┐                 ┌─────────┐            │
│  │ Message │            │   Context   │                 │ Response│            │
│  │ from    │───────────▶│  Assembly   │                 │   to    │            │
│  │ Channel │            └──────┬──────┘                 │ Channel │            │
│  └─────────┘                   │                        └────▲────┘            │
│                          ┌─────▼─────┐                       │                 │
│  ┌─────────┐            │   Model    │                 ┌────┴────┐            │
│  │ Event   │───────────▶│ Inference  │────────────────▶│  Reply  │            │
│  │ from    │            │            │                 │ Stream  │            │
│  │ Tool    │            └─────┬──────┘                 └─────────┘            │
│  └─────────┘                  │                                               │
│                          ┌────▼─────┐                                         │
│                          │   Tool   │                                         │
│                          │Execution │                                         │
│                          └────┬─────┘                                         │
│                               │                                               │
│                          ┌────▼─────┐                                         │
│                          │  Memory  │                                         │
│                          │  Update  │                                         │
│                          └──────────┘                                         │
│                                                                                  │
│  EXTENSION POINTS                                                               │
│  • Tools: Add any tool        • Memory: Any storage backend                    │
│  • Channels: Any adapter      • Hooks: Intercept any step                      │
│  • Models: Any provider       • Platform: Connect or run standalone            │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Channel System

### 4.1 Humans Work Everywhere

Agents must be **native citizens** of every channel, not awkward bots.

```
COMMUNICATION           CODE & VERSION CONTROL      PROJECT MANAGEMENT
─────────────           ──────────────────────      ──────────────────
• Slack                 • GitHub                    • Jira
• Discord               • GitLab                    • Linear
• Microsoft Teams       • Bitbucket                 • Notion
• Email (Gmail/O365)    • Azure DevOps              • Asana
• WhatsApp Business                                 • Monday
                                                    • Confluence

CLOUD INFRASTRUCTURE    STORAGE & FILES             OBSERVABILITY
────────────────────    ───────────────             ─────────────
• AWS                   • Google Drive              • Datadog
• Azure                 • OneDrive/SharePoint       • PagerDuty
• GCP                   • Dropbox                   • Grafana
• On-Prem (SSH/K8s)     • Box                       • Splunk
```

### 4.2 Channel Adapter Interface

```typescript
// Open source - anyone can build adapters

interface ChannelAdapter {
  readonly type: string;           // 'slack', 'github', etc.
  
  // Connection
  connect(credentials: ChannelCredentials): Promise<void>;
  disconnect(): Promise<void>;
  
  // What this channel can do
  readonly capabilities: {
    messaging: boolean;
    threads: boolean;
    reactions: boolean;
    attachments: boolean;
    richText: boolean;
  };
  
  // Inbound events
  on(event: 'message', handler: (msg: InboundMessage) => void): void;
  on(event: 'mention', handler: (mention: Mention) => void): void;
  on(event: 'event', handler: (event: ChannelEvent) => void): void;
  
  // Outbound actions
  sendMessage(params: SendMessageParams): Promise<MessageResult>;
  editMessage(params: EditMessageParams): Promise<void>;
  react(params: ReactParams): Promise<void>;
  
  // Channel-specific actions
  executeAction(action: string, params: Record<string, unknown>): Promise<unknown>;
}
```

### 4.3 Multi-Channel Agent

```typescript
// Agent works across all channels seamlessly

const agent = new Agent({
  name: 'OpsBot',
  
  channels: [
    // Communication
    new SlackAdapter({ token: '...' }),
    new TeamsAdapter({ clientId: '...', clientSecret: '...' }),
    
    // Code
    new GitHubAdapter({ token: '...' }),
    
    // Project management
    new JiraAdapter({ host: '...', token: '...' }),
    new LinearAdapter({ apiKey: '...' }),
    
    // Infrastructure
    new AWSAdapter({ region: 'us-east-1' }),
    new DatadogAdapter({ apiKey: '...' })
  ],
  
  // Agent correlates context across channels
  onMessage: async (message, context) => {
    // Someone mentions JIRA ticket in Slack
    if (message.text.includes('PROJ-')) {
      const ticketId = extractTicketId(message.text);
      
      // Fetch from Jira
      const ticket = await context.channels.jira.getIssue(ticketId);
      
      // Find related GitHub PRs
      const prs = await context.channels.github.searchPRs(ticketId);
      
      // Respond in Slack with full context
      await message.reply({
        text: `Here's ${ticketId}:`,
        blocks: [ticketBlock(ticket), prsBlock(prs)]
      });
    }
  }
});
```

---

## 5. Model Gateway (BYOM)

### 5.1 Org Brings Their Own Models

AnyOrg is an **inference proxy**, not a model provider.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            MODEL GATEWAY                                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  WHAT IT IS                                                                      │
│  • Unified API for all model providers                                          │
│  • Org configures their own API keys                                            │
│  • Access control: who can use which models                                     │
│  • Usage tracking, budgets, rate limiting                                       │
│  • NOT a model provider (just a proxy/router)                                   │
│                                                                                  │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                           │  │
│  │   Agent/Human                 Model Gateway                   Provider    │  │
│  │       │                           │                              │        │  │
│  │       │── inference request ─────▶│                              │        │  │
│  │       │                           │── check access ──▶           │        │  │
│  │       │                           │   (allowed for this member?) │        │  │
│  │       │                           │── check budget ──▶           │        │  │
│  │       │                           │   (within limits?)           │        │  │
│  │       │                           │── route to provider ────────▶│        │  │
│  │       │                           │   (using org's API key)      │        │  │
│  │       │                           │◀── response ─────────────────│        │  │
│  │       │                           │── log usage ──▶              │        │  │
│  │       │◀── response ──────────────│                              │        │  │
│  │                                                                           │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Model Configuration

```yaml
# Org configures their models in AnyOrg Platform

providers:
  anthropic:
    apiKey: "${ANTHROPIC_API_KEY}"   # Org's key
    
  openai:
    apiKey: "${OPENAI_API_KEY}"
    
  azure_openai:
    endpoint: "https://mycompany.openai.azure.com"
    apiKey: "${AZURE_OPENAI_KEY}"
    deployments:
      gpt-4o: "my-gpt4o-deployment"
      
  ollama:  # Self-hosted
    endpoint: "http://internal-llm.company.com:11434"

# Aliases (org names models however they want)
aliases:
  default: anthropic/claude-sonnet-4-20250514
  fast: anthropic/claude-haiku
  powerful: anthropic/claude-opus-4
  internal-only: ollama/llama3:70b   # Data stays on-prem

# Access control
access:
  # Everyone gets basic models
  - scope: { type: org }
    allowedModels: [default, fast]
    limits: { tokensPerDay: 100000 }
    
  # Engineering gets more
  - scope: { type: unit, units: [engineering] }
    allowedModels: [default, fast, powerful]
    limits: { tokensPerDay: 500000 }
    
  # PII handlers must use internal model
  - scope: { type: role, roles: [pii_handler] }
    allowedModels: [internal-only]
    required: true  # Cannot use other models
```

---

## 6. Bring Your Own Agent (BYOA)

### 6.1 The Agent Protocol

Any agent from any framework can connect to AnyOrg by implementing the **Agent Protocol**.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            AGENT PROTOCOL                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ANY FRAMEWORK can connect:                                                      │
│                                                                                  │
│  ┌──────────────────┐     Agent Protocol     ┌──────────────────┐              │
│  │  LangChain       │◀──────────────────────▶│                  │              │
│  │  CrewAI          │                        │  AnyOrg Platform │              │
│  │  AutoGen         │   • Authentication     │                  │              │
│  │  Custom Code     │   • Permission checks  │  • Identity      │              │
│  │  AnyOrg Framework│   • Model access       │  • Access control│              │
│  └──────────────────┘   • Action logging     │  • Audit         │              │
│                         • Member interaction │                  │              │
│                                              └──────────────────┘              │
│                                                                                  │
│  PROTOCOL OPERATIONS                                                             │
│  ───────────────────                                                            │
│  1. register()        - Agent registers, gets identity & token                  │
│  2. authenticate()    - Validate token for each session                         │
│  3. checkPermission() - Before actions, check if allowed                        │
│  4. logAction()       - Report actions for audit                                │
│  5. inference()       - Get model access via Model Gateway                      │
│  6. canInteract()     - Check if can interact with another member               │
│  7. heartbeat()       - Report health status                                    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Protocol Implementation Examples

```python
# Python: LangChain agent connecting to AnyOrg

from langchain.agents import AgentExecutor
from anyorg_protocol import AnyOrgProtocol

anyorg = AnyOrgProtocol(
    endpoint="https://api.anyorg.io",
    token=os.environ["ANYORG_AGENT_TOKEN"]
)

class AnyOrgLangChainAgent:
    def __init__(self, agent: AgentExecutor):
        self.agent = agent
        
    async def run(self, input: str, context: dict):
        # Check permissions
        allowed = await anyorg.check_permission({
            "action": "task:execute",
            "resource": f"task:{context.get('task_id')}"
        })
        if not allowed["allowed"]:
            raise PermissionError(allowed["reason"])
        
        # Run with AnyOrg model gateway
        result = await self.agent.arun(input)
        
        # Log action
        await anyorg.log_action({
            "action": "task:execute",
            "resource": f"task:{context.get('task_id')}",
            "result": "success"
        })
        
        return result
```

```typescript
// TypeScript: Custom agent with protocol

import { AnyOrgProtocol } from '@anyorg/agent-protocol';

const protocol = new AnyOrgProtocol({
  endpoint: 'https://api.anyorg.io',
  token: process.env.ANYORG_AGENT_TOKEN
});

// Before any action
const allowed = await protocol.checkPermission({
  action: 'github:create_pr',
  resource: 'repo:myorg/myrepo'
});

if (allowed.allowed) {
  // Do the action
  await github.createPR({...});
  
  // Log it
  await protocol.logAction({
    action: 'github:create_pr',
    resource: 'repo:myorg/myrepo',
    result: 'success'
  });
}
```

### 6.3 Registration Flow

```
1. Human goes to AnyOrg Dashboard → Agents → Register External Agent
2. Enters: Name, Framework (LangChain/CrewAI/Custom), Capabilities
3. Platform generates: Agent ID + Token
4. Developer adds protocol client to their agent code
5. Agent connects, authenticates, operates under org policies
```

---

## 7. AnyOrg Platform (Control Plane)

### 7.1 What the Platform Does

The platform is the **organizational control plane**. It doesn't run agents—it governs them.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      ANYORG PLATFORM RESPONSIBILITIES                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  IDENTITY                        ACCESS CONTROL                                  │
│  • Human registry                • Who can use which models                     │
│  • Agent registry (any framework)• Who can use which channels                   │
│  • Unified member identity       • Who can interact with whom                   │
│  • Auth tokens                   • What actions are allowed                     │
│                                                                                  │
│  ORGANIZATION                    MODEL GATEWAY                                   │
│  • Org structure (teams)         • Unified API for all providers                │
│  • Reporting relationships       • Org's own API keys                           │
│  • Role definitions              • Usage tracking & budgets                     │
│  • Policy management             • Access control per model                     │
│                                                                                  │
│  CHANNEL REGISTRY                AUDIT & COMPLIANCE                              │
│  • Which channels enabled        • Every action logged                          │
│  • Credential management         • Full audit trail                             │
│  • Channel-specific policies     • Compliance reports                           │
│                                                                                  │
│  DOES NOT:                                                                       │
│  ✗ Run agent code (agents run in customer's infra)                              │
│  ✗ Store conversation content (only metadata)                                   │
│  ✗ Provide models (org brings their own)                                        │
│  ✗ Force specific framework (any framework works)                               │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Platform Services

```typescript
// Platform API surface

interface AnyOrgPlatform {
  // Identity
  identity: {
    getMe(): Promise<Member>;
    getMember(id: string): Promise<Member>;
    listMembers(filters: MemberFilters): Promise<Member[]>;
  };
  
  // Access Control
  access: {
    check(params: AccessCheckParams): Promise<AccessDecision>;
    checkBatch(checks: AccessCheck[]): Promise<AccessDecision[]>;
    canInteract(memberId: string): Promise<boolean>;
  };
  
  // Models
  models: {
    list(): Promise<AvailableModel[]>;
    inference(params: InferenceParams): Promise<InferenceResponse>;
    stream(params: InferenceParams): AsyncIterable<InferenceChunk>;
    getUsage(): Promise<UsageStats>;
  };
  
  // Channels
  channels: {
    list(): Promise<AvailableChannel[]>;
    getCredentials(channelType: string): Promise<ChannelCredentials>;
  };
  
  // Audit
  audit: {
    log(params: AuditLogParams): Promise<void>;
  };
  
  // Agent lifecycle
  agent: {
    register(params: RegisterParams): Promise<AgentCredentials>;
    heartbeat(status: HealthStatus): Promise<void>;
    declareCapabilities(caps: AgentCapabilities): Promise<void>;
  };
}
```

---

## 8. Access Control

### 8.1 Unified Access Control

Access control applies to **everything**:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│    MODELS    │  │   CHANNELS   │  │   MEMBERS    │  │   RESOURCES  │
│              │  │              │  │              │  │              │
│ Who can use  │  │ Who can use  │  │ Who can      │  │ Who can      │
│ which models │  │ which        │  │ interact     │  │ access which │
│              │  │ channels     │  │ with whom    │  │ projects     │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
```

### 8.2 Policy Structure

```yaml
# Policy = WHO + WHAT + CONDITIONS

policies:
  # Model access
  - name: "Engineering gets powerful models"
    who: { type: unit, units: [engineering] }
    what: { type: model, models: ["claude-opus-4", "gpt-4o"] }
    conditions: { tokensPerDay: 500000 }
    
  # Channel access
  - name: "Only DevOps can use AWS"
    who: { type: role, roles: [devops, sre] }
    what: { type: channel, channels: [aws, azure, gcp] }
    conditions: { audit: enhanced }
    
  # Agent restrictions
  - name: "Agents need approval for production"
    who: { type: memberType, memberType: agent }
    what: { type: channel, channels: [aws, azure, gcp] }
    conditions:
      requireApproval: true
      approvers: { role: devops_lead }
      
  # Member interaction
  - name: "Cross-team needs relationship"
    who: { type: org }
    what: { type: interaction, scope: cross_unit }
    conditions:
      requireRelationship: true
      types: [collaborates, manages, project_member]
```

### 8.3 Access Check Flow

```
Request: Agent "DataBot" wants to query AWS Athena

1. IDENTIFY REQUESTER
   Member: DataBot (agent), Owner: Alice, Unit: data-science

2. IDENTIFY REQUEST
   Action: channel:aws:athena:query
   Resource: database:analytics

3. EVALUATE POLICIES
   ✓ "Data Science AWS read access" → DataBot in data-science? YES
   ✓ "Sensitive data access" → Has role data_analyst? YES
   ✓ Not production → No approval needed

4. APPLY CONDITIONS
   Rate limit: 100/hour → 45 used → OK
   Audit level: enhanced

5. DECISION: ALLOWED

6. AUDIT LOG
   { actor: DataBot, owner: Alice, action: athena:query, allowed: true }
```

---

## 9. Implementation Strategy

### 9.1 Build Order

```
PHASE 1: Open Source Foundation
═══════════════════════════════
Release the agent framework (MIT license)
• @anyorg/agent-core, tools, memory, skills
• Channel adapters: Slack, Discord, GitHub, Jira
• Model providers: Anthropic, OpenAI, Ollama
• Docs, examples, tutorials

→ Drives adoption, creates funnel

PHASE 2: Agent Protocol
═══════════════════════
Release protocol spec + clients
• Protocol specification (OpenAPI)
• TypeScript client
• Python client
• Integration guides

→ Enables any framework

PHASE 3: Platform Core (Paid)
═════════════════════════════
• Identity service
• Access control engine
• Model Gateway
• Channel registry
• Audit logging

PHASE 4: Full Platform (Paid)
═════════════════════════════
• Project management
• Agent builder UI
• Advanced policies
• Analytics
• Enterprise (SSO, on-prem)
```

### 9.2 Open Source Strategy

```
OPEN SOURCE (MIT)                      PAID (AnyOrg Platform)
─────────────────                      ─────────────────────

✓ Agent Framework                      ✓ Organization management
  • Core runtime                       ✓ Access control engine
  • Tool system                        ✓ Model Gateway (managed)
  • Memory                             ✓ Audit & compliance
  • Skills                             ✓ Project management
                                       ✓ Agent Builder UI
✓ Channel Adapters                     ✓ Enterprise features
  • All adapters
  • Community contribs

✓ Agent Protocol
  • Specification
  • Client libraries

FUNNEL:
Developer finds framework → Builds agent → Wants org controls → AnyOrg Platform
```

### 9.3 Deployment Options

```
PATTERN 1: Fully Managed (SaaS)
Customer channels ←→ AnyOrg Cloud ←→ Model providers
                         ↓
                   Agents (in AnyOrg cloud)

PATTERN 2: Hybrid
Customer channels ←→ AnyOrg Cloud ←→ Model providers
        ↑                ↑
        └────────────────┘
           Agents (customer's K8s)

PATTERN 3: On-Prem (Enterprise)
┌─────────── Customer Infrastructure ───────────┐
│ Channels ←→ AnyOrg Platform ←→ On-prem LLMs  │
│                    ↓                          │
│              Customer Agents                  │
└───────────────────────────────────────────────┘
```

---

## Summary

This architecture provides **maximum flexibility**:

| Principle | Implementation |
|-----------|----------------|
| **No model lock-in** | Org brings their own API keys, any provider |
| **No framework lock-in** | Any agent framework via Agent Protocol |
| **No channel lock-in** | Open source adapters, community contributions |
| **No platform lock-in** | Framework runs standalone, protocol is open |
| **Work where humans work** | Native presence in Slack, GitHub, Jira, AWS, etc. |

**Business model:**
- **Free**: Open source framework + protocol (drives adoption)
- **Paid**: Platform for org control, compliance, scale (monetization)

The platform adds value through **governance**, not through controlling what agents can do.
