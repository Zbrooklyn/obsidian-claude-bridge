# Install — full setup (plugin + MCP server)

> ~5 minutes total. Two pieces: the **Obsidian plugin** that opens the bridge port, and the **MCP server** that lets Claude Code (or any MCP client) drive the bridge.

## Prerequisites

- Obsidian (desktop) v1.5.0+
- Node.js v18+ (`node -v` to check)
- Claude Code (or any MCP-capable client)

## Step 1 — Install the Obsidian plugin via BRAT

[BRAT](https://github.com/TfTHacker/obsidian42-brat) is the standard installer for Obsidian plugins not yet in the official community store.

1. In Obsidian: **Settings → Community plugins → Browse → search "BRAT" → Install + Enable**
2. **Settings → BRAT → "Add Beta plugin"**
3. Enter: `Zbrooklyn/obsidian-claude-bridge`
4. Click "Add Plugin"
5. **Settings → Community plugins → enable "Claude Browser Bridge"**

**A notice fires:** _"Claude Browser Bridge enabled. N Obsidian shortcut(s) updated. The bridge will be active the next time you launch Obsidian."_

The plugin auto-patched your Obsidian shortcuts (Desktop, Start Menu, Taskbar) to add `--remote-debugging-port=9222`. **No special launcher** — just close and reopen Obsidian normally next time.

## Step 2 — Restart Obsidian

Quit and reopen Obsidian via your normal shortcut. The status bar should now show **🟢 Bridge active** (bottom-right).

If it still shows 🟡 "active on next launch" — you launched via a shortcut the plugin didn't see. Run **Cmd/Ctrl-Shift-P → "Claude Browser Bridge: Restart Obsidian to apply bridge"** to force.

## Step 3 — Install the MCP server

Open a terminal:

```bash
npm install -g obsidian-bridge-mcp
```

That's it. Verify:

```bash
obsidian-bridge-mcp --version
```

(Should print `0.6.0` or higher.)

## Step 4 — Wire into Claude Code

Open `~/.claude.json` (`%USERPROFILE%\.claude.json` on Windows). Find the `"mcpServers"` object and add:

```json
"obsidian-bridge": {
  "type": "stdio",
  "command": "obsidian-bridge-mcp"
}
```

Should look like:

```json
{
  "mcpServers": {
    "...other servers...": { "..." },
    "obsidian-bridge": {
      "type": "stdio",
      "command": "obsidian-bridge-mcp"
    }
  }
}
```

## Step 5 — Restart Claude Code and verify

Close Claude Code, reopen it. Then ask Claude:

> Use `obsidian_status` to check the bridge.

Expected output:

```json
{
  "reachable": true,
  "browser": "Chrome/142.0.7444.265",
  "port": 9222,
  ...
}
```

You're done. Try:

> Open `https://news.ycombinator.com` in my Obsidian Web viewer and tell me the top 3 headlines.

Claude will use `obsidian_new_tab` + `obsidian_read_text` (or `obsidian_snapshot`) to do the work — and you'll watch it happen live in Obsidian.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Status bar 🔴 "needs setup" | Toggle "Bridge enabled" ON in plugin settings. |
| Status bar 🟡 "active on next launch" | Restart Obsidian, OR run "Restart Obsidian to apply bridge" from command palette. |
| `obsidian_status` returns `reachable: false` | Obsidian isn't running, OR you launched via a shortcut the plugin didn't patch. Check status bar. |
| Tools don't appear after Claude Code restart | Check `~/.claude.json` syntax (no trailing comma on the new entry). Run `obsidian-bridge-mcp` in a terminal — if it prints help, the install is good and the issue is in the JSON config. |
| Plugin slow to enable (>30 sec) | You're on an old version. BRAT → check for updates. v0.2.1+ fixed this. |
| Cloudflare error on some sites | Some sites (cloudflare error 1002) refuse to load in Obsidian's webview because of how its DNS resolution looks. Use a regular browser MCP for those sites. |

## Privacy & security

- The plugin runs entirely locally. No telemetry.
- The debug port binds to `127.0.0.1` only (localhost) — not exposed to LAN.
- The MCP server connects only to `localhost:9222` and only drives webview targets it finds there. It does not spawn its own browser, does not touch your Chrome profile, and does not affect other browser MCPs you might have configured.
- Logging into a website in Obsidian's Web viewer is the same as logging in via Chrome — cookies are stored locally in Obsidian's Electron session.

## Uninstall

1. **Plugin:** Settings → Community plugins → toggle "Claude Browser Bridge" OFF (this restores your shortcuts to their original state). Then click 🗑 to fully remove.
2. **MCP server:** `npm uninstall -g obsidian-bridge-mcp`. Remove the `"obsidian-bridge"` entry from `~/.claude.json`.

That's it. Your shortcuts are back to original; nothing to clean up.
