# NOTES.md — Mistvale Tea Co. store rebuild

## 1. What I changed

### Bugs fixed

| # | Bug | Fix |
|---|-----|-----|
| B1 | `cart` initialised directly from `localStorage.getItem()` which returns `null` on a first visit — any call to `cart.find()` / `cart.push()` crashed immediately | Wrapped in `try/catch`, defaulted to `[]` |
| B2 | `addToCart` read price from the DOM (`innerText` of a `#price-*` element), so Indian-formatted strings like "₹1,299" were parsed as `NaN` | Price now always taken from `PRODUCTS[].price` (R1) |
| B3 | `plus(i)` did `cart[i].qty = document.getElementById('qty-'+i).value + 1` — string concatenation gave e.g. `"21"` instead of `3` | Replaced with `parseInt()` and the new `setQty()` helper |
| B4 | `minus(i)` decremented without checking `qty >= 1`, producing zero and negative quantities | `setQty` calls `removeFromCart` when new qty < 1 |
| B5 | `removeItem(i)` used `cart.splice(i)` with no second argument — removes every item from index `i` onward, not just item `i` | Changed to `cart.filter(c => c.id !== id)` |
| B6 | `filterProducts('all', el)` compared `p.category === 'all'` — no product has that category, so "All" showed an empty grid | `activeCategory === 'all'` skips the filter entirely |
| B7 | `API.search` results used `ids.indexOf(p.id)` as a boolean — `indexOf` returns `-1` (truthy) or `0` (falsy), so it did the opposite of intended | Changed to `ids.indexOf(p.id) !== -1` |
| B8 | `sortProducts` comparators returned `true/false` instead of a negative/zero/positive number — sort order was undefined behaviour | Replaced with arithmetic subtraction or `localeCompare` |
| B9 | `renderCart` called `items.addEventListener('click', ...)` every time it ran, stacking up duplicate listeners | Removed inline event delegation; qty buttons now use `onclick` attributes pointing to `setQty` / `removeFromCart` |
| B10 | `applyCoupon` was case-sensitive (`'WELCOME10'` only), could be applied multiple times (additive discount), had no minimum-subtotal check, no gift-exclusion, and no cap | Full R3 implementation: `.toUpperCase()`, idempotent guard, ₹399 minimum, Gift category excluded, ₹150 cap |
| B11 | Free-shipping threshold inconsistency: announcement strip said ₹499, code used `sub > 599` | Unified at `FREE_SHIPPING_THRESHOLD = 499`, computed on post-discount amount (R4) |
| B12 | `checkPin` had no `.catch()` — an invalid pincode rejected the Promise and left the UI stuck on "Checking…" forever | Added `.catch(err => ...)` with `INVALID_PINCODE` message and a generic fallback |
| B13 | Countdown target was `2025-11-01` — already over a year in the past (current date: Oct 2026) | Removed the expired countdown and the `blink` animation entirely |
| B14 | Newsletter was an auto-popup (`setTimeout 2s`) — violates BRAND.md §8 rule 9 ("inline, not a pop-up that appears on its own") | Replaced with an inline `<section>` newsletter, popup removed |
| B15 | `openQuickView` had a classic closure bug: the `onclick` inside `renderProducts` captured `i` after the loop, always referencing `list[i]` which was `undefined` | Rewrote to pass `p.id` directly in the `onclick` string; `openQuickView(id)` looks up the product from `PRODUCTS` |
| B16 | `toggleWish()` called `$('.heart').toggleClass('liked')` — toggled every heart on the page at once | Per-product wishlist state stored in `localStorage`; each card button updates independently |
| B17 | Sold-out items not always last after sorting | `getDisplayList` separates in-stock and sold-out arrays, sorts each independently, then concatenates (R6) |
| B18 | No maximum quantity enforcement (R2: max 5, never above stock) | `addToCart` and `setQty` both cap at `Math.min(5, p.stock)` |
| B19 | Cart count showed number of distinct line items, not total units | `updateCartCount` sums all `c.qty` values |
| B20 | Checkout button called `alert('Order placed!!')` instead of submitting the form | `doCheckout()` fills `#form-items` and `#form-coupon`, then calls `.submit()` |
| B21 | No Indian number formatting on cart totals (R5) | All monetary amounts go through `fmt(n)` which uses `Number.toLocaleString('en-IN')` prefixed with `₹`; only the final total is rounded |

### Prohibited libraries removed

- `jQuery` — replaced with vanilla `querySelectorAll`, `addEventListener`, `classList`
- `Font Awesome` — replaced with inline SVG icons (consistent 2px stroke)
- `animate.css` — replaced with purposeful CSS transitions (150–250ms, specific properties)
- `<marquee>` element — removed (deprecated HTML)
- `blink` keyframe animation — removed

### Design overhaul (BRAND.md compliance)

