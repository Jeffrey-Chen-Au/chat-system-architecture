# Live Chat — Firebase Backend Architecture

## 1. Overview

This document describes the Firebase backend architecture used by the Live Chat app.

### Focus

- Firestore data model  
- Core vs Tenant separation  
- Cloud Functions responsibilities  
- multi-tenant coordination  

### Out of Scope

- frontend behavior  
- UI flows  

---

## 2. Firestore Architecture

### 2.1 Overview

Firestore is the primary **durable data store** for Live Chat.

It stores:

- user profiles  
- channels  
- channel membership  
- messages  
- tenant registry  
- membership registry  
- unified inbox index  

---

### 2.2 Firestore Topology

Firestore is split into:

👉 **Core (Control Plane)**  
👉 **Tenant (Data Plane)**

---

### Core Firestore (Control Plane)

Used for:

- identity data  
- tenant metadata  
- cross-company communication  
- unified inbox  

#### Collections

```text
/users/{uid}
/coreTenants/{tenantId}
/coreMemberships/{uid}
/coreUserInboxes/{uid}/items/{itemId}
/bridgeChannels/{channelId}
/channels/{channelId}   (legacy Core data)
```

---

### Tenant Firestore (Data Plane)

Each tenant project stores its own internal chat data.

#### Collections

```text
/channels/{channelId}
/channels/{channelId}/members/{uid}
/channels/{channelId}/messages/{messageId}
```

👉 Each tenant is isolated at the project level

---

### 2.3 Core Collections

#### `coreTenants/{tenantId}`

Stores tenant configuration:

- displayName  
- status  
- dataHome  
- clientConfig  

Used to determine:

- where tenant data lives (Core vs tenant project)  
- whether tenant authentication is required  

---

#### `coreMemberships/{uid}`

Stores user-to-tenant relationships:

- tenantIds  
- rolesByTenant  
- defaultTenantId  

Used for:

- access control  
- tenant discovery  
- token minting validation  

---

#### `coreUserInboxes/{uid}/items/{itemId}`

This is the **unified inbox index**.

Each item contains:

- sourceType (`tenant | bridge | legacyCore`)  
- tenantId  
- channelId  
- title  
- photoURL  
- lastMessage  
- lastMessageAt  
- lastMessageSenderId  
- unreadCount  
- lastReadAt  
- updatedAt  
- optional pinned / muted  

### Why Inbox Exists

Without this:

- client must query multiple Firebase projects  
- sorting becomes inconsistent  
- unread state becomes unreliable  

👉 Inbox acts as the **single source of truth for channel list UI**

### Inbox ID Strategy

```text
tenant_${tenantId}_${channelId}
legacy_${tenantId}_${channelId}
bridge_core_${channelId}
```

This enables stable routing across:

- tenant  
- legacy Core  
- bridge  

### Bridge Data

Stored in Core:

```text
/bridgeChannels/{channelId}
/bridgeMessages/{channelId}/items/{messageId}
```

Used for:

- cross-company communication  
- shared channels across tenants  

---

### 2.4 Tenant Collections

#### `channels/{channelId}`

Stores:

- type (`dm | group`)  
- memberIds  
- name / photo  
- dmKey (for uniqueness)  
- lastMessage metadata  
- createdAt  

---

#### `channels/{channelId}/members/{uid}`

Stores:

- unread count  
- joinedAt  
- lastReadAt  

---

#### `channels/{channelId}/messages/{messageId}`

Stores:

- senderId  
- text / attachments  
- audio metadata  
- createdAt  
- edit/delete state  
- push delivery summary  

---

### 2.5 Source of Truth vs Derived Data

#### Source of Truth

- messages  
- channels  
- membership  
- user profiles  
- tenant registry  

#### Derived / Indexed Data

- coreUserInboxes  
- unread counters  
- last message preview  

👉 Inbox is a denormalized index maintained by backend logic

---

## 3. Cloud Functions

### 3.1 Overview

Cloud Functions act as the backend orchestration layer.

They handle:

- tenant discovery  
- tenant token minting  
- inbox indexing and unread updates  
- membership resolution  
- migration tooling  
- media processing  

---

### 3.2 Tenant Access

#### `getMyTenants`

- Type: callable  
- Runs in: Core  

**Reads:**

```text
/coreMemberships/{uid}
/coreTenants/{tenantId}
```

**Returns:**

- tenantIds  
- roles  
- default tenant  
- tenant configs  

---

#### `mintTenantToken(tenantId)`

- Type: callable  
- Runs in: Core  

**Responsibilities:**

- validate membership  
- initialize tenant Admin SDK  
- mint custom token  

**Claims:**

- tenantId  
- role  
- isBridgeUser  

👉 Enables secure cross-project authentication

---

### 3.3 Inbox Indexing

#### `upsertDmInbox`

- creates/updates DM inbox entries for both users  

---

#### `upsertGroupInbox`

- creates/updates group inbox entries for all members  

---

#### `updateCoreInboxAfterSend`

**Updates:**

- lastMessage  
- timestamps  
- unread counts  

**Fan-out logic:**

- sender → unread = 0  
- others → increment unread  

---

#### `markInboxItemRead`

**Updates:**

- unreadCount = 0  
- lastReadAt  

👉 These functions make the Core inbox authoritative

---

### 3.4 Member Resolution

#### `getChannelMembersWithProfiles`

- merges membership + user profile  
- uses Core user data as source of truth  

---

### 3.5 Media Processing

#### `transcodeWebmToM4a`

- Type: storage trigger  

**Flow:**

1. upload `.webm`  
2. transcode → `.m4a`  
3. upload converted file  
4. delete original  
5. update Firestore message  

**Fields updated:**

- attachmentURL  
- audioMime  
- audioStoragePath  

---

## 4. Core Design Principles

### 4.1 Core as Control Plane

Core manages:

- identity  
- tenant access  
- routing  
- inbox index  

---

### 4.2 Tenant as Data Plane

Tenants store:

- actual chat data  

---

### 4.3 Inbox as Index

Inbox:

- replaces multi-query client logic  
- centralizes unread state  
- enables cross-project routing  

---

### 4.4 Function-Orchestrated Consistency

Cloud Functions:

- enforce correctness  
- reduce client complexity  
- prevent race conditions  

---

👉 Backend logic ensures system reliability and consistency

