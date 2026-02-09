# Turn Your Old MacBook Into an Always-On AI Assistant with OpenClaw

![macOS Big Sur+](https://img.shields.io/badge/macOS-Big_Sur_11.7+-blue?logo=apple) ![Node.js 22](https://img.shields.io/badge/Node.js-22-green?logo=node.js) ![License MIT](https://img.shields.io/badge/License-MIT-yellow)

> **Battle-tested on a 2014 MacBook Pro 15" running macOS Big Sur 11.7.10.** This isn't theoretical — it's running right now, lid closed, tucked on a shelf, answering messages on Discord 24/7.

---

## Why an Old MacBook?

You've probably got one. A 2012–2017 MacBook Pro sitting in a drawer, too slow for daily use but too nice to throw away. Here's the thing: it's a perfectly capable always-on server. It has:

- A built-in UPS (the battery)
- Wi-Fi
- Silent operation (no fans needed at idle)
- macOS, which means no driver headaches
- Enough grunt to run a gateway process that mostly just passes messages around

Pair it with [OpenClaw](https://github.com/openclaw/openclaw) and an Anthropic subscription, and you get a full-time AI assistant you can message from your phone — on Discord, Telegram, WhatsApp, Signal, whatever. No cloud server to rent. No API metering anxiety. Just an old Mac doing something useful again.

### What This Gets You

- 🤖 AI assistant reachable from Discord, Telegram, WhatsApp, Signal, etc.
- 🔄 Always-on — survives reboots, runs with the lid closed
- 🧠 Memory and continuity across sessions
- 🛠️ Tool use: browser control, file management, cron jobs, shell commands
- 💰 No API costs beyond your Anthropic subscription

---

## The Cost Math

Let's talk money, because this is where it gets interesting:

| Approach | Monthly Cost |
|---|---|
| Claude API (pay-per-use, heavy usage) | $200–500+ |
| **Claude Pro subscription** | **$20** |
| **Claude Max subscription** | **$100** |
| The old MacBook you already own | $0 |
| Electricity (Mac mini-level idle draw) | ~$3 |

**Total: $20–100/month for a full-time AI assistant.** That's it. The Mac is a sunk cost sitting in your drawer. Put it to work.

Pro and Max plans give you flat-rate access through Claude Code — no tokens to count, no surprise bills.

---

## What You'll Need

- **A Mac that can run macOS Big Sur or later** (natively — no hackintosh)
  - 2012–2017 MacBook Pro/Air, 2013+ Mac mini, etc.
  - This guide was tested on a 2014 MacBook Pro 15" (MacBookPro11,3) on Big Sur 11.7.10
- **An Anthropic account** — Pro ($20/mo) or Max ($100/mo) recommended
- **Power adapter** — required for clamshell/lid-closed mode
- **Wi-Fi or Ethernet** — the Mac needs internet, obviously
- ~30 minutes for setup

---

## Step 1: Install Homebrew

[Homebrew](https://brew.sh) is the package manager for macOS. On Big Sur, it still works — but it's not Tier 1 supported, so some packages may compile from source instead of downloading pre-built binaries. This is fine, just slower.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, follow the instructions it prints to add Homebrew to your PATH.

> **⚠️ Big Sur note:** If a `brew install` fails, try `brew install --build-from-source <package>`. Big Sur is Tier 2 for Homebrew, meaning binary bottles aren't guaranteed for every formula.

---

## Step 2: Install Node.js 22 via nvm

Don't use Homebrew's Node.js — on older macOS it can have compatibility issues. Use [nvm](https://github.com/nvm-sh/nvm) instead:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

Restart your terminal, then:

```bash
nvm install 22
nvm use 22
nvm alias default 22
```

Verify:

```bash
node --version  # Should show v22.x.x
npm --version
```

---

## Step 3: Install pnpm and Claude Code

```bash
npm install -g pnpm
```

Now install [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — Anthropic's official CLI tool. OpenClaw uses it under the hood, but it's also incredibly useful right now: **Claude Code can help you with the rest of this setup.** Seriously — if you get stuck on any step from here on, just ask it.

```bash
npm install -g @anthropic-ai/claude-code
```

Authenticate by running `claude` once and following the login prompts:

```bash
claude
```

This opens a browser to log in with your Anthropic account. Once authenticated, you've got an AI assistant in your terminal that can debug issues, edit configs, and guide you through the remaining steps.

> **💡 Pro tip:** From this point on, if anything goes wrong — a Homebrew formula fails, a permission dialog confuses you, a config file looks wrong — just ask Claude Code. It's like having a senior engineer pair-programming with you for the rest of the install.

---

## Step 4: Clone and Install OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

Then make the `openclaw` command available globally:

```bash
pnpm link --global
```

Verify it works:

```bash
openclaw --help
```

---

## Step 5: Set Up a Dedicated Identity

Your AI assistant will need its own accounts — GitHub, Discord bot, maybe others. Rather than tying everything to your personal email and phone number, give it a dedicated identity:

1. **Grab a cheap old iPhone** (or any spare phone)
2. **Get a prepaid SIM** — this gives you a real phone number for 2FA
3. **Create a new Gmail** with that number — this becomes the AI's primary email
4. **Use that Gmail** for all the AI's accounts: GitHub, Discord developer portal, API signups, etc.

This keeps your personal accounts completely separate, makes 2FA straightforward, and means you can hand the setup to someone else without entangling your own identity.

> **💡 Tip:** Label the phone and keep it plugged in near the Mac. You'll only need it occasionally for 2FA prompts.

---

## Step 6: Configure Your Channels

OpenClaw connects to messaging platforms via channel plugins. Configuration lives in:

```
~/.openclaw/openclaw.json
```

Set up whichever channels you want — Discord, Telegram, WhatsApp, Signal, etc. Each channel has its own setup (bot tokens, QR codes, etc.). Check the [OpenClaw docs](https://github.com/openclaw/openclaw) for channel-specific instructions.

A minimal Discord config looks something like:

```json
{
  "channels": [
    {
      "type": "discord",
      "token": "YOUR_DISCORD_BOT_TOKEN"
    }
  ]
}
```

---

## Step 7: Run the Gateway

Test it first in the foreground:

```bash
openclaw gateway start
```

If everything connects, you should see your bot come online in Discord (or whichever channel you configured). Send it a message. It should respond.

🎉 **That's the basic setup done.** Everything below is about making it bulletproof and always-on.

---

## Making It Always-On

This is the fun part. We're going to make this Mac run headless, lid closed, surviving reboots — a proper little server.

### Install Amphetamine (Keep Mac Awake)

1. Install **[Amphetamine](https://apps.apple.com/app/amphetamine/id937984704)** from the Mac App Store (free)
2. Download and install **[Amphetamine Enhancer](https://github.com/x74353/Amphetamine-Enhancer)** from the developer's site — this is a separate companion app that unlocks closed-display mode without an external monitor

#### Configure Amphetamine:

1. Open Amphetamine → Preferences
2. Start an indefinite session ("Indefinitely" trigger)
3. **Critical:** Go to Amphetamine Enhancer's settings and **UNCHECK** "Allow system sleep when display is closed"

Without this, macOS will sleep the instant you close the lid — even with Amphetamine running.

> **📌 Important:** The Mac must be plugged into power for clamshell mode to work. This is a macOS requirement, not an Amphetamine limitation.

### Auto-Start on Boot with launchd

OpenClaw can install itself as a Launch Agent so it starts automatically on login:

```bash
openclaw gateway install
```

This creates a plist at:

```
~/Library/LaunchAgents/bot.molt.gateway.plist
```

Logs go to:

```
~/.clawdbot/logs/gateway.log
```

To manage the service:

```bash
openclaw gateway status   # Check if it's running
openclaw gateway stop     # Stop it
openclaw gateway start    # Start it
openclaw gateway restart  # Restart it
```

> **⚠️ Port conflict gotcha:** If you're running OpenClaw manually in Terminal AND launchd tries to start it too, they'll fight over the port and one will crash. Stop the Terminal instance before relying on launchd, or vice versa.

### Enable Auto-Login (Optional)

If the Mac reboots (power outage, update, etc.), you want it to log in automatically so the Launch Agent can start:

1. **System Preferences → Users & Groups → Login Options**
2. Set **Automatic login** to your user account

> On newer macOS with FileVault enabled, auto-login may not be available. On Big Sur without FileVault, it works fine.

---

## Remote Access

You'll want to get into this Mac remotely for maintenance, troubleshooting, or just checking on things.

### TeamViewer 14

**This is the version that works on Big Sur.** Newer TeamViewer versions (15+) require macOS 13.5 or later and will not install on Big Sur.

- Download [TeamViewer 14](https://www.teamviewer.com/en/download/previous-versions/) (you may need to find archived versions)
- Install and set up unattended access with a password
- **Important:** Enable "Full access control at login window" — this lets you control the Mac even at the login screen after a reboot

> **Note on alternatives:** RustDesk 1.4.5's GUI doesn't render properly on Big Sur. If you need an alternative to TeamViewer, SSH is always reliable — just enable it in **System Preferences → Sharing → Remote Login**.

### SSH (The Reliable Fallback)

```bash
# Enable SSH on the Mac
# System Preferences → Sharing → check "Remote Login"
```

From another machine:

```bash
ssh your-username@your-macs-local-ip
```

For access outside your network, set up port forwarding on your router or use a tunnel service.

---

## macOS Permissions

macOS is strict about privacy permissions. You'll need to grant a few:

**System Preferences → Security & Privacy → Privacy:**

| Permission | What to Add | Why |
|---|---|---|
| **Accessibility** | `/usr/local/bin/cliclick` and/or the `node` binary | UI automation (clicking, typing) |
| **Screen Recording** | Terminal and/or `node` | Screenshot/screen capture capability |
| **Full Disk Access** | Terminal and/or `node` | Some file operations need this |

> **💡 launchd tip:** If OpenClaw runs via launchd (not Terminal), you need to grant permissions to the **`node` binary itself** — not Terminal. The node binary is typically at `~/.nvm/versions/node/v22.x.x/bin/node`. You can find it with `which node`.

---

## Troubleshooting

### Homebrew formulae fail to install

Big Sur is Tier 2 for Homebrew. Pre-built bottles may not be available for every formula.

```bash
brew install --build-from-source <formula>
```

This compiles from source — slower but usually works.

### launchd service keeps dying

Most likely a port conflict. Check if another OpenClaw instance is running:

```bash
ps aux | grep openclaw
```

Kill any stray processes, then restart the service:

```bash
openclaw gateway restart
```

### Mac sleeps with lid closed

- Make sure **Amphetamine** is running with an active session
- Make sure **Amphetamine Enhancer** is installed and "Allow system sleep when display is closed" is **UNCHECKED**
- Make sure the Mac is **plugged into power**

### TeamViewer disconnects when lid closes

Same fix as above — it's the Mac sleeping, not TeamViewer's fault. Fix the Amphetamine Enhancer settings.

### Node.js permission issues with launchd

When running via launchd, the `node` binary needs permissions directly — not Terminal. Add the actual node binary path to Accessibility/Screen Recording in System Preferences:

```bash
# Find your node binary
which node
# Typically: ~/.nvm/versions/node/v22.22.0/bin/node
```

### "Command not found: openclaw" after reboot

Your shell may not be loading nvm on non-interactive login (which is what launchd uses). Make sure your `~/.zshrc` (or `~/.bash_profile`) sources nvm, and that the launchd plist has the correct PATH set.

---

## Compatible Macs

Any Mac that can run **macOS Big Sur (11) or later natively** will work. That includes:

- MacBook Pro: 2013 and later (some 2012 models)
- MacBook Air: 2013 and later
- MacBook: 2015 and later
- Mac mini: 2014 and later
- iMac: 2014 and later
- Mac Pro: 2013 and later

Check [Apple's Big Sur compatibility list](https://support.apple.com/en-us/102861) for the full breakdown.

Newer Macs (Apple Silicon) work too, of course — but the whole point of this guide is that you don't need to buy anything new.

---

## Summary

| Step | Command/Action |
|---|---|
| Install Homebrew | `/bin/bash -c "$(curl -fsSL ...)"` |
| Install nvm + Node 22 | `nvm install 22` |
| Install pnpm | `npm install -g pnpm` |
| Install Claude Code | `npm install -g @anthropic-ai/claude-code` |
| Authenticate | `claude` |
| Clone OpenClaw | `git clone https://github.com/openclaw/openclaw.git` |
| Install & build | `pnpm install && pnpm build && pnpm link --global` |
| Configure channels | Edit `~/.openclaw/openclaw.json` |
| Test | `openclaw gateway start` |
| Install as service | `openclaw gateway install` |
| Keep awake | Amphetamine + Amphetamine Enhancer |
| Remote access | TeamViewer 14 or SSH |

---

## Final Thoughts

There's something satisfying about giving an old machine a second life. A 2014 MacBook Pro isn't going to win any benchmarks in 2025, but it doesn't need to — it just needs to run a Node.js process, keep a WebSocket open, and stay awake. It does all of that beautifully.

The real magic is in the economics: instead of paying per API call (where heavy usage can easily hit $200–500/month), you pair a flat-rate Anthropic subscription with hardware you already own. The result is an always-on AI assistant for $20–100/month, reachable from any messaging app on your phone.

Your old MacBook was always capable of this. It just needed the right software.

---

*This guide is part of the [OpenClaw](https://github.com/openclaw/openclaw) project. Contributions and corrections welcome.*
