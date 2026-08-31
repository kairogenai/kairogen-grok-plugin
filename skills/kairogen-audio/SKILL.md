---
name: kairogen-audio
description: >-
  Generate speech, sound effects, and music with Kairogen, and add audio to
  video — narration, dubbing, and voice replacement. Use for "text to speech",
  "read this out loud", "generate a sound effect", "background music",
  "narrate this video", "add a voice-over", "dub this to English", "translate
  this video", "change the voice", "clone my voice", "which voice should I
  use". Read the `kairogen` skill first for auth, polling, and cost rules.
---

# Kairogen — audio

There are two families of audio tools, and picking the wrong one is the main
failure mode. Route first, then read the section for that tool.

## Route the request

**Standalone audio — synchronous, returns the file immediately:**

| The user wants | Call |
|---|---|
| Spoken audio from a script, no video | `generate_audio` with `kind: "speech"` |
| A sound effect | `generate_audio` with `kind: "sound-effect"` |
| Music | `generate_audio` with `kind: "music"` |

**Audio applied to a video — asynchronous jobs, each with its own tool:**

| The user wants | Call | Poll with `job_kind` |
|---|---|---|
| Add narration / voice-over to a video | `create_narration` | `narration` |
| Translate or dub a video to another language | `create_dubbing` | `dubbing` |
| Replace the spoken voice, same language | `create_video_voice_change` | `video-voice-change` |
| Swap the voice on an existing Kairogen narration | `create_voice_alteration` | `voice-alteration` |

Never use `generate_audio` for a video workflow — it produces a loose audio
file and does nothing to the video. Never use `create_narration` when the user
already has a finished narration audio file — that tool only synthesizes
speech from a script. Muxing existing audio onto a video is not available on
this host; see *create_narration* below.

Cost quotes for every kind come from `estimate_audio_cost`. For dubbing,
voice-change, and narration, pass `source_generation_id` or `source_video_ref`
so the backend can infer duration — without it, the quote is meaningless.

## Picking a voice

`list_voices` is a widget-only tool. Grok Build does not render widgets, so its
instruction to "always call it when the user asks to browse voices" does not
apply here — calling it produces nothing the user can see.

Use **`get_voices_data`** instead, for both cases: resolving a `voice_id`
yourself and showing the user their options. It returns the same unified JSON
(provider voices plus custom clones, `source` = `all` | `provider` | `clone`,
paginated via `page_token`). When the user is choosing, format a short list in
your reply — name, language, and a one-line character description, a handful at
a time — rather than dumping raw JSON at them.

Preview URLs come back in the payload; give the user the link so they can
listen before you spend credits on a full narration.

## Cloning a voice

The clone widget does not render here either, but the flow works headlessly:

1. Get the sample to Kairogen with `upload_reference_audio`, using
   `purpose: "voice-clone"`. It takes base64 audio and returns a `file://`
   reference plus a public URL. Samples run 10s–2min; warn the user that
   base64-encoding a long sample costs a lot of context, and prefer a short
   clean one.
2. `create_voice_clone` with the `sample_file_refs` from step 1.
3. `preview_voice_clone` if the response has no `previewUrl`, and give the user
   the preview link.
4. Use the resulting `voiceId` as `voice_id` (speech, narration) or
   `target_voice_id` (voice change, voice alteration).

`delete_voice_clone` is permanent — only on an explicit request.

## generate_audio

Required fields per kind:

- `speech` — `prompt` (the script) and `voice_id`. Optional `language_code`,
  `voice_settings`.
- `sound-effect` — `prompt`. Optional `duration_seconds`, `prompt_influence`,
  `loop`.
- `music` — `prompt`. Optional `duration_seconds`, `instrumental`.

It is synchronous: the audio comes back in the response. Read the URL out of
the result and give it to the user.

## create_narration

Ask one question before anything else: **does the user already have the
narration audio, or should Kairogen generate it?**

- **They have it** — stop. `create_narration` requires `script` + `voice_id`
  and cannot attach an existing audio file. The graphical path
  (`open_upload_reference_widget`) does not render here. Tell the user to mux
  the ready audio onto the video in the Kairogen web app
  (https://app.kairogen.ai), then pick the result back up here by
  `generation_id` if they want to continue. Do not invent a pipeline, and do
  not call `open_*_widget`.
- **Kairogen generates it** — collect, before calling:
  - the source video: `source_generation_id`, or `source_video_ref` from an
    upload / public URL;
  - the script — ask what to narrate, or offer to write one;
  - tone and style, and the language (`language_code`, e.g. `pt-BR`, `en`);
  - the voice, resolved through `get_voices_data`.

Also settle what they actually want out of it: just the script (answer in
chat), audio with no video (`generate_audio` with `kind: "speech"`), or a
narrated video (`create_narration`). Required fields are `script` +
`voice_id` + one of `source_generation_id` / `source_video_ref`.

## create_dubbing

Ask the target language (`target_language`: `pt`, `en`, `es`, `fr`, …) and
confirm the source video. Ask about accent or region when it matters for the
audience. Dubbing replaces the spoken audio in the target language — if the
user wants a different voice in the *same* language, that is
`create_video_voice_change`.

## create_video_voice_change

Ask whether to use a catalog voice (`get_voices_data`) or a custom clone (the
cloning flow above), confirm the source video, then call it with
`target_voice_id` plus `source_generation_id` / `source_video_ref`.

## create_voice_alteration

For a narration Kairogen already produced. Needs `source_narration_id` — from
the earlier `create_narration` response or from `list_audio_history` — plus
`target_voice_id`. It re-voices the narration without reprocessing the whole
video, so it is the cheap path when only the voice was wrong.

## Polling the async jobs

All four `create_*` tools return a job id and finish in the background. The
audio-result widget that normally polls them does not exist here, so you poll:
`get_audio_job_status` with the job id and the matching `job_kind` from the
routing table. It returns normalized status, `done: true` when terminal, and
output URLs when available.

Tell the user the job id in plain text before you start waiting. If they leave
and come back, that id plus `get_audio_job_status` is the only way to recover
the job — and `list_audio_history` (default `page: 1`, `limit: 12`, covering
speech, sound-effect, music, narration, dubbing, video-voice-change, and
voice-alteration) is the fallback when they lost it.

Do not re-run a create_* tool because a poll is still pending. Audio jobs on
video are billed per run.
