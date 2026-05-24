# Mastering GPT Image 2 (ChatGPT Images 2.0): A Prompting Guide for Claude and Gemini Agents

**A comprehensive reference for LLM-driven image generation pipelines**
**Audience:** Claude-based and Gemini-based agents that author, refine, and grade image prompts for GPT Image 2
**Model versions covered:** `gpt-image-2` (released April 21, 2026), with notes on `gpt-image-1.5`, `gpt-image-1`, and `gpt-image-1-mini` for migration contexts
**Last updated:** May 2026
**License:** CC BY 4.0 — see repo root LICENSE.md
**Sources:** See Section 11 for full citations. Primary sources include OpenAI Developer Cookbook, fal.ai prompting guide, and OpenAI release documentation.
**Prepared for:** production teams

---

## 0. How to use this document

This guide is written for **agents**, not for human prompters scrolling past. If you are an LLM (Claude or Gemini) about to author a GPT Image 2 prompt or refine one a user has supplied, read in this order:

1. **Section 1** — Internalize what the model actually is and what its O-series reasoning layer changes about how you prompt it.
2. **Section 2** — Lock in the canonical prompt template. Every prompt you write should map cleanly onto these five slots.
3. **Section 3** — Pick the right *mode*: generate, edit, or compose. Each has different invariants.
4. **Section 4** — Apply the anti-slop rules. These eliminate the most common failure modes.
5. **Section 5** — Handle text in images correctly. This is where GPT Image 2 most outperforms its predecessors *if* you treat text as typography, not language.
6. **Section 6** — Use the use-case patterns (photoreal, product, UI, infographic, ads, comics, character consistency, etc.) as starting scaffolds.
7. **Section 7** — Apply the LLM-as-refiner loop. This is where you, the agent, multiply prompt quality through self-critique.
8. **Section 8** — Reach for the API parameter cheat sheet when shipping.
9. **Section 9** — Read the failure-mode catalog before declaring a prompt "done."

---

## 1. What GPT Image 2 actually is, and why it changes how you prompt

GPT Image 2 (model ID `gpt-image-2`, marketed as ChatGPT Images 2.0) shipped on April 21, 2026 as OpenAI's third-generation native image model, succeeding `gpt-image-1` (March 2025) and `gpt-image-1.5` (December 2025). Three architectural shifts matter for prompting:

**1. O-series reasoning before pixels.**
The model researches, plans, and self-checks before rendering. In practice this means it can hold roughly fifteen-element scenes coherently where earlier diffusion-style models dropped elements past six or seven. It also means your prompt is read as a *brief*, not as a keyword stack. Stuffing adjectives ("8K, masterpiece, ultra-detailed, cinematic, award-winning") actively dilutes the signal — the model reads them as redundant noise around your real instructions.

**2. Near-perfect text rendering.**
~99% character accuracy on Latin scripts and ~95%+ across Chinese, Japanese, Korean, Hindi, Bengali, and Arabic. Mixed-script layouts — a Japanese poster with English product names, an Arabic menu with Western prices — work for the first time in a commercial model. This unlocks single-prompt posters, menus, infographics, UI mockups, and ad creatives that previously required Photoshop after generation.

**3. Surgical multi-turn editing with up to 16 reference images.**
The same endpoint family (`gpt-image-2` for generation, `gpt-image-2/edit` for edits) handles text-to-image, in-place editing, style transfer, virtual try-on, and multi-reference compositing. Edits do not reinterpret the whole scene — they change only what you specify, *if* you tell the model what to preserve.

**What this means for you as a prompting agent:**

| Old model behavior | GPT Image 2 behavior | Implication for your prompt |
|---|---|---|
| Rewards keyword stacking | Rewards structured natural language | Write briefs, not tag lists |
| Hallucinates text | Renders text verbatim if asked properly | Treat text as typography specification |
| Loses elements past 6-7 | Holds 15+ elements coherently | Include every element you want; the model will manage them |
| Reimagines scenes on edit | Surgical, no drift | Use explicit change-vs-preserve language |
| "Style" was vague | "Style" needs visual targets | Describe the visual properties, not the genre name |

**Key model parameters to know before prompting:**

- `outputQuality`: `low` / `medium` / `high`. `low` is genuinely usable for high-volume work; reserve `medium`/`high` for dense text, small detail, identity-sensitive work.
- `size`: any resolution where max edge < 3840px, both edges multiples of 16, ratio ≤ 3:1, total pixels between 655,360 and 8,294,400. Anything above 2560×1440 is experimental.
- `n`: number of variations per call. Use 4 for logo/exploration work; 1 for production renders.
- `input_fidelity`: `high` engages strong identity preservation on edits. **Not** available on `gpt-image-2` (output is already high-fidelity by default); used on `gpt-image-1.5`/`1`.
- `background`: `opaque` / `transparent`. Transparency works on PNG/WebP; JPEG falls back to opaque.
- **Thinking Mode** (ChatGPT product) allows up to 8 coherent images from one prompt with self-review. Add "self-check for text accuracy and layout balance" to engage it most effectively.
- **Max 16 reference images per call** on the edit endpoint.

