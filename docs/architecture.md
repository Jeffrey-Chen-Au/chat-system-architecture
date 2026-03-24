# Architecture

## Overview

The system evolves from a **single Firebase project** into a **multi-tenant architecture**.

---

## Core Design

### Core (Control Plane)
- authentication
- tenant registry
- membership
- unified inbox
- bridge messaging

---

### Tenant (Data Plane)
Each company has its own Firebase project:
- users
- channels
- messages

---

### Bridge
- cross-company communication
- managed in Core

---

## Unified Inbox
