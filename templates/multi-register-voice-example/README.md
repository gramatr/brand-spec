# Multi-register voice example

Demonstrates the v1.2 multi-register voice pattern. Use this when a brand
has more than one distinct on-brand voice that varies by channel or
context (e.g., corporate marketing voice vs. engineering/agent voice vs.
founder byline voice — all on-brand, contextually distinct).

## Layout

```
voice/
  README.md              # lists registers, documents when each applies
  registers/
    corporate.md         # marketing, website, customer comms
    engineering.md       # code comments, agent prompts, technical docs
```

## Rules

- A brand uses EITHER single-register (`voice.md` at root, the v1.1
  default) OR multi-register (`voice/` directory). Never both — see the
  `voice-pattern-exclusive` validation rule in `brand.yaml`.
- `voice/README.md` (or `voice/_index.md`) is required when the
  multi-register pattern is used. It lists the registers and documents
  precedence rules for ambiguous contexts.
- Each register file requires `register:` and `applies_to:` frontmatter.
  All other voice frontmatter (`perspective`, `default_tone`,
  `reading_level`) follows the v1.1 single-register contract.

Most brands won't need this. Start with `templates/empty-brand/voice.md`;
graduate to multi-register only when the single file is being pulled in
contradictory directions.
