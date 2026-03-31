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

- Developed a production flight booking mobile application  
- Integrated with existing PHP + MySQL backend  
- Designed complete booking workflow:
  search → results → booking → confirmation  

- Implemented authentication, analytics, and environment-based builds  
- Delivered cross-platform app using Expo (iOS + Android)

---

### GChat

- Designed and built a Slack-like internal communication platform  
- Implemented a **multi-tenant architecture using multiple Firebase projects**  
- Developed a **unified inbox indexing system** to support cross-tenant channel listing  
- Built real-time messaging with unread synchronization and presence  

Platforms:

- iOS (React Native)
- Android (React Native)
- Web (Firebase SDK)

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

- Designing a multi-tenant system without breaking existing users  
- Maintaining consistent unread state across multiple Firebase projects  
- Handling derived state (inbox, unread) with eventual consistency  
- Coordinating frontend simplicity with backend-driven correctness 

---

## 6. What I Would Improve

- Introduce observability (logging, tracing, alerting) for backend workflows  
- Optimize write fan-out for large channels  
- Improve tenant bootstrapping latency on frontend  
- Add repair tooling for derived state (inbox backfill, unread re-sync)  

---

## 7. Engineering Profile

I design and build **end-to-end systems** across mobile and backend, with a focus on:

- multi-tenant architecture  
- real-time systems  
- frontend–backend integration  

I have experience:

- evolving systems without full rewrites  
- designing derived state models (inbox, unread)  
- balancing client simplicity with backend consistency  

👉 Currently expanding into deeper areas of scalability, reliability, and distributed system design

---

## 8. Core Strength

- Designing multi-tenant systems with clear separation of concerns  
- Building real-time applications with derived state models  
- Integrating frontend and backend into a cohesive system 