---

## 2. The canonical prompt template (memorize this)

Every prompt you write should map onto these five slots, in this order, with line breaks between sections once the prompt runs past a short paragraph. This is the structure OpenAI's own cookbook and fal.ai's production guide converged on independently.

```
Scene:
[where this happens — environment, time of day, background, atmosphere]

Subject:
[who or what is the main focus, with concrete attributes]

Important details:
[materials, clothing, texture, lighting direction and quality,
 camera angle, lens behavior, composition, mood, props]

Use case:
[editorial photo / product mockup / poster / UI screen / infographic /
 concept frame / children's book illustration / etc.]

Constraints:
[what must NOT appear — no watermark, no logos, no extra text;
 what must be preserved — face, layout, brand elements;
 what must be exact — text quoted verbatim, exact placement]
```

**Why this works:** Each slot answers a question the model would otherwise have to guess at. The fifth slot — Constraints — is where most mediocre prompts fail silently. Describe the idea without bounding it and the model gets inventive in directions you will regret.

**Filled example (editorial photoreal):**

```
Scene:
A quiet classical museum gallery in soft afternoon light.

Subject:
A woman in her 30s standing casually in front of a large oil painting.

Important details:
Natural smile, realistic skin texture, beige knit sweater, dark jeans,
white sneakers, eye-level full-body framing, marble floor reflections,
warm neutral color balance, shallow depth of field, believable indoor
ambient light.

Use case:
Editorial lifestyle photograph.

Constraints:
No watermark, no logos, no extra people in the foreground,
no heavy retouching.
```

**Filled example (product ad with exact text):**

```
Scene:
Roadside billboard mockup at sunset.

Subject:
A matte black wireless speaker featured against a clean cream background.

Important details:
Bottle on the right third, headline on the left third, generous negative
space, soft directional lighting from camera left, slight contact shadow,
clean product edges, premium campaign photography quality.

Use case:
16:9 campaign banner for an outdoor advertising mockup.

Constraints:
Billboard headline (EXACT TEXT, one line only): "SOUND YOU CAN FEEL"
Typography: bold sans-serif, centered vertically in the left half,
clean kerning, high contrast, readable from a distance.
Render the text verbatim. No extra words. No duplicate text.
No additional logos. No watermark.
```

**Variants of this template that also work** (use whichever is easiest to maintain in your pipeline):
- Single descriptive paragraph (for short prompts only).
- JSON-like structure (good when an agent is constructing prompts programmatically).
- Markdown bullet sections (good for human review).

What matters is that the **intent is clear and constraints are explicit**. Prioritize a skimmable template over clever prompt syntax.

---

## 3. The three modes: generate, edit, compose

Real image work falls into three buckets. Each has its own template and its own failure modes.

### 3.1 Generate from scratch (text → image)

Use the five-slot template above. This covers editorial photos, posters, product scenes, concept art, logos, UI screenshots, illustrations, infographics, diagrams.

**Endpoint:** `gpt-image-2` (text-to-image).

### 3.2 Edit one image (text + image → image)

Use a **two-column logic**: what changes, what stays locked.

```
Change:
[exactly what should change]

Preserve:
[face, identity, pose, lighting, framing, background, geometry,
 text, layout — list every invariant explicitly]

Constraints:
[no extra objects, no redesign, no logo drift, no watermark,
 no color grading changes unless requested]
```

**Endpoint:** `gpt-image-2/edit`.

**The three-sentence pattern for object edits:**

```
Sentence 1: Replace the parked car with a vintage bicycle.
Sentence 2: Preserve the house, fence, driveway concrete, landscaping,
            lighting direction, and time of day exactly.
Sentence 3: Match the bicycle scale and shadow pattern to the existing
            scene.
```

**Critical rule:** Repeat the preserve list on every iteration. Drift compounds across turns; explicit re-anchoring resets it.

### 3.3 Combine multiple images (multi-image → image)

Used for virtual try-on, style transfer, compositing, inserting an object into a scene, or mixing a style reference with a content source. Up to **16 reference images** per call on `gpt-image-2/edit`.

**The labeling rule (critical):** Reference each input by index and describe its role. The model needs to know which image is content and which is reference.

```
Image 1: base scene to preserve.
Image 2: jacket reference.
Image 3: boots reference.

Instruction:
Dress the person from Image 1 using the jacket from Image 2 and the
boots from Image 3. Preserve the face, body shape, pose, background,
lighting, and framing from Image 1. No extra accessories.
```

Labeling each input by role keeps compositing prompts grounded instead of making the model guess which image is content and which is reference.

---

## 4. Anti-slop rules (do these every time)

These six rules eliminate ~80% of bad outputs.

### Rule 1: Visual facts over vague praise

- **Avoid:** stunning, incredible, epic, masterpiece, gorgeous, insane detail, 8K, ultra-detailed, professional, award-winning.
- **Prefer:** overcast daylight, brushed aluminum, chipped paint, clean kerning, 50mm feel, soft bounce light, slightly worn canvas, condensation at the edge.

