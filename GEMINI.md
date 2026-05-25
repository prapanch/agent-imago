# Agent Imago — Skills for Gemini Agents

Use this file as context when working with Gemini-based agents. Each section is a self-contained skill.


## Track: creative-brief

---

### creative-brief

> Define the visual intent before generating anything. A creative brief is the contract between what you're asked to make and what you actually produce. No generation without a brief.

# 📝 Creative Brief

The most common reason a generated asset misses the mark: generation started before the intent was understood. A brief takes 3 minutes. Re-generating a failed asset takes 30. Write the brief first.

---

## The Shot Brief Template

Every generative media task — image or video — starts here. Fill every slot before touching a model.

```markdown
# Shot Brief

## Intent
[One sentence: what does this image/video accomplish? What is it FOR?]

## Subject
[Who or what is the primary focus? Be specific: not "a person" but "a woman
in her 30s with auburn hair wearing a navy blazer, seated at a desk."]

## Action
[What is the subject doing or how do they exist in frame?
For video: what movement happens? For images: what moment is frozen?]

## Environment
[Where is this? Time of day, weather, location, atmosphere.]

## Framing & Composition
[Camera angle, shot type, aspect ratio, rule of thirds placement,
negative space needs. Match to the target platform — don't generate 1:1 for 16:9.]

## Lighting
[Quality (soft/hard), direction (45° from camera-left), source (natural/studio),
color temperature (warm/cool), mood it creates.]

## Style
[Photographic genre, art movement, film reference, color grade. Be specific:
not "cinematic" but "cinematic film tone, 35mm, teal-and-orange grade."]

## Mood / Feeling
[What should the viewer feel when they see this? One or two words.]

## Text in Frame
[If any text must appear, list it verbatim here. Platform: [image/video model].
Text must be quoted exactly in the prompt.]

## Model
[Which model? GPT Image 2 / Nano Banana Pro / Nano Banana 2 / Seedance 2.0]
[Which mode? T2I / edit / compose / I2V / T2V / V2V]

## Resolution & Aspect Ratio
[Target aspect ratio + resolution. Set this BEFORE generating the image.
The image's ratio must match the video's ratio if animating.]

## Reference Assets
[List any reference images, style images, character images, or videos.
Assign each a role: identity / environment / style / motion / camera move]

## Constraints
[What must NOT appear. What must be preserved. What must match exactly.]

## Success Criteria
[How do you know when this is done?]
```

---

## The Two-Question Test

Before any generation call, ask:

