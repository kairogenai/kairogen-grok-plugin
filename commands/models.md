---
description: Browse the Kairogen model catalog with prices, filtered by what you need
argument-hint: [image|video|audio or a description of the job]
---

# Kairogen Models

Show the user what Kairogen can run right now.

Call `list_models`. Filter by the argument if one was given (`image`, `video`,
or a free-text description of the job); otherwise show both image and video.

Present a compact table — model id, what it is good for, key constraints
(supported durations, whether it takes reference images), and price. Sort by
price ascending within each medium so the cheap options are visible first. Do
not paste the raw JSON, and do not list every parameter of every model.

Then, in one or two lines: name the model you would pick for the job the user
described, and say why. If they gave no job description, name the sensible
default for images and the sensible default for video, and mention that
`estimate_cost` quotes any of them before a run.

If the user asked about audio, `list_models` does not cover it — audio models
are selected per `kind` inside `generate_audio`, and voices come from
`get_voices_data`. Say so and offer to list voices instead.
