# AnyOrg Platform Specification

> A unified organization platform for managing hybrid human/agent workforces

**Version:** 1.0  
**Date:** January 2026

---

## Table of Contents

1. [Vision & Core Concepts](#1-vision--core-concepts)
2. [Architecture Overview](#2-architecture-overview)
3. [Identity System](#3-identity-system)
4. [Organization Structure](#4-organization-structure)
5. [Access Control System](#5-access-control-system)
6. [Registry Systems](#6-registry-systems)
7. [Management Hierarchy](#7-management-hierarchy)
8. [Project Management](#8-project-management)
9. [Component Marketplace](#9-component-marketplace)
10. [Data Models](#10-data-models)
11. [API Design](#11-api-design)
12. [User Interface](#12-user-interface)
13. [Implementation Guide](#13-implementation-guide)

---

## 1. Vision & Core Concepts

### 1.1 The Problem

As AI agents become capable collaborators, organizations need infrastructure to:
- Treat agents as first-class organizational members
- Define clear accountability chains (who manages whom)
- Control what capabilities agents have access to
- Enable humans to build and customize agents
- Manage projects with mixed human/agent teams

### 1.2 Core Principle: Identity Parity

In AnyOrg, **humans and agents are both "Members"** with:
- Unique identities
- Roles and permissions
- Reporting relationships
- Project assignments
- Communication channels

The key difference: **every agent MUST have a human accountable for it**.

### 1.3 Key Concepts

| Concept | Description |
|---------|-------------|
| **Member** | Any entity (human or agent) in the organization |
| **Human** | A real person with authentication credentials |
| **Agent** | An AI entity with capabilities and a human owner |
| **Org Unit** | A team, department, or division in the hierarchy |
| **Role** | A named set of permissions (Admin, Manager, Developer, etc.) |
| **Relationship** | A connection between members (manages, collaborates, owns) |
| **Component** | A building block for agents (tools, skills, channels) |
| **Project** | A scrum-style board with tasks assigned to members |

---

## 2. Architecture Overview

### 2.1 System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              ANYORG PLATFORM                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │                           PRESENTATION LAYER                                │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │ │
│  │  │   Web App    │ │  Mobile App  │ │   CLI Tool   │ │  Agent SDK   │      │ │
│  │  │  (React/Vue) │ │ (React Native)│ │  (anyorg)   │ │ (@anyorg/sdk)│      │ │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘      │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                                      │                                           │
│                                      ▼                                           │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │                              API GATEWAY                                    │ │
│  │  • Authentication (OAuth2, API Keys, Agent Tokens)                         │ │
│  │  • Rate Limiting (per-member, per-org)                                     │ │
│  │  • Request Routing                                                          │ │
│  │  • Audit Logging                                                            │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                                      │                                           │
│         ┌────────────────────────────┼────────────────────────────┐             │
│         │                            │                            │             │
│         ▼                            ▼                            ▼             │
│  ┌──────────────┐          ┌──────────────┐          ┌──────────────┐          │
│  │   IDENTITY   │          │     ORG      │          │   PROJECT    │          │
│  │   SERVICE    │          │   SERVICE    │          │   SERVICE    │          │
│  │              │          │              │          │              │          │
│  │ • Members    │          │ • Org Units  │          │ • Boards     │          │
│  │ • Auth       │          │ • Hierarchy  │          │ • Tasks      │          │
│  │ • Tokens     │          │ • Roles      │          │ • Sprints    │          │
│  └──────┬───────┘          └──────┬───────┘          └──────┬───────┘          │
│         │                         │                         │                   │
│         │    ┌────────────────────┼────────────────────┐    │                   │
│         │    │                    │                    │    │                   │
│         ▼    ▼                    ▼                    ▼    ▼                   │
│  ┌──────────────┐          ┌──────────────┐          ┌──────────────┐          │
│  │    AGENT     │          │   ACCESS     │          │  COMPONENT   │          │
│  │   SERVICE    │          │   CONTROL    │          │   SERVICE    │          │
│  │              │          │   SERVICE    │          │              │          │
│  │ • Registry   │          │ • Policies   │          │ • Catalog    │          │
│  │ • Builder    │          │ • Evaluation │          │ • Allocation │          │
│  │ • Runtime    │          │ • Audit      │          │ • Versions   │          │
│  └──────┬───────┘          └──────┬───────┘          └──────┬───────┘          │
│         │                         │                         │                   │
│         └─────────────────────────┼─────────────────────────┘                   │
│                                   │                                             │
│                                   ▼                                             │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │                            DATA LAYER                                       │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │ │
│  │  │  PostgreSQL  │ │    Redis     │ │ Elasticsearch│ │     S3       │      │ │
│  │  │  (Primary)   │ │   (Cache)    │ │   (Search)   │ │  (Storage)   │      │ │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘      │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │                         AGENT RUNTIME LAYER                                 │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐      │ │
│  │  │Agent Runtime │ │Agent Runtime │ │Agent Runtime │ │Agent Runtime │      │ │
│  │  │  (Agent 1)   │ │  (Agent 2)   │ │  (Agent 3)   │ │  (Agent N)   │      │ │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘      │ │
│  │                        (Kubernetes / Serverless)                            │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Data Flow: Human Creates Agent

```
Human                    AnyOrg Platform                    Agent Runtime
  │                            │                                  │
  │──── Create Agent ─────────▶│                                  │
  │     {name, components,     │                                  │
  │      permissions}          │                                  │
  │                            │                                  │
  │                            │◀─── Check Permissions ──────────▶│
  │                            │     (Can user access these       │
  │                            │      components?)                │
  │                            │                                  │
  │                            │──── Validate Org Policy ────────▶│
  │                            │     (Components allowed for      │
  │                            │      this org unit?)             │
  │                            │                                  │
  │                            │──── Create Agent Identity ──────▶│
  │                            │     {id, owner, orgUnit,         │
  │                            │      capabilities}               │
  │                            │                                  │
  │                            │──── Provision Runtime ──────────▶│
  │                            │                                  │
  │                            │◀─── Agent Ready ─────────────────│
  │                            │                                  │
  │◀─── Agent Created ─────────│                                  │
  │     {agentId, endpoint,    │                                  │
  │      credentials}          │                                  │
```

### 2.3 Data Flow: Agent Requests Action

```
Agent                    AnyOrg Platform                    Target System
  │                            │                                  │
  │──── Request Action ───────▶│                                  │
  │     {action, target,       │                                  │
  │      context}              │                                  │
  │                            │                                  │
  │                            │──── Identify Agent ─────────────▶│
  │                            │     (Validate token)             │
  │                            │                                  │
  │                            │──── Check Relationships ────────▶│
  │                            │     (Can agent interact with     │
  │                            │      target?)                    │
  │                            │                                  │
  │                            │──── Evaluate Policies ──────────▶│
  │                            │     (Action allowed for this     │
  │                            │      agent's role?)              │
  │                            │                                  │
  │                            │──── Audit Log ──────────────────▶│
  │                            │                                  │
  │                            │──── Execute / Proxy ────────────▶│
  │                            │                                  │
  │◀─── Result ────────────────│◀─── Result ──────────────────────│
```

---

## 3. Identity System

### 3.1 Member Identity Model

Every entity in AnyOrg is a **Member** with a unified identity:

```typescript
interface Member {
  // Core identity
  id: string;                    // Globally unique ID
  type: 'human' | 'agent';       // Member type
  displayName: string;           // Human-readable name
  email?: string;                // Contact (humans only)
  avatarUrl?: string;            // Profile image
  
  // Organizational
  orgId: string;                 // Organization they belong to
  orgUnitId: string;             // Team/department
  status: 'active' | 'suspended' | 'archived';
  
  // Relationships
  managerId?: string;            // Direct manager (Member ID)
  ownerId?: string;              // For agents: human accountable
  
  // Metadata
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;             // Member ID who created this
  
  // Type-specific data
  humanProfile?: HumanProfile;
  agentProfile?: AgentProfile;
}

interface HumanProfile {
  authMethods: AuthMethod[];     // OAuth, password, SSO
  mfaEnabled: boolean;
  timezone: string;
  locale: string;
  lastLoginAt?: Date;
  
  // Capabilities
  canCreateAgents: boolean;
  maxAgentsOwned: number;
}

interface AgentProfile {
  // Accountability
  ownerId: string;               // REQUIRED: Human owner
  
  // Capabilities
  components: ComponentBinding[];
  model: string;                 // e.g., "anthropic/claude-sonnet-4"
  
  // Runtime
  runtimeId?: string;            // Current runtime instance
  endpoint?: string;             // API endpoint
  
  // Configuration
  systemPrompt?: string;
  skills: string[];
  tools: string[];
  
  // Limits
  tokenBudget?: number;          // Monthly token limit
  rateLimitRpm?: number;         // Requests per minute
}
```

### 3.2 Authentication Flows

```
┌─────────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION FLOWS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  HUMAN AUTHENTICATION                                            │
│  ───────────────────                                            │
│                                                                  │
│  ┌─────────┐    OAuth2/OIDC     ┌─────────┐    ┌─────────┐     │
│  │  Human  │ ─────────────────▶ │   IdP   │ ──▶│ AnyOrg  │     │
│  │         │ ◀───── Token ───── │(Google, │    │         │     │
│  └─────────┘                    │ Okta)   │    └─────────┘     │
│                                 └─────────┘                     │
│                                                                  │
│  ┌─────────┐    Username/Pass   ┌─────────┐                     │
│  │  Human  │ ─────────────────▶ │ AnyOrg  │                     │
│  │         │ ◀─── JWT + MFA ─── │   Auth  │                     │
│  └─────────┘                    └─────────┘                     │
│                                                                  │
│  AGENT AUTHENTICATION                                            │
│  ────────────────────                                           │
│                                                                  │
│  ┌─────────┐    Agent Token     ┌─────────┐                     │
│  │  Agent  │ ─────────────────▶ │ AnyOrg  │                     │
│  │         │ ◀─── Validated ─── │  Auth   │                     │
│  └─────────┘    (scoped perms)  └─────────┘                     │
│                                                                  │
│  Agent tokens include:                                           │
│  • Agent ID                                                      │
│  • Owner ID (human accountable)                                  │
│  • Org Unit ID                                                   │
│  • Permitted scopes                                              │
│  • Expiration                                                    │
│  • Rate limit tier                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 Token Structure

```typescript
interface AgentToken {
  // Standard JWT claims
  iss: string;           // "anyorg.io"
  sub: string;           // Agent ID
  aud: string;           // Org ID
  exp: number;           // Expiration
  iat: number;           // Issued at
  
  // AnyOrg claims
  type: 'agent';
  ownerId: string;       // Human accountable
  orgUnitId: string;     // Team/department
  managerId?: string;    // Direct manager
  
  // Scopes
  scopes: string[];      // ["projects:read", "tasks:write", ...]
  
  // Allowed interactions
  canInteractWith: {
    members: string[] | '*';      // Member IDs or wildcard
    orgUnits: string[] | '*';     // Org unit IDs or wildcard
    projects: string[] | '*';     // Project IDs or wildcard
  };
  
  // Rate limiting
  rateLimitTier: 'basic' | 'standard' | 'premium';
}
```

---

## 4. Organization Structure

### 4.1 Org Unit Model

```typescript
interface Organization {
  id: string;
  name: string;
  slug: string;                  // URL-safe identifier
  plan: 'free' | 'team' | 'enterprise';
  
  // Hierarchy
  rootOrgUnitId: string;         // Top-level org unit
  
  // Settings
  settings: OrgSettings;
  
  // Limits
  limits: {
    maxMembers: number;
    maxAgents: number;
    maxProjects: number;
  };
}

interface OrgUnit {
  id: string;
  orgId: string;
  
  // Identity
  name: string;                  // "Engineering", "Sales", etc.
  slug: string;
  description?: string;
  
  // Hierarchy
  parentId?: string;             // Parent org unit
  path: string;                  // Materialized path: "/company/engineering/backend"
  level: number;                 // Depth in tree (0 = root)
  
  // Leadership
  leaderId?: string;             // Member who leads this unit
  
  // Settings
  settings: OrgUnitSettings;
  
  // Component access (inherited + overrides)
  componentPolicy: ComponentPolicy;
}

interface OrgUnitSettings {
  // Agent creation
  allowAgentCreation: boolean;
  maxAgentsPerMember: number;
  
  // Defaults for agents in this unit
  defaultAgentModel: string;
  defaultAgentTokenBudget: number;
  
  // Collaboration
  allowCrossUnitCollaboration: boolean;
  visibleToUnits: string[];      // Which units can see this one
}
```

### 4.2 Org Chart Visualization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              ACME CORP                                       │
│                           (Organization)                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                           ┌─────────────┐                                    │
│                           │     CEO     │                                    │
│                           │   (Human)   │                                    │
│                           │   Jane Doe  │                                    │
│                           └──────┬──────┘                                    │
│                                  │                                           │
│          ┌───────────────────────┼───────────────────────┐                  │
│          │                       │                       │                  │
│          ▼                       ▼                       ▼                  │
│   ┌─────────────┐         ┌─────────────┐         ┌─────────────┐          │
│   │ Engineering │         │    Sales    │         │   Support   │          │
│   │  (OrgUnit)  │         │  (OrgUnit)  │         │  (OrgUnit)  │          │
│   └──────┬──────┘         └──────┬──────┘         └──────┬──────┘          │
│          │                       │                       │                  │
│    ┌─────┴─────┐           ┌─────┴─────┐           ┌─────┴─────┐           │
│    │           │           │           │           │           │           │
│    ▼           ▼           ▼           ▼           ▼           ▼           │
│ ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐          │
│ │ Bob  │   │CodeBot│   │Alice │   │SalesAI│   │ Tom  │   │HelpBot│         │
│ │(Human│   │(Agent)│   │(Human│   │(Agent)│   │(Human│   │(Agent)│         │
│ │ Dev) │   │  Dev) │   │ Rep) │   │ Rep)  │   │ Lead)│   │ L1)   │         │
│ └──┬───┘   └───────┘   └──────┘   └───────┘   └──┬───┘   └───────┘         │
│    │                                              │                         │
│    │ manages                                      │ manages                 │
│    ▼                                              ▼                         │
│ ┌──────┐                                      ┌──────┐                      │
│ │TestBot│                                     │TicketBot                    │
│ │(Agent)│                                     │(Agent)│                     │
│ │ QA)   │                                     │ L2)   │                     │
│ └───────┘                                     └───────┘                     │
│                                                                              │
│  Legend:                                                                     │
│  ┌──────┐ = Human    ┌──────┐ = Agent    ───▶ = Reports to / Manages       │
│  │      │            │      │                                               │
│  └──────┘            └──────┘                                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.3 Hierarchy Operations

```typescript
interface OrgService {
  // Org unit operations
  createOrgUnit(params: CreateOrgUnitParams): Promise<OrgUnit>;
  updateOrgUnit(id: string, params: UpdateOrgUnitParams): Promise<OrgUnit>;
  moveOrgUnit(id: string, newParentId: string): Promise<OrgUnit>;
  archiveOrgUnit(id: string): Promise<void>;
  
  // Hierarchy queries
  getOrgTree(orgId: string): Promise<OrgTreeNode>;
  getAncestors(orgUnitId: string): Promise<OrgUnit[]>;
  getDescendants(orgUnitId: string): Promise<OrgUnit[]>;
  getSiblings(orgUnitId: string): Promise<OrgUnit[]>;
  
  // Member queries
  getMembersInUnit(orgUnitId: string, options?: {
    includeAgents: boolean;
    includeDescendants: boolean;
  }): Promise<Member[]>;
  
  // Leadership
  setUnitLeader(orgUnitId: string, memberId: string): Promise<void>;
  getLeadershipChain(memberId: string): Promise<Member[]>;
}
```

---

## 5. Access Control System

### 5.1 Permission Model

AnyOrg uses a **hybrid RBAC + ReBAC** (Role-Based + Relationship-Based) model:

```typescript
// Role-Based Access Control
interface Role {
  id: string;
  name: string;                  // "Admin", "Manager", "Developer", etc.
  description: string;
  
  // Permissions this role grants
  permissions: Permission[];
  
  // Scope
  scope: 'org' | 'unit' | 'project';
  
  // Inheritance
  inheritsFrom?: string;         // Parent role ID
}

interface Permission {
  resource: string;              // "member", "agent", "project", "task", etc.
  action: string;                // "create", "read", "update", "delete", "manage"
  conditions?: Condition[];      // Additional constraints
}

// Relationship-Based Access Control
interface Relationship {
  id: string;
  
  // The two members in the relationship
  subjectId: string;             // Member initiating action
  subjectType: 'human' | 'agent';
  objectId: string;              // Member being acted upon
  objectType: 'human' | 'agent';
  
  // Relationship type
  type: RelationshipType;
  
  // Permissions granted by this relationship
  grantedPermissions: Permission[];
  
  // Validity
  validFrom?: Date;
  validUntil?: Date;
  status: 'active' | 'suspended' | 'expired';
}

type RelationshipType =
  | 'manages'          // Direct management
  | 'owns'             // Agent ownership
  | 'collaborates'     // Peer collaboration
  | 'delegates_to'     // Temporary delegation
  | 'supervises'       // Oversight without direct management
  | 'mentors';         // Guidance relationship
```

### 5.2 Access Control Decision Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ACCESS CONTROL DECISION FLOW                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Request: Agent "CodeBot" wants to assign task to Human "Alice"              │
│                                                                              │
│  1. IDENTIFY REQUESTER                                                       │
│     │                                                                        │
│     ├─▶ Extract agent token                                                  │
│     ├─▶ Validate token signature & expiry                                    │
│     └─▶ Load agent profile: {id: "codebot", owner: "bob", unit: "eng"}      │
│                                                                              │
│  2. CHECK ROLE PERMISSIONS                                                   │
│     │                                                                        │
│     ├─▶ Agent's roles: ["developer", "task_assigner"]                       │
│     ├─▶ Check: "task_assigner" has "task:assign" permission? ✓              │
│     └─▶ Role check: PASSED                                                   │
│                                                                              │
│  3. CHECK RELATIONSHIP PERMISSIONS                                           │
│     │                                                                        │
│     ├─▶ Query relationships: CodeBot ──?──▶ Alice                           │
│     │                                                                        │
│     ├─▶ Direct relationship? NO                                              │
│     │                                                                        │
│     ├─▶ Through management chain?                                            │
│     │   CodeBot ──[owned_by]──▶ Bob ──[manages]──▶ Alice? NO                │
│     │                                                                        │
│     ├─▶ Same org unit?                                                       │
│     │   CodeBot.unit = "engineering"                                         │
│     │   Alice.unit = "sales"                                                 │
│     │   Same unit? NO                                                        │
│     │                                                                        │
│     ├─▶ Cross-unit collaboration allowed?                                    │
│     │   eng.settings.allowCrossUnitCollaboration = true                      │
│     │   eng.settings.visibleToUnits includes "sales"? YES                    │
│     │                                                                        │
│     └─▶ Relationship check: PASSED (via org unit visibility)                 │
│                                                                              │
│  4. CHECK RESOURCE-SPECIFIC POLICIES                                         │
│     │                                                                        │
│     ├─▶ Project policy: "project-123" allows agents to assign? YES          │
│     └─▶ Policy check: PASSED                                                 │
│                                                                              │
│  5. FINAL DECISION: ALLOW                                                    │
│                                                                              │
│  6. AUDIT LOG                                                                │
│     │                                                                        │
│     └─▶ Record: {                                                            │
│           actor: "codebot",                                                  │
│           action: "task:assign",                                             │
│           target: "alice",                                                   │
│           resource: "task-456",                                              │
│           decision: "allow",                                                 │
│           reasons: ["role:task_assigner", "orgunit:visible"]                │
│         }                                                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Access Control API

```typescript
interface AccessControlService {
  // Permission checks
  check(params: {
    actor: string;               // Member ID
    action: string;              // e.g., "task:assign"
    resource: string;            // e.g., "task:456"
    target?: string;             // Target member ID (if applicable)
    context?: Record<string, unknown>;
  }): Promise<AccessDecision>;
  
  // Bulk permission checks
  checkBatch(checks: AccessCheck[]): Promise<AccessDecision[]>;
  
  // Role management
  assignRole(memberId: string, roleId: string, scope: RoleScope): Promise<void>;
  revokeRole(memberId: string, roleId: string): Promise<void>;
  getMemberRoles(memberId: string): Promise<RoleAssignment[]>;
  
  // Relationship management
  createRelationship(params: CreateRelationshipParams): Promise<Relationship>;
  updateRelationship(id: string, params: UpdateRelationshipParams): Promise<Relationship>;
  endRelationship(id: string): Promise<void>;
  
  // Queries
  getRelationships(memberId: string, options?: {
    type?: RelationshipType;
    direction?: 'outgoing' | 'incoming' | 'both';
  }): Promise<Relationship[]>;
  
  canInteract(member1Id: string, member2Id: string): Promise<boolean>;
  getInteractionPath(member1Id: string, member2Id: string): Promise<InteractionPath>;
}

interface AccessDecision {
  allowed: boolean;
  reasons: string[];
  missingPermissions?: string[];
  suggestedActions?: string[];   // e.g., "Request access from manager"
}
```

### 5.4 Predefined Roles

| Role | Scope | Key Permissions |
|------|-------|-----------------|
| **Org Admin** | org | Full access to everything |
| **Unit Lead** | unit | Manage unit members, create agents, manage projects |
| **Manager** | unit | Manage direct reports, assign tasks, view reports |
| **Developer** | unit | Create agents (limited), manage own agents, work on projects |
| **Agent Owner** | unit | Full control over owned agents |
| **Viewer** | unit | Read-only access to assigned projects |
| **Agent** | unit | Execute assigned tasks, interact with permitted members |

---

## 6. Registry Systems

### 6.1 Human Registry

```typescript
interface HumanRegistry {
  // CRUD
  create(params: CreateHumanParams): Promise<Human>;
  get(id: string): Promise<Human | null>;
  update(id: string, params: UpdateHumanParams): Promise<Human>;
  archive(id: string): Promise<void>;
  
  // Queries
  list(options: ListOptions): Promise<PaginatedResult<Human>>;
  search(query: string, options?: SearchOptions): Promise<Human[]>;
  findByEmail(email: string): Promise<Human | null>;
  
  // Org queries
  getByOrgUnit(orgUnitId: string): Promise<Human[]>;
  getManagers(orgUnitId: string): Promise<Human[]>;
  getAgentOwners(orgId: string): Promise<Human[]>;
  
  // Stats
  getStats(orgId: string): Promise<HumanRegistryStats>;
}

interface CreateHumanParams {
  email: string;
  displayName: string;
  orgUnitId: string;
  managerId?: string;
  roles: string[];
  
  // Capabilities
  canCreateAgents?: boolean;
  maxAgentsOwned?: number;
}

interface HumanRegistryStats {
  total: number;
  active: number;
  byOrgUnit: Record<string, number>;
  agentOwners: number;
  avgAgentsPerOwner: number;
}
```

### 6.2 Agent Registry

```typescript
interface AgentRegistry {
  // CRUD
  create(params: CreateAgentParams): Promise<Agent>;
  get(id: string): Promise<Agent | null>;
  update(id: string, params: UpdateAgentParams): Promise<Agent>;
  archive(id: string): Promise<void>;
  
  // Lifecycle
  activate(id: string): Promise<void>;
  suspend(id: string, reason: string): Promise<void>;
  restart(id: string): Promise<void>;
  
  // Queries
  list(options: ListOptions): Promise<PaginatedResult<Agent>>;
  search(query: string, options?: SearchOptions): Promise<Agent[]>;
  
  // Ownership queries
  getByOwner(ownerId: string): Promise<Agent[]>;
  getByManager(managerId: string): Promise<Agent[]>;
  getByOrgUnit(orgUnitId: string): Promise<Agent[]>;
  
  // Component queries
  getByComponent(componentId: string): Promise<Agent[]>;
  getByCapability(capability: string): Promise<Agent[]>;
  
  // Runtime
  getRuntime(id: string): Promise<AgentRuntimeInfo>;
  getEndpoint(id: string): Promise<string>;
  
  // Stats
  getStats(orgId: string): Promise<AgentRegistryStats>;
}

interface CreateAgentParams {
  displayName: string;
  ownerId: string;               // REQUIRED: Human owner
  orgUnitId: string;
  managerId?: string;            // Can be human or agent
  
  // Configuration
  model: string;
  systemPrompt?: string;
  components: string[];          // Component IDs to bind
  
  // Limits
  tokenBudget?: number;
  rateLimitRpm?: number;
}

interface AgentRuntimeInfo {
  status: 'starting' | 'running' | 'stopped' | 'error';
  instanceId?: string;
  endpoint?: string;
  startedAt?: Date;
  lastActiveAt?: Date;
  resourceUsage: {
    cpu: number;
    memory: number;
    tokensUsedToday: number;
  };
}

interface AgentRegistryStats {
  total: number;
  active: number;
  byStatus: Record<string, number>;
  byOrgUnit: Record<string, number>;
  byModel: Record<string, number>;
  totalTokensToday: number;
}
```

### 6.3 Registry UI

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  MEMBER REGISTRY                                        [+ Add Member ▼]    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Filters: [All Types ▼] [All Units ▼] [Active ▼]    Search: [____________] │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │ Type │ Name          │ Unit        │ Manager     │ Status  │ Actions   ││
│  ├──────┼───────────────┼─────────────┼─────────────┼─────────┼───────────┤│
│  │ 👤   │ Jane Doe      │ Executive   │ —           │ Active  │ [⋮]       ││
│  │ 👤   │ Bob Smith     │ Engineering │ Jane Doe    │ Active  │ [⋮]       ││
│  │ 🤖   │ CodeBot       │ Engineering │ Bob Smith   │ Active  │ [⋮]       ││
│  │ 🤖   │ TestBot       │ Engineering │ CodeBot     │ Active  │ [⋮]       ││
│  │ 👤   │ Alice Johnson │ Sales       │ Jane Doe    │ Active  │ [⋮]       ││
│  │ 🤖   │ SalesAI       │ Sales       │ Alice       │ Active  │ [⋮]       ││
│  │ 👤   │ Tom Williams  │ Support     │ Jane Doe    │ Active  │ [⋮]       ││
│  │ 🤖   │ HelpBot       │ Support     │ Tom         │ Active  │ [⋮]       ││
│  │ 🤖   │ TicketBot     │ Support     │ HelpBot     │ Stopped │ [⋮]       ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│  Showing 9 of 9 members                              [< 1 >] [10 per page ▼]│
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Management Hierarchy

### 7.1 Management Rules

```typescript
interface ManagementRules {
  // Core rule: Every agent must have a human owner
  agentMustHaveHumanOwner: true;
  
  // Managers can be human or agent
  allowAgentManagers: true;
  
  // Agent managers must have a human in their upward chain
  agentManagersMustHaveHumanSupervision: true;
  
  // Maximum depth of agent-only management chain
  maxAgentManagementDepth: 2;
  
  // Humans an agent can manage
  agentCanManageHumans: true;
  maxHumansPerAgentManager: 5;
  
  // Agents an agent can manage
  agentCanManageAgents: true;
  maxAgentsPerAgentManager: 10;
}
```

### 7.2 Management Hierarchy Examples

```
VALID HIERARCHIES:
═══════════════════════════════════════════════════════════════════

Example 1: Human manages Agent manages Agent
─────────────────────────────────────────────
    ┌─────────┐
    │  Human  │  (Owner & Manager)
    │   Bob   │
    └────┬────┘
         │ manages
         ▼
    ┌─────────┐
    │  Agent  │  (Manager)
    │ LeadBot │
    └────┬────┘
         │ manages
         ▼
    ┌─────────┐
    │  Agent  │  (Worker)
    │ WorkBot │
    └─────────┘

✓ Valid: Human at top of chain


Example 2: Agent manages Human (with human supervision)
───────────────────────────────────────────────────────
    ┌─────────┐
    │  Human  │  (Executive, supervises)
    │  Jane   │
    └────┬────┘
         │ supervises
         ▼
    ┌─────────┐
    │  Agent  │  (Team Lead)
    │ TeamBot │──────────┐
    └────┬────┘          │ owned by
         │ manages       │
         ▼               ▼
    ┌─────────┐     ┌─────────┐
    │  Human  │     │  Human  │
    │  Alice  │     │   Bob   │
    └─────────┘     └─────────┘

✓ Valid: Agent has human supervision, humans chose to report to agent


Example 3: Mixed team with human accountability
───────────────────────────────────────────────
    ┌─────────┐
    │  Human  │  (Director, accountable for all)
    │  Maria  │
    └────┬────┘
         │
    ┌────┴────┬────────────┐
    │         │            │
    ▼         ▼            ▼
┌───────┐ ┌───────┐  ┌─────────┐
│ Human │ │ Agent │  │  Agent  │
│  Tom  │ │ DevBot│  │ OpsBot  │
└───┬───┘ └───┬───┘  └────┬────┘
    │         │           │
    ▼         ▼           ▼
┌───────┐ ┌───────┐  ┌─────────┐
│ Agent │ │ Agent │  │  Human  │
│QABot  │ │DocBot │  │  Sara   │
└───────┘ └───────┘  └─────────┘

✓ Valid: Clear human accountability at top


INVALID HIERARCHIES:
═══════════════════════════════════════════════════════════════════

Example 4: Agent without human owner
────────────────────────────────────
    ┌─────────┐
    │  Agent  │  (No owner!)
    │ RogueBot│
    └─────────┘

✗ Invalid: Every agent MUST have a human owner


Example 5: Agent-only management chain too deep
───────────────────────────────────────────────
    ┌─────────┐
    │  Human  │
    │   Bob   │
    └────┬────┘
         │
         ▼
    ┌─────────┐
    │  Agent  │  (Depth 1)
    │ Agent1  │
    └────┬────┘
         │
         ▼
    ┌─────────┐
    │  Agent  │  (Depth 2)
    │ Agent2  │
    └────┬────┘
         │
         ▼
    ┌─────────┐
    │  Agent  │  (Depth 3 - TOO DEEP!)
    │ Agent3  │
    └─────────┘

✗ Invalid: Exceeds maxAgentManagementDepth (2)
```

### 7.3 Management API

```typescript
interface ManagementService {
  // Set manager
  setManager(memberId: string, managerId: string): Promise<void>;
  removeManager(memberId: string): Promise<void>;
  
  // Queries
  getDirectReports(managerId: string): Promise<Member[]>;
  getAllReports(managerId: string): Promise<Member[]>;  // Recursive
  getManagementChain(memberId: string): Promise<Member[]>;
  
  // Validation
  validateManagerAssignment(memberId: string, proposedManagerId: string): Promise<{
    valid: boolean;
    errors: string[];
    warnings: string[];
  }>;
  
  // Agent ownership
  transferAgentOwnership(agentId: string, newOwnerId: string): Promise<void>;
  getOwnedAgents(humanId: string): Promise<Agent[]>;
  
  // Accountability
  getAccountableHuman(memberId: string): Promise<Human>;
  getAccountabilityChain(memberId: string): Promise<Human[]>;
}
```

---

## 8. Project Management

### 8.1 Scrum Board Model

```typescript
interface Project {
  id: string;
  orgId: string;
  
  // Identity
  name: string;
  description?: string;
  slug: string;
  
  // Ownership
  ownerId: string;               // Project owner (human)
  orgUnitId: string;             // Primary org unit
  
  // Team
  members: ProjectMember[];
  
  // Board configuration
  columns: BoardColumn[];
  
  // Sprints
  currentSprintId?: string;
  
  // Settings
  settings: ProjectSettings;
}

interface ProjectMember {
  memberId: string;
  memberType: 'human' | 'agent';
  role: 'owner' | 'admin' | 'member' | 'viewer';
  joinedAt: Date;
  
  // For agents: what they can do in this project
  agentPermissions?: {
    canCreateTasks: boolean;
    canAssignTasks: boolean;
    canUpdateTasks: boolean;
    canComment: boolean;
    autoAssignTypes?: string[];  // Task types auto-assigned to this agent
  };
}

interface BoardColumn {
  id: string;
  name: string;                  // "Backlog", "To Do", "In Progress", "Done"
  position: number;
  
  // WIP limits
  wipLimit?: number;
  
  // Automation
  autoAssignTo?: string;         // Member ID to auto-assign when task enters
  autoNotify?: string[];         // Member IDs to notify
}

interface Task {
  id: string;
  projectId: string;
  
  // Content
  title: string;
  description?: string;
  
  // Status
  columnId: string;
  position: number;              // Position within column
  
  // Assignment
  assigneeId?: string;           // Can be human or agent
  reporterId: string;            // Who created it
  
  // Sprint
  sprintId?: string;
  
  // Metadata
  type: 'story' | 'task' | 'bug' | 'spike';
  priority: 'low' | 'medium' | 'high' | 'critical';
  points?: number;               // Story points
  labels: string[];
  
  // Dates
  dueDate?: Date;
  createdAt: Date;
  updatedAt: Date;
  completedAt?: Date;
  
  // Agent-specific
  agentContext?: {
    objective: string;           // Clear goal for agent
    constraints: string[];       // Limitations
    successCriteria: string[];   // How to know it's done
    humanReviewRequired: boolean;
  };
}

interface Sprint {
  id: string;
  projectId: string;
  
  name: string;
  goal?: string;
  
  startDate: Date;
  endDate: Date;
  
  status: 'planning' | 'active' | 'review' | 'completed';
  
  // Metrics
  plannedPoints: number;
  completedPoints: number;
  velocity?: number;
}
```

### 8.2 Board UI

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PROJECT: Website Redesign                              Sprint: Sprint 5 ▼  │
│  Team: 👤 Bob, 👤 Alice, 🤖 CodeBot, 🤖 DesignBot       [+ Add Task]       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  BACKLOG (12)    TO DO (3)      IN PROGRESS (4)   REVIEW (2)    DONE (8)   │
│  ───────────     ────────       ──────────────    ─────────     ────────   │
│                                                                              │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐    ┌───────────┐  ┌─────────┐ │
│  │ WEB-45    │  │ WEB-42    │  │ WEB-40    │    │ WEB-38    │  │ WEB-35  │ │
│  │ Update    │  │ Fix nav   │  │ Implement │    │ Code      │  │ ✓ Done  │ │
│  │ footer    │  │ menu      │  │ auth flow │    │ review    │  │         │ │
│  │           │  │           │  │           │    │           │  │ 👤 Alice│ │
│  │ 🏷️ UI     │  │ 🏷️ bug    │  │ 🏷️ feature│   │ 🏷️ feature│  └─────────┘ │
│  │ 2 pts     │  │ 1 pt      │  │ 5 pts     │    │ 3 pts     │              │
│  │           │  │           │  │           │    │           │  ┌─────────┐ │
│  │ Unassigned│  │ 🤖 CodeBot│  │ 👤 Bob    │    │ 👤 Alice  │  │ WEB-34  │ │
│  └───────────┘  └───────────┘  └───────────┘    └───────────┘  │ ✓ Done  │ │
│                                                                 │         │ │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐    ┌───────────┐  │🤖CodeBot│ │
│  │ WEB-46    │  │ WEB-43    │  │ WEB-41    │    │ WEB-39    │  └─────────┘ │
│  │ Add dark  │  │ Write     │  │ Design    │    │ Test      │              │
│  │ mode      │  │ unit tests│  │ new icons │    │ coverage  │  ┌─────────┐ │
│  │           │  │           │  │           │    │           │  │ WEB-33  │ │
│  │ 🏷️ feature│  │ 🏷️ test   │  │ 🏷️ design │   │ 🏷️ test   │  │ ✓ Done  │ │
│  │ 3 pts     │  │ 2 pts     │  │ 3 pts     │    │ 2 pts     │  │         │ │
│  │           │  │           │  │           │    │           │  │👤 Bob   │ │
│  │ Unassigned│  │ 🤖 CodeBot│  │🤖DesignBot│    │ 🤖 CodeBot│  └─────────┘ │
│  └───────────┘  └───────────┘  └───────────┘    └───────────┘              │
│                                                                              │
│  ...more       ┌───────────┐  ┌───────────┐                     ...more    │
│                │ WEB-44    │  │ WEB-39    │                                 │
│                │ Update    │  │ API       │                                 │
│                │ README    │  │ endpoint  │                                 │
│                │           │  │           │                                 │
│                │ 🏷️ docs   │  │ 🏷️ backend│                                │
│                │ 1 pt      │  │ 3 pts     │                                 │
│                │           │  │           │                                 │
│                │🤖 CodeBot │  │ 👤 Alice  │                                 │
│                └───────────┘  └───────────┘                                 │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  Sprint Progress: ████████████░░░░░░░░ 60%   |   23/38 points completed    │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.3 Agent Task Assignment

When a task is assigned to an agent, special handling applies:

```typescript
interface AgentTaskExecution {
  taskId: string;
  agentId: string;
  
  // Execution state
  status: 'pending' | 'running' | 'paused' | 'completed' | 'failed' | 'review';
  
  // Execution log
  runs: AgentTaskRun[];
  
  // Human oversight
  humanReviewRequired: boolean;
  reviewerId?: string;
  reviewStatus?: 'pending' | 'approved' | 'rejected' | 'changes_requested';
  
  // Results
  artifacts: TaskArtifact[];
  summary?: string;
}

interface AgentTaskRun {
  id: string;
  startedAt: Date;
  endedAt?: Date;
  
  // What happened
  actions: AgentAction[];
  toolCalls: ToolCallLog[];
  
  // Outcome
  status: 'completed' | 'failed' | 'interrupted';
  error?: string;
  
  // Metrics
  tokensUsed: number;
  durationMs: number;
}

// When agent picks up a task
async function agentStartsTask(agentId: string, taskId: string): Promise<void> {
  // 1. Verify agent has access to this project
  const canAccess = await accessControl.check({
    actor: agentId,
    action: 'task:work',
    resource: `task:${taskId}`
  });
  
  if (!canAccess.allowed) {
    throw new Error('Agent cannot work on this task');
  }
  
  // 2. Load task with agent context
  const task = await tasks.get(taskId);
  
  // 3. Build agent prompt with task context
  const prompt = buildTaskPrompt(task);
  
  // 4. Execute agent with task constraints
  const execution = await agentRuntime.execute({
    agentId,
    prompt,
    context: {
      projectId: task.projectId,
      taskId: task.id,
      objective: task.agentContext?.objective,
      constraints: task.agentContext?.constraints,
      successCriteria: task.agentContext?.successCriteria
    },
    tools: getProjectTools(task.projectId),
    onProgress: (update) => {
      // Update task with progress
      tasks.addComment(taskId, {
        author: agentId,
        type: 'progress',
        content: update.summary
      });
    }
  });
  
  // 5. Handle completion
  if (task.agentContext?.humanReviewRequired) {
    await tasks.moveToColumn(taskId, 'review');
    await notifications.notify(task.reporterId, {
      type: 'task_needs_review',
      taskId,
      agentId
    });
  } else {
    await tasks.moveToColumn(taskId, 'done');
  }
}
```

---

## 9. Component Marketplace

### 9.1 Component Model

Components are the building blocks humans use to create agents:

```typescript
interface Component {
  id: string;
  
  // Identity
  name: string;
  slug: string;
  description: string;
  
  // Type
  type: ComponentType;
  
  // Versioning
  version: string;
  versions: ComponentVersion[];
  
  // Publisher
  publisherId: string;           // Organization or "anyorg" for official
  verified: boolean;
  
  // Categorization
  category: string;
  tags: string[];
  
  // Requirements
  requirements: {
    models?: string[];           // Compatible models
    dependencies?: string[];     // Other required components
    permissions?: string[];      // Required permissions
  };
  
  // Pricing (for marketplace)
  pricing: {
    type: 'free' | 'paid' | 'subscription';
    price?: number;
    billingPeriod?: 'monthly' | 'yearly' | 'per_use';
  };
  
  // Stats
  stats: {
    installs: number;
    activeAgents: number;
    rating: number;
    reviews: number;
  };
}

type ComponentType =
  | 'tool'           // A single tool (web_search, file_read, etc.)
  | 'tool_pack'      // Collection of related tools
  | 'skill'          // Skill file with instructions
  | 'skill_pack'     // Collection of skills
  | 'channel'        // Communication channel adapter
  | 'integration'    // External service integration
  | 'model'          // LLM model configuration
  | 'template'       // Agent template (pre-configured agent)
  | 'runtime';       // Custom runtime environment

interface ComponentVersion {
  version: string;
  changelog: string;
  publishedAt: Date;
  
  // The actual component definition
  definition: ComponentDefinition;
  
  // Compatibility
  minPlatformVersion: string;
  maxPlatformVersion?: string;
}

interface ComponentDefinition {
  // For tools
  tools?: ToolDefinition[];
  
  // For skills
  skills?: SkillDefinition[];
  
  // For channels
  channel?: ChannelDefinition;
  
  // For integrations
  integration?: IntegrationDefinition;
  
  // Configuration schema
  configSchema?: JSONSchema;
  
  // Secrets required
  secrets?: SecretRequirement[];
}
```

### 9.2 Component Allocation

Admins control which components are available to the organization and specific members:

```typescript
interface ComponentAllocation {
  id: string;
  orgId: string;
  componentId: string;
  
  // Scope
  scope: AllocationScope;
  
  // Limits
  limits?: {
    maxAgents?: number;          // Max agents that can use this
    maxUsagePerDay?: number;     // For metered components
  };
  
  // Configuration overrides
  configOverrides?: Record<string, unknown>;
  
  // Status
  status: 'active' | 'suspended' | 'expired';
  validUntil?: Date;
}

type AllocationScope =
  | { type: 'org' }                              // Available to entire org
  | { type: 'unit'; unitIds: string[] }          // Specific org units
  | { type: 'role'; roles: string[] }            // Members with these roles
  | { type: 'member'; memberIds: string[] };     // Specific members

interface ComponentService {
  // Catalog
  listAvailable(orgId: string): Promise<Component[]>;
  search(query: string, filters?: ComponentFilters): Promise<Component[]>;
  getDetails(componentId: string): Promise<Component>;
  
  // Allocation (admin only)
  allocate(params: AllocateComponentParams): Promise<ComponentAllocation>;
  updateAllocation(id: string, params: UpdateAllocationParams): Promise<ComponentAllocation>;
  revokeAllocation(id: string): Promise<void>;
  
  // Queries
  getAllocations(orgId: string): Promise<ComponentAllocation[]>;
  getMemberComponents(memberId: string): Promise<Component[]>;
  getAgentComponents(agentId: string): Promise<Component[]>;
  
  // Usage
  getUsageStats(componentId: string, orgId: string): Promise<ComponentUsageStats>;
  
  // For agent creation
  validateComponentAccess(memberId: string, componentIds: string[]): Promise<{
    valid: boolean;
    accessible: string[];
    denied: Array<{ id: string; reason: string }>;
  }>;
}
```

### 9.3 Component Marketplace UI

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  COMPONENT MARKETPLACE                              [My Components] [Admin] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Search: [________________________] [🔍]     Filters: [All Types ▼] [Free ▼]│
│                                                                              │
│  Categories: [All] [Tools] [Skills] [Channels] [Integrations] [Templates]   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                         FEATURED COMPONENTS                              ││
│  ├─────────────────────────────────────────────────────────────────────────┤│
│  │                                                                          ││
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         ││
│  │  │ 🔧 GitHub       │  │ 📧 Gmail        │  │ 🎨 Design Tools │         ││
│  │  │   Integration   │  │   Integration   │  │   Pack          │         ││
│  │  │                 │  │                 │  │                 │         ││
│  │  │ ⭐ 4.8 (234)    │  │ ⭐ 4.9 (567)    │  │ ⭐ 4.7 (123)    │         ││
│  │  │ 🏢 Official     │  │ 🏢 Official     │  │ 🏢 Official     │         ││
│  │  │ Free            │  │ Free            │  │ $9.99/mo        │         ││
│  │  │                 │  │                 │  │                 │         ││
│  │  │ [View] [Add]    │  │ [View] [Add]    │  │ [View] [Add]    │         ││
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘         ││
│  │                                                                          ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                           ALL COMPONENTS                                 ││
│  ├─────────────────────────────────────────────────────────────────────────┤│
│  │                                                                          ││
│  │  ┌───────────────────────────────────────────────────────────────────┐  ││
│  │  │ 🔧 Web Search Tool                               ⭐ 4.9 (1.2k)    │  ││
│  │  │ Search the web using Brave Search API                             │  ││
│  │  │ 🏷️ tool • search • web        🏢 Official        Free             │  ││
│  │  │ ✓ Allocated to your org                          [View] [Manage]  │  ││
│  │  └───────────────────────────────────────────────────────────────────┘  ││
│  │                                                                          ││
│  │  ┌───────────────────────────────────────────────────────────────────┐  ││
│  │  │ 🔧 Browser Control                               ⭐ 4.7 (890)     │  ││
│  │  │ Full browser automation with Playwright                           │  ││
│  │  │ 🏷️ tool • browser • automation  🏢 Official      Free             │  ││
│  │  │ ⚠️ Not allocated                                  [View] [Request]│  ││
│  │  └───────────────────────────────────────────────────────────────────┘  ││
│  │                                                                          ││
│  │  ┌───────────────────────────────────────────────────────────────────┐  ││
│  │  │ 📚 Coding Agent Skill                            ⭐ 4.8 (456)     │  ││
│  │  │ Complete software development workflow                            │  ││
│  │  │ 🏷️ skill • coding • development 🏢 Official      Free             │  ││
│  │  │ ✓ Allocated to Engineering only                  [View] [Manage]  │  ││
│  │  └───────────────────────────────────────────────────────────────────┘  ││
│  │                                                                          ││
│  │  ...more components                                                      ││
│  │                                                                          ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 9.4 Agent Builder

Humans use allocated components to build agents:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  CREATE NEW AGENT                                              [Cancel]     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  BASIC INFO                                                                  │
│  ──────────                                                                  │
│  Name:        [DevAssistant_______________]                                  │
│  Description: [AI assistant for development tasks_________________________] │
│  Org Unit:    [Engineering ▼]                                               │
│  Manager:     [Bob Smith (me) ▼]                                            │
│                                                                              │
│  MODEL                                                                       │
│  ─────                                                                       │
│  Base Model:  [Claude Sonnet 4 ▼]                                           │
│  Thinking:    [● Off ○ Low ○ Medium ○ High]                                 │
│                                                                              │
│  COMPONENTS                                                                  │
│  ──────────                                                                  │
│  Available to you:                    Selected:                              │
│  ┌─────────────────────────┐         ┌─────────────────────────┐            │
│  │ □ Web Search           │         │ ✓ GitHub Integration    │  [Remove]  │
│  │ □ Browser Control      │   ──▶   │ ✓ Coding Agent Skill    │  [Remove]  │
│  │ □ Gmail Integration    │         │ ✓ File Operations       │  [Remove]  │
│  │ □ Slack Integration    │         └─────────────────────────┘            │
│  │ □ Database Tools       │                                                 │
│  │ □ API Builder          │                                                 │
│  └─────────────────────────┘                                                │
│                                                                              │
│  PERMISSIONS                                                                 │
│  ───────────                                                                 │
│  ☑ Can interact with team members                                           │
│  ☑ Can be assigned to projects                                              │
│  ☐ Can manage other agents                                                  │
│  ☐ Can access external APIs                                                 │
│                                                                              │
│  LIMITS                                                                      │
│  ──────                                                                      │
│  Token Budget:    [100,000___] tokens/month                                 │
│  Rate Limit:      [60________] requests/minute                              │
│                                                                              │
│  IDENTITY (Optional)                                                         │
│  ────────                                                                    │
│  System Prompt:   [___________________________________________]             │
│                   [___________________________________________]             │
│                   [___________________________________________]             │
│                                                                              │
│                                            [Cancel]  [Save Draft]  [Create] │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Data Models

### 10.1 Complete Schema

```typescript
// ═══════════════════════════════════════════════════════════════════════════
// CORE ENTITIES
// ═══════════════════════════════════════════════════════════════════════════

interface Organization {
  id: string;
  name: string;
  slug: string;
  plan: 'free' | 'team' | 'enterprise';
  rootOrgUnitId: string;
  settings: OrgSettings;
  limits: OrgLimits;
  createdAt: Date;
  updatedAt: Date;
}

interface OrgUnit {
  id: string;
  orgId: string;
  name: string;
  slug: string;
  parentId?: string;
  path: string;
  level: number;
  leaderId?: string;
  settings: OrgUnitSettings;
  componentPolicy: ComponentPolicy;
  createdAt: Date;
  updatedAt: Date;
}

interface Member {
  id: string;
  type: 'human' | 'agent';
  displayName: string;
  email?: string;
  avatarUrl?: string;
  orgId: string;
  orgUnitId: string;
  managerId?: string;
  ownerId?: string;               // For agents only
  status: 'active' | 'suspended' | 'archived';
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}

interface Human extends Member {
  type: 'human';
  profile: HumanProfile;
}

interface Agent extends Member {
  type: 'agent';
  ownerId: string;                // Required for agents
  profile: AgentProfile;
}

// ═══════════════════════════════════════════════════════════════════════════
// RELATIONSHIPS & ACCESS CONTROL
// ═══════════════════════════════════════════════════════════════════════════

interface Relationship {
  id: string;
  subjectId: string;
  subjectType: 'human' | 'agent';
  objectId: string;
  objectType: 'human' | 'agent';
  type: RelationshipType;
  grantedPermissions: Permission[];
  validFrom?: Date;
  validUntil?: Date;
  status: 'active' | 'suspended' | 'expired';
  createdAt: Date;
  createdBy: string;
}

interface Role {
  id: string;
  orgId: string;
  name: string;
  description: string;
  permissions: Permission[];
  scope: 'org' | 'unit' | 'project';
  inheritsFrom?: string;
  isSystem: boolean;
  createdAt: Date;
  updatedAt: Date;
}

interface RoleAssignment {
  id: string;
  memberId: string;
  roleId: string;
  scope: RoleScope;
  grantedAt: Date;
  grantedBy: string;
  expiresAt?: Date;
}

interface Permission {
  resource: string;
  action: string;
  conditions?: Condition[];
}

// ═══════════════════════════════════════════════════════════════════════════
// PROJECTS & TASKS
// ═══════════════════════════════════════════════════════════════════════════

interface Project {
  id: string;
  orgId: string;
  name: string;
  description?: string;
  slug: string;
  ownerId: string;
  orgUnitId: string;
  members: ProjectMember[];
  columns: BoardColumn[];
  currentSprintId?: string;
  settings: ProjectSettings;
  createdAt: Date;
  updatedAt: Date;
}

interface Task {
  id: string;
  projectId: string;
  title: string;
  description?: string;
  columnId: string;
  position: number;
  assigneeId?: string;
  reporterId: string;
  sprintId?: string;
  type: TaskType;
  priority: TaskPriority;
  points?: number;
  labels: string[];
  dueDate?: Date;
  agentContext?: AgentTaskContext;
  createdAt: Date;
  updatedAt: Date;
  completedAt?: Date;
}

interface Sprint {
  id: string;
  projectId: string;
  name: string;
  goal?: string;
  startDate: Date;
  endDate: Date;
  status: SprintStatus;
  createdAt: Date;
  updatedAt: Date;
}

// ═══════════════════════════════════════════════════════════════════════════
// COMPONENTS & MARKETPLACE
// ═══════════════════════════════════════════════════════════════════════════

interface Component {
  id: string;
  name: string;
  slug: string;
  description: string;
  type: ComponentType;
  version: string;
  publisherId: string;
  verified: boolean;
  category: string;
  tags: string[];
  requirements: ComponentRequirements;
  pricing: ComponentPricing;
  stats: ComponentStats;
  createdAt: Date;
  updatedAt: Date;
}

interface ComponentAllocation {
  id: string;
  orgId: string;
  componentId: string;
  scope: AllocationScope;
  limits?: AllocationLimits;
  configOverrides?: Record<string, unknown>;
  status: 'active' | 'suspended' | 'expired';
  validUntil?: Date;
  createdAt: Date;
  createdBy: string;
}

interface ComponentBinding {
  componentId: string;
  version: string;
  config?: Record<string, unknown>;
  enabledAt: Date;
}

// ═══════════════════════════════════════════════════════════════════════════
// AUDIT & LOGGING
// ═══════════════════════════════════════════════════════════════════════════

interface AuditLog {
  id: string;
  orgId: string;
  timestamp: Date;
  
  // Actor
  actorId: string;
  actorType: 'human' | 'agent' | 'system';
  
  // Action
  action: string;
  resource: string;
  resourceId?: string;
  
  // Context
  targetId?: string;
  targetType?: string;
  
  // Result
  outcome: 'success' | 'failure' | 'denied';
  reason?: string;
  
  // Details
  metadata?: Record<string, unknown>;
  ipAddress?: string;
  userAgent?: string;
}
```

---

## 11. API Design

### 11.1 REST API Endpoints

```
IDENTITY
────────────────────────────────────────────────────────────────────
POST   /api/v1/auth/login                    # Human login
POST   /api/v1/auth/logout                   # Logout
POST   /api/v1/auth/refresh                  # Refresh token
POST   /api/v1/auth/agent-token              # Generate agent token
GET    /api/v1/me                            # Get current member
PATCH  /api/v1/me                            # Update profile

ORGANIZATIONS
────────────────────────────────────────────────────────────────────
GET    /api/v1/orgs                          # List orgs (for member)
GET    /api/v1/orgs/:orgId                   # Get org details
PATCH  /api/v1/orgs/:orgId                   # Update org (admin)
GET    /api/v1/orgs/:orgId/tree              # Get org tree

ORG UNITS
────────────────────────────────────────────────────────────────────
GET    /api/v1/orgs/:orgId/units             # List units
POST   /api/v1/orgs/:orgId/units             # Create unit
GET    /api/v1/units/:unitId                 # Get unit
PATCH  /api/v1/units/:unitId                 # Update unit
DELETE /api/v1/units/:unitId                 # Archive unit
POST   /api/v1/units/:unitId/move            # Move unit in hierarchy

MEMBERS (Unified Human + Agent)
────────────────────────────────────────────────────────────────────
GET    /api/v1/orgs/:orgId/members           # List all members
GET    /api/v1/members/:memberId             # Get member
PATCH  /api/v1/members/:memberId             # Update member
DELETE /api/v1/members/:memberId             # Archive member
GET    /api/v1/members/:memberId/reports     # Get direct reports
GET    /api/v1/members/:memberId/chain       # Get management chain

HUMANS
────────────────────────────────────────────────────────────────────
POST   /api/v1/orgs/:orgId/humans            # Invite human
GET    /api/v1/orgs/:orgId/humans            # List humans
GET    /api/v1/humans/:humanId               # Get human
GET    /api/v1/humans/:humanId/agents        # Get owned agents

AGENTS
────────────────────────────────────────────────────────────────────
POST   /api/v1/orgs/:orgId/agents            # Create agent
GET    /api/v1/orgs/:orgId/agents            # List agents
GET    /api/v1/agents/:agentId               # Get agent
PATCH  /api/v1/agents/:agentId               # Update agent
DELETE /api/v1/agents/:agentId               # Archive agent
POST   /api/v1/agents/:agentId/activate      # Activate agent
POST   /api/v1/agents/:agentId/suspend       # Suspend agent
POST   /api/v1/agents/:agentId/restart       # Restart agent
GET    /api/v1/agents/:agentId/runtime       # Get runtime info
POST   /api/v1/agents/:agentId/transfer      # Transfer ownership

RELATIONSHIPS
────────────────────────────────────────────────────────────────────
GET    /api/v1/members/:memberId/relationships       # Get relationships
POST   /api/v1/relationships                         # Create relationship
GET    /api/v1/relationships/:relId                  # Get relationship
PATCH  /api/v1/relationships/:relId                  # Update relationship
DELETE /api/v1/relationships/:relId                  # End relationship
GET    /api/v1/relationships/can-interact            # Check if two can interact
        ?subject=:memberId&object=:memberId

ROLES & PERMISSIONS
────────────────────────────────────────────────────────────────────
GET    /api/v1/orgs/:orgId/roles             # List roles
POST   /api/v1/orgs/:orgId/roles             # Create role
GET    /api/v1/roles/:roleId                 # Get role
PATCH  /api/v1/roles/:roleId                 # Update role
DELETE /api/v1/roles/:roleId                 # Delete role
POST   /api/v1/members/:memberId/roles       # Assign role
DELETE /api/v1/members/:memberId/roles/:roleId   # Revoke role
POST   /api/v1/access/check                  # Check permission

PROJECTS
────────────────────────────────────────────────────────────────────
POST   /api/v1/orgs/:orgId/projects          # Create project
GET    /api/v1/orgs/:orgId/projects          # List projects
GET    /api/v1/projects/:projectId           # Get project
PATCH  /api/v1/projects/:projectId           # Update project
DELETE /api/v1/projects/:projectId           # Archive project
GET    /api/v1/projects/:projectId/board     # Get board state
POST   /api/v1/projects/:projectId/members   # Add member
DELETE /api/v1/projects/:projectId/members/:memberId  # Remove member

TASKS
────────────────────────────────────────────────────────────────────
POST   /api/v1/projects/:projectId/tasks     # Create task
GET    /api/v1/projects/:projectId/tasks     # List tasks
GET    /api/v1/tasks/:taskId                 # Get task
PATCH  /api/v1/tasks/:taskId                 # Update task
DELETE /api/v1/tasks/:taskId                 # Delete task
POST   /api/v1/tasks/:taskId/move            # Move task (column/position)
POST   /api/v1/tasks/:taskId/assign          # Assign task
POST   /api/v1/tasks/:taskId/comments        # Add comment
GET    /api/v1/tasks/:taskId/execution       # Get agent execution (if agent assigned)

SPRINTS
────────────────────────────────────────────────────────────────────
POST   /api/v1/projects/:projectId/sprints   # Create sprint
GET    /api/v1/projects/:projectId/sprints   # List sprints
GET    /api/v1/sprints/:sprintId             # Get sprint
PATCH  /api/v1/sprints/:sprintId             # Update sprint
POST   /api/v1/sprints/:sprintId/start       # Start sprint
POST   /api/v1/sprints/:sprintId/complete    # Complete sprint

COMPONENTS
────────────────────────────────────────────────────────────────────
GET    /api/v1/components                    # List marketplace components
GET    /api/v1/components/:componentId       # Get component details
GET    /api/v1/orgs/:orgId/components        # List allocated components
POST   /api/v1/orgs/:orgId/components        # Allocate component (admin)
PATCH  /api/v1/orgs/:orgId/components/:allocId   # Update allocation
DELETE /api/v1/orgs/:orgId/components/:allocId   # Revoke allocation
GET    /api/v1/members/:memberId/components  # Get member's available components

AUDIT
────────────────────────────────────────────────────────────────────
GET    /api/v1/orgs/:orgId/audit             # Query audit logs
GET    /api/v1/members/:memberId/audit       # Member's audit trail
```

### 11.2 WebSocket API

For real-time updates:

```typescript
// Connection
ws://api.anyorg.io/ws?token=<jwt>

// Subscribe to channels
{ "type": "subscribe", "channel": "project:123:board" }
{ "type": "subscribe", "channel": "agent:456:status" }
{ "type": "subscribe", "channel": "org:789:members" }

// Events received
{ "type": "task.moved", "taskId": "...", "from": "todo", "to": "in_progress", "by": "..." }
{ "type": "agent.status", "agentId": "...", "status": "running", "activity": "..." }
{ "type": "member.joined", "memberId": "...", "orgUnitId": "..." }
```

### 11.3 Agent SDK

```typescript
// @anyorg/agent-sdk

import { AnyOrgAgent } from '@anyorg/agent-sdk';

// Initialize agent
const agent = new AnyOrgAgent({
  token: process.env.ANYORG_AGENT_TOKEN,
  baseUrl: 'https://api.anyorg.io'
});

// Get agent's identity and permissions
const me = await agent.getMe();
console.log(`I am ${me.displayName}, owned by ${me.ownerId}`);
console.log(`My components: ${me.components.map(c => c.name).join(', ')}`);

// Check what I can do
const canAssign = await agent.access.check({
  action: 'task:assign',
  resource: 'project:123'
});

// Get my assigned tasks
const myTasks = await agent.tasks.getAssigned();

// Work on a task
for (const task of myTasks) {
  if (task.status === 'todo') {
    // Start working
    await agent.tasks.startWorking(task.id);
    
    // Do the work (using bound components/tools)
    const result = await doWork(task);
    
    // Complete or request review
    if (task.agentContext?.humanReviewRequired) {
      await agent.tasks.submitForReview(task.id, {
        summary: result.summary,
        artifacts: result.artifacts
      });
    } else {
      await agent.tasks.complete(task.id, {
        summary: result.summary,
        artifacts: result.artifacts
      });
    }
  }
}

// Interact with team members (if permitted)
const teammate = await agent.members.get('alice-id');
if (await agent.access.canInteract(teammate.id)) {
  await agent.messages.send(teammate.id, {
    content: 'Completed the refactoring task. Ready for review.'
  });
}
```

---

## 12. User Interface

### 12.1 Navigation Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🏢 ANYORG                                         [🔔] [👤 Bob Smith ▼]   │
├──────────────────┬──────────────────────────────────────────────────────────┤
│                  │                                                          │
│  📊 Dashboard    │                                                          │
│                  │                                                          │
│  👥 Members      │                                                          │
│    ├─ Humans     │                                                          │
│    ├─ Agents     │                                                          │
│    └─ Org Chart  │                                                          │
│                  │                    [Main Content Area]                   │
│  📋 Projects     │                                                          │
│    ├─ All        │                                                          │
│    ├─ My Tasks   │                                                          │
│    └─ Sprints    │                                                          │
│                  │                                                          │
│  🧩 Components   │                                                          │
│    ├─ Marketplace│                                                          │
│    └─ My Library │                                                          │
│                  │                                                          │
│  ⚙️ Settings     │                                                          │
│    ├─ Org        │                                                          │
│    ├─ Roles      │                                                          │
│    └─ Policies   │                                                          │
│                  │                                                          │
│  📜 Audit Log    │                                                          │
│                  │                                                          │
└──────────────────┴──────────────────────────────────────────────────────────┘
```

### 12.2 Key Screens

**Dashboard**
- Organization health metrics
- Agent status overview (running/stopped/errors)
- Recent activity feed
- Quick actions (create agent, create project)

**Org Chart**
- Interactive hierarchy visualization
- Drag-and-drop reorganization
- Filter by type (humans/agents/both)
- Show relationships and reporting lines

**Member Profile**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  MEMBER PROFILE                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  ┌─────────┐                                                          │  │
│  │  │  🤖     │  CodeBot                                    [Edit] [⋮]   │  │
│  │  │ Avatar  │  AI Development Assistant                                │  │
│  │  └─────────┘                                                          │  │
│  │                                                                        │  │
│  │  Status: 🟢 Active      Model: Claude Sonnet 4                        │  │
│  │  Unit: Engineering      Owner: Bob Smith                              │  │
│  │  Manager: Bob Smith     Created: Jan 15, 2026                         │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │ [Overview] [Components] [Relationships] [Projects] [Activity] [Config] ││
│  ├─────────────────────────────────────────────────────────────────────────┤│
│  │                                                                          ││
│  │  COMPONENTS (5)                                                          ││
│  │  ─────────────                                                           ││
│  │  ✓ GitHub Integration          Enabled Jan 15, 2026                     ││
│  │  ✓ Coding Agent Skill          Enabled Jan 15, 2026                     ││
│  │  ✓ File Operations             Enabled Jan 15, 2026                     ││
│  │  ✓ Web Search                  Enabled Jan 20, 2026                     ││
│  │  ✓ Browser Control             Enabled Jan 22, 2026                     ││
│  │                                                                          ││
│  │  [+ Add Component]                                                       ││
│  │                                                                          ││
│  │  REPORTS TO                          MANAGES                             ││
│  │  ──────────                          ───────                             ││
│  │  👤 Bob Smith (Human)                🤖 TestBot                          ││
│  │     Engineering Lead                 🤖 DocBot                           ││
│  │                                                                          ││
│  │  COLLABORATES WITH                                                       ││
│  │  ─────────────────                                                       ││
│  │  👤 Alice (Sales) • 🤖 DesignBot (Design) • 👤 Tom (Support)            ││
│  │                                                                          ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 13. Implementation Guide

### 13.1 Technology Stack Recommendations

| Layer | Recommended Technologies |
|-------|-------------------------|
| **Frontend** | React/Vue + TypeScript, TailwindCSS, React Query |
| **API** | Node.js/Bun + TypeScript, Hono/Fastify, tRPC |
| **Database** | PostgreSQL (primary), Redis (cache/pubsub) |
| **Search** | Elasticsearch or Meilisearch |
| **Auth** | JWT + OAuth2 (Auth0/Clerk/custom) |
| **Agent Runtime** | Kubernetes, Serverless (Lambda/Cloud Run) |
| **Message Queue** | Redis Streams, BullMQ, or Kafka |
| **Storage** | S3-compatible (AWS S3, MinIO, R2) |
| **Real-time** | WebSockets (Socket.io, ws) |

### 13.2 Implementation Phases

```
PHASE 1: FOUNDATION (Weeks 1-4)
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│ 1.1 Core Infrastructure                                          │
├─────────────────────────────────────────────────────────────────┤
│ • Set up monorepo structure                                      │
│ • Configure CI/CD pipeline                                       │
│ • Set up PostgreSQL + Redis                                      │
│ • Implement basic API framework                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 1.2 Identity System                                              │
├─────────────────────────────────────────────────────────────────┤
│ • Member model (unified human/agent)                             │
│ • Human authentication (OAuth2 + password)                       │
│ • Agent token generation and validation                          │
│ • Basic profile CRUD                                             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 1.3 Organization Structure                                       │
├─────────────────────────────────────────────────────────────────┤
│ • Organization CRUD                                              │
│ • Org unit hierarchy (nested set or materialized path)           │
│ • Member assignment to units                                     │
│ • Basic management relationships                                 │
└─────────────────────────────────────────────────────────────────┘


PHASE 2: ACCESS CONTROL (Weeks 5-8)
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│ 2.1 Role-Based Access Control                                    │
├─────────────────────────────────────────────────────────────────┤
│ • Role definitions and permissions                               │
│ • Role assignment to members                                     │
│ • Permission inheritance                                         │
│ • Access check middleware                                        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 2.2 Relationship-Based Access Control                            │
├─────────────────────────────────────────────────────────────────┤
│ • Relationship types (manages, collaborates, etc.)               │
│ • Relationship CRUD                                              │
│ • Graph-based permission queries                                 │
│ • canInteract() implementation                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 2.3 Policy Engine                                                │
├─────────────────────────────────────────────────────────────────┤
│ • Combine RBAC + ReBAC decisions                                 │
│ • Org unit visibility rules                                      │
│ • Resource-specific policies                                     │
│ • Audit logging for all decisions                                │
└─────────────────────────────────────────────────────────────────┘


PHASE 3: AGENT INFRASTRUCTURE (Weeks 9-14)
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│ 3.1 Agent Registry                                               │
├─────────────────────────────────────────────────────────────────┤
│ • Agent CRUD with owner validation                               │
│ • Component binding                                              │
│ • Agent lifecycle (activate/suspend/restart)                     │
│ • Runtime status tracking                                        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 3.2 Agent Runtime                                                │
├─────────────────────────────────────────────────────────────────┤
│ • Runtime provisioning (containers/serverless)                   │
│ • Agent execution engine integration                             │
│ • Token budget and rate limiting                                 │
│ • Health monitoring                                              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 3.3 Component System                                             │
├─────────────────────────────────────────────────────────────────┤
│ • Component definitions and versioning                           │
│ • Component allocation to orgs/units/members                     │
│ • Component binding to agents                                    │
│ • Configuration and secrets management                           │
└─────────────────────────────────────────────────────────────────┘


PHASE 4: PROJECT MANAGEMENT (Weeks 15-18)
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│ 4.1 Project Basics                                               │
├─────────────────────────────────────────────────────────────────┤
│ • Project CRUD                                                   │
│ • Project membership                                             │
│ • Board columns and configuration                                │
│ • Task CRUD                                                      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 4.2 Scrum Features                                               │
├─────────────────────────────────────────────────────────────────┤
│ • Sprint management                                              │
│ • Task assignment (human or agent)                               │
│ • Board drag-and-drop                                            │
│ • Sprint metrics and burndown                                    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 4.3 Agent Task Integration                                       │
├─────────────────────────────────────────────────────────────────┤
│ • Agent task context (objectives, constraints)                   │
│ • Task execution tracking                                        │
│ • Human review workflow                                          │
│ • Progress updates and artifacts                                 │
└─────────────────────────────────────────────────────────────────┘


PHASE 5: POLISH & SCALE (Weeks 19-24)
═══════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────┐
│ 5.1 User Interface                                               │
├─────────────────────────────────────────────────────────────────┤
│ • Dashboard with metrics                                         │
│ • Interactive org chart                                          │
│ • Component marketplace UI                                       │
│ • Agent builder wizard                                           │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 5.2 Real-time Features                                           │
├─────────────────────────────────────────────────────────────────┤
│ • WebSocket infrastructure                                       │
│ • Board real-time updates                                        │
│ • Agent status streaming                                         │
│ • Notifications                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ 5.3 Scale & Production                                           │
├─────────────────────────────────────────────────────────────────┤
│ • Multi-tenant isolation                                         │
│ • Horizontal scaling                                             │
│ • Comprehensive audit logging                                    │
│ • Backup and disaster recovery                                   │
└─────────────────────────────────────────────────────────────────┘
```

### 13.3 Database Schema (PostgreSQL)

```sql
-- ═══════════════════════════════════════════════════════════════
-- CORE TABLES
-- ═══════════════════════════════════════════════════════════════

-- Organizations
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT NOT NULL UNIQUE,
  plan TEXT NOT NULL DEFAULT 'free',
  root_org_unit_id UUID,
  settings JSONB NOT NULL DEFAULT '{}',
  limits JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Org Units (using materialized path for hierarchy)
CREATE TABLE org_units (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID NOT NULL REFERENCES organizations(id),
  name TEXT NOT NULL,
  slug TEXT NOT NULL,
  parent_id UUID REFERENCES org_units(id),
  path TEXT NOT NULL,  -- e.g., '/company/engineering/backend'
  level INTEGER NOT NULL DEFAULT 0,
  leader_id UUID,
  settings JSONB NOT NULL DEFAULT '{}',
  component_policy JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  
  UNIQUE(org_id, slug),
  UNIQUE(org_id, path)
);

CREATE INDEX idx_org_units_path ON org_units USING btree (path);
CREATE INDEX idx_org_units_parent ON org_units(parent_id);

-- Members (unified human + agent)
CREATE TABLE members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type TEXT NOT NULL CHECK (type IN ('human', 'agent')),
  display_name TEXT NOT NULL,
  email TEXT,
  avatar_url TEXT,
  org_id UUID NOT NULL REFERENCES organizations(id),
  org_unit_id UUID NOT NULL REFERENCES org_units(id),
  manager_id UUID REFERENCES members(id),
  owner_id UUID REFERENCES members(id),  -- Required for agents
  status TEXT NOT NULL DEFAULT 'active',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES members(id),
  
  -- Agents must have owner
  CONSTRAINT agent_must_have_owner CHECK (
    type != 'agent' OR owner_id IS NOT NULL
  ),
  -- Only agents have owners
  CONSTRAINT only_agents_have_owners CHECK (
    type = 'agent' OR owner_id IS NULL
  )
);

CREATE INDEX idx_members_org ON members(org_id);
CREATE INDEX idx_members_unit ON members(org_unit_id);
CREATE INDEX idx_members_manager ON members(manager_id);
CREATE INDEX idx_members_owner ON members(owner_id);

-- Human-specific data
CREATE TABLE human_profiles (
  member_id UUID PRIMARY KEY REFERENCES members(id),
  auth_methods JSONB NOT NULL DEFAULT '[]',
  mfa_enabled BOOLEAN NOT NULL DEFAULT false,
  timezone TEXT NOT NULL DEFAULT 'UTC',
  locale TEXT NOT NULL DEFAULT 'en',
  last_login_at TIMESTAMPTZ,
  can_create_agents BOOLEAN NOT NULL DEFAULT true,
  max_agents_owned INTEGER NOT NULL DEFAULT 10
);

-- Agent-specific data
CREATE TABLE agent_profiles (
  member_id UUID PRIMARY KEY REFERENCES members(id),
  model TEXT NOT NULL,
  system_prompt TEXT,
  skills TEXT[] NOT NULL DEFAULT '{}',
  tools TEXT[] NOT NULL DEFAULT '{}',
  components JSONB NOT NULL DEFAULT '[]',
  runtime_id TEXT,
  endpoint TEXT,
  token_budget INTEGER,
  rate_limit_rpm INTEGER
);


-- ═══════════════════════════════════════════════════════════════
-- RELATIONSHIPS & ACCESS CONTROL
-- ═══════════════════════════════════════════════════════════════

CREATE TABLE relationships (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID NOT NULL REFERENCES members(id),
  subject_type TEXT NOT NULL,
  object_id UUID NOT NULL REFERENCES members(id),
  object_type TEXT NOT NULL,
  type TEXT NOT NULL,
  granted_permissions JSONB NOT NULL DEFAULT '[]',
  valid_from TIMESTAMPTZ,
  valid_until TIMESTAMPTZ,
  status TEXT NOT NULL DEFAULT 'active',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES members(id),
  
  UNIQUE(subject_id, object_id, type)
);

CREATE INDEX idx_relationships_subject ON relationships(subject_id);
CREATE INDEX idx_relationships_object ON relationships(object_id);

CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID NOT NULL REFERENCES organizations(id),
  name TEXT NOT NULL,
  description TEXT,
  permissions JSONB NOT NULL DEFAULT '[]',
  scope TEXT NOT NULL,
  inherits_from UUID REFERENCES roles(id),
  is_system BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  
  UNIQUE(org_id, name)
);

CREATE TABLE role_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  member_id UUID NOT NULL REFERENCES members(id),
  role_id UUID NOT NULL REFERENCES roles(id),
  scope_type TEXT NOT NULL,  -- 'org', 'unit', 'project'
  scope_id UUID,             -- ID of specific unit/project (null for org)
  granted_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  granted_by UUID REFERENCES members(id),
  expires_at TIMESTAMPTZ
);

CREATE INDEX idx_role_assignments_member ON role_assignments(member_id);


-- ═══════════════════════════════════════════════════════════════
-- PROJECTS & TASKS
-- ═══════════════════════════════════════════════════════════════

CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID NOT NULL REFERENCES organizations(id),
  name TEXT NOT NULL,
  description TEXT,
  slug TEXT NOT NULL,
  owner_id UUID NOT NULL REFERENCES members(id),
  org_unit_id UUID NOT NULL REFERENCES org_units(id),
  columns JSONB NOT NULL DEFAULT '[]',
  current_sprint_id UUID,
  settings JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  
  UNIQUE(org_id, slug)
);

CREATE TABLE project_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id),
  member_id UUID NOT NULL REFERENCES members(id),
  role TEXT NOT NULL DEFAULT 'member',
  agent_permissions JSONB,
  joined_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  
  UNIQUE(project_id, member_id)
);

CREATE TABLE sprints (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id),
  name TEXT NOT NULL,
  goal TEXT,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  status TEXT NOT NULL DEFAULT 'planning',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id),
  title TEXT NOT NULL,
  description TEXT,
  column_id TEXT NOT NULL,
  position INTEGER NOT NULL,
  assignee_id UUID REFERENCES members(id),
  reporter_id UUID NOT NULL REFERENCES members(id),
  sprint_id UUID REFERENCES sprints(id),
  type TEXT NOT NULL DEFAULT 'task',
  priority TEXT NOT NULL DEFAULT 'medium',
  points INTEGER,
  labels TEXT[] NOT NULL DEFAULT '{}',
  due_date TIMESTAMPTZ,
  agent_context JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  completed_at TIMESTAMPTZ
);

CREATE INDEX idx_tasks_project ON tasks(project_id);
CREATE INDEX idx_tasks_assignee ON tasks(assignee_id);
CREATE INDEX idx_tasks_sprint ON tasks(sprint_id);


-- ═══════════════════════════════════════════════════════════════
-- COMPONENTS
-- ═══════════════════════════════════════════════════════════════

CREATE TABLE components (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT NOT NULL UNIQUE,
  description TEXT,
  type TEXT NOT NULL,
  version TEXT NOT NULL,
  publisher_id UUID,
  verified BOOLEAN NOT NULL DEFAULT false,
  category TEXT,
  tags TEXT[] NOT NULL DEFAULT '{}',
  requirements JSONB NOT NULL DEFAULT '{}',
  pricing JSONB NOT NULL DEFAULT '{}',
  definition JSONB NOT NULL DEFAULT '{}',
  stats JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE component_allocations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID NOT NULL REFERENCES organizations(id),
  component_id UUID NOT NULL REFERENCES components(id),
  scope JSONB NOT NULL,  -- { type: 'org' | 'unit' | 'role' | 'member', ids?: [] }
  limits JSONB,
  config_overrides JSONB,
  status TEXT NOT NULL DEFAULT 'active',
  valid_until TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_by UUID REFERENCES members(id)
);

CREATE INDEX idx_allocations_org ON component_allocations(org_id);
CREATE INDEX idx_allocations_component ON component_allocations(component_id);


-- ═══════════════════════════════════════════════════════════════
-- AUDIT LOG
-- ═══════════════════════════════════════════════════════════════

CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id UUID NOT NULL REFERENCES organizations(id),
  timestamp TIMESTAMPTZ NOT NULL DEFAULT now(),
  actor_id UUID,
  actor_type TEXT NOT NULL,
  action TEXT NOT NULL,
  resource TEXT NOT NULL,
  resource_id UUID,
  target_id UUID,
  target_type TEXT,
  outcome TEXT NOT NULL,
  reason TEXT,
  metadata JSONB,
  ip_address INET,
  user_agent TEXT
);

CREATE INDEX idx_audit_org_time ON audit_logs(org_id, timestamp DESC);
CREATE INDEX idx_audit_actor ON audit_logs(actor_id, timestamp DESC);
```

### 13.4 Core Service Implementations

```typescript
// ═══════════════════════════════════════════════════════════════
// ACCESS CONTROL SERVICE
// ═══════════════════════════════════════════════════════════════

import { db } from './db';

interface AccessCheckParams {
  actor: string;           // Member ID
  action: string;          // e.g., 'task:assign'
  resource: string;        // e.g., 'project:123'
  target?: string;         // Target member ID
  context?: Record<string, unknown>;
}

interface AccessDecision {
  allowed: boolean;
  reasons: string[];
  missingPermissions?: string[];
}

export class AccessControlService {
  async check(params: AccessCheckParams): Promise<AccessDecision> {
    const { actor, action, resource, target } = params;
    
    // 1. Load actor
    const actorMember = await db.members.findById(actor);
    if (!actorMember || actorMember.status !== 'active') {
      return { allowed: false, reasons: ['Actor not found or inactive'] };
    }
    
    // 2. Check role-based permissions
    const roleDecision = await this.checkRolePermissions(actor, action, resource);
    if (!roleDecision.hasPermission) {
      return {
        allowed: false,
        reasons: ['Insufficient role permissions'],
        missingPermissions: [action]
      };
    }
    
    // 3. If there's a target, check relationship
    if (target) {
      const relationshipDecision = await this.checkRelationship(actor, target, action);
      if (!relationshipDecision.allowed) {
        return relationshipDecision;
      }
    }
    
    // 4. Check resource-specific policies
    const policyDecision = await this.checkResourcePolicy(actor, action, resource);
    if (!policyDecision.allowed) {
      return policyDecision;
    }
    
    // 5. Audit
    await this.audit({
      actorId: actor,
      action,
      resource,
      target,
      outcome: 'success'
    });
    
    return {
      allowed: true,
      reasons: [...roleDecision.reasons, ...policyDecision.reasons]
    };
  }
  
  private async checkRolePermissions(
    memberId: string,
    action: string,
    resource: string
  ): Promise<{ hasPermission: boolean; reasons: string[] }> {
    // Get member's role assignments
    const assignments = await db.roleAssignments.findByMember(memberId);
    
    const [resourceType] = resource.split(':');
    const [actionType, actionVerb] = action.split(':');
    
    for (const assignment of assignments) {
      const role = await db.roles.findById(assignment.roleId);
      if (!role) continue;
      
      for (const perm of role.permissions) {
        if (this.matchesPermission(perm, actionType, actionVerb, resourceType)) {
          return {
            hasPermission: true,
            reasons: [`role:${role.name}`]
          };
        }
      }
      
      // Check inherited roles
      if (role.inheritsFrom) {
        const parentResult = await this.checkRolePermissions(
          memberId,
          action,
          resource
        );
        if (parentResult.hasPermission) {
          return parentResult;
        }
      }
    }
    
    return { hasPermission: false, reasons: [] };
  }
  
  private async checkRelationship(
    subjectId: string,
    objectId: string,
    action: string
  ): Promise<AccessDecision> {
    // 1. Check direct relationship
    const direct = await db.relationships.findBetween(subjectId, objectId);
    if (direct && this.relationshipAllows(direct, action)) {
      return { allowed: true, reasons: [`relationship:${direct.type}`] };
    }
    
    // 2. Check management chain
    const subjectMember = await db.members.findById(subjectId);
    const objectMember = await db.members.findById(objectId);
    
    // If subject manages object (directly or through chain)
    if (await this.isInManagementChain(subjectId, objectId)) {
      return { allowed: true, reasons: ['management_chain'] };
    }
    
    // 3. Check org unit visibility
    const subjectUnit = await db.orgUnits.findById(subjectMember!.orgUnitId);
    const objectUnit = await db.orgUnits.findById(objectMember!.orgUnitId);
    
    if (subjectUnit && objectUnit) {
      // Same unit
      if (subjectUnit.id === objectUnit.id) {
        return { allowed: true, reasons: ['same_org_unit'] };
      }
      
      // Check visibility settings
      if (subjectUnit.settings.visibleToUnits?.includes(objectUnit.id) ||
          objectUnit.settings.visibleToUnits?.includes(subjectUnit.id)) {
        return { allowed: true, reasons: ['org_unit_visibility'] };
      }
      
      // Check cross-unit collaboration
      if (subjectUnit.settings.allowCrossUnitCollaboration &&
          objectUnit.settings.allowCrossUnitCollaboration) {
        return { allowed: true, reasons: ['cross_unit_collaboration'] };
      }
    }
    
    return {
      allowed: false,
      reasons: ['No relationship or visibility allows this interaction']
    };
  }
  
  private async isInManagementChain(
    managerId: string,
    subordinateId: string
  ): Promise<boolean> {
    let current = await db.members.findById(subordinateId);
    const visited = new Set<string>();
    
    while (current && current.managerId && !visited.has(current.id)) {
      visited.add(current.id);
      if (current.managerId === managerId) {
        return true;
      }
      current = await db.members.findById(current.managerId);
    }
    
    return false;
  }
  
  async canInteract(member1Id: string, member2Id: string): Promise<boolean> {
    const decision = await this.checkRelationship(member1Id, member2Id, 'interact');
    return decision.allowed;
  }
}


// ═══════════════════════════════════════════════════════════════
// AGENT SERVICE
// ═══════════════════════════════════════════════════════════════

interface CreateAgentParams {
  displayName: string;
  ownerId: string;
  orgUnitId: string;
  managerId?: string;
  model: string;
  systemPrompt?: string;
  componentIds: string[];
}

export class AgentService {
  constructor(
    private db: Database,
    private accessControl: AccessControlService,
    private componentService: ComponentService,
    private runtime: AgentRuntimeService
  ) {}
  
  async create(params: CreateAgentParams): Promise<Agent> {
    const { displayName, ownerId, orgUnitId, componentIds } = params;
    
    // 1. Verify owner is a human
    const owner = await this.db.members.findById(ownerId);
    if (!owner || owner.type !== 'human') {
      throw new Error('Agent owner must be a human');
    }
    
    // 2. Check owner can create agents
    const ownerProfile = await this.db.humanProfiles.findById(ownerId);
    if (!ownerProfile?.canCreateAgents) {
      throw new Error('Owner is not allowed to create agents');
    }
    
    // 3. Check agent limit
    const ownedAgents = await this.db.members.countByOwner(ownerId);
    if (ownedAgents >= (ownerProfile.maxAgentsOwned || 10)) {
      throw new Error('Agent limit reached');
    }
    
    // 4. Validate component access
    const componentAccess = await this.componentService.validateAccess(
      ownerId,
      componentIds
    );
    if (!componentAccess.valid) {
      throw new Error(
        `Cannot access components: ${componentAccess.denied.map(d => d.reason).join(', ')}`
      );
    }
    
    // 5. Validate manager if specified
    if (params.managerId) {
      const validManager = await this.validateManagerAssignment(
        'agent',
        params.managerId
      );
      if (!validManager.valid) {
        throw new Error(validManager.errors.join(', '));
      }
    }
    
    // 6. Create the agent
    const agent = await this.db.transaction(async (tx) => {
      // Create member record
      const member = await tx.members.create({
        type: 'agent',
        displayName,
        orgId: owner.orgId,
        orgUnitId,
        ownerId,
        managerId: params.managerId || ownerId,
        status: 'active',
        createdBy: ownerId
      });
      
      // Create agent profile
      await tx.agentProfiles.create({
        memberId: member.id,
        model: params.model,
        systemPrompt: params.systemPrompt,
        components: componentIds.map(id => ({
          componentId: id,
          enabledAt: new Date()
        }))
      });
      
      // Create ownership relationship
      await tx.relationships.create({
        subjectId: ownerId,
        subjectType: 'human',
        objectId: member.id,
        objectType: 'agent',
        type: 'owns',
        status: 'active',
        createdBy: ownerId
      });
      
      return member;
    });
    
    // 7. Provision runtime
    await this.runtime.provision(agent.id);
    
    return agent;
  }
  
  async validateManagerAssignment(
    memberType: 'human' | 'agent',
    managerId: string
  ): Promise<{ valid: boolean; errors: string[]; warnings: string[] }> {
    const errors: string[] = [];
    const warnings: string[] = [];
    
    const manager = await this.db.members.findById(managerId);
    if (!manager) {
      return { valid: false, errors: ['Manager not found'], warnings: [] };
    }
    
    // If agent manager, check depth
    if (manager.type === 'agent') {
      const depth = await this.getAgentManagementDepth(managerId);
      if (depth >= 2) {
        errors.push('Agent management chain too deep (max 2)');
      }
      
      // Verify there's a human in the upward chain
      const hasHumanSupervisor = await this.hasHumanInChain(managerId);
      if (!hasHumanSupervisor) {
        errors.push('Agent manager must have human supervision in chain');
      }
    }
    
    return {
      valid: errors.length === 0,
      errors,
      warnings
    };
  }
  
  private async getAgentManagementDepth(agentId: string): Promise<number> {
    let depth = 0;
    let current = await this.db.members.findById(agentId);
    
    while (current && current.type === 'agent') {
      depth++;
      if (!current.managerId) break;
      current = await this.db.members.findById(current.managerId);
    }
    
    return depth;
  }
  
  private async hasHumanInChain(memberId: string): Promise<boolean> {
    const chain = await this.getManagementChain(memberId);
    return chain.some(m => m.type === 'human');
  }
  
  async getManagementChain(memberId: string): Promise<Member[]> {
    const chain: Member[] = [];
    let current = await this.db.members.findById(memberId);
    const visited = new Set<string>();
    
    while (current && !visited.has(current.id)) {
      visited.add(current.id);
      chain.push(current);
      if (!current.managerId) break;
      current = await this.db.members.findById(current.managerId);
    }
    
    return chain;
  }
}
```

---

## Summary

**AnyOrg** provides a complete platform for managing hybrid human/agent organizations:

1. **Identity Parity**: Humans and agents are both "Members" with unified identity, authentication, and organizational placement.

2. **Human Accountability**: Every agent must have a human owner, ensuring clear accountability chains.

3. **Flexible Management**: Agents can manage humans and other agents, with safeguards (depth limits, human supervision requirements).

4. **Hybrid RBAC + ReBAC**: Combines role-based permissions with relationship-based access for fine-grained control.

5. **Component Marketplace**: Admins control which building blocks are available; humans build agents from allocated components.

6. **Scrum-Style Projects**: Unified boards where tasks can be assigned to humans or agents, with agent-specific execution tracking and human review workflows.

7. **Full Audit Trail**: Every action by every member (human or agent) is logged for compliance and debugging.

This architecture enables organizations to treat AI agents as true team members while maintaining appropriate human oversight and control.