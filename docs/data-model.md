# Data Model Overview

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
