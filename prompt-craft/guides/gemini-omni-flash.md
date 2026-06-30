# Gemini Omni Flash — The Agent's Video Prompting Guide

**Audience:** Claude-based and Gemini-based agents producing video via Gemini Omni Flash.
**Scope:** Video generation, image-to-video, stateful video editing, and audio direction for agent pipelines.
**Model ID:** `gemini-omni-flash-preview`
**API:** Interactions API (not the Generate Content API used by image models)
**Status:** Preview — in active development; expect feature additions and breaking changes.
**Last refreshed:** June 30, 2026
**License:** CC BY 4.0 — see repo root LICENSE.md
**Sources:** Google AI for Developers — Gemini API Docs (ai.google.dev/gemini-api/docs/omni), cited in Sources section.

---

## TL;DR for agents (read this first)

1. **Use the Interactions API, not the Generate Content API.** The model ID is `gemini-omni-flash-preview`. Calls go through `client.interactions.create(...)`, not `client.models.generate_content(...)`.
2. **By default, the model cuts between multiple shots.** If you want one continuous scene, explicitly say: *"In a single unbroken scene"* or *"No scene cuts."*
3. **Simple editing prompts outperform descriptive ones.** Use natural language deltas: *"Make the violin invisible. Keep everything else the same."* Long re-descriptions of the full scene cause unintended changes.
4. **Stateful editing uses `previous_interaction_id`.** Reference the prior interaction's ID — you never need to re-upload the previous video.
5. **Tags bind reference images to roles.** Use `<FIRST_FRAME>` for the starting frame and `<IMAGE_REF_N>` for subject/style references in your prompt text.
6. **Audio is generated automatically.** Direct it explicitly if you want something specific; suppress it with *"No dialogue"* or *"No sound effects"* if you need silence.
7. **For videos > 4MB, use URI delivery.** Set `response_format: {type: "video", delivery: "uri"}` and poll the Files API for `ACTIVE` state before downloading.
8. **No system instructions, temperature, top_p, or negative prompts at the API level.** Fold negatives into the regular prompt: *"No dialogue"*, *"No scene cuts"*, *"Avoid slow motion."*
9. **Text in video works well.** Define exact strings and where they appear. Omni will render them readable — even in motion.
10. **Timing events use natural language.** *"After 3 seconds, a woman enters the frame."* or `[0-3s] Walking [3-6s] Stops` — both work.

---

## 1. What Gemini Omni Flash actually is

Gemini Omni Flash (`gemini-omni-flash-preview`) is Google's high-speed video generation and editing model. Three capabilities distinguish it from prior video models:

**Native multimodality.** Processes text, image, audio, and video simultaneously. You can anchor a video from a still image, use multiple character references in a single generation call, and apply style references alongside scene references — all in one prompt.

**Conversational editing via the Interactions API.** Unlike generation-only video models, Omni Flash supports stateful multi-turn editing. Each turn builds on the prior video. You reference prior turns using `previous_interaction_id` — no re-uploading required. The model retains context: adjust lighting, swap background, add characters — without re-describing the whole scene.

**World knowledge + physics.** Trained on Gemini's broader knowledge base. Understands physics, cultural context, history, and real-world subjects. This bridges photorealism and storytelling in ways that generation-only models can't.

### 1.1 Core capabilities at a glance

| Capability | Support | Notes |
| :--- | :--- | :--- |
| Text-to-video | ✅ | Multi-shot by default; use "single unbroken scene" for continuous |
| Image-to-video | ✅ | Use `<FIRST_FRAME>` tag or `task: image_to_video` |
| Reference-based generation | ✅ | Use `<IMAGE_REF_N>` tags for subject/style anchors |
| Stateful multi-turn editing | ✅ | `previous_interaction_id` tracks conversation state |
| Upload your own video & edit | ✅ | Files API upload → pass URI in input (regional restrictions apply) |
| Audio generation | ✅ | Auto-generated; directible via prompt |
| Voice/dialogue editing | ❌ | Not supported in current version |
| Multi-video referencing | ❌ | Degrades model performance |
| Video extension / interpolation | ❌ | Not supported |
| YouTube as media source | ❌ | Not supported |
| System instructions | ❌ | Use prompt text instead |
| Negative prompts (API field) | ❌ | Fold negatives into the regular prompt |
| Aspect ratios | 16:9 (default), 9:16 | Landscape and portrait |
| SynthID watermarking | ✅ | Automatic, invisible to viewers |

