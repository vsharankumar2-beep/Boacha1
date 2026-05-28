# Schema Fix Quick Reference

## The Problem
```
Error: Failed to save chat session: Could not find the 'session_id' column of 'chat_sessions'
```

## One-Minute Fix

### Step 1: Check Status
```bash
npm run schema:check
```

### Step 2: Add Missing Columns (if needed)

Go to **Supabase Dashboard** → **SQL Editor** → paste this:

```sql
ALTER TABLE chat_sessions
ADD COLUMN session_id TEXT NULL,
ADD COLUMN topic_label TEXT NULL;

CREATE INDEX idx_chat_sessions_session_id ON chat_sessions(session_id);
CREATE INDEX idx_chat_sessions_topic_label ON chat_sessions(topic_label);
```

### Step 3: Restart Server
```bash
npm run dev
```

## That's It! 🎉

The system will now:
- ✅ Save chat history
- ✅ Track conversation sessions
- ✅ Detect conversation topics
- ✅ Work perfectly with memory systems

## Without the Fix?

Still works fine! ✅
- Chat history saves normally
- Conversation resume won't track sessions
- Topic detection won't be stored
- Everything else works as expected

## Logs to Expect

**If columns were missing (now fixed):**
```
[SCHEMA] session_id column missing from chat_sessions. Future inserts will skip it.
[SCHEMA] topic_label column missing from chat_sessions. Future inserts will skip it.
```

Then after adding columns and restarting:
```
✓ Boa AI server is running on port 4000
  Environment: development
  Ready to accept connections
```

## Need More Info?

- [SCHEMA_MIGRATION.md](./SCHEMA_MIGRATION.md) - Complete SQL guide
- [SCHEMA_FIX_SUMMARY.md](./SCHEMA_FIX_SUMMARY.md) - Detailed troubleshooting
- `npm run schema:check` - Check current status
