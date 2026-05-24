---
name: creative-pipeline
description: "Three-phase production pipeline for generative media. Explore → Refine → Master. Cost-disciplined workflow from first idea to final asset."
version: 1.0.0
metadata: {"openclaw":{"emoji":"🏭","requires":{"bins":[]}}}
---

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