1. **What is this for?** (If you can't answer in one sentence, the brief is incomplete.)
2. **What does success look like?** (If you can't describe what "good" looks like, you can't evaluate the output.)

If either answer is unclear — stop. Clarify before spending credits.

---

## Generating the Brief From a Thin Request

Users rarely give complete briefs. They say: "make me a hero image for the product launch." When the request is thin, do one of these:

**Option A — Ask one targeted question:**
Ask about the one most important missing element. If you don't know the use case, ask that. If you have the use case but not the composition, ask that. One question, not five.

**Option B — Proceed with explicit assumptions:**
State your assumptions inline before generating:
> "Assuming: studio product shot, 16:9 for web hero, white background, no copy in image — confirm or I'll proceed."

Never silently guess. Always surface the assumptions so they can be corrected.

---

## The Creative Stack

A complete creative brief drives two downstream documents when producing still images that will be animated:

```
Shot Brief
    ↓
Image Prompt       ← describes what the still image should contain
    ↓
Video Prompt       ← describes what motion the still image should become
```

The aspect ratio in the brief locks both. The lighting in the brief applies to both. The character identity in the brief persists across both.

Get the brief right once. Let both downstream prompts derive from it.

---

## Brief Quality Checklist

Before proceeding to generation, verify:

- [ ] Intent is one sentence (if it takes a paragraph to explain, the request needs scoping)
- [ ] Subject is specific (not "a person," not "a product" — the actual subject)
- [ ] Aspect ratio is set (not defaulted)
- [ ] Model is chosen and appropriate for the task
- [ ] Text in frame is listed verbatim (or confirmed absent)
- [ ] References have assigned roles
- [ ] Success criteria is defined (you can tell whether the output passes)

---

## Anti-Patterns

❌ Starting a generation from a Slack message without a brief
✅ "Before I generate: let me write a quick shot brief so we're aligned on what we're making."

❌ Assuming the aspect ratio (defaults to 1:1 and the video is 16:9)
✅ Setting the ratio in the brief, generating at that ratio, animating without composition loss

❌ "Make it look good" as success criteria
✅ "The product is center-frame with motivated lighting from the left, no text, no watermark, passes at mobile and desktop sizes"

❌ Generating five variants without knowing what success looks like, then picking the least bad
✅ Defining success before generating, then evaluating the first result against it


## Track: creative-pipeline

---

### creative-pipeline

> Three-phase production pipeline for generative media. Explore → Refine → Master. Cost-disciplined workflow from first idea to final asset.

# 🏭 Creative Pipeline

The most common waste in generative media production: spending final-asset budget on exploration, or shipping exploration-quality work as a final asset. This skill defines the three-phase pipeline that prevents both.

---

## The Three Phases

```
Phase 1: EXPLORE
  Model: Fast / low-cost (Nano Banana 2 at 1K, GPT Image 2 quality=low)
  Goal:  Lock composition, framing, subject geometry
  Cost:  Cheap. Generate many. A/B aggressively.
  Output: 1–2 winning compositions

Phase 2: REFINE
  Model: Production model (Nano Banana Pro at 2K, GPT Image 2 quality=medium/high)
  Goal:  Lock lighting, materials, color, text. Multi-turn refinement.
  Cost:  Medium. Run 2–4 conversational turns.
  Output: Approved internal deliverable

Phase 3: MASTER
  Model: Same production model at maximum quality
  Goal:  Final delivery asset
  Cost:  Per-asset, on approved Phase 2 prompts only.
  Output: Final deliverable for use/publication
```

---

## Phase 1 — Explore

**Budget:** ~$0.04/image (Nano Banana 2 at 1K), ~$0.04/image (GPT Image 2 quality=low)

**Goal:** Find the right composition. Composition is the thing you cannot easily change later. Lock it here.

**Workflow:**
1. Write the shot brief (`creative-brief` skill)
2. Use the LLM refiner to generate 4–5 variant prompts from the brief (`llm-refiner` skill)
3. Generate all variants at minimum viable resolution
4. Evaluate against the brief's success criteria
5. Pick 1–2 winners

**What to evaluate in Phase 1:**
- Subject placement — is it where you want it?
- Framing — does the composition work at this aspect ratio?
- Subject identity — is the person/object recognizable and correct?
- General mood — does it feel right?

**Do NOT evaluate in Phase 1:**
- Fine texture, material detail
- Typography accuracy
- Color grading precision
- Final lighting quality

These are Phase 2 concerns. Evaluating them in Phase 1 leads to premature escalation.

---

## Phase 2 — Refine

**Budget:** ~$0.13–0.15/image (Nano Banana Pro 2K, GPT Image 2 quality=medium)

**Goal:** Produce the final-quality version of the Phase 1 winner. Use conversational multi-turn refinement.

**Workflow:**
1. Promote the Phase 1 winning prompt to the production model
2. Generate at 2K / medium quality
3. Refine iteratively — one change per turn:
   - Turn 1: base composition
   - Turn 2: lighting adjustment
   - Turn 3: material/color polish
   - Turn 4: text placement (if needed)
   - Turn 5: final polish
4. Get internal approval

**Multi-turn refinement rules:**
- Stay in the same conversation (preserves thought signatures in Nano Banana models)
- Change one element per turn — never bundle multiple changes
- Re-state what must be preserved when making any change
- Expect character drift after ~5–6 turns in portrait-heavy work — re-anchor with a reference image

**Text-first hack (for typography-heavy images):**
Don't try to one-shot a poster with multiple lines of copy. Instead:
- Turn 1: "Draft three short slogans for [brand/brief]."
- Turn 2: "Render a poster using the second slogan, in bold sans-serif at the top."
The model locks in the text logic before drawing it. This dramatically improves text rendering.

---

## Phase 3 — Master

**Budget:** ~$0.24/image (Nano Banana Pro 4K, GPT Image 2 quality=high at large size)

**Goal:** Final delivery. Run only on approved Phase 2 prompts. No refinement turns here.

**Workflow:**
1. Take the exact approved Phase 2 prompt, unchanged
2. Regenerate at maximum quality/resolution
3. Log the full metadata (see logging section below)
4. Deliver

**Rules:**
- Never run Phase 3 on a prompt that hasn't cleared Phase 2 review
- Never do refinement turns in Phase 3 — if something's wrong, go back to Phase 2
- For licensed IP work: mandatory human review before delivery

---

## The Image-to-Video Extension

For workflows that animate still images, the pipeline extends:

```
Phase 1: Explore (image)
Phase 2: Refine (image)  ← Must match video aspect ratio
Phase 3A: Image Master
Phase 3B: Animate with Seedance 2.0 (I2V)
```

Critical constraint: the still image must be generated at the **video's target aspect ratio**. Don't generate 1:1 and plan to animate at 16:9. Generate at 16:9 from Phase 1.

**Image-to-video prompt formula:**
Once you have the right image, the video prompt is short:
```
Animate the provided image, preserve composition and colors.
[One specific motion]. [One camera move]. [Lighting consistency].
[Duration] seconds. [Aspect ratio]. Avoid identity drift.
```

The image carries the composition, color, character, and lighting. The video prompt directs motion only.

---

## Cost Reference (approximate, verify against current pricing)

| Phase | Model | Resolution | Cost/image |
|-------|-------|-----------|-----------|
| 1 (explore) | Nano Banana 2 | 1K | ~$0.04 |
| 1 (explore) | GPT Image 2 | 1024² low | ~$0.04 |
| 2 (refine) | Nano Banana Pro | 2K | ~$0.13 |
| 2 (refine) | GPT Image 2 | 1536×1024 med | ~$0.15 |
| 3 (master) | Nano Banana Pro | 4K | ~$0.24 |
| 3 (master) | GPT Image 2 | 2560×1440 high | ~$0.30 |
| Video T2V/I2V | Seedance 2.0 | 1080p | ~$0.14/sec |
| Video V2V | Seedance 2.0 | 1080p | ~$0.08/sec |

A typical production asset (12 explorations + 3 refinements + 1 master): ~$1.00–1.50 total.

---

## Grounding Hygiene (for IP-licensed work)

When using Google Search / Image Search grounding in Nano Banana:

- **Phase 1:** Enable grounding to lock in real-world geometry, landmarks, subjects
- **Phase 2/3:** Disable grounding before applying licensed brand colors, characters, or IP elements

Live image search can pull copyrighted or trademarked visuals into a generation. Never submit a grounded output to a licensed IP partner without a human IP review.

---

## Asset Logging

Every delivered asset must have a logged sidecar. Minimum:

```json
{
  "model": "gemini-3-pro-image-preview",
  "prompt": "[exact prompt text]",
  "system_instruction": "[if any]",
  "parameters": {
    "aspect_ratio": "16:9",
    "resolution": "4K",
    "thinking_level": "forced"
  },
  "references": ["[file path or content hash]"],
  "grounding": false,
  "generated_at": "2026-05-24T18:00:00Z",
  "output_url": "[URL or path]",
  "quality_score": 9
}
```

Without this log: three months later when you need "more of that style," you have nothing to reproduce it from.

---

## The Pipeline Decision Tree

```
New creative request
    ↓
Does a brief exist? ────No────→ Write the brief (creative-brief skill)
    ↓ Yes
Is composition clear? ──No────→ Phase 1: Explore (minimum resolution, fan-out)
    ↓ Yes
Is this the final quality? ─No→ Phase 2: Refine (production model, conversational)
    ↓ Yes
Was Phase 2 approved? ──No────→ Back to Phase 2
    ↓ Yes
Phase 3: Master
    ↓
Log the asset
    ↓
For video: Animate with Seedance 2.0 (I2V)
```


## Track: llm-refiner

---

### llm-refiner

> Use LLMs to write, critique, and improve image and video prompts before spending generation credits. The generate → critique → revise loop.

# 🔗 LLM Refiner

A failed Seedance 2.0 generation costs ~$0.84–1.68 per clip. A failed Nano Banana Pro render costs ~$0.13–0.24. A Claude Sonnet critique call costs cents. The math on running a refinement loop before generation is straightforward.

This skill encodes the pattern of using LLMs — specifically Claude and Gemini — to improve generative media prompts before credits are spent.

---

## The Core Pattern

```
User brief (thin)
     ↓
Claude: Expand into structured prompt
     ↓
Gemini (with reference image): Critique against the image
     ↓
Claude: Apply revisions
     ↓
Claude: Final sanity check
     ↓
Generate
```

Why Claude + Gemini? They complement each other:

| Model | Strength | Default role |
|-------|----------|-------------|
| **Claude** | Structured creative writing, holding long-form constraints, directorial voice | Drafter & Revisor |
| **Gemini** | Native multimodal — can actually SEE the reference image, grounded fact-checking | Visual Critic |

Claude drafts the best prompt structure. Gemini stress-tests it against what the image actually contains — catching motion conflicts, missing elements, lighting mismatches that a text-only model cannot see.

---

## Stage 1: Prompt Expansion (Claude)

When the user gives a thin brief ("make me a hero image of our new product"), expand it into a structured prompt before generating.

### System prompt for Claude as image prompt drafter:

```
You are an image-prompting specialist for [target model].
Your job is to convert a user's brief into a production-quality prompt.

Workflow:
1. Parse the brief. If critical context (use case, aspect ratio, exact copy,
   brand constraints) is missing, state assumptions inline and proceed.
2. Draft the prompt in the [model's formula]:
   [GPT Image 2: Scene / Subject / Important details / Use case / Constraints]
   [Nano Banana: Subject + Action + Location + Composition + Lighting + Style + Technical specs]
   [Seedance: Subject + Action + Environment + Camera + Style + Constraints]
3. Run a silent self-critique (see Stage 2 criteria). Revise if needed.
4. Output the final prompt inside <prompt>...</prompt> tags.
5. Output the API parameters inside <params>...</params> tags.
6. Output a one-line rationale inside <rationale>...</rationale> tags.

Rules:
- No filler adjectives (stunning, masterpiece, 8K, ultra-detailed, award-winning)
- Concrete visual facts over abstract style names
- For image-to-video work: set aspect ratio to match the intended video format
- For text-in-image: quote it exactly; specify placement and typography
```

---

## Stage 2: Self-Critique (Claude)

Before sending to a generator, run this critique against the draft prompt. Use it as an internal pass:

```
Score this prompt (1-5 each, with one-line justification):

1. Specificity: are subject, scene, lighting, and composition concrete
   visual facts, or vague adjectives?
2. Structure: does the prompt fill the formula cleanly?
3. Text rendering: if the image must contain text, is it quoted exactly,
   placement specified, typography described, verbatim rendering enforced?
4. Mode discipline: if this is an edit, is there an explicit change-vs-preserve
   list? If this is a composition, is each input labeled by role?
5. Anti-slop: does the prompt avoid filler adjectives?
6. Constraints: are exclusions explicit and invariants stated?
7. Parameter fit: is quality / size / thinking-level appropriate for the use case?

For any criterion scoring < 4, propose a specific rewrite of just that section.
Surgical edits only — do not rewrite the whole prompt.
Output the final revised prompt.
```

---

## Stage 3: Visual Critique (Gemini — when a reference image exists)

This is the decisive step for image-to-video workflows. Gemini can look at the reference image and verify whether the prompt actually matches what's in it.

```
You are a video prompt critic. The attached image is the reference that will be animated.
Read the prompt and answer these five questions:

1. Does the prompt describe motion that is physically plausible from the pose
   in the image? Name any conflict.
2. Does the prompt reference any element that is NOT visible in the image?
   List each one.
3. Is the lighting described in the prompt consistent with the lighting in the
   image? Name any mismatch.
4. Are there multiple competing camera movements? List them.
5. Does the prompt mix camera motion and subject motion in the same clause?
   Quote the clause if so.

For each problem found, give one specific revision in one sentence.
Output as a numbered list. Be terse.
```

Pass this critique and the original draft back to Claude:

```
Revise this prompt based on the critic's feedback.
Apply every revision grounded in the image. Reject any revision that
breaks the formula structure. Output only the revised prompt.

Original: [draft]
Critique: [Gemini output]
```

---

## Stage 4: Sanity Check (Claude)

A final Y/N pass before sending to the generator:

**For image prompts:**
```
Final check:
- No filler adjectives (stunning, 8K, masterpiece)? Y/N
- Subject front-loaded? Y/N
- Lighting clause specific (source, direction, quality)? Y/N
- Camera/lens named? Y/N
- Aspect ratio set? Y/N
- Text quoted exactly if present? Y/N
- Constraints explicit? Y/N

If any answer is N, fix it and reprint.
```

**For video prompts (Seedance):**
```
Final check:
- Word count 60–100? Y/N
- All six steps present (Subject, Action, Environment, Camera, Style, Constraints)? Y/N
- Single primary camera move? Y/N
- Camera and subject motion in separate clauses? Y/N
- Lighting clause present? Y/N
- Negative tail present (avoid jitter, bent limbs, identity drift...)? Y/N
- All @ references have assigned roles? Y/N

If any N, fix it and reprint.
```

---

## The Two-Stage Pipeline for Nano Banana (Gemini 3.5 Flash as Refiner)

For Nano Banana workflows, use Gemini 3.5 Flash as the prompt rewriter before invoking the image model:

```
User intent (1 sentence)
     ↓
Gemini 3.5 Flash (text-only — prompt rewriter)
     ↓
Structured Nano Banana prompt
     ↓
Nano Banana Pro / Nano Banana 2 (image generator)
```

Gemini 3.5 Flash system prompt for Nano Banana rewriting:

```
You are an expert prompt engineer for Nano Banana image generation models.
Convert the user's brief into a single, well-structured image prompt.

Rules:
1. Write a narrative paragraph, not a keyword list.
2. Order: subject → action → environment → composition → lighting → style → technical specs → text content.
3. Front-load the most important visual subject in the first 12 words.
4. Be hyper-specific. Replace generic nouns with concrete ones.
5. Specify lighting direction, quality, and color temperature.
6. Specify camera language: lens, angle, depth of field.
7. If text appears, quote it exactly; specify font style descriptively.
8. Use semantic positives, never negatives.
9. Include target aspect ratio and resolution at the end.
10. Output the refined prompt only. No preamble. Single paragraph.
```

---

## Multi-Prompt Fan-Out (Exploration Phase)

For exploration, ask the refiner to produce N variant prompts in one call:

```
User intent: [thin user input]

Generate 5 distinct image prompts exploring different creative directions.
Each prompt should follow the formula. Number them 1–5.
Vary at least one of: camera/lens, lighting setup, color grading, time of day, or compositional emphasis.
```

Run all 5 through the fast/low-resolution model to A/B them cheaply. Pick the winner. Promote to the full-quality model for refinement. See `creative-pipeline` for the full three-phase workflow.

---

## Pre-Mortem (High-Stakes Shots)

Before generating a high-value final asset, run a written pre-mortem:

```
It is 4 hours from now. The generation failed review.
What are the top 5 most likely reasons the output failed?
For each, propose one specific change to the prompt that would have prevented it.

Prompt: [final prompt]
Reference image attached.
Context: [IP constraints, brand requirements, known failure modes from prior attempts]
```

Apply defensible mitigations. Then generate.

---

## When to Skip the Loop

The refinement loop costs time and a few cents. Skip it when:
- The user is making a small adjustment to an existing result ("warmer lighting," "remove the second cup")
- The visual stakes are very low (internal thumbnails, fast brainstorm sketches)
- The prompt is already detailed and formula-shaped

Run the full loop for: final assets, licensed IP work, multi-reference compositions, any generation with text in frame, first pass of a new creative direction.


## Track: output-discipline

---

### output-discipline

> No approximations. No 'close enough.' Visual work is complete when it matches the brief — not when generation stops.

# 🚫 Output Discipline

Generation stopping is not the same as work being done. A generated image or video is done when it matches the brief that commissioned it — not when the model returns a result.

This skill defines what "complete" means for generative visual work, and what an agent must do when output is incomplete.

---

## The Completeness Standard

### Images

A generated image is complete when:

- [ ] It matches the brief's subject, composition, and lighting specification
- [ ] Any text in frame is exact, legible, and placement is correct
- [ ] No watermarks, artifacts, or unintended elements are present
- [ ] The aspect ratio matches the delivery spec (not cropped to fit after)
- [ ] The resolution is appropriate for the delivery channel
- [ ] It has passed the quality rubric (≥8/10 on the criteria)
- [ ] The prompt and parameters are logged

### Video

A generated video clip is complete when:

- [ ] Motion is stable (no jitter, no bent limbs, no temporal flicker)
- [ ] Subject identity is consistent across the full duration
- [ ] Camera movement is clean (one primary move, not competing)
- [ ] Lighting is stable and motivated
- [ ] Audio is present and synced (or intentionally silent with justification)
- [ ] Duration matches the specification
- [ ] It has passed the quality rubric (≥8/10 on the criteria)
- [ ] Prompt and parameters are logged

---

## What "Close Enough" Looks Like (and Why It's Not Done)

Common ways agents declare work done when it isn't:

❌ **"The composition is mostly right"** — If subject placement is wrong for the use case, it's not done. Crop decisions affect message.

❌ **"The text is readable"** — If the text is garbled, paraphrased, or has extra characters, it's not done. Text must be exact.

❌ **"It looks good to me"** — The brief defines done, not aesthetic preference. Check against the brief's success criteria.

❌ **"It's close, the human can adjust it in post"** — The agent's job is to deliver a complete asset. Incomplete work passed to a human is a quality failure.

❌ **"I tried three times and this is the best I got"** — Three blind retries without diagnosing the failure mode is not iteration — it's guessing. Diagnose, then retry with one specific change.

---

## When Generation Consistently Fails

If a prompt fails quality review multiple times:

1. **Diagnose with the failure mode table** (quality-review skill) — identify the specific symptom and its likely cause
2. **Change one variable** — apply the specific fix identified
3. **If still failing after 3 targeted iterations:** escalate to a different model or mode, or flag the brief for revision

Do not keep retrying the same prompt with hopes of a different result.

---

## Partial Delivery Protocol

Sometimes you're asked to deliver something quickly and not all elements are achievable in the time allowed. The protocol:

1. **Deliver what is complete and correct**
2. **Flag explicitly what is missing or approximate** — not buried, stated clearly
3. **Provide a specific plan for the gap** — not "will fix later" but "needs one refinement turn to lock the text — estimated 5 minutes"

Never present partial work as complete. Never let the human discover the gap by looking at the asset.

---

## The Logging Rule

Every delivered asset must have a logged record. Without it:
- You cannot reproduce "that style we used last quarter"
- You cannot diagnose why a generation failed at a specific parameter combination
- You cannot build a prompt library that improves over time

Minimum log:
```
model: [exact model string]
prompt: [exact prompt text]
parameters: [quality/size/thinking level/aspect ratio]
references: [list of reference asset paths or hashes]
generated_at: [timestamp]
output: [URL or path]
quality_score: [rubric score]
notes: [any iteration history, failure modes encountered]
```

---

## The Final Test

Before marking any asset delivered, ask:

> *"If I showed this to someone who read the brief but had not seen any prior attempts — would they say this is what was asked for?"*

If the answer is yes — it's done.
If the answer is no — it's not done, and the specific gap needs one more iteration.


## Track: prompt-craft

---

### prompt-craft

> Universal prompting principles for generative image and video models. What works across all reasoning-based image and video models — and what actively hurts quality.

# 🎯 Prompt Craft

The single most important shift in modern AI media prompting: these models are **reasoning engines**, not keyword matchers. They read your prompt as a brief, not as a tag cloud. Write briefs.

This skill covers the universal principles that apply across all current reasoning-based image and video models (GPT Image 2, Nano Banana Pro/2, Seedance 2.0). Model-specific details live in `guides/`.

---

## The Universal Principles

### 1. Write narrative prose, not keyword lists

Every reasoning-based model penalizes keyword stacking.

```
❌ "8K, masterpiece, ultra-detailed, cinematic, award-winning, professional"

✅ "A weathered florist closing up shop at dusk, tired posture, green apron,
   bundle of unsold flowers in one arm. 50mm documentary feel, mixed cool
   street light and warm shop window glow. Film grain, no posed glamour."
```

The first is noise. The second is a brief. The model spends reasoning budget on the second; it ignores the first.

### 2. Front-load the subject

Whatever you describe first gets the lion's share of the model's reasoning attention. If you open with the environment, the subject will be generic.

```
❌ "In a vast neon-lit cyberpunk city, a person walks..."
✅ "A courier in a worn synthetic jacket moves through neon-lit streets..."
```

### 3. Lighting is the highest-leverage variable

The single most quality-moving element in any visual prompt. Between a weak prompt and a strong one, the lighting clause often makes the difference.

```
❌ "good lighting"
✅ "soft golden hour backlight from camera-right, warm rim on the shoulders,
   cool ambient fill from the open sky above"
```

See `style-direction` for the complete lighting vocabulary.

### 4. Visual facts over vague praise

Every adjective that doesn't constrain a visual property is noise.

| Kill | Replace with |
|------|-------------|
| stunning, incredible, gorgeous | specific lighting + composition |
| epic, dramatic | low-angle hero shot / high-contrast chiaroscuro |
| professional, premium | camera + lens + lighting setup |
| cinematic | cinematic film tone, 35mm, teal-and-orange grade |
| detailed, sharp | 85mm macro lens / 100mm macro lens |
| modern, clean | specific material + color + spacing description |

### 5. Specify what must NOT appear

Exclusions are real constraints. State them explicitly.

For models with negative prompt fields (Seedance): use the negative tail.
For models without (Nano Banana Pro): translate to positive framing.

| Negative framing | Positive framing |
|-----------------|-----------------|
| "no people" | "an empty, deserted street at dawn" |
| "no clutter" | "a clean minimalist surface with generous negative space" |
| "not blurry" | "tack-sharp focus throughout" |
| "no watermark" | "a clean image with no text, signatures, or overlay elements" |
| "no extra fingers" | "anatomically accurate hands, five fingers each, naturally posed" |

### 6. One change per iteration

When refining with follow-up edits: change one axis at a time. Never bundle multiple orthogonal changes.

```
❌ "Make it more premium, more realistic, fix the text, change the outfit,
   improve the background, and make it warmer"

✅ Turn 1: "Warm the lighting — shift key light to sunset color temperature."
   Turn 2: "Remove the extra chair on the left. Keep everything else."
   Turn 3: "The sign on the wall should read 'DAILY GRIND'. Render verbatim."
```

### 7. Text in images is typography, not language

When an image must contain readable text, treat it as a typography specification.

**The five-step text protocol:**
1. **Quote it exactly** — `"SOUND YOU CAN FEEL"` in the prompt
2. **Specify placement** — "centered in the left third, headline above subhead"
3. **Specify typography** — "bold condensed sans-serif, white on black, tight kerning"
4. **Add a hard stop** — "Render verbatim. No extra characters. No duplicate text."
5. **For tricky words** — spell letter-by-letter in the prompt

**Quality escalation for text:**
- 1–5 words: low/medium quality is fine
- Dense panels, small text, multi-script: use `quality="high"` (GPT Image 2) or `thinking_level: "high"` (Nano Banana)

### 8. Aspect ratio is creative — not a crop setting

Generating at 1:1 and planning to crop to 16:9 wastes half your composition. The model composes for the frame you give it.

Set the aspect ratio in the brief. Generate at that ratio. This is especially critical for image-to-video workflows — the still and the animation must share the same ratio.

### 9. Iterate in the same session

For multi-turn models (Nano Banana Pro/2, GPT Image 2 edit mode): stay in the same conversation. The model carries reasoning context across turns. Starting a new session loses the accumulated context and forces a cold restart.

### 10. Save every prompt

The prompt is the asset. Not the image. The image can be regenerated from the prompt. Without the prompt — three months from now when you need "more of that style" — you have nothing.

Log every call with: model, prompt, parameters, reference asset paths, output URL, quality score.

---

## Model Selection Guide

| Task | Best model | Why |
|------|-----------|-----|
| Text in images (posters, UI, infographics) | **GPT Image 2** | Near-perfect text rendering |
| Final production image, complex scenes | **Nano Banana Pro** | Deep reasoning, forced full thinking |
| Fast iteration, exploration, batch work | **Nano Banana 2** | Pro-grade quality at Flash speed |
| Animating a still image | **Seedance 2.0** (I2V) | Image-to-video with audio |
| Text-to-video from scratch | **Seedance 2.0** (T2V) | Multimodal, native audio |
| Editing an existing video | **Seedance 2.0** (V2V) | ~39% cheaper, more consistent |

**Cost ladder (approximate, verify against current provider pricing):**
- Nano Banana 2 at 1K: ~$0.04/image
- Nano Banana Pro at 2K: ~$0.13/image
- Nano Banana Pro at 4K: ~$0.24/image
- Seedance 2.0 (T2V/I2V): ~$0.14/second
- Seedance 2.0 (V2V): ~$0.08/second

Default rule: *explore at minimum resolution and cost, promote to final quality only after composition is locked.*

---

## Formula Quick Reference

Each model has its own formula. The universal skeleton:

```
[What/Who] + [Doing What] + [Where/Context] + [Camera/Framing] + [Lighting] + [Style] + [Constraints]
```

| Model | Formula name | Slots |
|-------|-------------|-------|
| GPT Image 2 | 5-slot template | Scene / Subject / Important details / Use case / Constraints |
| Nano Banana 2 | 7-element formula | Subject + Action + Location + Composition + Lighting + Style + Technical specs |
| Nano Banana Pro | 5-element formula | Subject + Action + Location/context + Composition + Style (+ Text) |
| Seedance 2.0 | 6-step formula | Subject + Action + Environment + Camera + Style + Constraints |

See the model guides in `guides/` for each formula in full detail.

---

## The Prompt Craft Loop

For any important generation:

```
1. Write the brief (creative-brief skill)
2. Draft the prompt (this skill's formulas)
3. Critique the prompt (llm-refiner skill)
4. Generate at low resolution
5. Evaluate against the brief (quality-review skill)
6. Iterate with one change per turn
7. Promote to final resolution
8. Log the prompt + parameters
```

Skipping steps 3, 5, or 8 is where most quality is lost.


## Track: quality-review

---

### quality-review

> Evaluate generated images and video against the brief. What passes, what fails, and how to iterate without losing the thread.

# 🔍 Quality Review

A generated asset is not done because generation finished. It's done when it passes review against the brief that commissioned it.

This skill defines how to evaluate — and how to iterate when evaluation fails.

---

## The Two Review Questions

Before any further evaluation, ask:

1. **Does it match the brief?** (Does it show what the brief asked for, in the framing asked for, with the lighting asked for?)
2. **Would the intended viewer understand what this is?** (Not "does it look impressive" but "does it communicate correctly?")

If either answer is no — evaluate why, then iterate with one change.

---

## Image Quality Rubric

Score 0–2 on each axis (10 total). 8+ ships. 6–7 iterate. 5 or below: diagnose before retrying.

| Axis | 0 | 1 | 2 |
|------|---|---|---|
| **Brief adherence** | Major elements missing or wrong | Captures intent, misses specifics | Matches brief precisely |
| **Subject clarity** | Subject is unclear or wrong | Subject is present but generic | Subject is specific and correct |
| **Lighting accuracy** | Lighting is wrong or inconsistent | Lighting is present but off-brief | Lighting matches the specification |
| **Composition** | Framing / placement is wrong | Roughly correct, imprecise | Composition is exactly as specified |
| **Output discipline** | Watermarks, placeholders, or artifacts | Minor artifact | Clean, complete, production-ready |

---

## Video Quality Rubric (Seedance)

| Axis | 0 | 1 | 2 |
|------|---|---|---|
| **Instruction adherence** | Ignores major brief elements | Captures gist, misses specifics | Matches brief precisely |
| **Motion stability** | Visible jitter, warping, bent limbs | Minor artifacts at moments | Smooth throughout |
| **Identity consistency** | Subject visibly drifts across clip | Mild drift in one or two frames | Locked across full clip |
| **Lighting coherence** | Lighting changes inexplicably | One lighting hiccup | Consistent and motivated |
| **Audio-visual sync** | Audio and visuals don't align | Mostly aligned with one slip | Perfectly synced |

---

## What to Do When It Fails

**Rule: Diagnose before retrying.** Blind retries waste credits and produce no learning.

Match the symptom to the cause, then apply one fix:

### Image Failures

| Symptom | Likely cause | One fix |
|---------|-------------|---------|
| Text is garbled or invented | Text not quoted; no verbatim instruction | Quote exactly, add "Render verbatim. No extra characters." Use quality=high. |
| Edit changed the whole scene | No preserve list | Re-state the full preserve list. Use "change only X, keep everything else the same." |
| Composition is generic / AI-looking | Filler adjectives crowded out visual facts | Strip stunning/8K/masterpiece. Replace with concrete facts (lens, lighting, materials). |
| Subject is in the wrong position | Composition not specified | State framing, viewpoint, and placement explicitly. |
| Multiple elements drifted across turns | Bundled multi-axis edit | One change per turn. Split into a chain of small edits. |
| Character / brand element drifted | No anchor; preserve list missing identity | Provide reference image. Lock identity in preserve list. |
| Output looks staged when realism wanted | Studio language used | Remove "professional photography," "cinematic," "award-winning." Add "candid," "documentary," "unposed." |
| Dense infographic blurry / wrong | quality=low or size too small | Use quality=high. Use landscape size. Include exact numbers in prompt. |
| Mixed-script text only one script correct | Each script not separately specified | Quote each script separately with font behavior for each. |

### Video Failures

| Symptom | Likely cause | One fix |
|---------|-------------|---------|
| Jittery, shaky frame | Multiple camera moves OR camera+subject mixed | Reduce to one primary camera move; separate clauses |
| Bent or extra limbs | Missing negative tail OR fast subject motion | Add `avoid bent limbs`; downshift speed one tier |
| Character face drifts | I2V image too small OR motion too dramatic | Re-export image at ≥1536px; reduce motion to subtle |
| Text in frame warps | Animating text-bearing elements | Pin text: `keep all text sharp and unchanged`; animate background only |
| Lighting changes mid-clip | No lighting clause OR conflict with image | Add explicit lighting clause; match to image |
| Lip-sync wrong | Spoken text not in double quotes | Quote the dialogue |
| Output looks "generic AI" | Adjective stack | Replace every "epic/cinematic/amazing" with a specific visual fact |
| Background changes between cuts | No environment continuity instruction | Add `maintain same location and lighting across cuts` |

---

## The One-Variable Rule

When something is wrong, change **exactly one element** for the next attempt:
- Camera move OR subject action OR lighting OR style OR text — never two at once.

This is the only way to learn which change actually moved the result. Changing three things at once means you don't know what fixed it (or whether it's actually fixed for the right reason).

