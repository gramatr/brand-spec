# Versioning Policy

`brand-spec` follows [semver](https://semver.org), but because the schema is consumed by AI agents and downstream tools (not just humans reading docs), the lines between patch / minor / major matter more than usual. This doc fixes those lines so future contributors don't drift.

## The discipline

| Bump | When | Examples |
|---|---|---|
| **Patch** (1.x.**y**) | Additive optional fields, new conventions, doc clarifications, template tweaks. Existing brands continue validating with zero changes. | New optional frontmatter field. New recommended body section. Template improvements. New validation rule that emits `info` or `warn` only. |
| **Minor** (1.**x**.0) | A genuinely new capability domain — a new layer, a new structural concept that opens use cases prior versions couldn't support. Still backward-compatible. | A new `interactions/` layer for documenting brand-customer touchpoints. A `taxonomies/` layer brands declare and reference from `applies_to`. |
| **Major** (**x**.0.0) | Breaking changes. A required field is added that existing brands lack. An enum is narrowed. A field is renamed or removed. A layer's required/optional status is tightened. | Renaming `voice` → `voices`. Making `agent-context.md` required. Removing the legacy `audience: internal` backcompat shim. |

## Default to patch

When in doubt, **bump the patch number**. The bias should be conservative — minor bumps signal "something materially new is now possible," and that signal loses meaning if every additive PR claims it.

Most issues currently labeled "v1.3 candidate" or "v1.4 candidate" should be patch-level under this discipline. Examples that would land as patches:

- Register-aware vocabulary deltas (per-register forbidden/preferred terms)
- `corpus/` directory convention for checked-in sample reproducibility
- Enforcement upgrades for existing rules (e.g., `applies_to` baseline-warn → strict-error)
- Additional optional frontmatter fields on existing layers

Examples that would warrant a minor bump:

- A whole new top-level layer (e.g., `taxonomies/`, `interactions/`, `governance/`)
- A new structural primitive (e.g., schema-level support for cross-brand inheritance)
- A new validation severity beyond `error / warn / info`

## Examples that would warrant a major bump

- Making any currently-recommended layer required
- Removing `voice.md` single-register support (forcing all brands to multi-register)
- Renaming `wordmark.text_ascii` to `wordmark.fallback_text`
- Tightening the `status:` enum to drop `draft`

## What this isn't

This policy is **not** about being slow to ship. Patch releases can land daily if there's real work. It's about preserving the meaning of the version number so consumers (AI agents, validators, CI pipelines) can reason about compatibility from the version string alone.

A brand that conforms to v1.3.0 will conform to every v1.3.x and every later 1.x release. A consumer pinning to `>= 1.3, < 2` knows exactly what compatibility surface they're committing to. That guarantee is what semver is for.

## Lesson behind this policy

Between v1.0 and v1.3, the schema bumped through four minor versions in a single working session. Each bump was technically valid (additive, backward-compatible), but the cadence over-signaled change. The retrospective: most of those minor bumps should have been patches.

Going forward: if a PR doesn't introduce a fundamentally new capability domain, it bumps the patch number, full stop.

## Tagging

Every released version gets a git tag at the merge SHA. Tag name matches `contract_version` exactly (`v1.6.1`, `v1.4.0`, etc.). Without tags, downstream consumers (the validator, brand repos pinning a version, CI) have no reliable way to reference a specific release — they end up syncing from `main` and inheriting whatever's there at the moment, which defeats the point of versioning.

Detailed steps for tagging on merge live in [`CONTRIBUTING.md`](./CONTRIBUTING.md). The 9 versions released between v1.0.0 and v1.6.1 were tagged retroactively; v1.6.2+ ships with the tag at merge time.

## Validator versioning

`gramatr/brand-spec-validator` follows its own semver track. Today it's at v0.1.x — pre-1.0 because the validator's API surface (`validateBrand()` signature, `ValidationResult` shape) hasn't been hardened yet. The validator's CHANGELOG and `package.json` description document which `brand-spec` version each release supports. When the validator API stabilizes, it bumps to v1.0.0; until then, expect v0.1.x patches as the spec evolves.
