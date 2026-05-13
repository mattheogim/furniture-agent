---
name: budget
description: Sets per-room / per-furniture budgets, and tracks status against research results.
trigger: When intake provides budget info, or when the user asks "how's the budget?"
---

# 07 — Budget Tracking

## When does it run?
- Initialize right after budget info is captured in `01_intake`
- Auto-update after `05_organization` completes
- When the user asks "what's my budget looking like?"
- When items are selected/removed from the shortlist

---

## Initialize Budget

Set up using info from `01_intake`:

```
budget_tracker = {
  total: {
    limit_cad: 3500,
    estimated_low: 0,    # lowest total based on used items
    estimated_high: 0,   # highest total based on new items
    committed: 0,        # what the user has confirmed they're buying
  },
  per_furniture: {
    "living_sofa": { limit: 1000, current_best: 799, status: "candidate" },
    "living_table": { limit: 300, current_best: 229, status: "candidate" },
    "bedroom_frame": { limit: 800, current_best: 350, status: "candidate" },
    "office_desk": { limit: 250, current_best: 0, status: "pending" }
  }
}
```

---

## Budget Dashboard

```markdown
# 💰 Budget Dashboard
Updated: 2026-03-12

## Overall Budget
| | Amount |
|--|--------|
| Total budget | $3,500 CAD |
| Lowest estimate (used-heavy) | $1,850 CAD |
| Highest estimate (new-heavy) | $3,980 CAD |
| Confirmed purchases | $0 CAD |
| Remaining budget | $3,500 CAD |

⚠️ Going new-heavy will exceed the budget by ~$480

---

## Per-Furniture Budget
Compares the overall budget list against the price of the current top (recommended) option.

| Category / Furniture | Pick | Estimated / Current Price | Per-furniture Budget | Status |
|----------------------|------|---------------------------|----------------------|--------|
| [Living] Sofa | KIVIK beige | $799 | $1,000 | 🟡 Candidate |
| [Living] Coffee table | LISTERBY | $229 | $300 | 🟡 Candidate |
| [Living] TV unit | Copen Stand | $599 | $400 | ⚠️ Over |
| [Bedroom] Frame | FB Used Oak | $350 | $800 | 🟡 Candidate |
| [Bedroom] Chair | - | $0 | $150 | Pending |
| **Subtotal (candidates)** | | **$1,977** | **$2,650** | |

Progress: ▓▓▓▓▓▓▓▓▓░ ~56% of total budget projected
```

---

## Budget Status Alerts

### Over-Budget Warning
```
"⚠️ Living room budget close to over!
Current selection: $1,950 / budget $2,000
Remaining: $50 — lighting might be hard to fit in.

Options:
- Switch the sofa to used → $799 → $350 (frees up $649 in living room)
- Coffee table to FB used → $229 → $80 (frees up $149)"
```

### Plenty of Room
```
"✅ Bedroom budget has room: $780 of $1,500 used ($720 left)
Want to upgrade to a premium mattress or dresser?"
```

---

## Confirm Purchase

When the user says "I'll buy this" / "lock this in":

```
The item:
- status: "committed"
- subtracted from the budget
- marked with ✔️ in the master list

"✅ KIVIK sofa ($799) confirmed!
Remaining living room budget: $1,201"
```

---

## Budget Adjustments

When the user updates the budget:
```
"Bumping living room budget to $2,500.
That's $500 more — anything else you want me to look for?
Or want me to surface premium upgrades?"
```

→ Automatically re-runs `06_filter-sort.md` against the new budget

---

## Output

```
Save location: /outputs/furniture_budget.md
Budget column in the master list updates automatically
```