---

## Evaluating Against the Brief (Structured)

Use this as a review prompt for an LLM evaluator (Gemini with the image, or Claude reading a detailed description):

```
Evaluate this generated [image / video clip] against the following brief.

Brief:
[paste the shot brief]

Score each criterion:
1. Subject correctness (is the right subject present with the right attributes?) — 0/1/2
2. Composition match (is the framing/placement/aspect ratio what was specified?) — 0/1/2
3. Lighting match (does the lighting match the specification?) — 0/1/2
4. Style match (does the visual style match the specified genre/grade?) — 0/1/2
5. Constraints met (are all exclusions respected and invariants preserved?) — 0/1/2

Total: /10

For any criterion scoring < 2, describe what is wrong in one sentence.
Then propose one specific prompt change that would fix it.
```

---

## Pre-Delivery Checklist

Before calling any asset done:

**Images:**
- [ ] No watermarks, placeholder text, or artifacts
- [ ] Text in frame is exact (if any) and legible
- [ ] No unintended brand marks, logos, or trademarks
- [ ] Aspect ratio matches the delivery spec
- [ ] Resolution is appropriate for the delivery channel
- [ ] Metadata sidecar logged (prompt, model, parameters)

**Video:**
- [ ] No jitter, bent limbs, or temporal flicker
- [ ] Identity is consistent across the full clip duration
- [ ] Lighting is stable and motivated
- [ ] Audio (if present) is synced and appropriate
- [ ] Duration matches the specified length
- [ ] For licensed IP work: human review completed before submission
- [ ] Metadata sidecar logged

