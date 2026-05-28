# FlashMD — Format Specification v0.2

> Part of the **LearnSpec** suite  
> Status: Draft — May 28, 2026

---

## Core Principle

FlashMD is the **flashcard format** of the LearnSpec suite. It enables the creation of front/back cards designed for spaced-repetition review. FlashMD is designed to be simple to read, simple to generate with an LLM, and usable without any tooling.

FlashMD is a **review format**: it is consumed in a context separate from the lesson (review session, notification, flashcard mode), never embedded inline within a LearnMD. TrackMD orchestrates the relationship between a LearnMD and its associated FlashMD files.

FlashMD may reference visual resources via `!ref` + `media:slug`, but imports no other LearnSpec format.

| Principle | Description |
|---|---|
| **Markdown-first** | A `.flash.md` file is valid Markdown readable in any editor |
| **File-native** | All data lives in the file — no database required |
| **Graceful degradation** | Each card is readable as a code block in any standard reader |
| **LaTeX from Level 0** | Mathematical formulas are available without frontmatter |
| **AI-native** | Generatable and consumable by an LLM without specific tooling |

---

## Format Levels

| Level | Mechanism | Purpose |
|---|---|---|
| 0 | ` ```flash ` fenced block with front/back | Minimal cards, readable everywhere |
| 1 | YAML frontmatter | File metadata, language, spaced repetition settings |
| 2 | Per-card fields, MediaMD references, front variants | Per-card tags, images, alternative phrasings |

Each level is a strict superset of the previous one.

---

## Basic Syntax (Level 0)

### Card Structure

```
```flash id:slug
[front content — text, LaTeX, inline Markdown]
---
[back content — text, LaTeX, inline Markdown]
```
```

- `id`: unique identifier for the card within the file (slug, lowercase, hyphens)
- `---`: front/back separator — appears exactly once per block
- Both sides support **inline Markdown** (bold, italic, `code`, links) and **inline LaTeX** (`$...$`) and **block LaTeX** (`$$...$$`)
- Both sides may be multi-line
- The front may declare multiple variants — see [Front Variants (Level 2)](#front-variants-level-2)

### Minimal Example

````markdown
# Cell Biology — Flashcards

```flash id:photosynthesis
What is photosynthesis?
---
The process by which plants convert sunlight into chemical energy, using $CO_2$ and $H_2O$.
```

```flash id:mitosis-phases
What are the 4 phases of mitosis?
---
**Prophase** → **Metaphase** → **Anaphase** → **Telophase**
```

```flash id:atp-formula
What is the chemical formula of ATP?
---
$$C_{10}H_{16}N_5O_{13}P_3$$

Adenosine **tri**phosphate — the energy currency of the cell.
```
````

### Graceful Degradation

In a standard Markdown reader (GitHub, Obsidian, VS Code), each card renders as a code block. The `---` separator is visible as plain text — the front and back are readable, in order.

---

## Frontmatter (Level 1)

```yaml
---
title: "Cell Biology — Flashcards"    # optional — inferred from first # H1
lang: en                               # REQUIRED — BCP-47 code
spec_version: "0.1"                    # optional
author: Jane Smith                     # optional
tags: [biology, cell, high-school]     # optional — file-level tags
new_per_day: 20                        # optional — new cards per day (player default)
created: 2026-05-10                    # optional — ISO 8601
updated: 2026-05-10                    # optional — ISO 8601
---
```

### FlashMD-Specific Fields

| Field | Status | Type | Description |
|---|---|---|---|
| `new_per_day` | Optional | integer | Number of new cards to introduce per day in spaced repetition. Indicative — the player may ignore or override it. |

---

## Per-Card Fields (Level 2)

Optional attributes may be added on the opening line of the block, after the `id`:

```
```flash id:slug tags:[tag1,tag2] hint:"Think about the membrane"
```

| Attribute | Status | Type | Description |
|---|---|---|---|
| `id` | **Required** | string | Unique identifier within the file. |
| `tags` | Optional | string[] | Card-specific tags. Added to the file-level tags from frontmatter. |
| `hint` | Optional | string | Hint displayed on demand before flipping the card. |

---

## Front Variants (Level 2)

A single card may declare **multiple front phrasings** sharing the same back. The player picks one at presentation time.

### Rationale

In classical spaced-repetition, a learner ends up recognising the *surface form* of a card rather than recalling the underlying concept (cue-dependency). Front variants break that surface memorisation: the concept is constant, the prompt is not. The FSRS scheduler still operates on a **single entry per `id`** — variants are alternative presentations of the same mnemonic object, not separate cards.

### Syntax

Front variants are separated by a line of **three or more equal signs** (`={3,}`) on its own line. The front/back separator (`---`) remains unique and marks the boundary between the last variant and the shared back.

````
```flash id:photosynthesis
What is photosynthesis?
===
How do plants convert sunlight into chemical energy?
===
Define photosynthesis.
---
The process by which plants convert sunlight into chemical energy, using $CO_2$ and $H_2O$.
```
````

- A card may declare **1 or more** front variants (a card with no `===` is a single-variant card — fully backwards-compatible with v0.1).
- Each variant supports the same inline Markdown and LaTeX as a regular front.
- Each variant may be multi-line.
- All variants share the **same back**, the same `id`, the same `tags`, the same `hint`, and the same FSRS state.