---

## 2. The Interactions API — essential differences from image models

Omni Flash does NOT use the same API as Nano Banana Pro. This is the most common integration mistake.

### 2.1 Core call structure

```python
from google import genai

client = genai.Client()

interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="A marble rolling fast on a chain reaction track, continuous smooth shot.",
)

# Access the output video
video_data = interaction.output_video.data  # Base64-encoded MP4
```

### 2.2 REST response schema

The REST API does not have the `output_video` convenience field — that is SDK-only. For raw REST, extract video from the `steps` array:

```json
{
  "steps": [
    {"type": "user_input", "content": [{"type": "text", "text": "..."}]},
    {"type": "thought", "content": [{"text": "...", "type": "thought"}]},
    {
      "type": "model_output",
      "content": [{
        "type": "video",
        "mime_type": "video/mp4",
        "data": "AAAAIGZ0eXBpc29t..."
      }]
    }
  ],
  "id": "v1_...",
  "status": "completed",
  "model": "gemini-omni-flash-preview",
  "object": "interaction"
}
```

### 2.3 Aspect ratio control

```python
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="A futuristic city with neon lights and flying cars, cyberpunk style",
    response_format={
        "type": "video",
        "aspect_ratio": "9:16"   # Supported: "9:16", "16:9" (default)
    }
)
```

### 2.4 Task parameter (optional but recommended)

Explicitly set `task` in `generation_config.video_config` to avoid the model inferring incorrectly:

```python
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "image", "data": base64_image, "mime_type": "image/jpeg"},
        {"type": "text", "text": "Bring this scene to life with a slow camera pull-back."}
    ],
    generation_config={
        "video_config": {
            "task": "image_to_video"   # text_to_video | image_to_video | reference_to_video | edit
        }
    },
)
```

### 2.5 URI delivery (for large videos)

Use `delivery: "uri"` when you expect videos > 4MB (typical for anything above 720p or > 8 seconds). Returns a URI you poll until `ACTIVE`:

```python
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="A sweeping aerial shot of the Himalayas at dawn.",
    response_format={
        "type": "video",
        "delivery": "uri"
    }
)

# Poll for ACTIVE state before downloading
import time
file_name = interaction.output_video.uri.split("/")[-1]
while True:
    f_info = client.files.get(name=f"files/{file_name}")
    if f_info.state.name == "ACTIVE":
        break
    elif f_info.state.name == "FAILED":
        raise RuntimeError("Generation failed.")
    time.sleep(5)

video_bytes = client.files.download(file=interaction.output_video.uri)
with open("output.mp4", "wb") as f:
    f.write(video_bytes)
```

### 2.6 Performance optimization

```python
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="...",
    response_format={
        "type": "video",
        "background": False,   # Don't run as background job
        "store": False,        # Don't store for editing (faster, but kills stateful follow-up)
        "stream": False        # Synchronous unary call
    }
)
```

**Important:** Setting `store=False` disables `previous_interaction_id` editing on that output. Only use when stateful follow-up is not needed.

---

## 3. Prompt structure — the video formula

Omni Flash is not a keyword-matcher. It reasons over your prompt before generating. Write **scene descriptions**, not tag lists.

### 3.1 The 6-element video prompt formula

```
[Subject] + [Action] + [Environment] + [Camera] + [Audio] + [Constraints]
```

