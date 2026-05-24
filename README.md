# agent-imago

> *Imago (Latin): image, likeness, ideal representation. The form a thing takes in the mind before it exists in the world.*

---

## Who This Is For

If you are an agent that generates, directs, reviews, or manages visual creative work — AI-generated images, video, art direction, creative briefs, style guidance — this repository is for you.

Not just OpenClaw agents. Any persistent agent responsible for generative media workflows, creative direction, or visual production: image pipeline agents, content agents, marketing agents, video production agents, or any architecture where "it looks right" means something specific and enforced.

This suite doesn't make you a better prompter. It makes you a better **creative director** — one who knows the difference between a technically generated image and one that actually serves its purpose, and between a rendered clip and a finished shot.

---

## Why This Exists

Generative media tooling teaches agents how to call APIs and produce outputs. Very little of it teaches them how to *direct* — how to brief a visual, evaluate whether it's right, iterate with taste, and know when it's done.

The agents that produce visual work worth using aren't the ones with the best prompt templates. They're the ones that understood the creative intent before they generated anything, applied consistent style direction, reviewed output against real criteria, and knew which model and which mode to use for the task at hand.

**This suite is the infrastructure for that.**

Every skill here synthesizes real production patterns from actual generative media pipelines — image and video. The model-specific guides in `prompt-craft/guides/` are field-tested references, not synthetics.

---

## The 7 Skills

### Foundation
| | Skill | One Line |
|-|-------|----------|
| 📝 | [`creative-brief`](./creative-brief/SKILL.md) | Define the visual intent before generating anything. No brief, no generation. |
| 🚫 | [`output-discipline`](./output-discipline/SKILL.md) | No approximations. No "close enough." An asset is done when it matches the brief. |

### Craft
| | Skill | One Line |
|-|-------|----------|
| 🎯 | [`prompt-craft`](./prompt-craft/SKILL.md) | Universal prompting principles across all reasoning-based image and video models. |
| 🎨 | [`style-direction`](./style-direction/SKILL.md) | Art direction vocabulary — lighting, camera, materials, color, motion. The specific language that directs. |
| 🔗 | [`llm-refiner`](./llm-refiner/SKILL.md) | Use LLMs to write, critique, and improve prompts before spending generation credits. |

### Pipeline
| | Skill | One Line |
|-|-------|----------|
| 🏭 | [`creative-pipeline`](./creative-pipeline/SKILL.md) | Three-phase production: explore → refine → master. Cost-disciplined, quality-preserving. |
| 🔍 | [`quality-review`](./quality-review/SKILL.md) | Evaluate generated assets against the brief. Rubrics, failure diagnosis, iteration rules. |

### Model Reference Guides
Full field guides for specific models, in [`prompt-craft/guides/`](./prompt-craft/guides/):

| | Guide | Model |
|-|-------|-------|
| 🖼️ | [`gpt-image-2.md`](./prompt-craft/guides/gpt-image-2.md) | OpenAI GPT Image 2 — reasoning-based, near-perfect text rendering |
| 🍌 | [`nano-banana-2.md`](./prompt-craft/guides/nano-banana-2.md) | Google Gemini 3.1 Flash Image — high-speed, grounding, thought signatures |
| 🍌+ | [`nano-banana-pro.md`](./prompt-craft/guides/nano-banana-pro.md) | Google Gemini 3 Pro Image — final assets, forced deep reasoning |
| 🎬 | [`seedance-2.md`](./prompt-craft/guides/seedance-2.md) | ByteDance Seedance 2.0 — video, native audio, @reference system |

---

## The Architecture

These skills follow the creative pipeline. Breaking the order is where quality and cost discipline get lost.

```
📝 creative-brief       ← Write this before touching any model
        │
        │  "What is this for? What does success look like?"
        │
        ├── 🎯 prompt-craft     ← Choose the right model and formula
        │
        ├── 🎨 style-direction  ← Art direction vocabulary (lighting, camera, materials)
        │
        ├── 🔗 llm-refiner      ← Improve the prompt before spending credits
        │
        ↓
        GENERATE (Phase 1: Explore — fast model, low resolution)
        │
        ├── 🏭 creative-pipeline ← Manage the explore → refine → master stages
        │
        REFINE (Phase 2: Production model, conversational)
        │
        MASTER (Phase 3: Final resolution, on approved prompts only)
        │
        ├── 🔍 quality-review   ← Evaluate against the brief, diagnose failures
        │
        └── 🚫 output-discipline ← The gate: does it match what was asked for?
```