---

## Learning from Failure

Every evaluation that results in a retry should be logged:

```
Failed generation: [brief one-liner]
Symptom: [what was wrong]
Cause: [why it happened]
Fix applied: [what was changed]
Did it work: [yes/no]
```

This log is how prompt quality compounds over time. Patterns in the log reveal recurring failure modes that can be caught at the briefing stage before any credits are spent.


## Track: style-direction

---

### style-direction

> Art direction vocabulary for generative media. Lighting, camera, film, color, materials — the specific language that turns vague prompts into directed visual work.

# 🎨 Style Direction

This skill is a vocabulary reference. Generative models respond to specific, named visual properties. The difference between a generic output and a directed output is almost always in the specificity of these terms.

---

## Lighting

Lighting is the single highest-leverage element in any visual prompt. Be precise: name the source, direction, quality, and color temperature.

### Lighting Setups

| Setup | Use when | Key phrase |
|-------|----------|-----------|
| **Three-point softbox** | Product photography, studio portrait | `three-point softbox setup, key at 45°, fill at 30% intensity, hair light from behind` |
| **Rembrandt** | Dramatic portrait, editorial | `Rembrandt lighting from camera-left, soft diffusion, triangular cheek highlight` |
| **Chiaroscuro** | Moody, theatrical | `chiaroscuro, single hard key light, deep blacks, 5:1 contrast ratio` |
| **Caravaggio** | Extreme drama, single source | `single dramatic source from above, deep shadow blacks, theatrical` |
| **Golden hour** | Warm portraits, hero shots | `soft golden hour backlight from the right, long warm shadows` |
| **Blue hour** | Cool twilight, atmospheric | `cool blue-hour ambient light, just after sunset` |
| **Natural window** | Lifestyle, product, editorial | `soft natural north-facing window light, diffused` |
| **Overcast diffused** | Even editorial, product without shadows | `even overcast diffused light, no harsh shadows, flat` |
| **Practical lights** | Interior scenes, urban | `warm practical lamps, cool TV glow from camera-left` |
| **Neon** | Nightscapes, cyberpunk | `neon-lit street, magenta key from above-left, cyan rim from behind` |
| **Backlit / Rim** | Silhouettes, edge separation | `dramatic rim light against dark background, backlit silhouette at sunset` |
| **Volumetric** | Atmosphere, depth, scale | `volumetric god-rays through dust and mist` |
| **Caustic** | Underwater, dreamy | `caustic light patterns, refracted blues and greens, soft ambient diffusion` |