| Element | What it is | Examples |
| :--- | :--- | :--- |
| **Subject** | The primary entity — describe specifically | "A fluffy tabby cat", "A woman in her 40s with silver hair" |
| **Action** | What the subject does and how | "Bats at a ball of yarn", "Walks slowly through rain" |
| **Environment** | Location, time, weather, atmosphere | "In a sunlit apartment", "On a rain-slicked street at dusk" |
| **Camera** | Shot type, angle, movement | "Handheld close-up", "Slow orbital pull-back", "Fixed wide shot" |
| **Audio** | Sound design, music, dialogue direction | "Ambient street noise, no dialogue", "Low cinematic score builds" |
| **Constraints** | What must NOT happen | "No scene cuts", "No dialogue", "No text overlays" |

### 3.2 Single-scene generation

By default, Omni Flash generates multi-shot videos with narrative cuts. To override:

```
❌ "A cat sitting on a windowsill"
   → Will likely cut between angles

✅ "In a single unbroken scene, a fluffy tabby cat sits on a sunny windowsill,
   looking out into a leafy garden. The cat's tail twitches slowly, and its
   ears rotate slightly toward ambient noises. Continuous handheld shot.
   No scene cuts."
```

The phrases *"single unbroken scene"*, *"continuous shot"*, *"no scene cuts"*, *"single continuous shot"* all reliably enforce single-scene output.

### 3.3 Multi-shot / narrative generation

Lean into the default multi-shot behavior for storytelling or rapid-fire sequences. The model crafts interesting narratives from brief descriptions.

```
A chef preparing dinner service. Show the whole arc — mise en place at dusk,
the kitchen heat rising, plates going out, the last quiet moment after close.
No dialogue. Kitchen ambience and light jazz.
```

### 3.4 Timing events

Use natural language timing — no special syntax required:

```
After 3 seconds, a woman enters the scene from the left.
At 5s, a low cinematic score begins in the background.
Every 2s cut to a new location.
```

Or use timecode blocks for precision:

```
[0-3s] A person is walking along a forest path.
[3-6s] They stop and turn around, looking puzzled.
[6-10s] They start running, leaves scattering.
```

---

## 4. Image-to-video and reference generation

### 4.1 Simple I2V (first-frame approach)

Provide an image and a motion description. The model uses the image as the starting frame:

```python
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "image", "data": base64_image, "mime_type": "image/jpeg"},
        {"type": "text", "text": "The product slowly rotates 360°. Soft studio lighting holds constant. No scene cuts."}
    ],
    generation_config={"video_config": {"task": "image_to_video"}}
)
```

**Prompt tips for I2V:**
- Use high-resolution source images (≥1024px)
- Describe motion specifically — not *"make it move"* but *"slow orbital pull-back, subject holds center frame"*
- Add *"No scene cuts"* if you want the starting frame to persist as-is

### 4.2 Using the `<FIRST_FRAME>` tag

In the prompt text, reference the image as the first frame:

```python
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "image", "data": base64_image, "mime_type": "image/jpeg"},
        {"type": "text", "text": "<FIRST_FRAME> The violin player slowly rises from her chair and walks to center stage. Warm spotlight follows. No scene cuts."}
    ],
)
```

### 4.3 Subject reference with `<IMAGE_REF_N>`

Provide character or object references for the model to incorporate as subjects — without using them as literal starting frames:

```python
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "image", "data": cat_b64, "mime_type": "image/png"},     # IMAGE_REF_0
        {"type": "image", "data": yarn_b64, "mime_type": "image/png"},    # IMAGE_REF_1
        {"type": "text", "text": "A cat <IMAGE_REF_0> playfully batting at a ball of yarn <IMAGE_REF_1>. Warm apartment light. No scene cuts."}
    ],
    generation_config={"video_config": {"task": "reference_to_video"}}
)
```

Image references in prompt text are **zero-indexed**: first image = `<IMAGE_REF_0>`, second = `<IMAGE_REF_1>`, etc.

### 4.4 Complex multi-reference with explicit declarations

For scenes with multiple characters and multiple props, use the explicit declaration syntax:

