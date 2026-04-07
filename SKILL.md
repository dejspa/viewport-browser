---
name: viewport-browser
description: Vision-first web browser — navigate websites, click by coordinates, fill forms, extract text. Uses screenshots + coordinate-based clicking with auto-snap to nearest interactive element.
version: 1.0.0
requires:
  env: []
  bins:
    - uv
---

# Viewport Browser — Vision-First Web Navigation

You have access to a browser that lets you navigate websites, click elements, type text, and extract content. Everything is vision-based: you see screenshots and click by (x, y) pixel coordinates.

## Connection

The browser runs as an MCP server. If your runtime supports MCP, connect to it:

```json
{
  "mcpServers": {
    "viewport": {
      "command": "uv",
      "args": ["run", "--directory", "/path/to/viewport-browser", "viewport-browser"],
      "transport": "stdio"
    }
  }
}
```

If your runtime does not support MCP, use the REST API at `http://localhost:8080` (see API section below).

## Tools

| Tool | Parameters | Description |
|------|-----------|-------------|
| `navigate` | `url` | Go to a URL. If a tab with that domain is already open, switches to it. |
| `click` | `x`, `y` | Click at pixel coordinates on the screenshot. Auto-snaps to nearest interactive element. |
| `type_text` | `text`, `press_enter`, `clear_first` | Type into the currently focused element. |
| `scroll` | `direction` ("up"/"down") | Scroll the page. |
| `get_text` | — | Extract the page's main text content (articles, prices, product details). |
| `screenshot` | — | Take a fresh screenshot of the current page. |
| `go_back` | — | Browser back button. |
| `new_tab` | `url`, `pin` | Open a new tab. Set `pin="name"` to protect it from closing. |
| `switch_tab` | `index` | Switch to a tab by index. |
| `list_tabs` | — | Show all open tabs. |
| `close_tab` | `index` | Close a tab by index (cannot close pinned tabs). |

## How Clicking Works

1. Look at the screenshot and estimate the **(x, y) pixel coordinates** of what you want to click.
2. The screenshot is **~896 pixels wide** and **~630 pixels tall**.
3. Subtle **tick marks** along the top and left edges at **200px intervals** help you gauge position.
4. Your click is **automatically snapped** to the nearest interactive element (button, link, input).
5. After each click you get feedback like `Clicked: <button> 'Add to cart'` confirming what was hit.
6. To type into a field: **click its coordinates first** (to focus it), then use `type_text()`.
7. To search: click the search field, then `type_text(query, press_enter=true, clear_first=true)`.
8. Some actions return **text-only feedback** (no screenshot) when the page didn't visually change. Use `screenshot()` if you need to see the current state.

## Strategy Guide

### 1. SEARCH & ADD (e.g. "add product X to cart")
```
navigate → click search field → type_text(query, press_enter=true, clear_first=true) → screenshot → click "add" button
```

### 2. COMPARE & PICK (e.g. "find the cheapest X")
```
navigate → click search → type_text(query) → get_text (read ALL names and prices) → screenshot → click
```
ALWAYS use `get_text` first to read prices — don't guess prices from screenshots.

### 3. RESEARCH (e.g. "find info about X")
```
navigate → screenshot → get_text → report
```
Use `get_text` for article content — don't read long text from screenshots.

### 4. BROWSE FEED (e.g. "scroll through feed, find articles about X")
```
screenshot → scroll → screenshot → scroll (repeat)
```
Use `get_text` on interesting items.

## Important Behaviors

- If a **cookie banner**, ad interstitial, or overlay blocks the page, click its accept/dismiss/close button.
- **Popup tabs** (ads, new windows) are auto-closed.
- When comparing products: first filter to products that genuinely match the request, THEN pick cheapest among those. Think like a human — "milk" means regular milk, not oat milk.

## Rules

- Be efficient — never repeat the same action twice.
- Don't scroll unnecessarily — check what's already visible first.
- Don't open product detail pages when the info is already visible on the card.
- If an overlay or popup blocks you, take a new screenshot — it may have been auto-dismissed.

## REST API (for non-MCP runtimes)

If MCP is not available, use HTTP endpoints at `http://localhost:8080`:

```bash
# Navigate
curl -X POST http://localhost:8080/navigate -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}'

# Screenshot (returns base64 JPEG)
curl -X POST http://localhost:8080/screenshot

# Click
curl -X POST http://localhost:8080/click -H "Content-Type: application/json" \
  -d '{"x": 400, "y": 300}'

# Type text
curl -X POST http://localhost:8080/type_text -H "Content-Type: application/json" \
  -d '{"text": "search query", "press_enter": true, "clear_first": true}'

# Scroll
curl -X POST http://localhost:8080/scroll -H "Content-Type: application/json" \
  -d '{"direction": "down"}'

# Get text content
curl -X POST http://localhost:8080/get_text

# Go back
curl -X POST http://localhost:8080/go_back

# Tab management
curl -X POST http://localhost:8080/new_tab -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "pin": "mysite"}'
curl -X POST http://localhost:8080/switch_tab -H "Content-Type: application/json" \
  -d '{"index": 0}'
curl -X POST http://localhost:8080/list_tabs
curl -X POST http://localhost:8080/close_tab -H "Content-Type: application/json" \
  -d '{"index": 1}'
```

All endpoints return JSON with `screenshot` (base64 JPEG), `text` (feedback/context), and `url` (current URL).
