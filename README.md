# OpenClaw Setup Guide

Complete step-by-step guide for installing and configuring OpenClaw with Quatarly AI and Telegram bot integration.

## Requirements

- Linux system (tested on Ubuntu/Debian)
- Node.js v24 (recommended) or v22 LTS
- npm package manager
- Internet connection
- Telegram account (for bot integration)
- Quatarly AI API key (get yours from Quatarly dashboard)

## Step 1 - Install Node.js

Check if Node.js is already installed:
```bash
node --version
```

If not installed or version is below v22, install Node.js:
```bash
# Download and install Node.js v24+
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Verify installation:
```bash
node --version
npm --version
```

## Step 2 - Install OpenClaw

Install OpenClaw globally via npm:
```bash
npm install -g openclaw
```

Verify installation:
```bash
openclaw --version
```

Run the onboarding wizard:
```bash
openclaw onboard
```

## Step 3 - Configure Quatarly AI

### 3.1 Configure the Main OpenClaw Settings

Edit the main configuration file:
```bash
nano ~/.openclaw/openclaw.json
```

Add the Quatarly provider configuration in the `models` section:
```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "custom-api-quatarly-cloud": {
        "baseUrl": "https://api.quatarly.cloud/",
        "api": "anthropic-messages",
        "apiKey": "Your API Key",
        "models": [
          {
            "id": "claude-opus-4-6-thinking",
            "name": "claude-opus-4-6-thinking (Custom Provider)",
            "contextWindow": 200000,
            "maxTokens": 16384,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "reasoning": false
          },
          {
            "id": "claude-sonnet-4-6-thinking",
            "name": "claude-sonnet-4-6-thinking (Custom Provider)",
            "contextWindow": 200000,
            "maxTokens": 16384,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "reasoning": false
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": {
        "primary": "custom-api-quatarly-cloud/claude-sonnet-4-6-thinking"
      }
    }
  }
}
```

### 3.2 Configure Agent Authentication

Create or edit the auth profiles file:
```bash
nano ~/.openclaw/agents/main/agent/auth-profiles.json
```

Add ONLY the Quatarly provider (remove any openrouter entries):
```json
{
  "quatarly": {
    "baseUrl": "https://api.quatarly.cloud/",
    "apiKey": "API KEY",
    "provider": "anthropic"
  }
}
```

### 3.3 Configure Agent Model

Create or edit the agent configuration:
```bash
nano ~/.openclaw/agents/main/agent/agent.json
```

Set the provider to quatarly:
```json
{
  "id": "main",
  "provider": "quatarly",
  "model": "claude-opus-4-6-thinking"
}
```

### 3.4 Create BOOTSTRAP.md (if missing)

```bash
mkdir -p ~/.openclaw/workspace
nano ~/.openclaw/workspace/BOOTSTRAP.md
```

Add basic content:
```markdown
# OpenClaw Workspace