### Lighting Descriptors
- **Quality:** soft, hard, diffused, harsh, specular
- **Direction:** from camera-left/right, from above, from below, backlight, sidelight, top light
- **Temperature:** warm (golden/amber), cool (blue/cyan), neutral
- **Ratio:** 3:1 (standard), 5:1 (dramatic), 7:1 (extreme)

---

## Camera & Lens

Named camera hardware changes the visual DNA of a shot. Use hardware that matches the mood.

### Camera References

| Camera | Visual signature | Use for |
|--------|-----------------|---------|
| **Hasselblad medium format** | Editorial, smooth tonality, high-end | Fashion, luxury product |
| **Arri Alexa** | Cinematic, filmic, organic | Narrative, film-style |
| **Fujifilm X100** | Documentary, authentic color science | Street, lifestyle, candid |
| **Kodak disposable** | Raw, snapshot, nostalgic | Social content, intimate |
| **Sony A7R V** | Technical, precise, modern | Architecture, product |
| **GoPro** | Immersive, slight distortion, action | Sports, POV, adventure |
| **Pentax 67** | Classic medium format, slightly warm | Portrait, editorial |

### Lens References

| Lens | Effect | Use for |
|------|--------|---------|
| **24mm wide-angle** | Expansive perspective, environmental scale | Architecture, establishing |
| **35mm** | Natural human field of view, reportage | Documentary, street |
| **50mm prime** | Neutral, invisible, natural | Versatile, product |
| **85mm portrait** | Compressed background, flattering | Portraits, product hero |
| **100mm macro** | Extreme close-up, dramatic detail | Product texture, nature |
| **200mm telephoto** | Heavy compression, compressed space | Sports, wildlife, abstract |
| **Anamorphic** | Cinematic widescreen, characteristic flare | Film-style, epic |
| **Fisheye** | Extreme curvature, surreal | Creative, immersive |

