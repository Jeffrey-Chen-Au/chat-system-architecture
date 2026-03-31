# Multi-Tenant Chat System Architecture

> ⚡ This repository showcases the architecture and design of a real-world, Slack-like chat system, adapted for public sharing.

---

## 🚀 Overview

This project documents the design of a **multi-tenant chat platform** built using:

- React Native (Expo)
- Firebase (Auth, Firestore, Cloud Functions)

The system was originally implemented as a **single Firebase project** and later evolved into a **multi-tenant architecture** to support multiple companies with proper data isolation and scalability.

---

## 🧠 Key Highlights

- multi-tenant architecture to support multiple companies while maintaining data isolation and a consistent user experience.
- Unified inbox indexing system (single-query channel list across tenants, avoids multi-database queries)
- Scope-based routing across multiple Firebase projects
- Backend-controlled consistency (Cloud Functions)
- Incremental migration strategy (no full rewrite)

---

## 📱 UI Context (Reference Only)

These screenshots provide context for how the system design maps to real user interactions.

They are included to support the architecture explanation, not as a product showcase.

### Unified Inbox (Core Index)

<img src="./assets/gchat-inbox.png" width="300"/>

### Chat Thread (Tenant/Core Routing)

<img src="./assets/gchat-thread.png" width="300"/>

---

## 📚 Documentation

- [Architecture](./docs/architecture.md)
- [Firebase Backend](./docs/firebase-backend.md)
- [Frontend](./docs/frontend.md)
- [Personal Engineering Summary](./docs/personal-summary.md)

---

## ⚙️ Tech Stack

**Frontend**
- React Native (Expo)
- Expo Router

**Backend**
- Firebase Auth
- Firestore
- Cloud Functions

---

## ⚠️ Disclaimer

This repository contains **architecture and design documentation only**.

- No proprietary company code is included  
- No sensitive data is exposed  

---

## 👤 Author

Mobile engineer with experience building:
- real-time chat systems  
- multi-tenant architectures  
- React Native + Firebase apps  
