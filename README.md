<!-- GENERATED from README.template.md by scripts/sync-mcp-server-readme.mjs - do not edit directly. -->
# @clipform/mcp-server

MCP server for [Clipform](https://clipform.io) - build and manage video-style forms from any MCP client (Claude, ChatGPT, Cursor, Windsurf, etc.).

## Install

There are two ways to connect: **remote** (recommended, uses your Clipform account) and **local** (anonymous, free-tier limits).

### Remote (recommended)

Use this if you have a Clipform account. Forms are created directly in your workspace and your plan tier applies (no 3-node cap on Pro).

**Claude (claude.ai):** Settings → Connectors → Add custom connector → enter `https://mcp.clipform.io`

**ChatGPT:** Settings → Connectors → Advanced → enable Developer Mode → Create connector → enter `https://mcp.clipform.io` (requires Pro/Team/Enterprise)

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

Your MCP client lists these automatically on connect (via `tools/list`). Full reference - arguments, schemas, and examples - is in the [docs](https://clipform.io/docs/api/mcp).

| Tool | Description |
|------|-------------|
| `clipform_create_form` | Create a new Clipform (interactive video-style form). |
| `clipform_import_form` | Convert a public Google Form, Typeform, or Tally form into a new Clipform. |
| `clipform_list_forms` | List forms in your workspace with optional filtering. |
| `clipform_whoami` | Show the current identity: auth mode (api key, oauth, session, or anonymous), active workspace, and plan limits. |
| `clipform_get_form` | Retrieve a form's details including all nodes in sequential order and their routing. |
| `clipform_update_form` | Update a form's title, publish status, settings, or tags. |
| `clipform_delete_form` | Move a form and all its nodes to the trash. |
| `clipform_get_results` | View response counts, choice/action answer breakdowns, and recent open-text answers for a form. |
| `clipform_get_responses` | List a Clipform form's individual responses (submissions) with time filtering - complements clipform_get_results (aggregates). |
| `clipform_add_node` | Add a new node to an existing form. |
| `clipform_update_node` | Update one or more existing nodes' text, type, config, or options. |
| `clipform_delete_node` | Delete a node from a form. |
| `clipform_upload_media_asset` | Put one or more media files into your workspace media library (max 10, uploaded sequentially). |
| `clipform_complete_media_upload` | Confirm a signed-PUT still image upload finished, after PUTting the bytes to the upload_url returned by clipform_upload_media_asset. |
| `clipform_attach_node_media` | Attach an existing workspace media asset (from clipform_upload_media_asset) to one or more nodes (max 10). |
| `clipform_get_node_media` | Get the media attached to a node, including processing status. |
| `clipform_delete_node_media` | Remove media from a node. |
| `clipform_set_logic` | Set the next node for one or more nodes, wiring a linear route (A to B to C). |
| `clipform_log_generation` | Internal audit step run by a form-generation workflow. |
| `clipform_search_news` | Fallback news lookup for clients without native web search. |
| `clipform_generate_tts` | Generate narration audio from text with word-level captions. |
| `clipform_generate_video` | Generate a video from images, video clips, or both, synced to an audio track. |
| `clipform_search_media` | Search images or stock video clips. |
| `clipform_render_composition` | Render a specialised video composition to MP4 or PNG - custom animated visuals that clipform_generate_video can't provide, such as geography animations or designed motion graphics. |
| `clipform_search_music` | Search for royalty-free music tracks and ambient sounds. |
| `clipform_list_compositions` | Browse available video compositions and their expected props schemas. |
| `clipform_list_video_templates` | Browse available video templates - curated Scene arrangements (bed + overlay + sane defaults) that render through the Scene composition from a small controls object. |
| `clipform_render_video_template` | Render a curated video template (a pre-arranged Scene: bed + overlay + sane defaults) to MP4 or PNG from a small controls object, instead of hand-assembling Scene layers. |
| `clipform_list_assets` | List available creative assets (sound effects, animations, fonts) for video compositions. |
| `clipform_check_render` | Check the status of render jobs started by clipform_generate_video, clipform_render_video_template, or clipform_render_composition. |
| `clipform_fetch_boundary` | Fetch a GeoJSON boundary polygon for a country, city, or region. |
| `clipform_get_guide` | Retrieve craft knowledge for building a specific form type. |
| `clipform_get_workflow` | Retrieve a step-by-step build workflow for creating a specific form type. |

## Example

> Create a Clipform called "Customer Feedback" with a choice question asking "How would you rate our service?" with options Excellent, Good, Fair, Poor, then an open-ended question asking "Any additional comments?", and finish with an end screen saying "Thanks for your feedback!"

## How it works

**Remote (OAuth):** Bearer tokens are audience-bound to `https://mcp.clipform.io` (RFC 8707) and scoped to `mcp` only. Forms land directly in the workspace you approved during consent.

**Local with API key:** The `CLIPFORM_API_KEY` is sent as a standard `Authorization: Bearer` header. Forms are created directly in the key's workspace with your plan tier limits.

**Local anonymous (no key):** Forms go into a shared unclaimed workspace. No auth is needed to edit them - the form UUID is the only credential. You'll get a claim URL to transfer ownership. Free-tier 3-node limit applies.

Forms are created with a start node and end screen automatically - you just add the nodes in between.

## Links

- [Clipform](https://clipform.io) - Create interactive video forms
- [Documentation](https://clipform.io/docs/api/mcp) - Full guide with node types and more

> Craft guides (`clipform://guides/*`, `clipform_get_guide`) are fetched from the Clipform API at runtime, so the server needs a reachable `API_URL` and valid `CLIPFORM_API_KEY` to serve them.
