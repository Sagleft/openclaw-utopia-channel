# OpenClaw Utopia Channel Plugin

Adds [Utopia](https://u.is/) messenger support to OpenClaw, enabling you to send and receive messages through the decentralized Utopia network.

## Prerequisites

1. **Utopia Desktop Client** installed and running on the same machine (or accessible via network)
   - Download from: https://u.is/
   - Only the desktop (PC) version supports the API

2. **Utopia API enabled** in the client settings:
   - Open Utopia → Settings → API
   - Enable API access
   - Note the **Host** (default: `127.0.0.1`), **Port** (default: `22659`), and **Token**
   - See: https://udocs.gitbook.io/utopia-api/utopia-api/how-to-enable-api-access

3. **OpenClaw** installed and configured
   - See: https://docs.openclaw.ai/start/getting-started

## Installation

### Option 1: Install from npm (recommended)

```bash
openclaw plugin install @openclaw/utopia
```

### Option 2: Install from local source

```bash
# Clone the OpenClaw repo (if not already)
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Install dependencies for the utopia extension
cd extensions/utopia
npm install --omit=dev
```

## Configuration

Add the following to your OpenClaw config (`~/.openclaw/config.json`):

```json
{
  "channels": {
    "utopia": {
      "enabled": true,
      "host": "127.0.0.1",
      "port": 22659,
      "apiToken": "YOUR_UTOPIA_API_TOKEN",
      "wsPort": 25000,
      "dmPolicy": "allowlist",
      "allowFrom": ["ADMIN_PUBLIC_KEY_HERE"]
    }
  }
}
```

Or use the CLI:

```bash
openclaw config set channels.utopia.enabled true
openclaw config set channels.utopia.host "127.0.0.1"
openclaw config set channels.utopia.port 22659
openclaw config set channels.utopia.apiToken "YOUR_UTOPIA_API_TOKEN"
openclaw config set channels.utopia.wsPort 25000
openclaw config set channels.utopia.dmPolicy "allowlist"
openclaw config set channels.utopia.allowFrom '["YOUR_PUBLIC_KEY"]'
```

### Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enabled` | boolean | `true` | Enable/disable the Utopia channel |
| `host` | string | `127.0.0.1` | Utopia API host address |
| `port` | number | `22659` | Utopia API port |
| `apiToken` | string | — | API token from Utopia settings (required) |
| `wsPort` | number | `25000` | WebSocket port for receiving notifications |
| `useSsl` | boolean | `false` | Use HTTPS/WSS instead of HTTP/WS |
| `dmPolicy` | string | `pairing` | Access policy: `pairing`, `allowlist`, `open`, or `disabled` |
| `allowFrom` | string[] | `[]` | List of allowed sender public keys |

### Access Policies

- **`pairing`** (default): New users must be approved before they can interact. They send a message, you approve their pubkey.
- **`allowlist`**: Only pubkeys listed in `allowFrom` can interact.
- **`open`**: Anyone can send messages (not recommended for production).
- **`disabled`**: Ignore all incoming messages.

### Finding Your Public Key

In the Utopia client:
1. Go to your profile
2. Your public key (pk) is displayed there — it's a 64-character hex string

## How It Works

1. The plugin connects to your local Utopia client via its HTTP API
2. It enables WebSocket notifications and listens for incoming messages
3. When a message arrives from an authorized user, it forwards the text to OpenClaw for processing
4. OpenClaw's response is sent back to the user via `sendInstantMessage`

### Message Flow

```
User (Utopia) → WebSocket Event → Plugin → OpenClaw → Plugin → sendInstantMessage → User (Utopia)
```

## Verifying the Connection

After configuration, check the channel status:

```bash
openclaw channels status --probe
```

This calls `getSystemInfo()` on the Utopia client to verify connectivity.

## Troubleshooting

### "Utopia API token not configured"
- Ensure `apiToken` is set in your config

### WebSocket disconnects / reconnects
- The plugin automatically reconnects with exponential backoff (5s → 60s max)
- Check that the `wsPort` doesn't conflict with other services
- Ensure the Utopia client is running

### "Utopia API HTTP error: 403"
- Verify your API token is correct
- Check that the API is enabled in Utopia settings

### Messages not being received
- Verify `dmPolicy` is not set to `disabled`
- Check that the sender's pubkey is in `allowFrom` (if using `allowlist` policy)
- Try setting WebSocket notifications to `all` for debugging:
  - This can be done temporarily by modifying the `setWebSocketState` call

## Development

```bash
# From the OpenClaw repo root
pnpm install

# Run tests
pnpm test -- extensions/utopia

# Type check
pnpm build
```
