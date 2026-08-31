---
description: Guided first run of the Kairogen plugin — connect, browse models, generate an image
---

# Kairogen Tutorial

Walk the user through the plugin one short step at a time. Keep every message
brief. After each step, ask whether to continue.

Do not spend the user's credits without asking. Step 4 is the only step that
costs anything, and it must be confirmed first.

Grok Build's built-in `/tutorial` is the CLI onboarding tour. This command is
`/kairogen-tutorial` — do not send the user to `/tutorial`.

## Step 1: Connection

Check whether the `kairogen` MCP server is connected. If it is not ready, tell
the user to open `/mcp`, select **kairogen**, and press `i` to sign in with
their Kairogen account in the browser. Authentication is OAuth — there is no
API key to paste. If they do not have an account yet, point them at
https://kairogen.ai.

Once it is ready, call `get_credits` and tell them their balance.

## Step 2: The catalog

Explain that Kairogen is one account in front of many model families, and that
`list_models` is always the source of truth for what is available and what it
costs. Call it, and summarize what is there: a few image models, a few video
models, and roughly what they cost. Do not paste the whole payload.

## Step 3: Cost before spending

Explain that `estimate_cost` gives a quote before any generation runs. Pick a
mid-tier image model from step 2 and show the estimated cost for a single
image.

## Step 4: Generate an image (asks for confirmation first)

Tell the user this step will spend the credits quoted in step 3, and ask
whether to proceed. If they decline, skip to step 5.

If they agree, ask for a one-line prompt (or offer one), then call
`generate_image` with `wait_for_completion: true`. Explain while it runs that
Grok Build has no inline preview, so the plugin waits for the result instead of
polling in the background. When it finishes, give them the output URL.

## Step 5: What comes next

Point at the skills, one line each:

- `kairogen-image` — model choice, edits with reference images, upscaling
- `kairogen-video` — text-to-video, image-to-video, animating a still
- `kairogen-audio` — speech, music, sound effects, narration, dubbing

Mention `/models` for browsing the catalog, and that they can just ask in plain
language from here — the skills load on their own.
