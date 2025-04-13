# SCHEMA.md

## Overview

This schema is designed for a simplified Facebook Messenger backend, optimized for Apache Cassandra. The following use cases are supported:

- Sending messages between users
- Fetching user conversations ordered by recent activity
- Fetching all messages in a conversation
- Fetching messages before a given timestamp (for pagination)

---

## Tables

### 1. `messages_by_conversation`

Stores all messages in a conversation.

```sql
CREATE TABLE IF NOT EXISTS messages_by_conversation (
    conversation_id UUID,
    timestamp TIMESTAMP,
    message_id UUID,
    sender_id UUID,
    recipient_id UUID,
    content TEXT,
    PRIMARY KEY ((conversation_id), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

**Purpose:**

- Fetch all messages in a conversation
- Fetch messages before a given timestamp (for pagination)

---

### 2. `conversations_by_user`

Stores a user's conversations, ordered by most recent activity.

```sql
CREATE TABLE IF NOT EXISTS conversations_by_user (
    user_id UUID,
    last_updated TIMESTAMP,
    conversation_id UUID,
    participant_id UUID,
    last_message TEXT,
    PRIMARY KEY ((user_id), last_updated, conversation_id)
) WITH CLUSTERING ORDER BY (last_updated DESC);
```

**Purpose:**

- Fetch conversations for a user, ordered by recent activity
- Supports fast lookup of latest conversations

---

### 3. `messages_by_user_pair`

Stores direct messages between two users (useful for filtering one-to-one conversations).

```sql
CREATE TABLE IF NOT EXISTS messages_by_user_pair (
    user1_id UUID,
    user2_id UUID,
    timestamp TIMESTAMP,
    message_id UUID,
    content TEXT,
    sender_id UUID,
    PRIMARY KEY ((user1_id, user2_id), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

**Purpose:**

- Optional: Optimized for direct messaging between specific user pairs
- Can be used for advanced filters or analytics

---

## Notes

- UUIDs are time-based (e.g., using `uuid1()` in Python) to preserve ordering.
- Timestamps are used as clustering keys for message ordering and pagination.
- Any updates to conversations (e.g., sending new messages) also update `conversations_by_user`.
