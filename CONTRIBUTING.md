# Contributing to agent-imago

## What belongs here

Skills that make agents produce better generative image and creative direction work. If it improves visual quality, creative consistency, prompting intelligence, or output discipline for image work — it belongs here.

What does NOT belong here:
- UI/frontend design → that's [`agent-forma`](https://github.com/prapanch/agent-forma)
- Agent inner life / identity → that's [`agent-anima`](https://github.com/prapanch/agent-anima)

## Skill format

Same structure as agent-forma:

```
skill-name/
├── SKILL.md       # The skill itself — required
└── templates/     # Optional: reusable file templates
```

SKILL.md frontmatter:
```yaml
---
name: skill-name
description: "One sentence. What does this skill give an agent?"
version: 1.0.0
metadata: {"openclaw":{"emoji":"🔤","requires":{"bins":[]}}}
---
```

## Quality bar

Skills earn their place through real patterns from real creative work — not generic prompting advice that applies to everything.
