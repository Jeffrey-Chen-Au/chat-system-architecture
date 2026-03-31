# Live Chat — Multi-Tenant Internal Chat System Architecture

## 1. Purpose

This document defines how the live chat system evolves from a **single Firebase project** into a **multi-tenant architecture**.

### Focus

- system structure  
- tenant isolation model  
- cross-company communication  
- authentication and routing logic

### Out of Scope

- detailed backend implementation  
- Firestore schema definitions  
- frontend/UI behavior  

### Related Documents

For implementation details, refer to:

- Firebase Backend (Firestore + Cloud Functions)  
- Frontend (screens, flows, UX)

---

## 2. Background

The current system runs in production using a **single Firebase project**.

This works for one company, but has limitations:

- all chat data lives in one project  
- no proper tenant isolation  
- scaling to multiple companies becomes complex  
- cross-company communication lacks clear boundaries  
- onboarding new tenants is risky  

👉 Goal: evolve without a full rewrite  

---

## 3. Core Design

The system adopts a:

👉 **Core + Tenant + Bridge architecture**

### 3.1 Core Project (Control Plane)

Core acts as the **central authority**.

#### Responsibilities

- global identity (Firebase Auth)  
- tenant registry  
- membership management  
- unified inbox index  
- cross-company messaging (Bridge)  
- tenant token minting  

---

### 3.2 Tenant Projects (Data Plane)

Each company runs in its own Firebase project.

#### Responsibilities

- internal users  
- internal channels  
- internal messages  
- tenant-local presence and storage  

👉 Key property: **full isolation between tenants**

---

### 3.3 Bridge Layer

Bridge lives in Core and enables **cross-company communication**.

#### Responsibilities

- bridge channels  
- bridge messages  
- cross-tenant membership enforcement  

---

### 3.4 Legacy Support

Existing users remains in Core during migration.

- treated as a tenant logically  
- stored as legacy data technically

```text
sourceType = "legacyCore"
```
---

## 4. Final Target Architecture

### 4.1 Core Project

#### Core Collections
```text
/users/{uid}
/coreTenants/{tenantId}
/coreMemberships/{uid}
/coreUserInboxes/{uid}/items/{itemId}
```
#### Bridge Collections
```text
/bridgeChannels/{channelId}
/bridgeMessages/{channelId}/items/{messageId}
```
#### Legacy Chat (Temporary)
```text
/channels/{channelId}
/channels/{channelId}/messages/{messageId}
```
---

### 4.2 Tenant Projects

Each tenant stores internal chat data:
```text
/channels/{channelId}
/channels/{channelId}/members/{uid}
/channels/{channelId}/messages/{messageId}
```
👉 Provides strict project-level isolation

---

### 4.3 Special Case: existing users

- behaves as tenant in business logic  
- stored as legacy source in Core 

sourceType = “legacyCore”

---

## 5. Authentication and Access Model

### Core-first Authentication

Users always authenticate via Core.

---

### Flow

1. User signs into Core  
2. Read `/coreMemberships/{uid}`  
3. Read `/coreTenants/{tenantId}`  
4. Check `dataHome`  

---

### Routing

#### If `dataHome = "core"`

- stay in Core  
- no tenant token  

#### If `dataHome = "project"`

- call `mintTenantToken(tenantId)`  
- Core validates membership  
- Core mints custom token  
- client signs into tenant  

👉 Centralized identity with tenant isolation

---

## 6. Tenant Routing Model

The system introduces **scope-based routing**.

### Scope

- `core`
- `tenant(tenantId)`

---

### Rule

| Data Type | Scope |
|----------|------|
| Core collections | Core |
| Bridge data | Core |
| Tenant data | Tenant |

👉 Enables dynamic multi-project routing without duplicating logic

---

## 7. Unified Inbox Strategy

Instead of querying multiple databases:
```text
/coreUserInboxes/{uid}/items/{itemId}
```
### Why

Without this:

- client must query multiple Firebase projects  
- sorting becomes inconsistent  
- unread counts break  
- pagination becomes complex  
- UI logic becomes brittle 

---

### Inbox Contains

- sourceType  
- tenantId  
- channelId  
- title  
- lastMessage  
- unreadCount  
- lastReadAt  
- updatedAt  