The model already produces high-fidelity work by default. Adjective stacking signals nothing it doesn't already have, and it crowds out the specific visual facts that *would* change the output.

### Rule 2: Style tags need visual targets

A style name on its own ("brutalist," "editorial," "premium") is too abstract.

| Weak | Usable |
|---|---|
| minimalist brutalist editorial luxury photoreal | Cream background, heavy black condensed sans serif, asymmetrical type block, one hero object, generous negative space, studio tabletop lighting. |
| cyberpunk cinematic moody | Wet pavement reflections, mixed cool street light and warm shop light, 50mm documentary feel, slight film grain. |

### Rule 3: Say the real thing

If the image must show a transit kiosk, say *transit kiosk*. If it must contain a readable boarding pass, say *boarding pass*. If it must preserve a face, say *preserve the face*. Mood language buries the brief.

### Rule 4: In edits, separate change from preserve

Use "change only X" and "keep everything else the same." Repeat the preserve list on every iteration. If the edit must be surgical, also say not to alter saturation, contrast, layout, arrows, labels, camera angle, or surrounding objects.

### Rule 5: Treat text like typography

Wrap literal text in quotes or ALL CAPS. Specify font style, size, color, placement. Spell tricky brand names letter by letter. See Section 5 for the full text-rendering protocol.

### Rule 6: One revision per turn

Small iterative edits read better than one giant rewrite. The model gets confused by simultaneous orthogonal changes.

| Good | Bad |
|---|---|
| Make the light warmer. Remove the extra chair on the left. Restore the original wall texture. Keep everything else the same. | Make it more premium, more realistic, more stylish, more cinematic, more emotional, more modern, fix the text, change the outfit, improve the background, and also keep everything. |

---

## 5. Text in images — the protocol

Text rendering is GPT Image 2's biggest leap and the most common place agents under-prompt it. Treat in-image text as **typography specification**, not as language to be paraphrased.

### 5.1 The five-step text protocol

1. **Wrap the exact copy** in quotation marks or ALL CAPS inside the prompt. Example: `Headline (EXACT TEXT): "Fresh and Clean"`.
2. **Specify placement** — where on the canvas, how the text relates to other elements. Example: "centered vertically in the left half, headline above subhead, generous space below."
3. **Specify typography behavior** — font style (bold sans-serif, condensed slab, hand-lettered), color, contrast, kerning, weight. Example: "bold condensed sans serif, white on black, tight kerning, all caps."
4. **Add a hard stop** — explicitly forbid additions. Example: "Render the text verbatim. No extra characters. No duplicate text. No additional logos."
5. **For tricky words** — uncommon brand names, foreign spellings, made-up product names — spell the word letter-by-letter in the prompt. Example: "the brand name (spelled letter by letter: H-Y-D-R-A) rendered in clean serif."

### 5.2 Quality settings for text

- `quality="low"` is fine for short headlines (1-5 words).
- `quality="medium"` is the production default.
- `quality="high"` is required for: small text, dense information panels, multi-font layouts, mixed-script content, infographics, slides with footnotes, packaging copy.

### 5.3 Mixed-script and multilingual text

Mixed-script layouts (a Japanese poster with Latin product names, an Arabic menu with Western prices, Chinese subtitles over English titles) work reliably in GPT Image 2. The protocol is identical: quote each language's text separately, specify font behavior for each script, place each on the canvas explicitly.

**Example (bilingual poster):**

```
Chinese New Year marketing poster, 3:4 vertical.

Top text (EXACT TEXT, Chinese, traditional brush-style script, gold on red):
"新年快乐"

Below it, smaller English text (EXACT TEXT, modern sans serif, red on cream background):
"Happy Lunar New Year 2026"

Composition: traditional paper-cut elements at the corners, festive red and
gold palette, generous negative space in the center.

Render both lines verbatim. No additional text. No extra characters.
No watermark.
```

### 5.4 When text rendering still fails

GPT Image 2's known weak spots remain:
- **Exact logo reproduction** (proprietary vector marks). Composite logos in Photoshop or Figma after generation.
- **Tiny legal copy** at very small sizes.
- **Proprietary typefaces** the model hasn't seen. Specify family characteristics instead ("a slab serif similar in feel to Roboto Slab").
- **Very long text blocks** (paragraphs of body copy). Split into multiple shorter blocks or generate the layout and composite real text afterward.

---

## 6. Use-case patterns

These are starting scaffolds. Each is a known-good pattern that holds up under production conditions.

### 6.1 Photorealistic / editorial photography

Prompt as if a real photo is being captured in the moment. Use photography language (lens, lighting, framing) and explicitly ask for real texture (pores, wrinkles, fabric wear, imperfections). Include the word "photorealistic" to engage the model's photorealistic mode. Avoid words that imply studio polish or staging unless that's the brief.

