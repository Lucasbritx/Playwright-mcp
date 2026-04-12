# Playwright MCP — Learn & Test Guide

> **Playwright MCP** is a [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server built on top of [Playwright](https://playwright.dev/). It lets AI assistants (GitHub Copilot, Claude, Cursor, etc.) directly control a real browser — clicking, typing, navigating, taking screenshots and running assertions — all through natural-language instructions.

---

## Table of Contents

1. [What is Playwright MCP?](#what-is-playwright-mcp)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [Configure Your AI Assistant](#configure-your-ai-assistant)
   - [VS Code + GitHub Copilot](#vs-code--github-copilot)
   - [Claude Desktop](#claude-desktop)
   - [Cursor](#cursor)
5. [Quick-Start: Your First Browser Session](#quick-start-your-first-browser-session)
6. [Key MCP Actions Reference](#key-mcp-actions-reference)
7. [Writing Tests with Playwright MCP](#writing-tests-with-playwright-mcp)
8. [Tips & Best Practices](#tips--best-practices)
9. [Learning Resources](#learning-resources)

---

## What is Playwright MCP?

The [Model Context Protocol](https://modelcontextprotocol.io/) is an open standard that lets AI models talk to external tools via a structured JSON-RPC interface. `@playwright/mcp` packages Playwright's browser automation as an MCP server, so any MCP-compatible AI assistant can:

- Open a browser and navigate to URLs
- Click elements, fill forms, press keyboard shortcuts
- Take screenshots and read accessibility snapshots
- Wait for network events or text to appear on the page
- Drag-and-drop, hover, select options, upload files
- Run assertions and extract structured data

No special test code needs to be written up front — you describe what you want in plain English and the AI figures out the Playwright calls.

---

## Prerequisites

| Requirement | Version |
|-------------|---------|
| [Node.js](https://nodejs.org/) | ≥ 18 |
| npm | ≥ 9 (bundled with Node 18+) |
| An MCP-compatible AI client | See section below |

Check your versions:

```bash
node -v   # should print v18.x.x or higher
npm -v    # should print 9.x.x or higher
```

---

## Installation

Install the Playwright MCP server globally so it is available to any project:

```bash
npm install -g @playwright/mcp
```

Then install the Playwright browser binaries (Chromium is used by default):

```bash
npx playwright install chromium
```

> **Tip:** You can also install `firefox` or `webkit` if you plan to test on those engines:
> ```bash
> npx playwright install firefox webkit
> ```

---

## Configure Your AI Assistant

### VS Code + GitHub Copilot

1. Open **VS Code** and make sure the [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) extension is installed and signed in.
2. Open your user or workspace settings JSON (`Ctrl/Cmd + Shift + P` → *"Open User Settings (JSON)"*).
3. Add the MCP server entry:

```json
{
  "github.copilot.chat.mcp.servers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

4. Reload VS Code. Open Copilot Chat and switch to **Agent** mode (`@workspace` drop-down → *Agent*).
5. You should now see **playwright** listed as an available tool in the chat panel.

---

### Claude Desktop

1. Open (or create) the Claude Desktop configuration file:
   - **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
2. Add:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

3. Restart Claude Desktop. You will see a 🔌 **plug icon** in the toolbar — click it and enable the *playwright* server.

---

### Cursor

1. Open **Cursor Settings** → **MCP** → **+ Add MCP Server**.
2. Fill in:
   - **Name:** `playwright`
   - **Type:** `command`
   - **Command:** `npx @playwright/mcp@latest`
3. Click **Save** and restart the Cursor window.

---

## Quick-Start: Your First Browser Session

Once the MCP server is configured, open a chat with your AI assistant (Agent / MCP mode) and try these prompts one by one:

```
Open https://playwright.dev and take a screenshot.
```

```
Click the "Docs" link in the navigation bar and take a screenshot.
```

```
Search for "getting started" in the search box and tell me the first result.
```

```
Go to https://demo.playwright.dev/todomvc and add three todo items: "Buy milk", "Walk the dog", "Read a book".
Then mark "Walk the dog" as completed and take a screenshot.
```

Watch the browser open in real time while the assistant drives it. Each step is fully observable — you can pause, correct the AI, or ask it to take a screenshot at any point.

---

## Key MCP Actions Reference

| Action | What it does |
|--------|-------------|
| `browser_navigate` | Navigate to a URL |
| `browser_snapshot` | Capture the accessibility tree of the current page |
| `browser_take_screenshot` | Take a PNG/JPEG screenshot |
| `browser_click` | Click an element |
| `browser_type` | Type text into an input |
| `browser_fill_form` | Fill multiple form fields at once |
| `browser_press_key` | Press a keyboard key (e.g. `Enter`, `Tab`) |
| `browser_hover` | Hover over an element |
| `browser_select_option` | Choose a value from a `<select>` dropdown |
| `browser_drag` | Drag one element onto another |
| `browser_wait_for` | Wait for text to appear/disappear or a timeout |
| `browser_handle_dialog` | Accept or dismiss a browser dialog |
| `browser_file_upload` | Upload a file via a file-chooser dialog |
| `browser_resize` | Resize the browser viewport |
| `browser_tabs` | List, open, close or switch between tabs |
| `browser_evaluate` | Run arbitrary JavaScript on the page |
| `browser_network_requests` | List all network requests since page load |
| `browser_console_messages` | Read browser console output |
| `browser_close` | Close the browser |

> Full documentation: <https://playwright.dev/docs/mcp>

---

## Writing Tests with Playwright MCP

You can use the MCP server interactively **or** as a building block inside automated Playwright test files. Below is an example workflow:

### 1. Explore the page with AI

Use your AI assistant to navigate the site and identify the selectors you need:

```
Open https://example.com/login
Snapshot the page and tell me the exact selector for the email input and the submit button.
```

### 2. Generate the test file

Ask the AI to turn the recorded steps into a Playwright test:

```
Based on the actions you just performed, write a Playwright test in TypeScript that logs in
with email "user@example.com" and password "secret", then asserts the dashboard heading is visible.
```

### 3. Run the generated test

```bash
npx playwright test
```

### 4. Debug failures interactively

When a test fails, paste the error into the chat:

```
This test is failing with: "Element not found: [data-testid=submit-btn]".
Open the page in the browser, snapshot it, and suggest a fix.
```

---

## Tips & Best Practices

- **Use `browser_snapshot` before clicking** — the accessibility tree is more reliable than screenshots for picking the right element.
- **Prefer descriptive element references** — tell the AI "the blue 'Sign in' button in the top-right corner" rather than a raw selector.
- **Chain steps in one prompt** — the MCP server keeps browser state between calls, so you can describe a full flow in a single message.
- **Use `browser_wait_for`** — always wait for expected content after navigation or async actions to avoid flaky behavior.
- **Headless vs. headed mode** — by default the browser runs headed (visible). Add `--headless` to the `args` array in your config to run silently in CI:
  ```json
  "args": ["@playwright/mcp@latest", "--headless"]
  ```
- **Record once, replay forever** — use Playwright's built-in [codegen](https://playwright.dev/docs/codegen) tool (`npx playwright codegen <url>`) to generate test skeletons, then let the AI refine them.

---

## Learning Resources

| Resource | Link |
|----------|------|
| Playwright Official Docs | <https://playwright.dev/docs/intro> |
| Playwright MCP Docs | <https://playwright.dev/docs/mcp> |
| Model Context Protocol Spec | <https://modelcontextprotocol.io/> |
| Playwright GitHub | <https://github.com/microsoft/playwright> |
| @playwright/mcp npm | <https://www.npmjs.com/package/@playwright/mcp> |
| Playwright Discord Community | <https://aka.ms/playwright/discord> |
| YouTube: Playwright Crash Course | Search *"Playwright tutorial"* on YouTube |

---

## Contributing

Contributions are welcome! Feel free to open an issue or pull request if you find an error, have a suggestion, or want to add more examples.

---

## License

This repository is open source. See [LICENSE](LICENSE) for details.
