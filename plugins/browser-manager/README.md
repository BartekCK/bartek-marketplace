# browser-manager

Browser automation and inspection plugin for Claude Code, powered by the Chrome DevTools MCP. Navigate, interact, and inspect live Chrome browser sessions directly from your Claude conversations.

## Features

- Navigate to URLs and read page content
- Click buttons, links, and interactive elements
- Fill and submit forms
- Scroll pages
- Read console logs (errors, warnings, info)
- Inspect network requests and responses
- DOM inspection

## Prerequisites

- Chrome 144 or later (if your stable Chrome is below 144, use Chrome Beta or Canary)
- Node.js v20.19 or later (for `npx`)
- Remote debugging enabled in Chrome

### Enable Remote Debugging

1. Open Chrome
2. Go to `chrome://inspect/#remote-debugging`
3. Toggle the **Enable remote debugging** switch to ON

Chrome will show a banner saying "Chrome is being controlled by automated test software" and ask for permission when a session connects.

> **Security note:** Remote debugging grants programmatic access to your browser session. Disable it when not actively using the browser-agent.

## Installation

This plugin is part of the `websoftware-tools` marketplace. If the marketplace is already configured, the plugin is available automatically.

To use standalone, add the `browser-manager` directory to your Claude Code plugins configuration.

## Usage

The `browser-agent` triggers automatically on natural language requests involving browser interaction:

- "Check in browser if the login works"
- "Are there console errors on the dashboard?"
- "Open https://example.com in browser and tell me what's on the page"
- "Fill in the registration form with test data"
- "Check the network requests when I submit this form"
- "Test this in the browser"

## How It Works

The plugin uses [chrome-devtools-mcp](https://www.npmjs.com/package/chrome-devtools-mcp) to connect Claude to your live Chrome session via the Chrome DevTools Protocol. This means Claude can interact with your already-authenticated browser — no separate login needed.

## Troubleshooting

**"No browser session found"**
Ensure Chrome is open and remote debugging is enabled at `chrome://inspect/#remote-debugging`.

**"Permission denied"**
Click "Allow" when Chrome shows the remote debugging permission dialog.

**MCP server not starting**
Ensure Node.js v20.19 or later is installed. `npx` is bundled with npm and does not need separate installation.