```
Scene: A narrow side street in Istanbul just after light rain at blue hour.
Subject: A florist locking up for the night.
Important details: Wet pavement reflections, metal shutter half closed,
  green apron, tired posture, a paper bundle of unsold tulips in one hand,
  mixed cool street light and warm shop light, 50mm documentary feel, slight
  film grain, realistic skin texture, no posed glamour.
Use case: Editorial newspaper feature photo.
Constraints: No watermark, no logos, no tourist postcard color grading.
```

Settings: `size="1024x1536"` (portrait) or `1536x1024` (landscape), `quality="high"` for skin detail.

### 6.2 Product photography (with label integrity)

Material accuracy, lighting consistency, label fidelity, clean intended use.

```
Professional studio product photograph of [product] centered on a reflective
black acrylic surface creating a clean mirror reflection underneath.
Dramatic lighting with focused spotlight from above-right creating
sculptural shadows, secondary fill light at 30% intensity from left
preventing complete shadow blacks. Dark gradient background fading from
charcoal to deep black. Shot with 100mm macro lens capturing intricate
surface details and material textures. Professional retouching quality.
Square 1:1 format for product catalog. No watermark, no extra branding.
```

For background removal: use `background="opaque"` with a downstream cutout step, OR `background="transparent"` if outputting PNG/WebP.

### 6.3 UI mockups and app screenshots

Describe the product as if it already exists. Focus on layout, hierarchy, spacing, and real interface elements. Avoid concept art language.

```
Create a vertical mobile onboarding screen for a fictional app called NESTING.
Headline: "WELCOME TO NESTING."
Supporting line: "A quieter way to gather people around a table."
Buttons: "Get started", "I already have an account".
Small line illustration of three plates and two wine glasses.
Warm cream background, coral primary button, rounded sans serif,
clean spacing, exact readable copy.
No watermark. No real app branding.
```

Quality: `high` for dense interfaces. Size: `1024x1536` for portrait mobile, `1536x1024` for desktop screenshots.

### 6.4 Infographics, slides, diagrams, charts

Productivity visuals work best when written like an artifact spec, not an illustration request. Name the exact deliverable, define canvas and hierarchy, **provide the real text or data**, describe the visual language. Use `quality="high"` whenever the image contains small text, legends, axes, or footnotes. Use landscape sizes for deck-style outputs.

```
Create one pitch-deck slide titled "Market Opportunity" that feels like a
real Series A fundraising slide.

Use a clean white background, modern sans-serif typography like Inter,
and a crisp, minimal layout. The slide should include:

- A TAM/SAM/SOM concentric-circle diagram in muted blues and grays
- Specific, believable market sizing numbers:
  TAM: $42B    SAM: $8.7B    SOM: $340M
- A clean bar chart below showing market growth from 2021 to 2026,
  with a subtle upward trend
- Small footnotes: "AGI Research, 2024" and "Internal analysis"
- A company logo placeholder in the bottom-right corner

The design should look like it belongs in a deck that actually raised money:
highly readable text, clear data hierarchy, polished spacing, professional
startup-style visual language.

Avoid clip art, stock photography, gradients, shadows, decorative elements,
or anything that feels generic or overdesigned.
```

Settings: `size="1536x864"`, `quality="high"`.

### 6.5 Ads and marketing creatives

Write the prompt like a **creative brief**, not a technical image spec. Describe brand, audience, culture, concept, composition, and exact copy — then let the model make taste-driven decisions inside those boundaries.

```
Give me a cool in-culture ad/fashion shot for a brand called Thread.
It's a hip young street brand. The ad shows a group of friends hanging out
together with the tagline "Yours to Create."
Make it feel like a polished campaign image for a youth streetwear audience:
stylish, contemporary, energetic, and tasteful. Use clean composition,
strong color direction, natural poses, and premium fashion photography cues.
Render the tagline exactly once, clearly and legibly, integrated into the
ad layout.
No extra text, no watermarks, no unrelated logos.
```

### 6.6 Logo generation

Describe brand personality and use case. Ask for clean, original mark with strong shape, balanced negative space, scalability. Use `n=4` to get variants in one call.

```
Create an original, non-infringing logo for a company called Field & Flour,
a local bakery. The logo should feel warm, simple, and timeless. Use clean,
vector-like shapes, a strong silhouette, and balanced negative space.
Favor simplicity over detail so it reads clearly at small and large sizes.
Flat design, minimal strokes, no gradients unless essential. Plain background.
Deliver a single centered logo with generous padding. No watermark.
```

Settings: `size="1024x1024"`, `n=4`, `quality="medium"`.

### 6.7 Story-to-comic / multi-panel

Define the narrative as a sequence of clear visual beats, one per panel. Keep descriptions concrete and action-focused.

```
Create a short vertical comic-style reel with 4 equal-sized panels.

Panel 1: The owner leaves through the front door. The pet is framed in the
  window behind them, small against the glass, eyes wide, paws pressed high,
  the house suddenly quiet.

Panel 2: The door clicks shut. Silence breaks. The pet slowly turns toward
  the empty house, posture shifting, eyes sharp with possibility.

Panel 3: The house transformed. The pet sprawls across the couch like it
  owns the place, crumbs nearby, sunlight cutting across the room like a
  spotlight.

Panel 4: The door opens. The pet is seated perfectly by the entrance,
  alert and composed, as if nothing happened.
```