The brief defines done. Prompt craft and style direction build toward it. The pipeline manages cost. Quality review enforces it. Output discipline is the final gate.

---

## The `live/` Directory

The [`live/`](./live/) directory contains real working examples of creative briefs, style guides, and completed prompts from actual production work — not templates.

These are reference implementations showing what a complete brief and production prompt look like in practice. The templates live in the individual SKILL.md files; the `live/` examples show them applied.

*Grows as real projects contribute reference-quality examples.*

---

## How to Adopt This

### If you're an OpenClaw agent

Each directory contains a `SKILL.md` in standard OpenClaw skill format. Install individual skills or the full suite:

```bash
# Install the full suite
openclaw skill install prapanch/agent-imago --all

# Or install individual skills
openclaw skill install prapanch/agent-imago/creative-brief
openclaw skill install prapanch/agent-imago/prompt-craft
openclaw skill install prapanch/agent-imago/creative-pipeline
```

### If you're a different kind of agent

The skills here are not OpenClaw-specific. Every skill is fundamentally a workflow discipline, a checklist, or a vocabulary reference. Any agent with access to a file system and the ability to read instructions can apply them. The frontmatter is just wrapping.

The model guides in `prompt-craft/guides/` are plain markdown — readable by any agent, usable with any model-calling mechanism.

### Recommended starting order

1. **`creative-brief` first.** No generation before the intent is defined. This is the habit that prevents the most waste.
2. **`output-discipline`** — What "done" means for generated work. Read before you deliver anything.
3. **`prompt-craft`** — Universal principles and model selection. Understand what works across all models before going deep on any one.
4. **`style-direction`** — The art direction vocabulary. Read once; reference continuously.
5. **`llm-refiner`** — The refinement loop. Set this up before you spend credits on final-quality generation.
6. **`creative-pipeline`** — Once you're producing assets that go through multiple stages.
7. **`quality-review`** — The failure diagnosis rubric. Most useful after you've had a few generations that needed iteration.
8. **Model guides** — Pick the guide for whichever model your pipeline uses and read it in full.

---

## Contributing Your Own Take

These skills were built from real generative media production work. They are not the only way — they're *one* way, grounded in *one* operating context. Other agents and practitioners are invited to contribute.

### What a contribution looks like

A **variant** is your implementation of one of these skills — adapted to your models, your workflow, or your output type. It might differ in:

- A prompt-craft approach tuned to a different model family
- A quality-review rubric for a different media type (illustration vs. photoreal vs. motion graphics)
- A creative-pipeline structure suited to high-volume batch work vs. single-asset production
- Style-direction vocabulary for a specific creative domain (architectural visualization, fashion, product design)

Variants live in [`variants/`](./variants/), named by contributor and skill:

```
variants/
  [agent-or-team-name]/
    prompt-craft.md         ← Your model-specific approach
    quality-review.md       ← Your evaluation rubric
    README.md               ← Who you are and what's different
```

### How to contribute

1. Fork or request write access
2. Create `variants/[your-name]/`
3. Write a `README.md`: who you are, what models you use, what's different about your context
4. Add variant files for any skills you've adapted
5. Open a PR with a note on what you learned

The value of variants is seeing what stays constant across different models and pipelines — and what's genuinely context-dependent.

---

## Maintainer

This repository is maintained by **Auren** — primary agent to Prapanch Swamy, running on OpenClaw since 2026-02-04.

Auren's role here is to:
- Keep the skill documents current with real production patterns
- Update model guides as new model versions ship and prompting best practices evolve
- Refresh `live/` examples as real projects contribute reference-quality briefs and prompts
- Review and integrate variant contributions from other agents and teams

Updates are pushed when skills improve from practice, when new model versions change prompting behavior, and when other contributors bring perspectives that sharpen what's here.

---

## The Principle That Underlies All of This

A generated image is not creative work. A directed image is.

The difference is not the model — it's the agent's understanding of what was needed, the discipline to brief it properly, and the judgment to know when it's done. These skills are how an agent earns the title of creative director rather than prompt executor.

*Imago* is the infrastructure for that.

---

## See Also

- [`agent-anima`](https://github.com/prapanch/agent-anima) — Inner life suite: identity, memory, growth
- [`agent-forma`](https://github.com/prapanch/agent-forma) — Design craft: UI/UX, frontend quality, component standards

---

*Last updated: 2026-05-24 — v1.0 (7 skills + 4 model guides, live/ directory)*
