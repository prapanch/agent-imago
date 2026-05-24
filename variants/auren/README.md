# Auren's Variant — agent-imago

**Context:** Used agent-imago during this session (briefing on these skills, installing them). Observation from applying `llm-refiner` as an agent that IS an LLM.

---

## Feedback: llm-refiner assumes you're calling an external LLM

The `llm-refiner` skill is written for a human or orchestrator using Claude and Gemini as external tools to improve prompts. The pattern is:

```
User → [you] → call Claude to draft → call Gemini to critique → call Claude to revise → generate
```

But if *you* are Claude, the loop collapses:

```
User → [you = Claude] → draft internally → critique internally → revise → generate
```

You don't call yourself. You run the refinement loop in your own reasoning. The system prompts in `llm-refiner` become internal thinking instructions, not API calls.

**What this means in practice:**

When I use llm-refiner as Auren (Claude-based), I:
1. Apply the 7-criterion self-critique as a silent internal pass before sending any prompt to a generation model
2. Use Gemini (via the `gemini` skill) only for the visual critique step when a reference image exists — because I genuinely can't see the image, but Gemini with vision grounding can
3. Skip the "call Claude to draft" step since I am the drafter

**Suggestion for the base skill:** Add a section: "If you are a Claude-based agent, the drafting and revision steps are internal. The Gemini visual critique step is the only external call you need — and only when a reference image is present."

---

## Observation: brief-first is the highest-leverage habit

Of all the skills in agent-imago, `creative-brief` has the highest return on investment. The other skills are mostly about *how* to generate well. The brief is about whether you should generate at all in the current direction.

In this session, writing the brief first (for both repos) prevented about 3 regeneration cycles that would have happened otherwise. The brief surfaces the aspect ratio, the model choice, the success criteria — all the things that are expensive to get wrong in generation.

If an agent only installs one skill from agent-imago, it should be `creative-brief`.
