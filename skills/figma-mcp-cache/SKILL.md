---
name: figma-mcp-cache
description: >-
  Read Figma files through the cache-enabled Figma Context MCP
  (@pactortester/figma-mcp-cached): prepare a file, fetch node data, export
  images, and list or clear the local cache. Use when a Figma URL or file key
  is involved and that MCP server is available, when setting it up, or when
  the user asks to refresh, list, or clear the Figma cache.
---

# Figma MCP cache

Use the cache-enabled Figma Context MCP, package `@pactortester/figma-mcp-cached`. It stores full file responses on disk so later reads skip the Figma API until the TTL expires.

This is a different server from the official Figma MCP. Do not mix their tools. Official tools such as `get_design_context` and `get_screenshot` are not part of this flow.

Server and tool reference: [Pactortester/figma-mcp-cached](https://github.com/Pactortester/figma-mcp-cached).

## When to use

- A Figma design URL or file key needs to be read, and `figma_prepare_file` is available.
- The user wants the cached MCP configured.
- The user asks for the latest design, a cache refresh, cache status, or cache cleanup.

If those tools are missing, say the cached server is not connected and give the setup below. Do not invent file contents.

## Setup

Never write a real token into the repo, a skill file, or chat. The user creates a personal access token in Figma developer settings and puts it only in their local MCP config.

Cursor MCP config, 7-day cache:

```json
{
  "mcpServers": {
    "figma-mcp-cached": {
      "command": "npx",
      "args": [
        "-y",
        "@pactortester/figma-mcp-cached",
        "--stdio",
        "--figma-api-key=YOUR_FIGMA_API_KEY",
        "--figma-caching={\"ttl\":{\"value\":7,\"unit\":\"d\"}}"
      ]
    }
  }
}
```

TTL units: `ms`, `s`, `m`, `h`, `d`. Optional `cacheDir` accepts `~` or a relative path. Defaults: Linux `~/.cache/figma-mcp`, macOS `~/Library/Caches/FigmaMcp`, Windows `%LOCALAPPDATA%/FigmaMcpCache`.

Caching is off unless `FIGMA_CACHING` or `--figma-caching` is set. Command-line args override environment variables.

Prefer a long TTL (7–30 days) for stable files. Prefer a short TTL (hours) while a file is still changing. Cached data stays stale until TTL expiry or a force refresh.

## Read a file

1. Parse the URL.
   - `https://www.figma.com/design/{fileKey}/{title}?node-id={nodeId}`
   - `node-id` uses dashes (`2777-9428`). The API also accepts colons (`2777:9428`).
2. Call `figma_prepare_file` with the full `figmaUrl`.
3. Call `get_figma_data` with `fileKey`. Pass `nodeId` when the URL has `node-id`.
4. Omit `depth` unless the user asks for a depth limit.
5. Use `nodes` for structure, `globalVars` for reused colors and type, and `components` / `componentSets` for component definitions.

`figma_prepare_file` checks the cache, fetches on a miss or expiry, and checks that `nodeId` exists when the URL includes one. A hit means the file is ready. It does not replace `get_figma_data`.

Set `forceRefresh: true` on `figma_prepare_file` when the user says the design just changed, asks for the latest, or asks to refresh. Then call `get_figma_data` again.

## Export images

Call `download_figma_images` after the file is prepared.

- `nodes`: `{ nodeId, fileName }` entries. `fileName` must end in `.png` or `.svg`.
- `pngScale` defaults to `2`.
- `localPath` is an absolute directory. Omit it to use the system Downloads folder.
- For an image fill, pass `imageRef`. Add `needsCropping` and `cropTransform` only when the node is cropped.

Batch every export in one call.

## Cache tools

- `list_cache` takes no arguments. It returns whether caching is on, the directory, file count, size, and TTL.
- `cleanup_cache` takes no arguments. It deletes expired and corrupt cache files.

Use these when the user asks about cache status or disk use. Do not delete the cache directory by hand unless those tools are unavailable and the user asked to clear it.

## Failures

- **429 / rate limit:** stop refetching. Lengthen TTL and keep using `figma_prepare_file` before `get_figma_data`.
- **Node missing:** confirm the id, then `forceRefresh: true`. The node may have been deleted.
- **Stale design:** `forceRefresh: true`. Do not treat a cache hit as proof the file matches Figma right now.
- **Cache never hits:** caching is unset, TTL is `0`, or the cache directory is not writable.
