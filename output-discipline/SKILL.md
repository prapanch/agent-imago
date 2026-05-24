---
name: output-discipline
description: "No approximations. No 'close enough.' Visual work is complete when it matches the brief — not when generation stops."
version: 1.0.0
metadata: {"openclaw":{"emoji":"🚫","requires":{"bins":[]}}}
---

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
