# Seedance 2.0 — Agent Prompting Guide

**Audience:** Claude-based and Gemini-based agents that call Seedance 2.0 to generate video.
**Purpose:** A single source of truth for writing, refining, and orchestrating Seedance 2.0 prompts at production quality.
**Owner:** Maintain this document on every quarterly review or whenever ByteDance ships a Seedance minor version.
**Last refreshed:** 2026-05-23.
**License:** CC BY 4.0 — see repo root LICENSE.md
**Sources of truth:** ByteDance Seed official launch blog, Volcengine documentation, Replicate model card, and field reports from Apiyi, HeyMarmot, ZenCreator, and Cutout.pro (cited inline where relevant).

---

## How to read this guide

This file is written to be parsed by agents. Sections labelled `## AGENT_RULE` are non-negotiable behaviour. Sections labelled `## REFERENCE` are lookup material the agent should retrieve into context only when relevant. Section IDs in square brackets — for example `[S2]` — are stable anchors that other prompts can reference.

If you are a Claude or Gemini agent: load this file at the start of any session where you will call the Seedance 2.0 API. Re-load on every new generation task; do not assume prior context survives across runs.

---

## Table of contents

- Section 0 — Mental model `[S0]`
- Section 1 — What Seedance 2.0 actually is `[S1]`
- Section 2 — The 6-Step Formula `[S2]`
- Section 3 — Camera language `[S3]`
- Section 4 — Lighting and style `[S4]`
- Section 5 — Negative prompts and pitfalls `[S5]`
- Section 6 — The three generation modes `[S6]`
- Section 7 — Multimodal `@` reference system `[S7]`
- Section 8 — Audio and dialogue `[S8]`
- Section 9 — Multi-shot and editing `[S9]`
- Section 10 — Bridging from images (GPT Image 2, Nano Banana Pro, Nano Banana 2) `[S10]`
- Section 11 — LLM-driven prompt refinement (Claude + Gemini) `[S11]`
- Section 12 — Agent operating procedure `[S12]`
- Section 13 — Templates library `[S13]`
- Section 14 — Failure mode field guide `[S14]`
- Appendix A — API quick reference `[A]`
- Appendix B — Quality scoring rubric `[B]`
- Appendix C — Glossary `[C]`

---

## `[S0]` Mental model: write like a director, not an engineer

The single highest-leverage shift in Seedance 2.0 prompting is treating the prompt as a **production instruction to a film crew**, not a keyword stack to a generator. ByteDance's own guidance repeatedly frames this distinction: "Describe the scene, the action, and the mood you want, rather than just stacking technical parameters."

Concretely, this means:

- A prompt has a single subject and a single primary camera move. Multiple competing instructions confuse the model and produce jitter.
- Camera movement and subject movement are described as separate clauses, never blended.
- Lighting is described as a clause of its own — it is the single most quality-moving element in a Seedance prompt.
- Adjectives like "epic," "amazing," "beautiful" carry no signal. Replace each one with a specific visual fact.
- Length matters: 60-100 words is the official recommended range. Shorter than ~35 words loses critical detail; longer than ~120 words begins introducing internal contradictions the model has to resolve, usually badly.

If the prompt would not make sense read aloud to a human cinematographer, it will not generate well.

---

## `[S1]` What Seedance 2.0 actually is

### Origin and architecture

Seedance 2.0 is ByteDance's flagship multimodal video generation model, released on **February 10, 2026** by the Seed research team (the same group behind TikTok's recommendation stack). It is the successor to Seedance 1.0 (silent 5-second clips) and Seedance 1.5 Pro (which introduced basic audio).

Architecturally, Seedance 2.0 is a **Dual-Branch Diffusion Transformer** that generates video and audio jointly through a unified multimodal architecture. This matters operationally: dialogue, lip-sync, sound effects, ambient, and music are not bolted on in post — they are emitted in the same forward pass as the pixels. The practical consequence is dramatically better audio-visual sync than any model that handles audio as a separate stage.

### Input and output specs

| Property | Spec |
|---|---|
| Modalities accepted | text, image, video, audio |
| Max images per call | 9 |
| Max video references per call | 3 (each 2-15s) |
| Max audio references per call | 3 (each ≤15s, ≤15MB) |
| Total reference files per call | up to 12 |
| Output duration | 4-15 seconds |
| Output resolution | up to 2K (1080p widely, 2K on official endpoints) |
| Output frame rate | 24 fps |
| Native audio | yes (dual-channel stereo: dialogue, SFX, ambient, music) |
| Aspect ratios | 16:9, 9:16, 4:3, 3:4, 1:1, 21:9, plus `adaptive` |
| Languages supported (dialogue) | English, Chinese, Japanese, Korean, Spanish, French, German, Portuguese |
| Watermark | none |

### Resolution table (Replicate-published dimensions)

| Resolution | 16:9 | 4:3 | 1:1 | 3:4 | 9:16 | 21:9 |
|---|---|---|---|---|---|---|
| 480p | 864×496 | 752×560 | 640×640 | 560×752 | 496×864 | 992×432 |
| 720p | 1280×720 | 1112×834 | 960×960 | 834×1112 | 720×1280 | 1470×630 |

### Access and pricing

Seedance 2.0 is reachable through:

- **Volcengine** (ByteDance, China, primary)
- **BytePlus** (ByteDance international, USD billing)
- **Replicate** (`bytedance/seedance-2.0`, OpenAI-compatible patterns, fastest path for prototyping)
- **fal.ai**, **PiAPI**, and other third-party aggregators
- **Dreamina** (consumer interface, formerly Jimeng)

Official token-based pricing (Volcengine, March 2026 publication):

- Text/image/audio input → video output: **46 CNY per million tokens** (~$6.40)
- Video input → video output (editing/extension): **28 CNY per million tokens** (~$3.90)

A 15-second clip consumes roughly 308,880 tokens, which works out to about **$0.14/sec** on the official rate. Editing/extension mode is ~39% cheaper because the model reprocesses existing frames rather than generating from scratch. Through third-party providers and Fast tier variants, 5-second 720p clips can drop to roughly **$0.05/clip** — but reliability and queue times vary.

### Model variants

