---
name: creative-brief
description: "Define the visual intent before generating anything. A creative brief is the contract between what you're asked to make and what you actually produce. No generation without a brief."
version: 1.0.0
metadata: {"openclaw":{"emoji":"📝","requires":{"bins":[]}}}
---

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