---

### Routing Behavior

| sourceType | Action |
|----------|--------|
| bridge | open Core |
| tenant | open tenant |
| legacyCore | open Core |

👉 Inbox = **single source of truth for channel list**

---

## 8. Internal Tenant Workflows

### 8.1 dataHome = "core"

- user stays in Core  
- reads/writes legacy chat  
- inbox updated in Core  

---

### 8.2 dataHome = "project"

- user signs into Core  
- tenant token minted  
- user signs into tenant  
- reads/writes tenant data  
- inbox updated in Core  

---

## 9. Backend Responsibilities

Core backend acts as the **consistency and security layer**:

- validates tenant membership
- mints secure tenant tokens
- updates Core inbox index on message events
- maintains unread counts
- enforces cross-tenant communication rules

👉 Backend owns all critical state transitions

---

### Why Backend is Required

Without backend:

- race conditions  
- inconsistent unread  
- duplicated logic  
- security risks   

👉 Backend ensures system consistency

---

## 10. Why Cloud Functions Are Required

If client handled everything:

- unread would break  
- ordering becomes inconsistent  
- security issues  
- duplicated logic  
- notification inconsistencies  

---

### Cloud Functions centralize:

- authorization  
- tenant auth  
- inbox updates  
- metadata generation  

👉 Required for system correctness

---

## 11. Cross-Company Communication

Handled via **Bridge layer in Core**.

---

### Rules

- no direct tenant-to-tenant access  
- all shared chat lives in Core  

---

### Benefits

- strong tenant isolation  
- explicit shared layer  
- simpler security 

---

## 12. Summary

This architecture:

- separates control plane and data plane
- centralizes identity and routing in Core
- isolates tenant data at the project level
- uses a derived inbox index for efficient querying
- applies eventual consistency for scalable state management
- enables cross-company communication via a controlled bridge layer
- supports safe, incremental migration from legacy systems

👉 Core insight:

**A global inbox index decouples client experience from underlying multi-project data storage.**

---

## 13. Data Model Overview

The system separates data into three logical domains:

- **Core** → identity, memberships, tenant registry, unified inbox  
- **Tenant** → internal company chat data  
- **Bridge** → cross-company shared chat data  

---

## 🏗️ Diagram (Conceptual)

```
Users
  ↓
Core Memberships
  ↓
Core Tenants

Core Memberships → Inbox Index

Inbox Index → Bridge Channels → Bridge Messages
Inbox Index → Tenant Channels → Tenant Members → Tenant Messages
```

---

## 📊 Collection Mapping

### Core Project

- Users  
  `/users/{uid}`  

- Memberships  
  `/coreMemberships/{uid}`  

- Tenants  
  `/coreTenants/{tenantId}`  

- Inbox (Unified Index)  
  `/coreUserInboxes/{uid}/items/{itemId}`  

---

### Bridge Layer (in Core)

- Bridge Channels  
  `/bridgeChannels/{channelId}`  

- Bridge Messages  
  `/bridgeMessages/{channelId}/items/{messageId}`  

---

### Tenant Projects

- Channels  
  `/channels/{channelId}`  

- Channel Members  
  `/channels/{channelId}/members/{uid}`  

- Messages  
  `/channels/{channelId}/messages/{messageId}`  

---

## 🔗 Relationship Summary

- `users` defines global identity  
- `coreMemberships` defines tenant access  
- `coreTenants` stores tenant metadata and routing config  
- `coreUserInboxes` acts as unified channel index  
- `bridgeChannels` enables cross-company communication  
- tenant collections store isolated company data  

---

## 🧠 Design Note

The inbox is a **derived index layer**, not the source of truth.

- Messages live in **Tenant** or **Bridge**
- Inbox exists for:
  - fast channel listing  
  - unread tracking  
  - routing  

---

## ⚠️ Important Principle

- **Source of truth = message storage (Tenant / Bridge)**  
- **Inbox = derived state (can be rebuilt)**  

---

## 🚀 Why This Design

- avoids querying multiple Firebase projects  
- enables O(1) channel list query  
- simplifies frontend logic  
- supports multi-tenant scalability  