- **Seedance 2.0 (Standard / Pro)** — full 2K, all features, full duration range
- **Seedance 2.0 Fast** — same Dual-Branch architecture, reduced resolution ceiling and optimized inference; use for iteration and ideation, not final delivery
- **Seedance 2.1** — referenced in trade press but not yet broadly available as of this writing

### `## AGENT_RULE` — version-pinning

Always log the exact model string used (e.g. `bytedance/seedance-2.0` on Replicate, or `Doubao-Seedance-2.0` on Volcengine) alongside the generated asset. Seedance is iterating quickly; reproducibility depends on this metadata.

---

## `[S2]` The 6-Step Formula

This is the canonical structure for any text-to-video Seedance 2.0 prompt. Use it by default. Deviate only when you have a documented reason.

```
[Subject] + [Action] + [Environment] + [Camera] + [Style] + [Constraints]
```

| Step | What it does | Specificity test |
|---|---|---|
| **1. Subject** | Who or what the model must keep important | "A young woman in a white dress" — not "a person" |
| **2. Action** | What is happening, with verbs that quantify intensity | "slowly turns around, breeze blowing the skirt" — not "moves" |
| **3. Environment** | Where, including lighting/atmosphere cues | "seaside at dusk, golden glow" — not "outdoors" |
| **4. Camera** | One primary camera instruction, plus pacing | "slow push-in" — not "dynamic cinematic shots" |
| **5. Style** | Visual reference language and aesthetic | "35mm film tone, warm grade" — not "cinematic" alone |
| **6. Constraints** | Negative directives and quality guardrails | "avoid jitter and bent limbs" |

### Reference example

**Good:**

> A skateboarder lands a clean trick in an empty dawn parking lot, low tracking shot then subtle rise, modern cinematic contrast with soft natural backlight, 6 seconds, 16:9. Avoid jitter and bent limbs.

**Bad:**

> cool skateboard video, cinematic, fast, amazing tricks, lots of movement, epic style

The bad example fails because every word is an adjective with no commitment to a visual fact: no specific subject, no specific action, no camera, no environment.

### `## AGENT_RULE` — formula completeness

Before submitting any text-to-video prompt, an agent must verify all six steps are present and individually specific. If any slot would only contain adjectives or generic phrases, the agent rewrites that slot before submission. Missing a step is a defect; pad it from the reference image or shot brief.

---

## `[S3]` Camera language

Camera direction is the second highest-leverage improvement after lighting. Seedance 2.0 has been trained on professional cinematography vocabulary; use the terms it expects.

### The eight officially-supported camera movements

| Type | English term | Effect | Best use |
|---|---|---|---|
| Push-in | `push-in` / `dolly in` | camera moves toward subject | emotional emphasis, hero shots |
| Pull-out | `pull-out` / `dolly out` | camera moves away to reveal | establishing shots, reveals |
| Pan | `lateral motion` / `pan` | horizontal sweep | scanning scenes, follow subjects |
| Tracking | `tracking shot` / `follow` | camera follows subject | walking, action |
| Orbit | `orbit` / `arc` | rotation around subject | product turntables, portraits |
| Aerial | `aerial` / `drone shot` | high-altitude / bird's-eye | landscape, scale |
| Handheld | `handheld` | natural micro-shake | documentary, intimacy |
| Fixed | `fixed` / `locked-off` | static frame | letting subject action carry |

### Three non-negotiable camera rules

**Rule 1 — One primary camera instruction.** Pile-ups produce jitter.

- ✅ `camera slow push-in`
- ❌ `camera push-in, then pan left, zoom out, orbit around`
- ✅ for compound: `low tracking shot then subtle rise` (one primary + one secondary)

**Rule 2 — Pacing language beats photography parameters.** The model parses rhythm, not f-stops.

- ✅ `slow, smooth, stable, gradual, gentle`
- ❌ `24fps, f/2.8, ISO 800, focal length 85mm`

You can name a lens by feel (`85mm portrait feel`, `wide angle`) but don't try to pass exposure triangle data.

**Rule 3 — Separate camera motion from subject motion.** This is the single most common cause of jittery output.

- ✅ `The dancer spins slowly. Camera holds fixed framing.`
- ❌ `spinning camera around a dancing person`

### Speed keyword ladder

| Tier | Keywords | Effect |
|---|---|---|
| Imperceptible | `barely moves`, `subtle drift` | almost static |
| Slow | `slow`, `gentle`, `gradual` | safe default for quality |
| Medium | `smooth`, `controlled` | natural pacing |
| Fast | `dynamic`, `swift` | **high jitter risk** |

### `## AGENT_RULE` — the "fast" budget

The word `fast` (and synonyms like `rapid`, `quick`) is the most reliable predictor of degraded output in Seedance 2.0. Agents may use it on **at most one element per prompt** — either fast camera, fast subject motion, OR a busy scene. Never combine two. If a brief requires all three (e.g. a chase sequence), default to fast subject action with a stable camera and let pacing come from the cut.

---

## `[S4]` Lighting and style

If an agent can only change one thing about a weak prompt, change the lighting clause. The qualitative gap between `a person walking` and `a person walking in soft golden hour lighting with warm backlight` is larger than any other single substitution.

### Lighting recipes that work

| Keyword | When to reach for it | Example phrasing |
|---|---|---|
| `golden hour` | warm, flattering portraits, hero shots | `soft golden hour lighting from the right` |
| `rim light` | edge-defined silhouettes against dark BG | `dramatic rim light against dark background` |
| `natural light` | documentary, lifestyle, product realism | `soft natural window light, north-facing` |
| `neon` | nightscapes, cyberpunk, urban | `neon-lit rainy street, cyan and magenta` |
| `backlit` | emotional, dramatic, silhouette work | `backlit silhouette at sunset, lens flare` |
| `overcast` | even, flat, editorial product | `even overcast diffused light, no harsh shadows` |
| `practical lights` | interiors with motivated sources | `warm practical lamps, cool TV glow from camera left` |
| `volumetric` | atmosphere, depth, scale | `volumetric god-rays through dust` |

### Style keyword catalog