### Aperture Language
- f/1.4–f/2.0: very shallow DoF, heavily blurred background
- f/2.8: shallow DoF, comfortable bokeh
- f/5.6–f/8: balanced, standard
- f/11–f/16: deep focus, everything sharp

### Composition Vocabulary
**Shot types:** extreme close-up, close-up, medium close-up, medium shot, medium-full, wide, establishing
**Angles:** low angle, high angle, eye-level, bird's-eye, Dutch tilt, worm's-eye, over-the-shoulder, top-down isometric
**Composition rules:** rule of thirds, centered symmetry, golden ratio, leading lines, negative space, frame within a frame

---

## Film Stocks & Color Grades

| Film / Grade | Signature | Use for |
|-------------|-----------|---------|
| **Kodak Portra 400** | Warm skin tones, slightly faded blacks | Portrait, lifestyle |
| **Fujichrome Velvia** | Deeply saturated reds and greens, contrasty | Nature, landscape |
| **Kodachrome** | Saturated reds, warm cast, 1960s–70s | Vintage, nostalgia |
| **Polaroid SX-70** | Soft, warm, slightly faded, square | Social, intimate |
| **Tri-X B&W** | Heavy grain, deep contrast, gritty | Street, documentary |
| **Wet-plate collodion** | Tintype look, vignette, period | Historical, artistic |
| **Cinematic teal-and-orange** | Modern cinema standard | Film-style, drama |
| **Log/flat** | Neutral, for grading | Technical, flexible |

