---
name: llm-refiner
description: "Use LLMs to write, critique, and improve image and video prompts before spending generation credits. The generate → critique → revise loop."
version: 1.0.0
metadata: {"openclaw":{"emoji":"🔗","requires":{"bins":[]}}}
---

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
