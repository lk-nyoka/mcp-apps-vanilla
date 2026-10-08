# mcp-apps-vanilla

A dependency-free starting point for an [MCP Apps](https://apps.extensions.modelcontextprotocol.io/) view: one HTML file
that speaks the postMessage protocol directly, plus a test page that plays the host so you can develop without one.

Made while building [Open Counter](https://github.com/lk-nyoka/open-counter) for Alexa+, whose booking card is built this way.

## Why

Hosts render MCP Apps views in a sandboxed iframe, often with a strict Content Security Policy. A view that needs no
bundler, no SDK and no network loads anywhere, is easy to audit, and fits inside an MCP resource as a string.

## Files

| File | What |
|---|---|
| `view-template.html` | The view. Keep the protocol block, replace `render()`. |
| `test-host.html` | Open in a browser, pick a view file, paste a `CallToolResult`, press send. Logs every message. |

## The protocol in brief (MCP Apps 2026-01-26)

1. View → host: request `ui/initialize` with `appInfo`, `appCapabilities`, `protocolVersion`.
2. Host → view: the result, including `hostContext.theme`.
3. View → host: notification `ui/notifications/initialized`.
4. Host → view: `ui/notifications/tool-result` with the tool's `CallToolResult` (read `structuredContent`).
5. Optional: view → host `ui/message` (ask the host to send a user message), `ui/notifications/size-changed`.

## Serving it from an MCP server

```ts
server.registerResource("view", "ui://my-app/view.html", { mimeType: "text/html;profile=mcp-app" },
  async () => ({ contents: [{ uri: "ui://my-app/view.html", mimeType: "text/html;profile=mcp-app", text: VIEW_HTML }] }));

server.registerTool("my_tool", {
  description: "…",
  inputSchema: {},
  // Both keys: current hosts read _meta.ui.resourceUri, older ones "ui/resourceUri".
  _meta: { ui: { resourceUri: "ui://my-app/view.html" }, "ui/resourceUri": "ui://my-app/view.html" },
}, async () => ({ content: [{ type: "text", text: "…" }], structuredContent: { /* what render() reads */ } }));
```

## Licence

Apache 2.0.
