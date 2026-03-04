# Part 1：Claude Code 安装与配置

## Claude Code + AMD LLM Gateway Configuration Guide (Linux)

Use Claude Code through AMD's internal LLM Gateway without requiring an official Anthropic account.

---

## Environment Information

| Item                | Value                                |
| ------------------- | ------------------------------------ |
| Operating System    | Linux (Ubuntu)                       |
| AMD Gateway API Key | your api key                         |
| Gateway Address     | `https://llm-api.amd.com/Anthropic`  |
| Available Models    | `claude-opus-4.6`, `claude-sonnet-4` |

---

## Step 1: Install Node.js and Claude Code

```bash
# For RHEL / CentOS / Fedora / Rocky Linux
# curl -fsSL https://rpm.nodesource.com/setup_22.x | bash -
# yum install -y nodejs

# For Ubuntu — Install Node.js 22.x
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt-get install -y nodejs

# Verify installation
node --version   # v22.x.x
npm --version    # 10.x.x

# Install Claude Code globally
npm install -g @anthropic-ai/claude-code

# Check version
claude --version  # 2.x.x
```

---

## Step 2: Bypass Anthropic OAuth Login

Claude Code requires Anthropic OAuth login by default. Create a configuration file to skip it.

Edit (or create) `~/.claude.json`:

```json
{
  "hasCompletedOnboarding": true,
  "lastOnboardingVersion": "2.1.45",
  "primaryApiKey": "dummy"
}
```

> **Note**: `lastOnboardingVersion` must match your installed Claude Code version. Run `claude --version` to check.

---

## Step 3: Configure Environment Variables

### Option A: Add to `~/.bashrc` (Recommended — persistent across sessions)

Append the following to the end of `~/.bashrc`:

```bash
# === Claude Code + AMD LLM Gateway ===
export ANTHROPIC_BASE_URL="https://llm-api.amd.com/Anthropic"
export ANTHROPIC_API_KEY="dummy"
export ANTHROPIC_CUSTOM_HEADERS="Ocp-Apim-Subscription-Key: your apikey,user: $(whoami)"
export DISABLE_PROMPT_CACHING="1"
alias claude='claude --model claude-opus-4.6'
```

Then run:

```bash
source ~/.bashrc
```

After that, simply type `claude` to launch — it will automatically use the `claude-opus-4.6` model.

---

## Step 4: First Launch Confirmation

On first launch, Claude Code may display a prompt:

```
Do you want to use this API key? [Yes/No]
```

Select **Yes** — you won't be prompted again after that.

---

## Environment Variables Reference

| Variable                   | Value                               | Description                                     |
| -------------------------- | ----------------------------------- | ----------------------------------------------- |
| `ANTHROPIC_BASE_URL`       | `https://llm-api.amd.com/Anthropic` | AMD Gateway address                             |
| `ANTHROPIC_API_KEY`        | `dummy`                             | Placeholder — actual auth is via custom headers |
| `ANTHROPIC_CUSTOM_HEADERS` | See below                           | AMD Gateway authentication headers              |
| `DISABLE_PROMPT_CACHING`   | `1`                                 | AMD Gateway does not support Prompt Caching     |

### `ANTHROPIC_CUSTOM_HEADERS` Format

```
Ocp-Apim-Subscription-Key: <your API key>
user: <your username>
```

---

## Skip Permission Confirmations

For sandbox/demo environments, you can skip all permission prompts:

```bash
IS_SANDBOX=1 claude --dangerously-skip-permissions
```

> **Warning**: This disables all safety confirmations. Only use in isolated environments where Claude has full access to the filesystem and shell.
