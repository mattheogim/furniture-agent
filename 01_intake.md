---
name: intake
description: The first-stage skill that collects interior taste, room information, and budget from the user.
trigger: At the start of a conversation, or when the user asks to add a new room/new furniture
---

# 01 — Intake: Collecting User Information

## When does it run?
- At the agent's first start
- When the user adds a new room
- When the user mentions "start over" or a new project

---

## Step 1: Detect Input Type

Possible user inputs:

| Type | Detection | Next Step |
|------|-----------|-----------|
| 📸 Interior photo | Image attachment | → `02_style-analysis.md` |
| 💬 Keywords/description | Text (e.g. "Nordic style, solid wood") | → `02_style-analysis.md` |
| 🔗 Link (Pinterest, Instagram, etc.) | URL included | → URL fetch then `02_style-analysis.md` |
| 📋 Direct furniture list | "I need a sofa, desk, and bed" | → Build checklist directly |

---

## Step 2: Collect Required Information

If anything is still missing, ask the user. **Don't ask everything at once — work it into the conversation naturally.**

### 2-1. Room List
```
"Which rooms are you decorating? (e.g. living room, bedroom, home office, dining)"
```
→ Tracked separately per room

### 2-2. Style (when no photo is provided)
```
"What kind of feeling are you going for?
Examples: Nordic/Minimalist, Industrial, Bohemian, Modern, Cozy/Scandi, Luxury"
→ Multiple options are fine
```

### 2-3. Budget
```
"What's your budget?
- A total overall budget? (e.g. $5,000 CAD)
- Or do you want to set it per room?"
```
→ Pass to `07_budget.md` for initialization

### 2-4. New vs. Used Preference
```
"Is used furniture OK? Should I also look on places like Facebook Marketplace?"
→ Default: both are OK
```

### 2-5. Facebook Marketplace Usage
```
If used is OK:
"To search Facebook Marketplace I need to log in.
Can you share your username/password? (used in this session only, never stored)"
→ If provided: credentials kept in session only
→ If declined: proceed without FB
```

### 2-5. Existing Furniture (optional)
```
"Do you already have any furniture? I can find pieces that match."
→ Record existing furniture list + colors/style
```

---

## Step 3: Build User Profile

Once collection is complete, save to the internal data structure:

```
user_profile = {
  rooms: [
    {
      name: "living room",
      style_tags: [],           # filled in by 02_style-analysis
      budget_cad: 2000,
      furniture_checklist: [], # filled in by 02_style-analysis
      priority: 1
    }
  ],
  global_style: {
    tags: [],
    references: [],            # original photos/links
    existing_furniture: []
  },
  preferences: {
    facebook_enabled: false,
    facebook_credentials: null  # temporary, session-only
  },
  budget: {
    total_cad: null,
    per_room: {}
  }
}
```

---

## Step 4: Confirmation Summary

```
"Okay, here's what I've got:

🏠 Rooms to decorate: living room, bedroom
🎨 Style: Nordic minimalist + warm wood tones
💰 Budget: living room $2,000 / bedroom $1,500 / total $3,500 CAD
🛍️ Scope: new + used both OK
📍 Location: Vancouver, BC

Does that look right? If so, I'll start the style analysis now!"
```

→ Once confirmed, call `02_style-analysis.md`
