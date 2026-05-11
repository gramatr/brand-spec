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

See [`VERSIONING.md`](./VERSIONING.md) for what counts as patch / minor / major and the discipline that governs version bumps.

## Release discipline — tag every version

**Every merged version MUST be tagged at the merge SHA.** Tags are how downstream consumers (the validator's vendored sync, brands that pin to a version, CI that validates against a specific version) reliably reference a release. An untagged main HEAD is not a version anyone can rely on.

Steps when a version-bumping PR merges:

```bash
git checkout main && git pull
git tag -a v<X.Y.Z> -m "v<X.Y.Z> — <one-line summary matching CHANGELOG entry>" <merge-sha>
git push origin v<X.Y.Z>
```

The tag name MUST match the `contract_version` field in `brand.yaml` for that release. The CHANGELOG entry justifies the specific bump per [`VERSIONING.md`](./VERSIONING.md).

If a PR merges without a version bump (docs-only, internal cleanup), no tag is required. Anything that touches `brand.yaml`'s `contract_version` requires a tag at the merge SHA.

## License

By contributing you agree your contribution will be licensed under the MIT License (see `LICENSE`).
