---
name: kairogen-video
description: >-
  Generate video with Kairogen — text-to-video and image-to-video. Use for
  "make a video", "animate this image", "turn this photo into a clip", "video
  from this prompt", "first frame / last frame", "which video model", and for
  choosing between Sora, Veo, Kling, Seedance, and KairoClone. Also covers the
  extra_params key remapping that causes most failed video generations. Read
  the `kairogen` skill first for auth, polling, and cost rules.
---

# Kairogen — video

Tools: `list_models`, `estimate_cost`, `generate_video`,
`animate_from_generation`, `get_generation`, `recreate_generation`.

Video is the most expensive thing in the catalog and the slowest — most clips
land in 5–12 minutes. Quote the cost before generating, and never start a batch
without checking plan concurrency first (`kairogen` skill).

## Read this before your first call: parameter remapping

This is the single most common cause of failed video generations.

`generate_video` exposes generic arguments (`first_frame_url`,
`last_frame_url`, `reference_image_urls`). Many models define **different key
names** in their own `param_schema`. When they do, the value must go inside
`extra_params` under the schema's key, and the generic argument must not be
sent at all.

The common remappings:

| Generic argument | Model schema key |
|---|---|
| `first_frame_url` | `extra_params.image` |
| `last_frame_url` | `extra_params.last_image` |
| `reference_image_urls` | `extra_params.reference_images` |

So the rule is: call `list_models`, read the chosen model's `param_schema`, and
if it defines `image` / `last_image` / `reference_images`, send those keys via
`extra_params` — for example `extra_params: { "image": "https://..." }` — and
leave `first_frame_url` and friends unset. If the schema uses the generic names,
use the generic arguments. Check per model; do not assume it carries over from
the last one you used.

## Text-to-video or image-to-video

**Image-to-video (i2v)** — there is a starting image. Model ids in this family
carry `image-to-video` in the name (`seedance-v2-0-image-to-video`,
`kling-o1-image-to-video`, `kairoclone-v3-image-to-video`). The start frame is
required; without it the call fails. This is the reliable path when the look
has to match something exact, so when the user cares about a specific subject,
generate the frame with `generate_image` first and animate that.

**Text-to-video (t2v)** — no source image. `first_frame_url` is ignored by
these models even if you send it.

If the user already has a Kairogen image generation, `animate_from_generation`
is shorter than wiring the URL by hand: pass the image's `generation_id` and
optionally `video_model`.

## Choosing a model

Filter `list_models` to video and read `supported_durations`, `param_schema`,
and price for the candidates. Starting heuristics — confirm against the
catalog, which wins over this list:

- **Native audio in the clip** (dialogue, sound) — the Sora and Veo families
  are the ones that generate sound with the video. If the user wants speech
  over an existing clip instead, that is `kairogen-audio`, not this.
- **Animating a specific still** — the Seedance, Kling, and KairoClone i2v
  variants.
- **Longest reach and multimodal references** (reference images, reference
  video, reference audio) — the Seedance 2.0 family exposes the most inputs.
- **Fast and cheap iteration** — the `fast` variants (e.g. `veo-3-1-fast`).
  Use these while the user is still deciding what they want, then re-run the
  winning prompt on the full-quality model.

Always tell the user which model you picked and roughly what it costs before
you spend the credits.

## Writing the call

- `duration_seconds` — must be one of the model's `supported_durations` from
  `list_models`. An unsupported value is rejected, never adjusted (Seedance,
  for example, allows 5 / 10 / 15).
- `aspect_ratio` — `'16:9'`, `'9:16'`, `'1:1'`, model-dependent. Ask which
  platform the video is for if the user has not said; vertical and horizontal
  are not interchangeable and a re-render costs full price.
- `quantity` — max 2, and gated by plan concurrency.
- `seed` — model-dependent; use it to iterate on a near-miss.
- `reference_video_urls` / `reference_audio_urls` — only on multimodal models
  such as Seedance 2.0, and subject to the remapping rule above. For
  motion-control models, prefer `extra_params.video`.

## Waiting for the result

With no widget to poll, decide up front (see the `kairogen` skill):

- **Blocking:** `wait_for_completion: true` with `timeout_seconds` — default
  900s, max 1800s. Raise it for long or premium renders. Progress
  notifications arrive every 5 seconds, so the connection holds.
- **Async:** keep the default, and tell the user the `generation_id` in plain
  text so it is not lost. Call `get_generation` when they come back for it —
  that tool polls up to 20 minutes on its own.

Do not start a batch of videos with the async default and no id reported. That
is how a user pays for renders nobody ever collects.

Read `output_urls` from the JSON summary and give the user the link. To save a
copy locally, download the URL with a shell command — not
`download_image_from_url`, which returns base64 for a widget.

## When a generation fails

- **"Unexpected parameter" / the start frame was ignored** — the remapping rule
  at the top of this skill. Move the value into `extra_params` under the
  schema's key.
- **i2v model with no image** — supply `first_frame_url` (or
  `extra_params.image`), or switch to a t2v model.
- **Duration rejected** — retry with a value from `supported_durations`.
- **Plan or quantity error** — retry at the allowed size, then loop. Report the
  limit, but deliver the videos.
- **Timeout** — the generation is still running server-side. Do not re-generate
  and double-charge the user; poll `get_generation` with the id instead.
