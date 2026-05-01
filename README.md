# Claude Browser Bridge

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/github/manifest-json/v/Zbrooklyn/obsidian-claude-bridge?label=version)](https://github.com/Zbrooklyn/obsidian-claude-bridge/releases)
[![Stars](https://img.shields.io/github/stars/Zbrooklyn/obsidian-claude-bridge?style=social)](https://github.com/Zbrooklyn/obsidian-claude-bridge/stargazers)
[![Issues](https://img.shields.io/github/issues/Zbrooklyn/obsidian-claude-bridge)](https://github.com/Zbrooklyn/obsidian-claude-bridge/issues)
[![Obsidian](https://img.shields.io/badge/obsidian-1.5.0%2B-purple)](https://obsidian.md)

> An Obsidian plugin that lets AI agents (Claude, Playwright, Chrome DevTools MCP, anything that speaks the Chrome DevTools Protocol) drive Obsidian's built-in Web viewer.

You log in to a website once in Obsidian's Web viewer (Gmail, GitHub, Upwork — whatever) and an external AI tool can then drive that tab using your already-authenticated session. **No headless browser, no separate authentication, no token storage. The login is yours; the bridge is just an attach point.**

Pairs with the [`obsidian-bridge-mcp`](https://github.com/Zbrooklyn/obsidian-bridge-mcp) MCP server, which exposes the bridge to Claude Code (and any other MCP client) as 25 tools — navigate, click, type, screenshot, snapshot, scroll, etc.

> _Screenshot here — capture your 🟢 status bar + Claude driving a Web viewer pane. Replace this line with `![demo](docs/demo.png)` once captured._

---

## Table of contents

- [What you can do with it](#what-you-can-do-with-it)
- [How it works](#how-it-works)
- [Install (full setup)](#install-full-setup)
- [Settings](#settings)
- [Status bar indicator](#status-bar-indicator)
- [Commands](#commands)
- [Comparison](#comparison-vs-other-browser-mcps)
- [FAQ](#faq)
- [Roadmap](#roadmap)
- [Platforms](#platforms)
- [Privacy & data](#privacy--data)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Acknowledgments](#acknowledgments)
- [License](#license)

---

## What you can do with it

Real things people use this for:

- **Manage Gmail without Claude seeing your password.** Sign into Gmail in Obsidian's Web viewer once. Ask Claude to triage, draft replies, search, archive — Claude drives the inbox using your existing session.
- **AI-assisted research that lands in your vault.** "Read this article and write me a summary as a note" — Claude opens the article in Obsidian, reads the DOM, writes a summary directly into a vault file. Same window the whole time.
- **Drive logged-in dashboards.** Stripe, Vercel, GitHub, Upwork, Shopify Admin — anywhere you're authenticated. Claude scrapes data, fills forms, runs reports without you switching context.
- **Inspect web pages while taking notes.** Screenshot, read HTML, extract structured data — all into your vault, all without leaving Obsidian.
- **Test your own web apps with AI.** Localhost dashboards, internal tools, prototypes — Claude can drive them via the bridge.

The shape: anywhere you'd say "I wish AI could do this in my browser without me copying things back and forth," this fits.

---

## How it works

Obsidian is built on Electron. When Electron launches with `--remote-debugging-port=9222`, its embedded Chromium exposes the Chrome DevTools Protocol on `localhost:9222`. Any tool that speaks CDP can attach and drive the Web viewer.

This plugin's job is to make that one launch flag a one-toggle setting:

1. You enable the plugin
2. It scans your Obsidian shortcuts (Desktop, Start Menu, Taskbar pin) and adds `--remote-debugging-port=9222` to each
3. Your next normal Obsidian launch opens the debug port automatically — no special launcher
4. Toggle the plugin off → shortcuts revert cleanly

The plugin itself is small (~600 lines of vanilla JS, no build step). Most of what it does is shortcut management; the actual driving happens via CDP from outside.

## Install (full setup)

Two pieces work together. **You install BOTH for it to work.** Total time ~5 min.

### Prerequisites
- Obsidian (desktop) v1.5.0+
- Node.js v18+ (`node -v` to check) — install from https://nodejs.org
- Git (`git --version` to check)
- Claude Code OR Claude Desktop

### Part 1 — Obsidian plugin (via BRAT)

[BRAT](https://github.com/TfTHacker/obsidian42-brat) is the standard installer for Obsidian plugins not yet in the official community store.

1. In Obsidian: **Settings → Community plugins → Browse** → search "BRAT" → **Install + Enable**
2. **Settings → BRAT → "Add Beta plugin"** → paste `Zbrooklyn/obsidian-claude-bridge` → **Add Plugin**
3. **Settings → Community plugins** → toggle **"Claude Browser Bridge"** ON
4. A notice fires: _"patched N Obsidian shortcut(s). Active on next launch."_
5. **Quit and reopen Obsidian** normally. Status bar (bottom-right) should show 🟢 **Bridge active**.

### Part 2 — MCP server (via npm)

In a terminal (PowerShell on Windows, Terminal on Mac/Linux):

```bash
npm install -g github:Zbrooklyn/obsidian-bridge-mcp
```

### Part 3 — Wire into Claude

**For Claude Code** (CLI):

```bash
claude mcp add obsidian-bridge -- obsidian-bridge-mcp
```

Then in any Claude Code session, run `/reload-plugins` — the 25 `obsidian_*` tools appear immediately, no restart needed.

**For Claude Desktop** (GUI), edit your config file:

| OS | Path |
|---|---|
| Mac | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |

Add this entry under `"mcpServers"`:

```json
{
  "mcpServers": {
    "obsidian-bridge": {
      "command": "obsidian-bridge-mcp"
    }
  }
}
```

Save, fully quit and reopen Claude Desktop.

### Part 4 — Verify

Ask Claude:

> Use `obsidian_status` to check the bridge.

You should see `reachable: true` and Chrome version. Done.

> **Need a more detailed walkthrough?** See [INSTALL.md](INSTALL.md) for a step-by-step click-by-click guide including non-technical setup and troubleshooting.

## Install (manual plugin only — if you skip BRAT)

1. Download `main.js`, `manifest.json` from the [latest release](https://github.com/Zbrooklyn/obsidian-claude-bridge/releases)
2. Put both into `<your-vault>/.obsidian/plugins/claude-browser-bridge/`
3. Settings → Community plugins → reload → enable Claude Browser Bridge

(You still need Parts 2-4 above for the MCP server.)

## Settings

Open Settings → Claude Browser Bridge:

- **Bridge enabled** (master toggle) — flip ON to patch shortcuts; flip OFF to revert
- **Debug port** — default `9222`. Change only if it conflicts with another tool
- **Patched shortcuts** — list of which `.lnk` files the plugin modified, with original args saved so the toggle can restore
- **Restart Obsidian to apply bridge now** — optional. Quits and relaunches via a patched shortcut so the bridge activates immediately. Same effect as closing and reopening Obsidian normally — just faster

## Status bar indicator

| State | Meaning |
|---|---|
| 🟢 Bridge active | Debug port is live, AI agents can attach |
| 🟡 Bridge active on next launch | Shortcuts patched, but THIS Obsidian instance was launched without the flag — restart to activate |
| 🔴 Bridge needs setup | Toggle "Bridge enabled" on |
| ⚪ Bridge disabled | You toggled it off |

Click the status bar item to see the list of CDP targets currently visible to external tools.

## Commands

`Ctrl/Cmd+Shift+P` → "Claude Browser Bridge:":

- **Open URL in Web viewer** — prompts for URL, opens in a new Web viewer pane
- **List CDP targets** — shows all attachable targets
- **Check bridge status now** — forces a status poll
- **Restart Obsidian to apply bridge** — quits + relaunches with the flag
- **Enable / Disable bridge** — same as the settings toggle

## Platforms

- ✅ Windows (tested on Win11)
- ⚠️ macOS — should work, shortcut-patching code path is Windows-specific. The bridge mechanism (CDP attach) is platform-agnostic; only the auto-patch step is Windows-only for now. macOS users can manually launch Obsidian with `open -a Obsidian.app --args --remote-debugging-port=9222`. PRs welcome.
- ⚠️ Linux — same as macOS. Manual launch with `obsidian --remote-debugging-port=9222`.
- ❌ Mobile — Obsidian Mobile doesn't run on Electron, no debug port available.

## Privacy & data

- The plugin runs entirely locally. No telemetry, no remote calls beyond polling `localhost:9222` for status.
- The debug port binds only to `127.0.0.1` (localhost) — no LAN exposure unless you explicitly add `--remote-debugging-address=0.0.0.0` (don't do this).
- Patched shortcut paths are stored in `data.json` inside this plugin's folder. That's gitignored and never leaves your machine.

## Troubleshooting

**Status bar stays 🔴 after enabling**
The plugin couldn't find any Obsidian shortcuts to patch. Check Settings → Claude Browser Bridge → "Patched shortcuts" — should list at least one. If empty, your Obsidian shortcuts might be in non-standard locations. Run "Patch shortcuts now" from the command palette to force a re-scan.

**Status bar stays 🟡 forever**
You launched Obsidian via a shortcut the plugin didn't patch (e.g. a custom shortcut elsewhere on disk, or a .desktop file on Linux). Either run "Restart Obsidian to apply bridge" from the command palette, or close + reopen Obsidian via a patched shortcut.

**Plugin slow to load (>30 sec)**
Old issue with v0.2 of the plugin (pre-filtered .lnk scan was slow). Fixed in v0.2.1+. If you're seeing this, you're on an old version — update via BRAT.

**External tool can't see the webview**
Open a Web viewer pane in Obsidian first (Cmd palette → "Web viewer: Open"). The plugin doesn't auto-open one; the bridge only enumerates webviews that exist.

## Comparison vs other browser MCPs

Quick guide for when to use this vs alternatives:

| Tool | What it drives | When to use it |
|---|---|---|
| **claude-browser-bridge** + obsidian-bridge-mcp (this) | Obsidian's Web viewer (uses your real Obsidian session) | When you want to use logged-in sessions, when you want to watch the work happen inside Obsidian, when you want results to land in your vault |
| [`chrome-devtools-mcp`](https://github.com/ChromeDevTools/chrome-devtools-mcp) | A separate Chromium spawned by the MCP | When you want a fresh anonymous browser, when you don't want cookies / state to persist, when you need parallel contexts |
| [`@playwright/mcp`](https://github.com/microsoft/playwright-mcp) | Playwright-managed browsers (Chromium, Firefox, WebKit) | When you want headless mode, cross-browser testing, or Playwright's full automation API surface |
| [`browsermcp`](https://browsermcp.io/) | Your actual desktop Chrome via extension | When you want to drive the browser you actually use, but Claude has limited control over Chrome's UI compared to its own Chromium |

These tools don't compete — they complement. Many users run several at once. The routing rule we use: **bridge for logged-in personal sessions, the others for fresh anonymous work.**

## FAQ

**Does this work with Claude Desktop?**
Yes — see the install steps for editing `claude_desktop_config.json`. Same MCP server, different client.

**Does this work without Claude — just Playwright or another tool?**
Yes. The bridge is the Chrome DevTools Protocol exposed at `localhost:9222`. Anything that speaks CDP — raw `chrome-remote-interface`, Playwright's `connectOverCDP()`, Puppeteer, custom Node scripts — can attach. The MCP server is just one consumer.

**Is my data safe?**
The plugin runs entirely on your machine. The debug port only listens on `127.0.0.1` (localhost). Cookies stay in Obsidian's local Electron session — same place they'd be in any browser. Nothing leaves your computer through this plugin.

**Why not just use Playwright MCP / Chrome DevTools MCP?**
Those spawn their own browser, which means: separate logins, separate cookies, doesn't reuse your Obsidian session. This plugin's whole point is to use sessions you already have authenticated in Obsidian. If you don't need that, the others are great.

**Will this slow down Obsidian?**
The plugin itself is ~600 lines of JS, polls every 5 seconds, negligible overhead. The Web viewer is heavy when loaded with sites like Gmail — but that's the site, not the plugin.

**What if I have multiple Obsidian vaults?**
The plugin patches shortcuts at the OS level, so any vault you launch via a patched shortcut gets the bridge. The MCP server connects to whichever Obsidian instance is running. Don't run two instances simultaneously on the same debug port.

**Can I change the port from 9222?**
Yes — Settings → Claude Browser Bridge → Debug port. Change there, restart Obsidian via the plugin's restart command. Update the MCP server's `OBSIDIAN_BRIDGE_PORT` env var to match.

**What about Obsidian Sync / Obsidian Publish?**
Unrelated. This plugin doesn't touch your vault data, only the Web viewer's runtime.

**The plugin doesn't appear in the official Obsidian community store. Is it official?**
Not yet — this is a beta release distributed via BRAT. If it gets enough usage and stability, official store submission comes next.

## Roadmap

What's planned and what isn't:

**Planned:**
- Cross-platform shortcut patching (macOS .desktop equivalents, Linux .desktop files)
- One-line installer that handles both plugin and MCP server in one command
- Optional persistent CDP connection (avoids per-tool reconnect overhead)
- Submission to official Obsidian community store
- Demo videos

**Maybe (open to discussion via Issues):**
- Multi-tab UI hints in Obsidian (visual badge on which Web viewer tab Claude is currently driving)
- Webhook / event subscription so external tools can listen for tab navigations
- Per-tab cookie isolation

**Not in scope:**
- Replacing your actual browser — this is a complement, not a replacement
- Mobile support — Obsidian Mobile isn't Electron, no debug port
- Headless mode — Obsidian is always headed by design

## Contributing

Issues, suggestions, and PRs welcome at https://github.com/Zbrooklyn/obsidian-claude-bridge/issues.

For PRs, please:
1. Open an issue first to discuss the change
2. Match the existing code style (vanilla JS, no build step, idiomatic Obsidian Plugin API)
3. Test on your own vault before submitting
4. Update README/INSTALL if your change affects user-facing behavior

For bug reports, please include:
- Obsidian version (Settings → About)
- Plugin version (`manifest.json` or BRAT settings)
- OS + version
- What you ran, what you expected, what happened
- Status bar state (🟢 / 🟡 / 🔴 / ⚪)
- Any output from the developer console (Ctrl+Shift+I in Obsidian)

## Acknowledgments

Built on:
- [Obsidian](https://obsidian.md) and its plugin API
- The [Model Context Protocol](https://modelcontextprotocol.io/) and the [@modelcontextprotocol/sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- [BRAT](https://github.com/TfTHacker/obsidian42-brat) for beta plugin distribution
- The Chrome DevTools Protocol — without which none of this would work

Inspired by the broader ecosystem of "AI inside Obsidian" plugins:
- [Claudian](https://github.com/YishenTu/claudian) — Claude Code as sidebar chat in Obsidian
- [obsidian-mind](https://github.com/breferrari/obsidian-mind) — vault as persistent memory for AI agents
- [obsidian-agent-client](https://github.com/RAIT-09/obsidian-agent-client) — agent client protocol bridge

Each does a different piece. This plugin's contribution is the browser bridge specifically.

## License

MIT — see [LICENSE](LICENSE).