Settings: `size="1024x1536"` (vertical 4-panel), `quality="medium"`.

### 6.8 Character consistency across multiple images

Two-step pattern: anchor image, then continuation references the anchor.

**Step 1 — Establish the anchor:**

```
Create a children's book illustration introducing a main character.
A young forest helper wearing a green hooded tunic, soft brown boots,
and a small belt pouch. Kind expression, gentle eyes, warm but brave
personality.
Hand-painted watercolor look, earthy colors, soft outlines, whimsical
but grounded.
No text. No watermark.
```

**Step 2 — Continue (use `gpt-image-2/edit` with the anchor image as input):**

```
Continue the children's book story using the same character.
The same forest helper is rescuing a frightened squirrel after a winter storm.
Keep the same face, same green hooded tunic, same proportions, same color
palette, and same gentle personality.
Same watercolor look, snowy forest light, warm comforting mood.
Do not redesign the character.
No text. No watermark.
```

A **character reference sheet** (front/back/side views with callouts) is the strongest anchor possible — it compresses identity, wardrobe, palette, and turnarounds into one frame the model can lean on.

### 6.9 Style transfer

"Same style" is not enough. Name the parts that constitute the style.

```
Use the same visual language as the input image:
chunky pixel forms, limited arcade palette, bright glow accents,
clean silhouette edges, playful 1980s poster energy.

Generate a new scene of a motorcycle chase through a neon desert at night.
White background. No watermark.
```

### 6.10 Drawing → photoreal rendering

Tell the model whether the sketch is a **suggestion** or a **contract**.

```
Turn this drawing into a photorealistic image.
Preserve the exact layout, horizon line, river path, mountain placement,
tree placement, and overall perspective.
Use realistic natural materials and sunrise lighting.
Soft morning mist, believable rock texture, natural vegetation,
gentle water reflections.
Do not add people, buildings, animals, or text.
```

### 6.11 Virtual try-on (multi-image edit)

```
Image 1: the person to preserve.
Image 2: top reference.
Image 3: jacket reference.
Image 4: shoes reference.

Dress the person from Image 1 using the clothing from Images 2, 3, and 4.
Preserve face, facial features, skin tone, body shape, hands, pose, hair,
expression, background, camera angle, framing, and lighting exactly.
Replace only the clothing.
Fit garments naturally with realistic folds, drape, occlusion, and shadows.
Match lighting, shadows, and color temperature to the original photo.
Do not add jewelry, bags, text, or logos.
```

### 6.12 Object removal / cleanup

```
Remove every advertising sign and poster from the shop windows in this
storefront photograph.
Preserve the awning, the brick facade, the mullions, the window reflections,
the sidewalk, and every person on the sidewalk exactly.
Reconstruct the glass naturally: clean reflections of the street, no
ghosting of the removed posters, no leftover adhesive marks, no logo drift.
Match the original lighting, white balance, and film grain.
No watermark.
```

Use `input_fidelity="high"` on the edit endpoint (note: not applicable on `gpt-image-2`; use on legacy `1.5`/`1` only).

### 6.13 Lighting / weather / season transformation

```
Change only the weather and lighting.
Make the scene look like a winter evening with light snowfall.
Preserve identity, geometry, camera angle, object placement, and composition.
Keep all signs, buildings, and people in the same positions.
```

### 6.14 Image translation (localize without rebuilding)

```
Translate the text in the infographic to Spanish. Do not change any other
aspect of the image.
```

Keep typography style, placement, spacing, hierarchy. Translate verbatim. No reflow unless necessary. No edits to logos, icons, or imagery.

### 6.15 Collectibles / merch / packaging mockups

(Directly relevant to collectible product production pipelines.)

```
Create a collectible action figure of [character_description] in blister
packaging.

Concept: [the brief, audience, and emotional tone]

Style: Premium toy photography, realistic plastic and painted metal textures,
studio lighting, shallow depth of field, sharp label printing, high-end
retail presentation.

Constraints:
- Original design only
- No trademarks
- No watermarks
- No logos

Include ONLY this packaging text (verbatim): "[short_copy]"
```

Settings: `size="1024x1536"`, `quality="medium"` or `high` if packaging text needs to be readable.

---

## 7. The LLM-as-refiner loop (Claude and Gemini as prompt engineers)

This is where you, the agent, multiply prompt quality. The pattern is **generate → critique → refine**, where the LLM doing the critique is the same agent that will issue the next prompt. This is the production version of OPRO ("Optimization by Prompting") and self-refinement.

### 7.1 The four refinement stages

**Stage 1 — Intake & disambiguation.**
A human asks the agent for "a hero shot of our new sneaker for the spring campaign." Before composing a GPT Image 2 prompt, the agent fills any missing slots by asking targeted questions or by making explicit assumptions:

- *Scene:* studio? outdoor? lifestyle?
- *Subject:* which colorway? from what angle?
- *Use case:* hero image for Instagram? web banner? print billboard?
- *Constraints:* exact copy on-image? brand-mandated background? logo placement?

