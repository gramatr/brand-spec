# Changelog

All notable changes to `brand-spec` are documented here. The schema
follows semver: minor bumps are additive (no breaking changes to
prior-version brands); major bumps may tighten or rename fields.

## [1.4.0] — 2026-05-10

Closes Phase 1 of the brand-spec coverage epic
[`gramatr/brand-spec#13`](https://github.com/gramatr/brand-spec/issues/13):
sub-issues
[#14](https://github.com/gramatr/brand-spec/issues/14) (journey schema
design),
[#15](https://github.com/gramatr/brand-spec/issues/15) (journey docs +
methodology Provenance), and
[#16](https://github.com/gramatr/brand-spec/issues/16)
(`methodology_provenance:` convention). Sub-issues #17 (legal/trademark
relocation) and #18 (lean-media journey validation) remain open and
will ship as separate follow-up PRs.

Semver justification: **MINOR bump (per `VERSIONING.md`).** A new
top-level layer is the canonical example of "a genuinely new capability
domain" called out in the policy as warranting a minor bump (the policy
gives `interactions/` and `taxonomies/` as the two named examples;
`journey/` is the same shape of change). The default-to-patch rule has
a documented exception for new layers; this bump invokes it explicitly.
No required field added, no enum tightened, no existing field renamed
or removed. v1.3.0 brands (NEXT90, gramatr-brand, lean-media-brand)
validate cleanly against v1.4 with zero changes — none of them
currently have a `journey/` directory and the layer is
`recommended: true, required: false`.

This release introduces the first methodology brought into brand-spec
with explicit, structured attribution. **KYKC (Know Your Customer)** and
**CognitiveJourney** were developed by Brian Handrigan at the grāmatr
digital marketing agency circa 2005 and refined across 20 years of
agency, adtech, and AI work. The grāmatr brand identity has been
continuous across that arc — same brand carrying the same methodology
through three industry eras, not a rebrand. Both methodologies are
open-sourced into the spec under MIT license; brands MAY adopt the
methodology and the attribution as-is. The
`methodology_provenance:` frontmatter convention sets the precedent for
how future contributed methodologies (voice-register patterns,
channel-guide templates, compliance-gate frameworks, etc.) get
attributed: credible originator, real history, no false claims.

### Added