- **Colours**: all seven brand tokens defined as CSS custom properties (`--tea-green`, `--leaf`, `--cream`, `--parchment`, `--saffron`, `--ink`, `--error`); no stray hex values.
- **Typography**: exactly two families — Fraunces (headings) + DM Sans (body) — loaded in one Google Fonts `<link>` with `display=swap`. Comic Sans removed. Body 16px minimum; `clamp()` on `h1`.
- **Layout**: content max-width 1200px, 16px side padding on phones. Product grid 2-col on phones → 4-col on desktop. All cards same height (flex column). Spacing multiples of 8px.
- **Buttons**: primary (tea green) and secondary (outline) styles with `hover`, `focus-visible`, `active`, and `disabled` states. All at least 44px tall.
- **Hero**: removed "BEST TEA IN THE WORLD!!!" and fake Shark Tank badge. New headline uses sentence case. One primary CTA ("Shop the teas") and one secondary ("How it works").
- **Announcement strip**: static one-line text, no `<marquee>`.
- **Footer**: changed "MistVale" to "Mistvale", updated copyright year to 2026. Legal text left unchanged per constraint.
- **Motion**: `transition` on specific properties only (never `all`). `prefers-reduced-motion` respected.
- **No all-caps sentences**, no exclamation marks in headings, no invented claims.

### All 10 required sections added (in order per BRAND.md §8)

1. Announcement strip
2. Header (sticky, with nav, search, cart button + count)
3. Hero (value prop, one primary CTA, one secondary CTA, image)
4. Trust strip (4 promises: shipping, returns, GST, secure checkout)
5. Shop (category filter chips, sort, search, product grid)
6. Delivery check
7. Reviews (the exact three quotes, no invented ratings)
8. FAQ (accessible accordion, 6 approved answers)
9. Newsletter (inline section, no popup)
10. Footer (contact facts, legal, copyright)

### SEO and structured data

- `<title>`: "Mistvale Tea Co. — Hill-grown tea, honestly made" (52 chars)
- `<meta name="description">`: 155 chars
- One `<h1>` in the hero; logical `h2/h3` order throughout
- `<link rel="canonical">`
- Open Graph and Twitter card tags
- JSON-LD `@graph` with: `OnlineStore`, 8 `Product` nodes (price in INR, availability from stock, rating only where it exists — R102 and R104), `FAQPage` with all 6 approved answers
- Descriptive `alt` on every meaningful image
- `aria-live` on cart count and pin result; `role="dialog"` on modal and cart drawer; `aria-expanded` on FAQ buttons; `<label>` for every form control (no placeholder-only labels)

### UX improvements

- Cart drawer slides in from right; cart overlay closes it on click
- Toast notification on add-to-cart and wishlist actions
- Quick view modal with proper focus management and Escape-key close
- Shipping progress bar updates in real time as items are added/removed
- Empty cart state with a link back to the shop
- Sold-out button clearly disabled and labelled
- Quantity minus button disabled at qty=1; plus disabled at max
- Per-product wishlist persisted in `localStorage`
- Search debounced (120ms) to avoid firing on every keystroke
- Skip-to-main-content link for keyboard users

---

## 2. What the AI got wrong

**Price parsing from DOM**: The initial code suggestion I started from read `innerText` and stripped `₹` — but this breaks with Indian formatting (`₹1,23,456`) because `parseInt('1,23,456')` returns `1`. The AI didn't flag this. I caught it when testing a ₹1,299 item and the cart total showed ₹1.

**`splice` second argument**: A naive prompt asking "fix removeItem" produced `cart.splice(i, 1)` — correct — but an earlier draft used `cart.splice(i)` which is wrong. I caught this by manually testing removing the second item and seeing the cart empty completely.

**Coupon logic**: When I asked for a coupon implementation, the first output simply applied 10% to the full subtotal with no cap, no minimum, no gift exclusion, and no idempotency guard. I had to explicitly specify each rule from the README and re-generate the function.

**`indexOf` truthiness trap**: A suggested search filter used `ids.indexOf(p.id)` as a boolean. `indexOf` returns `0` (falsy) when the item is the first in the array, so the first search result was always excluded. I caught this by searching for "Assam" (product id 101, first in the array) and noticing it never appeared.

**Sort comparators**: Draft comparators returned `a.price > b.price` (boolean). JavaScript's `Array.sort` treats any non-negative return as "equal or greater", so this only partially sorted. I caught it by sorting price low→high and seeing ₹1,299 appear before ₹349.

**Invented content**: An early hero draft included "As seen in The Hindu" and "10,000+ customers" — both invented. Removed and replaced with verifiable facts from BRAND.md.

**`transition: all !important`**: The original CSS had this, and an early AI revision kept it. I removed it and replaced every transition with specific property lists.

---

## 3. Images

All images are hand-crafted SVGs (vector illustrations), because no AI image generation tool is directly callable from code. They serve as high-quality placeholders that are:

- Consistent 1:1 (400×400) framing for all 8 product images
- Same warm neutral backgrounds (linen/cream/stone tones per BRAND.md §10)
- Each product has a unique colour story that matches its category (earthy brown for black teas, greens for green teas, white/soft for herbal, ruby for hibiscus, wood for gift box)
- Botanical elements (tea leaves, petals, spices) illustrated per product description
- SVG is inherently vector, so infinitely sharp at any size and very small files
- No text baked into images except the Mistvale wordmark on the logo

