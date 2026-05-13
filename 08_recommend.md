---
name: recommend
description: Proposes additional recommendations on top of the shortlist. Includes items the user might have missed, pieces needed to complete the style, and in-budget upgrade options.
trigger: After filter-sort completes, or when the user asks "anything else?"
---

# 08 — Additional Recommendations

## When does it run?
- After `06_filter-sort` (auto or on request)
- When the user asks "something different" / "recommend more"
- When the user isn't happy with results for a particular item

---

## Recommendation Types

### Type 1 — Style Completion (Missing Pieces)

```
Analyze the current shortlist and suggest items missing from a complete style:

Example:
"For a Nordic-cozy style, these would round it out:
🌿 Plants + planter → completes the mood
🕯️ Candle holders or accent lighting
📚 Wood wall shelves
🛁 Linen cushion / throw set

Want me to find them?"
```

---

### Type 2 — Budget Alternatives (Swap)

```
When the current pick is over-budget or unloved:

"If the sofa budget is tight, here are alternatives:

💰 Cheaper new: Article Sven ($599, good reviews)
♻️ Used alternatives: a few similar fabric sofas on FB for $250–400
🔄 IKEA → Structube Agathe (similar fabric, a bit more character)"
```

---

### Type 3 — Upgrades (When Budget Allows)

```
When the budget has room:

"$720 left in bedroom. How about these upgrades?

✨ Mattress: Endy Queen at Sleep Country, $975
  → memory foam over basic spring, Canadian brand
✨ Nightstands: Article Culla ($199×2 = $398)
  → much nicer wood texture than IKEA
✨ Bed frame upgrade: Structube Lenia ($799)
  → thicker headboard, more premium feel"
```

---

### Type 4 — Unique / Designer Recommendations

```
Condition: when the user mentions "something unique" or "designer brands"
Or auto-suggested when budget has 20%+ remaining

"Having one unique piece really lifts a space:

🟣 EQ3 (Canadian brand, Vancouver showroom):
  - Nera lounge chair: $899 — Nordic feel, very Instagrammable
  
🟣 Article (Vancouver-based D2C):
  - Sven sofa: $1,299 — favored by many interior bloggers
  - great quality for the price
  
🟣 Tuck Studio (Vancouver local):
  - handcrafted solid-wood furniture, made-to-order available"
```

---

### Type 5 — "How about this?" Style

```
Asking within the user's existing boundaries:

"Thought you might like this — what do you think?

[Item photo or link]
→ Has the warm wood tone you want, but thinner legs
  so the space feels a bit lighter.

Sound good? I'll find more if so."
```

---

## Recommendation Boundaries

```
Never cross these lines:
- "No metal" → don't recommend metal items
- "No used" → don't recommend used items
- 30%+ over budget → always ask first

Free to explore within bounds:
- Stylistic expansion (e.g. cozy → with a bohemian touch)
- Brand variety
- Suggest additional furniture categories (with user agreement)
```

---

## Recommendation Output Format

```markdown
# 💡 Additional Recommendations

## Pieces to complete the style
[Type 1 items]

## Budget-friendly alternatives
[Type 2 items]

## If you've got room in the budget
[Type 3/4 items]
```

---

## Collect Feedback

After the recommendations:
```
"Anything catch your eye?
→ If yes: I'll add it to the master list
→ Want me to look for more in this direction?
→ Or try a different style?"
```

→ Feed feedback into `03_research.md` re-run (allow up to 2 loops)
→ If still unhappy after 2 loops → suggest re-running `02_style-analysis.md` to redefine the style
→ Never auto-loop without explicit user approval