| Category | Recommended keywords |
|---|---|
| Cinematic | `cinematic`, `film tone`, `35mm`, `anamorphic` |
| Quality flag | `4K`, `high detail`, `sharp` |
| Film texture | `film grain`, `analog`, `vintage`, `Kodachrome` |
| Tone bias | `warm tone`, `cool palette`, `desaturated`, `teal-and-orange` |
| Mood | `moody`, `dreamy`, `ethereal`, `oppressive`, `serene` |
| Realism level | `realistic`, `natural`, `documentary`, `hyperreal` |

### `## AGENT_RULE` — never use "cinematic" alone

`cinematic` on its own is too vague to be a directive. Always qualify it: `cinematic film tone, 35mm, warm grade` or `cinematic anamorphic, teal-and-orange`. Treat `cinematic` like a folder name, not a file.

---

## `[S5]` Negative prompts and pitfalls

Seedance 2.0 explicitly accepts negative directives at the end of a prompt. Treat them as part of the standard structure, not an afterthought.

### The standard negative tail

For any character-driven shot, the default negative tail is:

> `avoid jitter, bent limbs, identity drift, temporal flicker`

For product shots, add `avoid label warp, text artifacts`. For multi-subject scenes, add `avoid chaotic composition, overlapping subjects`.

### Keywords that often degrade quality

| Risky term | Why | Replace with |
|---|---|---|
| `fast` (unqualified) | spawns jitter cascades | `swift footwork while camera stays slow` |
| `cinematic` (alone) | no concrete direction | `cinematic film tone, 35mm, warm grade` |
| `epic` | no defined visual signature | name the effect: `large scale, low-angle hero shot` |
| `amazing` / `beautiful` | pure adjective noise | name the lighting and composition |
| `lots of movement` | provokes jitter | `one specific motion: a slow turn toward camera` |
| `multiple cuts` | cuts already handled by `lens switch` | use `lens switch` once per intended cut |

### The five common failure modes (memorize)

1. **Multiple conflicting camera moves** → jitter. Use one primary movement.
2. **`fast` everywhere** → chaos. Apply `fast` to at most one element.
3. **Camera and subject motion mixed in the same clause** → uncontrollable shake. Separate them.
4. **No lighting clause** → flat, dead-looking output. Always include lighting.
5. **No negative tail** → bent limbs and identity drift on character shots. Always include the negative tail.

---

## `[S6]` The three generation modes

### Text-to-video (T2V)

Use the full 6-Step Formula. Describe everything: subject, action, environment, camera, style, constraints. Aim for 60-100 words.

### Image-to-video (I2V)

The image carries the subject and composition. Do not re-describe what is in the image. Focus the prompt entirely on motion, camera, and atmospheric change. Include the phrase `preserve composition and colors` (or equivalent) to lock visual fidelity.

```
Animate the provided image, preserve composition and colors. Add gentle
wind motion to the leaves; the woman blinks once and turns her head three
degrees toward the camera. Camera slowly pushes in. Keep consistent lighting.
6 seconds. Avoid identity drift.
```

**I2V resolution floor.** Official documentation requires a minimum of **768 pixels on the shortest side** for the input image. Below this, output blurs noticeably. Production rule: feed 1024px minimum for product work; 1536px+ for portraits.

**I2V composition affects motion freedom.** A tight headshot (shoulders-up) preserves identity better but limits motion range to subtle head tilts, blinks, micro-expressions. A full-body shot unlocks walking, turning, gesture — at the cost of more identity drift risk. Choose the framing during the *image generation* stage with the video's needs in mind.

### Video-to-video (V2V)

A reference video plus a prompt describes a **transformation**. Always state what changes and what stays.

```
Transform source clip to anime watercolor style. Preserve core motion and
timing. Adjust color palette to pastel. Keep identity consistent.
Avoid identity drift.
```

### Comparison table

| Element | T2V | I2V | V2V |
|---|---|---|---|
| Subject description | required, detailed | omit (already in image) | omit (already in video) |
| Motion description | full | focus here | describe changes |
| "Preserve composition and colors" | n/a | **required** | partial — what stays |
| Camera | flexible | must respect image framing | usually inherited |
| Token cost | 46 CNY/Mtok tier | 46 CNY/Mtok tier | **28 CNY/Mtok tier (~39% cheaper)** |

### `## AGENT_RULE` — mode-token-cost awareness

When iterating on a V2V or video-extension task, prefer V2V over re-generating from T2V. The pricing tier alone gives a ~39% per-call discount, and the model produces more consistent results because it's reprocessing rather than rebuilding.

---

## `[S7]` Multimodal `@` reference system

This is Seedance 2.0's signature feature and the lever that separates it from every competing model. Up to 12 reference files combine: 9 images, 3 videos, 3 audio. Each gets an auto-assigned label — `@Image1`, `@Image2`, `@Video1`, `@Audio1` — and your prompt references those labels directly.

### The reference-role principle

Every reference must have a stated role. Never make the model guess.

**Bad:**

> Use the images and make a cool ad.

**Better:**

> Use @Image1 as the main character reference. Use @Image2 as the outfit reference. Use @Image3 as the logo reference. Create a premium beauty ad in a bright studio. The logo stays visible in the lower-right corner near the end.

### What each reference type is good for

| Reference type | Best at | Don't ask it for |
|---|---|---|
| `@Image` (subject) | locking face, character, product identity | motion |
| `@Image` (scene) | locking environment, set, background | character |
| `@Image` (storyboard) | shot order, framing, pacing signal | fine appearance details |
| `@Video` (action) | inheriting motion patterns, choreography | look/appearance |
| `@Video` (camera) | borrowing a camera move that already works | subject appearance |
| `@Video` (effects) | particles, glow trails, stylized treatments | composition |
| `@Audio` (rhythm) | beat-matched cutting and motion | dialogue identity |
| `@Audio` (voice) | tone and pacing reference for narration | exact word match without quotes |

The two-axis principle: **images lock identity; videos lock behaviour.**

### Reference assignment patterns

**Single subject, multiple angles (product turntable):**

> Reference @Image1, @Image2, @Image3 for the same product from front, side, rear. Generate a clean studio showcase. Keep the product appearance consistent while the camera slowly orbits 180 degrees. 8 seconds. Avoid label warp.

**Multi-image scene assembly:**

