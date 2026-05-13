---
name: style-analysis
description: Analyzes the interior style from photos or keywords provided by the user, and generates a checklist of needed furniture.
trigger: After intake is complete, or when the user provides photos/keywords
---

# 02 — Style Analysis

## When does it run?
- Immediately after `01_intake` completes
- When the user provides additional photos/references
- When the user wants to change the style

---

## Step 1: Process Inputs

### 1-A. Photo Analysis (when an image is attached)

Claude analyzes the image directly:

**Information to extract:**
```
- Color palette: 3–5 main colors (e.g. cream white, oak brown, mustard)
- Materials: solid wood / fabric / metal / marble / rattan / glass / leather
- Mood tags: minimal / cozy / industrial / bohemian / clean / layered
- Style category: Nordic / Japanese / Mid-Century Modern / Contemporary / French / Luxury
- Lighting feel: warm / cool / natural-light dominant / accent lighting
- Visible furniture: list out sofa, coffee table, rug, shelves, lighting, bed, etc.
```

### 1-B. OCR (photos with embedded text — e.g. magazines, catalogs)
```
Extract text from the image:
- Brand names, product names, price info → use directly in research
- Style description text → extract style tags
```

### 1-C. Keyword Analysis (text only)
```
Parse user input:
"Nordic style, lots of solid wood, bright feel, but not too minimal"
→ tags: [nordic, warm-wood, bright, layered, cozy-minimal]
```

### 1-D. Pinterest/Instagram Links
```
Fetch the link → analyze the image (same as 1-A)
If multiple images: extract the common pattern
```

---

## Step 2: Build Style Profile

```
style_profile = {
  primary_style: "Nordic Cozy",       # main style
  secondary_style: "Mid-Century Warm", # sub-style (if any)
  
  color_palette: {
    dominant: ["cream white", "oak brown", "sage green"],
    accent: ["mustard", "terracotta"]
  },
  
  materials: ["solid wood", "linen fabric", "ceramic"],
  avoid_materials: ["chrome metal", "cold glass"],
  
  mood_tags: ["cozy", "warm", "natural", "uncluttered"],
  
  search_keywords_en: [
    "scandinavian", "nordic", "warm wood", "linen", "hygge",
    "natural materials", "cozy modern"
  ]
}
```

---

## Step 3: Build the Furniture Checklist

### Identify what each room needs

**Method 1 — Extract from photos:**
Furniture visible in the photos → "Want me to find all of these?" confirmation

**Method 2 — Suggest a default list per room type:**

```
Living Room defaults:
□ Sofa (main)
□ Coffee table
□ TV unit / sideboard
□ Rug
□ Lighting (floor or pendant)
□ Side table
□ Shelving / bookcase (optional)
□ Cushions, decor (optional)

Bedroom defaults:
□ Bed frame
□ Mattress
□ Nightstands (x2)
□ Dresser / wardrobe
□ Lighting (bedside)
□ Mirror (optional)
□ Rug (optional)

Home Office:
□ Desk
□ Chair
□ Shelving / bookcase
□ Lighting (desk lamp)
□ Storage (optional)

Dining:
□ Table
□ Chairs (x4–6)
□ Sideboard / buffet (optional)
□ Lighting (pendant)
```

→ Confirm with the user and finalize the checklist

---

## Step 4: Output Summary

```
"Analysis done! Here's your style:

🎨 Main style: Nordic Cozy + warm wood tones
🎨 Colors: cream white / oak / sage green / mustard accent
🪵 Materials: solid wood, linen, ceramic (I'll avoid cold metals)

📋 Furniture to find:
Living room: sofa, coffee table, TV unit, rug, floor lamp
Bedroom: bed frame, nightstands x2, dresser

Look right? Anything to add or drop?
→ If not, I'll kick off research!"
```

→ Once confirmed, call `03_research.md`
Pass styles and checklist to research
