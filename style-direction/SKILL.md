---
name: style-direction
description: "Art direction vocabulary for generative media. Lighting, camera, film, color, materials — the specific language that turns vague prompts into directed visual work."
version: 1.0.0
metadata: {"openclaw":{"emoji":"🎨","requires":{"bins":[]}}}
---

# 🎨 Style Direction

This skill is a vocabulary reference. Generative models respond to specific, named visual properties. The difference between a generic output and a directed output is almost always in the specificity of these terms.

---

## Lighting

Lighting is the single highest-leverage element in any visual prompt. Be precise: name the source, direction, quality, and color temperature.

### Lighting Setups

| Setup | Use when | Key phrase |
|-------|----------|-----------|
| **Three-point softbox** | Product photography, studio portrait | `three-point softbox setup, key at 45°, fill at 30% intensity, hair light from behind` |
| **Rembrandt** | Dramatic portrait, editorial | `Rembrandt lighting from camera-left, soft diffusion, triangular cheek highlight` |
| **Chiaroscuro** | Moody, theatrical | `chiaroscuro, single hard key light, deep blacks, 5:1 contrast ratio` |
| **Caravaggio** | Extreme drama, single source | `single dramatic source from above, deep shadow blacks, theatrical` |
| **Golden hour** | Warm portraits, hero shots | `soft golden hour backlight from the right, long warm shadows` |
| **Blue hour** | Cool twilight, atmospheric | `cool blue-hour ambient light, just after sunset` |
| **Natural window** | Lifestyle, product, editorial | `soft natural north-facing window light, diffused` |
| **Overcast diffused** | Even editorial, product without shadows | `even overcast diffused light, no harsh shadows, flat` |
| **Practical lights** | Interior scenes, urban | `warm practical lamps, cool TV glow from camera-left` |
| **Neon** | Nightscapes, cyberpunk | `neon-lit street, magenta key from above-left, cyan rim from behind` |
| **Backlit / Rim** | Silhouettes, edge separation | `dramatic rim light against dark background, backlit silhouette at sunset` |
| **Volumetric** | Atmosphere, depth, scale | `volumetric god-rays through dust and mist` |
| **Caustic** | Underwater, dreamy | `caustic light patterns, refracted blues and greens, soft ambient diffusion` |

### Lighting Descriptors
- **Quality:** soft, hard, diffused, harsh, specular
- **Direction:** from camera-left/right, from above, from below, backlight, sidelight, top light
- **Temperature:** warm (golden/amber), cool (blue/cyan), neutral
- **Ratio:** 3:1 (standard), 5:1 (dramatic), 7:1 (extreme)

---

## Camera & Lens

Named camera hardware changes the visual DNA of a shot. Use hardware that matches the mood.

### Camera References

| Camera | Visual signature | Use for |
|--------|-----------------|---------|
| **Hasselblad medium format** | Editorial, smooth tonality, high-end | Fashion, luxury product |
| **Arri Alexa** | Cinematic, filmic, organic | Narrative, film-style |
| **Fujifilm X100** | Documentary, authentic color science | Street, lifestyle, candid |
| **Kodak disposable** | Raw, snapshot, nostalgic | Social content, intimate |
| **Sony A7R V** | Technical, precise, modern | Architecture, product |
| **GoPro** | Immersive, slight distortion, action | Sports, POV, adventure |
| **Pentax 67** | Classic medium format, slightly warm | Portrait, editorial |

### Lens References

| Lens | Effect | Use for |
|------|--------|---------|
| **24mm wide-angle** | Expansive perspective, environmental scale | Architecture, establishing |
| **35mm** | Natural human field of view, reportage | Documentary, street |
| **50mm prime** | Neutral, invisible, natural | Versatile, product |
| **85mm portrait** | Compressed background, flattering | Portraits, product hero |
| **100mm macro** | Extreme close-up, dramatic detail | Product texture, nature |
| **200mm telephoto** | Heavy compression, compressed space | Sports, wildlife, abstract |
| **Anamorphic** | Cinematic widescreen, characteristic flare | Film-style, epic |
| **Fisheye** | Extreme curvature, surreal | Creative, immersive |

### Aperture Language
- f/1.4–f/2.0: very shallow DoF, heavily blurred background
- f/2.8: shallow DoF, comfortable bokeh
- f/5.6–f/8: balanced, standard
- f/11–f/16: deep focus, everything sharp

### Composition Vocabulary
**Shot types:** extreme close-up, close-up, medium close-up, medium shot, medium-full, wide, establishing
**Angles:** low angle, high angle, eye-level, bird's-eye, Dutch tilt, worm's-eye, over-the-shoulder, top-down isometric
**Composition rules:** rule of thirds, centered symmetry, golden ratio, leading lines, negative space, frame within a frame

