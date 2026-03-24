# Multi-Tenant Chat System Architecture

> ⚡ This repository showcases the architecture and design of a real-world, Slack-like chat system, adapted for public sharing.

---

## 🚀 Overview

This project documents the design of a **multi-tenant chat platform** built using:

- React Native (Expo)
- Firebase (Auth, Firestore, Cloud Functions)

The system was originally implemented as a **single Firebase project** and later evolved into a **multi-tenant architecture** to support multiple companies with proper data isolation and scalability.

---

## 🧠 Key Design Highlights

### 🔹 Multi-Tenant Architecture

The system is structured into three layers:

- **Core (Control Plane)**
  - authentication
  - tenant registry
  - membership management
  - unified inbox
  - cross-company messaging

- **Tenant Projects (Data Plane)**
  - isolated chat data per company
  - users, channels, messages

- **Bridge Layer**
  - enables cross-company communication
  - managed centrally in Core

---

### 🔹 Unified Inbox System

A centralized inbox index is maintained in Core:
/coreUserInboxes/{uid}/items

This enables:
- a single query for channel list
- consistent unread state
- unified experience across tenants

👉 Eliminates the need for client-side multi-database queries

---

### 🔹 Scope-Based Routing (Frontend)

The frontend uses a scope abstraction:

```ts
{ kind: "core" }
{ kind: "tenant", tenantId }
This allows:

dynamic Firebase project selection
clean separation of logic
scalable multi-tenant support
🔹 Backend-Orchestrated Consistency

Cloud Functions are used to:

discover tenant access
mint tenant auth tokens
maintain inbox index
update unread counts

👉 Ensures consistency and avoids client-side race conditions

🔹 Incremental Migration Strategy

The system was migrated without a full rewrite:

introduced routing abstraction first
added tenant projects
built unified inbox
migrated data in phases
performed safe cutover

👉 Inbox-based routing allowed switching data sources without UI changes

📚 Documentation

Detailed design documents:

Architecture
Firebase Backend
Frontend
Personal Engineering Summary
⚙️ Tech Stack

Frontend

React Native (Expo)
Expo Router

Backend

Firebase Authentication
Firestore
Cloud Functions

Other

Expo Notifications
Firebase Analytics
🎯 What This Demonstrates

This repository highlights:

multi-tenant system design
control-plane vs data-plane separation
real-time messaging architecture
data indexing strategies
frontend-backend coordination
safe production migration approach
⚠️ Disclaimer

This repository contains architecture and design documentation only.

No proprietary company code is included
No sensitive data or credentials are exposed
Implementation details are generalized
👤 About Me

Mobile engineer with experience building:

real-time chat systems
multi-tenant architectures
React Native + Firebase applications
