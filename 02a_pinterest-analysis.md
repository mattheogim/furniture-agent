---
name: pinterest-analysis
description: Accesses Pinterest to analyze the user's saved pins/boards, or searches by style keyword to capture interior mood. Acts as an input source for style-analysis.
trigger: When the user provides a Pinterest link / asks "find it on Pinterest" / when style-analysis lacks references
---

# 02a — Pinterest Analysis

## When does it run?
- When the user provides a Pinterest board URL
- When the user provides a Pinterest pin URL
- When the user asks "find [keyword] on Pinterest"
- When `02_style-analysis` decides reference images are insufficient

---

## How it runs: Computer Use (browser)

Pinterest is partially accessible without login,
but full board/pin browsing benefits from logging in.

```
Login decision:
1. When the user gives a URL directly → try non-logged-in first
2. If blocked without login → "Should I log in to Pinterest? Share credentials"
3. For keyword-only search → non-logged-in is often enough
```

---

## Step 1: Branch by Input Type

### Case A — Board URL (e.g. pinterest.com/username/board-name/)
```
Run:
1. Open the board in the browser
2. Scroll and collect up to 20 pins
3. Capture image + description text for each pin
4. → Move to Step 2 (multi-image analysis)
```

### Case B — Single Pin URL (e.g. pinterest.com/pin/123456/)
```
Run:
1. Open the pin page
2. Capture image + description + related pins section
3. Collect top 5 related pins (for style consistency)
4. → Move to Step 2
```

### Case C — Keyword Search
```
Example input: "nordic living room cozy" / "japandi bedroom"

Run:
1. Open pinterest.com/search/pins/?q=[keyword]
2. Collect top 15–20 result images
3. Look for repeating patterns (extract elements that show up across pins)
4. → Move to Step 2

Auto-generate search keywords:
User keywords → translate to English + add interior context
Example: "Nordic living room" → "nordic living room", "scandinavian cozy interior"
```

### Case D — User's Own Board (analyze saved pins)
```
"Want me to analyze your saved board? I'll need to log in."
→ After login → filter saved pins down to interior-related only
→ Auto-exclude non-interior pins (food, fashion, etc.)
```

---

## Step 2: Image Pattern Analysis

Extract common elements from the collected images:

### Color Analysis
```
Extract main colors per image → compute frequency across all images
→ The top 5 colors are this user's true preferred palette

Example output:
1st: cream white (appears in 18/20 images)
2nd: oak/warm brown (15/20)
3rd: sage green (9/20)
4th: terracotta (6/20)
5th: off-black (4/20)
```

### Material Analysis
```
Frequently seen materials:
- Wood family: solid wood / plywood / rattan
- Fabric: linen / velvet / cotton
- Other: ceramic / marble / metal

→ Top 3 materials become preferred materials
→ Materials that never appear → add to the avoid list
```

### Style Tag Frequency
```
Tag each image with style labels, then tally:

Nordic/Scandinavian: 14
Japandi: 8
Minimalist: 12
Cozy/Hygge: 11
Mid-Century Modern: 3
Boho: 1

→ Top 3 become the main styles
```

### Recurring Furniture Items
```
Furniture types that appear across many images:
- Low-leg sofa: 16 times
- Round coffee table: 12 times
- Walnut/oak wood shelves: 10 times
- Linen cushions: 9 times
- Arch floor lamp: 7 times

→ The user's "frequently saved furniture style" = what they actually want
→ Feed this directly into research search keywords
```

---

## Step 3: Detecting Specific Products (Bonus)

```
If brand/product names appear in pin descriptions or links, extract them:
e.g. "IKEA Ranarp lamp" / "West Elm Andes Sofa"

→ Already concrete → bump these to top priority in research direct search
→ Validator confirms whether they're purchasable in Vancouver
```

---

## Step 4: Output Style Profile

```markdown
## Pinterest Analysis Result

📌 Analyzed pins: 20 (board: "Dream Home")

### 🎨 Preferred Palette
■ Cream white (90%)  ■ Oak brown (75%)  ■ Sage green (45%)
△ Accents: terracotta / off-black

### 🪵 Preferred Materials
1st: Solid wood (especially oak/walnut)
2nd: Linen fabric
3rd: Ceramic / pottery accents

🚫 Rarely seen: chrome metal, glass, plastic

### 🏷️ Style Tags
Main: Nordic Cozy, Minimalist Warm
Sub: Japandi elements (low furniture, negative space)

### 🛋️ Recurring Furniture Forms
- Low, wide fabric sofa (low legs)
- Round wooden coffee table
- Arched lighting
- Wall-mounted wood shelves

### 💡 Specific Products Detected
- IKEA FRIHETEN sofa (appeared 3 times)
- Anthropologie rattan basket (5 times)
```

→ Pass this result to `02_style-analysis.md` to merge into the combined style profile

---

## Caveats

```
⚠️ Pinterest crawling limits:
- Scrolling too fast can trigger bot detection → go slow
- Some boards are private without login
- Don't download images directly → use screen capture + analysis

⚠️ Analysis limits:
- Based on ~20 pins → if not enough, ask "Want me to analyze more?"
- If the board mixes topics (food + interiors) → filter to interiors only
```
