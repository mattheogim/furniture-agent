---
name: research
description: Based on the furniture checklist and style tags, searches as broadly as possible across all Vancouver-relevant platforms. Allows ±30% of budget. No filtering — only collection.
trigger: After style-analysis completes / re-run when validator deems results insufficient / on user re-search request
---

# 03 — Research: Per-Platform Furniture Search

## Core Principle
> **Collect as broadly as possible. Filtering and selection is the validator's job.**
> Research doesn't judge — it just brings back as much as possible.

---

## Budget Range
```
Collection range: 0 ~ 150% (+50%) of the per-furniture max_price
Example: sofa max $1,000 → collect everything up to $1,500

Hard cap: anything beyond 150% of max is excluded
Reason: anything beyond 130% gets a -10 penalty in the validator so it's sorted to the bottom.
```

---

## Search Strategy

### Keyword Layering
```
Search each piece of furniture across 3 layers:

Layer 1 — Precise keywords (high precision)
  e.g. "solid oak round coffee table scandinavian Vancouver"

Layer 2 — Broad keywords (more results)
  e.g. "wood coffee table Vancouver" / "round table Vancouver"

Layer 3 — Style keywords (entire related style)
  e.g. "nordic furniture Vancouver" / "japandi table BC"

→ Merge all three layers, dedupe, and pass forward
```

### Apply Personalization (on re-search)
```
Apply signals from 09_personalization.md:

liked materials/styles → add to keywords
  e.g. "solid oak" liked → include "solid oak" in search

disliked materials/styles → soft-exclude from results
  (still collect but tag as low_match — validator makes the final call)

hard_exclude → skip collection entirely
  (traits that received NO twice or more)
```

---

## Per-Platform Execution

### 🔵 Facebook Marketplace

**Login: computer use → user-provided credentials**

```
Run:
1. browser: open facebook.com/marketplace
2. Enter credentials (user-provided)
   → Not stored, session-only
   → If 2FA appears, ask the user for the code
3. Search bar: [furniture name] + [style keywords]
4. Location filter: Vancouver, BC / within 40km
5. Price filter: 0 ~ max_price × 1.3
6. Sort by: Newest
7. Scroll the results → collect up to 15
8. Click into items to also collect details (description, extra photos)

Collected info:
- Title / price / condition description / posted date
- Photo (first) / location (neighborhood name)
- Link URL

⚠️ On login failure: skip FB, continue with the rest, leave a note
⚠️ Click into each listing to also collect details (description, extra photos)
```

---

### 🟡 IKEA Canada

```
Run: web_search + web_fetch

Search:
1. "IKEA Canada [English furniture name] [style]"
2. Browse ikea.com/ca/en categories directly
   e.g. /cat/sofas-fu003/ → apply filters

Collect:
- Product name / product number / price (CAD)
- All color options (each as a separate row)
- Coquitlam / Richmond store stock status
- Dimensions / material info
- Product URL

→ If multiple color options exist, collect each as its own row
```

---

### 🟠 Structube

```
Run: web_search + web_fetch

Search:
1. Browse structube.com categories
2. "Structube [furniture name] [style]"

Collect:
- Product name / price / color options
- Vancouver delivery availability (enter postal V6B)
- Store pickup (Vancouver Broadway store)
- Dimensions / materials
- URL
```

---

### 🔴 Wayfair Canada

```
Run: web_search + web_fetch

Search:
1. Browse wayfair.ca categories
2. "Wayfair Canada [furniture name] [style] [material]"

Collect:
- Product name / brand / price
- Vancouver delivery availability (postal V6B) + delivery window
- Rating / review count
- Dimensions / material
- URL

⚠️ Some US sellers don't ship to Canada → validator filters them out
```

---

### 🟣 Other Brands (Vancouver-local + Canada D2C)

```
Always searched (regardless of remaining budget)

Fixed brands to explore:
- Article (article.com) — Vancouver-based D2C, good value
- EQ3 (eq3.com) — Canadian brand, Vancouver showroom
- CB2 Canada — modern design
- West Elm Canada — mid-century / bohemian
- Tuck Studio — Vancouver local
- Kits Interiors — Vancouver local

Search:
"[brand] [furniture name]" → explore individual sites

Collect:
- Product name / price / link
- Vancouver delivery/pickup availability
- Style notes
```

---

## Result Storage Format

```
raw_result = {
  furniture_item: "sofa",
  room: "living room",
  search_round: 1,          # increments on re-search (2, 3, ...)
  total_collected: 28,
  items: [
    {
      id: "sofa_001",
      platform: "IKEA",
      product_name: "KIVIK 3-seat sofa",
      condition: "new",
      price_cad: 799,
      color: "beige",
      material: "fabric, wood legs",
      dimensions: "228x83x45cm",
      link: "https://ikea.com/ca/...",
      image_url: "https://...",
      location: null,        # FB only
      listing_date: null,    # FB only
      notes: "In stock at Coquitlam",
      search_layer: 1,       # which layer surfaced it
      raw_description: "..."
    },
    ...
  ]
}
```

---

## Re-search Logic (when validator requests)

```
Re-search triggers:
- Validator signals "insufficient results"
- User asks "find more"

Re-search rules:
1. Receive the list of already-collected item IDs
2. Same product name + same platform → duplicate → skip
3. Same product on a different platform → collect (price comparison has value)
4. Expand search keywords to Layer 2, 3
5. Apply the latest personalization signals

Duplicate handling:
- Exact match (same URL): exclude
- Similar product (different color/size): collect (has value as an option)
- Same product but cheaper: always collect (note the price comparison)

Maximum re-search attempts: 3
→ After 3 attempts and still insufficient → notify the user:
  "Options for this piece are limited in Vancouver right now.
   Want to adjust the criteria? (e.g. widen style, adjust budget)"
```

---

## On Completion

Pass the full collection → `04_validator.md`
Also pass the personalization_profile (validator uses it in priority scoring)