If the agent must proceed without answers, it states its assumptions inline: *"Assuming studio shot, three-quarter front, charcoal seamless background, no copy on image — confirm or correct."*

**Stage 2 — Initial composition.**
Draft a prompt using the five-slot template. Choose `quality`, `size`, `n`, `background` deliberately, not by default.

**Stage 3 — Self-critique (the meta-prompting step).**
Before sending to GPT Image 2, the agent runs a critique pass on its own draft. Use this internal prompt:

```
You are reviewing a prompt that will be sent to GPT Image 2.

Score the prompt against these criteria (1-5 each, with one-line justification):

1. Specificity: are subject, scene, lighting, and composition concrete
   visual facts, or vague adjectives?
2. Structure: does the prompt cleanly fill the five slots
   (Scene / Subject / Important details / Use case / Constraints)?
3. Text rendering: if the image must contain text, is it quoted exactly,
   placement specified, typography described, and verbatim rendering enforced?
4. Mode discipline: if this is an edit, is there an explicit change-vs-preserve
   list? If this is a composition, is each input image labeled by role?
5. Anti-slop: does the prompt avoid filler adjectives
   ("stunning," "8K," "masterpiece," "ultra-detailed")?
6. Constraints: are exclusions explicit ("no watermark, no extra text,
   no logos") and invariants stated?
7. Parameter fit: is the chosen quality / size / n appropriate for the use case?

For any criterion scoring < 4, propose a specific rewrite of just that section.
Do not rewrite the whole prompt — surgical edits only.

Then output the final revised prompt.
```

**Stage 4 — Post-generation review and iteration.**
Once GPT Image 2 returns the image, the agent evaluates the result against the original brief. If a follow-up edit is needed, the agent issues a *small, single-axis* edit (Rule 6 above). Never bundle multiple unrelated changes.

### 7.2 Claude-specific refinement patterns

Claude (Opus 4.7, Sonnet 4.6, Haiku 4.5) is particularly strong as a prompt refiner because of:

- **Extended thinking** — visible reasoning before output, useful for chain-of-thought critique of image prompts.
- **Structured XML output** — Claude reliably emits prompts wrapped in `<prompt>...</prompt>` tags for downstream parsing.
- **Long context** — Claude can hold a whole brand style guide, previous successful prompts, and the current brief simultaneously.

**Suggested Claude system prompt for an image-prompting agent:**

```
You are an image-prompting specialist for GPT Image 2.

Your job is to convert a user's brief into a production-quality GPT Image 2
prompt and matching API parameters.

Workflow:
1. Parse the brief. If critical context (use case, exact copy, brand
   constraints) is missing, ask up to 2 clarifying questions OR state
   assumptions inline and proceed.
2. Draft the prompt in the five-slot template
   (Scene / Subject / Important details / Use case / Constraints).
3. Run a silent self-critique against the seven criteria
   (specificity, structure, text rendering, mode discipline, anti-slop,
   constraints, parameter fit). Revise as needed.
4. Output the final prompt inside <prompt>...</prompt> tags.
5. Output the API parameters inside <params>...</params> tags
   (model, size, quality, n, background, optional input_fidelity).
6. Output a one-line rationale inside <rationale>...</rationale> tags
   explaining the most important decisions.

Default to gpt-image-2 unless the user specifies otherwise.
Default to quality="medium" unless the image contains dense text or
small detail, in which case use "high".
For exploration / variant generation, set n=4.
For production renders, set n=1.

Never include filler adjectives ("stunning," "masterpiece," "8K,"
"ultra-detailed," "award-winning"). They dilute the signal.

When in doubt, prefer concrete visual facts over abstract style names.
```

### 7.3 Gemini-specific refinement patterns

Gemini (3 Pro and later) brings complementary strengths:

- **Native multimodal grounding** — Gemini can be given the user's existing image references and reason about them alongside the prompt draft.
- **Search grounding** — for prompts requiring factual accuracy (depicting a real product, a real event, a real location), Gemini can pull current visual references.
- **Conversational thread state** — for multi-turn refinement, Gemini maintains thought signatures across turns.

**Suggested Gemini system instruction:**

```
You are an image-prompting agent that produces GPT Image 2 prompts.

For every brief:
1. If reference images are attached, analyze them and incorporate
   relevant visual facts (palette, composition, lighting direction,
   subject placement) into the prompt's "Important details" slot.
2. If the brief mentions a real product, real person, real location,
   or recent event, use search grounding to verify visual details
   before writing the prompt.
3. Compose the prompt using the canonical five-slot structure:
   Scene / Subject / Important details / Use case / Constraints.
4. Quote any in-image text exactly. Specify placement and typography.
   Add "Render verbatim. No extra text. No duplicate text." for text-
   heavy images.
5. Choose API parameters (size, quality, n) based on the use case.
6. Return the prompt, parameters, and a brief rationale in this format:

PROMPT:
[the prompt text]

PARAMS:
model: gpt-image-2
size: [chosen size]
quality: [low/medium/high]
n: [number of variants]
background: [opaque/transparent]

RATIONALE:
[2-3 sentences on key choices]

Never use filler adjectives. Never invent constraints the user did not
state. When a brand or character must remain consistent, anchor with a
prior approved image as a reference input.
```