```python
# 6 images: 3 characters, 3 props
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "image", "data": woman1_b64, "mime_type": "image/png"},   # REF 0
        {"type": "image", "data": item1_b64, "mime_type": "image/png"},    # REF 1
        {"type": "image", "data": man_b64, "mime_type": "image/png"},      # REF 2
        {"type": "image", "data": item2_b64, "mime_type": "image/png"},    # REF 3
        {"type": "image", "data": woman2_b64, "mime_type": "image/png"},   # REF 4
        {"type": "image", "data": item3_b64, "mime_type": "image/png"},    # REF 5
        {"type": "text", "text": """
[# References <IMAGE_REF_0>@Image1 <IMAGE_REF_1>@Image2 <IMAGE_REF_2>@Image3 <IMAGE_REF_3>@Image4 <IMAGE_REF_4>@Image5 <IMAGE_REF_5>@Image6]

[0-3s] A studio fashion sequence. Starting with woman <IMAGE_REF_0>, she is holding <IMAGE_REF_1>
[3-6s] Then we see the man <IMAGE_REF_2> holding <IMAGE_REF_3>
[6-10s] And finally another woman <IMAGE_REF_4> who is holding <IMAGE_REF_5> while walking.

Use the given images as references for video generation. The images should not be used as literal initial frames.
"""}
    ],
)
```

### 4.5 First-frame + reference (combined)

Use both tags together when you want a specific starting frame AND a reference subject:

```python
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "image", "data": scene_b64, "mime_type": "image/jpeg"},    # Starting frame
        {"type": "image", "data": person_b64, "mime_type": "image/jpeg"},   # Subject reference
        {"type": "text", "text": """
[# Sources <FIRST_FRAME>@Image1]
[# References <IMAGE_REF_0>@Image2]

A woman <IMAGE_REF_0> walks into the scene and sits at the empty chair.

Use Image1 as the starting frame.
Use Image2 as a reference for the video generation. The image should not be used as a literal initial frame.
"""}
    ],
)
```

---

## 5. Stateful video editing

This is Omni Flash's most powerful differentiator from generation-only video models.

### 5.1 The `previous_interaction_id` pattern

```python
# Turn 1: Generate initial video
res1 = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="A woman playing violin in a park at golden hour."
)

# Turn 2: Edit — model retains full context from res1
res2 = client.interactions.create(
    model="gemini-omni-flash-preview",
    previous_interaction_id=res1.id,
    input="Make the violin invisible. Keep everything else the same."
)

# Turn 3: Further edit on top of res2
res3 = client.interactions.create(
    model="gemini-omni-flash-preview",
    previous_interaction_id=res2.id,
    input="Change the lighting to dusk. Add a warm orange tint to the sky. Keep everything else the same."
)
```

Each turn produces a new video. The model understands what changed and what must be preserved — without you re-describing the whole scene.

### 5.2 The three-sentence editing pattern

The most reliable editing structure:

```
Sentence 1: [What changes]
Sentence 2: [What stays exactly the same]
Sentence 3: [Technical behavior, if needed]
```

Example:
```
Add a small tabby cat that jumps onto the violinist's lap.
Keep the person, park, lighting, and music exactly the same.
The cat's arrival should feel natural — no sudden cuts.
```

### 5.3 Simple edits vs. complex edits

| Use simple edits for | Use explicit preserve lists for |
| :--- | :--- |
| Single-element additions/removals | Multi-element scenes where drift risk is high |
| Style transformations ("Make this anime") | Identity-sensitive subjects (specific people, products) |
| Background swaps | Text changes on signs/labels |

**Simple:** *"Put a fashionable hat on this person."*
**Explicit:** *"Change the hat on the person from a baseball cap to a fedora. Preserve the person's face, clothing, pose, lighting, background, and all other elements exactly."*

### 5.4 Editing your own uploaded videos

Upload with the Files API, then pass the URI:

```python
import time

# Upload
video_file = client.files.upload(file="my_video.mp4")
while video_file.state == "PROCESSING":
    time.sleep(10)
    video_file = client.files.get(name=video_file.name)
if video_file.state == "FAILED":
    raise ValueError(video_file.state)

# Edit
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "document", "uri": video_file.uri},
        {"type": "text", "text": "Change the text on the street sign to say 'OMNI FLASH'. Keep everything else the same."}
    ],
)
```

**Regional restriction:** Uploading and editing user-provided videos is not available in the EEA, Switzerland, or UK. Editing AI-generated videos via `previous_interaction_id` IS available in those regions.

---

## 6. Audio direction

Audio is generated automatically based on the scene. Actively direct it to get what you want.

### 6.1 Describing the audio

Include audio direction in the prompt, after the visual description:

```
A chef preparing mise en place at 5 PM.
Sound design: kitchen ambience — knife on cutting board, distant radio playing jazz,
occasional pot clatter. No dialogue.
```

For music:
```
[0-5s] The runner approaches the finish line.
Audio: Low, building cinematic score. Percussion enters at 3s. Climax at 5s.
No crowd noise.
```

For silence:
```
A slow-motion flower opening in morning dew. 
Silent. No audio.
```

### 6.2 Audio tags that work reliably

| Intent | Prompt phrase |
| :--- | :--- |
| Silence | `"Silent."` or `"No audio."` |
| No speech | `"No dialogue."` |
| Clean ambience | `"Ambient sound only. No music."` |
| Specific music style | `"Include calm background music"` / `"High-energy techno beat"` |
| Diegetic radio | `"A low tinny radio broadcast in the background, playing a jazz song"` |
| Timed music entry | `"At 5s the chorus starts in the background audio"` |

---

## 7. Text in videos

Omni Flash renders text in video well. Treat text as a layout element, not just language.

### 7.1 Text rendering

Quote exact strings and describe placement:

```
One word appears on screen at a time: "did", "you", "know", "that", "Omni", "can", "do", "text".
Each word appears for 1 second with a different animated style. Clean white text on black. No dialogue.
```

For text in scene elements:
```
There is a street sign that says: "This is Omni Flash"
There is a storefront that says: "All You Need AI"
There's a car with the number plate: "OMN111"
```

### 7.2 Editing existing text

Using stateful editing to change text is one of Omni Flash's strongest use cases:

```
Change the text on the sign to say "FLASH SALE".
Keep everything else the same.
```

---

## 8. Prompt patterns — by use case

### 8.1 Product visualization

```
<FIRST_FRAME> [attach product image]

The product rotates slowly 360° in place, center frame.
Three-point studio lighting, soft key from camera-left.
Reflective tabletop surface. Dark gradient background.
No scene cuts. No text. No dialogue.
```

### 8.2 Character-driven narrative

```python
# Attach 2 character reference images
input = [
    {"type": "image", "data": char_b64, "mime_type": "image/png"},
    {"type": "text", "text": """
A woman <IMAGE_REF_0> closes the door to a rain-soaked apartment.
She drops her bag, walks to the window, watches the street below.
[0-5s] Door close, bag drop — diegetic sound only.
[5-10s] She reaches the window. Rain on glass. No dialogue. Low ambient music begins.
Single unbroken shot. No scene cuts.
"""}
]
```

### 8.3 Rapid-fire visual sequence

```
Make a rapid-fire video that shows a different rare fish every 1 second.
Upbeat electronic music. Include text labels naming each fish.
High energy. 10 seconds total.
```

### 8.4 Collectible / sports content

```python
# Attach a still sports photo
input = [
    {"type": "image", "data": photo_b64, "mime_type": "image/jpeg"},
    {"type": "text", "text": """
<FIRST_FRAME> 
[0-2s] Freeze-frame effect on this moment. Particle effects emanate from the subject.
[2-4s] Slow camera pull-back reveals the arena crowd.
[4-6s] The photo becomes an in-game highlight card. Text appears: "LEGENDARY PLAY".
Sound design: Crowd roar builds. Cinematic hit at 2s. No dialogue.
"""}
]
```