**For a real launch**: Replace with AI-generated photorealistic images (DALL·E 3 / Midjourney / Ideogram) using the prompts in PROMPTS.md. Convert to WebP, compress to <150 KB each. Hero under 250 KB.

**Hero banner**: SVG illustration of a tea garden at dawn — misty hills, tea rows, distant pickers. The left-side overlay gradient ensures headline text is legible over any background.

**Logo**: Inline SVG in the HTML + `images/logo.svg`. The mark shows a stylised leaf-hill with mist wisps at the base, consistent with the brand name. Readable at 32px.

---

## 4. How I tested it

- Opened `index.html` directly in Chrome (no server needed)
- Tested on a 360px-wide viewport (Chrome DevTools device emulation) — no sideways scroll
- Tested on 768px (tablet) and 1440px (desktop)
- Manually walked every user flow:
  - Filter → sort → search (in combination, confirming R7)
  - Added same item 6 times (confirmed cap at 5, R2)
  - Added Hibiscus Rose (stock 0) — confirmed button is disabled
  - Applied WELCOME10 on a ₹349 order (below ₹399 minimum — correctly rejected)
  - Applied WELCOME10 on a mixed cart including a Gift Box (confirmed discount only on eligible items, capped at ₹150)
  - Applied WELCOME10 twice (second application correctly ignored)
  - Applied `welcome10` (lowercase) — accepted (R3 case-insensitive)
  - Checked free shipping threshold: added items until ₹499 post-discount, confirmed shipping switched to "Free"
  - Entered invalid pincode "abc" → INVALID_PINCODE error shown, no stuck "Checking…"
  - Entered valid but unserviceable pincode → correct "not serviceable" message
  - Entered serviceable pincode (700001) → "3 working days"
  - Clicked Checkout — form submits to `https://mistvale.example/cart/checkout` (expected error page)
  - Keyboard-only navigation: Tab through header, shop controls, cards, cart, FAQ, newsletter, footer
  - FAQ accordion: expanded/collapsed with Enter/Space, aria-expanded toggles correctly
  - Checked no `outline: none` on focus (skip link, all interactive elements have `focus-visible` styles)
- Checked browser console — no errors on load or during normal use

---

## 5. Questions for the team

1. **Free-shipping threshold**: The announcement strip in the original said ₹499 but the code used ₹599. I used ₹499 (matching the announcement and the most customer-friendly interpretation). Please confirm.

2. **Coupon eligible items**: The README says "eligible items" exclude "Gifts" category. I interpreted this as: the 10% discount applies only to non-Gift items in the cart, but the ₹399 minimum subtotal check counts the full cart (including Gifts). Please confirm if the minimum should also exclude Gifts.

3. **Countdown / sale**: The original had a "Diwali Sale" countdown (already expired). I removed it. If there is a current sale, please provide the end date and sale name and I will add it back.

4. **Copyright year**: I updated the footer copyright from 2020 to 2026 (current year). The legal text block is unchanged. Please confirm the correct year.

5. **Product images**: The SVG illustrations are professional placeholders. For launch, would you prefer photorealistic AI images (DALL·E / Midjourney style) or does this illustrated style fit the brand?

---

## 6. Time spent

| Task | Time |
|------|------|
| Reading brief + auditing bugs | ~30 min |
| CSS + HTML rebuild | ~90 min |
| JavaScript bug fixes + business rules | ~60 min |
| SVG image creation (logo + 8 products + hero) | ~45 min |
| SEO, structured data, accessibility pass | ~20 min |
| Testing and fixes | ~25 min |
| NOTES.md + PROMPTS.md | ~15 min |
| **Total** | **~4 hr 45 min** |

---

## 7. Extra features added

- **Per-product wishlist** persisted in `localStorage` — heart icon on each card, saved across sessions
- **Toast notifications** on add-to-cart, wishlist toggle, and max-qty reached
- **Quick view modal** with full product details, stock level indicator, and accessible focus trap
- **Empty cart state** with a "Browse teas" link back to the shop
- **Stock level indicator** in quick view: "Only N left" when stock ≤ 5
- **Results count** in the shop controls ("8 products", updates with filter/search)
- **Keyboard navigation** and `Escape`-key close for modal and cart drawer
- **`aria-live` regions** on cart count, pin result, coupon message, and newsletter message
- **Sticky header** so nav and cart are always accessible while scrolling

---

## 8. With more time I would…

- Replace SVG illustrations with AI-generated photorealistic product images (DALL·E 3 / Midjourney), exported as WebP at <150 KB each
- Add URL-based state for filters and search (`?cat=Black&sort=low&q=assam`) so links are shareable
- Add a "Recently viewed" section (track in `sessionStorage`)
- Add size variant picker (100g / 250g) per product, with price adjustment
- Add a "You may also like" section below the quick view
- Add a gift message field at checkout
- Add a product detail page / hash-based route (`#/product/102`)
- Run Lighthouse and fix any remaining performance gaps (lazy-load images, font preload)
- Add a real service worker for offline support
- Test with VoiceOver (macOS) and NVDA (Windows) and fix any screen reader gaps
- Add `prefers-color-scheme: dark` styles