> Set the scene inside the restaurant from @Image4. Use the woman from @Image1 wearing the outfit from @Image2. The man from @Image3 enters and approaches the counter. Keep the logo from @Image5 visible in the lower-right corner throughout. 12 seconds. Avoid identity drift.

**Storyboard-as-input:**

> Follow the panel order in @Image1. Recreate the framing in sequence, then continue the scene naturally into the action implied by the last panel. 10 seconds.

**Borrow camera move from reference video:**

> Reference @Video1 for the camera movement only. Create a first-person concept video flying toward the central tower from @Image1. Match the speed and dive trajectory of the reference. Blue-toned sci-fi atmosphere. 8 seconds.

**Borrow action from reference video:**

> Reference @Video1 for the running motion. The horse in @Image1 sprints across an open grassland with the same stride rhythm and energy as the reference. 6 seconds. Avoid bent limbs.

**Borrow effect from reference video:**

> Reference @Video1 for the golden-particle effect only. The musician in @Image1 plays a flute while matching golden particles spiral around the body in the same trajectory and density as the reference. 7 seconds.

### `## AGENT_RULE` — explicit role assignment

Every `@` reference must be tagged with its role in the prompt. An untagged reference is a defect; the agent must rewrite the prompt to either (a) assign the role or (b) drop the reference before submission.

### `## AGENT_RULE` — identity vs motion separation

Never use the same reference for both identity and motion. If the brief says "make this character do this dance," use one image as `@Image1` for identity and one video as `@Video1` for motion. Mixing roles on a single reference produces unreliable output.

---

## `[S8]` Audio and dialogue

Seedance 2.0 generates dual-channel stereo audio jointly with video, including dialogue with phoneme-level lip-sync in 8+ languages. Treat audio as part of the prompt from the start, not as a post-production worry.

### How to specify dialogue

Put spoken words in **double quotes**. The model uses quoted strings as both the spoken text and the lip-sync target.

```
The man stops walking, turns to face the camera, and says:
"Remember this moment." His voice is calm and low. Soft jazz plays in the
background. 6 seconds.
```

For multiple lines, quote each one and indicate the speaker:

```
The girl looks at the boy and says: "We can do this."
The boy replies: "Are you sure?"
She smiles brightly: "Absolutely."
```

### Audio reference for music sync

```
Reference @Audio1 as the background music. Sync the dancer's footwork in
@Image1 to the beat. Bright studio lighting, fixed wide shot. 10 seconds.
```

### Sound design vocabulary that lands

Seedance 2.0's audio module recognizes specific foley language. The official launch examples include phrases like `the light scratching of frosted glass`, `rubbing of plush fabric`, `gentle tapping on an acrylic board`, `light pinching of bubble wrap`. The more specifically you describe the *texture* of a sound, the better the output.

| Audio element | Effective phrasing |
|---|---|
| Ambient | `gentle wind through pines`, `distant city hum`, `ocean swell` |
| Foley | name the material AND the action: `crunching gravel underfoot` |
| Music | name the genre, mood, and presence: `subtle synth pad in the background` |
| Dialogue | quoted text + tone descriptor: `says calmly: "—"`, `whispers: "—"` |
| ASMR / texture | name the trigger: `light scratching of frosted glass` |

### The silence option

If audio is undesired (e.g. for an asset that will be scored later), explicitly request silence: `no dialogue, no music, ambient room tone only` or `silent output, no audio track`.

### `## AGENT_RULE` — quote your dialogue

Any spoken line in a Seedance 2.0 prompt must be in double quotes. Unquoted lines may be interpreted as scene description rather than dialogue, and lip-sync will not engage.

---

## `[S9]` Multi-shot and editing

### Multi-shot within one generation

Use the keyword **`lens switch`** to signal a cut. The model maintains character/environment continuity across cuts within a single generation.

```
A detective in a trench coat approaches an abandoned warehouse. Close-up
of his hand pushing the door open. Lens switch to wide shot of the dark
interior with a single bulb hanging from the ceiling. Lens switch to
over-shoulder shot as he sees a figure in the shadows. Tense orchestral
score builds, creaking door, echoing footsteps. 12 seconds.
```

Limits:

- More than ~3-4 `lens switch` instructions in a single 15-second clip degrades coherence
- Each shot should be a coherent action, not just a different angle on the same moment
- Maintain ONE consistent lighting plan across cuts unless you explicitly call for a transition

### Editing existing video (V2V edit)

Think like an editor with a surgical instruction: state what changes, then state what must remain unchanged.

**Add:**

> Add a steaming coffee cup to the table in @Video1. Keep all original motion and lighting.

**Remove:**

> Remove the extra tools and scattered parts from the tabletop in @Video1. Keep the hands, timing, and camera movement unchanged.

**Replace:**

> Replace the perfume bottle in @Video1 with the cream jar from @Image1. Preserve the original hand motion and camera move. Match shadow direction.

### Extending a clip

You do not need to regenerate the original. Use extend instead — it's cheaper (V2V tier) and more coherent.

```
Generate what happens after @Video1: two late-arriving friends run into
the frame and join the conversation. Match lighting and camera style.
```

```
Extend @Video1 backward. Start with an over-the-shoulder shot of the man
in white speaking before the current scene begins. Keep all character
and outfit details consistent.
```

The model handles the overlap region automatically; it does not simply repeat the source clip verbatim.

### Bridging multiple clips

Supported pattern: up to **3 input videos, total input length ≤15 seconds**. Useful for transitions, short-form assembly.

```
@Video1: a leaf falls to the ground.
Bridge: a burst of golden particles rises on impact; a gust of wind sweeps
across the frame.
Transition into @Video2.
```

### `## AGENT_RULE` — what stays, what changes

For every editing or extension prompt, the agent must explicitly state both: what must remain unchanged (motion, timing, lighting, identity, etc.) and what should change. Omitting the "what stays" clause produces inconsistent edits.

---

## `[S10]` Bridging from images: GPT Image 2, Nano Banana Pro, Nano Banana 2

This is where art direction lives. The image generation step is where the agent makes the *creative* choices that the video step then animates. A precise, well-art-directed reference image produces dramatically better Seedance 2.0 output than even the most polished prompt against a stock photo.

### Why this matters

