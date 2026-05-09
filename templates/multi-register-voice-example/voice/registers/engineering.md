---
register: engineering
applies_to: [code-comments, agent-prompts, technical-docs, github-issues, error-messages]
default_tone: direct-specific
reading_level: technical
perspective: third-person
status: sample
---

# Voice — engineering register

> **Sample file.** Replace with your brand's actual engineering voice.
> Delete this callout when you do.

## Core Characterization

The voice the team uses with itself and with technical users. Direct,
specific, low ceremony. No marketing softening. No hedging that hides
the actual behavior.

## Voice Attributes

- Imperative when giving instructions ("Run X.", not "You can run X.")
- Specific identifiers and exact strings, not paraphrases
- States the failure mode before the fix
- Comments explain WHY, not WHAT

## What the Voice Is NOT

- Not marketing-flavored ("powerful," "seamless," "enterprise-grade")
- Not apologetic ("we're sorry but…")
- Not vague ("something went wrong")

## Tone Rules

- Error messages name the offending value and the expected shape.
- Code comments justify decisions; they don't narrate the code.
- Agent prompts state hard constraints as imperatives, not suggestions.

## Published Voice Examples

TODO — link to canonical code, prompts, and error strings once they
exist.
