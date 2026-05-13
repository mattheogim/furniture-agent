---
name: personalization
description: Records the user's like/dislike feedback and updates the taste profile so validator and recommend grow more accurate over time.
trigger: Runs automatically whenever the user reacts to an item (like/dislike/comment)
---

# 09 — Personalization: Taste Learning

## Role
Tinder-style: every time the user reacts to an item,
analyze the signal and **update the profile so the next research and validator are more accurate**.

---

## How Feedback Is Collected

### Reaction Types
```
✅ YES   → "I like this" / "save" / "yes, this vibe"
❌ NO    → "no" / "not for me" / "pass"
🤔 MAYBE → "decent, want to see more" / "anything similar?"
💬 COMMENT → "legs are too thin" / "color is too bright" / "more rustic than this"
```

### What to Show for Reactions
- Items from the organization-sorted list
- Show 1–3 at a time and collect reactions
- Casual framing: "what about this one?"

---

## Personalization Profile Structure

Accumulated within the session and updated continuously as the conversation goes:

```
personalization_profile = {

  # confirmed preferences (YES reactions)
  liked: {
    materials: ["solid oak", "linen", "ceramic"],
    colors: ["cream", "warm white", "sage green"],
    styles: ["low leg", "rounded edges", "natural grain"],
    price_range_by_category: {
      "sofa": { min: 300, max: 900 },
      "coffee_table": { min: 80, max: 300 }
    },
    brands: ["IKEA", "Article"],
    condition_preference: "both",  # new / used / both
    specific_items: [
      { name: "KIVIK beige", platform: "IKEA", reason: "liked" }
    ]
  },

  # confirmed dislikes (NO reactions)
  disliked: {
    materials: ["chrome", "glass top", "plastic"],
    colors: ["cold grey", "black"],
    styles: ["thin metal legs", "very minimalist", "too modern"],
    price_sensitivity: null,
    brands: [],
    specific_items: [
      { name: "KALLAX", platform: "IKEA", reason: "too boxy" }
    ]
  },

  # detailed signals extracted from comments
  signals: [
    { type: "NO", item: "metal frame sofa", extracted: "legs too thin" },
    { type: "NO", item: "white gloss table", extracted: "color too cold" },
    { type: "YES", item: "oak round table", extracted: "this wood vibe is right" },
    { type: "COMMENT", raw: "something a bit more rustic", extracted_preference: "rustic, aged wood" }
  ],

  # learned rules (auto-inferred)
  inferred_rules: [
    "When solid wood, prefers oak/walnut, avoids pine/MDF",
    "Material/feel matters more than price",
    "Used is OK (if condition is good)",
    "Doesn't like things that are too minimal"
  ],

  # stats
  stats: {
    total_shown: 15,
    yes: 4,
    no: 8,
    maybe: 2,
    comment: 1,
    yes_rate: 0.27
  }
}
```

---

## Update Logic

### On YES
```
1. Add the item's material/color/style/price → into liked
2. If the same trait already exists in liked → +1 confidence (track frequency)
3. The brand/platform → +1 confidence
4. Consider updating inferred_rules
   e.g. 3 consecutive solid-wood YESes → create "strong preference: solid wood" rule
```

### On NO
```
1. Add the item's material/color/style → into disliked
2. If there's a comment → store in signals + parse natural language
   "legs are too thin" → disliked.styles += "thin legs"
3. Same trait NO'd 2+ times → mark as "hard dislike"
   → Validator auto-deprioritizes it
```

### On MAYBE
```
1. Save the item to a "hold" list
2. First, look within the master list already gathered by 03_research for similar alternatives and show them
3. If there are no more alternatives within the existing list:
   Send "I've shown everything I found. Let me search for more in this direction!" along with a re-search signal to research (with a cap of 2 re-searches per furniture category to avoid infinite loops)
4. Hold off on liked/disliked updates (uncertain)
```

### Comment Parsing
```
User text → intent extraction:

"warmer than this" → search_adjust: +warm +wood -cold
"too expensive" → price_sensitivity: lower for this category
"any other colors?" → fetch_color_variants: true
"is there a used one?" → search_used: true for this item
"something more unique" → brand_tier: designer/boutique
"not IKEA" → exclude: IKEA (for this search)
```

---

## Signal Sent to Validator

Before every validator run, pass the personalization_profile:

```
Additional input received by validator:
{
  boost_if: {
    materials: ["solid oak", "linen"],     # if material matches → score +15
    styles: ["rounded", "low leg"],         # if style matches → score +10
    condition: "used"                       # if used is preferred → +5
  },
  penalize_if: {
    materials: ["chrome", "glass"],         # if material matches → score -20
    styles: ["thin metal legs"],            # if style matches → score -15
  },
  hard_exclude: [
    "chrome material",                      # NO'd 2+ times → exclude entirely
  ]
}
```

---

## Signal Sent to Research

When re-search is needed, adjust keywords based on personalization:

```
Original keywords: "coffee table vancouver"

After personalization:
"solid oak coffee table rounded natural grain vancouver"
+ exclude: "glass top", "chrome", "metal frame"
```

---

## Profile Summary (when showing it to the user)

```
"Here's what I've picked up about your taste so far:

✅ Likes: solid oak, linen fabric, low legs, rounded edges
❌ Dislikes: chrome/metal, cold whites, very thin legs
💡 Pattern: material/feel matters more than price, used is OK

Sound right? Anything to tweak?"
```

→ The user can edit the profile directly

---

## Session Persistence

```
⚠️ Currently: kept only within the session (reset when the conversation ends)
Future extension: save to file → load in next session
  Save location: /outputs/personalization_profile.json
```