- **`journey/` layer (issue #14, #15, Gap 10 from #12).** New top-level
  layer, `required: false, recommended: true`. Operationalizes the KYKC
  discipline by capturing the prospect's cognitive journey in ordered
  stages. Layer files:
    - `journey/_framework.md` — discipline-level documentation. Six
      recommended body sections: "The discipline (KYKC)", "The artifact
      (CognitiveJourney)", "Methodology origins", "Why this matters for
      AI search", "How to use this layer", "Relationship to other
      layers". Carries an optional `methodology_provenance:`
      frontmatter block attributing KYKC the discipline.
    - `journey/stages.md` — at-a-glance stage overview. Authoritative
      `stages:` frontmatter array of `{slug, label, order}` records.
      Body MAY include a tabular human-readable stage list. Carries an
      optional `methodology_provenance:` block attributing
      CognitiveJourney the artifact (co-equal with the discipline-level
      block — two methodologies, attributed distinctly).
    - `journey/stages/<stage-slug>.md` — per-stage detail files.
      Frontmatter captures `slug`, `label`, `order`, `description`,
      `triggers[]`, `hurdles[]`, `mental_vocabulary[]`,
      `questions_asked[]`, `content_goal`. These are the load-bearing
      fields AI tooling reads to generate on-stage content.
  The 5 canonical KYKC stages (`problem-recognition`,
  `information-seeking`, `option-evaluation`, `decision-validation`,
  `post-decision`) are the default; brands MAY rename, reorder, add, or
  omit (canonical-with-extension model).

- **`methodology_provenance:` frontmatter convention (issue #16).**
  Documented once under `conventions:` in `brand.yaml`, applicable to
  any markdown file that documents a named methodology. Sub-fields:
  `name` (required), `originator` (required), `developed_year`
  (required); `short_name`, `developed_at`, `refined_through`,
  `continuity_note`, `references`, `notes` all optional. Co-equal
  methodologies get separate blocks (not a merged one) — first
  demonstrated by the journey layer's separate blocks for KYKC the
  discipline and CognitiveJourney the artifact.

- **Validation rules (4 new):**
    - `journey-stages-monotonic-order` (error) — every stage MUST
      declare a non-negative integer `order`; values MUST be unique
      across the stage set.
    - `journey-stage-slug-resolves` (warn) — when a file outside the
      journey layer references a stage slug, the slug MUST resolve to
      an entry in `stages.md` or an existing
      `journey/stages/<slug>.md`. Warn-only in v1.4 because cross-layer
      reference syntax is not finalized; will tighten to error when
      journey-stage messaging variants land.
    - `journey-canonical-stage-warn` (warn) — when a brand renames a
      canonical KYKC stage slug, validators warn that downstream tools
      may not recognize the rename without an alias map. Soft signal,
      not blocking.
    - `methodology-provenance-required-when-present` (error) — when
      the block appears, `name`, `originator`, and `developed_year`
      MUST be present and non-empty.
    - `methodology-provenance-recommended-on-framework` (info) —
      framework files documenting a named methodology SHOULD declare
      the block when there is a credible originator.

- **`templates/journey-example/`** — worked example for a hypothetical
  Acme HR brand. Demonstrates the full layer shape: `_framework.md`
  with both `methodology_provenance:` blocks filled in (using the
  open-sourced KYKC attribution as the canonical example), `stages.md`
  with all 5 canonical stages, and 3 of 5 per-stage detail files
  (`problem-recognition`, `option-evaluation`, `post-decision`).
  Intentionally omits 2 per-stage files to demonstrate that per-stage
  detail files are recommended-not-required.

### Changed

- `contract_version` bumped to `1.4.0`.
- `README.md` updated: layer overview now lists the journey layer; new
  "What's new in v1.4" section documents the layer, the
  `methodology_provenance:` convention, and the open-source attribution
  of KYKC and CognitiveJourney.

### Out of scope (deferred)

The following are intentionally NOT in v1.4 and will land in later
phases of the #13 epic:

- **Journey-stage messaging variants** (e.g.,
  `messaging/for-{audience}-at-{stage}.md`). Phase 2 work; cross-layer
  reference syntax requires a brand to operationalize first
  (`lean-media-brand` is the canonical validation brand in #18).
- **KYKC compliance gates** (the 4 booleans Studio's MessageBrief
  uses today: `authenticClaims`, `questionsBeforeKeywords`,
  `capabilityNeedAlignment`, `cognitiveAuthority`, plus `eeatCompliance`).
  Phase 3 work; will likely live in `prompts/` or a new `compliance/`
  layer, not in `journey/`.
- **`evidence_required[]`** and **`transition_signals_to_next[]`**
  per-stage fields from the original schema sketch in #14. Judged
  speculative until a real brand surfaces the need; `triggers` and
  `hurdles` cover most of the same intent for v1.4.
- **Legal/trademark relocation** for `gramatr-brand`'s service mark
  (sub-issue #17, Gap 6 from #12). Independent patch-level PR.

## [1.3.0] — 2026-05-09

Closes nine schema gaps tracked in
[`gramatr/brand-spec` issue #8](https://github.com/gramatr/brand-spec/issues/8).
The first five were surfaced by `lean-media-brand`'s multi-register
voice work (PR #3 there); the remaining four were surfaced by
`gramatr-brand`'s subsequent multi-register restructure (gramatr-brand
PRs #5 and #6). Two independent brands hitting the same gaps moved
the corpus-metadata fields and the inheritance rule from "candidate"
to "must-have" for v1.3.

Semver justification: minor bump. Multiple new optional fields and
conventions, plus new validation rules — but no required field added,
no enum tightened, no existing field renamed or removed. v1.2.1
brands (NEXT90, gramatr-brand, lean-media-brand) validate cleanly
against v1.3 with zero changes.

### Added

- **Corpus metadata for register frontmatter (issue #8 item 1).**
  Optional `corpus_size` (integer), `corpus_source` (string),
  `corpus_pulled` (ISO date), and `confidence` (`high`/`medium`/
  `low`) fields on `voice/registers/<slug>.md` frontmatter. Signals
  trust and decay so downstream agents can weight a register's rules
  appropriately. Validation rules: `corpus-metadata-recommended`
  (warn), `corpus-pulled-freshness-warn` (warn at >365 days).

- **`applies_to` baseline channel slugs (issue #8 item 2).** New
  `conventions.applies_to_baseline_slugs` block in `brand.yaml`
  catalogs canonical kebab-case channel/context slugs grouped by
  surface family (web, sales, email, paid, social, technical, long-
  form). Brands MAY extend with brand-specific slugs. Validation
  rule: `applies-to-baseline-warn` (warn on drift like `email` vs
  `marketing-emails`). Convention-only enforcement in v1.3; full
  taxonomy is v2.

- **Register inheritance enforcement (issue #8 items 3 and 6).** New
  validation rule `register-inheritance-cannot-subtract` (error):
  forbidden terms in `vocabulary.md` MUST NOT appear inside an
  "allowed" body section of any register file. Forbidden terms inside
  `> Quoted:` callouts, "Published Voice Examples" quote blocks, or
  "What does NOT fit this register" sections are ignored.

- **Quoted-voice exemption pattern (issue #8 item 4).** Two co-equal
  conventions:
    - `contains_quoted_voices: true` register frontmatter flag
      (file-level signal; validators MAY relax voice/vocabulary
      enforcement on the entire file body).
    - `> Quoted: <attribution>` markdown blockquote callout (inline;
      validators MUST skip evaluation of the blockquote body).
  Validation rule: `quoted-voice-callout-skipped` (info).
  Both conventions land because frontmatter is validator-friendly
  and the markdown callout is author-friendly; brands MAY use
  either.

- **Body section guidance for register 2+ (issue #8 item 5).** Two
  new entries in `body_sections_recommended` for register files —
  "What this register allows that {primary} does not" and "What is
  still forbidden (inherited)" — replacing the awkward "What the
  Voice Is NOT" framing for any register beyond the primary.
  Validation rule: `register-inheritance-doc-recommended` (warn).
  Brands MAY use other section shapes; the rule is informational.

- **Cross-register precedence (issue #8 item 7).** Optional
  `precedence:` frontmatter block on `voice/README.md` (or
  `voice/_index.md`) with `default_order` (register slugs in winning
  order) and/or `surface_overrides` (surface slug → winning register
  slug). Machine-readable equivalent of the prose "Precedence when
  contexts overlap" section both real-world brands document today.
  Validation rule: `precedence-references-resolve` (error — every
  named register slug MUST resolve to an existing
  `voice/registers/<slug>.md`).

- **Platform-overlay subsections (issue #8 item 8).** Optional
  `platform_overlays:` register frontmatter block — keys are kebab-
  case platform slugs (`linkedin`, `twitter`, etc.), values are
  free-form objects whose shape is brand-defined but SHOULD include
  one of `hashtag_convention`, `emoji_budget`, `divider_convention`,
  or `cta_permission`. An equivalent `On {Platform}` H2 section is
  acceptable when frontmatter is absent. Validation rule:
  `platform-overlay-keys-recommended` (warn).

- **"What does NOT fit this register" first-class section (issue #8
  item 9).** New entry in `body_sections_recommended` for register
  files. Drift-detection tooling reads this section to flag
  generated content that resembles the negative-space examples.
  Validation rule: `drift-detection-section-recommended` (info).

- **README:** new "What's new in v1.3" section enumerating all nine
  closures with brand-author summary.

- **Multi-register template** (`templates/multi-register-voice-
  example/`) — updated to demonstrate items 1, 5, 7, 8, and 9 in
  use. The template is the working reference for brands implementing
  multi-register voice; it should show the v1.3 conventions in use.

### Changed

- `voice/registers/<slug>.md` `body_sections_recommended` extended
  with five new recommended sections (items 5, 8, 9 above). The
  existing "What the Voice Is NOT" remains valid for the primary
  register.

### Backward compatibility

- v1.2.1 brands (NEXT90, gramatr-brand, lean-media-brand) validate
  cleanly against v1.3 with zero modifications. No required fields
  added; no v1.2.1 fields tightened, renamed, or removed; no enums
  narrowed.
- All nine new fields and frontmatter blocks are optional. The
  `register-inheritance-cannot-subtract` rule is the only error-
  severity addition; it triggers only when a register file
  explicitly allows a brand-wide forbidden term, which neither real-
  world brand does today (both already assert the inheritance rule
  in prose).
- The `applies-to-baseline-warn` rule is warn-only; brands using
  brand-specific slugs (e.g., `linkedin-brian-handle`,
  `linkedin-company-page`) continue to validate without warnings
  because the rule recognizes brand-named surfaces as legitimate
  extensions.

### Migration notes

None required. Existing brands work unchanged. Brands MAY adopt the
new conventions opportunistically:

- Add `corpus_size` / `corpus_source` / `corpus_pulled` /
  `confidence` to register files when next refreshing voice from
  corpus (both gramatr-brand and lean-media-brand already declare
  these ad-hoc — those become spec-conforming with no edits).
- Move existing prose "Precedence when contexts overlap" sections
  into the structured `precedence:` frontmatter block on
  `voice/README.md` when convenient.
- Move existing ad-hoc "On LinkedIn" H2 sections into
  `platform_overlays:` frontmatter when convenient (the H2 form
  remains valid).
- Add "What does NOT fit this register" sections to register files
  when a drift-detection signal would be useful (gramatr-brand's
  `founder-byline.md` already has one).

### Validator

The reference validator (`gramatr/brand-spec-validator`) pins to
v1.2.1 today via its vendored `vendor/brand-spec/brand.yaml`. The
validator picks up v1.3 on the next maintainer-triggered spec-sync
run; the recommended sequence is:

1. Merge this PR.
2. Run the validator's spec-sync script to vendor v1.3 `brand.yaml`.
3. Tag validator v0.1.2.
4. (Optional, per-brand) Open adoption PRs to add the new optional
   fields. None required for v1.3 conformance.

The new error-severity rule (`register-inheritance-cannot-subtract`)
is the only one that requires non-trivial validator code (parse
register file body sections, extract "allowed" lists, intersect with
`vocabulary.md` forbidden list). The other warn/info rules are
straightforward frontmatter checks. Issue #8 item 6 (vocabulary
inheritance enforcement) is **shipped in this spec** but remains
**implementation-deferred in the validator** until the body-section
parser lands; until then validators MAY skip the rule with a single
"v1.3 rule not yet enforced" warning.

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
