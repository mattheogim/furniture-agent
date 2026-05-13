---
name: furniture-agent
description: Furniture research agent for a Vancouver move. Taste analysis → broadest possible collection → Tinder-style feedback learning → recommendations that become more accurate over time.
---

# 🛋️ Furniture Finder Agent — Overall Flow

## Core Philosophy
```
Research = Collect as broadly as possible (budget ±30%)
Validator = Judgment + prioritization (minimize removal, just reorder)
Tinder UX = User reactions → Personalization learning
→ Gets progressively more accurate
```

---

## Skill Flow

```
[01_intake]
  Photos / keywords / room list / max price per furniture item
        ↓
[02_style-analysis] ←─── [02a_pinterest-analysis]
  Generate style tags + furniture checklist        Analyze Pinterest boards/pins/keywords
        ↓
[03_research]  ◄──────────────────────────────────┐
  Collect as broadly as possible across platforms │ Re-search if
  FB / IKEA / Structube / Wayfair / local brands  │ results are insufficient (skip duplicates)
        ↓                                         │
[04_validator] ──────────────────────────────────►┘
  Hard Filter (remove only Vancouver-unavailable / expired links)
  + Apply Personalization to compute scores
  + Priority sort (don't hide, just reorder)
  + Group duplicates (flag the cheaper one)
        ↓
[05_organization]
  Per-furniture view (default) + per-room view (on request)
  Master list sorted by score
        ↓
[TINDER UX] ◄──────────────────────────────────┐
  Show items (1–3 at a time)                   │
  YES / NO / MAYBE / Comment                   │
        ↓                                      │
[09_personalization] ──────────────────────────┘
  Analyze feedback + update taste profile
  → Feed back into next validator score
  → Adjust keywords on re-search
        ↓
[06_filter-sort]      [07_budget]      [08_recommend]
  Filter & sort        Budget tracking   Additional recommendations
  (on request)        (per room/item)   (empty categories,
                                        alternatives, upgrades)
```

---

## Skill File List

| File | Role |
|------|------|
| `01_intake.md` | Collect user information (rooms, max price per furniture item, FB login) |
| `02_style-analysis.md` | Photos/keywords → style tags + furniture checklist |
| `02a_pinterest-analysis.md` | Pinterest boards/pins/search → style analysis |
| `03_research.md` | Broadest possible collection across all platforms (±30% budget) |
| `04_validator.md` | Validation + scoring + priority sort + re-search request |
| `05_organization.md` | Generate per-furniture / per-room master list |
| `06_filter-sort.md` | Filter & sort (on request) |
| `07_budget.md` | Per-room / per-furniture budget tracking (single source) |
| `08_recommend.md` | Additional recommendations (empty categories, alternatives, upgrades) |
| `09_personalization.md` | Tinder feedback learning + taste profile management |

---

## Common Rules

- **Region**: Based in Vancouver, Canada (Vancouver, BC)
- **Currency**: CAD ($)
- **Language**: User speaks Korean / searches are in English
- **Budget**: Set max_price per furniture item / allow ±30% collection
- **Facebook**: Log in via computer use with user-provided credentials (session only, never stored)
- **Links required**: Every item must include a purchase/inquiry link
- **Source required**: Every item must list its platform
- **Duplicate handling**: Same product across multiple platforms → group + price compare

---

## Data Structure

```
session = {
  user_profile: {
    rooms: [{ name, furniture_checklist, budget_cad }],
    global_style: { tags, references },
    preferences: { facebook_enabled: bool, facebook_credentials: null }  # session only
  },

  personalization_profile: {
    liked: { materials, colors, styles, brands },
    disliked: { materials, colors, styles },
    signals: [...],
    inferred_rules: [...],
    stats: { total_shown, yes, no, maybe }
  },

  master_list: [
    {
      id, furniture_item, room, platform, condition,
      price_cad, link, final_score, rank_tier,
      flags, duplicate_group
    }
  ],

  budget_tracker: {
    per_furniture: {
      "living_sofa": { max_price: 1000, best_candidate_price: 799, status: "pending" },
      "bedroom_chair": { max_price: 150, best_candidate_price: 80, status: "committed" }
    },
    total: { budget, estimated_total }
  }
}
```
