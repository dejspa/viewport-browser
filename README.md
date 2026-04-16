# OpenEyes

Vision-first automation for AI agents via [MCP](https://modelcontextprotocol.io). No DOM parsing, no accessibility APIs, no selectors — the agent sees a screenshot and acts by coordinates, like a human.

**[OpenEyes Web](web/)** (`openeyes-web`) — Headless Chromium with per-agent session isolation. For browsing, scraping, filling forms.

OpenEyes Desktop (full OS control — mouse, keyboard, screenshots of the whole desktop) is in active development on the [`desktop-experimental`](../../tree/desktop-experimental) branch and not yet stable.

## Why

DOM-based tools break on shadow DOM, iframes, web components, and dynamic frameworks. They also cost 2–5× more in tokens per step because HTML dumps are huge. Screenshots are ~800 tokens each regardless of site complexity, and `elementFromPoint` + coordinate clicks naturally pierce every kind of DOM boundary.

## Quick start

```bash
git clone https://github.com/dejspa/openeyes
cd openeyes/web
uv sync
uv run playwright install chromium
uv run openeyes-web-serve
```

See [web/README.md](web/README.md) for detailed setup, MCP client config, and tool reference.

## License

MIT