Seedance 2.0's I2V mode preserves the look of the input frame. That means:

- Your **composition** comes from the image, not the prompt
- Your **color palette** comes from the image, not the prompt
- Your **character identity** comes from the image, not the prompt
- Your **lighting** is set in the image and re-reinforced in the prompt

Get the image right and the video prompt becomes short and focused on motion only.

### Choosing the image generator

| Use case | Model | Why |
|---|---|---|
| Embedded text, posters, brand reveals, infographics | **GPT Image 2** | Near-perfect text rendering, 4K output, intelligent multi-size variants |
| Multi-character scenes with up to 5 consistent faces, 14 objects, real-world grounding | **Nano Banana Pro** (Gemini 3 Pro Image) | Thinking-mode composition, up to 14 reference images in one call |
| Fast iteration, batch storyboard frames, real-time web grounding at lower cost | **Nano Banana 2** (Gemini 3.1 Flash Image) | Pro-level quality at Flash speed/price; same 5-person/14-object consistency |
| Stylized non-photoreal looks at high speed | Nano Banana 2 | Best speed/cost for variant exploration |

### The art-direction workflow

**Step 1 — Storyboard the shot before generating anything.**
Write a one-paragraph shot brief that includes: subject, action, environment, framing, lighting, mood, color palette, aspect ratio. This is the human/agent decision; both downstream stages (image and video) consume it.

**Step 2 — Generate the reference image with a model matched to the brief.**
Use Nano Banana Pro's 7-element structure: `Subject + Action + Location + Composition + Lighting + Style + Technical Specs`. For text in frame, switch to GPT Image 2 and quote the exact text. For character continuity across multiple shots, use Nano Banana Pro's multi-image conversational refinement, building scene-by-scene in one thread so the 5-person consistency engine carries identity forward.

**Step 3 — Match the image aspect ratio to the intended video.**
Generating at 1:1 then planning a 16:9 video forces the model to invent composition outside the original frame. Pick the video aspect ratio first; generate the reference at that ratio.

**Step 4 — Constrain the image to a "video-friendly" pose.**
Reference frames that work best as I2V inputs share three traits:

- The subject is **not in the middle of an extreme action pose** (an arm mid-swing is harder to continue than an arm at rest)
- There is **visible negative space** for the subject to move into
- The **lighting is motivated** — there is an implied light source the video can preserve naturally

Tell the image model these things explicitly in the prompt: `subject in a stable resting pose with room to move toward camera left`, `motivated key light from the right at 30 degrees`, `composition with the subject placed in the right third`.

