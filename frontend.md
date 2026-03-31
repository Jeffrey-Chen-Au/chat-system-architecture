#Live Chat — Frontend Architecture

---

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
- Backend handles consistency, frontend stays lightweight  

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

👉 Designed for production use and future expansion