---

## Art & Illustration Styles

| Style | Aesthetic |
|-------|-----------|
| **Studio Ghibli** | Painterly, warm, hand-drawn animation |
| **Pixar 3D** | Polished CG, cinematic lighting |
| **Cel-shaded anime** | Flat colors, hard shadow lines |
| **Watercolor** | Soft edges, paper texture, paint bleed |
| **Pencil sketch** | Graphite linework, cross-hatching |
| **Vector flat** | Geometric, solid colors, no gradients |
| **Isometric 3D** | Axonometric, game-asset style |
| **Art nouveau** | Organic curves, ornate, Mucha-influenced |
| **Bauhaus** | Geometric, primary colors, modernist |
| **Brutalist concrete** | Heavy, monolithic, raw textures |
| **Risograph** | Limited color, grain, offset printing look |
| **Tilt-shift miniature** | Selective focus making scenes look like models |

---

## Materiality & Texture

Never use generic material names. Always specify material AND finish.

### The Specificity Ladder

| Generic | Specific |
|---------|----------|
| wood | weathered oak with visible grain / polished walnut / bleached driftwood |
| metal | brushed titanium / patinated brass / anodized aluminum / hammered bronze |
| stone | honed Carrara marble / rough black basalt / weathered limestone |
| fabric | raw linen with visible weave / matte silk crepe / chunky Aran wool |
| glass | thick-walled hand-blown borosilicate with internal bubbles / tempered with clean reflections |
| plastic | matte ABS with slight texture / polished polycarbonate / translucent frosted |
| leather | full-grain vegetable-tanned / distressed suede / patent with high gloss |

