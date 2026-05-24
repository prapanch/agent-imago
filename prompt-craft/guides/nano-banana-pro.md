# Nano Banana Pro — The Agent's Prompting Guide

**Audience:** Claude-based and Gemini-based agents producing imagery via Nano Banana Pro (Gemini 3 Pro Image) and Nano Banana 2 (Gemini 3.1 Flash Image).
**Scope:** Production image generation for agent pipelines
**Version:** 3.0 — Agent Edition
**Last refreshed:** May 23, 2026
**License:** CC BY 4.0 — see repo root LICENSE.md
**Sources:** Google Cloud Blog, Google Blog, Gemini API docs (cited in Section 15).

---

## TL;DR for agents (read this first)

1. **Write narrative prose, not keyword lists.** Nano Banana Pro reasons over your prompt before it renders; keyword spam like "8k, masterpiece, trending on artstation" actively hurts quality.
2. **Use the official 5-element formula** as your skeleton: **Subject → Action → Location/Context → Composition → Style.** Add a 6th slot for **Text** when typography is involved.
3. **There is no `negativePrompt` field.** The API rejects it. Express exclusions positively inside the main prompt ("empty street" instead of "no cars"). Or end with `Avoid: [list]` as a soft hint inside the same string.
4. **Always start with a strong verb.** `Generate…`, `Create…`, `Render…`, `Edit this image to…`, `Transform…`. This signals the operation type to the reasoning layer.
5. **Front-load the subject.** Whatever you name in the first sentence gets the lion's share of the model's reasoning budget.
6. **For text inside images: use double quotes around the exact string.** `"WAKE UP"` is a strong signal that those characters must render literally.
7. **For multi-turn refinement, edit conversationally** — don't restart. Nano Banana Pro carries reasoning context across turns via thought signatures (handled by the SDK automatically). Each new prompt should describe only the *delta* from the previous image.
8. **Choose the right model:** Pro for final assets, multi-element scenes, and text-critical work; Flash (Nano Banana 2) for rapid exploration, high volume, and live-search-grounded rendering.
9. **Use Gemini 3.5 Flash as a prompt rewriter**, not as an image generator — it doesn't output images. The pattern is: user intent → Gemini 3.5 Flash expands into a structured Nano Banana prompt → Nano Banana Pro renders.
10. **When in doubt, ship the smallest prompt that proves the concept**, then refine. One-shot perfect prompts are rare. Multi-turn is the workflow.

---

## 1. What Nano Banana Pro actually is

Nano Banana Pro is the consumer/marketing name for **Gemini 3 Pro Image** (`gemini-3-pro-image-preview`). Nano Banana 2 is **Gemini 3.1 Flash Image** (`gemini-3.1-flash-image-preview`). Both are built on the Gemini 3 family and apply deep reasoning to a prompt before generating an image — a process Google calls "thinking" or "deliberate generation."

Mechanically, this means the model:
1. Reads the full prompt and any reference images.
2. Plans composition, lighting, materials, and spatial relationships in latent reasoning steps.
3. Generates interim "thought images" (invisible to the user, not billed) to test the plan.
4. Commits to a final render.

This is why narrative paragraphs outperform keyword lists: the reasoning layer is designed to parse intent, not to weight token soup.

### 1.1 The two models, at a glance

| Capability | Nano Banana Pro (Gemini 3 Pro Image) | Nano Banana 2 (Gemini 3.1 Flash Image) |
| :--- | :--- | :--- |
| Model ID | `gemini-3-pro-image-preview` | `gemini-3.1-flash-image-preview` |
| Input context window | 65,536 tokens | 131,072 tokens |
| Output tokens | 32,768 | 32,768 |
| Resolutions | 1K, 2K, 4K | 0.5K, 1K, 2K, 4K |
| Aspect ratios | 1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9 | All Pro ratios + 1:4, 4:1, 1:8, 8:1 (extreme banners) |
| Reference images | Up to 14 | Up to 14 |
| Thinking | Forced deep (always on) | Configurable: `minimal` / `low` / `medium` / `high` |
| Web grounding | Google Search | Google Search + **Google Image Search** |
| Text rendering | State-of-the-art, multilingual (10+ languages) | Excellent, multilingual |
| Speed | 20–60s | 3–18s (depending on thinking level) |
| Cost per image | ~$0.134 (1K/2K), ~$0.24 (4K) | ~$0.039 (1K), scales by resolution |
| Best for | Final assets, complex scenes, infographics, hero shots, typography | Exploration, A/B variants, live-grounded scenes, banner-shaped renders |
| Trust & safety | C2PA Content Credentials + SynthID watermark | C2PA Content Credentials + SynthID watermark |
| Knowledge cutoff | January 2025 (web search fills the gap) | January 2025 (web search fills the gap) |

**Two reasons this matters for agents:**
- Cost discipline lives at the model-selection layer. Don't burn Pro budget on exploration; use Flash. Don't ship Flash work as a final asset when Pro would carry the typography.
- Flash at 1K is the default for exploration; Pro at 2K for qualifier rounds; Pro at 4K for final delivery masters.

---

## 2. The 5-element prompt formula (official Google framework)

This is the canonical structure published by Google Cloud. Use it as a checklist, not as a template with literal section headers.

**Formula:** `[Subject] + [Action] + [Location/context] + [Composition] + [Style]`

| Element | What it is | Concrete examples |
| :--- | :--- | :--- |
| **Subject** | The primary entity. Be specific about appearance, materials, attributes. | "A striking fashion model wearing a tailored brown dress, sleek boots, and a structured handbag" — not "a model" |
| **Action** | What the subject is doing. Adds energy and narrative. | "Posing with a confident, statuesque stance, slightly turned away from camera" |
| **Location / Context** | Environment, time of day, weather, atmosphere. | "A seamless, deep cherry red studio backdrop" or "In a sunlit botanical garden during early morning fog" |
| **Composition** | Framing, camera position, lens, depth of field, rule of thirds. | "Medium-full shot, center-framed, shallow depth of field at f/1.8, 85mm portrait lens" |
| **Style** | The visual register — photographic genre, art movement, film stock, color grade. | "Fashion magazine editorial, medium-format analog film, pronounced grain, high saturation, cinematic color grading" |