### 8.5 Logo / brand animation

```
Animate this logo with a simple, elegant reveal.
[0-1s] Dark background, silence.
[1-2s] The logo draws in from a single point, particle trail.
[2-3s] Logo fully formed, subtle glow pulse. Holds.
Sound design: Soft cinematic swell. Resolve at 3s. No dialogue.
No additional text.
```

### 8.6 Style transfer on existing video

```python
# Upload existing video, then:
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "document", "uri": video_file.uri},
        {"type": "text", "text": "Make this video anime. Keep all motion and timing the same."}
    ],
)
```

---

## 9. Meta-prompting — quality multipliers

These general-direction prompts ask the model to apply broader quality principles:

```
Consider micro-detail, expression, and timing to create a very rich,
detailed but entirely natural scene.

Be extremely detailed in your descriptions of characters and environments.

Apply costume design principles to characters.

Be very specific about the people, items, and objects in the scene.

Include plenty of appropriate detail in the background elements to make
the scene feel realistic and natural.
```

Use meta-prompting when creative freedom is preferred and you want maximum richness — not when you need precise control.

---

## 10. Failure modes and fixes

### Video generation

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| Unwanted scene cuts / camera switches | No single-scene constraint | Add "In a single unbroken scene" and "No scene cuts" |
| Generic, low-quality motion | Vague motion description | Describe specific motion: camera angle, subject action, speed |
| Audio doesn't match scene | No audio direction | Add explicit sound design note after visual description |
| Unwanted dialogue or voice | Model adds ambient speech | Add "No dialogue" |
| Character doesn't look like reference | Vague `<IMAGE_REF_N>` tag usage | Label roles explicitly; use `[# References ...]` syntax for complex scenes |
| Text rendered incorrectly | Text not quoted exactly | Quote strings in double quotes; describe font and placement |
| Video too long / too short | No duration constraint | Add "[X seconds total]" or use timecode blocks |
| Editing changed the whole scene | No "Keep everything else the same" | End every edit prompt with "Keep everything else the same." |
| Model ignored the edit | Edit prompt too complex | Simplify: describe only the change, not the whole scene |
| Identity drift across edit turns | Character not anchored | Re-attach reference image in the edit turn |

### API / integration

| Symptom | Likely cause | Fix |
| :--- | :--- | :--- |
| `output_video` undefined in REST | REST does not have SDK convenience field | Extract from `steps[].content[].data` where type=video |
| Video payload too large | Inline base64 over 4MB | Use `delivery: "uri"` and poll Files API |
| Edit doesn't persist state | `store: false` was set | Only set `store: false` when stateful editing is not needed |
| 400 error on `negativePrompt` | API has no `negativePrompt` field | Move negatives into regular prompt: "No X, no Y" |
| Regional block on video upload | EEA/Switzerland/UK restriction | Use model-generated video editing (`previous_interaction_id`) instead |

---

## 11. API quick reference

```python
# Text-to-video (basic)
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="[prompt]"
)

# Text-to-video (portrait)
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="[prompt]",
    response_format={"type": "video", "aspect_ratio": "9:16"}
)

# Image-to-video
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "image", "data": b64, "mime_type": "image/jpeg"},
        {"type": "text", "text": "[motion description]"}
    ],
    generation_config={"video_config": {"task": "image_to_video"}}
)

# Reference-to-video (multiple subject refs)
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "image", "data": ref0, "mime_type": "image/png"},
        {"type": "image", "data": ref1, "mime_type": "image/png"},
        {"type": "text", "text": "Subject <IMAGE_REF_0> interacts with <IMAGE_REF_1>. [scene description]"}
    ],
    generation_config={"video_config": {"task": "reference_to_video"}}
)

# Stateful edit
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    previous_interaction_id=prior_interaction.id,
    input="[edit description]. Keep everything else the same."
)

# Upload and edit existing video
video_file = client.files.upload(file="video.mp4")
# ... poll for ACTIVE state ...
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input=[
        {"type": "document", "uri": video_file.uri},
        {"type": "text", "text": "[edit description]. Keep everything else the same."}
    ]
)

# URI delivery (large videos)
interaction = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="[prompt]",
    response_format={"type": "video", "delivery": "uri"}
)
# ... poll files API for ACTIVE state, then download ...
```