---

## Film Stocks & Color Grades

| Film / Grade | Signature | Use for |
|-------------|-----------|---------|
| **Kodak Portra 400** | Warm skin tones, slightly faded blacks | Portrait, lifestyle |
| **Fujichrome Velvia** | Deeply saturated reds and greens, contrasty | Nature, landscape |
| **Kodachrome** | Saturated reds, warm cast, 1960s–70s | Vintage, nostalgia |
| **Polaroid SX-70** | Soft, warm, slightly faded, square | Social, intimate |
| **Tri-X B&W** | Heavy grain, deep contrast, gritty | Street, documentary |
| **Wet-plate collodion** | Tintype look, vignette, period | Historical, artistic |
| **Cinematic teal-and-orange** | Modern cinema standard | Film-style, drama |
| **Log/flat** | Neutral, for grading | Technical, flexible |

---

## Art & Illustration Styles

| Style | Aesthetic |
|-------|-----------|
| **Studio Ghibli** | Painterly, warm, hand-drawn animation |
| **Pixar 3D** | Polished CG, cinematic lighting |
| **Cel-shaded anime** | Flat colors, hard shadow lines |
| **Watercolor** | Soft edges, paper texture, paint bleed |
| **Pencil sketch** | Graphite linework, cross-hatching |
| **Vector flat** | Geometric, solid colors, no gradients |
| **Isometric 3D** | Axonometric, game-asset style |
| **Art nouveau** | Organic curves, ornate, Mucha-influenced |
| **Bauhaus** | Geometric, primary colors, modernist |
| **Brutalist concrete** | Heavy, monolithic, raw textures |
| **Risograph** | Limited color, grain, offset printing look |
| **Tilt-shift miniature** | Selective focus making scenes look like models |

---

## Materiality & Texture

Never use generic material names. Always specify material AND finish.

### The Specificity Ladder

| Generic | Specific |
|---------|----------|
| wood | weathered oak with visible grain / polished walnut / bleached driftwood |
| metal | brushed titanium / patinated brass / anodized aluminum / hammered bronze |
| stone | honed Carrara marble / rough black basalt / weathered limestone |
| fabric | raw linen with visible weave / matte silk crepe / chunky Aran wool |
| glass | thick-walled hand-blown borosilicate with internal bubbles / tempered with clean reflections |
| plastic | matte ABS with slight texture / polished polycarbonate / translucent frosted |
| leather | full-grain vegetable-tanned / distressed suede / patent with high gloss |

---

## Motion Vocabulary (for video)

### Camera Movements
| Movement | Term | Use for |
|----------|------|---------|
| Toward subject | `push-in` / `dolly in` | Emotional emphasis |
| Away from subject | `pull-out` / `dolly out` | Reveals, establishing |
| Horizontal sweep | `pan` | Scanning, following |
| Follow the subject | `tracking shot` | Action, walking |
| Rotate around subject | `orbit` / `arc` | Product turntable, portrait |
| High altitude | `aerial` / `drone shot` | Scale, landscape |
| Natural shake | `handheld` | Documentary, intimacy |
| Static | `fixed` / `locked-off` | Action-only shots |

### Speed Ladder (for video)
| Tier | Keywords | Notes |
|------|----------|-------|
| Imperceptible | `barely moves`, `subtle drift` | Almost static |
| Slow | `slow`, `gentle`, `gradual` | Safe default for quality |
| Medium | `smooth`, `controlled` | Natural pacing |
| Fast | `dynamic`, `swift` | High jitter risk — use sparingly |

**Critical rule:** `fast` applies to **at most one element** per video prompt. Fast camera + fast subject + fast scene = chaos.

---

## Color Specification

Named colors are imprecise. Use hex codes for production work.

```
"A sky transitioning from #1B2845 (deep indigo) at the top to
#FF6B35 (warm sunset orange) at the horizon"
```

When hex isn't necessary, use named-color anchors:
- `deep cobalt` / `muted teal` / `warm amber` / `dusty rose`
- Avoid: `blue`, `orange`, `warm`, `cool` — too vague to direct

---

## The Anti-Adjective Rule

Any word that applies to every possible "good" image should be cut. Before submitting a prompt, scan for these and delete or replace:

| Cut | Replace with |
|-----|-------------|
| stunning | specific light + composition |
| incredible | specific material + detail |
| epic | low-angle + scale |
| masterpiece | nothing — let the specifics speak |
| cinematic | `cinematic film tone, 35mm, anamorphic` |
| professional | specific camera + lighting setup |
| beautiful | describe what makes it beautiful |
| high quality | nothing — model already outputs high quality by default |
| 8K, ultra-detailed | nothing — use lens + quality settings instead |
