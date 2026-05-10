# brand-spec

**A portable, machine-readable representation of a brand — designed to be loaded by AI agents.**

`brand-spec` defines the file/folder layout, frontmatter conventions, and validation rules a brand repo follows so any AI tool can clone it, parse it, and immediately write on-brand. It is the **brand context window for the LLM era**: voice, vocabulary, personas, design tokens, validated examples, and prompt templates, all in one consistent shape.

The schema lives in [`brand.yaml`](./brand.yaml). It was reverse-engineered from a real production brand repo, not invented top-down — every field in the schema is backed by a working example.

## For AI agents

If you are an AI agent that needs to operate as a brand:

1. **Clone the brand repo.** Any repo conforming to this spec is self-describing.
2. **Read `brand.yaml`** to discover which layers the brand provides.
3. **If `agent-context.md` exists, load it first.** It is a 2-4K-token digest of the brand designed to fit a single prompt window. Its frontmatter `priority_layers` tells you which full layers to load next when you have more budget.
4. **Otherwise, load layers in priority order:**
   - **First (always):** `identity.md`, `voice.md`, `vocabulary.md` — who the brand is, how it sounds, words to use and avoid.
   - **Next:** `design-tokens.md`, `personas/` — visual system and audience definitions.
   - **Last (for few-shot anchoring):** `examples/`, `prompts/` — validated reference outputs and content-type briefs.
5. **Respect frontmatter.** Fields like `validated: true` on a prompt mean the listed examples are trustworthy anchors. Fields like `status: open` on a design spec mean the answer is not yet settled.

The repo is the LLM context. No API call required.

## For tools

`brand-spec` is consumer-agnostic. Any tool that wants to validate, ingest, render, or generate against a brand can read `brand.yaml` and walk the repo.

