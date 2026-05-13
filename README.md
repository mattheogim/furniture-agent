# Furniture Finder Agent

[![Live demo](https://img.shields.io/badge/live-furniture--agent.vercel.app-black?style=flat-square)](https://furniture-agent.vercel.app)

A furniture research agent built for a Vancouver move. It analyzes the user's taste, collects options as broadly as possible across every relevant platform, then uses Tinder-style feedback to learn what the user actually loves — so recommendations get more accurate with every reaction.

## What It Does

- Captures style preferences from photos, keywords, or Pinterest boards
- Searches Facebook Marketplace, IKEA Canada, Structube, Wayfair Canada, and Vancouver-local D2C brands (Article, EQ3, CB2, West Elm, Tuck Studio, Kits Interiors)
- Validates and ranks results without hiding them — the user always sees everything, just in the right order
- Tracks per-room and per-furniture budgets in CAD
- Learns from like/dislike/maybe/comment feedback and feeds that back into the next search and ranking pass

## Core Philosophy

```
Research = Collect as broadly as possible (budget ±30%)
Validator = Judgment + prioritization (minimize removal, just reorder)
Tinder UX = User reactions → Personalization learning
→ Gets progressively more accurate
```

## Architectural Flow

The pipeline runs through 9 composable skill files plus a Pinterest sub-skill:

```
intake → style-analysis → research → validator → organization
                ↑                                      ↓
        pinterest-analysis                         [Tinder UX]
                                                       ↓
                                              personalization
                                                       ↓
                                  filter-sort | budget | recommend
```

### How the Skills Compose

| File | Role |
|------|------|
| `01_intake.md` | Collect rooms, max price per furniture item, FB login |
| `02_style-analysis.md` | Photos/keywords → style tags + furniture checklist |
| `02a_pinterest-analysis.md` | Pinterest boards/pins/search → style analysis (feeds style-analysis) |
| `03_research.md` | Broadest possible collection across all platforms (±30% budget) |
| `04_validator.md` | Validate + score + priority sort + request re-search if thin |
| `05_organization.md` | Per-furniture (default) and per-room master lists |
| `06_filter-sort.md` | Filter & sort on user request |
| `07_budget.md` | Single source of truth for per-room / per-furniture budgets |
| `08_recommend.md` | Missing pieces, budget alternatives, upgrades, designer picks |
| `09_personalization.md` | Tinder feedback → taste profile → feeds validator & research |

### Feedback Loops

- **Validator → Research**: when results per furniture category are too thin (fewer than 2 at ⭐ or above, or fewer than 3 after hard filter), validator signals research to re-run with expanded layers (up to 3 attempts).
- **Personalization → Validator**: each YES/NO/MAYBE updates a taste profile that boosts or penalizes future scoring. After 2 NOs on the same trait it becomes a hard_exclude.
- **Personalization → Research**: on re-search, the latest preferences become keyword adjustments (added liked materials, excluded disliked ones).

## Common Rules

- **Region**: Vancouver, BC (Canada)
- **Currency**: CAD ($)
- **Language**: User speaks Korean, searches run in English
- **Budget tolerance**: ±30% per furniture item; >130% gets a -10 penalty rather than removal
- **Facebook**: logged in via computer use with user-provided credentials, session-only, never stored
- **Every item must include**: a working link and a clear platform source
- **Duplicates**: not removed — grouped, with the cheapest platform flagged

## Status

All 10 skill files are translated to English. The directory also contains:

- `prototype.html` — UI prototype for the Tinder-style feedback flow
- `photos/` — reference photos (not touched by this README)
- `market-radar/` — separate subproject (not touched by this README)

The skills are designed as sub-agents in a single agent pipeline. Each file is a self-contained spec with its own `trigger` and YAML frontmatter, so they can be wired into any agent runner that loads skill files by name.

## Entry Points

- Start at `00_AGENT_OVERVIEW.md` for the full flow diagram and data structures
- Each numbered file documents one skill's role, inputs, outputs, and handoff target
- `02a_pinterest-analysis.md` is a side-input source for style analysis, not a sequential step