### Graceful Degradation

In a standard Markdown reader, a line of `===` under a text line renders as a setext H1 heading. The variants remain readable in plain order; the visual artefact is acceptable for fallback rendering.

### Player Behaviour

| Aspect | Specification |
|---|---|
| Tirage | The player SHOULD rotate through variants to ensure coverage over time rather than picking purely at random (e.g. round-robin with per-user memory of seen variants). |
| FSRS rating | Ratings are recorded against the card `id`, not the variant. A rating reflects the recall of the concept, with variant-induced noise treated as desirable difficulty (Bjork). |
| Hint | The `hint` attribute, if present, applies to all variants. |
| Reproducibility | The player MAY persist the variant index per session to allow a learner to re-read a card in the form they just saw. |

### Validation Rules (additional)

| Condition | Level |
|---|---|
| Card with `===` but no front before the first separator | Error |
| Empty variant (whitespace only between two `===` or between `===` and `---`) | Error |
| Duplicate variants within a card (after Markdown normalisation) | Warning |
| `===` appearing after the `---` separator | Error |

---

## Images in Cards (Level 2)

Images are referenced within card content via the standard `media:slug` syntax, with a fallback URL:

````markdown
!ref ./media-biology.media.md

```flash id:chloroplast
What does a chloroplast look like?
---
![Chloroplast diagram](media:chloroplast-diagram "https://upload.wikimedia.org/.../chloroplast.svg")

An oval organelle surrounded by a double membrane, containing thylakoids and stroma.
```
````

The `!ref` directive is placed at the top of the file, before the `flash` blocks. Card content may freely mix text, LaTeX, and images.

---

## Complete Example

````markdown
---
title: "Cell Biology — Flashcards"
lang: en
spec_version: "0.1"
tags: [biology, cell, high-school]
new_per_day: 15
---

# Cell Biology — Flashcards

!ref ./media-biology.media.md

```flash id:photosynthesis
What is photosynthesis?
---
The process by which plants convert sunlight into chemical energy, using $CO_2$ and $H_2O$.

$$6CO_2 + 6H_2O + \text{light} \rightarrow C_6H_{12}O_6 + 6O_2$$
```

```flash id:mitosis-phases tags:[cell-division]
What are the 4 phases of mitosis?
===
List the phases of mitosis, in order.
===
Mitosis proceeds through four stages — which ones?
---
**Prophase** → **Metaphase** → **Anaphase** → **Telophase**
```

```flash id:chloroplast tags:[organelle] hint:"Think about the green colour"
Which organelle is responsible for photosynthesis?
---
The **chloroplast**.

![Chloroplast diagram](media:chloroplast-diagram "https://upload.wikimedia.org/.../chloroplast.svg")
```

```flash id:dna-bases
What are the 4 nitrogenous bases of DNA?
---
- **Adenine** (A) — pairs with Thymine
- **Thymine** (T) — pairs with Adenine
- **Guanine** (G) — pairs with Cytosine
- **Cytosine** (C) — pairs with Guanine
```
````

---

## Field Reference

### Frontmatter — Universal LearnSpec Fields

| Field | Status | Description |
|---|---|---|
| `title` | Optional | File title — inferred from the first `# H1` if absent |
| `lang` | **Required** | BCP-47 code (`en`, `fr`, `en-US`…) |
| `spec_version` | Optional | Targeted spec version (`"0.1"`) |
| `author` | Optional | File author |
| `tags` | Optional | File-level tags |
| `created` | Optional | Creation date, ISO 8601 |
| `updated` | Optional | Last update date, ISO 8601 |

### Frontmatter — FlashMD-Specific Fields

| Field | Status | Description |
|---|---|---|
| `new_per_day` | Optional | New cards per day (spaced repetition) |

### Block Attributes

| Attribute | Status | Description |
|---|---|---|
| `id` | **Required** | Unique slug within the file |
| `tags` | Optional | Card-specific tags |
| `hint` | Optional | Hint displayed on demand |

---

## Interoperability

| Mechanism | Support |
|---|---|
| `!ref ./media.media.md` | ✅ — for images within cards |
| `!ref ./glossary.glossary.md` | ✅ — for term highlighting |
| `!import` | ❌ — FlashMD imports no other format |
| Imported by TrackMD via `!import` | ✅ |
| Imported by LearnMD via `!import` | ❌ — flashcards are a separate review mode |

---

## Validation

### Lenient Mode (default)

| Condition | Level |
|---|---|
| `lang` absent from frontmatter | Warning |
| `id` missing on a `flash` block | Error |
| Duplicate `id` within the file | Error |
| `---` separator missing from a `flash` block | Error |
| `---` separator appears more than once in a block | Error |
| Empty front | Error |
| Empty back | Error |
| `media:slug` without a matching `!ref` | Warning |
| Variant-specific rules | See [Front Variants — Validation Rules](#validation-rules-additional) |

### Strict Mode (`--strict`)

All warnings are promoted to errors.

---

*Released under the MIT License. Copyright © 2024-present LearnSpec Contributors*