[grāmatr Studio](https://github.com/gramatr) is the reference implementation: it ingests brand-spec-conforming repos, validates them against `brand.yaml`, and writes the parsed brand into runtime tables. But the spec is open — a CMS, a design tool, a code generator, or a custom in-house pipeline can consume the same repo with no Studio dependency.

## Agent context layer

`agent-context.md` is the entry point for context-constrained agents. It is **recommended, not required**: small brands can skip it and let agents load the full layer set; larger brands benefit from a curated digest that compresses the brand into one prompt window.

Frontmatter:

```yaml
---
schema_version: 1
summary: "One paragraph describing the brand: who they are, who they serve, how they sound."
priority_layers:
  - identity
  - voice
  - vocabulary
  - personas
  - examples
---
```

Body (target ≈ 2-4K tokens):

- Voice in one paragraph
- Top 5 vocabulary do/don't items
- Persona summary (matrix shape + the cells that matter most)
- Anything else essential — proprietary concepts, what the brand is NOT, hard tone rules

Think of it as the brand's `CLAUDE.md`: the file you load when you can only load one file.

## What's new in v1.2.1

Two coupled additive conventions on the assets layer. Both are optional
and backward-compatible — v1.2 brands validate cleanly against v1.2.1
with zero changes.

1. **Asset format preference (PNG accepted, SVG preferred for future).**
   PNG (or other raster) is an accepted format for any logo file. SVG is
   preferred for future additions or replacements when vector source
   becomes available — useful for scaling beyond native resolution,
   programmatic recoloring, vector ops in design tools, and large-format
   print. PNG remains the right choice for email, social profile/cover
   images, and any context bounded to native resolution. Brands MUST
   NOT auto-trace PNG to SVG for typographic wordmarks (tracing
   produces noisy polygons that fail to match optical kerning).

2. **Wordmark structured metadata.** An optional `wordmark:` frontmatter
   block on `assets/_manifest.md` captures the canonical glyph sequence
   (`text`), an ASCII fallback (`text_ascii`, required only when `text`
   contains non-ASCII characters), and metadata about typeface, weight,
   case, and glyph behavior. AI agents and downstream tools can use
   this to render brand-bearing artifacts correctly across surfaces
   that vary in their support for non-ASCII characters.

See the schema's `assets` layer in `brand.yaml` for full field
definitions and `templates/empty-brand/assets/_manifest.md` for the
canonical example.

### Wordmark guidance for AI agents

When generating brand-bearing artifacts (decks, emails, code, social
posts, package metadata), an agent SHOULD read the `wordmark:` block
from `assets/_manifest.md` and apply this precedence rule: `text` is
the canonical form and SHOULD be used wherever the consuming surface
can render it. `text_ascii` is the fallback ONLY when the surface
cannot render the canonical glyphs — typical examples are CLI output
in environments with restricted encodings, URLs, npm package names,
GitHub org slugs, and social handles that reject non-ASCII characters.

Do not silently substitute `text_ascii` when the canonical form works.
If the brand also documents a two-spelling rule in `vocabulary.md`,
defer to that — the wordmark block exists to make the canonical/ASCII
pair machine-readable, not to override editorial guidance.

## What's new in v1.2

Four additive schema changes, all backward-compatible with v1.1:

1. **Multi-register voice.** Brands can now use a `voice/` directory
   with one file per register (e.g., `voice/registers/corporate.md`,
   `voice/registers/engineering.md`) plus a `voice/README.md` index.
   Use this when one brand has multiple distinct on-brand voices that
   vary by channel or context. Single-register `voice.md` (the v1.1
   default) is unchanged. A brand uses one pattern OR the other —
   never both. See [`templates/multi-register-voice-example/`](./templates/multi-register-voice-example/).

2. **Source authority metadata.** Optional `source_authority`
   frontmatter on any file declares whether the file is `canonical`,
   `mirror`, `historical`, or `fixture-only`. Lets validators and
   agents know not to treat an intentionally-stale fixture as truth.
   Defaults to `canonical` when omitted.

3. **File-level visibility classification.** Optional `visibility`
   frontmatter on any file: `public`, `customer`, `internal`, or
   `sales-enablement`. Tools generating public artifacts MUST filter
   out non-public files. Defaults to `public` when omitted.
   Backward-compat: legacy `audience: internal` on
   `proof/competitive.md` is treated as `visibility: internal` with a
   migration warning.

4. **Data file lineage.** When `source_authority.status: mirror`, an
   optional `refresh:` sub-block declares cadence (`manual`,
   `on-deploy`, `weekly`, `monthly`, `on-source-change`),
   `last_synced` date, and optional `sync_method`. Validators warn
   when mirrors look stale relative to declared cadence.

All four are additive. v1.1 brands validate cleanly against v1.2 with
zero changes.

See [`CHANGELOG.md`](./CHANGELOG.md) for the full version history.

## Required vs. Recommended vs. Optional

The three levels are explicit on every layer in `brand.yaml`:

| Level | Meaning | Consumer behavior |
|---|---|---|
| `required: true` | The brand is not valid without it. | Refuse to ingest. |
| `recommended: true` | Tool will ingest, but downstream features that depend on this layer will be unavailable or produce lower-quality output. | Ingest with warnings. |
| (neither) | Optional. Purely additive. | Ingest silently. |

When a layer is `recommended`, the contract honestly reflects that we have limited evidence about whether a future brand could safely omit it. The contract will tighten in v2 once a second and third brand are ingested.

## Validating a brand repo

A validator walks the repo and checks against `brand.yaml`:

1. Required files exist at the documented paths.
2. Required directories exist; required `min_files` thresholds are met for pattern-based contents (e.g., at least one `for-{audience}.md`).
3. Every `---`-delimited frontmatter block parses as YAML.
4. Required frontmatter fields are present on each file type.
5. Cross-file rules under `validation:` hold (e.g., a `validated: true` prompt must list real example paths; ui-tokens layers stack in order).

A reference validator implementation is planned. Until then, the schema is human-readable enough to spot-check by eye.

## Scaffolding a new brand

A starter skeleton lives in [`templates/empty-brand/`](./templates/empty-brand/). Copy it as the seed for a new brand repo. Steps:

1. Copy `templates/empty-brand/` to your new repo root.
2. Fill the required frontmatter fields in `identity.md`, `voice.md`, `vocabulary.md`, `design-tokens.md`, and at least one `messaging/for-{audience}.md`.
3. Add `recommended` layers as the brand matures: persona matrix, channel guides, proof library, data registry, prompts, examples.
4. Once the brand is stable enough to summarize, write `agent-context.md`.
5. Run validation. Iterate until clean.

## Mapping content into the abstract slots

The contract describes universal patterns. Each brand fills those patterns with its own values. A worked hypothetical: a B2B SaaS company called Acme that sells a single product to three customer segments would map content like this:

| Pattern in spec | Acme's instance |
|---|---|
| `messaging/for-{audience}.md` | `for-enterprise.md`, `for-mid-market.md`, `for-smb.md` |
| `personas/{audience}-{role}.md` | `enterprise-champion.md`, `enterprise-buyer.md`, `mid-market-champion.md`, … |
| `messaging/channel-{channel}.md` | `channel-deck.md`, `channel-email.md`, `channel-landing-page.md` |
| `product/{product-slug}.md` | `product/acme-platform.md` |
| `data/{topic}.yaml` | `data/team.yaml`, `data/market.yaml`, `data/stats.yaml` |
| `prompts/{content-type}.md` | `one-pager.md`, `email-sequence.md`, `deck-sales.md` |

The audience taxonomy (enterprise / mid-market / smb), the role taxonomy (champion / buyer / end-user), and the channel list are all **brand-defined**. Another brand could just as easily use `commercial / public-sector / non-profit` audiences, or audiences keyed on industry vertical, or no segmentation at all. The pattern is in the spec; the values are in the brand.

## Reading the contract

`brand.yaml` is organized as:

- `conventions:` — cross-cutting rules (file naming, frontmatter syntax, reserved prefixes, common fields seen across multiple file types).
- `layers:` — one entry per layer (agent_context, identity, voice, vocabulary, design, ui_tokens, messaging, personas, proof, data, product, examples, prompts, design_specs, meta). Each layer declares its required level, the files or directories it owns, and the frontmatter contract for each file type.
- `validation:` — cross-layer rules a validator should enforce.

## Known gaps for v2

These are real gaps observed during the v1 reverse-engineering. They are intentionally **not** fields in the v1 schema — they will be added when a second and third brand have been ingested or a clear product need emerges.

1. **No top-level brand manifest.** There's no single file declaring "this repo conforms to brand-spec v1." Today the conformance is implicit. v2 SHOULD add a top-level manifest (e.g., `.brand-spec.yaml`) that pins the schema version and lists which optional layers the brand opts into.
2. **Ambiguous required vs. recommended for personas, proof, data, product, prompts, examples.** All are marked `recommended` because we only have one production instance and cannot yet tell which would degrade consumers to a degree that warrants `required`. v2 should harden these once more brands exist.
3. **No taxonomy declaration.** The brand's `audience`, `role`, and `channel` taxonomies are implicit (they emerge from the filenames and frontmatter values present). v2 SHOULD add an explicit taxonomy declaration so validators can catch typos and misclassifications.
4. **`data/` schemas are file-internal.** Each YAML file in `data/` documents its own schema in a comment header. That's fine for human readers but invisible to a validator. v2 should either link each `data/*.yaml` to a JSON Schema or formalize a tiny meta-schema for data files.
5. **`design-tokens.md` is prose.** Colors, type, and email tokens are documented in markdown tables. A machine-readable token export (e.g., `design-tokens.json` in the W3C Design Tokens format) would let downstream tools consume them directly. The `ui-tokens/` files point in this direction but are also still prose.
6. **No multi-product story.** `product/` works as a directory pattern, but cross-product relationships (suite hierarchy, dependencies, shared messaging) have no schema slot.
7. **Persona `_archive/` convention is by example, not by rule.** The underscore-prefix-means-not-content convention is documented in `conventions.reserved_prefixes` but isn't enforced beyond a comment.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). Schema changes require a version bump. Discuss larger changes in an issue first.

## License

MIT. See [`LICENSE`](./LICENSE).
