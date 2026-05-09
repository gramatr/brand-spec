# Contributing

Contributions to `brand-spec` are welcome. The spec is intentionally minimal and reverse-engineered from real-world use — please bring evidence, not theory.

## Philosophy

- **Reverse-engineer, don't invent.** A new field or layer SHOULD be backed by at least one real brand repo that already needs it.
- **Honest required levels.** Don't tighten `recommended` to `required` without evidence from multiple brands. Don't loosen `required` without strong reason.
- **Brand-agnostic.** The spec describes patterns; values belong in the brand. If a proposal names a specific industry, audience, or product, it probably belongs in a brand repo, not the spec.

## How to contribute

1. **Open an issue first** for anything beyond a typo or clarification. Schema changes especially need discussion before a PR.
2. **Schema changes require a version bump.** Update `schema_version` and `contract_version` in `brand.yaml`. Note the change in the README's "Known gaps" section if it closes one.
3. **Update `templates/empty-brand/`** if your change adds or renames a required file or frontmatter field.
4. **Update the README** if your change affects how an agent or tool would load or validate a brand.

## Versioning

`schema_version` is an integer that bumps for breaking changes (renames, removals, type changes, required-level tightening). `contract_version` is a semver string for finer-grained tracking.

## License

By contributing you agree your contribution will be licensed under the MIT License (see `LICENSE`).
