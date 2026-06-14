# FlashMD — Format Specification v0.3

> Part of the **LearnSpec** suite  
> Status: Draft — June 14, 2026

---

## Core Principle

FlashMD is the **flashcard format** of the LearnSpec suite. It enables the creation of front/back cards designed for spaced-repetition review. FlashMD is designed to be simple to read, simple to generate with an LLM, and usable without any tooling.

FlashMD is a **review format**: it is consumed in a context separate from the lesson (review session, notification, flashcard mode), never embedded inline within a LearnMD. TrackMD orchestrates the relationship between a LearnMD and its associated FlashMD files.

FlashMD may reference visual resources via `!ref` + `media:slug`, and may point a card back to the lesson that explains it via the `lesson:` reference (v0.3), but imports no other LearnSpec format.

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
| 2 | Per-card fields, MediaMD references, front variants, lesson references | Per-card tags, images, alternative phrasings, links back to the source lesson |

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
lesson: ./01-fondations.learn.md       # optional — default source lesson for cards (v0.3)
created: 2026-05-10                    # optional — ISO 8601
updated: 2026-05-10                    # optional — ISO 8601
---
```

### FlashMD-Specific Fields

| Field | Status | Type | Description |
|---|---|---|---|
| `new_per_day` | Optional | integer | Number of new cards to introduce per day in spaced repetition. Indicative — the player may ignore or override it. |
| `lesson` | Optional | string | Deck-level default lesson reference (v0.3). A relative path to a `.learn.md` file, optionally with a `#section` anchor. Cards inherit it; see [Lesson References](#lesson-references-level-2). |

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
| `lesson` | Optional | string | Reference to the lesson section that explains this card (v0.3). See [Lesson References](#lesson-references-level-2). |

---

## Lesson References (Level 2)

A card may point back to the **lesson that explains it**, so a learner can jump from a card to the underlying course — typically once the answer has been revealed. The reference uses **hyperlink-style resolution**: the same mental model as an HTML/Markdown link.

### Deck-level default + per-card override

Set a default lesson once in the frontmatter, then let each card override it or pin a section:

| `lesson:` value | Resolves to | Like an HTML link… |
|---|---|---|
| *(absent)* | The deck-level default lesson, at its top | implicit link |
| `#section-anchor` | The **deck default lesson**, at that section | `<a href="#frag">` |
| `./other.learn.md` | **Another lesson**, at its top | `<a href="page.html">` |
| `./other.learn.md#anchor` | Another lesson, at that section | `<a href="page.html#frag">` |

The reference value is a relative path to a `.learn.md` file, optionally suffixed with `#<anchor>`. The anchor identifies a section of the target lesson: it resolves against a LearnMD heading slug or an explicit `!checkpoint` id. The value carries its own path, so **no `!ref` declaration is required** (unlike `media:slug`, whose slug is opaque).

### One lesson = one deck (common case)

````markdown
---
title: "Git — Fondations · Flashcards"
lang: fr
lesson: ./01-fondations.learn.md   # written once
---

# Git — Fondations

```flash id:index-role lesson:#role-de-lindex
À quoi sert la zone d'index (staging) ?
---
La zone de préparation du prochain commit.
```

```flash id:commit-def
Qu'est-ce qu'un commit ?
---
Un instantané versionné de l'arbre de travail.
```
````

The first card points at a **section** of the default lesson; the second, with no `lesson:`, links to the **top** of the same lesson.

### N lessons = one deck (cross-cutting deck)

````markdown
---
title: "Git — Révision générale · Flashcards"
lang: fr
# no deck-level `lesson:` — each card names its own source
---

```flash id:index-role lesson:./01-fondations.learn.md#role-de-lindex
À quoi sert la zone d'index ?
---
La zone de préparation du prochain commit.
```

```flash id:rebase lesson:./04-branches.learn.md#rebase
Différence entre rebase et merge ?
---
Rebase réécrit l'historique ; merge crée un commit de fusion.
```
````

Same attribute, same grammar — only the frontmatter default disappears. A deck may also **mix** a default with a few cards that point elsewhere.

### Graceful Degradation

In a standard Markdown reader, `lesson:` is plain text on the opening fence line of the card, like `id:` or `tags:` — readable and unobtrusive.

### Player Behaviour

| Aspect | Specification |
|---|---|
| Affordance | The player SHOULD surface the reference as a "back to the lesson" link, typically **after the answer is revealed**. |
| Track context | When the target lesson belongs to a track, the player SHOULD open it **within that track's context** (so prev/next and progress stay intact), falling back to the standalone lesson view otherwise. |
| Navigation | The player MAY navigate in place (same view) rather than a new tab; if it does, it SHOULD make returning to the review session obvious and preserve the learner's place in the queue. |
| Anchor resolution | The `#anchor` SHOULD resolve to a stable identifier first (a LearnMD `!checkpoint` id or an explicit heading anchor) and fall back to the heading slug. |

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
| `lesson` | Optional | Deck-level default lesson reference (`.learn.md` path, optional `#anchor`) — v0.3 |

### Block Attributes

| Attribute | Status | Description |
|---|---|---|
| `id` | **Required** | Unique slug within the file |
| `tags` | Optional | Card-specific tags |
| `hint` | Optional | Hint displayed on demand |
| `lesson` | Optional | Lesson reference for this card — `#anchor`, `path.learn.md`, or `path.learn.md#anchor` (v0.3) |

---

## Interoperability

| Mechanism | Support |
|---|---|
| `!ref ./media.media.md` | ✅ — for images within cards |
| `!ref ./glossary.glossary.md` | ✅ — for term highlighting |
| `lesson:` reference to a `.learn.md` section | ✅ — navigation only; resolves a path, declares no dependency (v0.3) |
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
| `lesson:` value whose path does not end in `.learn.md` | Error |
| Card `lesson:#anchor` (anchor-only) with no file-level `lesson:` default to resolve it | Error |
| `lesson:` anchor not found in the target lesson | Warning — checked by the collection-level validator (cross-file) |
| Variant-specific rules | See [Front Variants — Validation Rules](#validation-rules-additional) |

### Strict Mode (`--strict`)

All warnings are promoted to errors.

---

*Released under the MIT License. Copyright © 2024-present LearnSpec Contributors*