---

## 14. Unread Count Strategy

Unread state is maintained in the **Core inbox index** on a per-user, per-channel basis.

### Data Model

Each inbox item may include:

- `channelId`
- `sourceType`
- `tenantId`
- `lastMessageAt`
- `lastReadAt`
- `lastReadMessageId`
- `unreadCount`
- `updatedAt`

### Design Principles

- Message history in **Tenant / Bridge** is authoritative
- Unread count in **Core inbox** is derived state
- Read progress is tracked per user, per channel
- Backend owns unread count updates for consistency

### Write Flow

When a new message is created:

1. Message is written to Tenant or Bridge storage
2. Cloud Function triggers on message create
3. Backend identifies channel members
4. For each member except sender:
   - increment `unreadCount`
   - update `lastMessageAt`
   - update preview metadata
5. For sender:
   - keep `unreadCount = 0`
   - advance `lastReadAt` to current message

### Read Flow

When a user opens a channel or marks it as read:

1. Client sends read acknowledgement
2. Backend or trusted write updates inbox item:
   - `lastReadAt = latestSeenMessageAt`
   - `lastReadMessageId = latestSeenMessageId`
   - `unreadCount = 0`

### Why This Model

This avoids:

- scanning message history on every channel list load
- multi-project unread aggregation on the client
- inconsistent badge counts across devices

👉 Core inbox provides fast unread retrieval with a single query

---

## 15. Scaling Limits of Firebase

Firebase is well-suited for rapid development of real-time systems, but scaling a multi-tenant chat platform introduces practical limits that must be designed around.

### What Firebase Handles Well

- managed authentication
- real-time Firestore sync
- event-driven backend workflows with Cloud Functions
- fast iteration with low operational overhead

### Practical Scaling Constraints

#### 1. Write Fan-out
A single message can trigger multiple downstream updates:

- message write
- inbox preview update
- unread count updates for channel members
- notification workflows

This creates **write amplification**, especially in high-member channels.

#### 2. Cloud Function Latency
Critical derived state depends on backend triggers.

This means:

- inbox state may be briefly stale
- unread counts may lag temporarily
- tenant bootstrap may introduce extra latency

#### 3. Firestore Hotspots
High activity concentrated on the same documents or narrow document ranges can create contention.

Examples:

- very large shared channels
- heavily updated metadata documents
- centralized counters without careful design

#### 4. Cross-Project Operational Complexity
Multi-tenant isolation improves scalability, but adds complexity:

- tenant config management
- token minting workflows
- multi-environment deployment coordination
- observability across projects

#### 5. Query and Index Growth
As usage grows, index design becomes more important.

Poor query patterns lead to:

- higher read cost
- slower queries
- more operational maintenance

### Why Multi-Tenant Helps

The multi-tenant design improves scale by splitting workloads across separate Firebase projects.

Benefits include:

- tenant-level isolation
- reduced blast radius
- cleaner security boundaries
- easier enterprise onboarding
- better long-term operational separation

### What Still Needs Care

This architecture does not remove all scaling limits.

Special attention is still required for:

- large channel unread fan-out
- bridge-layer traffic growth
- global search and analytics
- repair/rebuild flows for derived state

### Future Scaling Options

If usage grows significantly, the next steps would include:

- batching or sharding inbox update workflows
- exporting analytics to dedicated systems
- adding stronger observability and repair tooling
- reducing dependency on synchronous derived updates for non-critical paths

---

## 16. Performance Considerations

- Channel list is resolved via single Core query (O(1))
- Avoids N queries across multiple tenant projects
- Reduces client complexity and network overhead

---

## 17. Consistency Model

The system follows an **eventual consistency model** between chat data and derived state.

### Model

- Message data in Tenant / Bridge is **authoritative**
- Inbox and unread counts in Core are **derived**
- Temporary inconsistencies are acceptable

### Behavior

- Writes to message storage occur first
- Cloud Functions asynchronously update:
  - inbox index
  - unread counts
  - metadata

### Guarantees

- No message loss
- Inbox state may lag briefly but converges
- Derived data can be rebuilt if needed

👉 System prioritizes correctness of primary data over immediate consistency of derived state
