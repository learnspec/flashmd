# Changelog

All notable changes to the FlashMD specification are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

---

## [0.2] — 2026-05-28

### Added
- **Front variants**: a single card may declare multiple front phrasings sharing the same back, separated by `={3,}` on its own line. The player rotates through variants at presentation time; FSRS state remains attached to the card `id`, not the variant. Rationale: counter cue-dependency / surface memorisation while preserving a single mnemonic object per concept.
- Validation rules for variants: at least one front before the first `===`, no empty variants, no `===` after the front/back `---` separator, duplicate variants flagged as warning.

### Notes
- Backwards-compatible: a card without `===` is a single-variant card and parses identically to v0.1.

---

## [0.1] — 2026-05-10

### Added
- Initial draft of the FlashMD specification as part of the LearnSpec suite.
