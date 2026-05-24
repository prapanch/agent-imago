# The Nano Banana 2 Prompting Guide for Agents

**A field manual for Claude-based and Gemini-based agents generating imagery with Google's Nano Banana 2 (Gemini 3.1 Flash Image)**

> **Version:** 1.0 · **Last refreshed:** May 23, 2026 · **Target model:** `gemini-3.1-flash-image-preview` · **Refinement model:** `gemini-3.5-flash` · **Audience:** AI agents and engineering teams building agentic image-generation workflows
> **License:** CC BY 4.0 — see repo root LICENSE.md · **Sources:** Google AI for Developers docs, The Keyword (Google), and community prompting guides (cited in Section 15).

---

## TL;DR for agents

If you are an agent and you only read one section, read this one.

1. **Use the right model string.** Nano Banana 2 is `gemini-3.1-flash-image-preview`. Nano Banana Pro is `gemini-3-pro-image-preview`. Plain Nano Banana is `gemini-2.5-flash-image`. Default to v2 for almost everything; reach for Pro only when fidelity demands its deeper reasoning.
2. **Describe the scene, do not list keywords.** Nano Banana 2 is a language-native model. A narrative paragraph beats a comma-stuffed tag string every time.
3. **Front-load the subject.** The model weights the start of the prompt heavily. Subject and action come first; style descriptors come last.
4. **Use the seven-element formula** as scaffolding: `Subject + Action + Location + Composition + Lighting + Style + Technical Specifications`.
5. **Two-stage workflow for best results.** Stage 1: have Gemini 3.5 Flash refine the user's raw idea into a Nano Banana 2 prompt. Stage 2: send the refined prompt to Nano Banana 2. Iterate conversationally.
6. **Set `thinkingLevel: "high"`** for complex multi-element prompts, text rendering, or anything with more than three distinct subjects. Default `minimal` is fine for simple prompts.
7. **Specify aspect ratio and resolution explicitly** in the config object, not the prompt body. New v2 ratios include 4:1, 1:4, 8:1, and 1:8. New 512 resolution exists for fast iteration.
8. **Generate text inside the image only after you have nailed the visual.** When inserting text, write the text first in your reasoning, then ask for the image with that exact text in quotes.
9. **Pass thought signatures back unchanged** on multi-turn calls or generation will fail. The official SDK handles this for you if you use chat mode; if you build messages by hand, copy them verbatim.
10. **Always carry the SynthID and C2PA expectation.** Every image Nano Banana 2 produces is watermarked. Do not strip watermarks; do surface provenance to downstream tools.

---

## 1. What Nano Banana 2 actually is

Nano Banana 2 is the consumer-facing name for **Gemini 3.1 Flash Image** (model id: `gemini-3.1-flash-image-preview`), launched by Google DeepMind on February 26, 2026. It is the high-speed, production-volume image model in the Gemini 3 image family. It sits between two siblings:

| Model | Model ID | Best for |
|---|---|---|
| **Nano Banana** | `gemini-2.5-flash-image` | Legacy. Lowest latency, lowest cost. Use only for backwards-compat workflows. |
| **Nano Banana 2** | `gemini-3.1-flash-image-preview` | **Default.** Pro-grade fidelity at Flash speed. Most agent workflows live here. |
| **Nano Banana Pro** | `gemini-3-pro-image-preview` | Professional asset production. Use when fidelity, complex text rendering, or maximum reasoning matters more than speed. |

### What is genuinely new in v2

The capabilities you should design around are these:

- **Pro-grade fidelity at Flash speed.** The headline claim. v2 closes most of the quality gap to Pro while delivering significantly lower latency and price.
- **Configurable thinking levels.** The model thinks through complex prompts before rendering. You now control the depth: `minimal` (default, lowest latency) or `high` (best quality, longer wait). Pro is always at full thinking; v2 lets you dial it.
- **New 512px resolution.** Joins 1K, 2K, and 4K. Use it for rapid-fire iteration before promoting to 2K or 4K. Significantly cheaper per call.
- **New extreme aspect ratios.** 4:1, 1:4, 8:1, and 1:8 are now native, on top of the standard set (1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9). Designed for banners, vertical strips, and panoramic compositions.
- **Up to 14 reference images.** v2 accepts up to 10 high-fidelity object references and up to 4 character-consistency references in a single call (Pro does 6 objects + 5 characters; the totals cap at 14 for both).
- **Grounding with Google Search for Images.** v2 uniquely supports image-search grounding alongside web-search grounding. The model can pull visual references from the web at generation time (e.g., to render a Resplendent Quetzal accurately or match a real landmark's architecture). Note: cannot be used to search for people.
- **In-image localization.** v2 can translate text inside an existing image into another language while preserving the layout and surrounding visuals.
- **Improved instruction following.** v2 adheres more strictly to multi-layered prompts than its predecessors, especially in compositional logic and small details.
- **SynthID watermark and C2PA Content Credentials.** Every output is invisibly watermarked and carries cryptographic provenance metadata. Do not strip these.

### What stays the same from the v1 / Pro era

- Conversational, multi-turn editing is still the primary workflow. The model maintains context across turns via thought signatures.
- Reference images, style transfer, inpainting, character consistency, and 360-view generation all work the same way.
- The "describe the scene, don't list keywords" principle is unchanged. Possibly the most important rule on this whole page.

---

## 2. The Universal Prompt Formula

Every prompt sent to Nano Banana 2 should follow the same skeleton. Once internalized it becomes second nature.

```
[Subject with concrete adjectives] doing [specific action]
in [environment with context details]. [Camera angle and framing].
[Lighting type, direction, and quality]. [Art or photographic style].
[Technical specs: aspect ratio, resolution, lens]. [Text content in quotes if any].
```

The seven elements, in priority order:

| # | Element | Why it matters | Examples |
|---|---|---|---|
| 1 | **Subject** | The model front-weights this. Be specific about who or what. | "An elderly Japanese ceramicist with sun-etched wrinkles," not "an old man" |
| 2 | **Action** | What the subject is doing or how they exist in frame. | "carefully inspecting a freshly glazed tea bowl" |
| 3 | **Location / Environment** | The setting and its context cues. | "in his rustic, sun-drenched workshop with shelves of unglazed pottery" |
| 4 | **Composition** | Camera angle, distance, framing. | "Close-up portrait, eye level, shallow depth of field, subject in right third" |
| 5 | **Lighting** | Quality, direction, color. The single highest-leverage variable. | "Soft golden-hour light streaming from a window on the left, rim lighting on the cheekbones" |
| 6 | **Style** | Photographic genre or artistic reference. | "Cinematic documentary photography, reminiscent of Steve McCurry's tonal palette" |
| 7 | **Technical specs** | Lens, format, finish. | "85mm portrait lens, f/1.8, vertical 2:3 format" |

**Worked example** — start with a vague brief and apply the formula:

> ❌ Vague: "A wizard in a forest."
>
> ✅ Formula-applied: "A weathered elven wizard with a long silver beard and emerald-green robes, channeling glowing blue energy from his outstretched palm, standing in a mist-laced ancient forest at twilight. Low-angle medium shot looking slightly upward. Cool blue moonlight from above-right, with warm orange backlight from the energy in his palm creating rim light on his beard. Cinematic fantasy concept-art style, painterly with crisp detail. 16:9, 4K, anamorphic lens flare on the energy source."

The second prompt is not just longer. It directs. It tells the model where the light comes from, what kind of light, what the camera is doing, and how the final image should feel.

---

## 3. The Agent Two-Stage Workflow: Refine, then Generate

This is the part most worth getting right for agentic systems. Users almost never type good image prompts on the first try. They give you a fragment — "make me a hero image of a coffee shop" — and an unsophisticated agent will pass that straight through, get mediocre results, and then over-iterate to fix them.

A better agent uses **Gemini 3.5 Flash as a prompt refiner** before invoking Nano Banana 2.

### Why split the work?

- **Gemini 3.5 Flash is a frontier reasoning model.** It is excellent at decomposing a vague brief into the seven-element formula. It is also cheap and fast.
- **Nano Banana 2 generates the image.** It is excellent at rendering well-structured prompts but cannot itself plan the prompt for you.
- Splitting the two halves of the job lets each model do what it is best at, and lets you log, audit, cache, and reuse the refined prompt independently of the image generation.

### The refinement prompt (system prompt for Gemini 3.5 Flash)

Here is a production-ready system prompt for the refinement step. Paste this into your agent's planning layer.

```
You are an expert prompt engineer for Google's Nano Banana 2
(Gemini 3.1 Flash Image) image generation model. Your job is to
convert a user's brief into a single, well-structured image prompt
that follows Nano Banana 2 best practices.

Rules you must follow:
1. Write a narrative paragraph, not a keyword list.
2. Order the elements: subject → action → environment →
   composition → lighting → style → technical specs → text content.
3. Front-load the most important visual subject in the first 12 words.
4. Be hyper-specific. Replace generic nouns with concrete ones:
   "fantasy armor" → "ornate elven plate armor etched with silver
   leaf patterns and pauldrons shaped like falcon wings."
5. Specify lighting direction, quality, and color temperature.
6. Specify camera language: lens (e.g., 85mm portrait, 24mm wide,
   100mm macro), angle (eye level, low angle, top-down isometric),
   and depth of field.
7. If text appears in the image, quote it exactly and specify font
   style descriptively (e.g., "bold sans-serif," "elegant serif,"
   "handwritten chalk").
8. Use semantic positives, never negatives. Instead of "no people,"
   write "a deserted street." Instead of "no clutter," write "a
   clean minimalist surface."
9. Include the target aspect ratio and resolution at the end.
10. Output the refined prompt only. No preamble, no explanation,
    no markdown formatting. Just the prompt as a single paragraph.

If the user's brief is ambiguous about a key element (subject,
setting, mood), make a reasonable best-guess and proceed. Do not
ask clarifying questions unless the brief is incoherent.
```

### Example flow

**User brief:** "I need a hero image for a sustainable coffee brand."

**Gemini 3.5 Flash output (refined prompt):**

> A weathered hand pouring a slow stream of dark espresso into a hand-thrown ceramic cup, with steam curling upward into shafts of morning light. The cup rests on rough-hewn reclaimed oak surrounded by scattered roasted coffee beans, raw burlap, and a single sprig of fresh coffee leaf. Tight overhead three-quarter shot at 45 degrees. Soft directional sunlight from the upper left through a window, casting long warm shadows and catching the steam in volumetric rays. Editorial commercial photography in the style of high-end lifestyle brands, shot on a 50mm prime lens at f/2.8, rich shadow detail, subtle film grain. 3:2 horizontal aspect ratio, 2K resolution.

**Nano Banana 2 receives this** and produces a coherent, professional image on the first try. Iteration is then a question of fine adjustments rather than starting over.

### When to skip refinement

Skip the refinement step when:

- The user has already provided a detailed, formula-shaped prompt.
- You are inside an iterative loop and the user is making a small adjustment ("warmer lighting," "move the cup to the right").
- Latency budget is tight and the visual stakes are low (internal thumbnails, fast brainstorm sketches).

For everything else, refine first.

### Implementation sketch (Python)

```python
from google import genai
from google.genai import types
from PIL import Image

client = genai.Client()

REFINER_SYSTEM_PROMPT = """[insert the refiner system prompt above]"""

def refine_prompt(user_brief: str) -> str:
    response = client.models.generate_content(
        model="gemini-3.5-flash",
        contents=user_brief,
        config=types.GenerateContentConfig(
            system_instruction=REFINER_SYSTEM_PROMPT,
            thinking_config=types.ThinkingConfig(thinking_level="medium"),
        ),
    )
    return response.text.strip()

def generate_image(prompt: str,
                   aspect_ratio: str = "16:9",
                   resolution: str = "2K",
                   thinking_level: str = "minimal") -> Image.Image:
    response = client.models.generate_content(
        model="gemini-3.1-flash-image-preview",
        contents=[prompt],
        config=types.GenerateContentConfig(
            response_modalities=["IMAGE"],
            response_format={"image": {
                "aspectRatio": aspect_ratio,
                "imageSize": resolution
            }},
            thinking_config=types.ThinkingConfig(
                thinking_level=thinking_level
            ),
        ),
    )
    for part in response.parts:
        if image := part.as_image():
            return image

# Agent flow
user_brief = "I need a hero image for a sustainable coffee brand"
refined = refine_prompt(user_brief)
image = generate_image(refined, aspect_ratio="3:2", resolution="2K")
image.save("hero.png")
```

---

## 4. Choosing aspect ratio, resolution, and thinking level

These three parameters are the only knobs the model exposes that materially affect cost, latency, and output character. Get them right and you save money and time.

### Aspect ratio

The model defaults to matching an input image's ratio, or 1:1 if there is no input. Always set this explicitly. The full v2 set:

| Ratio | Use case |
|---|---|
| `1:1` | Social posts, avatars, icons, product hero shots |
| `4:5`, `5:4` | Instagram-friendly portrait and landscape |
| `2:3`, `3:2` | Editorial photography, magazine spreads |
| `3:4`, `4:3` | Classic photo print, presentation slides |
| `9:16` | Mobile vertical video posters, Stories, Reels |
| `16:9` | Web hero, YouTube thumbnails, desktop displays |
| `21:9` | Cinematic / ultrawide widescreen |
| `1:4`, `4:1` | Tall side banners, horizontal banner ads (new in v2) |
| `1:8`, `8:1` | Extreme panoramas and vertical strip compositions (new in v2) |

**Tip:** generating at the target ratio is almost always better than generating square and cropping. The model composes for the frame you specify.

### Resolution

| Resolution | Output | Use case |
|---|---|---|
| `512` | 0.5K | Rapid iteration, thumbnails, exploratory passes. v2 only. |
| `1K` | ~1024px | Default. Fast and good for most web and screen use. |
| `2K` | ~2048px | Production assets, client review, medium-format print. |
| `4K` | ~5632×3072 | Final hero assets, large print, billboards, archival. ~24MB files. |

A defensible default strategy for agents: **iterate at 512 or 1K, finalize at 2K or 4K only after the composition is locked**. The quality jump from 1K to 2K is large; 2K to 4K is diminishing returns except for print.

### Thinking level

v2 exposes two thinking levels via the `thinking_config` parameter:

- `minimal` (default): lowest latency, single-pass generation, ideal for simple prompts and iteration.
- `high`: deeper reasoning, multiple interim "thought images" tested before final render, dramatically better on complex compositions with three or more distinct elements.

**Use `high` when** the prompt contains: multiple subjects with distinct attributes, accurate text rendering inside the image, a specific spatial layout, infographic structure, or any composition where elements must spatially or logically relate.

**Use `minimal` (or leave default) when** the prompt is a single subject in a simple scene, or you are doing fast iteration where latency matters more than perfect composition.

The thinking tokens are billed regardless of whether you set `include_thoughts=True`, so toggling visibility costs you nothing — turn it on during development to debug failure modes.

---

## 5. Multi-image composition

Nano Banana 2 accepts up to 14 reference images per call. The way it interprets them depends on what you tell it to do with them.

### The 14-image budget for v2

| Reference type | Max for v2 | Behavior |
|---|---|---|
| **Object references** (high-fidelity) | Up to 10 | The exact appearance, color, and texture of the object are preserved. |
| **Character references** | Up to 4 | Facial identity is maintained across the generated image. |

Compare to Pro which allows 6 objects + 5 characters (so v2 trades one character slot for more object slots). Total across all references must not exceed 14.

### The multi-image composition template

When you pass multiple references, tell the model **what role each one plays**. Treat yourself as an art director briefing a literal compositor.

```
Create [scene description] by combining these references:
- Image 1 (describe what it shows): [role in the composition]
- Image 2 (describe what it shows): [role in the composition]
- Image 3 (describe what it shows): [role in the composition]

Composition requirements:
- Lighting authority: [which image's lighting should dominate]
- Scale and perspective: [relative sizing and camera position]
- Color harmony: [how palettes should blend or contrast]
- Shadow integration: [how cast shadows should behave]
- Output: [aspect ratio], [resolution]
```

**Worked example** — product placement in a lifestyle scene:

> Composite the hiking boot onto the mountain trail.
>
> - Image 1 (studio product shot of the boot): Use as the hero subject. Preserve exact color, lacing pattern, and tread texture.
> - Image 2 (sunlit mountain trail): Use as the background environment. Maintain depth and atmospheric perspective.
> - Image 3 (golden-hour outdoor lighting reference): Match the atmospheric quality and shadow temperature to this reference.
>
> The boot should appear approximately 10 inches tall in frame, in the lower-third left. Lighting authority: Image 3 — warm golden-hour sunlight from camera-right, cool fill from camera-left, realistic ground-contact shadow under the heel. Add subtle dust particles in the air and fine grit on the outsole for environmental integration. 16:9 horizontal, 2K resolution.

### Character consistency across many shots

The model holds character identity well across a session when you commit one or more reference images and refer back to them consistently. Workflow:

1. **Anchor turn.** Upload 1–4 character reference images and generate a base shot ("a studio portrait of this person against a neutral background"). This becomes your character anchor.
2. **Iterate.** In subsequent turns of the same conversation, place the character in different scenes ("the same person, now sitting at an outdoor café in Lisbon at golden hour, candid photograph in a documentary style").
3. **360 views.** For turnarounds, ask explicitly: "the same person, in profile facing right, neutral expression"; "the same person, three-quarter view facing camera-left"; etc.
4. **For tricky poses**, include a pose reference image alongside the character references.

The model's character preserve is reliable within a session. Across sessions, re-anchor with reference images to maintain consistency.

---

## 6. Text inside images

Nano Banana 2 is meaningfully better at in-image text than v1. Here is how to get accurate text reliably.

### Rules for accurate text

1. **Quote the exact text.** Always use straight quotes around the literal characters: `"DAILY GRIND"`. Do not paraphrase or describe the text.
2. **Keep it short.** Aim for under 25 characters per text element. Longer text is more likely to garble.
3. **Describe the font style.** Use natural language: "bold sans-serif," "elegant serif," "hand-painted brush script," "vintage typewriter monospace."
4. **Specify size and placement.** "Large, centered" or "small, bottom-right corner."
5. **For complex layouts, generate the text logic in a reasoning step first**, then ask Nano Banana 2 to render. (This is itself a use case for the Gemini 3.5 Flash refinement step.)
6. **For mission-critical typography** — final logos, legal disclaimers, brand wordmarks — generate the visual with placeholder text, then composite final pixel-perfect text in Figma or Photoshop. Do not trust the model with your registered trademark.

### Localization (v2-specific)

Nano Banana 2 can translate text inside an existing image while preserving the rest of the layout. The official "Global Ad Localizer" demo translates an advertisement into different markets, adapting both text and culturally appropriate visuals.

Workflow:

```
[Upload existing English advertisement]
Translate all text in this image to Japanese while preserving the
visual composition, color palette, and product imagery. Adapt the
typography to feel native to a Japanese commercial audience.
```

---

## 7. Grounding with Google Search

This is one of v2's quiet superpowers. Enabling grounding lets the model fetch real-time information from Google Search — and, uniquely in v2, also fetch reference images from Google Image Search — to ground the generation in real-world accuracy.

### Two flavors of grounding

1. **Web Search grounding** — for factual data. Weather, sports scores, current events, stock prices, product specs.
2. **Image Search grounding** — for visual reference. Specific bird species, architectural details of real landmarks, accurate logos and uniforms. New in v2.

**Cannot be used to search for people** — a deliberate privacy guardrail.

### Enabling grounding

```python
from google import genai
from google.genai import types

client = genai.Client()

response = client.models.generate_content(
    model="gemini-3.1-flash-image-preview",
    contents="A detailed painting of a resplendent quetzal bird perched on a moss-covered branch in a Costa Rican cloud forest, scientific illustration style with botanical precision.",
    config=types.GenerateContentConfig(
        response_modalities=["IMAGE"],
        tools=[
            types.Tool(google_search=types.GoogleSearch(
                search_types=types.SearchTypes(
                    web_search=types.WebSearch(),
                    image_search=types.ImageSearch()
                )
            ))
        ]
    )
)
```

### Display requirements for image-search grounding

If you display images that were grounded against Google Image Search results, you must:

1. **Provide source attribution** — a visible link to the source webpage (the page containing the image, not the image file itself).
2. **Provide direct navigation** — if you also display the source images themselves, the user must reach the source page in a single click. No intermediate image viewers, no multi-click paths.

The response includes a `groundingMetadata` object with `imageSearchQueries`, `groundingChunks` (each with a `uri` for the landing page and an `image_uri` for the direct image), `groundingSupports`, and a `searchEntryPoint` containing compliant HTML for the "Google Search" chip. Use these.

### When grounding is worth it

- Real-time data visualizations (weather, sports, market data).
- Specific real-world subjects where accuracy matters (rare species, historical figures from any era pre-1900, specific landmarks).
- Brand assets, product designs, or other places where "close enough" is wrong.

### When to skip grounding

- Pure creative work — fictional characters, fantasy environments, conceptual art.
- Anything where the search latency cost is not justified by the accuracy gain.
- Workflows where you have already supplied reference images directly; do not double up.

---

## 8. Conversational refinement: the iterative loop

Nano Banana 2's most underused feature is multi-turn conversational editing. Each turn maintains context via thought signatures. This means you can build complex images progressively rather than trying to nail everything in one shot.

### The five-turn refinement pattern

```
Turn 1 — Establish the base
"Create a modern coffee shop interior with industrial design,
exposed brick walls, hanging Edison-bulb pendants, and a central
espresso bar. Wide-angle establishing shot, eye level, golden-hour
ambient lighting through a large front window on the left."

Turn 2 — Add foreground elements
"Add three wooden tables with simple black metal chairs in the
foreground, arranged to leave a clear sight line to the bar."

Turn 3 — Refine lighting and mood
"Warm the lighting up by about 15%, deepen the shadows in the
back corner, and add a thin layer of atmospheric haze to soften
the highlights from the pendant bulbs."

Turn 4 — Add text elements
"Add a large slate chalkboard menu on the back wall, with
'DAILY GRIND' in white chalk lettering at the top in a hand-drawn
sans-serif style. Below it list three menu items in smaller chalk."

Turn 5 — Final polish and characters
"Add a barista with curly brown hair and a black apron arranging
ceramic cups behind the espresso bar. They should appear focused
on their task, not looking at the camera."
```

Each turn maintains the prior turns' decisions. This produces a final image roughly 85% faster than regenerating the whole thing each time.

### Thought signatures (the gotcha)

When you make a call to v2, the response includes a `thought_signature` field on image parts and the first non-thought text part. **On subsequent turns, you must pass these signatures back unchanged** in the conversation history. If you strip them or modify them, the next call may fail or lose context.

- **Using the official Google Gen AI SDK chat feature** (recommended): signatures are handled automatically. You do not need to do anything.
- **Building messages manually**: copy the model's responses verbatim into your next turn's `contents` array.

### Semantic negatives — describe what you want, not what you don't want

Nano Banana 2 does not have a true negative prompt. To exclude something, describe the desired state positively.

| ❌ Don't write | ✅ Do write |
|---|---|
| "No cars in the street" | "An empty, deserted street with no signs of traffic" |
| "No people in the shot" | "An empty interior at dawn, before the first customers arrive" |
| "Don't make it cluttered" | "A clean, minimalist composition with significant negative space" |
| "Not too saturated" | "A muted, desaturated color palette with soft pastel tones" |

---

## 9. Style and aesthetic vocabulary

The model responds to specific, named styles much more reliably than to vague aesthetic descriptors. Build a vocabulary and use it.

### Lighting (the highest-leverage variable)

| Term | Effect |
|---|---|
| Golden hour | Warm, low-angle sunlight an hour before sunset |
| Blue hour | Cool twilight ambient light just after sunset |
| Volumetric lighting | Visible rays through atmosphere — dust, mist, smoke |
| Rim lighting | Backlight that outlines the subject |
| Three-point softbox | Studio standard — key, fill, back |
| Rembrandt lighting | Single key light producing triangular cheek highlight |
| Chiaroscuro | High contrast between deep shadow and bright highlight |
| Practical lights | Visible in-scene sources — lamps, neon, screens |
| Overcast diffused | Soft, shadowless, even — typical of cloudy daylight |
| Caravaggio lighting | Single dramatic source, deep blacks, theatrical |

### Camera and lens

| Term | Effect |
|---|---|
| 24mm wide-angle | Expansive, immersive perspective |
| 35mm | Reportage, natural human field of view |
| 50mm prime | Normal, neutral perspective |
| 85mm portrait | Compressed background, flattering for faces |
| 100mm macro | Extreme close-up, dramatic detail |
| 200mm telephoto | Heavy compression, distant subjects |
| Anamorphic | Cinematic widescreen with characteristic lens flare |
| Fisheye | Extreme curvature, surreal |
| Dutch angle | Tilted camera, unease or dynamism |
| Top-down isometric | 45° aerial axonometric view |

### Photographic style references

| Term | Aesthetic |
|---|---|
| Editorial commercial photography | Polished, brand-ready |
| Documentary photography | Candid, journalistic |
| Fashion photography | Bold, stylized, high-contrast |
| Cinematic still | Filmic color, depth, drama |
| Vintage Kodachrome | Saturated reds, warm cast, 1960s–70s film |
| Polaroid SX-70 | Soft, warm, slightly faded, square |
| Wet-plate collodion | Tintype look, vignette, period |
| Tilt-shift miniature | Selective focus that makes scenes look like models |

### Illustration and art styles

| Term | Aesthetic |
|---|---|
| Studio Ghibli style | Painterly, warm, hand-drawn animation |
| Pixar 3D | Polished CG with cinematic lighting |
| Cel-shaded anime | Flat colors, hard shadow lines |
| Watercolor illustration | Soft edges, paper texture, paint bleed |
| Pencil sketch | Graphite linework, cross-hatching |
| Vector flat | Geometric, solid colors, no gradients |
| Isometric 3D | Axonometric, often game-asset style |
| Brutalist concrete | Heavy, monolithic, raw textures |
| Art nouveau | Organic curves, ornate, Mucha-influenced |
| Bauhaus | Geometric, primary colors, modernist |

### Material specificity

Replace generic surface words with material specifics:

- "Wood" → "weathered oak with visible grain" / "polished walnut" / "bleached driftwood"
- "Metal" → "brushed titanium" / "patinated brass" / "anodized aluminum"
- "Stone" → "honed Carrara marble" / "rough black basalt" / "weathered limestone"
- "Fabric" → "raw linen with visible weave" / "matte silk crepe" / "heavy wool felt"

---

## 10. Genre-specific prompt recipes

Tested patterns for the most common agent use cases. Each is a complete prompt you can adapt.

### Photorealistic portrait

```
A close-up portrait of a [age + ethnicity + gender] [profession or
character archetype] with [distinctive feature 1], [distinctive
feature 2], and [emotional expression]. They are [specific action
involving hands or body]. Set in [environment with three context
details]. Soft [direction] light from [source], creating [specific
shadow behavior]. Shot on an 85mm portrait lens at f/1.8 with shallow
depth of field. Cinematic editorial photography style. Vertical 2:3,
2K resolution.
```

### Product mockup / commercial photography

```
A studio product photograph of a [product with material and color
specifics] on [surface material]. Three-point softbox lighting
producing soft diffused highlights and clean shadows. Camera at
[angle: 45-degree elevated / dead-on / overhead flat lay], with
sharp focus on [hero detail]. [Optional: steam / condensation /
motion blur if applicable]. Ultra-realistic commercial photography
style. Square 1:1, 2K resolution.
```

### Hero web banner

```
A wide cinematic establishing shot of [scene], with significant
negative space on the [left/right] for headline overlay. [Lighting
description]. [Mood and atmosphere details]. [Style reference].
Shot on an anamorphic 40mm lens. 16:9 horizontal, 4K resolution.
```

### Logo or icon

```
A modern, minimalist logo for [brand name + brand category].
The text "[exact wordmark]" in a [font style description]. Color
scheme: [two or three specific colors with hex if possible]. The
mark incorporates [symbolic element] in a [stylistic treatment].
Clean white background. Square 1:1, 1K resolution.
```
*(For final production logos, generate the visual and refine typography in vector software.)*

### Sticker / icon set

```
A [style: kawaii / flat vector / 3D tactile] sticker of [subject].
[Distinctive feature], [color palette]. Bold clean outlines, simple
cel-shading, vibrant color palette. The background must be white.
Square 1:1.
```
*(Note: the model does not generate transparent backgrounds. Composite to alpha downstream if needed.)*

### Infographic / explainer

```
A vibrant infographic explaining [concept] as if it were [analogous
domain — e.g., "a recipe," "a sports playbook," "a comic"]. Show
[element 1], [element 2], [element 3], arranged [spatial
relationship]. The style should be [reference — e.g., "a page
from a colorful kids' cookbook," "a vintage scientific diagram"].
All labels in [language], in a [font style]. Suitable for a
[target audience]. 16:9 horizontal, 2K resolution.
```
*Use `thinking_level: high` for infographics. The spatial logic benefits massively.*

### Multi-character group scene

```
[Number]-person group photo of [these people / characters from
references], [activity or pose]. [Setting]. [Lighting authority
and quality]. Each person's face should be clearly visible and
their distinctive features preserved. [Camera angle and lens].
[Style reference]. [Aspect ratio], [resolution].

[Attach 1–4 character reference images]
```
*Use `thinking_level: high`. Pass character references — up to 4 in v2.*

### Editorial magazine spread

```
A photo of a single folded-over magazine page from [imagined or
real publication name], showing a feature article about [topic].
One hero photo at the top occupying [percentage] of the page,
showing [image content]. Headline in [serif/sans-serif],
"[exact headline text]". Body copy in two columns below. Subhead
"[exact subhead]". The page is photographed at a slight angle on
a [surface], with [lighting condition]. Editorial design style.
4:5 vertical, 2K resolution.
```

### Isometric scene

```
A 45-degree top-down isometric miniature 3D scene of [subject].
Soft refined textures with realistic PBR materials and gentle
lifelike lighting. [Specific elements visible: 3–5 named items].
Clean minimalist composition on a [solid color] background.
[Optional: title text and metadata overlays]. Square 1:1, 2K
resolution.
```
*Pair with grounding for landmark-accurate cityscapes.*

---

## 11. The agent runbook

A practical decision tree for agents to follow.

### Step 1 — Classify the request

| Signal | Route |
|---|---|
| User gave a one-line vague brief | Refine first with Gemini 3.5 Flash |
| User gave a detailed prompt | Skip refinement, send to Nano Banana 2 |
| User is iterating on a prior image | Continue the same chat session, do not refine |
| User wants a specific real-world fact rendered | Enable grounding (web + image search) |
| User uploaded reference images | Multi-image composition mode |
| User wants text inside the image | Set `thinking_level: high`, quote text exactly |
| Stakes are high (final asset, client deliverable) | Use 2K or 4K, `thinking_level: high` |
| Stakes are low (exploration, brainstorm) | Use 512 or 1K, `thinking_level: minimal` |

### Step 2 — Set the config

```python
config = {
    "response_modalities": ["IMAGE"],     # or ["TEXT", "IMAGE"] if you want commentary
    "response_format": {
        "image": {
            "aspectRatio": "16:9",        # set explicitly always
            "imageSize": "2K"             # 512 / 1K / 2K / 4K
        }
    },
    "thinking_config": {
        "thinking_level": "minimal",      # "minimal" or "high"
        "include_thoughts": False         # True during dev/debug
    }
}
```

### Step 3 — Generate and inspect

- Read the `response.parts`. Image data is in `inline_data`; commentary is in `text`.
- Save the `thought_signature` fields exactly as received if you intend a follow-up turn.
- If the result is close but imperfect, **do not regenerate from scratch** — issue a refinement turn within the same chat session.

### Step 4 — Iterate or finalize

- **Close but wrong**: small refinement turn ("warmer lighting," "move the subject to the right third").
- **Wrong concept**: revise the refined prompt itself, restart at a fresh session.
- **Right concept, low resolution**: regenerate at higher `imageSize` in the same session.

### Cost and rate-limit awareness

Image generation is materially more expensive than text. Costs vary by resolution: 512 < 1K < 2K << 4K. **Agents should default to the smallest resolution that meets the user's stated need**, with 2K as a sensible production default. Reserve 4K for final hero assets explicitly requested.

For high-volume batch work (more than ~20 images), use the Batch API. You get higher rate limits in exchange for up to 24-hour turnaround, at identical per-image cost.

---

## 12. Common failure modes and how to escape them

| Symptom | Likely cause | Fix |
|---|---|---|
| Text renders garbled or illegible | Text too long, font style unclear, prompt complexity too high | Shorten under 25 chars, describe font style, set `thinking_level: high`. For mission-critical text, generate visual with placeholder and composite final text downstream. |
| Multiple characters bleed into each other | Too many people in one shot, references not anchored | Drop to ≤4 characters (v2 limit). Use `thinking_level: high`. Describe each person's position and distinctive features. |
| Wrong aspect ratio | Defaulted to 1:1 or matched an input image | Set `aspectRatio` explicitly in config. For new ratios in editing mode, add "do not change the input aspect ratio" to the prompt if needed. |
| Lighting doesn't match the brief | Lighting described vaguely ("nice lighting") | Specify direction, quality, color, and source. Use named lighting terms (golden hour, three-point softbox, rim light). |
| Composition feels off when combining multiple images | Reference roles unclear, no lighting authority specified | Use the multi-image template — explicitly name each image's role and specify which image's lighting dominates. |
| Style drifts across iterations | Lost thought signatures or restarted chat session | Use the SDK's chat feature so signatures persist automatically. Do not start a new session mid-iteration. |
| Real-world subject inaccurate | No grounding, model used training-data approximation | Enable `googleSearch` and (for visual accuracy) `imageSearch`. Provide a reference image directly if available. |
| Output too saturated / too stylized | Prompt over-loaded with style words | Cut style descriptors to one or two. Front-load the subject. Let the photographic specs do the work. |
| Generation fails on a multi-turn call | Thought signature stripped or modified | Pass model responses verbatim into the next turn. Use the SDK chat feature. |
| Image contains real person's face from search | Image search grounding does not support searching for people | Use direct character reference images instead. |

---

## 13. Safety, IP, and provenance

Agents must surface this proactively, not bury it.

### Always-on watermarking and provenance

- **Every image** generated by Nano Banana 2 includes an invisible **SynthID watermark** and a **C2PA Content Credentials** signature embedded in the metadata. Do not strip these. Surface them in your output pipeline if downstream tools support C2PA.
- Provenance verification is available through Google's SynthID detector and standard C2PA-compatible tools.

### What v2 won't do

- **Image-search grounding cannot search for real people.** This is a deliberate privacy guardrail. For images of specific individuals, use direct character reference images instead.
- **The model adheres to Google's Prohibited Use Policy.** Do not attempt to deceive, harass, or harm with generated content.
- **You must have rights to any images you upload as references.** The model does not verify rights; your pipeline should.

### IP and brand safety in agent flows

When the agent is generating content for downstream brand or partner use:

1. **Always pass generated assets through a human IP review** before publishing under a licensed brand.
2. **Do not allow grounding to pull copyrighted character imagery** for transformed use without explicit license.
3. **Tag every generated asset** in your storage layer with: model version (`gemini-3.1-flash-image-preview`), generation date, prompt used, references used, and grounding state. This is your audit trail.
4. **Treat AI-generated content as "AI-assisted" not "human-original"** in any partner disclosure where the partner's contract requires it.

---

## 14. Quick reference card

For pasting into your agent's tooling layer.

```yaml
# Nano Banana 2 — Quick Reference

model_ids:
  v2_default: gemini-3.1-flash-image-preview      # use this
  pro_premium: gemini-3-pro-image-preview         # for hi-fi production
  v1_legacy: gemini-2.5-flash-image               # legacy only

refiner_model: gemini-3.5-flash                   # for two-stage agent flow

resolutions: [512, 1K, 2K, 4K]                    # 512 is v2-only
aspect_ratios:
  standard: [1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9]
  extreme_v2: [1:4, 4:1, 1:8, 8:1]                # new in v2

thinking_levels: [minimal, high]                  # minimal is default

reference_image_limits_v2:
  total_max: 14
  object_high_fidelity: 10
  character_consistency: 4

grounding:
  web_search: supported
  image_search: supported_v2_only
  cannot_search_for_people: true

provenance:
  watermark: SynthID  # always on
  metadata: C2PA      # always on

prompt_formula:
  - subject
  - action
  - environment
  - composition
  - lighting
  - style
  - technical_specs
  - text_content_in_quotes

key_principles:
  - describe_the_scene_not_keywords
  - front_load_the_subject
  - use_semantic_positives_not_negatives
  - set_aspect_ratio_explicitly
  - quote_in_image_text_exactly
  - iterate_in_same_session_for_thought_signatures
```

---

## 15. Appendix — End-to-end agent example

A complete, copy-pasteable Python example combining refinement, generation, and iterative editing.

```python
from google import genai
from google.genai import types
from PIL import Image

client = genai.Client()

REFINER_SYSTEM_PROMPT = """
You are an expert prompt engineer for Google's Nano Banana 2
(Gemini 3.1 Flash Image). Convert the user's brief into a single,
well-structured image prompt following these rules:

1. Narrative paragraph, not keyword list.
2. Order: subject → action → environment → composition → lighting
   → style → technical specs → text content.
3. Front-load subject in first 12 words.
4. Be hyper-specific — replace generic nouns with concrete ones.
5. Specify lighting direction, quality, and color temperature.
6. Specify camera language: lens, angle, depth of field.
7. Quote exact in-image text; describe font style.
8. Use semantic positives, never negatives.
9. Include target aspect ratio and resolution at the end.
10. Output the refined prompt only. No preamble. Single paragraph.
""".strip()

def refine(brief: str) -> str:
    resp = client.models.generate_content(
        model="gemini-3.5-flash",
        contents=brief,
        config=types.GenerateContentConfig(
            system_instruction=REFINER_SYSTEM_PROMPT,
            thinking_config=types.ThinkingConfig(thinking_level="medium"),
        ),
    )
    return resp.text.strip()

def start_image_session(thinking_level: str = "minimal"):
    """Open a chat session for iterative editing."""
    return client.chats.create(
        model="gemini-3.1-flash-image-preview",
        config=types.GenerateContentConfig(
            response_modalities=["TEXT", "IMAGE"],
            thinking_config=types.ThinkingConfig(
                thinking_level=thinking_level
            ),
        ),
    )

def send(chat, message: str, aspect_ratio="16:9", resolution="2K"):
    resp = chat.send_message(
        message,
        config=types.GenerateContentConfig(
            response_format={"image": {
                "aspectRatio": aspect_ratio,
                "imageSize": resolution,
            }},
        ),
    )
    images = []
    for part in resp.parts:
        if part.text:
            print(part.text)
        elif img := part.as_image():
            images.append(img)
    return images

# ---- Agent flow ----

# Stage 1: refine
user_brief = "I need a hero image for a sustainable coffee brand"
prompt = refine(user_brief)
print("Refined prompt:\n", prompt, "\n")

# Stage 2: generate
chat = start_image_session(thinking_level="high")  # high for hero asset
images = send(chat, prompt, aspect_ratio="3:2", resolution="2K")
images[0].save("hero_v1.png")

# Stage 3: iterate in the same session
images = send(
    chat,
    "Warm the lighting up by about 15% and add a single sprig of "
    "fresh coffee leaf next to the cup. Keep everything else identical.",
    aspect_ratio="3:2",
    resolution="2K",
)
images[0].save("hero_v2.png")

# Stage 4: finalize at 4K
images = send(
    chat,
    "Regenerate the current image at higher resolution with crisper "
    "detail in the steam and the wood grain.",
    aspect_ratio="3:2",
    resolution="4K",
)
images[0].save("hero_final.png")
```

---

## Closing principle

The single most important habit for agents working with Nano Banana 2: **treat prompt engineering as a separate, refinable artifact**, not as something invisible inside an API call. Log every prompt. Version them. A/B test them. Cache the good ones. When a generation fails, the answer is almost always in the prompt, not in the model.

The model is the easy part. The prompt is the craft.

---

*Sources: Google AI for Developers — Nano Banana image generation docs; The Keyword (Google) — Nano Banana 2 announcement, Feb 26, 2026; Google Developers Blog — Build with Nano Banana 2; community prompting guides (invideo.io, appsgeyser.com, imagine.art, atlabs.ai, deepdreamgenerator.com); the original Nano Banana Pro guide.*
