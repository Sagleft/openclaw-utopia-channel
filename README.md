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

## How it works

Openclaw is an AI agent orchestrator that allows you to automate many tasks.

This plugin allows you to send requests to your openclaw server via the Utopia messenger.

To do this, you will need 2 Utopia accounts: 1 will be on the server (you can install the Desktop version on the server), 1 for communication (for example, the Utopia mobile app - convenient for communicating with Openclaw from a smartphone).

To authorize on the server you will need:
1. Find the public key of a Utopia account on the server.
2. Enable Utopia API on the server and configure access.
3. Install & setup this plugin.
4. Restart openclaw gateway service.

## Installation

### Option 1: Install from npm (recommended)

```bash
openclaw plugin install @sagleft/openclaw-utopia
```

### Option 2: Install with shell script

**macOS / Linux / WSL2:**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows (PowerShell):**
```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

### Option 3: Install from local source

```bash
# Clone the OpenClaw repo (if not already)
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Install dependencies for the utopia extension
cd extensions/utopia
npm install --omit=dev
```

### Option 4: Install dev version

```bash
# get source
git clone https://github.com/Sagleft/openclaw-utopia-channel.git utopia-plugin
cd utopia-plugin

# install dependencies
npm install --omit=dev

# install plugin
openclaw plugins install --link .
```

Check that the plugin is enabled:
```bash
openclaw plugins list
```
utopia should appear in the list

### Openclaw Installation Check

```bash
openclaw --version
openclaw doctor
```

`doctor` will show if there are any installation or configuration issues.

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

- `ADMIN_PUBLIC_KEY_HERE` - this is the public key of your account from which you will send messages.

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
2. Your public key (pk) is displayed there — it's a 64-character hex string.

### How to enable Utopia API

1. Open Tools → Settings → API
2. Toggle Enable API
3. Remember:
- HTTP Port (e.g. 20000)
- Listen IP (e.g. 127.0.0.1)
4. Click Add token, give it a name (e.g. openclaw), and copy the token.

After all the settings are complete, restart the openclaw service:
```bash
openclaw gateway restart
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
- Ensure the Utopia client is running
- Check that wsPort isn't busy with another process:
```bash
lsof -i :25000
```

### "Utopia API HTTP error: 403"
- The token is invalid or expired
- Regenerate the token in Utopia Settings → API and update the configuration

### Messages not being received
- Verify `dmPolicy` is not set to `disabled`
- Check that the sender's pubkey is in `allowFrom` (if using `allowlist` policy)
- Try setting WebSocket notifications to `all` for debugging:
  - This can be done temporarily by modifying the `setWebSocketState` call

### Responses are not being sent
- Check the gateway logs for errors
- Ensure the Utopia client is accessible via `host:port`
- Run `openclaw channels status --probe`

### `openclaw: command not found` after installation
```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```
Add to `~/.bashrc` or `~/.zshrc` for permanent effect.

---

### Disabling / Clearing

Remove the token from the config:

```bash
openclaw channels logout utopia
```

The channel will stop working until the token is reconfigured.

## Security

- Never commit `apiToken` to git
- Keep `host` on `127.0.0.1` unless remote access is needed
- Use `allowlist` or `pairing` in production — not `open`
- The token grants full access to the Utopia client — store it as a password

---

## Development

```bash
# From the OpenClaw repo root
pnpm install

# Run tests
pnpm test -- extensions/utopia

# Type check
pnpm build
```

---

## Message Flow

1. The plugin connects to your local Utopia client via its HTTP API
2. It enables WebSocket notifications and listens for incoming messages
3. When a message arrives from an authorized user, it forwards the text to OpenClaw for processing
4. OpenClaw's response is sent back to the user via `sendInstantMessage`

```
User (Utopia)
│
│ Private message
▼
Utopia Desktop (local)
│
│ WebSocket event: newInstantMessage (:wsPort)
▼
OpenClaw Utopia Plugin
│
│ Passes to OpenClaw pipeline
▼
OpenClaw (processes, generates response)
│
│ sendInstantMessage via HTTP API (:port)
▼
Utopia Desktop (local)
│
│ Delivers message
▼
User (Utopia)
```
