# Supabase Schema Migration Guide

## Issue

The Boa AI backend expects the `chat_sessions` table to have these columns:

### Required Columns
- `user_id` (text)
- `user_input` (text)
- `ai_response` (text)
- `created_at` (timestamp)

### Optional Columns (for enhanced functionality)
- `session_id` (text) - Tracks conversation sessions for resume functionality
- `topic_label` (text) - Stores detected conversation topics for better memory retrieval

## Current Error

```
Failed to save chat session: Could not find the 'session_id' column of 'chat_sessions'
```

This means the `session_id` or `topic_label` columns are missing from your table.

## Solution

### Option 1: Automatic Detection (Recommended)

The backend now automatically detects missing columns and gracefully handles them:

1. **On first insert**, if a column is missing, the system logs a warning
2. **All future inserts** skip that column and save successfully
3. **No manual SQL needed** - the system adapts to your schema

Example log:
```
[SCHEMA] session_id column missing from chat_sessions. Future inserts will skip it.
```

### Option 2: Add Missing Columns (For Full Functionality)

If you want the conversation resume and topic detection features working optimally, add these columns:

#### Add session_id column

```sql
ALTER TABLE chat_sessions
ADD COLUMN session_id TEXT NULL;

-- Optional: Create an index for faster lookups
CREATE INDEX idx_chat_sessions_session_id ON chat_sessions(session_id);
```

#### Add topic_label column

```sql
ALTER TABLE chat_sessions
ADD COLUMN topic_label TEXT NULL;

-- Optional: Create an index for topic-based queries
CREATE INDEX idx_chat_sessions_topic_label ON chat_sessions(topic_label);
```

#### Add both together

```sql
ALTER TABLE chat_sessions
ADD COLUMN session_id TEXT NULL,
ADD COLUMN topic_label TEXT NULL;

-- Create indexes for performance
CREATE INDEX idx_chat_sessions_session_id ON chat_sessions(session_id);
CREATE INDEX idx_chat_sessions_topic_label ON chat_sessions(topic_label);
```

### Option 3: Create Full Schema (If Starting Fresh)

If your `chat_sessions` table doesn't exist yet:

```sql
CREATE TABLE chat_sessions (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  user_id TEXT NOT NULL,
  user_input TEXT NOT NULL,
  ai_response TEXT NOT NULL,
  session_id TEXT NULL,
  topic_label TEXT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- Enable Row Level Security if needed
  -- CONSTRAINT fk_user_id FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Create indexes for common queries
CREATE INDEX idx_chat_sessions_user_id ON chat_sessions(user_id);
CREATE INDEX idx_chat_sessions_created_at ON chat_sessions(created_at DESC);
CREATE INDEX idx_chat_sessions_session_id ON chat_sessions(session_id);
CREATE INDEX idx_chat_sessions_topic_label ON chat_sessions(topic_label);

-- Enable Row Level Security (optional but recommended)
ALTER TABLE chat_sessions ENABLE ROW LEVEL SECURITY;
```

## How to Run SQL in Supabase

1. Go to **Supabase Dashboard** → Your Project → **SQL Editor**
2. Click **New Query**
3. Paste the SQL from above
4. Click **Run**

Or use the **Supabase CLI**:

```bash
# Run a migration file
supabase db execute < migration.sql

# Or directly:
supabase db execute --file migration.sql
```

## Verify Schema Changes

To check if your columns were added successfully:

```sql
-- View table structure
SELECT * FROM information_schema.columns 
WHERE table_name = 'chat_sessions'
ORDER BY ordinal_position;
```

Or use Supabase Dashboard:
1. Go to **SQL Editor**
2. Click the table name in the left sidebar
3. Click **Columns** tab to see all columns

## Backend Schema Check

The backend now includes a schema check function. To use it:

```javascript
import { checkChatSessionsSchema } from './supabaseClient.js';

const schemaStatus = await checkChatSessionsSchema();
console.log(schemaStatus);
// Output:
// {
//   status: 'ok' | 'warning' | 'error',
//   availableColumns: [...],
//   hasSessionId: true,
//   hasTopicLabel: true,
//   missingOptional: []
// }
```

## Impact of Missing Columns

### Without session_id
- ❌ Conversation resume won't track sessions
- ❌ Can't correlate related messages
- ✅ Basic chat history still works

### Without topic_label
- ❌ Topic detection won't be stored
- ❌ Topic-based memory retrieval less effective
- ✅ Semantic memory retrieval still works

### Without either
- ✅ Basic chat history saves and retrieves correctly
- ✅ Memory, emotion, personality systems still work
- ✅ AI responses continue normally
- ⚠️ Conversation resume and topic detection disabled

## Rollback

If you need to remove the columns (not recommended):

```sql
ALTER TABLE chat_sessions
DROP COLUMN session_id;

ALTER TABLE chat_sessions
DROP COLUMN topic_label;
```

## FAQ

**Q: Will my existing chat history break?**
A: No. The system gracefully adapts to your schema.

**Q: Do I need both new columns?**
A: No. Each is independent. Add based on features you need.

**Q: Can I add columns later?**
A: Yes. The backend detects and uses them once available.

**Q: What if the table doesn't exist?**
A: Run Option 3 to create it with all columns.

**Q: How do I know if it's working?**
A: Check server logs for `[SCHEMA]` messages. No errors = success.
