# Changelog

All notable changes to `brand-spec` are documented here. The schema
follows semver: minor bumps are additive (no breaking changes to
prior-version brands); major bumps may tighten or rename fields.

## [1.2.1] — 2026-05-09

Two coupled additive conventions on the assets layer, surfaced by recent
brand work:

1. **Asset format preference** — codifies the framing landed in
   `lean-media-brand` PR #6: PNG-only is operational reality for many
   brands; auto-tracing PNG to SVG is wrong for typographic wordmarks.
2. **Wordmark structured metadata** — closes the gap that surfaced when
   `gramatr-brand` needed a slot to capture the canonical glyph
   sequence (the macron over the 'a' in `grāmatr`), the ASCII fallback
   (`gramatr` for CLI / URLs / package names), and the typeface used.

Semver justification: patch bump. Both additions are optional. No
required fields added. No enum tightened. No existing field renamed or
removed. v1.2 brands (NEXT90, gramatr-brand, lean-media-brand) validate
cleanly against v1.2.1 with zero changes.

### Added

- **Asset format preference convention.** Documented in the empty-brand
  `assets/_manifest.md`, the `assets/` layer description in
  `brand.yaml`, and the README. PNG (or other raster) is accepted for
  any logo file; SVG is preferred for future additions or replacements
  when vector source becomes available. Brands MUST NOT auto-trace PNG
  to SVG for typographic wordmarks. Recommended convention; not
  enforced by the validator.

- **Wordmark structured metadata.** Optional `wordmark:` frontmatter
  block on `assets/_manifest.md`:
    - `text` — canonical glyph sequence (required when block is present).
    - `text_ascii` — ASCII fallback (required when `text` contains
      non-ASCII characters; optional otherwise).
    - `typeface` — font family the wordmark is set in (optional).
    - `weight` — numeric font weight (optional).
    - `case` — `lowercase` | `uppercase` | `titlecase` | `mixed`
      (optional).
    - `glyph_notes` — human/agent-readable notes on special characters
      (optional).
  Precedence: `text` is canonical and SHOULD be used wherever the
  consuming surface can render it. `text_ascii` is the fallback ONLY
  when the surface cannot render the canonical glyphs.

- **README:** new "What's new in v1.2.1" section and a "Wordmark
  guidance for AI agents" subsection explaining how an agent should
  consume the wordmark metadata when generating brand-bearing
  artifacts.

### Backward compatibility

- v1.2 brands (NEXT90, gramatr-brand, lean-media-brand) validate
  cleanly against v1.2.1 with zero modifications. No required fields
  added; no fields renamed or removed; no enums tightened.
- The `wordmark:` block is optional and top-level in
  `assets/_manifest.md` frontmatter. Brands without a wordmark
  (icon-only or no logo) omit it cleanly.
- The format-preference convention is recommended prose, not a
  validator-enforced rule.

### Migration notes

None. Existing brands work unchanged. Brands MAY adopt the new
conventions opportunistically:
- Add the `wordmark:` block when capturing canonical/ASCII spellings
  becomes useful (especially for brands with non-ASCII wordmarks).
- Adopt the format-preference framing in their own `assets/_manifest.md`
  when revisiting asset documentation.

### Validator

The reference validator (`gramatr/brand-spec-validator`) pins to v1.2
today via its vendored `vendor/brand-spec/brand.yaml`. The validator
will pick up v1.2.1 on the next maintainer-triggered spec-sync run; no
validator code changes are required (additive-only schema).

## [1.2.0] — 2026-05-09

Surfaced by friction items hit while building the second brand
(`gramatr/gramatr-brand`) on top of v1.1. Closes four schema gaps
before the third brand inherits the same problems. All changes are
additive — v1.1 brands validate cleanly against v1.2 with zero
modifications.

### Added

- **Multi-register voice (Fix #2).** New optional `voice/` directory
  pattern: `voice/registers/{register-slug}.md` plus a
  `voice/README.md` (or `voice/_index.md`) index. Each register file
  requires `register:` and `applies_to:` frontmatter; other voice
  frontmatter follows the v1.1 contract. The single-register
  `voice.md` at the brand root remains the default. A brand uses one
  pattern OR the other; mixing is invalid.
  - Validation rules: `voice-required` (updated to accept either
    pattern), `voice-pattern-exclusive`,
    `voice-multi-register-index`.
  - New template: `templates/multi-register-voice-example/`.

- **Source authority metadata (Fix #4).** Optional `source_authority`
  frontmatter on any file:
    - `status`: `canonical` (default) | `mirror` | `historical` |
      `fixture-only`
    - `upstream`: required when status is `mirror` or `historical`
    - `notes`: required when status is `historical` or `fixture-only`
  - Validation rules: `source-authority-upstream-required`,
    `source-authority-notes-required`.

- **File-level visibility classification (Fix #6).** Optional
  `visibility` frontmatter on any file: `public` (default) |
  `customer` | `internal` | `sales-enablement`. Tools generating
  public artifacts MUST filter out files where `visibility` ≠
  `public`.
  - Validation rules: `visibility-default-public`,
    `visibility-audience-internal-backcompat`.
  - Template change: `templates/empty-brand/proof/competitive.md` now
    uses `visibility: sales-enablement` instead of `audience: internal`.

- **Data file lineage (Fix #8).** Optional `refresh:` sub-block under
  `source_authority` (only meaningful when `status: mirror`):
    - `cadence`: `manual` | `on-deploy` | `weekly` | `monthly` |
      `on-source-change`
    - `last_synced`: `YYYY-MM-DD` (required)
    - `sync_method`: free text (optional)
  - Validation rule: `source-authority-mirror-freshness-warn` (warns
    when `last_synced` is older than the declared cadence implies).

### Changed

- `proof/competitive.md` schema: `audience` is now deprecated in favor
  of `visibility`. Validators treat legacy `audience: internal` as
  `visibility: internal` and emit a migration warning.
- README adds a "What's new in v1.2" section above
  "Required vs. Recommended vs. Optional".

### Backward compatibility

- v1.1 brands (NEXT90, gramatr-brand) validate cleanly against v1.2
  with zero changes. No required fields were added; no v1.1 fields
  were tightened or removed. The `audience: internal` field on
  `proof/competitive.md` continues to work, with a warning.

## [1.1.0]

- Widen `status` enum, require `last_updated` on freshness-sensitive
  layers, add product body sections, fix `ui-tokens` `depends-on` type.

## [1.0.0]

- Initial brand-spec v1: schema, conventions, agent-context layer,
  empty-brand template.