**Tasks parameter values:** `text_to_video` | `image_to_video` | `reference_to_video` | `edit`
**Aspect ratios:** `"16:9"` (default) | `"9:16"`
**No API fields for:** system instructions, temperature, top_p, stop sequences, negative prompts

---

## 12. Positioning vs. other models in the pipeline

The existing agent-imago pipeline had Seedance 2.0 as the sole video model. Gemini Omni Flash changes the decision tree:

| Task | Recommended | Why |
| :--- | :--- | :--- |
| Generate video from text, need multi-shot narrative | **Gemini Omni Flash** | World knowledge, richer multi-shot narrative |
| Generate video from still image | **Gemini Omni Flash** | Native I2V with reference tags |
| Edit an existing AI-generated video iteratively | **Gemini Omni Flash** | Stateful `previous_interaction_id` editing |
| Edit your own uploaded video | **Gemini Omni Flash** | Files API + interaction edit (regional restrictions apply) |
| Animate a Nano Banana-generated image | **Gemini Omni Flash** (preferred) or **Seedance 2.0** | Both work; Omni Flash is more controllable via stateful editing |
| High-fidelity commercial video, need audio sync | Evaluate both; **Seedance 2.0** is proven in pipeline | Seedance 2.0 has the @reference system and V2V mode |
| V2V (video-to-video) transformation | **Seedance 2.0** | Seedance 2.0 V2V is ~39% cheaper; Omni Flash does V-style edit via stateful edit |

### Updated creative pipeline (image + video)

```
📝 creative-brief
        ↓
[For images]
Nano Banana Flash → Nano Banana Pro → (optional) GPT Image 2 for text
        ↓
[For video from image]
Gemini Omni Flash (I2V with <FIRST_FRAME> tag, stateful editing)
    or
Seedance 2.0 I2V (proven @reference system)
        ↓
[For text-to-video]
Gemini Omni Flash (richer narrative, conversational editing)
    or
Seedance 2.0 T2V (proven @reference, audio-sync patterns)
        ↓
🔍 quality-review → 🚫 output-discipline
```

**Decision rule:** When stateful iterative editing matters — especially for product visualization and collectibles work where you're refining across multiple turns — use Gemini Omni Flash. When you need the battle-tested @reference system and V2V mode, use Seedance 2.0.

---

## 13. Limitations (current — preview model)

- **Uploading and editing images containing minors** not supported in EEA, Switzerland, and UK.
- **Uploading and editing certain recognizable people** not supported.
- **Editing uploaded videos** (user-provided) not available in EEA, Switzerland, and UK.
- **Uploading audio references** unsupported.
- **Video references ≤ 3 seconds** accepted by API schema but not correctly processed.
- **Referencing/reasoning across multiple videos** not supported — degrades output.
- **Video extension / interpolation** (first frame → last frame → generate middle) not supported.
- **Voice editing** not supported.
- **Provisioned throughput** not supported.
- **System instructions, temperature, top_p, stop sequences, negative prompts** not available as API parameters.
- **YouTube videos** as media source not supported.
- Language: English fully supported; other languages may work but are not officially evaluated.

---

## 14. Sources

- Google AI for Developers — *Generate and edit videos with Gemini Omni Flash* (ai.google.dev/gemini-api/docs/omni, accessed June 30, 2026).
- Google AI for Developers — *Interactions API* documentation.
- Google AI for Developers — *Files API* documentation.

---

*Last updated: 2026-06-30 — v1.0. Refresh when Google ships breaking API changes or updates the omni docs. This is a preview model — expect rapid iteration.*
