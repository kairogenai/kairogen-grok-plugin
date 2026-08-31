---
name: kairogen-image
description: >-
  Generate, edit, and upscale images with Kairogen. Use for "generate an
  image", "create a picture", "make me a logo/poster/thumbnail", "edit this
  image", "change the background", "same character in another scene", "make
  this higher resolution", "upscale", "4x", "which image model should I use",
  and for picking between Seedream, Flux, Nano Banana, GPT Image, and Topaz.
  Read the `kairogen` skill first for auth, polling, and cost rules.
---

# Kairogen — images

Tools: `list_models`, `estimate_cost`, `generate_image`, `upscale_image`,
`get_generation`, `recreate_generation`, `animate_from_generation`.

Grok Build does not render Kairogen's inline widgets. The `kairogen` skill
explains the four overrides that follow from that — polling, URLs, uploads, and
the `open_*_widget` tools. Apply them here.

## The loop

1. `list_models` with the image filter — the live catalog is the source of
   truth for ids, parameters, and price.
2. Choose a model (below), and read its `param_schema`.
3. `estimate_cost` when the request is batched, expensive, or exploratory.
4. `generate_image` with `wait_for_completion: true`.
5. Put the URL from `output_urls` in your reply.

## Choosing a model

Ask two questions before you look at names.

**Is there a source image?** If the user wants to change an existing image —
new background, different outfit, remove an object, restyle — that is an edit,
and it goes through `reference_image_urls`, not a from-scratch prompt. Filter
`list_models` for models whose `param_schema` accepts reference or edit inputs;
a text-only model handed a reference will quietly ignore it and generate
something unrelated.

**What has to be right?** These are starting heuristics — confirm against the
descriptions and `param_schema` that `list_models` returns, and prefer what the
catalog says over what is written here:

- **Legible text in the image** (posters, UI mockups, packaging, memes) — the
  GPT Image and Nano Banana families are the ones built for instruction
  following and text rendering.
- **Instructed edits of an existing image** — the Nano Banana family is the
  editing specialist; hand it the reference and describe the change.
- **Aesthetic quality and prompt adherence from scratch** — the Flux 2 family.
- **Photoreal work and high native resolution** — the Seedream family.
- **The image already exists and just needs to be bigger or cleaner** — this is
  not a generation at all. Use `upscale_image`.

When the user has no preference and no special constraint, pick the current
default-tier model from `list_models` rather than the most expensive one, and
say which you picked and why. Offer the premium tier as an upgrade instead of
spending their credits on it unasked.

## Writing the call

- `prompt` — carry the user's intent, do not inflate it. If they asked for
  something simple, do not add ten adjectives of your own; if they asked for
  something vague, ask one clarifying question rather than guessing a style.
- `aspect_ratio` — `'1:1'`, `'16:9'`, `'9:16'`, `'3:4'`, `'4:3'`, and
  model-dependent. `size` (e.g. `'1024x1024'`) overrides it when both are set.
  Do not send both.
- `quantity` — max 4, and gated by plan concurrency. Read the batching rules in
  the `kairogen` skill before sending more than 1.
- `reference_image_urls` — public URLs. Getting a local file to a public URL is
  covered in the `kairogen` skill.
- `negative_prompt` — not every model honors it. Check `param_schema` before
  relying on it to fix a problem; reworking the prompt is usually better.
- `seed` — pass one when the user wants to iterate on a result rather than roll
  again. Same seed plus same params reproduces the image; changing the prompt
  slightly then explores near it.
- `extra_params` — the escape hatch for anything in `param_schema` that has no
  generic argument. Send the schema's own key name.

## Consistency across images

Without the character wizard (see `kairogen` skill), consistency comes from
references and seeds:

- Generate a first image and keep its output URL.
- Pass that URL in `reference_image_urls` on every follow-up, with a prompt
  that describes only what changes ("same woman, same jacket, now on a subway
  platform at night").
- Hold `seed` fixed while you iterate on wording; change it when you want a
  genuinely different take.

Use an edit-capable model for this. A text-only model will not hold the face.

## Upscaling

`upscale_image` runs Topaz Photo AI at 2x or 4x, with optional face
enhancement. It is the right tool whenever the source already exists —
including an image Kairogen just generated. Quote the cost first: a 4x upscale
is not a rounding error, and the output file is large.

Face enhancement helps portraits and hurts stylized or illustrated faces. Ask
if it is not obvious which one you are looking at.

## Handing off to video

`animate_from_generation` takes a generation id from an image you just made and
uses its first output as the starting frame of a video. It defaults to
`seedance-v2-0-image-to-video` and accepts `video_model` to switch. Duration
snaps to each model's allowed values. For anything beyond a straight animate —
choosing the model, controlling the motion, a last frame — read
`kairogen-video`.

## When a generation fails

- **Plan or quantity error** — retry at the allowed size, then loop. Never drop
  the request. The `kairogen` skill has the procedure.
- **Rejected or unexpected parameter** — you sent a generic argument where the
  model's `param_schema` defines its own key. Move it into `extra_params` under
  the schema's name.
- **Auth error** — `/mcp` → **kairogen** → `i` to sign in again.
- **Reference ignored** — the model is text-only. Switch to an edit-capable one.