**Step 5 — Resolution floor.**
Export the image at minimum 1024px on the short side (Seedance's official minimum is 768px, but quality scales noticeably up to ~1536px). For product or portrait work where identity must be tight, use 2K outputs from Nano Banana Pro.

**Step 6 — Write the Seedance prompt around the image.**
Now the Seedance prompt is short:

> Animate the provided image, preserve composition and colors. [One specific motion]. [One camera move]. [Lighting consistency]. 6 seconds. Avoid identity drift.

### Multi-shot continuity workflow

For a sequence of shots that need to look like the same world:

1. Generate **all** reference frames in a single Nano Banana Pro conversation, building each shot off the prior turn. The model's thought-signature continuity keeps characters, props, and lighting coherent across the set.
2. Export each frame at matching aspect ratio and resolution.
3. Generate each Seedance 2.0 clip independently, using the matching frame as I2V input.
4. For shots where character motion needs to match a previously-generated clip, use the prior Seedance clip as a `@Video` reference for action, and the new Nano Banana frame as I2V — this combines identity (image) with motion continuity (video).

### GPT Image 2 → Seedance 2.0 specific notes

GPT Image 2's strength is text-in-frame. When the video needs a brand title card, packaging shot, or end-frame logo reveal:

- Generate the still in GPT Image 2 with the exact text in quotation marks
- Match the video's aspect ratio at generation time
- Feed as I2V; ask Seedance to animate **around** the text without warping it, e.g. `keep all text in the frame sharp and unchanged, animate only the background elements`
- For end-card reveals: use Seedance's slogan-appearance pattern (see [S13] Template Library)

### Nano Banana 2 → Seedance 2.0 batch workflow

For rapid storyboard development:

1. Generate 8-shot character-consistent storyboard panel in one Nano Banana 2 call (its sweet spot)
2. Crop each panel to its intended aspect ratio
3. Submit each as I2V to Seedance 2.0 Fast for first-pass timing/motion check
4. Promote winning shots to Seedance 2.0 Standard for final delivery

### `## AGENT_RULE` — image-first art direction

For any I2V task where art direction matters (i.e. nearly all of them), the agent must:

1. Confirm the aspect ratio with the user/brief before generating the image
2. Generate the reference image using the model matched to the requirement (text → GPT Image 2; multi-character → Nano Banana Pro; fast batch → Nano Banana 2)
3. Verify image resolution ≥ 1024px on short side before feeding to Seedance
4. Write the Seedance prompt to preserve, not re-describe, the image

---

## `[S11]` LLM-driven prompt refinement (Claude + Gemini)

This section is the heart of the agent workflow. Seedance 2.0 prompts can be drafted, critiqued, and refined entirely by Claude and Gemini before a single video credit is spent. The combination of the two models produces measurably better prompts than either alone.

### The roles

| Model | Best at | Default role |
|---|---|---|
| Claude (Opus or Sonnet) | structured creative writing, holding long-form constraints, finding ambiguity, "directorial" voice | **Drafter & Director** |
| Gemini (3 Pro / 3.1) | multimodal critique (can actually see the reference image), grounded fact-checking, adversarial gap-finding | **Critic & Verifier** |

This pairing is deliberate. Claude generates a director-style prompt that holds all the right structural constraints. Gemini then *looks at the reference image* and stress-tests the prompt against what it actually sees — catching motion conflicts, off-image references, identity-drift risks, and lighting mismatches that a text-only model would miss.

### The four-stage refinement loop

**Stage 1 — Brief intake (agent only).** The agent parses the human brief into a structured shot brief:
- Subject + identifying details
- Action (single sentence, verb-first)
- Environment + lighting
- Aspect ratio + duration
- References (with assigned roles)
- Constraints (brand, IP, audio)

**Stage 2 — Draft (Claude).** Claude takes the structured brief and produces a Seedance 2.0 prompt following the 6-Step Formula. Use this system instruction:

```
You are a Seedance 2.0 video prompt director. Your job is to convert
the shot brief below into a single Seedance 2.0 prompt that follows
the 6-Step Formula: Subject → Action → Environment → Camera → Style →
Constraints. The prompt must be 60-100 words. Use one primary camera
movement. Separate camera motion from subject motion. Include a
lighting clause. Include the negative tail "avoid jitter, bent limbs,
identity drift, temporal flicker". If references are listed in the
brief, tag them as @Image1/@Video1/etc with explicit roles.

Output only the prompt. No preamble.

Shot brief:
[structured brief here]
```

**Stage 3 — Critique (Gemini).** Pass the draft prompt AND the reference image (if any) to Gemini with this instruction:

```
You are a Seedance 2.0 prompt critic. The image attached is the reference
that will be animated. Read the prompt and answer five questions:

1. Does the prompt describe motion that is physically plausible from the
   pose in the image? Name any conflict.
2. Does the prompt reference any element that is NOT visible in the image?
   List each one.
3. Is the lighting described in the prompt consistent with the lighting
   visible in the image? Name any mismatch.
4. Are there multiple competing camera movements? List them.
5. Does the prompt mix camera motion and subject motion in the same
   clause? Quote the clause if so.

For each "yes / found a problem," give one specific revision in one sentence.
Output as a numbered list. Be terse.
```

**Stage 4 — Revision (Claude).** Pass Gemini's critique and the original draft back to Claude:

```
You are revising a Seedance 2.0 prompt based on a critic's feedback.
Apply every revision that is grounded in the image. Reject any revision
that breaks the 6-Step Formula. Output only the revised prompt, no commentary.

Original prompt:
[draft]

Critic feedback:
[Gemini output]
```

**Stage 5 — Sanity check (Claude).** A final terse pass with Claude:

```
Final check on this Seedance 2.0 prompt:

- Word count 60-100? Y/N
- Has all six steps (Subject, Action, Environment, Camera, Style, Constraints)? Y/N
- Single primary camera move? Y/N
- Camera and subject motion in separate clauses? Y/N
- Lighting clause present? Y/N
- Negative tail present? Y/N
- All @ references have assigned roles? Y/N

If any answer is N, fix it and reprint. If all Y, print the prompt as-is.
```

### Why the loop works

The decisive insight is that **a text-only LLM cannot critique a prompt that animates an image — it has no way to know whether the prompt and image agree.** Gemini's native multimodality solves this: it sees the image, reads the prompt, and tells you where they disagree. Claude is the better drafter and the better revisor because it holds long-form structural constraints (the 6-Step Formula) more reliably across iterations.

Running the loop adds 2-4 model calls to the workflow. At Claude Sonnet / Gemini Flash pricing, this costs cents. A failed Seedance 2.0 generation at 1080p costs roughly **$0.84-1.68 per clip**. The loop pays for itself the first time it catches a failure mode.

### Adversarial pre-mortem (optional, for high-stakes shots)

Before submitting a prompt for a high-stakes generation (e.g. a collectible reveal, a licensed hero shot, a marketing render for an IP partner), run a written pre-mortem:

```
[Claude, system prompt]
It is 4 hours from now. We generated this Seedance 2.0 prompt and the
output failed IP review. What are the top 5 most likely reasons the
output failed? For each, propose one specific change to the prompt
that would have prevented it.

Prompt:
[final prompt]
Reference image attached.
Brand context: [IP, style guide, prior failed reviews if any]
```

Apply any defensible mitigations. Then run.

### `## AGENT_RULE` — never skip Gemini critique when an image is present

If the generation is I2V or uses any `@Image` reference, the Gemini critique step is **mandatory**. The cost is trivial; the catch rate on image-prompt mismatches is high.

### `## AGENT_RULE` — preserve the brief

The agent retains the original structured shot brief (Stage 1) as the source of truth. If the prompt drifts during refinement, the agent re-anchors against the brief, not against the last draft.

---

## `[S12]` Agent operating procedure

This is the runbook. Agents should follow this sequence for any Seedance 2.0 generation task.

### Step-by-step

```
1. INTAKE
   - Parse user/brief into structured shot brief (Section S11 Stage 1)
   - Confirm: aspect ratio, duration, IP scope, audio requirements
   - If any ambiguity, ask exactly one targeted clarifying question

2. ART DIRECTION (if I2V or reference-image-driven)
   - Choose image model (S10): text → GPT Image 2, multi-character → Nano Banana Pro,
     fast batch → Nano Banana 2
   - Generate reference image(s) at matching aspect ratio, ≥1024px short side
   - For multi-shot: generate all frames in one Nano Banana Pro conversation
     for consistency

3. DRAFT
   - Claude drafts Seedance 2.0 prompt using 6-Step Formula (S2)
   - Aim for 60-100 words

4. CRITIQUE (mandatory if image present)
   - Gemini reads prompt + image, returns five-point critique (S11 Stage 3)

5. REVISE
   - Claude applies grounded revisions (S11 Stage 4)

6. SANITY CHECK
   - Claude runs final Y/N checklist (S11 Stage 5)

7. PRE-MORTEM (high-stakes shots only)
   - Claude lists top 5 failure modes; agent mitigates

8. GENERATE
   - Submit to Seedance 2.0
   - Log: model version, prompt text, reference asset paths/hashes,
     seed/parameters, output URL
   - Initial pass: Seedance 2.0 Fast for iteration
   - Final pass: Seedance 2.0 Standard

9. EVALUATE
   - Score against quality rubric (Appendix B)
   - If score < threshold, single-variable iteration (change one element,
     not three)

10. DELIVER + RETAIN
    - Output final asset
    - Retain prompt + reference + score in prompt library for future reuse
```

### `## AGENT_RULE` — one-variable iteration

When a Seedance 2.0 output is close but not right, change only **one** element of the prompt for the next attempt: camera move OR subject action OR lighting OR style — never two at once. This is the only way to learn which change actually moved the result.

### `## AGENT_RULE` — log every generation

Every Seedance 2.0 call must be logged with: model version string, exact prompt text, reference asset paths (or content hashes), seed/parameters used, output URL, and a quality score. Without this log, no learning compounds, and continuous improvement is impossible. Your prompt library is built from these logs.

### `## AGENT_RULE` — IP review pass before delivery

Any Seedance 2.0 output intended for a licensed IP-partner deliverable must be reviewed by a human before submission to the partner. AI-generated content has passed IP review before, but the pass rate depends on human review catching subtleties the model cannot. Never short-circuit this step for speed.

---

## `[S13]` Templates library

These are field-tested starting points. Copy, adapt, generate. Every template ends with the negative tail by default.

### Template — cinematic portrait (T2V)

```
A woman with dark curly hair wearing a red silk dress stands on a rooftop
at golden hour. She turns slowly toward the camera with a slight smile.
Medium shot, slow push-in. Shallow depth of field, 85mm portrait feel.
Warm amber light catches her hair from the right; cool sky behind.
Cinematic film tone, 35mm. Ambient city sounds below, gentle wind.
8 seconds, 16:9. Avoid jitter, bent limbs, identity drift, temporal flicker.
```

### Template — action sequence (T2V)

```
A parkour runner in a black hoodie sprints across a rain-soaked rooftop
at night. He leaps a gap, lands in a roll, keeps running. Handheld camera
follows from behind, lens switch to low-angle shot at the jump. Neon
reflections on wet surfaces, cyan and magenta. Footsteps on concrete, rain,
heavy breathing, no music. 10 seconds, 16:9. Avoid jitter, bent limbs,
identity drift, temporal flicker.
```

### Template — product showcase (T2V)

```
A matte black wireless headphone sits on a dark marble surface. Camera
slowly orbits 180 degrees around the product. Soft studio lighting, single
key light from the left creates dramatic shadows, subtle rim light on the
right edge. Minimal ambient electronic music, no dialogue. 8 seconds, 16:9.
Avoid label warp, text artifacts, identity drift.
```

### Template — character I2V with motion (I2V)

```
Animate the provided image, preserve composition and colors. The character
walks slowly through the snowy forest at twilight, reaches out to catch a
snowflake, then looks up with quiet wonder. Slow tracking shot from the
side. Keep blue-purple ambient light and warm visible breath. Crunching
snow footsteps, distant wind through pine trees. 8 seconds.
Avoid identity drift, bent limbs, temporal flicker.
```

### Template — product I2V with text preserved (I2V, GPT Image 2 source)

```
Animate the provided image, preserve composition and colors. Keep all text
in the frame sharp and unchanged. Animate only the background: gentle
particles drift from camera left to right; soft warm light pulses subtly
across the product surface. Camera holds fixed framing. 6 seconds, 1:1.
Avoid label warp, text artifacts, jitter.
```

### Template — multi-shot narrative (T2V, lens switch)

```
A detective in a trench coat approaches an abandoned warehouse. Close-up
of his hand pushing the door open. Lens switch to wide shot of the dark
interior with a single light hanging from the ceiling. Lens switch to
over-shoulder shot as he sees a figure in the shadows. Tense orchestral
score builds, creaking door, echoing footsteps. 12 seconds, 16:9.
Avoid identity drift, chaotic composition.
```

### Template — multi-image scene assembly (R2V, multimodal)

```
Set the scene inside the restaurant from @Image4. Use the woman from
@Image1 wearing the outfit from @Image2. The man from @Image3 enters and
approaches the counter. Keep the logo from @Image5 visible in the
lower-right corner throughout. Warm practical lighting, handheld feel,
single slow push-in toward the counter. Ambient restaurant murmur,
light jazz. 12 seconds, 16:9. Avoid identity drift, bent limbs.
```

### Template — camera borrowed from reference video (R2V)

```
Reference @Video1 for the camera movement only. Create a first-person
fly-through toward the central building from @Image1. Match the speed and
dive trajectory of the reference. Blue-toned sci-fi atmosphere, subtle
synth ambience, no dialogue. 8 seconds, 21:9. Avoid jitter, identity drift.
```

### Template — action borrowed from reference video (R2V)

```
Reference @Video1 for the running motion only. The horse in @Image1
sprints across an open grassland with the same stride rhythm and energy.
Golden hour backlight, dust kicked up behind the hooves. Wind, hoofbeats,
no music. 6 seconds, 16:9. Avoid bent limbs, identity drift.
```

### Template — replace an object in an existing clip (V2V edit)

```
Replace the bottle in @Video1 with the skincare jar from @Image1.
Preserve the hand gesture, timing, lighting, and camera move. Match shadow
direction. Keep all original audio. 6 seconds. Avoid label warp.
```

### Template — extend a clip forward (V2V extend)

```
Generate what happens after @Video1. Two friends run into the frame from
camera right and join the conversation at the table. Match the existing
lighting, camera style, and ambient audio. 8 seconds.
Avoid identity drift, lighting mismatch.
```

### Template — slogan reveal end card (T2V, text in frame)

```
Hand-drawn commercial illustration style. Three friends share a bucket of
fried chicken and laugh together. Camera holds a medium-wide shot. As the
scene gradually softens out of focus over the final two seconds, the slogan
"Good Times, Frame by Frame" appears in the center in a playful handwritten
style, warm yellow lettering. Warm interior lighting, ambient laughter, soft
acoustic guitar. 8 seconds, 16:9. Avoid text artifacts, identity drift.
```

### Template — dialogue with lip-sync (T2V)

```
A man in a navy suit stops walking on a quiet city street at dusk, turns
to face the camera, and says: "Remember this moment." His voice is calm
and low. Soft jazz plays from a nearby cafe. Medium shot, fixed framing,
warm streetlamp glow with cool blue sky behind. 6 seconds, 16:9.
Avoid identity drift, jitter, audio drift.
```

### Template — ASMR / texture (T2V)

```
Immersive first-person hand ASMR video. Close-up shot, warm soft lighting.
A pair of slender hands gently triggers different objects in sequence:
the light scratching of frosted glass, the rubbing of plush fabric, the
gentle tapping on an acrylic board. Finger movements are slow and gentle.
Pure natural trigger sounds, no background music, no dialogue. The visual
atmosphere is relaxing. Fixed camera. 12 seconds, 16:9. Avoid jitter.
```

---

## `[S14]` Failure mode field guide

When a Seedance 2.0 output is bad, diagnose before re-rolling. Most failures are addressable with a single prompt change.

| Symptom | Likely cause | Single-variable fix |
|---|---|---|
| Jittery, shaky frame | Multiple camera moves OR camera+subject mixed in one clause | Reduce to one primary camera move; separate clauses |
| Bent or extra limbs | Missing negative tail, OR fast subject motion | Add `avoid bent limbs`; downshift speed keyword one tier |
| Character face drifts across the clip | I2V image too small, OR motion too dramatic | Re-export image at ≥1536px; reduce motion to subtle |
| Text in frame warps | Asking model to animate text-bearing elements | Pin text with `keep all text sharp and unchanged`; animate only background |
| Lighting changes mid-clip | No lighting clause OR conflict with image | Add explicit lighting clause; ensure it matches image |
| Action feels sluggish | Speed keywords too soft for the subject | Upgrade one tier (gentle → smooth → swift) for subject only |
| Audio doesn't match motion | No audio direction in prompt | Add explicit foley/music/dialogue language |
| Lip-sync wrong | Spoken text not in double quotes | Quote the dialogue |
| Identity drift on multi-shot | Multiple references without role assignment | Tag every `@` reference with its explicit role |
| Output looks "generic AI" | Adjective-stack with no specific facts | Replace every "epic/cinematic/amazing" with a specific visual fact |
| Background changes between cuts | No environment continuity instruction | Add `maintain same location and lighting across cuts` |
| Two characters swap features | Multi-subject consistency limit | Generate as two separate clips, edit together; or use @Image references for each |

### `## AGENT_RULE` — diagnose before re-rolling

When a generation fails, the agent's next action is **not** to retry. It is to consult this table, identify the symptom, apply the single-variable fix, and log both the diagnosis and the fix in the prompt library. Blind retries waste credits and produce no learning.

---

## `[A]` Appendix A — API quick reference

### Replicate (fastest path for prototyping)

```python
import replicate

output = replicate.run(
    "bytedance/seedance-2.0",
    input={
        "prompt": "<your 6-step formula prompt>",
        "duration": 6,                # 4-15, or -1 for adaptive
        "aspect_ratio": "16:9",       # or "adaptive"
        "resolution": "720p",         # 480p | 720p
        "image": "<url or upload>",   # for I2V; first frame
        "image_last": "<url>",        # optional last-frame anchor
        "audio_enabled": True,
        # Reference inputs for multimodal/@ tags:
        "image_refs": ["<url1>", "<url2>"],   # @Image1, @Image2, ...
        "video_refs": ["<url1>"],             # @Video1
        "audio_refs": ["<url1>"],             # @Audio1
    }
)
```

### Volcengine / BytePlus (production, China / international)

- Submit-poll-download async job pattern
- Submit returns a job ID; poll status until `succeeded`
- Typical wall time: 30-120 seconds for 720p, 2-10 minutes for 1080p+
- Token-based pricing: 46 CNY/Mtok for T2V/I2V; 28 CNY/Mtok for V2V

### Common parameter cheat sheet

| Parameter | Common values | Notes |
|---|---|---|
| `duration` | 4-15 | `-1` = adaptive (model picks) |
| `aspect_ratio` | `16:9`, `9:16`, `4:3`, `3:4`, `1:1`, `21:9`, `adaptive` | match to target platform |
| `resolution` | `480p`, `720p`, `1080p`, `2K` | Standard tier supports 2K; Fast caps lower |
| `audio_enabled` | true / false | false for silent assets to be scored later |
| `seed` | int | record for reproducibility |

### `## AGENT_RULE` — record every parameter

Every API call must log every parameter, not just the prompt. Reproducibility requires the full input dictionary.

---

## `[B]` Appendix B — Quality scoring rubric

Score every Seedance 2.0 output on these five axes, 0-2 each (10 total). Outputs scoring ≥8 ship; 6-7 iterate with single-variable change; ≤5 diagnose with the failure mode table before retrying.

| Axis | 0 | 1 | 2 |
|---|---|---|---|
| **Instruction adherence** | Output ignores major brief elements | Captures gist, misses specifics | Matches brief precisely |
| **Motion stability** | Visible jitter, warping, or bent limbs | Minor artifacts at moments | Smooth throughout |
| **Identity consistency** | Subject visibly drifts across clip | Mild drift in one or two frames | Locked across full clip |
| **Lighting coherence** | Lighting changes inexplicably | One lighting hiccup | Consistent and motivated |
| **Audio-visual sync** | Audio and visuals don't align | Mostly aligned with one slip | Perfectly synced |

Log the score with the prompt. Patterns in the log are the most valuable Kaizen signal the team has.

---

## `[C]` Appendix C — Glossary

| Term | Definition |
|---|---|
| **6-Step Formula** | Subject → Action → Environment → Camera → Style → Constraints |
| **@ reference system** | Seedance 2.0's labeling convention for multimodal inputs (`@Image1`, `@Video1`, `@Audio1`) |
|  |
| **Dual-Branch Diffusion Transformer** | Seedance 2.0's architecture, generating video and audio jointly |
| **I2V / T2V / V2V / R2V** | Image-to-video / Text-to-video / Video-to-video / Reference-to-video |
| **Lens switch** | Keyword that signals an internal cut within a single Seedance generation |
| **Negative tail** | The closing `avoid …` clause on every Seedance prompt |
| **Pre-mortem** | Written "what could go wrong" exercise before high-stakes generation |
| **Single-variable iteration** | Iteration rule: change exactly one element between attempts |
| **Standard / Fast** | Seedance 2.0 quality tiers — Fast for iteration, Standard for delivery |

---

## Closing rule of thumb

> If you would not say this prompt out loud to a director of photography
> and expect them to nod, do not submit it to Seedance 2.0.

Read every prompt aloud, in the voice of someone briefing a film crew. The ones that survive that test ship. The ones that don't, rewrite.

---

*End of guide. Update on every Seedance minor version or on a quarterly review. Maintainers: update this file and submit a PR.*