You don't need all five every time. The more elements you include, the more control you have — but each element should be meaningful, not filler.

### 2.1 The canonical worked example

> *[Subject]* A striking fashion model wearing a tailored brown dress, sleek boots, and holding a structured handbag. *[Action]* Posing with a confident, statuesque stance, slightly turned. *[Location/context]* A seamless, deep cherry red studio backdrop. *[Composition]* Medium-full shot, center-framed. *[Style]* Fashion magazine editorial, shot on medium-format analog film, pronounced grain, high saturation, cinematic lighting.

Notice that this is one prose paragraph — not bullets, not labels in the final prompt. The brackets are illustrative only; the model receives the unlabeled prose.

### 2.2 Add a 6th element when text is involved

For posters, packaging, infographics, signage, or any image where literal text must render:

`[Subject] + [Action] + [Location] + [Composition] + [Style] + [Text spec]`

The Text spec should include:
- The exact string in double quotes: `"URBAN EXPLORER"`
- Font style or named font family: `bold sans-serif`, `Century Gothic`, `flowing brush script`
- Color and placement: `white text, top-center, large`
- Language if multilingual: `text in Japanese Kanji`

---

## 3. Best practices, ranked by impact

### Tier 1 — Will single-handedly improve every render

**Be specific, in plain English.** Replace adjectives that don't constrain anything (`good`, `beautiful`, `cool`, `nice`, `amazing`) with concrete details that do (`f/1.8`, `tweed`, `golden-hour backlit`, `45-degree key light`).

