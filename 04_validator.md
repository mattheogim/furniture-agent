---
name: validator
description: Validates the results gathered by research and applies the personalization profile to rank them in the order the user is most likely to love. Doesn't hide — just reorders.
trigger: Runs automatically after research / re-runs after a re-search
---

# 04 — Validator: Validation + Priority Sorting

## Core Principle
> **Show everything. But surface what the user is most likely to love first.**
> Minimize exclusions — only what's truly unusable (not available in Vancouver, expired link, etc.).
> Everything else just gets a lower rank, but stays in the list.

---

## Step 1: Hard Filter (only truly unusable items)

Remove if ANY of the following applies:

```
REMOVE conditions:
❌ Not deliverable / pickupable in Vancouver (only when confirmed)
❌ Expired link / sold out
❌ No price info (only available by contacting the seller)
❌ Completely wrong furniture category (e.g. searching for a bed, returned a mattress cover)

KEEP (do NOT remove):
✅ Within budget ±30% → keep all
✅ Low style score → keep (just rank lower)
✅ Used condition unclear → keep + ⚠️ tag
✅ FB link unverified (not logged in) → keep + "needs manual check" tag
```

---

## Step 2: Validation Checks (scored)

Each item is checked on:

### 2-A. Vancouver Availability Score
```
+20: Direct pickup in Vancouver possible (FB local, in-store stock)
+15: Delivers to Vancouver (confirmed)
+10: Ships to Canada (Vancouver not confirmed)
 0: Unclear
-30: Doesn't ship (→ removed in Hard Filter)
```

### 2-B. Price Realism Score
```
+10: Below max_price
  0: 101–130% of max_price (acceptable range)
-10: 131%+ of max_price (over range but kept, rank lowered)

Suspicious-price detection (used items):
80%+ cheaper than average used price for the category → ⚠️ scam warning tag
(no impact on ranking — tag only)
```

### 2-C. Link Validity
```
IKEA / Structube / Wayfair:
→ Verify via web_fetch
→ Valid: +10 / 404 or out of stock: Hard Remove

Facebook Marketplace (logged in):
→ Verify via computer use browser
→ "No longer available" detected: Hard Remove
→ Valid: +10

Facebook (not logged in):
→ Cannot verify → score 0 + "unverified" tag
```

---

## Step 3: Style Matching Score

```
Base score:
Material match: up to +30
Color match: up to +25
Style category match: up to +25
Mood match: up to +20
→ Max base total: 100

Personalization adjustment (from 09_personalization):
Has liked material/style: +15
Has disliked material/style: -20
Hits hard_exclude: -40 (effectively sinks to the bottom)
Brand that received a YES: +5
```

---

## Step 4: Final Score Composition + Priority Sort

```
Final score = style match + availability + price + link

100+ : ⭐⭐⭐ Strongly recommended (user will almost certainly love)
80–99: ⭐⭐ Recommended
60–79: ⭐ Acceptable
40–59: Average (show, but lower down)
0–39 : Low (bottom of list, collapsed)
```

→ Sort by score and pass to organization

---

## Step 5: Detect Insufficient Results → Request Research Re-run

```
Criteria for "insufficient":
- Fewer than 2 items at ⭐ or above per furniture category
- Fewer than 3 items remain after Hard Filter
- User said NO three times in a row in Tinder UX

If insufficient:
→ Send research re-run signal
→ Pass list of already-collected item IDs (avoid duplicates)
→ Also pass the latest personalization signals (for keyword tuning)
→ Auto-retry up to 3 times
→ After 3, notify the user
```

---

## Step 6: Duplicate Handling

```
Same product found on multiple platforms:
→ Don't remove — group them
→ "Sold in [N] places, cheapest is [platform] $XXX"

Example output:
KIVIK sofa (beige, 3-seat) — found in 3 places
  🟡 IKEA new: $799
  🔵 FB used: $350 (2 years used)
  🔴 Wayfair new: $849
  → Recommendation: FB used (if condition is good) or IKEA new
```

---

## Validator Output Format

```
validated_results = {
  furniture_item: "sofa",
  hard_removed: 3,      # count + reasons
  total_passed: 14,     # how many passed
  items: [
    {
      id: "sofa_001",
      ...raw_result fields,
      validation: {
        hard_filtered: false,
        availability_score: 20,
        price_score: 10,
        link_score: 10,
        style_score: 87,
        personalization_boost: 15,
        final_score: 142,
        rank_tier: "⭐⭐⭐",
        flags: []          # e.g. ["⚠️ price suspiciously low", "unverified link"]
      }
    },
    ...
  ],
  duplicates_grouped: [
    {
      product: "KIVIK sofa beige 3-seat",
      found_in: ["IKEA $799", "FB $350", "Wayfair $849"],
      cheapest: "FB $350"
    }
  ],
  needs_more_research: false,   # if true, re-run research
  research_signal: null         # keyword hints for re-search
}
```

---

## On Completion

→ Pass sorted results to `05_organization.md`
→ Send "validation complete" signal to `09_personalization.md` (for stats update)
→ If needs_more_research = true, re-run `03_research.md` first
