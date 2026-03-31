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

---

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

---

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

---

👉 Enables dynamic multi-project routing without duplicating logic

---

## 7. Unified Inbox Strategy

Instead of querying multiple databases:
```text
/coreUserInboxes/{uid}/items/{itemId}
```

---

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

---

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

---

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

---

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
- keeps identity centralized  
- isolates tenant data  
- enables cross-company communication  
- supports safe incremental migration  

---

## 13. Consistency Model

The system follows an **eventual consistency model** between Tenant data and Core inbox index.

### Write Flow

1. Message is written to Tenant (or Core for bridge)
2. Cloud Function triggers:
   - updates Core inbox item
   - updates lastMessage / timestamps
   - updates unread counts

### Guarantees

- Tenant data is the **source of truth**
- Core inbox is a **derived index**
- Temporary inconsistencies are tolerated

### Failure Handling

- Inbox can be rebuilt from message history if needed
- Background jobs or manual triggers can re-sync inbox state

## 14. Failure Scenarios

### Case 1: Inbox update fails
- Message still exists in Tenant
- Inbox may show stale data
- Can be recovered via re-sync or backfill

### Case 2: Tenant auth fails
- User cannot access tenant data
- Core remains accessible
- Retry token minting

### Case 3: Partial writes
- Backend ensures atomic updates where possible
- Client should rely on server-confirmed state

👉 System is designed to degrade gracefully without data loss

## 15. Source of Truth

- Messages → Tenant / Bridge storage (authoritative)
- Inbox → derived index in Core
- Membership → Core

👉 Core does not store primary chat data for tenants

## 16. Performance Considerations

- Channel list is resolved via single Core query (O(1))
- Avoids N queries across multiple tenant projects
- Reduces client complexity and network overhead
