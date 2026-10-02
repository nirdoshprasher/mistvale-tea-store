# PROMPTS.md — Mistvale Tea Co. rebuild prompt log

All prompts used for code and images, in order, exactly as typed.

---

## Prompt 1
- Tool: Kiro (Claude Sonnet)
- Type: code
- Prompt:
  > Read the index.html, BRAND.md and README.md in this project. Give me a complete audit of every bug, business rule violation, design problem and constraint violation you can find. Be specific — name the line numbers and variable names where possible.
- Outcome: accepted
- Why: This produced the 31-item bug list that drove the entire rebuild. The AI found most bugs correctly; I cross-checked the `indexOf` truthiness trap and the `splice` missing-argument bug by re-reading the code manually.

---

## Prompt 2
- Tool: Kiro (Claude Sonnet)
- Type: code
- Prompt:
  > Now rebuild the full index.html from scratch following every rule in BRAND.md and the README constraints. Requirements: (1) One HTML file, no separate CSS/JS files. (2) No jQuery, no Font Awesome, no animate.css, no Bootstrap — plain HTML, CSS, JS only. Google Fonts allowed. (3) Use CSS custom properties for all brand colour tokens. (4) Fraunces + DM Sans, one Google Fonts link, display=swap. (5) Fix all 31 bugs from the audit. (6) Implement all business rules R1–R8 exactly. (7) All 10 required page sections in order from BRAND.md §8. (8) Complete SEO: unique title 50–60 chars, meta description 120–160 chars, one h1, logical heading order, canonical, OG + Twitter card. (9) JSON-LD @graph with OnlineStore, all 8 Products (price INR, availability from stock, rating only where it exists), FAQPage with the 6 approved answers. (10) Inline SVG icons only, consistent 2px stroke. (11) All buttons 44px tall minimum with hover/focus-visible/active/disabled states. (12) Responsive 360px–1440px, 2-col product grid on phones, 4-col on desktop. (13) Cart drawer (not inline), shipping bar updates on post-discount amount. (14) Accessible: skip link, aria-live on cart count and pin result, visible focus rings, labels on all inputs. (15) prefers-reduced-motion respected. Write the full file in chunks if needed.
- Outcome: accepted (written in three chunks due to size)
- Why: The full rebuild. I reviewed every section as it was written. Caught: the shipping bar HTML was missing from the cart drawer (JS referenced elements that didn't exist in the DOM), and the product image paths needed updating from .png to .svg after images were created.

---

## Prompt 3
- Tool: Kiro (Claude Sonnet)
- Type: code
- Prompt:
  > The cart drawer JS references `document.getElementById('ship-fill')` and `document.getElementById('ship-msg')` but those elements are not in the HTML. Add the shipping bar HTML inside the cart drawer, before the `cart-body` div. Use the existing `.ship-bar-wrap`, `.ship-msg`, `.ship-track`, `.ship-fill` CSS classes.
- Outcome: accepted
- Why: Caught during DOM inspection — `updateShipBar` was silently failing because the elements didn't exist.

---

## Prompt 4
- Tool: Kiro (Claude Sonnet)
- Type: image / SVG
- Prompt:
  > Create an SVG logo for Mistvale Tea Co. Rules: (1) Simple, recognisable at 32px. (2) A small mark — leaf, hill or mist motif — plus the wordmark "Mistvale". (3) Colours: tea green #1f3d2b, leaf #4f7942, cream #f6f1e7. (4) No text baked in except "Mistvale". (5) viewBox="0 0 200 56". Produce the raw SVG code.
- Outcome: accepted (minor colour tweaks applied)
- Why: The mark combined a leaf shape with hill and mist wisps, which matches the brand name well. I adjusted the mist-wisp stroke opacity downward because it was too heavy at small sizes.

---

## Prompt 5
- Tool: Kiro (Claude Sonnet)
- Type: image / SVG
- Prompt:
  > Create 8 product illustration SVGs for Mistvale Tea Co., one per product. Rules for all: (1) 400×400 square, same framing. (2) Warm neutral backgrounds in cream, linen, parchment or stone tones per BRAND.md. (3) Each image shows a pack or tin plus a hint of the tea/spice itself. (4) No text baked in except the product name label on the pack. (5) Brand colours: --tea-green #1f3d2b, --leaf #4f7942, --cream #f6f1e7, --parchment #ebe2cf, --saffron #d9962b. (6) Consistent artistic style across all 8. Products: p101 Assam Breakfast Black Tea (malty, CTC, morning), p102 Darjeeling First Flush (delicate, floral, tin), p103 Kashmiri Kahwa (green tea, saffron/cardamom/cinnamon/almonds), p104 Masala Chai (bold, ginger/cardamom/clove/pepper, bestseller), p105 Nilgiri Green Tea (smooth, Blue Mountains), p106 Chamomile & Tulsi (caffeine-free, flowers, calming), p107 Hibiscus Rose Infusion (tangy, deep ruby, sold out), p108 Tea Lover's Sampler Gift Box (wooden box, 6 mini tins).
- Outcome: accepted (p107 had the sold-out indicator added manually; p108 gift box lid position adjusted)
- Why: The AI produced good foundational SVGs. I manually added the dashed border on p107 to visually reinforce "sold out", and adjusted the gift box lid to look more naturally open.

---

## Prompt 6
- Tool: Kiro (Claude Sonnet)
- Type: image / SVG
- Prompt:
  > Create an SVG hero banner for Mistvale Tea Co. Dimensions 1200×600. Scene: misty tea garden at dawn, viewed from slightly above, with rows of tea bushes in the foreground, soft rolling hills in the middle distance, a dawn sky with a subtle sun glow. Include: (1) A left-side dark gradient overlay so white text is legible over the image. (2) Mist wisps at ground level. (3) Distant silhouettes of tea pickers. (4) Colour palette: deep tea green #1f3d2b for hills/sky, warm greens #4f7942 and #3a6332 for bushes, cream #f6f1e7 for mist, saffron #d9962b for sun glow. No text in the image.
- Outcome: accepted
- Why: The layered gradient approach (sky → hills → tea rows → mist → left-side text overlay) produced a convincing depth effect without photorealistic rendering.

---

## Prompt 7
- Tool: Kiro (Claude Sonnet)
- Type: code
- Prompt:
  > Update the PRODUCTS array in index.html to reference .svg files instead of .png for all 8 product images (p101.svg through p108.svg). Also update the hero <img> src from hero-banner.png to hero-banner.svg.
- Outcome: accepted
- Why: Straightforward reference update after images were created.

---

## Prompt 8
- Tool: Kiro (Claude Sonnet)
- Type: other
- Prompt:
  > Write NOTES.md for the Mistvale assessment following the exact format in the README. Sections: 1) What I changed (bugs, design, UX, images, technical), 2) What the AI got wrong (specific examples of wrong/risky/made-up output and how I caught them), 3) Images (which tool made each, what was done afterwards), 4) How I tested it, 5) Questions for the team, 6) Time spent, 7) Extra features added, 8) With more time I would…