### 7.4 Iterative refinement: the chain-of-edits pattern

When an initial generation needs revision, use a **small-step chain** rather than a rewrite.

```
Turn 1 (initial generation):
[full five-slot prompt]

Turn 2 (refine lighting):
Make the lighting warmer — shift the key light toward sunset color
temperature. Preserve subject, composition, background, framing,
and all other details.

Turn 3 (refine prop):
Remove the second coffee cup on the right. Preserve everything else,
including the table grain, the spilled sugar, and the napkin position.

Turn 4 (refine text):
The poster on the back wall should read "DAILY GRIND" in white chalk
lettering at the top. Render verbatim. No additional words. Preserve
the rest of the image exactly.
```

Each turn changes one thing. The model keeps the rest of the scene stable.

### 7.5 Programmatic refinement loop (for agent pipelines)

For automated pipelines, structure the loop as:

```
function generateImageWithRefinement(brief, maxIterations=3) {
  prompt = composeInitialPrompt(brief)         // Section 2 template
  prompt = selfCritiqueAndRevise(prompt)        // Section 7.1 stage 3

  for (i = 0; i < maxIterations; i++) {
    image = callGPTImage2(prompt, params)
    evaluation = evaluateImageAgainstBrief(image, brief)

    if (evaluation.passes) return image

    // Surgical, single-axis edit
    editPrompt = composeEditPrompt(evaluation.gap)
    image = callGPTImage2Edit(image, editPrompt, params)
  }

  return image
}
```

For the `evaluateImageAgainstBrief` step, an LLM with vision (Claude with vision, Gemini, or GPT-4 Vision) can score the output against the original brief and return either `passes: true` or a specific gap description (`"text 'Fresh and clean' is missing the second word"` / `"product is in the right third but brief asked for left third"`).

---

## 8. API parameter cheat sheet

### 8.1 `gpt-image-2` (text-to-image)

| Parameter | Values | Notes |
|---|---|---|
| `model` | `gpt-image-2` | Default for all new work |
| `prompt` | string | Use the five-slot template |
| `size` | any WxH within constraints | Constraints: max edge < 3840, both edges multiples of 16, ratio ≤ 3:1, total pixels 655,360–8,294,400. Above 2560×1440 is experimental. |
| `quality` | `low` / `medium` / `high` | Default `medium`. Use `high` for dense text, small detail, mixed-script content. |
| `n` | 1–N | Default 1. Use 4 for variant exploration. |
| `background` | `opaque` / `transparent` | Transparency on PNG/WebP only. |
| `output_format` | `png` / `jpeg` / `webp` | PNG for transparency or sharp text; JPEG for photographic outputs. |

### 8.2 `gpt-image-2/edit` (image editing and composition)

Same parameters as above, plus:

| Parameter | Values | Notes |
|---|---|---|
| `image` (or `image_urls`) | up to 16 references | Either file IDs or fully qualified URLs. |
| `mask_image_url` | URL | Optional. Used to scope edits to a region. |

### 8.3 Popular size presets

| Label | Resolution | Use |
|---|---|---|
| Square | `1024x1024` | General purpose; logos; social posts |
| HD portrait | `1024x1536` | Mobile UI; book covers; portrait editorial |
| HD landscape | `1536x1024` | Web banners; landscape photo; slides |
| Pitch slide | `1536x864` | 16:9 deck slides |
| 2K / QHD | `2560x1440` | Widescreen; upper reliability boundary |
| 4K / UHD | `3840x2160` (round down to `3824x2144`) | Experimental; print prep |

### 8.4 Pricing reference (as of May 2026)

- Text input: ~$8.00 per 1M input tokens
- Cached input: ~$2.00 per 1M tokens
- Image output: ~$30.00 per 1M output tokens
- Per-image cost typically lands between $0.04 (low quality, small) and $0.35 (high quality, 2K+).

Verify against OpenAI's current pricing page before relying on this in production cost models.

### 8.5 Model selection guide

- **Default to `gpt-image-2`** for all new production work, including customer-facing assets, photorealism, editing-heavy flows, brand-sensitive creative, and any work where first-pass quality matters more than absolute minimum cost.
- **Use `gpt-image-2` with `quality="low"`** when latency and unit cost dominate (high-volume generation, experimentation, draft assets).
- **Keep `gpt-image-1.5` or `gpt-image-1`** only for backward compatibility while migrating prompts.
- **Use `gpt-image-1-mini`** only for exploratory or low-stakes batch work where cost is the primary constraint.

---

## 9. Failure-mode catalog

Use this as a pre-flight checklist. For each failure mode, the symptom and the fix.

