---
name: prompt-craft
description: "Universal prompting principles for generative image and video models. What works across all reasoning-based image and video models — and what actively hurts quality."
version: 1.0.0
metadata: {"openclaw":{"emoji":"🎯","requires":{"bins":[]}}}
---

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
| Animating a still image | **Gemini Omni Flash** | Native I2V, stateful editing, reference tags |
| Text-to-video (multi-shot narrative) | **Gemini Omni Flash** | World knowledge, conversational editing |
| Iterative video editing | **Gemini Omni Flash** | Stateful `previous_interaction_id` |
| High-fidelity I2V or V2V | **Seedance 2.0** | Proven @reference system, V2V mode |

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
| Gemini Omni Flash | 6-element video formula | Subject + Action + Environment + Camera + Audio + Constraints |

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