- Outcome: accepted (lightly edited to ensure the "AI got wrong" examples matched the actual bugs I caught, not invented examples)
- Why: The AI draft was accurate in structure. I verified every "what the AI got wrong" example against the actual bugs before publishing to ensure honesty.

---

## Image prompts (for reference if using external AI image tools)

The following prompts are provided for anyone replacing the SVG illustrations with AI-generated photography. They were composed following BRAND.md §10 guidelines.

### Hero banner (Midjourney / DALL·E 3)
> A misty tea garden at dawn in the Darjeeling hills, India. Rows of dark green tea bushes stretch into the distance. Soft morning mist sits at ground level. A warm golden sunrise glows on the right horizon behind rolling green hills. Two distant figures in saris pick tea leaves. Natural light, soft shadows, calm and peaceful. No text. Cinematic wide shot, 2:1 aspect ratio. Colour palette: deep forest green, warm gold, soft cream mist.

### p101 — Assam Breakfast Black Tea
> A kraft paper tea pouch with a cream label, placed on a natural wood surface with scattered dark tea leaves around it. The label reads "Assam Breakfast". Natural side lighting, soft shadows. Square 1:1 crop. Warm brown and green tones. No people. Minimal and premium.

### p102 — Darjeeling First Flush
> An elegant gold and white cylindrical tea tin on a stone surface. A few dried white flowers and pale green tea leaves rest beside it. The label reads "Darjeeling First Flush". Soft window light from the left. Square 1:1. Pale gold and cream tones.

### p103 — Kashmiri Kahwa
> A dark green stand-up tea pouch with a saffron label band. Beside it: a cardamom pod, a cinnamon stick, a few saffron threads, and a small bowl of almonds on a slate surface. Square 1:1. Rich jewel greens and warm gold. Moody natural lighting.

### p104 — Masala Chai Blend
> A deep terracotta-red tea pouch on a warm wood surface. Scattered around it: fresh ginger root slices, green cardamom pods, whole cloves, and black pepper corns. The label reads "Masala Chai". Warm golden hour lighting. Square 1:1.

### p105 — Nilgiri Green Tea
> A sage green tea pouch on a cool linen surface with a few fresh green tea leaves and a sprig of mint beside it. Soft blue-grey background suggesting mountain mist. Square 1:1. Cool and fresh colour palette.

### p106 — Chamomile & Tulsi
> A white and cream tea pouch on a light stone surface. Beside it: three fresh chamomile flowers and a small bunch of holy basil (tulsi) leaves. Diffused soft light. Square 1:1. Pale and calming — cream, white, sage.

### p107 — Hibiscus Rose Infusion
> A deep ruby-red tea pouch on a dark slate surface. Dried hibiscus flowers and dried rose petals scattered around it. Moody natural light. Square 1:1. The pack looks slightly desaturated or has an "out of stock" label on it.

### p108 — Tea Lover's Sampler Gift Box
> An open wooden gift box on a natural wood surface, containing six small cylindrical tea tins in different colours. A small handwritten brewing guide card sits beside the box. Warm natural light from above. Square 1:1. Warm wood tones and rich tea colours.
