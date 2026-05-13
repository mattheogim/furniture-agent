---
name: organization
description: Organizes validated furniture results into a per-furniture view (default) and per-room view. Budget numbers are managed only in 07_budget and referenced here.
trigger: Runs automatically after validator completes
---

# 05 — Organization: Master List

## When does it run?
- Immediately after `04_validator`
- When the user asks to "organize it"
- When new items are added and the list needs updating

---

## Master List Structure

**Supports two views:**
- 🛋️ **Per-furniture view** (default) — sofa → compare all options (the core requirement)
- 🏠 **Per-room view** — grouped by living room/bedroom

### Main: Per-furniture View (default output)

```markdown
# 🛋️ Furniture Shopping Master List
📍 Vancouver, BC | 💱 CAD | Updated: 2026-03-12

---

## Sofa (Living Room)
Max price: $1,000 | Options found: 4

| Rank | Product | Platform | Condition | Price | Style | Link | Notes |
|------|---------|----------|-----------|-------|-------|------|-------|
| ⭐1 | KIVIK 3-seat beige | 🟡 IKEA | 🆕 New | $799 | 92 pts | [link] | In stock at Coquitlam |
| ✅2 | Used beige sectional | 🔵 FB | ♻️ Used | $350 | 78 pts | [link] | East Van pickup |
| ✅3 | Ektorp 2-seat | 🟡 IKEA | 🆕 New | $649 | 74 pts | [link] | |
| ✅4 | Copen sofa | 🟠 Structube | 🆕 New | $899 | 70 pts | [link] | |

---

## Coffee Table (Living Room)
Max price: $300 | Options found: 3

| Rank | Product | Platform | Condition | Price | Style | Link | Notes |
|------|---------|----------|-----------|-------|-------|------|-------|
| ⭐1 | LISTERBY oak | 🟡 IKEA | 🆕 New | $229 | 88 pts | [link] | |
| ✅2 | Oak round table | 🔵 FB | ♻️ Used | $80 | 71 pts | [link] | East Van pickup |
| ✅3 | Renne table | 🟠 Structube | 🆕 New | $279 | 68 pts | [link] | |

---

## Bed Frame (Bedroom)
Max price: $800 | Options found: 5
...
```

### Secondary: Per-room View

Generated when the user asks "show me by room."

```markdown
## 🏠 Living Room
[Sofa top pick + coffee table top pick + ... best pick per furniture item]
Budget status: → see 07_budget.md
```

**⚠️ Never compute budget figures here → 07_budget is the single source of truth**

---

## Section Composition Rules

### At Least 2 Options per Furniture Item
```
Ranking: style_match_score DESC, then price ASC
Display:
⭐ Rank 1 = Best Match
✅ Ranks 2–3 = Good Options
```

### Platform Badges
```
🔵 Facebook Marketplace (used)
🟡 IKEA
🟠 Structube
🔴 Wayfair
🟣 Designer brand
```

### Condition Markers
```
🆕 New
♻️ Used + year/condition note
⚠️ Warning flag (suspicious price, unverified link, etc.)
```

---

## Summary Section (top of file)

```markdown
# 📊 Overall Summary

| Furniture | Room | Max Price | Options Found | Top Pick Price |
|-----------|------|-----------|---------------|----------------|
| Sofa | Living room | $1,000 | 4 | $799 (IKEA) |
| Coffee table | Living room | $300 | 3 | $229 (IKEA) |
| Bed frame | Bedroom | $800 | 5 | $350 (FB used) |
| Nightstand | Bedroom | $150 | 2 | $99 (IKEA) |

⚠️ Couldn't find: bedroom mirror → needs re-search
```

---

## Update Rules

- New item added → add a row to the relevant furniture section + re-sort ranks
- Purchase confirmed → ~~strikethrough~~ + ✔️ mark (notify 07_budget)
- Item removed → delete + record reason
- Price changed → log the update date

---

## File Output

```
Save location: /outputs/furniture_master_list.md
```

---

## On Completion

```
"Master list ready!
[N] furniture categories, [M] total options organized.

Want to filter or sort? Or check the budget?"
```

→ Call `06_filter-sort.md` or `07_budget.md`
