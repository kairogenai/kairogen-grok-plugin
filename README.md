# Kairogen Plugin for Grok Build

Generate images, videos, and audio from Grok Build — one account, one credit
wallet, every model.

This plugin connects Grok Build to [Kairogen](https://kairogen.ai) through
Kairogen's hosted [MCP server](https://github.com/kairogenai/kairogen-mcp).
Kairogen sits in front of the generative-media catalog — Seedream, Flux, Nano
Banana, GPT Image, Sora, Veo, Kling, Seedance, KairoClone, and Topaz — so you
pick a model instead of managing a provider account for each one. Install it
once, sign in to your Kairogen account in the browser, and it works.

## Installation

1. Install Grok Build (see the [Grok Build docs](https://docs.x.ai/build/overview)):

```bash
curl -fsSL https://x.ai/cli/install.sh | bash
```

2. Sign in to your xAI account:

```bash
grok login
```

3. Start Grok Build by running `grok`, then open the marketplace:

```text
/marketplace
```

4. Find **kairogen** in the list and press `i` to install it.

5. Open the MCP servers tab with `/mcp`, select **kairogen**, and press `i` to
   sign in. Your browser opens the Kairogen sign-in page. Authentication is
   OAuth 2.0 — there is no API key to copy or rotate.

6. Once kairogen shows **ready**, run `/kairogen-tutorial` for a guided first
   run, or just ask for what you want. (`/tutorial` is Grok Build's own
   onboarding tour — it will not start this plugin.)

## Tools

The plugin ships no code of its own — it declares the hosted MCP server, which
provides these tools.

| Tool | What it does |
|---|---|
| `list_models` | The live catalog of image and video models with prices and per-model `param_schema` |
| `estimate_cost` | Credit quote for an image or video generation, before it runs |
| `get_credits` | Remaining credit balance |
| `get_me_context` | Plan limits and how many generations are currently running |
| `generate_image` | Generate 1–4 images, optionally from reference images |
| `generate_video` | Text-to-video and image-to-video, 1–2 clips |
| `upscale_image` | Topaz Photo AI 2x / 4x, optional face enhancement |
| `animate_from_generation` | Turn a generated image into a video |
| `get_generation` | Poll or recover any generation by id |
| `recreate_generation` | Re-run a past generation, optionally overriding prompt / seed / quantity |
| `list_my_gallery` | Past image and video generations |
| `generate_audio` | Speech, sound effects, and music — synchronous |
| `create_narration` | Add AI narration to a video (async job) |
| `create_dubbing` | Translate and dub a video (async job) |
| `create_video_voice_change` | Replace the spoken voice in a video (async job) |
| `create_voice_alteration` | Re-voice an existing Kairogen narration (async job) |
| `get_audio_job_status` | Poll the async audio jobs |
| `estimate_audio_cost` | Credit quote for any audio kind |
| `list_audio_history` | Unified audio history |
| `get_voices_data` | Voice catalog and custom clones as JSON |
| `create_voice_clone` / `preview_voice_clone` / `delete_voice_clone` | Custom voice clones |
| `upload_reference_image` / `upload_reference_audio` | Upload a local file and get a public URL for use as a reference |

The server also exposes widget-driven tools (`open_*_widget`, `list_voices`,
`list_characters`, `characters_*`, `download_*_from_url`) for hosts that render
[MCP Apps](https://github.com/modelcontextprotocol/ext-apps). Grok Build is a
terminal host and does not, so the skills below route around them.

## Skills

| Skill | What it does |
|---|---|
| `kairogen` | Entry point: connecting, credits and plan limits, cost discipline, references, safety, and the terminal-host adaptations |
| `kairogen-image` | Model choice, edits with reference images, consistency, upscaling |
| `kairogen-video` | Text-to-video vs image-to-video, the `extra_params` key remapping, durations, waiting on long renders |
| `kairogen-audio` | Routing between standalone audio and the four video-audio jobs, voices, cloning, polling |

## Commands

| Command | What it does |
|---|---|
| `/kairogen-tutorial` | Guided first run — connect, browse the catalog, quote a cost, generate one image |
| `/models` | Browse the live catalog with prices, filtered by the job you describe |

## What this plugin does on your machine

Declared for review and for anyone reading before installing:

- **No executable code.** The plugin is a `.mcp.json`, four skills, and two
  commands. Safety rules live in the `kairogen` skill (Grok Build does not
  load a plugin `rules/` directory). There are no scripts, no hooks, no
  install steps, and no dependencies.
- **Network endpoints.** One: `https://mcp.kairogen.ai/mcp`, contacted by the
  MCP client, not by the plugin. The MCP server calls Kairogen's API at
  `https://api.kairogen.ai` on the server side. OAuth sign-in happens at
  `https://api.kairogen.ai` and `https://app.kairogen.ai`.
- **Credentials.** OAuth 2.0, run by your MCP client and stored by it. The
  plugin never reads, writes, or transports a token, and never asks for a
  password or an API key.
- **Data leaving the machine.** Only what you generate with: prompts, and any
  reference file you explicitly point at. `upload_reference_image` and
  `upload_reference_audio` return a **public** URL for the file they upload.
  The skills instruct the agent to upload only files you named, and to tell you
  the result is publicly reachable.
- **Cost.** Generations debit credits from the signed-in Kairogen account. The
  skills require a cost quote before batches, video, and 4x upscales.

## Resources

- [Kairogen](https://kairogen.ai)
- [MCP docs](https://kairogen.ai/docs/mcp)
- [Web app](https://app.kairogen.ai)

## License

MIT — see [LICENSE](LICENSE).
