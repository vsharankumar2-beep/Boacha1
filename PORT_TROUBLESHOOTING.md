# Port Conflict Troubleshooting

## Problem: EADDRINUSE: address already in use :::4000

This error occurs when another process is already using port 4000.

## Solutions

### Quick Fix (Automated)

Use the automatic port-freeing script:

```bash
npm run port:free
```

Then start the server:

```bash
npm run dev
```

The `dev` script automatically frees port 4000 before starting the server.

### Manual Fix

**Step 1: Check what's using port 4000**

```bash
lsof -i :4000
```

Or on systems without `lsof`:

```bash
netstat -tlnp | grep :4000
```

**Step 2: Kill the process**

Replace `<PID>` with the actual process ID from the output above:

```bash
kill -9 <PID>
```

**Step 3: Start the server**

```bash
npm start
```

### Use a Different Port

If you need to keep the other process running, use a different port:

```bash
PORT=3000 npm start
```

## Available npm Scripts

- **`npm start`** - Start the server (manual port handling required)
- **`npm run dev`** - Start the server with automatic port conflict resolution
- **`npm run port:check`** - Check which process is using port 4000
- **`npm run port:free`** - Free port 4000 by killing the process using it

## Enhanced Server Error Handling

The server now includes:

✅ **Automatic error detection** - Catches EADDRINUSE errors immediately
✅ **Helpful error messages** - Clear instructions on how to resolve the issue
✅ **Graceful shutdown** - Properly handles SIGTERM and SIGINT signals
✅ **Permission error handling** - Suggests alternative ports if privileges are insufficient
✅ **Enhanced logging** - Shows environment and readiness status on startup

## Server Startup Output

### Success

```
✓ Boa AI server is running on port 4000
  Environment: development
  Ready to accept connections
```

### Port In Use Error

```
✗ Port 4000 is already in use
  Another Boa AI instance or process is running on this port
  Fix: Run "lsof -i :4000" to find the process, then kill it
```

## Prevention

To prevent this issue in the future:

1. Always use `npm run dev` during development (includes auto port-freeing)
2. Use graceful shutdown (Ctrl+C instead of killing the terminal)
3. Don't run multiple instances of the server on the same port
4. Consider using Docker for isolated environments
