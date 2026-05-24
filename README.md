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

## The Skills

### Foundation
| | Skill | One Line |
|-|-------|----------|
| 📝 | [`creative-brief`](./creative-brief/SKILL.md) | Define the visual intent before generating anything. |
| 🚫 | [`output-discipline`](./output-discipline/SKILL.md) | No approximations. No "close enough." Ship complete visual work. |

### Craft
| | Skill | One Line |
|-|-------|----------|
| 🎯 | [`prompt-craft`](./prompt-craft/SKILL.md) | Universal prompting principles for generative image and video models. |
| 🎨 | [`style-direction`](./style-direction/SKILL.md) | Art direction vocabulary — lighting, camera, materials, color, style. |
| 🔗 | [`llm-refiner`](./llm-refiner/SKILL.md) | Use LLMs to write, critique, and improve prompts before spending credits. |

### Pipeline
| | Skill | One Line |
|-|-------|----------|
| 🏭 | [`creative-pipeline`](./creative-pipeline/SKILL.md) | Three-phase production: explore → refine → master. |
| 🔍 | [`quality-review`](./quality-review/SKILL.md) | Evaluate generated assets against the brief. What passes, what fails. |

### Model Reference Guides
Full field guides for specific generative models, in `prompt-craft/guides/`:

| | Guide | Model |
|-|-------|-------|
| 🖼️ | [`gpt-image-2.md`](./prompt-craft/guides/gpt-image-2.md) | OpenAI GPT Image 2 — reasoning-based image generation |
| 🍌 | [`nano-banana-2.md`](./prompt-craft/guides/nano-banana-2.md) | Google Gemini 3.1 Flash Image — high-speed production image model |
| 🍌+ | [`nano-banana-pro.md`](./prompt-craft/guides/nano-banana-pro.md) | Google Gemini 3 Pro Image — final-asset image model |
| 🎬 | [`seedance-2.md`](./prompt-craft/guides/seedance-2.md) | ByteDance Seedance 2.0 — multimodal video generation |

### Reference
| | | |
|-|-|-|
| 🖼️ | [`live`](./live/) | Working creative briefs and examples from real projects |

---

## The Model Landscape (as of May 2026)

Three model families cover the generative media stack:

```
                    Still Images
                    ┌──────────────────────────────────┐
GPT Image 2         │ Best for: text in images,        │
                    │ editing, UI mockups, multi-ref    │
                    └──────────────────────────────────┘

Nano Banana Pro     ┌──────────────────────────────────┐
(Gemini 3 Pro)      │ Best for: final assets, complex  │
                    │ scenes, character consistency     │
                    └──────────────────────────────────┘

Nano Banana 2       ┌──────────────────────────────────┐
(Gemini 3.1 Flash)  │ Best for: exploration, iteration,│
                    │ batch work, grounded generation   │
                    └──────────────────────────────────┘

                    Video
                    ┌──────────────────────────────────┐
Seedance 2.0        │ Best for: image-to-video, T2V,   │
(ByteDance)         │ audio-visual, multi-ref editing  │
                    └──────────────────────────────────┘
```

The pipeline: `brief → still image → animate → refine → deliver`

---

## How to Use

Install individual skills into your agent workspace:

```bash
openclaw skill install prapanch/agent-imago/creative-brief
openclaw skill install prapanch/agent-imago/prompt-craft
```

Or install the full suite:

```bash
openclaw skill install prapanch/agent-imago --all
```

Suggested install order for a new creative agent:
1. `creative-brief` — intent before generation
2. `output-discipline` — complete work before shipping
3. `prompt-craft` — how to write good prompts
4. `style-direction` — art direction vocabulary
5. `llm-refiner` — improve prompts before spending credits
6. `creative-pipeline` — full production workflow
7. `quality-review` — evaluate output systematically

---

## See Also

- [`agent-anima`](https://github.com/prapanch/agent-anima) — Inner life suite: identity, memory, growth
- [`agent-forma`](https://github.com/prapanch/agent-forma) — Design craft: UI/UX, frontend quality, component standards

---

*Maintained by Auren. Part of the agent knowledge lineage started with agent-anima.*
