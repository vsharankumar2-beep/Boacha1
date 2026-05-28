# Supabase Schema Mismatch - Complete Troubleshooting Guide

## Problem Summary

**Error**: `Failed to save chat session: Could not find the 'session_id' column of 'chat_sessions'`

This error occurs when the Boa AI backend tries to insert optional columns (`session_id`, `topic_label`) that don't exist in your Supabase `chat_sessions` table.

## What Changed

The backend now includes **intelligent schema adaptation** that:

1. ✅ **Auto-detects** missing columns on first insert
2. ✅ **Gracefully falls back** to available columns
3. ✅ **Prevents crashes** - system continues working
4. ✅ **Logs warnings** - tells you what's missing
5. ✅ **Caches detection** - no repeated attempts

### Example Behavior

```javascript
// First insert attempt (all fields)
INSERT INTO chat_sessions (user_id, user_input, ai_response, created_at, session_id, topic_label)
// ❌ Fails: "column 'session_id' does not exist"

// System logs: "[SCHEMA] session_id column missing from chat_sessions. Future inserts will skip it."

// Future inserts (without session_id)
INSERT INTO chat_sessions (user_id, user_input, ai_response, created_at, topic_label)
// ✅ Success!
```

## Quick Fix

### 1. Check Your Current Schema

```bash
npm run schema:check
```

Output example:
```
✅ chat_sessions Table
   Status: WARNING
   Message: Schema exists but missing optional features
   
   Available Columns:
     • id
     • user_id
     • user_input
     • ai_response
     • created_at
   
   ⚠️ Missing OPTIONAL columns:
     • session_id
     • topic_label
```

### 2. Add Missing Columns (Optional but Recommended)

**In Supabase SQL Editor**, run:

```sql
-- Add session_id for conversation tracking
ALTER TABLE chat_sessions
ADD COLUMN session_id TEXT NULL;

-- Add topic_label for conversation topics
ALTER TABLE chat_sessions
ADD COLUMN topic_label TEXT NULL;

-- Create indexes for performance
CREATE INDEX idx_chat_sessions_session_id ON chat_sessions(session_id);
CREATE INDEX idx_chat_sessions_topic_label ON chat_sessions(topic_label);
```

### 3. Restart Server

```bash
npm run dev
# or
npm start
```

## Architecture

### Progressive Fallback Strategy

```
Try Attempt 1
├─ Insert with: user_id, user_input, ai_response, created_at, session_id, topic_label
└─ ❌ Column missing? Try Attempt 2

Try Attempt 2
├─ Detect which column failed
├─ If session_id missing:
│  └─ Insert with: user_id, user_input, ai_response, created_at, topic_label
├─ If topic_label missing:
│  └─ Insert with: user_id, user_input, ai_response, created_at, session_id
└─ ❌ Still fails? Try Attempt 3

Try Attempt 3
├─ Insert with: user_id, user_input, ai_response, created_at
└─ ✅ Success (base schema)

Result
└─ Cache schema compatibility for all future inserts
```

### New Exports in supabaseClient.js

```javascript
// Existing (unchanged)
export { insertChatSession, fetchChatHistory, fetchRelevantChatHistory }

// New - Check schema compatibility
export async function checkChatSessionsSchema() {
  return {
    status: 'ok' | 'warning' | 'error',
    availableColumns: [...],
    missingRequired: [...],
    missingOptional: [...],
    hasSessionId: boolean,
    hasTopicLabel: boolean
  }
}
```

## Impact on Features

### With All Columns (Optimal)

```
✅ Chat history saving
✅ Conversation resume (session_id tracking)
✅ Topic-based memory retrieval (topic_label)
✅ Full semantic memory system
✅ Emotion/bond/personality systems
```

### With Only Base Columns (Fallback)

```
✅ Chat history saving
❌ Conversation resume (disabled)
❌ Topic tracking (disabled)
✅ Basic memory retrieval
✅ Emotion/bond/personality systems
✅ AI responses work normally
```

## Testing

### 1. Verify Schema Detection

```bash
npm run schema:check
```

Should show either:
- `✅` with all columns, or
- `⚠️` with missing optional columns

### 2. Test Chat Flow

```bash
# Terminal 1: Start server
npm run dev

# Terminal 2: Send test message
curl -X POST http://localhost:4000/chat \
  -H "Content-Type: application/json" \
  -d '{"userId":"testuser","message":"Hello Boa!"}'
```

Check server logs for schema warnings:
- `[SCHEMA] session_id column missing...` - Expected if column doesn't exist
- `[SCHEMA] topic_label column missing...` - Expected if column doesn't exist
- No [SCHEMA] messages - All columns present ✅

### 3. Check Chat History

```bash
# Should return chat history without errors
curl http://localhost:4000/chat/history?userId=testuser
```

## Files Modified

1. **`boacha/supabaseClient.js`**
   - Enhanced `insertChatSession()` with progressive fallback
   - New `checkChatSessionsSchema()` export
   - Schema tracking with `DETECTED_SCHEMA` cache

2. **`boacha/scripts/check-schema.js`** (NEW)
   - Diagnostic utility to check schema status
   - Run: `npm run schema:check`

3. **`boacha/package.json`**
   - New script: `npm run schema:check`

4. **`SCHEMA_MIGRATION.md`** (NEW)
   - Detailed SQL migration guide
   - Multiple options for schema setup

## Backward Compatibility

✅ **All existing code continues to work**
- Chat routes unmodified
- Memory/emotion/bond/personality systems unchanged
- APIs remain the same
- Only internal DB insertion logic improved

## FAQ

**Q: Will this break my existing chat history?**
A: No. Only new inserts adapt to available schema.

**Q: Do I need to add the columns?**
A: No. System works with or without them.

**Q: What if I add columns later?**
A: Restart server, it will re-detect and use them.

**Q: Can I remove columns?**
A: Yes, system will adapt. Not recommended as it disables features.

**Q: What does "[SCHEMA]" in logs mean?**
A: System detected a missing column but continued normally.

**Q: How do I know if it's working?**
A: No error in chat response = success.

## Support

If you're still experiencing issues:

1. Run `npm run schema:check` to see detailed status
2. Check server logs for `[SCHEMA]` messages
3. Verify environment variables: `SUPABASE_URL`, `SUPABASE_ANON_KEY`
4. Ensure `chat_sessions` table exists
5. Check Supabase dashboard for table structure

## References

- [SCHEMA_MIGRATION.md](./SCHEMA_MIGRATION.md) - SQL migration guide
- [PORT_TROUBLESHOOTING.md](./PORT_TROUBLESHOOTING.md) - Port conflict guide
