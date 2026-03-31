# Unread Count Strategy

Unread state is maintained in the **Core inbox index** on a per-user, per-channel basis.

---

## Data Model

Each inbox item may include:

- `channelId`
- `sourceType`
- `tenantId`
- `lastMessageAt`
- `lastReadAt`
- `lastReadMessageId`
- `unreadCount`
- `updatedAt`

---

## Design Principles

- Message history in **Tenant / Bridge** is authoritative  
- Unread count in **Core inbox** is derived state  
- Read progress is tracked per user, per channel  
- Backend owns unread count updates for consistency  

---

## Write Flow

When a new message is created:

1. Message is written to Tenant or Bridge storage  
2. Cloud Function triggers on message create  
3. Backend identifies channel members  
4. For each member except sender:
   - increment `unreadCount`
   - update `lastMessageAt`
   - update preview metadata  
5. For sender:
   - keep `unreadCount = 0`
   - advance read cursor to current message  

---

## Read Flow

When a user opens a channel or marks it as read:

1. Client sends read acknowledgement  
2. Backend or trusted write updates inbox item:
   - `lastReadAt = latestSeenMessageAt`
   - `lastReadMessageId = latestSeenMessageId`
   - `unreadCount = 0`

---

## Multi-Device Behavior

- unread state is stored centrally in Core inbox  
- updates from any device affect all sessions  
- ensures consistent unread counts across devices  

---

## Unread Rules

- sender never receives unread increment for own message  
- unread count should never be negative  
- read action is idempotent  
- UI badge count comes from Core inbox, not message scans  

---

## Failure Handling

Unread count is treated as **rebuildable derived state**.

If unread count becomes inconsistent:

- message history remains authoritative  
- inbox state can be recalculated from:
  - latest message pointer  
  - read cursor  
  - message timestamps or IDs  

Recovery options:

- per-user backfill  
- per-channel re-sync  
- admin migration tooling  

---

## Why This Design

This design:

- avoids scanning message history on every channel list load  
- avoids multi-project unread aggregation on the client  
- keeps unread retrieval to a single Core query  
- supports consistent unread state across multiple devices  
