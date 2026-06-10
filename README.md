# @clipform/mcp-server

MCP server for [Clipform](https://clipform.io) - build and manage video-style forms from any MCP client (Claude, ChatGPT, Cursor, Windsurf, etc.).

## Install

There are two ways to connect: **remote** (recommended, uses your Clipform account) and **local** (anonymous, free-tier limits).

### Remote (recommended)

Use this if you have a Clipform account. Forms are created directly in your workspace and your plan tier applies (no 3-node cap on Pro).

**Claude (claude.ai):** Settings â†’ Connectors â†’ Add custom connector â†’ enter `https://mcp.clipform.io`

**ChatGPT:** Settings â†’ Connectors â†’ Advanced â†’ enable Developer Mode â†’ Create connector â†’ enter `https://mcp.clipform.io` (requires Pro/Team/Enterprise)

**Any MCP client with OAuth:** Point it at `https://mcp.clipform.io` - discovery, registration, and auth are handled automatically via OAuth 2.1 + Dynamic Client Registration (RFC 7591).

### Local - Claude Code / Claude Desktop / Cursor / etc.

Use this for stdio-based MCP clients. Pass your API key to authenticate with your workspace and plan tier.

**Claude Code:**

```bash
claude mcp add clipform -e CLIPFORM_API_KEY=cf_xxx -- npx -y @clipform/mcp-server
```

**Claude Desktop / Cursor / Windsurf / OpenClaw** - add to your MCP config:

```json
{
  "mcpServers": {
    "clipform": {
      "command": "npx",
      "args": ["-y", "@clipform/mcp-server"],
      "env": {
        "CLIPFORM_API_KEY": "cf_xxx",
        "API_URL": "https://api.clipform.io"
      }
    }
  }
}
```

Self-service API key generation is coming soon. For now, your API key is provided during onboarding or via your account settings - contact support if you need one.

You can also pass the key as a CLI flag: `npx -y @clipform/mcp-server --api-key=cf_xxx`

**Anonymous mode** (no API key): Forms go into a shared workspace with the free-tier 3-node limit. You'll get a claim URL to move forms into your account.

### Environment variables

| Variable | Required | Purpose |
|----------|----------|---------|
| `CLIPFORM_API_KEY` | For workspace auth | Bearer key - forms land in the key's workspace with its plan limits. Also unlocks the creative tools (search, TTS, video rendering). Omit for anonymous mode. |
| `API_URL` | Yes | Clipform API base URL (`https://api.clipform.io`, or your local/self-hosted API). |

## Tools

### Form & node management

| Tool | Description |
|------|-------------|
| `clipform_create_form` | Create a new form with nodes, theming, and tags in one call. Returns the created node IDs (structured output) so media steps need no follow-up lookup |
| `clipform_list_forms` | List forms in your workspace with filtering and pagination |
| `clipform_get_form` | View a form and all its nodes |
| `clipform_update_form` | Change title, publish status, or settings |
| `clipform_delete_form` | Delete a form and its nodes (asks the user to confirm) |
| `clipform_add_node` | Add a node to an existing form |
| `clipform_update_node` | Update node text, type, config, or options |
| `clipform_delete_node` | Remove a node (logic chain auto-relinks; asks the user to confirm) |

### Media & logic

| Tool | Description |
|------|-------------|
| `clipform_upload_node_media` | Attach video or image to one or more nodes (batch, max 10); set `fit_media: true` for generated renders |
| `clipform_get_node_media` | View a node's media details |
| `clipform_delete_node_media` | Remove media from a node (asks the user to confirm) |
| `clipform_attach_audio` | Attach audio to a still-image node |

### Creative

| Tool | Description |
|------|-------------|
| `clipform_render_composition` | Render a video composition to MP4 or PNG - flag reveals, emoji puzzles, grids, timelines, map motion and more. Pass `wait: false` to fire renders in parallel and poll for results |
| `clipform_generate_tts` | Generate narration audio with word-level captions |
| `clipform_generate_slideshow` | Create slideshow videos from images + audio (waits, returns URL) |
| `clipform_generate_video` | Generate video from images, clips, or both synced to audio. Supports a looping background-audio bed (crowd noise, ambience) and print-style texture overlays. Pass `wait: false` to fire renders in parallel and poll for results |
| `clipform_check_render` | Check status of a render started with `wait: false` |
| `clipform_search_media` | Search royalty-free images and videos, with orientation filter and alt-text descriptions |
| `clipform_search_music` | Search royalty-free music and ambient sounds |
| `clipform_list_compositions` | List available video compositions and prop schemas |
| `clipform_list_assets` | List available sound effects, animations, and fonts |
| `clipform_fetch_boundary` | Fetch a GeoJSON boundary polygon for a country, city, or region |

## Example

> Create a Clipform called "Customer Feedback" with a choice question asking "How would you rate our service?" with options Excellent, Good, Fair, Poor, then an open-ended question asking "Any additional comments?", and finish with an end screen saying "Thanks for your feedback!"

## How it works

**Remote (OAuth):** Bearer tokens are audience-bound to `https://mcp.clipform.io` (RFC 8707) and scoped to `mcp` only. Forms land directly in the workspace you approved during consent.

**Local with API key:** The `CLIPFORM_API_KEY` is sent as a standard `Authorization: Bearer` header. Forms are created directly in the key's workspace with your plan tier limits.

**Local anonymous (no key):** Forms go into a shared unclaimed workspace. No auth is needed to edit them - the form UUID is the only credential. You'll get a claim URL to transfer ownership. Free-tier 3-node limit applies.

Forms are created with a start node and end screen automatically - you just add the nodes in between.

## Links

- [Clipform](https://clipform.io) - Create interactive video forms
- [Documentation](https://clipform.io/docs/guides/mcp) - Full guide with node types, scoring, and more

> Craft guides (`clipform://guides/*`, `clipform_get_guide`) are fetched from the Clipform API at runtime, so the server needs a reachable `API_URL` and valid `CLIPFORM_API_KEY` to serve them.
