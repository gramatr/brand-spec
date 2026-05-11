# Changelog

All notable changes to `brand-spec` are documented here. The schema
follows semver: minor bumps are additive (no breaking changes to
prior-version brands); major bumps may tighten or rename fields.

## [1.10.0] — 2026-05-11

Minor release. Formalizes the persona matrix axes — adds optional
`audience_taxonomy:` and `role_taxonomy:` array fields to
`personas/_framework.md` frontmatter. Third entry of the
[v1.8 RFC](https://github.com/gramatr/brand-spec/issues/50)
("proposal 3") to land.

### The gap

Before v1.10, `personas/_framework.md` declared only `type:` and
`version:` in frontmatter. The matrix axes (audience segments and
role archetypes) were documented in prose, with no machine-readable
declaration. A three-brand audit found three different role
taxonomies in production:

- next90-brand: `{business-leader, analyst, operator}`
- gramatr-brand: `{practitioner, lead, buyer, champion}` (status:
  draft; role axis explicitly open)
- lean-media-brand: `{marketing-director, media-planner,
  performance-marketer, director}` (asymmetric 2×4 matrix; 4 of 8
  cells intentionally not filled)

The audience axis showed the same pattern. Validators had no way
to cross-check persona files' `audience:` / `role:` slugs against
the brand's declared taxonomy; downstream tooling could not read
the matrix without parsing prose.

### The fix

Two new optional array fields on `personas/_framework.md`
frontmatter:

- `audience_taxonomy:` — ordered list of audience slugs the brand's
  matrix uses. Each slug SHOULD correspond to a
  `messaging/for-{audience}.md` file.
- `role_taxonomy:` — ordered list of role slugs the brand's matrix
  uses.

Both fields are optional and additive. Brands that prefer to keep
the matrix in prose only continue to validate. Brands that adopt
the new fields enable validators to perform membership checks
against `audience:` / `role:` values on individual persona files.

### Design notes

**Membership semantics, not coverage.** The taxonomy fields are
domains of valid slugs, not assertions that every
`(audience, role)` cell exists. Asymmetric matrices (lean-media's
2×4 with 4 filled cells, gramatr's draft state where `champion` is
declared as a candidate but no file uses it) are explicitly valid.

**Slug form is brand-defined.** The spec does not prescribe
singular (`advertiser`) vs plural (`advertisers`) — brands SHOULD
pick a convention and stay consistent. Observed in practice:
next90 singular, gramatr/lean-media plural.

**Validator behavior is deferred.** v1.10.0 declares the fields;
specific validation rules (`persona-role-in-taxonomy`,
`persona-audience-in-taxonomy`, info-severity advisories for
audience-taxonomy entries without matching messaging files) are
left to a follow-up validator release.

### Changed

- `brand.yaml` (`contract_version`): 1.9.0 → 1.10.0.
- `brand.yaml` (`layers.personas.directories[].contents[_framework.md].frontmatter`):
  added `audience_taxonomy:` (array, optional) and
  `role_taxonomy:` (array, optional).
- `brand.yaml` (same path, `notes:`): expanded from one-line to
  documenting the v1.10.0 fields and the membership-vs-coverage
  semantics.

### Backward compatibility

Purely additive. No fields renamed, no required-field tightening.
v1.9.x brands validate cleanly against v1.10.0 with zero changes.
Validators with no taxonomy-membership rule continue to behave
identically.

## [1.9.0] — 2026-05-11

Minor release. Formalizes two cross-layer-reference fields that
brands already use in practice but the per-layer schemas didn't
enumerate. Second entry of the [v1.8 RFC](https://github.com/gramatr/brand-spec/issues/50)
("proposal 2") to land. No new layers, no new files, no schema
shape changes — both fields already validate under the v1.7
`^[a-z_]+_refs?$` convention; this release simply elevates them
from convention-implicit to schema-explicit.

### What's added

**1. `data_viz_refs:` on `messaging/channel-{channel}.md`.**
Optional array of paths into `data-viz/`. Lets a channel guide
declare which data-viz files govern charting on that channel.
Evidence: next90-brand uses this on three channels (deck, social,
landing-page), with 3–4 entries each.

**2. `design_tokens_ref:` on every `data-viz/*.md` file.**
Optional scalar path to the brand's design-tokens source (
conventionally `design-tokens.md`). Added to the frontmatter
schemas of `_framework.md`, `colors.md`, `chart-types.md`,
`axes-and-grid.md`, `annotations.md`, `numbers.md`, and
`tables.md` — all 7 file slots the layer declares. Formalizes
what the layer's prose at the top of `data_viz:` has always
said ("Color tokens live in `design-tokens.md` and this layer
assigns DATA ROLES to those token names"). Evidence:
next90-brand uses this on 4 of 7 data-viz files.

### Cardinality note

The v1.7 cross-layer-ref convention treats `*_ref` (singular,
scalar) and `*_refs` (plural, array) as distinct fields with
distinct cardinalities. v1.9.0 declares `data_viz_refs:` (plural)
on channel guides because brands typically reference multiple
data-viz files per channel; brands that only need one MAY use
`data_viz_ref:` (singular) instead — both validate under the
convention. `design_tokens_ref:` is singular because there is
typically one design-tokens file per brand.

### Why this is in 1.9 and not 1.8.1

Schema declarations under `layers:` constitute a contract
expansion (new validator-discoverable surface, even though the
underlying fields already validated). v1.8.x is reserved for
baseline-list grows and other strictly non-schema changes.
v1.9.0 is the next minor where layer schemas can grow.

### Changed

- `brand.yaml` (`contract_version`): 1.8.0 → 1.9.0.
- `brand.yaml` (`layers.messaging.directories[].contents[channel-{channel}.md].frontmatter`):
  added `data_viz_refs:` (array, optional).
- `brand.yaml` (`layers.data_viz.directories[].contents[]`):
  added `design_tokens_ref:` (string, optional) to all 7 file
  schemas (_framework, colors, chart-types, axes-and-grid,
  annotations, numbers, tables).

### Backward compatibility

Purely additive. No fields renamed, no required-field tightening,
no shape changes. Brands validating cleanly against v1.8.0
continue to validate cleanly against v1.9.0. Validators that
already resolve `*_refs?$` fields under the v1.7 convention need
no code change; the spec change just makes the fields
discoverable from per-layer schemas instead of from prose.

## [1.8.0] — 2026-05-11

Minor release. Expands `conventions.applies_to_baseline_slugs.technical`
to formalize the agent/SDK surface observed across production brands.
First entry of the [v1.8 RFC](https://github.com/gramatr/brand-spec/issues/50)
to land; the remaining proposals in that issue are deferred to later
v1.8.x / v1.9.x releases.

### The gap

A three-brand audit (next90-brand, gramatr-brand, lean-media-brand)
found that `gramatr-brand/voice/registers/engineering.md` declares 10
channel slugs (`claude-md`, `agent-system-prompts`,
`mcp-tool-descriptions`, `intelligence-packet-contract`,
`architecture-docs`, `cli-help-text`, `cli-error-messages`,
`status-line`, `eng-standards`, `technical-readmes`) — none of which
were in the v1.7.6 technical baseline. They are systematic refinements
of pre-existing baseline entries:

- `cli-output` → split into `cli-help-text`, `cli-error-messages`,
  `status-line` for the three distinct surfaces a CLI presents.
- `readmes` → `technical-readmes` when the audience is engineers
  specifically.
- `agent-prompts` → `agent-system-prompts` when the surface is the
  system-prompt string shipped to an agent runtime (vs. user-facing
  prompt templates).
- `error-messages` → `cli-error-messages` / `api-responses`
  (forthcoming) when the surface is more specific than generic UI
  errors.

`api-responses` and `sdk-output` were not yet used by any brand but
are the obvious next baseline entries for any brand shipping an SDK
or HTTP API; including them now avoids a follow-up baseline bump
the first time a brand documents either surface.

### Changed

- `brand.yaml` (`contract_version`): 1.7.6 → 1.8.0.
- `brand.yaml` (`conventions.applies_to_baseline_slugs.technical`):
  added 12 new baseline slugs — `agent-system-prompts`,
  `architecture-docs`, `cli-help-text`, `cli-error-messages`,
  `status-line`, `technical-readmes`, `eng-standards`, `claude-md`,
  `mcp-tool-descriptions`, `intelligence-packet-contract`,
  `api-responses`, `sdk-output`. Pre-v1.8 entries are unchanged and
  remain canonical for their original meanings.

### Backward compatibility

Purely additive. No fields renamed, no fields tightened, no schema
shape changes. v1.7.x brands validate cleanly against v1.8.0 with
zero changes. Validators that emit `applies-to-slug-not-in-baseline`
warnings will stop warning on the 12 new slugs; no validator code
change is required (the baseline is read from the spec).

### Validator impact

Zero code change required. brand-spec-validator reads the baseline
list from `brand.yaml` at validation time, so simply bumping its
contract-version pin to 1.8.0 picks up the new entries automatically.

## [1.7.6] — 2026-05-11

Patch release. Adds an `exempt_fields:` declaration to
`conventions.cross_layer_references` so that field names matching the
v1.7.0 `field_name_pattern` regex (`^[a-z_]+_refs?$`) but carrying
non-resolvable slug values are formally exempted from
target-resolution. Closes #46.

**Problem.** The v1.7.0 cross-layer-reference convention's
`field_name_pattern` regex is intentionally broad — any frontmatter
field ending in `_ref` or `_refs` is treated as a cross-layer
reference whose target MUST resolve to a layer / file / token /
path-with-fragment / url. But `applies_to_refs:` (the canonical
v1.7.0 form of the legacy v1.3 `applies_to:` field) carries
**channel slugs** (`website`, `linkedin-company-page`,
`marketing-emails`), not file/layer/token/url targets. Slug
vocabulary is governed by the orthogonal v1.3
`conventions.applies_to_baseline_slugs` convention with its own
warn-on-drift validation. Naïvely applying
`cross-layer-ref-target-resolves` to `applies_to_refs` produced 9
false errors on lean-media-brand (validator v0.3.0,
[brand-spec-validator PR #20](https://github.com/gramatr/brand-spec-validator/pull/20)) —
the validator shipped a workaround `NON_RESOLVING_REF_FIELDS` set
hard-coded in rule code while waiting for the spec to declare the
exemption canonically.

**Fix.** New `conventions.cross_layer_references.exempt_fields:`
list. Each entry declares (a) the field name, (b) why its values
do not resolve as cross-layer-reference targets, (c) the convention
that DOES govern its value-vocabulary. v1.7.6 ships with one entry:

- `applies_to_refs` — values are channel slugs governed by the v1.3
  `applies_to_baseline_slugs` convention.

The exempt-fields list is the documented home for similar future
cases (e.g., a hypothetical `themes_refs` whose values were brand
theme slugs rather than file paths). Authors and validator
implementers SHOULD consult this list before assuming a
regex-matching field is a target-resolving cross-layer reference.

**Validator impact.** Validators MUST read
`conventions.cross_layer_references.exempt_fields[].name` and skip
`cross-layer-ref-target-resolves` for those field names. The
hard-coded `NON_RESOLVING_REF_FIELDS` set in brand-spec-validator
v0.3.0 can be replaced by reading from the spec — tracked as a
separate validator follow-up.

No schema changes. No required fields added or renamed. No
field-name pattern change (which would have been a MINOR / breaking
change requiring brands to rename `applies_to_refs`). v1.7.0–1.7.4
brands validate cleanly against v1.7.6 with zero changes.

## [1.7.5] — 2026-05-11

Patch release. Closes [#45](https://github.com/gramatr/brand-spec/issues/45).
Schema-declaration correction: the three layer schemas affected by the
v1.7.0 cross-layer-reference convention now declare alias-aware
required-field semantics, matching the backward-compat promise the
spec already documented in prose.

### The gap

v1.7.0 introduced the `*_ref` / `*_refs` cross-layer reference naming
convention and explicitly preserved backward-compat for three legacy
field names (`priority_layers`, `applies_to`,
`source_authority.upstream`). The `conventions:` block documented this
correctly. But the affected layer schemas under `layers:` still
declared the legacy field names as `required: true` literally — so a
brand that adopted the canonical `*_refs` names would hard-fail the
layer schema's required-field check even though the convention block
said either form was accepted. Brand-spec-validator v0.3.0 worked
around the gap per-codebase; this release fixes it at the spec.

### The fix

1. New declarative alias registry under
   `conventions.cross_layer_references.backward_compat.field_aliases`
   — three entries mirroring the validator's `FieldAlias` interface
   (`brand-spec-validator/src/aliases.ts`). Validators MAY load this
   registry as data.
2. The three affected schema entries gain three fields:
   `accepts_aliases: true`, `canonical_name: <v1.7 name>`,
   `legacy_names: [<legacy name>]`. The inline arrays repeat the
   registry data so each schema entry is self-contained when read in
   isolation. Required-field semantics are satisfied when EITHER the
   canonical name OR any registered legacy name is present.

### Affected schema entries

- `layers.agent_context.files[agent-context.md].frontmatter.priority_layers`
  — alias-aware, canonical `priority_layers_refs`.
- `layers.voice.directories[voice/].contents[registers/{slug}.md].frontmatter.applies_to`
  — alias-aware, canonical `applies_to_refs`.
- `conventions.frontmatter.fields.source_authority.properties.upstream`
  — alias-aware, canonical `upstream_ref`.

### No behavior change

- Brands: nothing to do. Both name forms already worked in practice
  (legacy via the un-updated literal; canonical via the validator
  workaround). v1.7.0–1.7.4 brands validate cleanly against v1.7.5.
- Validators: existing v0.3.0 alias-aware code remains correct; the
  spec now matches the implementation. Future validator ports can
  read the registry directly off `brand.yaml`.

### Changed

- `brand.yaml` (`contract_version`): 1.7.4 → 1.7.5.
- `brand.yaml` (`conventions.cross_layer_references.backward_compat`):
  added `field_aliases:` registry (3 entries).
- `brand.yaml` (`conventions.frontmatter.fields.source_authority.properties.upstream`):
  added `accepts_aliases: true`, `canonical_name`, `legacy_names`.
- `brand.yaml` (`layers.agent_context.files[].frontmatter.priority_layers`):
  expanded from inline to block form; added alias-aware fields.
- `brand.yaml` (`layers.voice.directories[].contents[].frontmatter.applies_to`):
  added alias-aware fields.

## [1.7.4] — 2026-05-10

Patch release. Templates and `brand.yaml` examples now demonstrate the
v1.7.0 cross-layer reference convention (`*_ref` / `*_refs`) instead
of the legacy per-layer field names. Closes Phase C of #36 spec-side.
Brand-side adoption is incremental (per-brand PRs).

Field renames in templates:

- `templates/empty-brand/agent-context.md` — `priority_layers:` →
  `priority_layers_refs:` (1 rename)
- `templates/multi-register-voice-example/voice/registers/corporate.md`
  — `applies_to:` → `applies_to_refs:` (1 rename)
- `templates/multi-register-voice-example/voice/registers/engineering.md`
  — `applies_to:` → `applies_to_refs:` (1 rename)
- `templates/multi-register-voice-example/README.md` — prose updated to
  reference `applies_to_refs:` as the canonical convention form.

`brand.yaml` `cross_layer_references:` block gains a v1.7.4 note
recording that the spec's own templates now demonstrate the
convention end-to-end. Image-generation refs
(`photography_treatment_ref`, `assets_manifest_ref`,
`design_tokens_ref`) have been convention-conformant since v1.6.0 —
no migration needed.

No schema changes. No required fields added or renamed. Legacy field
names (`priority_layers`, `applies_to`, `upstream`) continue to
validate cleanly per the v1.7.0 backward-compat block; schema
property definitions in `brand.yaml` are unchanged. v1.7.0–1.7.3
brands validate cleanly against v1.7.4 with zero changes.

## [1.7.3] — 2026-05-10

Patch release. Canonicalizes the design-token name examples in
`brand.yaml` to a single convention: **`--{category}-{...}`**
(category-first), e.g., `--color-brand-primary`, `--color-gray-60`,
`--font-heading`, `--space-md`, `--icon-size-xs`. Prior examples mixed
two shapes — `--color-brand-blue` (category-first) and `--brand-primary`
(role-only) — leaving brand authors without a clear precedent to follow.

The category-first convention is the one used in practice by the three
real brand repos (NEXT90's `color.brand.blue` / `color.bg.dark` /
`font.body`, Lean Media's `color.brand.primary` / `color.surface.dark` /
`radius.button`, gramatr-brand's `--font-heading` / `--font-body`) and
aligns with W3C Design Tokens precedent (category at the root of the
token path: `color.*`, `font.*`, `space.*`, etc.). gramatr-brand's
color tokens use a brand-prefix variant (`--gmtr-primary`) — that local
choice is not the spec's concern; the spec's job is to pick one shape
for its own examples and stop confusing brand authors with a mixed
precedent.

No schema changes. No required fields added or renamed. v1.7.0–1.7.2
brands validate cleanly against v1.7.3 with zero changes. Real brand
`design-tokens.md` files are NOT touched by this release; they remain
free to use whatever local convention they prefer.

Closes B6 of [`gramatr/brand-spec#36`](https://github.com/gramatr/brand-spec/issues/36).

### Changed

- `brand.yaml` (`conventions.cross_layer_references.reference_value_types.token.notes`):
  example expanded from `'--brand-primary'` to
  `'--color-brand-primary, --font-heading, --space-md'` and explicitly
  labeled "category-first".
- `brand.yaml` (cross-layer references `example` block):
  `target: --color-brand-blue` → `target: --color-brand-primary` (the
  blue/purple specifics belong in a brand, not the spec example).
- `brand.yaml` (`data_viz` description prose): inline example updated
  from `--brand-primary` / `--brand-secondary` to
  `--color-brand-primary` / `--color-brand-secondary`.
- `brand.yaml` (`data-viz/colors.md` canonical role-to-token table):
  all seven role rows updated to category-first form
  (`--color-brand-primary`, `--color-brand-secondary`,
  `--color-surface-subtle`, `--color-semantic-success`,
  `--color-semantic-error`, `--color-gray-60`).
- `brand.yaml` (`image-gen-design-token-references-resolve` rule prose):
  inline example updated from `--brand-primary` to
  `--brand-secondary` → `--color-brand-primary` to `--color-brand-secondary`.

### Brand-side observations (not fixed here)

The three real brand repos use three different local token conventions:

- **NEXT90:** dotted (`color.brand.blue`, `font.body`) — no `--` prefix.
- **Lean Media:** dotted (`color.brand.primary`, `radius.button`) — no `--` prefix.
- **gramatr-brand:** CSS custom properties (`--gmtr-primary`,
  `--font-heading`) with a mix of brand-prefix (color) and
  category-first (typography).

This is a brand-side authoring inconsistency, not a spec gap. The spec
does not require brands to use any particular token-name shape; it only
needs its own examples to commit to one. Each brand may choose to
reconcile its internal mix in its own time. No spec change required.

## [1.7.2] — 2026-05-10

Fix B3 of [`gramatr/brand-spec#36`](https://github.com/gramatr/brand-spec/issues/36).

**KYKC canonicalization + scrub of v1.4 hallucination.**

The methodology authored by Brian Handrigan at the original grāmatr digital marketing agency (circa 2005) is **"Know Yourself, Know Your Customer"** — two halves, both required, with the **CognitiveJourney** as the structured artifact that operationalizes the customer half.

The v1.4.0 PR (commit `06f6938`) shipped a hallucinated alternative — "Know Your Kingdom + Know Your Customer" — in `templates/journey-example/journey/_framework.md` (4 references) and `brand.yaml`'s journey layer description (1 reference). It also shipped an incomplete expansion (just "Know Your Customer") in 4 other places (`README.md`, `CHANGELOG.md` v1.4 entry, `brand.yaml` example, template frontmatter), losing the self-knowledge half entirely.

This release scrubs both errors and propagates the canonical:

- `templates/journey-example/journey/_framework.md` — frontmatter `methodology_provenance.name`, the "## The discipline (KYKC)" body section, and the "what your Kingdom offers" prose all corrected to use "Know Yourself" / "Know Yourself, Know Your Customer" / "what you can credibly offer"
- `brand.yaml` — example in `methodology_provenance.name.notes` corrected; journey layer description's KYKC expansion corrected
- `README.md` — "What's new in v1.4" section corrected to the full canonical
- `CHANGELOG.md` — historical v1.4 entry corrected (the entry was documenting the canonical methodology, not "what shipped at the time"; the methodology itself was authored at the original grāmatr agency in 2005, not in 2026)

PATCH bump (1.7.1 → 1.7.2) per VERSIONING.md — additive doc/example correction; no schema change; no rule change; no template structural change. Existing brands that already use the v1.4 hallucinated wording in their own `journey/_framework.md` should opt in to the canonical at their own pace.

## [1.7.1] — 2026-05-10

Closes B2 of
[`gramatr/brand-spec#36`](https://github.com/gramatr/brand-spec/issues/36)
— rule-ID drift between spec and validator. The spec declared the
v1.5 rule as `data-viz-framework-recommended`; the validator
implementation in `gramatr/brand-spec-validator` (v0.2.0 body-parse
subsystem) emits issues with rule ID `data-viz-framework-presence`.
Same intent, two names — a contract bug surfaced by the deep audit
on epic #13 (comment-4417090502).

Semver justification: **PATCH bump (per `VERSIONING.md`).** Pure rule
rename to align the spec with the canonical validator emission. No
new behavior, no new field, no enum change. v1.7.0 brands continue
to validate identically — the rule body and severity are unchanged;
only the ID label moves. Picked the validator-side name (`presence`)
because it carries better semantics (the rule actually checks for
presence of expected topics in `_framework.md`) and appears in more
load-bearing locations (validator source, tests, validator
CHANGELOG) than the spec-side name (spec body, README, historical
v1.5 CHANGELOG entry).

### Changed

- `validation:` rule `data-viz-framework-recommended` renamed to
  `data-viz-framework-presence` in `brand.yaml` to match the rule ID
  emitted by `gramatr/brand-spec-validator`. README rule list
  updated to match. Historical v1.5 CHANGELOG entry left as the
  as-shipped record.


## [1.7.0] — 2026-05-10

Closes
[`gramatr/brand-spec#31`](https://github.com/gramatr/brand-spec/issues/31)
— the cross-layer reference syntax RFC. Lifts a pattern that has
been invented per-layer six times into a single canonical convention
so future cross-layer reference needs reuse one shape and validators
can offer one generic resolver instead of N per-layer rules.

Semver justification: **MINOR bump (per `VERSIONING.md`).** Cross-layer
reference syntax is a new structural primitive that opens use cases
prior versions could not support — generic reference-graph
construction by AI agents, one validator resolver across all layers,
authoritative `kind` (requires / recommends / cites) edges between
files. `VERSIONING.md` names "a new structural primitive (e.g.,
schema-level support for cross-brand inheritance)" as canonical
minor-bump territory; this release matches that bar. No required
field added; no enum tightened; no existing field renamed or removed.
v1.6.1 brands (NEXT90, gramatr-brand, lean-media-brand) validate
cleanly against v1.7.0 with zero modifications — verified by
inspection: the convention is purely additive and every legacy
per-layer reference field continues to validate as before.

The release intentionally ships the convention only. The six
existing invention sites are NOT migrated in this PR; each migration
lands as a separate v1.7.x patch so reviewers can audit them
individually and brand authors can opt in at their own pace.
Validator runtime resolution of the new generic rules is post-merge
work in `gramatr/brand-spec-validator` (likely v0.2.0 since the
generic resolver is a meaningful new subsystem).

### Added

- **`conventions.cross_layer_references:` block in `brand.yaml`
  (RFC #31).** New convention block under the existing
  `conventions:` section, alongside `methodology_provenance:` and
  `applies_to_baseline_slugs:`. Sub-fields:
    - **`field_name_pattern:`** — regex `^[a-z_]+_refs?$`. Documents
      which frontmatter fields are treated as references. Singular
      `*_ref` carries one scalar; plural `*_refs` carries an array.
      Less typing for the common single-value case while keeping
      array shape symmetrical.
    - **`reference_value_types:`** — five resolvable target types:
      `layer` (resolves against `brand.yaml`'s `layers:` block),
      `file` (path against the brand repo root), `token` (token name;
      resolution depends on the body-parse subsystem long-term —
      validator emits `warn` for unresolvable token refs until that
      subsystem lands), `path_with_fragment` (`path#section-anchor`),
      and `url` (external; well-formedness only).
    - **`structured_references_block:`** — optional top-level
      `references:` frontmatter block schema. Each entry carries
      `type`, `target`, `kind` (one of `requires` | `recommends` |
      `cites`), an optional `entries` array (for the
      `layer-entries` type with sub-selection), and optional
      `notes` (free-text edge label for downstream reference-graph
      consumers). Optional throughout — most refs use the naming
      convention alone; authors opt into the structured block only
      when the extra metadata is genuinely needed.
    - **`backward_compat:`** — explicit policy that legacy per-layer
      reference fields (`source_authority.upstream`,
      `agent-context.priority_layers`,
      `voice/registers.applies_to`, etc.) keep working unchanged.
      Carries the canonical legacy → convention mapping that the
      `legacy-ref-field-migration-recommended` rule enforces.
      Hard deprecation deferred to v2.

- **Validation rules (3 new):**
    - `cross-layer-ref-target-resolves` (error) — when a frontmatter
      field matches the `^[a-z_]+_refs?$` pattern, the resolved
      target MUST exist (file exists, layer is declared, URL is
      well-formed). Token references emit `warn` (not `error`) until
      the body-parse subsystem lands; the rule then tightens for
      tokens too. Replaces, in spirit, the per-layer resolver
      pattern repeated across `agent-context-priority-layers-resolve`,
      `image-gen-photography-treatment-link-valid`,
      `image-gen-overlay-manifest-link-valid`, and others — those
      legacy rules remain in force for backward compatibility but
      converge with the generic rule for new `*_ref` / `*_refs`
      fields.
    - `structured-references-kind-valid` (error) — when the
      `references:` block is present, every entry MUST declare
      `type`, `target`, and `kind`. `kind` MUST be one of
      `requires` | `recommends` | `cites`. `type` MUST be one of
      `layer` | `file` | `token` | `path_with_fragment` | `url` |
      `layer-entries`.
    - `legacy-ref-field-migration-recommended` (info) — when a
      known legacy reference field (`source_authority.upstream`,
      `agent-context.priority_layers`,
      `voice/registers.applies_to`, etc.) is present, the validator
      SHOULD emit an info-severity advisory pointing at the
      convention-conformant shape. Brands MAY ignore the advisory
      indefinitely.

- **`templates/cross-layer-refs-example/`** — small contrived but
  realistic worked example. A fictional
  `messaging/channel-deck.md` demonstrates both the naming
  convention (`data_viz_ref:`, `personas_refs:`) and the structured
  `references:` block (with `kind: requires` / `kind: cites`
  entries). A sibling `data-viz/colors.md` is the resolution
  target. Kept intentionally small — its job is to document the
  convention shape, not to demonstrate every edge case.

### Changed

- `contract_version` bumped to `1.7.0`.
- `README.md` updated:
    - The "For AI agents" section gains a paragraph explaining
      that agents can grep frontmatter for fields matching
      `^[a-z_]+_refs?$` (and read the optional `references:`
      block) to construct a brand reference graph generically,
      with a pointer to the `conventions.cross_layer_references:`
      block in `brand.yaml` for the full schema.
    - New "What's new in v1.7" section above the v1.6.1 entry.

### Backward compatibility

- v1.6.1 brands (NEXT90, gramatr-brand, lean-media-brand) validate
  cleanly against v1.7.0 with zero modifications — verified by
  inspection. The cross-layer reference convention is purely
  additive: the only new error-severity rules
  (`cross-layer-ref-target-resolves`,
  `structured-references-kind-valid`) trigger only on fields that
  match the new naming convention or on the new `references:`
  block, and no legacy field uses either shape.
- No required field added to any existing layer.
- No enum tightened, no existing field renamed or removed.
- Every legacy per-layer reference field continues to validate as
  before. The `legacy-ref-field-migration-recommended` rule is
  info-severity and never blocks; brands MAY ignore the advisory.

### Migration plan (deferred — NOT in this PR)

The 6 existing invention sites stay as-is in v1.7.0. Each migrates
in a separate v1.7.x patch PR so reviewers can audit them
independently:

1. v1.7.1 — `agent-context.md` `priority_layers` →
   `priority_layers_refs`
2. v1.7.2 — `voice/registers/<slug>.md` `applies_to` →
   `applies_to_refs`
3. v1.7.3 — `source_authority.upstream` → `upstream_ref`
4. v1.7.4 — journey-stage frontmatter pointers (deferred from v1.4)
   adopt `journey_stage_ref` / `journey_stages_refs`
5. v1.7.5 — `data-viz/colors.md` color-role token references adopt
   the `color_role_*_ref` convention
6. v1.7.6 — `image-generation/style.md` and
   `image-generation/brand-overlays.md` reference fields confirmed
   convention-conformant; legacy aliases formally cited in the
   convention

Each patch is opt-in for brand authors — the legacy field continues
to validate cleanly, and the migration is a doc/template change in
the spec plus a recommendation for brand maintainers.

### Out of scope (deferred)

- **Validator code implementation.** v1.7.0 ships the schema and
  documentation only. The reference validator
  (`gramatr/brand-spec-validator`) implements the generic resolver
  and the three new rules in a follow-up release (likely v0.2.0,
  since the generic resolver is a meaningful new subsystem and not
  a drop-in addition to the existing rule loop).
- **Inline body-reference syntax.** `{{ ref:name }}` templating
  inside markdown body is NOT in v1.7.0. Frontmatter refs only.
  Defer until a real consumer needs body-level references.
- **URI-scheme pointers (Option D from RFC #31).** Forms like
  `brand-spec://layer/path` are deferred to v2. Cross-brand
  references are the most plausible trigger; today's brand-spec is
  single-brand-per-repo.
- **Per-layer rule consolidation.** Existing per-layer reference
  rules (`agent-context-priority-layers-resolve`,
  `image-gen-*-link-valid`, etc.) remain in force in v1.7.x for
  backward compatibility. Consolidation under the generic
  resolver is v2 territory once all six legacy fields have
  migrated to the convention shape.

## [1.6.1] — 2026-05-10

Closes Phase 2 sub-issue
[`gramatr/brand-spec#25`](https://github.com/gramatr/brand-spec/issues/25)
of the brand-spec coverage epic
[`#13`](https://github.com/gramatr/brand-spec/issues/13). Addresses
gap #3 from audit
[`#12`](https://github.com/gramatr/brand-spec/issues/12) — iconography
systems were recommended-but-missing across all 3 brands.

Semver justification: **PATCH bump (per `VERSIONING.md`).** Extension
of the existing `design/` layer with optional fields and an optional
sibling directory (`design/icons/`) — not a new top-level layer and
not a new capability domain. The policy gives "additive optional
fields, new conventions" and "new validation rule that emits `info`
or `warn` only" as canonical patch examples; this release matches
both. The lone `warn`-severity rule
(`iconography-sizing-references-resolve`) only triggers when a brand
populates the new optional `sizing:` block, so v1.6.0 brands cannot
regress. No new top-level layer, no required field added, no enum
tightened, no existing field renamed or removed. v1.6.0 brands
(NEXT90, gramatr-brand, lean-media-brand) validate cleanly against
v1.6.1 with zero modifications — none currently has
`design/iconography.md` and the file is `recommended: true,
required: false`.

The release also formalizes the **decorative-vs-functional
distinction** so brands and downstream tools have a clean way to
classify ambiguous "icon-shaped" assets:

- **Decorative or gradient icons** that read as brand imagery
  (e.g., section-decoration gradient SVGs on a marketing site) live
  in `assets/_manifest.md` and are governed by the assets layer.
- **Functional UI icons** that participate in the icon system
  (navigation, actions, status, in-product chrome) live in
  `design/iconography.md`.

Real-world test case (lean-media-brand): the live site carries five
`custom-icon_*-grad.svg` files (audience, farm characteristics,
location, Venn diagram) as section-decoration gradient SVGs. They are
brand imagery — decorative, brand-tinted, not part of any sizing or
spacing system. They belong in `assets/_manifest.md` (and lean-media
already documents them there as "brand-tinted gradient icons" outside
the wordmark logo set). Lean-media also has implicit functional
iconography (a favicon at minimum, plus any future UI surface) — that
lands in a new `design/iconography.md` when the brand adopts the
extension.

### Added

- **`design/iconography.md` (issue #25, Gap 3 from #12).** Optional
  file extending the existing `design/` layer. Frontmatter:
    - `icon_set:` (required when file present) — object with `name`
      (e.g., `lucide`, `material-symbols`, `heroicons`, `phosphor`,
      `tabler`, `feather`, `custom`), optional `version`, optional
      `variant`.
    - `set_authority:` (optional) — URL to the canonical source.
    - `color_treatment:` (required) — enum
      `monochrome` | `brand-color` | `functional` | `mixed-with-rules`.
    - `stroke_vs_fill:` (optional) — enum `stroke` | `fill` | `mixed`.
    - `sizing:` (optional) — named t-shirt sizes (`xs`/`sm`/`md`/
      `lg`/`xl`) mapped to design-token names. Sizing tokens MUST
      be declared in `design-tokens.md` (or `ui-tokens/`); this
      block REFERENCES them by name. Hex/px values are never
      redeclared here.
    - `spacing:` (optional) — clear-space rules around icons.
      References design-tokens by name where possible.
    - `exceptions:` (optional array) — declared exceptions to the
      one-set rule with `set`, `use_case`, `reason`. Documentation
      only; the validator does not enforce one-set.
  Body sections recommended cover icon set choice, sizing scale,
  color treatment, stroke vs fill, spacing/clear-space, exceptions,
  decorative-vs-functional distinction, and how downstream tools
  consume the layer.

- **`design/icons/` directory (optional).** When the brand ships
  custom icons, source files live here alongside `iconography.md`
  (NOT in `assets/icons/` — keeps icon-system docs and source files
  together). Carries an optional `_manifest.md` analogous to
  `assets/_manifest.md`, with per-icon `filename`, `category`,
  `intended_use`, `alt_text_default` records.

- **Decorative-vs-functional distinction (documented).** The
  `design/` layer description now spells out which side of the line
  a given asset falls on: decorative or gradient "icons" that read
  as brand imagery → `assets/_manifest.md`; functional UI icons that
  participate in the icon system → `design/iconography.md`. Brands
  SHOULD document the distinction explicitly when both surfaces are
  present.

- **Validation rules (3 new):**
    - `iconography-set-resolves` (info) — when
      `icon_set.name: custom` (or another brand-provided set), the
      `design/icons/` directory SHOULD exist with at least one icon
      file and a `_manifest.md`. Public libraries (lucide,
      material-symbols, heroicons, etc.) need no local files; the
      library is the source of truth.
    - `iconography-sizing-references-resolve` (warn) — every value
      in `sizing:` MUST be a token name that resolves to a token
      defined in `design-tokens.md` or, when present, in
      `ui-tokens/`. The iconography file assigns sizes to existing
      tokens; it MUST NOT redeclare px or rem. Mirrors the v1.5
      `data-viz-color-tokens-resolve` and v1.6
      `image-gen-design-token-references-resolve` rules.
    - `iconography-decorative-distinction-recommended` (info) —
      when both functional iconography and decorative or gradient
      icons exist in a brand, `iconography.md` SHOULD include a
      "Decorative vs functional" body section spelling out which
      assets fall on which side.

- **`templates/empty-brand/design/iconography.md`** — worked
  example using Lucide as the icon set choice. Demonstrates the
  full convention: required and optional frontmatter fields with
  inline comments, the recommended body sections, and the
  decorative-vs-functional distinction stated explicitly.

### Changed

- `contract_version` bumped to `1.6.1`.
- `design:` layer description in `brand.yaml` extended with a
  paragraph documenting the iconography file convention and a
  paragraph documenting the decorative-vs-functional distinction.
- `README.md` updated: new "What's new in v1.6.1" section above
  the v1.6 section.

### Backward compatibility

- v1.6.0 brands (NEXT90, gramatr-brand, lean-media-brand) validate
  cleanly against v1.6.1 with zero modifications. None currently
  has `design/iconography.md` and the file is `recommended: true,
  required: false`. Lean-media's existing documentation of its
  decorative gradient SVGs in `assets/_manifest.md` is already
  spec-conforming under the new distinction (no edits needed).
- No required field added to any existing layer.
- No enum tightened, no existing field renamed or removed.
- The three new validation rules trigger only when a brand
  populates `design/iconography.md`; brands without the file see
  no new warnings or errors. The
  `iconography-decorative-distinction-recommended` rule fires only
  when BOTH the iconography file AND decorative-icon entries in
  `assets/_manifest.md` exist, so it cannot warn against any
  existing brand today.

### Out of scope (deferred)

- **First-adopter brand population.** lean-media-brand is the
  natural first adopter — it already has the decorative-side
  documented (gradient SVGs in `assets/_manifest.md`) and an
  implicit functional iconography surface (favicon plus any future
  UI) waiting to be declared. The first-adopter PR
  (lean-media-brand populates `design/iconography.md`, references
  the existing decorative entries to make the distinction
  explicit) will be filed as a follow-up sub-issue under epic #13.
- **Per-icon SVG validation.** Walking custom icon files for
  consistency (single stroke width, viewbox, etc.) is downstream
  tooling territory, not brand-spec validation surface.
- **Icon usage analytics.** Cross-referencing which icons a brand
  actually uses across `examples/`, `prompts/`, and `messaging/` is
  v2 territory if it ever lands.
- **Validator implementation of the new rules.** The reference
  validator (`gramatr/brand-spec-validator`) picks up v1.6.1 on
  the next maintainer-triggered spec-sync run. The two info-
  severity rules are straightforward presence checks; the
  `iconography-sizing-references-resolve` warn rule shares shape
  with the v1.5 `data-viz-color-tokens-resolve` and v1.6
  `image-gen-design-token-references-resolve` rules and reuses
  that implementation pattern.

## [1.6.0] — 2026-05-10

Closes Phase 2 sub-issue
[`gramatr/brand-spec#24`](https://github.com/gramatr/brand-spec/issues/24)
of the brand-spec coverage epic
[`#13`](https://github.com/gramatr/brand-spec/issues/13). Addresses
gap #2 from audit
[`#12`](https://github.com/gramatr/brand-spec/issues/12) — image
generation prompt guidelines, named directly as a user priority and
representing the highest-value AI-tooling slot the spec was missing.

Semver justification: **MINOR bump (per `VERSIONING.md`).** A new
top-level layer is the canonical example of "a genuinely new
capability domain" warranting a minor bump (the policy gives
`interactions/` and `taxonomies/` as named examples; v1.4's
`journey/` and v1.5's `data-viz/` set the same precedent). No
required field added; no enum tightened; no existing field renamed
or removed. v1.5.0 brands (NEXT90, gramatr-brand, lean-media-brand)
validate cleanly against v1.6 with zero changes — none of them
currently has an `image-generation/` directory and the layer is
`recommended: true, required: false`.

Relationship to existing layers — clarified:

- **`prompts/`** is for PROSE generation (one-pagers, emails, decks,
  long-form). The schemas (style anchors, negative prompts,
  aspect-ratio defaults, people-depiction rules, provenance) diverge
  enough from prose-prompt structure that unifying them would add
  friction; v1.6 keeps them as sibling top-level layers.
- **`design/photography-treatment.md`** (where present) governs
  HUMAN-shot photography. `image-generation/` references it as the
  visual baseline via the new `photography_treatment_ref:`
  frontmatter field — same brand intent, different surface.
- **`assets/_manifest.md`** is the source of truth for clear-space,
  minimum-size, and wordmark metadata. `image-generation/brand-overlays.md`
  references it via `assets_manifest_ref:`; never restates values.
- **`design-tokens.md`** is the source of truth for color values.
  `image-generation/brand-overlays.md` references token names by
  name (e.g., `--brand-primary`); never declares hex literals.

### Added

- **`image-generation/` layer (issue #24, Gap 2 from #12).** New
  top-level layer, `required: false, recommended: true`. Canonical
  home for AI image-generation conventions (Midjourney, DALL-E,
  Imagen, Flux, and successors). Layer files:
    - `image-generation/_framework.md` — discipline-level
      documentation (scope, when to use vs commission, relationship
      to `design/photography-treatment.md`, downstream consumption
      model).
    - `image-generation/style.md` — visual goal, style anchors,
      composition defaults, aspect-ratio defaults per channel.
      Optional `photography_treatment_ref:` frontmatter pointer to
      the brand's photography-treatment file. Optional
      `aspect_ratios:` map keyed by channel slug. (Folds the
      original issue sketch's separate `style-anchors.md` and
      `composition.md` — the two split cleanly only at very high
      coverage and most brands benefit from a unified style file.)
    - `image-generation/subject-matter.md` — what the brand depicts,
      what the brand never depicts. Optional `themes:` and
      `forbidden_themes:` frontmatter arrays for machine-readable
      cases (AI agents picking subjects from a controlled set);
      free-form body prose remains canonical.
    - `image-generation/negative-prompts.md` — universal-negatives
      array always appended to generation requests, plus
      brand-specific exclusions. Inverse frame of subject-matter
      and style; kept as a separate file because the authoring mode
      is different (a checklist, not narrative) and AI agents
      consume the array directly without reading body.
    - `image-generation/people.md` — when and how the brand depicts
      humans. Optional `age_range`, `diversity_intent`, and
      `forbidden_likenesses` frontmatter fields. The most ethically
      loaded slot in the layer; the schema documents the fields,
      brands decide their positions.
    - `image-generation/brand-overlays.md` — logo placement,
      watermarks, brand color overlays, attribution. References
      `assets/_manifest.md` (clear-space) and `design-tokens.md`
      (color tokens). Folds the original issue sketch's
      `color-treatment.md` and `brand-overlays.md` — both concern
      post-generation visual treatment that ties the image back to
      brand assets and tokens.
    - `image-generation/provenance.md` — what the brand captures
      for every generated image, what the brand requires before
      publishing, user-facing labeling convention, approval
      workflow, retention/audit, jurisdictional considerations.
      Optional `required_when_published:` and
      `recommended_when_published:` frontmatter arrays declare the
      brand's required vs recommended provenance field set; v1.6
      recommends a minimum of `model`, `prompt` (or `prompt_hash`),
      `generation_date`, `brand_approval_status`.

  All seven files are optional within the layer; brands MAY ship
  any subset.

  **Granularity decision.** The original issue sketch listed nine
  files; v1.6 ships seven by folding two pairs (style-anchors +
  composition → `style.md`; color-treatment + brand-overlays →
  `brand-overlays.md`). The cuts target ergonomics: brands writing
  these files benefit from the unified surface, and the underlying
  validation rules treat the consolidated files identically to the
  sketched separate files.

- **Model-agnostic in v1.** v1.6 prescribes brand intent — what the
  image should look like, what it must never depict — without
  branching by generator. Per-model variants (e.g.,
  `image-generation/midjourney.md`) MAY be added in a future patch
  when real divergence in prompt syntax surfaces; v1.6 does not
  bake that in.

- **Validation rules (6 new):**
    - `image-gen-photography-treatment-link-valid` (warn) — when
      `image-generation/style.md` declares
      `photography_treatment_ref:`, the referenced file MUST
      exist; when a brand has a photography-treatment file but
      `style.md` does not reference it, validators warn so the
      visual baseline stays consistent across human-shot and
      AI-generated imagery.
    - `image-gen-overlay-manifest-link-valid` (warn) — when
      `image-generation/brand-overlays.md` declares
      `assets_manifest_ref:`, the referenced file MUST exist.
      Encourages reference-not-duplication of clear-space rules.
    - `image-gen-design-token-references-resolve` (error) — every
      design-token reference (token names enclosed in backticks,
      conventionally prefixed with `--`) in
      `image-generation/brand-overlays.md` MUST resolve to a token
      defined in `design-tokens.md` or, when present, in a file
      under `ui-tokens/`. Mirrors the v1.5
      `data-viz-color-tokens-resolve` rule.
    - `image-gen-people-required-fields-when-present` (warn) —
      when `image-generation/people.md` exists, it SHOULD declare
      at least one of `age_range`, `diversity_intent`, or
      `forbidden_likenesses` in frontmatter, OR explicitly state in
      the body that the brand does not generate imagery
      containing people.
    - `image-gen-provenance-required-when-published` (info) — when
      `image-generation/provenance.md` declares
      `required_when_published:`, downstream pipelines SHOULD
      enforce that every published image carries those fields.
      Brand-spec validators treat this as informational against
      the brand repo itself; pipeline-level enforcement is the
      consumer's responsibility.
    - `image-gen-framework-recommended` (info) — when any file
      under `image-generation/` exists, `_framework.md` SHOULD
      also exist.

- **`templates/image-generation-example/`** — worked example for a
  hypothetical Acme Regenerative Media brand (B2B agriculture media
  publisher). The agriculture framing was chosen so the template
  can demonstrate the cross-link to a hypothetical
  `design/photography-treatment.md` (the real-world parallel is
  `lean-media-brand`'s photography-treatment file). Demonstrates
  all seven layer files: style anchors with concrete film-stock and
  documentary-tradition references; subject-matter with canonical
  themes (working agriculture, rural landscape, working equipment)
  and forbidden themes (urban-tech-stock, server-room, abstract
  network mesh, aspirational homestead lifestyle); universal
  negative prompts plus contextual additions; people-depiction with
  declared age range, diversity intent, and forbidden likenesses;
  brand overlays referencing the assets manifest and design tokens;
  provenance with required/recommended field sets and a
  caption-suffix labeling convention.

### Changed

- `contract_version` bumped to `1.6.0`.
- `README.md` updated: new "What's new in v1.6" section above the
  v1.5 section. Layer overview pattern matches v1.4 and v1.5.

### Backward compatibility

- v1.5.0 brands (NEXT90, gramatr-brand, lean-media-brand) validate
  cleanly against v1.6 with zero modifications. None of the three
  currently has an `image-generation/` directory; the layer is
  `recommended: true, required: false`.
- No required field added to any existing layer.
- No enum tightened, no existing field renamed or removed.
- The six new validation rules trigger only when a brand populates
  the image-generation layer; brands without the layer see no new
  warnings or errors. The
  `image-gen-photography-treatment-link-valid` rule is the one
  exception — it can warn against a brand that has
  `design/photography-treatment.md` but no
  `image-generation/style.md` referencing it; the warning is
  informational and brands without an image-generation layer are
  not affected.

### Out of scope (deferred)

The following are intentionally NOT in v1.6 and will land in later
phases:

- **First-adopter brand population.** lean-media-brand has the
  strongest evidence base (the `design/photography-treatment.md`
  file documents the visual baseline this layer would reference
  end-to-end). The first-adopter PR (lean-media or gramatr-brand
  populates `image-generation/`, cross-links to its
  photography-treatment) will be filed as a follow-up sub-issue
  under epic #13.
- **Iconography (#25)** — sibling Phase 2 issue; intentionally
  separate PR.
- **Per-model variants** (`image-generation/midjourney.md`,
  `image-generation/dalle.md`, etc.). v2 territory once real
  divergence in prompt syntax across generators stabilizes.
- **`video_generation/` sibling layer.** The natural extension when
  AI video generation matures into a brand-relevant surface; v1.6
  reserves the namespace by convention but ships nothing.
- **Validator implementation of the new rules.** The reference
  validator (`gramatr/brand-spec-validator`) picks up v1.6 on the
  next maintainer-triggered spec-sync run. The
  link-validity and people-required-fields rules are
  straightforward; the
  `image-gen-design-token-references-resolve` error rule shares
  shape with the v1.5 `data-viz-color-tokens-resolve` rule and
  will reuse that implementation pattern.

## [1.5.0] — 2026-05-10

Closes Phase 2 sub-issue
[`gramatr/brand-spec#23`](https://github.com/gramatr/brand-spec/issues/23)
of the brand-spec coverage epic
[`#13`](https://github.com/gramatr/brand-spec/issues/13). Addresses
gap #1 from audit
[`#12`](https://github.com/gramatr/brand-spec/issues/12) — the
clearest "missing slot is forcing repetition" signal in the entire
audit (NEXT90's chart guidance duplicated across three channel
guides).

Semver justification: **MINOR bump (per `VERSIONING.md`).** A new
top-level layer is the canonical example of "a genuinely new
capability domain" warranting a minor bump (the policy gives
`interactions/` and `taxonomies/` as named examples; v1.4's
`journey/` set the same precedent). No required field added; no
enum tightened; no existing field renamed or removed. v1.4.0
brands (NEXT90, gramatr-brand, lean-media-brand) validate cleanly
against v1.5 with zero changes — none of them currently have a
`data-viz/` directory and the layer is `recommended: true,
required: false`.

### Added

- **`data-viz/` layer (issue #23, Gap 1 from #12).** New top-level
  layer, `required: false, recommended: true`. Canonical home for
  charting and data-visualization conventions. Layer files:
    - `data-viz/_framework.md` — discipline-level documentation
      (scope, boundaries with `design-tokens.md`, downstream
      consumption model).
    - `data-viz/colors.md` — assigns DATA ROLES (categorical-1,
      sequential-low, diverging-positive, semantic-negative, etc.)
      to design-token names. Hex values are NEVER redeclared here;
      the token reference is the single source of truth.
    - `data-viz/chart-types.md` — per-brand allowed/forbidden
      chart-type policy. Optional `allowed:` and `forbidden:`
      frontmatter arrays drive the
      `data-viz-chart-type-coverage-warn` validator.
    - `data-viz/axes-and-grid.md` — origin behavior, gridline
      conventions, tick density, label rotation. Optional
      `origin_policy:` frontmatter (`always-zero` / `free` /
      `contextual`).
    - `data-viz/annotations.md` — big-stat callouts, inline
      highlights, source citation rules. Optional
      `source_citation_required:` boolean.
    - `data-viz/numbers.md` — currency, percentage, large-number
      abbreviation, precision rules. Optional `locale:` (BCP-47)
      and `currency_default:` (ISO 4217) frontmatter.
    - `data-viz/tables.md` — header treatment, column alignment,
      empty-cell convention.
  All seven files are optional within the layer; brands MAY ship
  any subset.

- **Color-role taxonomy.** The four canonical families
  (categorical / sequential / diverging / semantic) follow
  established information-design practice — Tableau, IBM Carbon
  Charts, Material Design Data Viz, and Observable Plot all use the
  same shape. The `colors.md` file frontmatter carries an optional
  `role_taxonomy:` (`categorical` / `semantic` / `sequential` /
  `diverging` / `mixed`) for downstream tools.

- **Validation rules (5 new):**
    - `data-viz-color-tokens-resolve` (error) — every design-token
      reference in `data-viz/colors.md` MUST resolve to a token
      defined in `design-tokens.md` or under `ui-tokens/`.
    - `data-viz-no-hex-redeclaration` (warn) — hex literals SHOULD
      NOT appear in `data-viz/*.md` body; they belong in
      `design-tokens.md`.
    - `data-viz-chart-type-coverage-warn` (warn) — when a brand
      declares `forbidden:` chart types, validators warn if a
      `messaging/channel-*.md`, `prompts/*.md`, or `examples/*.md`
      file references a forbidden type.
    - `data-viz-annotation-source-required` (warn) — when
      `source_citation_required: true`, downstream artifact
      generators MUST attach source citations to numeric
      annotations.
    - `data-viz-framework-recommended` (info) — when any file
      under `data-viz/` exists, `_framework.md` SHOULD also exist.

- **`templates/data-viz-example/`** — worked example for a
  hypothetical Acme Analytics brand. Demonstrates all seven layer
  files. Shows: color roles assigned to token references (no hex
  literals); a chart-type policy that forbids pie/donut/3D and
  radar; number formatting for currency / percentage / scientific-
  notation thresholds; the canonical big-stat callout format with
  required source citation.

### Changed

- `contract_version` bumped to `1.5.0`.
- `README.md` updated: new "What's new in v1.5" section above the
  v1.4 section. Layer overview pattern matches v1.4.

### Migration recommendation (not enforced)

Brands carrying duplicate chart guidance in channel files (e.g.,
NEXT90's deck/social/landing-page channel guides each restating
chart palette, axis, big-stat, and source-citation conventions)
SHOULD consolidate into `data-viz/` when next revisiting those
guides. The migration is opportunistic — channel files continue to
validate cleanly with the prose intact, and brands move at their
own pace — but the duplication is a known drift risk and
`data-viz/` is the canonical home going forward. Channel files
SHOULD then reference the data-viz layer rather than restate its
rules, retaining only the channel-specific deviations (chart count
per surface, image vs. interactive rendering, etc.).

### Backward compatibility

- v1.4.0 brands (NEXT90, gramatr-brand, lean-media-brand) validate
  cleanly against v1.5 with zero modifications. None of the three
  currently has a `data-viz/` directory; the layer is
  `recommended: true, required: false`.
- No required field added to any existing layer.
- No enum tightened, no existing field renamed or removed.
- The five new validation rules trigger only when a brand
  populates the data-viz layer; brands without the layer see no
  new warnings or errors.

### Out of scope (deferred)

The following are intentionally NOT in v1.5 and will land in later
phases:

- **First-adopter brand population.** NEXT90 has the strongest
  evidence base for the layer (chart guidance in three channel
  guides). The first-adopter PR (NEXT90 populates `data-viz/`,
  collapses the duplicated channel-guide prose) will be filed as a
  follow-up sub-issue under epic #13.
- **Iconography (#25)** and **image-generation (#24)** — sibling
  Phase 2 issues; intentionally separate PRs.
- **Cross-layer reference syntax** for channel guides to point at
  data-viz files. The migration recommendation above describes the
  pattern in prose; a structured reference shape (frontmatter
  pointer or markdown link convention) is v2 territory.
- **Validator implementation of the new rules.** The reference
  validator (`gramatr/brand-spec-validator`) picks up v1.5 on the
  next maintainer-triggered spec-sync run; the four
  warn/info-severity rules are straightforward, the
  `data-viz-color-tokens-resolve` error rule requires the validator
  to parse `design-tokens.md` body for token definitions and
  cross-reference (similar shape to the v1.3
  `register-inheritance-cannot-subtract` rule).

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
with explicit, structured attribution. **KYKC (Know Yourself, Know Your Customer)** and
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