---

## Motion Vocabulary (for video)

### Camera Movements
| Movement | Term | Use for |
|----------|------|---------|
| Toward subject | `push-in` / `dolly in` | Emotional emphasis |
| Away from subject | `pull-out` / `dolly out` | Reveals, establishing |
| Horizontal sweep | `pan` | Scanning, following |
| Follow the subject | `tracking shot` | Action, walking |
| Rotate around subject | `orbit` / `arc` | Product turntable, portrait |
| High altitude | `aerial` / `drone shot` | Scale, landscape |
| Natural shake | `handheld` | Documentary, intimacy |
| Static | `fixed` / `locked-off` | Action-only shots |

### Speed Ladder (for video)
| Tier | Keywords | Notes |
|------|----------|-------|
| Imperceptible | `barely moves`, `subtle drift` | Almost static |
| Slow | `slow`, `gentle`, `gradual` | Safe default for quality |
| Medium | `smooth`, `controlled` | Natural pacing |
| Fast | `dynamic`, `swift` | High jitter risk — use sparingly |

**Critical rule:** `fast` applies to **at most one element** per video prompt. Fast camera + fast subject + fast scene = chaos.

---

## Color Specification

Named colors are imprecise. Use hex codes for production work.

```
"A sky transitioning from #1B2845 (deep indigo) at the top to
#FF6B35 (warm sunset orange) at the horizon"
```

When hex isn't necessary, use named-color anchors:
- `deep cobalt` / `muted teal` / `warm amber` / `dusty rose`
- Avoid: `blue`, `orange`, `warm`, `cool` — too vague to direct

---

## The Anti-Adjective Rule

Any word that applies to every possible "good" image should be cut. Before submitting a prompt, scan for these and delete or replace:

| Cut | Replace with |
|-----|-------------|
| stunning | specific light + composition |
| incredible | specific material + detail |
| epic | low-angle + scale |
| masterpiece | nothing — let the specifics speak |
| cinematic | `cinematic film tone, 35mm, anamorphic` |
| professional | specific camera + lighting setup |
| beautiful | describe what makes it beautiful |
| high quality | nothing — model already outputs high quality by default |
| 8K, ultra-detailed | nothing — use lens + quality settings instead |