| Symptom | Likely cause | Fix |
|---|---|---|
| Text is garbled or invented | Text not quoted; no verbatim instruction | Quote text exactly, add "Render verbatim. No extra characters. No duplicate text." Use `quality="high"` for small text. |
| Edit changed the whole scene | No preserve list; preserve list dropped mid-conversation | Re-state the full preserve list in every iteration. Use "change only X, keep everything else the same." |
| Composition is generic / "AI-looking" | Filler adjectives crowded out visual facts | Strip "stunning," "8K," "masterpiece," "ultra-detailed." Replace with concrete visual facts (lens, lighting direction, materials, textures). |
| Subject is in the wrong position | Composition not specified | State framing, viewpoint, and placement explicitly ("subject centered with negative space on left," "three-quarter angle, eye-level"). |
| Multiple unrelated elements drift across turns | Bundled multi-axis edit | Issue one change per turn. Revert and split into a chain of small edits. |
| Brand element / character drifted | No anchor reference; preserve list missing identity details | Provide a reference image; lock identity in the preserve list ("preserve face, hair, exact green tunic, proportions"). |
| Logo or trademark appeared unwanted | No exclusion | Add "no logos, no trademarks, no watermark." |
| Multi-image composition merged elements wrong way | Inputs not labeled | Label each input: "Image 1: base. Image 2: jacket reference." Reference labels in instructions. |
| Image looks staged / posed when realism was wanted | Studio language used | Remove "professional photography," "cinematic," "award-winning." Add "candid," "documentary," "unposed," real imperfection cues. |
| Dense data visualization came back blurry / wrong | `quality="low"` or `"medium"`; size too small | Use `quality="high"`. Use a landscape size (`1536x864` or `1536x1024`). Include exact numbers and labels in the prompt. |
| Aspect ratio was respected but composition broke | Specified ratio without specifying composition for that ratio | Re-describe composition for the chosen aspect ratio ("wide 16:9: hero left, copy right, negative space center"). |
| Mixed-script text rendered only one script correctly | Each script not separately specified | Quote each language's text separately, specify font behavior for each script, place each on canvas explicitly. |
| Output is opaque when transparency was needed | `background` not set or JPEG format | Set `background="transparent"`, use PNG or WebP output format. |

---

## 10. Quick reference card

For agents that need a one-screen summary:

```
TEMPLATE
  Scene / Subject / Important details / Use case / Constraints

MODES
  Generate           → gpt-image-2          → five-slot template
  Edit (one image)   → gpt-image-2/edit     → change + preserve
  Compose (multi)    → gpt-image-2/edit     → label each input by role

ANTI-SLOP
  Visual facts > vague praise
  Style tags need visual targets
  Say the real thing
  Change vs preserve, every iteration
  Text = typography (quote, place, specify, hard-stop)
  One revision per turn

TEXT IN IMAGES
  1. Quote it exactly
  2. Specify placement
  3. Specify typography
  4. Add hard stop ("Render verbatim. No extra characters.")
  5. For tricky words, spell letter by letter

PARAMETERS
  size       any valid WxH (max edge <3840, multiples of 16, ratio ≤3:1)
  quality    low (speed) / medium (default) / high (text, detail)
  n          1 (production) / 4 (exploration)
  background opaque / transparent (PNG, WebP only)

REFINEMENT LOOP
  intake → draft → self-critique → generate → evaluate → small-edit chain

DEFAULT SIZES
  1024x1024  square / logo
  1024x1536  portrait / mobile UI / editorial
  1536x1024  landscape / banner / slide
  1536x864   16:9 deck slide
  2560x1440  2K widescreen (upper reliability boundary)
```

---

## 11. Sources

Primary sources consulted in preparing this guide:

- **OpenAI Developer Cookbook — "GPT Image Generation Models Prompting Guide"** (Mandeep Singh & Emre Okcular, April 21, 2026). The authoritative production reference.
- **OpenAI — "Introducing ChatGPT Images 2.0"** (April 21, 2026 release announcement).
- **fal.ai — "GPT Image 2 Prompting Guide and Examples"** (Ilker Izgi, April 21, 2026). Source of the five-slot template, anti-slop rules, and three-mode taxonomy used here.
- **VentureBeat** — coverage of feature set and Thinking Mode capabilities (April 21, 2026).
- **AI/ML API Blog — "GPT Image 2: Release Date, Features, and Everything You Need to Know"** (Valerii Brizhatiuk, updated May 15, 2026).
- **Segmind, PixVerse, ImagineArt, Framia, CrePal** — independent prompting analyses cross-referenced for consistency.
- **Anthropic — Claude prompting best practices** (for the meta-prompting and self-correction patterns in Section 7).
- **Industry research on Optimization by Prompting (OPRO) and Maestro self-improving T2I orchestration** for the refinement loop design.

GPT Image 2 was released on **April 21, 2026**. API access opened in early May 2026. DALL·E 2 and DALL·E 3 deprecated on May 12, 2026. Verify model availability and pricing against OpenAI's current documentation before shipping.

---

*Document prepared for the your agent pipeline. Refresh this guide whenever OpenAI ships a notable update to the gpt-image-2 family or when production data shows a recurring failure mode the catalog does not cover.*