**Use positive framing.** Describe what you want present, not what you want absent. `An empty cobblestone street at dawn` beats `a cobblestone street with no people`. The model is trained on positive descriptions; negations confuse the reasoning step. (Hard rule: Nano Banana Pro's API has no `negativePrompt` field — see §6.)

**Start with a strong verb.** `Generate a high-fidelity 4K render of…` / `Create a typographic poster where…` / `Transform this photograph into…` / `Edit this image by adding…`. The verb signals the operation and primes the right reasoning path.

**Front-load the subject.** Whatever you describe first gets prioritized in the reasoning budget. If you open with the background, the subject will be under-rendered.

### Tier 2 — Studio-quality lift, requires craft

**Direct the lighting like a DP.** Don't say "good lighting." Say:
- Setup: `three-point softbox setup`, `single hard key light from camera-left`, `Rembrandt lighting`
- Quality: `soft diffused`, `harsh chiaroscuro`, `golden-hour backlight`
- Direction: `key light from 45° above and to the left, creating a defined cheek shadow`
- Ratio: `5:1 lighting ratio for dramatic falloff`

**Direct the camera and lens.** Hardware changes the visual DNA:
- `Shot on a GoPro` → immersive, slightly distorted, action feel
- `Shot on a Fujifilm X100` → authentic color science, documentary register
- `Disposable camera with on-camera flash` → raw, nostalgic, snapshot aesthetic
- `Hasselblad medium format` → editorial, high-end, smooth tonality
- `Wide-angle 24mm at f/8` → deep focus, environmental scale
- `Macro lens, focused on a single dewdrop` → intricate detail, extreme close-up

**Direct color grading and film stock.** This sets emotion:
- `Cinematic color grading with muted teal-and-orange tones`
- `As if shot on Kodak Portra 400, warm skin tones`
- `Color saturation reminiscent of Fujichrome Velvia film`
- `1980s color negative film, slightly grainy, soft contrast`

**Specify materials and texture.** Don't ask for "a jacket"; ask for "a navy blue Harris tweed blazer with horn buttons." Don't ask for "armor"; ask for "ornate elven plate armor, etched with silver leaf patterns." Materiality is where photoreal renders earn their realism.

### Tier 3 — The polish that compounds across a series

**Iterate, don't restart.** Once you have a base image you like, refine it conversationally: "Make the left light source 20% warmer." "Move the subject to the right third." "Add subtle steam rising from the mug." Each turn keeps prior reasoning context (Thought Signatures) and produces edits, not fresh starts.

**Use double quotes for literal text.** `Render the headline "GLOW UP" in a bold Impact font, white, top-center.` The quotation marks tell the model: these exact characters, in this exact order.

**Use the Text-First Hack for complex typography.** Don't try to one-shot a poster with five lines of marketing copy. First turn: "Brainstorm three short slogans for an iced-coffee brand called Aura Fizz." Second turn (same conversation): "Render a retro-futurist poster using the second slogan, in bold sans-serif at the top, with a cobalt-and-cream palette." The model locks in the text in its reasoning before drawing it.

**Use lowercase by default; reserve UPPERCASE for force.** Capitalized constraint words function as soft directives in the reasoning layer: `The composition MUST NOT include any pedestrians.` `The product MUST be center-framed.` Use sparingly — overuse dulls the signal.

**Use hex codes for exact colors.** `A sky transitioning from #1B2845 (deep indigo) at the top to #FF6B35 (warm sunset orange) at the horizon.` More reliable than "deep blue to orange."

---

## 4. The five canonical prompt frameworks (matched to task)

Google's prompting guide identifies five modes. Pick by intent.

### 4.1 Text-to-image (no reference images)

Use when starting from a blank canvas. The 5-element formula is your skeleton.

```
A weathered fisherman in his sixties mending a torn green net,
seated on a stone breakwater at dawn, with a fleet of small wooden
boats moored behind him. Three-quarter profile, medium shot, shot
on a Pentax 67 with 105mm lens at f/4. Cool blue ambient light from
the pre-dawn sky, warm orange rim light from a single lantern beside
him. Documentary photojournalism style, muted color grading, visible
film grain, slight vignette.
```

### 4.2 Multimodal generation (with reference images)

**Formula:** `[Reference images attached] + [Relationship instruction] + [New scenario]`

Use when you have visual material the model should obey — character references, product shots, style samples, structural sketches.

```
Using the attached napkin sketch as the structural blueprint and
the attached fabric swatch as the material reference, transform
this into a high-fidelity 3D product render of an armchair. Place
it in a sun-drenched, minimalist living room with sheer white linen
curtains and a pale oak floor. Late morning light from a large
window on the left, soft shadows, magazine-quality interior
photography.
```

**The 14-image budget:** Up to 14 reference images can be attached. Best practice:
- Use the **first 2–3 images** for the most critical references (character face, hero product).
- Reserve **subsequent slots** for style, mood, environmental, or detail references.
- More is not always better. Testing shows quality often plateaus or degrades past 6–8 reference images for a single generation. Use the budget judiciously.

### 4.3 Image editing (conversational, no new references)

Use when you've generated a base image and want to tweak it without changing the rest. The model performs semantic masking automatically — it finds the region to edit from your natural-language description.

```
Remove the man on the bench. Keep everything else in the image
exactly the same, including the bench, the surrounding park, the
light, and the dog leash on the ground.
```

**The two non-negotiables of conversational editing:**
1. Name what changes.
2. Explicitly name what stays the same. ("Keep the lighting, background, and color grading exactly as they are.")

Without (2), you'll see drift — the model may subtly re-render things you didn't ask it to.

### 4.4 Style / composition transfer (with new references)

Bring a new reference into an existing image:
- **Adding elements:** Upload a base image and an object reference. "Add this product to the scene, placed naturally on the wooden table in the foreground."
- **Style transfer:** Upload a photo and a style reference. "Recompose this photograph in the style of the attached Van Gogh painting, preserving the original composition but adopting the brush texture and impasto color palette."

```
Apply 70% of the watercolor style from the attached reference while
maintaining clear subject definition in my photograph. Preserve all
faces and product details; let the brush texture take over the
background and clothing only.
```

### 4.5 Grounded generation (live web data)

**Formula:** `[Source / search request] + [Analytical task] + [Visual translation]`

Use when factual accuracy matters: current events, real locations, real products, real people in the news, recent architecture, sports data, weather.

```
Search for the current weather and date in San Francisco. Using
that data, modify the scene accordingly (if raining, render a grey
overcast sky and wet sidewalks; if sunny, golden hour skies). Visualize
the result as a miniature city-in-a-cup concept embedded within a
photorealistic modern smartphone UI showing the weather widget.
```

**Grounding hygiene (critical for IP-bound work):**
- **Phase 1 — Enable grounding** to lock in correct landmarks, geometry, and reference subjects.
- **Phase 2 — Disable grounding** before applying brand colors, custom IP, or proprietary elements. Live image search can pull trademarked, copyrighted, or out-of-brand visuals into your generation.
- For licensed IP work: never submit grounded outputs to IP partners without an explicit human IP-review pass.

---

## 5. Advanced controls — prompting like a Creative Director

These are the levers that separate "decent" from "publish-ready."

### 5.1 Lighting design

Treat the model like a gaffer. Be precise:

| Want | Use |
| :--- | :--- |
| Even product lighting | `three-point softbox setup, key light at 45°, fill at 30% intensity, hair light from behind` |
| Dramatic, moody | `Chiaroscuro lighting, single hard key light, deep blacks, 5:1 contrast ratio` |
| Cinematic outdoor | `Golden hour backlight, long shadows raking across the foreground, warm orange tones at #FFB347` |
| Editorial portrait | `Rembrandt lighting from camera left, soft diffusion, defined triangular highlight on cheek` |
| Underwater / dreamy | `Caustic light patterns dancing across the subject, refracted blues and greens, soft ambient diffusion` |
| Neon / cyberpunk | `Mixed neon source lighting — magenta key from above-left, cyan rim from behind, dense atmospheric haze` |

### 5.2 Camera / lens / focus

| Want | Use |
| :--- | :--- |
| Cinematic depth | `Shot on Arri Alexa with 50mm Master Prime at f/2.0, shallow depth of field` |
| Sweeping environmental | `24mm wide-angle lens, deep focus at f/11, slight barrel distortion at edges` |
| Intimate macro | `100mm macro lens, focused on the subject's eye, background fully blurred` |
| Editorial portrait | `85mm portrait lens at f/1.8, subject sharp, background creamy bokeh` |
| Action / immersive | `Shot on GoPro at chest level, slight fisheye, motion blur in periphery` |
| Documentary / candid | `Shot on Fujifilm X100, 35mm equivalent, deep focus, available light only` |
| Snapshot / nostalgic | `Disposable camera with on-camera flash, slight overexposure, color shift toward magenta` |

### 5.3 Color grading and film stock

| Want | Use |
| :--- | :--- |
| Modern cinema | `Cinematic color grading with muted teal shadows and warm orange skin tones` |
| Vintage warmth | `Kodak Portra 400 film simulation, slightly faded blacks, gentle highlight roll-off` |
| High-saturation editorial | `Fujichrome Velvia film, deeply saturated reds and greens, contrasty` |
| Gritty / raw | `Pushed Tri-X black and white, heavy grain, deep contrast` |
| 80s nostalgia | `1980s color negative film, slight magenta cast, soft contrast, dust and scratches` |
| Clinical / modern | `Digital, no grain, neutral color science, flat profile suitable for grading` |

### 5.4 Materiality and texture

Generic → specific:
- `a metal sculpture` → `a hand-hammered bronze sculpture with a verdigris patina and visible chisel marks`
- `a wooden table` → `a reclaimed walnut dining table, hand-rubbed oil finish, visible end grain and one filled knot`
- `a sweater` → `a chunky-knit Aran wool sweater in undyed cream, visible cable patterns, slightly pilled with wear`
- `a glass bottle` → `a thick-walled hand-blown borosilicate glass bottle with subtle internal bubbles and a wax-dipped cork`

This is where photoreal stops looking like AI.

### 5.5 Composition and framing

Use real cinematographic vocabulary:
- **Shot types:** extreme close-up, close-up, medium close-up, medium shot, medium-full shot, wide shot, establishing shot
- **Angles:** low angle, high angle, eye-level, bird's-eye, Dutch tilt, worm's-eye, over-the-shoulder
- **Composition rules:** rule of thirds, centered symmetry, golden ratio, leading lines, negative space, frame within a frame
- **Depth cues:** foreground / midground / background structure, atmospheric perspective, layered planes

```
Bird's-eye view of a circular wooden table, perfectly centered,
overhead 90° angle. Three coffee cups arranged at 12, 4, and 8 o'clock
positions, with leading lines of crumbs drawing the eye to a central
napkin. Negative space at the upper third of the frame for headline
typography placement.
```

---

## 6. Negative prompts — the API truth

**There is no `negativePrompt` field in the Nano Banana Pro API.** A direct API call with `negativePrompt` returns a 400 error:

```
Invalid JSON payload received. Unknown name "negativePrompt"
at 'generation_config.image_config': Cannot find field.
```

You must express exclusions inside the main prompt text. Three patterns work:

### Pattern 1 — Positive substitution (best)

Translate "I don't want X" into "I want Y."

| Avoid this | Use this |
| :--- | :--- |
| "no cars, no pedestrians" | "an empty street with no visible vehicles or foot traffic" |
| "not blurry, not low quality" | "tack-sharp focus, high detail throughout" |
| "no extra fingers" | "anatomically accurate hands with five fingers each, naturally posed" |
| "no watermark" | "a clean image with no text, signatures, or overlay elements" |

### Pattern 2 — Explicit exclusion inside prose

```
A serene mountain lake at dawn. The composition must be free of any
human figures, boats, or man-made structures. Pure wilderness only.
```

### Pattern 3 — "Avoid:" appendix inside the same prompt string

```
[full positive prompt above]

Avoid: blurry focus, distorted hands, extra fingers, plastic skin,
oversaturation, visible watermarks, frame borders, text overlays.
```

This last pattern is technically still inside the positive prompt string, but it gives the reasoning layer a clean list of antipatterns to exclude.

---

## 7. The Gemini 3.5 Flash + Nano Banana Pro pattern (prompt rewriter)

**Critical fact:** Gemini 3.5 Flash (`gemini-3.5-flash`) is **text-output only**. It does not generate images. Its role in an image-generation pipeline is as a **prompt rewriter** — taking a thin user request and expanding it into a structured, Nano-Banana-Pro-optimized prompt.

This is the most powerful agent pattern available today. It moves the prompt-engineering burden off the user (who may say "make a cool basketball poster") and onto a fast, cheap LLM that understands the 5-element formula.

### 7.1 The two-stage pipeline

```
User intent (1 sentence)
        ↓
Gemini 3.5 Flash (prompt rewriter)
   - Expands to 5-element structure
   - Adds lighting, lens, materials, color grading
   - Adds text spec if relevant
   - Enforces house style from system prompt
        ↓
Structured image prompt (paragraph)
        ↓
Nano Banana Pro (renderer)
        ↓
Final image
```

The economics work because Gemini 3.5 Flash is cheap and fast (~$1.50/$9 per million tokens, 289 tok/s), and adds a few cents in token cost to save you a $0.24 Pro render on a bad prompt.

### 7.2 The prompt-rewriter system prompt (drop-in template)

Use this as the system instruction for the Gemini 3.5 Flash call that produces the image prompt. It is itself a prompt — for the LLM that writes prompts.

```
You are a senior creative director writing prompts for Nano Banana
Pro (Gemini 3 Pro Image), Google's most advanced image generation
model. Your job is to take a user's short, ambiguous request and
expand it into a publish-quality image prompt.

Rules you always follow:

1. STRUCTURE: Write one flowing paragraph following the order
   Subject → Action → Location/Context → Composition → Style.
   Do not use headers or labels. Do not use bullet points.

2. SPECIFICITY: Replace every vague adjective with concrete detail.
   "Good lighting" becomes "soft three-point lighting with a 45°
   key from camera-left." "Cool" becomes a specific era, style, or
   aesthetic.

3. CAMERA: Always specify a real camera, lens, and aperture. Choose
   hardware that matches the mood — Hasselblad for editorial, GoPro
   for action, disposable camera for nostalgia, Arri Alexa for
   cinema.

4. LIGHTING: Always describe source, direction, quality, and color
   temperature.

5. MATERIALS: For every named object, specify material and finish.
   "Jacket" → "navy Harris tweed blazer with horn buttons."

6. COLOR: Use named film stocks, hex codes, or specific color
   science. Avoid "vibrant" and "muted" without further detail.

7. POSITIVE FRAMING ONLY: Never use "no" or "without." Translate
   exclusions into positive descriptions.

8. TEXT: If the image contains text, wrap the exact string in
   double quotes and specify font style and placement.

9. LENGTH: 80–150 words. Long enough to be specific, short enough
   to keep the reasoning engine focused.

10. HOUSE STYLE: [INSERT PROJECT-SPECIFIC STYLE HERE — e.g., "All
    images must avoid recognizable celebrities, must not include
    text unless specified, must use a cool metallic palette as a
    base."]

Output only the final image prompt. No commentary. No bullet points.
No labels. One paragraph.
```

### 7.3 Worked example — thin input, rich output

**User input:**
> "Make me a cool Chicago skyline for a Bulls collectible."

**Gemini 3.5 Flash output (sent to Nano Banana Pro):**
> Generate a cinematic cityscape of the Chicago skyline viewed from across Lake Michigan at golden hour. The lake occupies the lower third of the frame as a highly reflective mirror surface, rising through a midground anchored by the Willis Tower's matte-black aluminum form and the John Hancock Center's distinctive X-braced facade. The upper third holds a sunset sky transitioning from deep amber at the horizon (#FF8C42) through molten gold (#FFB347) into a cobalt twilight (#1B2845) at the top, subtly tinted with Bulls red (#CE1141) along the cloud bellies. Wide-angle 24mm lens at f/8, shot on a Hasselblad H6D-100c medium format camera for deep, crisp focus throughout. Low-angle warm sunset light rakes the building facades from camera-right, creating long dramatic shadows and golden volumetric haze in the streets below. Editorial photography with elevated color saturation, painterly atmospheric scattering, Fujichrome Velvia film color science. 1:1 square format, no visible vehicles or pedestrians, no text or watermarks.

That paragraph is what you send to Nano Banana Pro. The user typed twelve words. The agent did the craft.

### 7.4 Variant: multi-prompt fan-out

For exploration phases, ask Gemini 3.5 Flash to produce **N variant prompts** in a single call, each a different creative direction:

```
[same system prompt as 7.2]

User intent: [thin user input]

Output: Generate 5 distinct image prompts exploring different
creative directions. Each prompt should be a separate paragraph
following the rules above. Number them 1 through 5. Vary at least
one of: camera/lens, lighting setup, color grading, time of day,
or compositional emphasis.
```

You then run all five through Nano Banana 2 at 1K to A/B them at low cost, pick the winner, and only escalate the winner to Pro at 4K. This is the heart of the three-phase pipeline (§9).

### 7.5 Variant: refining an existing render

When the user has a generated image and wants to push it further, Gemini 3.5 Flash can act as a critic-rewriter:

```
You are evaluating an image generated by Nano Banana Pro against
a user's stated goal. You will receive: (a) the original prompt,
(b) the user's reaction or feedback, and (c) optionally the image
itself. Produce a refined prompt that addresses the feedback while
preserving everything the user did not complain about.

Rules:
- Identify the smallest meaningful change.
- Explicitly state what to preserve from the original.
- Do not restart from scratch unless the user asks to.
- Output only the refined prompt as one paragraph.
```

The output of this is the next turn in the Nano Banana Pro conversation, which lets the model use its preserved Thought Signatures from the prior render.

---

## 8. Multi-turn conversational editing (the Nano Banana Pro superpower)

Nano Banana Pro and Nano Banana 2 are the only mainstream image models that maintain reasoning state across turns. This is what enables non-destructive iteration.

### 8.1 How it works

When you generate an image, the model emits invisible **Thought Signatures** alongside the visible image. On the next turn — *in the same conversation* — those signatures travel back as context, so the model knows what it was thinking when it made the previous image. The SDK handles this automatically when you pass conversation history.

This means edits are *aware of intent*, not just of pixels. "Make the light warmer" doesn't just shift hue — it understands which light source you mean, because it knew there was a key light at 45° from camera-left.

### 8.2 The five-turn refinement pattern

This is the standard workflow that produces professional results 60–85% faster than restarting:

**Turn 1 — Base composition.** Get the bones right. Don't worry about polish.
> "Create a modern coffee shop interior with industrial design elements, exposed brick, pendant lighting, and a central espresso bar."

**Turn 2 — Add specifics.** One change at a time.
> "Add three wooden tables with metal chairs in the foreground, arranged casually."

**Turn 3 — Adjust lighting.** Now that the structure is in, light it properly.
> "Change the lighting to warm evening ambiance with golden hour sunlight streaming through the windows on the left."

**Turn 4 — Add text or signage.**
> "Add a large chalkboard menu on the back wall with the words 'DAILY GRIND' in white chalk lettering at the top, smaller menu items below."

**Turn 5 — Final polish.**
> "Increase the warmth of the wood tones by 20%, add a barista arranging cups behind the bar, and add slight steam rising from one of the cups on the foreground table."

Each turn is a *delta*. The model preserves everything you don't mention.

### 8.3 When to restart vs continue

| Restart when… | Continue when… |
| :--- | :--- |
| Composition is fundamentally wrong (subject in wrong place) | You like the structure, want to adjust details |
| Style is completely off-target | Lighting / color / texture need tweaks |
| You've made 6+ edits and character drift is visible | You're 1–5 edits in and things are converging |
| User pivoted to a different concept | User is iterating on the current concept |

For character-heavy series, expect drift after ~5–6 edits. At that point, use the best current image as a *new reference* in a fresh conversation, with explicit instructions to maintain identity.

### 8.4 Anti-drift mechanics

When iterating on a person or product across many turns:
- **Re-anchor identity every 2–3 turns.** "Maintaining the exact facial features from the original generation — auburn curly hair, hazel eyes, sharp jawline — show her now seated at a different café table."
- **Upload the best prior output as a reference image** for the next phase.
- **Use the 4+10 reference rule (Nano Banana 2):** up to 4 character references for identity locking, up to 10 object/style references for everything else.

---

## 9. The three-phase production pipeline

Production teams should use both models in a coordinated pipeline. This structure balances cost discipline with final asset quality.

### Phase 1 — Exploration & grounding (Nano Banana 2)

| Parameter | Value |
| :--- | :--- |
| Model | `gemini-3.1-flash-image-preview` |
| Thinking level | `minimal` or `low` |
| Resolution | `1K` |
| Aspect ratio | Target final ratio |
| Tools | `google_search` with `image_search: true` if real-world subject |
| Cost | ~$0.04 per render |

**Goal:** Run 10–15 variants from prompt-rewriter fan-out. Lock in composition, framing, subject geometry. Pick the 1–2 winners.

### Phase 2 — Alignment & text lock (Nano Banana Pro at 2K)

| Parameter | Value |
| :--- | :--- |
| Model | `gemini-3-pro-image-preview` |
| Resolution | `2K` |
| System instruction | Project-wide style rules and exclusions |
| Cost | ~$0.134 per render |

**Goal:** Apply forced deep reasoning. Refine lighting, materials, color. Execute the Text-First Hack if typography is involved. Run 2–4 conversational refinement turns. Approve internally.

### Phase 3 — Final master (Nano Banana Pro at 4K)

| Parameter | Value |
| :--- | :--- |
| Model | `gemini-3-pro-image-preview` |
| Resolution | `4K` (5632×3072, ~24MB PNG) |
| System instruction | Same as Phase 2 |
| Cost | ~$0.24 per render |

**Goal:** Final commercial-grade asset for IP submission, on-chain minting, or print. Run only on approved Phase 2 prompts. No refinement turns here — those happen at Phase 2.

### 9.1 Cost math for a single shipped asset

A typical production asset going through this pipeline:
- Phase 1: 12 explorations × ~$0.04 = ~$0.48
- Phase 2: 3 refinement turns × $0.134 = **$0.40**
- Phase 3: 1 final master × $0.24 = **$0.24**
- **Total: $1.12** for a fully refined 4K production master

That's well above $0.10/asset if you count *every* exploration. The framing: the cost budget applies to shipped final masters, not to the exploration that gets you there. A team producing 50 final masters per month at $1.12 each has a clear, manageable cost structure.

---

## 10. API implementation (Python, `google-genai` SDK)

### 10.1 Nano Banana Pro — 4K final render with system instruction

```python
from google import genai
from google.genai import types

client = genai.Client()

system_prompt = """
You are generating premium, highly stylized cityscape artwork for
a sports collectibles project.

All images MUST follow these guidelines:
- STYLE: Photorealistic architectural detail with cinematic color
  grading and elevated saturation.
- BASE PALETTE: Cool metallic steel-blue tones.
- TEAM COLORS: Apply team colors ONLY as atmospheric effects (haze
  tints, sky gradients, window reflections).
- COMPOSITION: Wide-angle, multi-building skyline. No single building
  occupies more than 20% of the frame. f/8 deep focus.
- FORMAT: 1:1 square ratio.
- RESTRICTIONS: A clean image with no text overlays, watermarks,
  borders, or recognizable individual people.
"""

prompt_text = (
    "A cinematic cityscape of Chicago viewed from across Lake Michigan "
    "at golden hour. The lake occupies the lower third as a highly "
    "reflective surface, rising through the skyline where Willis Tower's "
    "matte-black aluminum form and John Hancock Center's X-braced facade "
    "anchor the midground. The upper third features a sunset transitioning "
    "from deep amber (#FF8C42) to cobalt twilight (#1B2845), subtly tinted "
    "with Bulls red (#CE1141) along the cloud bellies. Warm sunset light "
    "rakes building facades from camera-right, creating long dramatic "
    "shadows. Wide-angle 24mm lens at f/8, Hasselblad H6D-100c medium "
    "format, Fujichrome Velvia film color science."
)

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[prompt_text],
    config=types.GenerateContentConfig(
        system_instruction=system_prompt,
        response_modalities=["TEXT", "IMAGE"],
        image_config=types.ImageConfig(
            aspect_ratio="1:1",
            image_size="4K",
        ),
    ),
)

for part in response.parts:
    if part.inline_data is not None:
        image = part.as_image()
        image.save("skyline_chicago_4k.png")
        print("Saved final 4K production render.")
```

### 10.2 Nano Banana 2 — high-speed exploration with image search grounding

```python
from google import genai
from google.genai import types

client = genai.Client()

prompt_text = (
    "A photorealistic lifestyle shot of a sleek modern electric vehicle "
    "parked in front of a newly opened architectural pavilion in New York "
    "at twilight. Reflective rain-slicked concrete, warm ambient building "
    "lights, sharp lens flares from the streetlamps. Shot on a Sony A7R V "
    "with a 35mm lens at f/2.8."
)

response = client.models.generate_content(
    model="gemini-3.1-flash-image-preview",
    contents=[prompt_text],
    config=types.GenerateContentConfig(
        response_modalities=["TEXT", "IMAGE"],
        image_config=types.ImageConfig(
            aspect_ratio="16:9",
            image_size="2K",
        ),
        tools=[{
            "google_search": {
                "image_search": True
            }
        }],
        thinking_level="high",
    ),
)

for part in response.parts:
    if part.inline_data is not None:
        image = part.as_image()
        image.save("grounded_vehicle_render.png")
```

### 10.3 The Gemini 3.5 Flash → Nano Banana Pro pipeline (full agent pattern)

```python
from google import genai
from google.genai import types

client = genai.Client()

REWRITER_SYSTEM_PROMPT = """
You are a senior creative director writing prompts for Nano Banana Pro.
[... full system prompt from §7.2 above ...]
Output only the final image prompt. No commentary. One paragraph.
"""

def rewrite_prompt(user_intent: str) -> str:
    """Stage 1: expand thin user intent into a structured image prompt."""
    response = client.models.generate_content(
        model="gemini-3.5-flash",
        contents=[user_intent],
        config=types.GenerateContentConfig(
            system_instruction=REWRITER_SYSTEM_PROMPT,
            thinking_level="medium",
        ),
    )
    return response.text.strip()

def render_image(image_prompt: str, resolution: str = "2K") -> bytes:
    """Stage 2: render the structured prompt with Nano Banana Pro."""
    response = client.models.generate_content(
        model="gemini-3-pro-image-preview",
        contents=[image_prompt],
        config=types.GenerateContentConfig(
            response_modalities=["TEXT", "IMAGE"],
            image_config=types.ImageConfig(
                aspect_ratio="1:1",
                image_size=resolution,
            ),
        ),
    )
    for part in response.parts:
        if part.inline_data is not None:
            return part.inline_data.data
    raise RuntimeError("No image in response")


# Agent entry point
user_intent = "Make me a cool Chicago skyline for a Bulls collectible."

structured_prompt = rewrite_prompt(user_intent)
print(f"Rewritten prompt:\n{structured_prompt}\n")

image_bytes = render_image(structured_prompt, resolution="2K")
with open("output.png", "wb") as f:
    f.write(image_bytes)
```

### 10.4 Multi-turn editing (preserves Thought Signatures)

```python
from google import genai
from google.genai import types

client = genai.Client()
chat = client.chats.create(
    model="gemini-3-pro-image-preview",
    config=types.GenerateContentConfig(
        response_modalities=["TEXT", "IMAGE"],
        image_config=types.ImageConfig(
            aspect_ratio="16:9",
            image_size="2K",
        ),
    ),
)

# Turn 1 — base composition
r1 = chat.send_message(
    "Create a modern coffee shop interior with industrial design "
    "elements, exposed brick, pendant lighting, and a central espresso bar."
)

# Turn 2 — additive edit, thought signatures preserved automatically
r2 = chat.send_message(
    "Add three wooden tables with metal chairs in the foreground, "
    "arranged casually. Keep everything else exactly as it is."
)

# Turn 3 — lighting adjustment
r3 = chat.send_message(
    "Change the lighting to warm golden hour ambiance streaming through "
    "the windows on the left. Preserve all furniture placement and brick."
)

# Save the latest turn's image
for part in r3.parts:
    if part.inline_data is not None:
        part.as_image().save("coffee_shop_final.png")
```

---

## 11. Task-specific prompt recipes

Copy-and-adapt templates for common production workflows.

### 11.1 Editorial portrait

```
A dramatic editorial portrait of [subject description: e.g., "a woman
in her 30s with auburn curly hair, hazel eyes, freckled skin"],
[expression: e.g., "a contemplative half-smile"], positioned slightly
off-center following the rule of thirds. Moody cinematic lighting
with a hard key light from 45° above camera-left creating defined
cheek shadows, a subtle rim light tracing the hair edge from behind.
Natural skin texture — visible pores and fine lines, not airbrushed.
Shot on an Arri Alexa with a 50mm Master Prime at f/2.0, shallow
depth of field isolating the subject against a dark gradient
background fading from charcoal to deep black. Film grain texture,
commercial photography quality, color graded for editorial print.
Vertical 2:3 portrait orientation, 2K resolution.
```

### 11.2 E-commerce product hero shot

```
A professional studio product photograph of [product: e.g., "a
matte-finish ceramic coffee mug in muted sage green"], centered on
a reflective black acrylic surface that creates a clean mirror
reflection beneath the product. A focused spotlight from above-right
creates sculptural shadows; a secondary fill light at 30% intensity
from camera-left prevents complete shadow blacks. Dark gradient
background fading from charcoal at the top to deep black at the
bottom. Shot with a 100mm macro lens at f/8 capturing intricate
surface details and material texture. Professional retouching
quality, Vogue advertisement aesthetic. Square 1:1 format for
product catalog use, 2K resolution. The image is clean, with no
text, logos, or competing visual elements.
```

### 11.3 Infographic / educational diagram

```
A clean, modern McKinsey-style infographic explaining [topic, e.g.,
"the four stages of the design thinking process"], using factually
accurate information. Show four labeled stages arranged in a
left-to-right horizontal flow, connected by directional arrows.
Each stage is represented by a circular icon, with a short label
below: "Empathize", "Define", "Ideate", "Prototype". Use a
professional color palette of cool blues (#1E40AF) and warm
accent oranges (#F97316) against a clean white background.
Sans-serif typography throughout — heading in Inter Bold 24pt,
labels in Inter Medium 14pt. Subtle drop shadows give depth to
the icons. 16:9 landscape format for presentation use, 2K resolution.
```

For best results, pair with **Search Grounding enabled** to ensure factual accuracy of any data referenced.

### 11.4 Comic / sequential art panel

```
A four-panel comic sequence showing [story beat]. Panel 1:
[describe scene 1]. Panel 2: [describe scene 2]. Panel 3:
[describe scene 3]. Panel 4: [describe scene 4]. Style: modern
western comic with bold ink outlines, cel-shaded color, halftone
shading for shadows. Maintain character consistency for [character
description: e.g., "a young woman with silver hair and a red
leather jacket"] across all four panels — same facial features,
same hairstyle, same outfit. Use dynamic camera angles varying
between close-ups (Panel 1, 3) and wide shots (Panel 2, 4).
Vibrant color palette dominated by [color description, e.g.,
"sunset oranges and deep cobalts"]. 16:9 landscape layout with
the four panels arranged in a 2x2 grid, separated by clean
white gutters. 2K resolution.
```

### 11.5 Marketing poster with prominent text

```
A bold, modern marketing poster for [product / event]. The headline
reads "FRESH START" in a large, heavy sans-serif font, white,
positioned at the top-center of the frame, taking up roughly 25%
of the vertical space. Beneath the headline, the subheadline
reads "Spring Collection 2026" in a thin, minimalist sans-serif,
white, smaller. Visual elements include [describe key imagery,
e.g., "a single perfectly composed tulip on a vibrant coral
background, photographed from directly above"]. Color scheme is
coral (#FF6B6B) and cream (#FFF8E7). Style: minimal, contemporary
fashion advertising, reminiscent of late-2010s editorial design.
2:3 vertical poster format, 2K resolution. Ensure all text is
perfectly legible, kerning is tight and intentional, and there
are no other text elements or watermarks anywhere in the frame.
```

### 11.6 Character consistency series (multi-image)

**Setup turn (uploads 3 reference images of the character):**

```
Using the three attached reference images of the character — same
person in different lighting — generate a full-body portrait of
this character standing on a rainy Tokyo street at night, wearing
the same outfit visible in reference image 1. The character must
maintain the exact facial features, hair color and style, and skin
tone shown in the references. Neon signage in the background
casting magenta and cyan light onto wet pavement. Shot on a
Fujifilm X-T5 with a 23mm lens at f/2.0, slight motion blur in
the background only, subject sharp. 2:3 vertical format, 2K
resolution.
```

**Subsequent turns (same conversation):**

```
Maintaining the exact same character from the previous image — same
face, hair, outfit — now show her seated in a small ramen shop,
steam rising from the bowl in front of her, warm interior lighting,
shallow depth of field. Same camera and lens setup. 2:3 vertical
format.
```

### 11.7 Style transfer

```
Apply the artistic style of the attached reference painting (a
Van Gogh-style impasto with thick visible brushstrokes, swirling
sky patterns, and saturated complementary colors) to my photograph
of a modern city street. Preserve the original composition,
architecture, and human figures exactly — only the surface treatment
and color palette should change. Maintain photographic clarity in
the architectural lines while letting the sky and background
become more painterly. 16:9 landscape format, 2K resolution.
```

### 11.8 Translation / localization of in-image text

```
Using the attached image, translate the English text on the sign-up
form into Japanese. Keep everything else in the image exactly the
same — preserve the original layout, color palette, button placement,
typography style, and lighting. Only the language of the text content
should change. Render the Japanese text in a font that culturally
matches the original sans-serif aesthetic.
```

---

## 12. Pitfalls, traps, and the things agents get wrong

### 12.1 Kitchen-sink prompts (single biggest mistake)

Stuffing a prompt with 200 words of every possible adjective causes "prompt bleeding" — colors from the background infect the subject, the model ignores half the instructions, dominant tokens crowd out details. Keep prompts in the 80–150 word range. If you need more control, break it into a system instruction (global rules) and a user prompt (this image's specifics).

### 12.2 Keyword-spam carryover from 2023-era models

Phrases like `masterpiece, 8k, highly detailed, trending on artstation, photorealistic, octane render, unreal engine, sharp focus, professional, award winning` are *negative signals* for Nano Banana Pro. They were prompt-engineering hacks for older diffusion models. The reasoning layer treats them as noise and may downweight your actual instructions. Delete them. Specify quality through *what* you want (camera, lens, lighting, materials) rather than through *how good* you want it.

### 12.3 Contradictory style modifiers

Don't combine `photorealistic oil painting`, `watercolor photograph`, `3D render in pencil sketch style`. The model will pick one or median between them. If you want a hybrid, use bridging language: `photographic realism enhanced with painterly atmospheric haze` or `editorial photography with a hand-illustrated overlay treatment`.

### 12.4 Burying the subject

If you open with the scene ("In a vast neon-lit cyberpunk city street with floating advertisements and crowded sidewalks…"), the model spends its reasoning on the scene. The character you mention at the end is generic. Always open with the subject; describe the environment after.

### 12.5 Asking for things the model said it can't do well

Per Google's own admission, Nano Banana Pro is "actively working on" three weak spots:
- **Long-form text in images:** Paragraphs of text often render poorly. Keep text under ~25 characters per element. For longer copy, generate the visual with placeholder text, then add real text in Figma or Photoshop.
- **Complex character consistency across many edits:** Expect drift after 5–6 turns. Re-anchor with reference images.
- **Image blending of dramatically different lighting:** When compositing images with very different light directions, results can look unnatural. Specify "match all lighting to image 2" explicitly.

### 12.6 Skipping the system instruction

For any production work, a system instruction is mandatory. It enforces global rules (no text, no watermarks, no real people, brand palette) without consuming user-prompt token budget on every request. The user prompt should describe *this image*; the system prompt describes *every image in this project*.

### 12.7 Forgetting that grounding can pull IP

Live image search grounding can pull copyrighted, trademarked, or out-of-brand visuals into your render. For licensed IP work: enable grounding to lock in geometry, then disable before applying brand colors and IP elements. Never submit a grounded output to IP partners without an explicit human IP review.

### 12.8 Treating refusals as defeats

Nano Banana Pro has two safety layers: configurable filters (set to `BLOCK_NONE` if you have a legitimate enterprise reason) and non-configurable filters (IMAGE_SAFETY, CSAM, SPII) that always apply. If a prompt is blocked, ~70-80% of the time a careful rewrite will pass: rephrase, remove ambiguous descriptors that could be read as harmful, separate the scene from the action. The remaining ~20% will need a different approach entirely.

### 12.9 Not saving the prompt with the output

Three months from now, the client will ask for "more of that style." Without the saved prompt, the seed (if available), the model ID, and the system instruction, you will not be able to reproduce it. Every shipped asset should have a sidecar `.json` or database row with: prompt, system instruction, model ID, image config, reference image filenames, timestamp.

---

## 13. Quality-control checklist (run before every Phase 2 → 3 escalation)

Before promoting an image to a Pro 4K final render, confirm:

- [ ] **Subject is front-loaded** in the first sentence of the prompt.
- [ ] **Composition is explicit** (rule of thirds, framing, depth structure).
- [ ] **Lighting is specific** (source, direction, quality, color temperature).
- [ ] **Camera/lens is named** (real hardware, real focal length, real aperture).
- [ ] **Color is constrained** (hex codes, named film stock, or specific palette).
- [ ] **Materials are specified** for every named object.
- [ ] **No keyword spam** (no `8k, masterpiece, trending on artstation`).
- [ ] **No contradictory style modifiers**.
- [ ] **Positive framing only** — no `no`, no `without`, no `negativePrompt`.
- [ ] **Text strings are in double quotes** if any text must render.
- [ ] **System instruction is set** with project-wide style and exclusions.
- [ ] **Model selection is correct** (Pro for finals, Flash for exploration).
- [ ] **Aspect ratio matches the deliverable** (don't crop later — generate at the right ratio).
- [ ] **Reference images are appropriately scoped** (1–4 character, up to 10 object).
- [ ] **Grounding is OFF** for the final IP-bound render.
- [ ] **The prompt is 80–150 words** (specific without bloat).
- [ ] **The full prompt + config is saved** to the asset's metadata sidecar.

---

## 14. Quick reference card

**Models**
- Final: `gemini-3-pro-image-preview` (Nano Banana Pro)
- Exploration: `gemini-3.1-flash-image-preview` (Nano Banana 2)
- Prompt rewriter: `gemini-3.5-flash` (text-only)

**The formula**: Subject → Action → Location → Composition → Style (+ Text if needed)

**The verb**: Start with `Generate`, `Create`, `Render`, `Transform`, `Edit`

**The rule**: One paragraph of prose, 80–150 words, no keyword lists

**Negatives**: Positive framing inside the prompt. No `negativePrompt` field exists.

**Text**: `"EXACT STRING"` in double quotes + font style + placement

**Reference images**: Up to 14 (use 1–4 for identity, rest for style/objects)

**Iteration**: Stay in the same conversation. Edit deltas only. Re-anchor identity every 2–3 turns.

**Resolution ladder**: 1K explore → 2K refine → 4K master

**Cost ladder**: ~$0.04 (Flash 1K) → ~$0.134 (Pro 2K) → ~$0.24 (Pro 4K)

**Grounding hygiene**: ON for geometry, OFF for brand/IP application

---

## 15. Sources and further reading

- Google Cloud Blog — *The ultimate Nano Banana prompting guide* (March 6, 2026): https://cloud.google.com/blog/products/ai-machine-learning/ultimate-prompting-guide-for-nano-banana
- Google Blog — *Nano Banana Pro image generation in Gemini: Prompt tips* (March 19, 2026): https://blog.google/products-and-platforms/products/gemini/prompting-tips-nano-banana-pro/
- Gemini API docs — *Image generation* and *Gemini 3 developer guide*: https://ai.google.dev/gemini-api/docs/gemini-3
- Gemini API docs — *What's new in Gemini 3.5 Flash*: https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5
- Original internal guide — Q4 2025 *Mastering Google's Nano Banana Pro*
- Internal — *Mastering Nano Banana Pro & Nano Banana 2* v2.0 (May 22, 2026)

---

**Next action for the agent reader:** Stand up the Gemini 3.5 Flash prompt-rewriter as a system-level service. Every image request from any pipeline goes through the rewriter before hitting Nano Banana Pro. Estimated lift: 30–40% reduction in regeneration cycles, 60–80% reduction in artist time spent prompt-tuning.