This is the default workspace for OpenClaw agents.
```

## Step 4 - Setup Gateway

### 4.1 Configure Gateway Settings

The gateway configuration should already be in `~/.openclaw/openclaw.json`:
```json
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "YOUR_SECURE_TOKEN_HERE"
    },
    "mode": "local",
    "port": 18789,
    "bind": "lan"
  }
}
```

### 4.2 Start the Gateway

Start the gateway service:
```bash
openclaw gateway start
```

Check gateway status:
```bash
openclaw gateway status
```

Restart gateway (if needed):
```bash
openclaw gateway restart
```

View gateway logs:
```bash
openclaw logs --follow
```

## Step 5 - Connect Telegram Bot

### 5.1 Create Telegram Bot

1. Open Telegram and search for `@BotFather`
2. Send `/newbot` command
3. Follow prompts to create your bot
4. Copy the bot token (format: `12890:ABCdefGHIjklMNOpqrsTUVwxyz`)

### 5.2 Configure Telegram Channel

Edit the OpenClaw configuration:
```bash
nano ~/.openclaw/openclaw.json
```

Add the Telegram channel configuration:
```json
{
  "channels": {
    "telegram": {
      "botToken": "YOUR_TELEGRAM_BOT_TOKEN",
      "enabled": true
    }
  }
}
```

Example with actual token:
```json
{
  "channels": {
    "telegram": {
      "botToken": "BOTTOKEN",
      "enabled": true
    }
  }
}
```

### 5.3 Restart Services

After configuration changes:
```bash
openclaw gateway restart
```

### 5.4 Test Telegram Bot

1. Open Telegram
2. Search for your bot by username
3. Send `/start` command
4. Send a test message to verify the bot responds

## Verification Commands

Check overall system health:
```bash
openclaw doctor
```

List configured agents:
```bash
openclaw agents list
```

Check channel status:
```bash
openclaw channels status
```

View active sessions:
```bash
openclaw sessions
```

## Common Errors and Fixes

### Error: "No API key found for provider openrouter"

**Cause:** The auth-profiles.json file contains an openrouter entry that conflicts with quatarly configuration.

**Fix:**
1. Edit the auth profiles file:
   ```bash
   nano ~/.openclaw/agents/main/agent/auth-profiles.json
   ```
2. Remove the entire `"openrouter"` section
3. Keep only the `"quatarly"` section
4. Save and restart gateway:
   ```bash
   openclaw gateway restart
   ```

### Error: "HTTP 404 error when reading BOOTSTRAP.md"

**Cause:** The BOOTSTRAP.md file is missing from the workspace.

**Fix:**
```bash
mkdir -p ~/.openclaw/workspace
echo "# OpenClaw Workspace" > ~/.openclaw/workspace/BOOTSTRAP.md
```

### Error: "Token limit exceeded (17k/16k)"

**Cause:** The model's context window is set too low.

**Fix:**
1. Edit openclaw.json:
   ```bash
   nano ~/.openclaw/openclaw.json
   ```
2. Update the model configuration to increase limits:
   ```json
   "contextWindow": 200000,
   "maxTokens": 16384
   ```
3. Restart gateway:
   ```bash
   openclaw gateway restart
   ```

### Error: "Multiple state directories detected"

**Cause:** OpenClaw found configuration in multiple user directories.

**Fix:** This is usually just a warning. The active directory is shown in the `openclaw doctor` output. If needed, consolidate to one location:
```bash
# Use root's openclaw directory
rm -rf /home/*/openclaw
```

### Error: "Gateway not responding"

**Cause:** Gateway process crashed or not running.

**Fix:**
```bash
# Kill any stuck processes
pkill -9 openclaw-gateway

# Restart gateway
openclaw gateway start

# Check status
openclaw gateway status
```

### Error: "Agent configuration not found"

**Cause:** Agent files are missing or corrupted.

**Fix:**
```bash
# Recreate agent configuration
openclaw agents add main
```

### Error: "Telegram bot not responding"

**Cause:** Invalid bot token or gateway not running.

**Fix:**
1. Verify bot token is correct in openclaw.json
2. Check gateway is running:
   ```bash
   openclaw gateway status
   ```
3. Check logs for errors:
   ```bash
   openclaw logs --follow
   ```
4. Restart gateway:
   ```bash
   openclaw gateway restart
   ```

## Useful Commands Reference

```bash
# View all logs in real-time
openclaw logs --follow

# Check system configuration
openclaw doctor

# List all agents
openclaw agents list

# Check gateway status
openclaw gateway status

# Restart gateway
openclaw gateway restart

# Stop gateway
openclaw gateway stop

# View sessions
openclaw sessions

# Check channel status
openclaw channels status --probe
```

## Configuration File Locations

- Main config: `~/.openclaw/openclaw.json`
- Agent config: `~/.openclaw/agents/main/agent/agent.json`
- Auth profiles: `~/.openclaw/agents/main/agent/auth-profiles.json`
- Workspace: `~/.openclaw/workspace/`
- Logs: `~/.openclaw/logs/`

## Notes

- Always use the `/root/.openclaw` directory for configuration (not `/home/rabbit/.openclaw`)
- Get your Quatarly API key from: https://quatarly.cloud
- Default gateway port is `18789`
- Context window is set to `200000` tokens for optimal performance
- Max output tokens is set to `16384`

## Support

For issues or questions:
- Check logs: `openclaw logs --follow`
- Run diagnostics: `openclaw doctor`
- Review configuration files in `~/.openclaw/`
root@rabit:~/Guide#
