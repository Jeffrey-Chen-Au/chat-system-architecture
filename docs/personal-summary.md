# Personal Engineering Summary — GChat & Gaura Travel

---

## 1. Overview

I am a mobile engineer specializing in building real-world applications using React Native (Expo) and Firebase.

I have designed and delivered two production-level applications:

- **Gaura Travel App** — flight booking platform  
- **GChat** — multi-tenant real-time chat system  

---

## 2. What I Built

### Gaura Travel

- Mobile flight booking app (India ↔ Australia focus)
- Integrated with existing PHP + MySQL backend
- Built full booking flow:
  - search → results → details → booking → passenger → confirmation
- Implemented Firebase authentication and analytics
- Managed environment-based builds (EAS)

---

### GChat

- Slack-like internal communication platform
- Supports:
  - direct messaging
  - group chats
  - multi-company (tenant) architecture
- Built across iOS, Android, and Web

---

## 3. Key Engineering Work

### Multi-Tenant Architecture

- Designed system evolution from single Firebase project → multi-tenant system
- Introduced:
  - Core (control plane)
  - Tenant projects (data plane)
  - Bridge layer (cross-company communication)

---

### Unified Inbox System

- Designed `/coreUserInboxes` as centralized index
- Solved:
  - multi-project querying complexity
  - unread consistency
  - sorting issues

---

### Tenant Authentication Model

- Implemented Core-first authentication
- Built token minting flow using Cloud Functions
- Enabled same UID across multiple Firebase projects

---

### Frontend Architecture

- Built modular routing using Expo Router
- Introduced scope-based routing:
  ```
  { kind: "core" }
  { kind: "tenant", tenantId }
  ```
- Enabled dynamic switching between Firebase projects

---

### Backend Coordination

- Designed Cloud Functions for:
  - inbox indexing
  - unread updates
  - tenant access
  - token minting

---

## 4. Key Decisions & Trade-offs

### Why Unified Inbox

- Avoid multi-database queries on client
- Centralize unread logic
- Improve performance and consistency

---

### Why Core + Tenant Separation

- Enforce tenant isolation
- Enable scalable onboarding of new companies
- Simplify security boundaries

---

### Why Backend-Controlled Consistency

- Prevent race conditions
- Avoid duplicated client logic
- Ensure reliable unread and ordering behavior

---

## 5. Challenges Faced

- Managing multi-project Firebase initialization
- Designing consistent unread logic across tenants
- Migrating from single-project to multi-tenant without downtime
- Keeping frontend simple while backend handles complexity

---

## 6. What I Would Improve

- Add monitoring and alerting (SLOs, logging)
- Improve cost awareness and scaling strategy
- Introduce clearer state management on frontend
- Enhance error handling for tenant switching

---

## 7. Engineering Level

I currently operate at a **strong mid-level engineer** level:

- capable of designing real systems  
- able to implement end-to-end features  
- understands architecture and trade-offs  

Next growth focus:

- scalability and performance at larger scale  
- production reliability and observability  
- deeper system design reasoning  

---

## 8. Summary

I have built:

- production mobile apps  
- a multi-tenant real-time system  
- cross-platform architecture  

My strength lies in:

- system design thinking  
- frontend + backend integration  
- building scalable foundations  

👉 Currently progressing toward senior-level engineering
