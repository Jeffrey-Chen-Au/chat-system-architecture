# Live Chat — Frontend Architecture

## 1. Overview

Live Chat App is a multi-platform (iOS, Android, Web) internal communication application designed to support:

- real-time messaging
- multi-tenant communication
- unified inbox experience
- future AI integration

---

## 2. Frontend Stack

- React Native (Expo)
- Expo Router (navigation)
- Firebase Client SDK (Auth, Firestore, Functions)
- Expo Notifications
- Firebase Analytics

---

## 3. Frontend Responsibilities

The frontend is responsible for:

- user authentication and session handling  
- tenant-aware routing and context switching  
- rendering unified inbox  
- rendering chat threads (DM + group)  
- message composition and sending  
- channel and member management  
- invoking backend Cloud Functions  
- handling UI state and user interactions  

---

## 4. App Boot Flow

1. App launches  
2. Restore Firebase authentication session  
3. Load user profile  
4. Fetch tenant memberships (`coreMemberships`)  
5. Determine default tenant / scope  
6. Initialize tenant Firebase app if required  
7. Load unified inbox (`coreUserInboxes`)  
8. Render channel list  

👉 This ensures consistent initialization across tenants

---

## 5. Route Structure

```
app/
  (auth)/
  (tabs)/
  [channelId]/
  profile/
```

### Auth Routes

- login
- reset password

### Tabs

- channels (inbox)
- settings

### Channel Routes

- thread view
- info
- search
- add members
- edit group

### Profile

- user profile view

---

## 6. Messaging Flows

### Open Inbox
- read from Core inbox index  
- display sorted channel list  

### Open Channel
- determine sourceType  
- resolve scope (core / tenant)  
- connect to correct Firestore instance  

### Send Message
- write to Firestore  
- trigger backend for inbox update  

### Create Chat / Group
- create channel  
- update inbox index  

---

## 7. Tenant-Aware Client Behavior

- Core is always initialized first  
- Tenant apps are initialized on demand  
- Same UID is used across projects  
- Routing is based on `scope` abstraction  

```
{ kind: "core" }
{ kind: "tenant", tenantId }
```

👉 Prevents duplication of logic across tenants

---

## 8. Design Considerations

- Unified inbox avoids multi-query complexity  
- Scope-based routing simplifies multi-project handling  
- Route grouping improves modularity  
- Backend owns all critical state transitions, allowing the frontend to remain stateless and reactive

---

## 9. Frontend Complexity / Future Improvements

### Current Complexity

- multi-project Firebase initialization  
- async tenant bootstrapping  
- unread sync coordination  
- routing based on dynamic scope  

### Future Improvements

- clearer state management strategy  
- improved caching for inbox and threads  
- better error handling for tenant switching  
- performance optimization for large channel lists  

---

## 10. Summary

The frontend:

- is modular and scalable  
- supports multi-tenant architecture  
- delegates consistency to backend  
- focuses on UI and interaction  

👉 Designed to support multi-tenant scaling while keeping client logic simple and maintainable

---

## 11. Frontend Data Flow

The frontend follows a **read-from-Core, route-to-scope, write-to-source** pattern.

### Inbox Flow

1. Query Core:
   `/coreUserInboxes/{uid}/items`
2. Render channel list from derived index
3. No direct message queries across tenants

### Thread Flow

1. User opens channel from inbox
2. Resolve:
   - `sourceType`
   - `tenantId`
3. Determine scope:
   - Core → use Core Firestore
   - Tenant → ensure tenant app initialized
4. Subscribe to:
   `/channels/{channelId}/messages`
5. On channel change or screen exit:
   - unsubscribe from previous message listener
   - release unnecessary subscriptions

👉 Prevents memory leaks and unnecessary real-time listeners

### Send Message Flow

1. Client writes message to:
   - Core (bridge / legacy)
   - Tenant Firestore
2. No client-side unread calculation
3. Backend updates inbox + unread asynchronously

👉 Frontend does not maintain derived state manually

---

## 12. State Management Strategy

The frontend uses a **server-driven state model**.

### Principles

- Firestore is the primary source of UI state  
- minimal local state is stored on the client  
- UI reacts to real-time updates via subscriptions  

### Local State

- input state (message composer)
- UI state (modals, loading, selection)
- temporary optimistic updates (optional)

### Remote State

- inbox list
- messages
- membership
- unread counts

👉 Derived state (unread, ordering) is not computed on the client

---

## 13. Error Handling and Edge Cases

### Tenant Initialization Failure

- fallback to retry mechanism
- user remains in Core context

### Network Issues

- Firestore offline persistence (if enabled)
- UI shows loading / retry states

### Stale Inbox Data

- relies on backend eventual consistency
- client does not attempt reconciliation

### Auth Expiry

- Firebase handles token refresh automatically
- client re-initializes tenant sessions if needed

### Tenant Bootstrap Latency

- initial tenant initialization may introduce delay
- UI should:
  - show loading state during tenant auth
  - prevent duplicate initialization
  - cache initialized tenant apps when possible

👉 Ensures smooth transition when entering tenant-scoped channels

---

## 14. Performance Considerations

- Inbox uses single Core query (O(1))
- Avoids querying multiple tenant databases
- Message subscriptions are scoped per channel
- Lazy initialization of tenant apps reduces startup cost

Potential bottlenecks:

- large channel message lists
- tenant switching latency
- multiple active subscriptions
