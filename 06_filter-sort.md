---
name: filter-sort
description: Filters and sorts the master list against user criteria to produce a shortlist.
trigger: After organization completes, or when the user asks to "sort" / "filter"
---

# 06 — Filter & Sort

## When does it run?
- After `05_organization` when the user requests it
- When the user changes the criteria
- On requests like "used only", "within budget", "best match"

---

## Supported Filters

### 📍 Basic Filters

```
1. Room filter
   → "show only living room"
   → room = "living room"

2. Condition filter
   → "new only" / "used only" / "both"
   → condition = "new" | "used" | "all"

3. Platform filter
   → "IKEA only" / "exclude Facebook"
   → platform = ["IKEA"] | exclude = ["Facebook"]

4. Furniture type filter
   → "show only sofas"
   → furniture_type = "sofa"
```

### 💰 Budget Filters

```
5. Exclude over-budget
   → "within budget"
   → price_cad <= room_budget

6. Price range
   → "between $200 and $800"
   → price_cad BETWEEN 200 AND 800

7. Remaining-budget filter (per room)
   → "only what fits in remaining living room budget"
   → price_cad <= remaining_budget["living room"]
```

### 🎨 Style Filters

```
8. Style score filter
   → "good matches only" (default: 60+)
   → style_match_score >= 60

9. Custom minimum score
   → "80 points and up only"
   → style_match_score >= 80
```

---

## Supported Sorts

```
Sort options (user picks):

A. Price ascending      → sort: price ASC
B. Price descending     → sort: price DESC  
C. Style match desc     → sort: style_match DESC ← default
D. By platform          → sort: platform
E. By room              → sort: room
F. Used first           → sort: condition (used first)
G. New first            → sort: condition (new first)

Composite sort allowed:
"highest style score first, then lowest price"
→ sort: style_match DESC, price ASC
```

---

## Building the Shortlist

After filter + sort, extract **Top 3 per furniture item**:

```markdown
# 🏆 Shortlist — Living Room

## Sofa TOP 3
| Rank | Product | Platform | Condition | Price | Style | Link |
|------|---------|----------|-----------|-------|-------|------|
| 1 | KIVIK beige | 🟡 IKEA | 🆕 New | $799 | ⭐ 92 pts | [link] |
| 2 | Used beige sectional | 🔵 FB | ♻️ Used | $350 | ✅ 78 pts | [link] |
| 3 | Ektorp 2-seat | 🟡 IKEA | 🆕 New | $649 | ✅ 74 pts | [link] |

## Coffee Table TOP 3
...
```

---

## Quick Filter Shortcuts

Recognized automatically when the user says:

```
"within budget" → exclude over-budget
"used only" → condition = used
"new only" → condition = new
"IKEA only" → platform = IKEA
"drop Facebook" → exclude Facebook
"good matches only" → style_match >= 70
"cheapest" → sort price ASC, top 1 per category
"recommend something" → style_match DESC, top 3 per category
"living room only" → room = living room
```

---

## When No Results Match

```
"Nothing matches those criteria.
Want me to loosen things up?
- +10% on budget → [N] more options
- Include used → [N] more options
- Lower style threshold to 60 → [N] more options

What do you want to do?"
```

---

## Output

```
Save location: /outputs/furniture_shortlist.md
+ Add "shortlisted" tag in the master list
```
