---
name: quality-review
description: "Evaluate generated images and video against the brief. What passes, what fails, and how to iterate without losing the thread."
version: 1.0.0
metadata: {"openclaw":{"emoji":"🔍","requires":{"bins":[]}}}
---

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
