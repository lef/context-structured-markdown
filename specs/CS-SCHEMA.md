---
Type: SPEC
Version: 0.0.1
Updated: 2026-06-28T23:45+09:00
Status: draft
Provenance: L0/contextus
Tags: schema, spec, cs-md, contextus
---

# CS-SCHEMA: Contextus Structured Markdown Schema


## Abstract

CS-SCHEMA defines what constitutes a valid CS-MD document.
It specifies the structural rules, field constraints, and validation
criteria that a CS-MD document MUST satisfy.

CS-MD is a document specification (how to write).
CS-SCHEMA is a schema specification (what makes it valid).
A validator reads CS-SCHEMA (via registry.jsonl) to check CS-MD documents.

## Status of This Document

This is a draft specification. A validator implementation does not yet exist.

## 1. Introduction

### 1.1 Relationship to Other Specs

```
CS-MD      → how to write documents (document spec)
CS-SCHEMA  → what makes a document valid (this spec)
CS-INDEX   → how to index documents
CONTEXTUS-REGISTRY → how to manage the schema data

registry.jsonl → the schema data (CS-SCHEMA's machine-readable form)
```

CS-SCHEMA is the CONCEPTUAL specification. registry.jsonl is its DATA.
A validator reads registry.jsonl to enforce CS-SCHEMA rules.

### 1.2 Terminology

Per RFC 8174. MUST/SHOULD/MAY in ALL CAPITALS have normative meaning.

## 2. Document Identification

### 2.1 CS-MD Document Detection

A file is a CS-MD document if and only if:

1. It is a markdown file (`.md` extension)
2. It contains a YAML frontmatter block with a `Type:` field

Files without a `Type:` field in YAML frontmatter are NOT CS-MD documents
and are not subject to CS-SCHEMA validation. Examples of non-CS-MD files:
README.md, AGENTS.md, CLAUDE.md, rules/*.md.

### 2.2 Type Field as Schema Selector

The `Type:` value in YAML frontmatter determines which schema rules apply:

```
Type: HANDOFF    → HANDOFF schema (§4.1)
Type: TODO       → TODO schema (§4.2)
Type: TASK       → TASK schema (§4.3)
Type: KNOWLEDGE  → KNOWLEDGE schema (§4.4)
Type: BELIEF     → BELIEF schema (§4.4)
Type: DRAFT      → DRAFT schema (§4.5)
Type: SPEC       → SPEC schema (§4.6)
Type: DESIGN     → DESIGN schema (§4.6)
Type: PLAN       → PLAN schema (§4.7)
Type: WORKLOG    → WORKLOG schema (§4.7)
Type: CONSTITUTION → CONSTITUTION schema (§4.8)
```

Unknown Type values MUST NOT cause a validation error (Postel's Law).
The validator SHOULD emit a warning for unregistered types.

### 2.3 Requirements Gradient

Document types follow a workflow/lifecycle gradient that determines
how strictly optional fields and sections are validated:

```
Distilled    KNOWLEDGE, BELIEF, SPEC, DESIGN  → first-wave (References MUST directionally)
Execution    TASK, WORKLOG                    → next-wave (strong upstream references)
Continuity   HANDOFF, CONTEXT                 → exact upstream refs expected; timing differs
Intent       PLAN, TODO                       → references expected; timing differs
Scratch      MINUTES                          → weakest enforcement target
```

When validating a MAY field's presence, the validator SHOULD use this gradient
to determine severity and enforcement wave. The main shared question is not
whether references matter, but which lifecycle class becomes a `BLOCK` target first.

## 3. Common Schema Rules

These rules apply to ALL CS-MD documents regardless of Type.

### 3.1 Structure

```
MUST:  YAML frontmatter block (--- delimited) at start of file
MUST:  Exactly one H1 (# Title) as the first heading after frontmatter
MUST:  Type: field in frontmatter
MUST:  Updated: field in frontmatter (ISO 8601 datetime)
MAY:   Additional frontmatter fields (per registry.jsonl)
MAY:   H2 sections (## Name)
MAY:   H3+ subsections within H2 sections
```

### 3.2 Header Validation

```
MUST:  Frontmatter is valid YAML (key: value pairs)
MUST:  Each field key is non-empty
MUST:  Each field value is non-empty
MUST:  Type value is a string
MUST:  Updated value matches ISO 8601 datetime with timezone (YYYY-MM-DDTHH:MM±HH:MM or Z required)
MAY:   Version value matches SemVer (X.Y.Z) if present
MAY:   Status value is one of registered enum values if present
MAY:   Provenance value matches identity format (repo@branch:hash) if present
MUST:  Unknown fields are accepted without error (Postel's Law)
```

### 3.3 Provenance Validation

If `Provenance:` is present in frontmatter:

```
Identity:  MUST match pattern: <repo>@<branch>:<short-hash>
           repo:       [a-zA-Z0-9_-]+
           branch:     [a-zA-Z0-9_/.-]+
           short-hash: [0-9a-f]{7,}
Chain:     MUST match: <identity>--><identity> (one or more)
           No spaces around -->
```

### 3.4 Section Name Validation

Section names (H2) are validated against registry.jsonl:

```
For each H2 in the document:
  IF the section name is registered for this Type:
    → valid (known section)
  ELSE IF the section name is registered for Type "*":
    → valid (universal section)
  ELSE:
    → valid but unregistered (Postel's Law: accept, MAY warn)
```

MUST sections (per Type) that are ABSENT → validation error.
SHOULD sections that are ABSENT → validation warning.

### 3.5 Marker Validation

```
[NEEDS CLARIFICATION]       → valid in DRAFT, SPEC, DESIGN
[NEEDS CLARIFICATION: ...]  → valid in DRAFT, SPEC, DESIGN
[NEEDS CLARIFICATION] in KNOWLEDGE → validation error (must be resolved)
[NEEDS CLARIFICATION] in CONSTITUTION → validation error
```

### 3.6 Reference Validation

Internal references `[text](path.md)` and `[text](path.md#heading)`:

```
IF path.md exists in git working tree → valid
IF path.md does NOT exist → validation warning (broken reference)
IF path.md was renamed (git log --follow) → validation warning + suggest fix
```

External references `[text](https://...)`:

```
No validation (external URLs are not checked)
```

## 4. Type-Specific Schema

### 4.1 HANDOFF

```
MUST sections:  ## Task, ## Context
SHOULD sections: ## Blockers, ## Completed
MAY sections:   ## Decisions, ## References, ## Changed Files
MUST NOT:       [NEEDS CLARIFICATION] (HANDOFF must be actionable)
```

### 4.2 TODO

```
MUST:   At least one H2 section (phase or category)
MAY:    Checkbox items (- [ ] / - [x]) within sections
MAY:    Nested items under checkboxes
```

### 4.3 TASK

```
MUST sections:  ## Goal, ## Context
SHOULD sections: ## Acceptance Criteria
MAY sections:   ## References, ## Constraints
MUST NOT:       [NEEDS CLARIFICATION] (TASK must be executable)
```

### 4.4 KNOWLEDGE

```
MUST:    At least one H2 section (topic, with date recommended)
SHOULD:  > Tags: header (for classification: decision/finding/lesson)
MUST NOT: [NEEDS CLARIFICATION] (knowledge must be verified)
```

### 4.4a BELIEF

```
MUST:    current assumptions must remain tied to upstream records
SHOULD:  explicit references to the evidence/spec/minutes that justify the assumption
MUST NOT: drift silently from upstream records
```

### 4.5 DRAFT

```
SHOULD sections: ## 問題 (or ## Problem), ## 未解決 (or ## Open Questions)
MAY:    [NEEDS CLARIFICATION] markers (exploration is expected)
MAY:    Any other sections (free-form exploration)
```

### 4.6 SPEC / DESIGN

```
SHOULD:  > Version: header (design documents evolve)
SHOULD:  L2-defined MUST sections (L2-dev may require ## 背景, ## 設計決定)
MUST:    [NEEDS CLARIFICATION] resolved before Status: confirmed
MAY:     [NEEDS CLARIFICATION] when Status: draft or discussion
```

### 4.7 PLAN

```
No required sections (free-form intent expression)
SHOULD:  Enough context for an agent to ask clarifying questions
```

### 4.7a WORKLOG

```
SHOULD:  point to the TASK and the evidence/attempt trail it records
MAY:     use ## References for exact upstream links
```

### 4.8 CONSTITUTION

```
MUST sections:  ## Inherited Principles
SHOULD: > Version: header (constraints are versioned)
MUST NOT: [NEEDS CLARIFICATION] (constraints must be unambiguous)
```

## 5. Validation Levels

A validator SHOULD support multiple severity levels:

| Level | Meaning | Example |
|---|---|---|
| ERROR | Document is invalid | Missing MUST field, [NEEDS CLARIFICATION] in KNOWLEDGE |
| WARNING | Document is suboptimal | Missing SHOULD section, broken reference |
| INFO | Observation | Unregistered field, unregistered section name |

A validator MUST NOT reject documents for INFO-level issues (Postel's Law).

## 6. Schema Extensibility

### 6.1 L2/L3 Extensions

L2 and L3 layers MAY add:
- New MUST/SHOULD sections for specific Types
- New header fields (registered in registry.jsonl with `registered_by`)
- New Type values

Extensions MUST NOT remove or weaken Common Schema Rules (§3).

### 6.2 Validation Profiles

A validator MAY support profiles corresponding to layers:

```
--profile L0          → validate Common Rules only
--profile L2-dev      → validate Common + L2-dev extensions
--profile L3-dev-sh   → validate Common + L2-dev + L3-dev-sh extensions
```

## 7. Registry as Schema Data

`registry.jsonl` is the machine-readable form of CS-SCHEMA.

The validator reads registry.jsonl to determine:
- Which header fields exist and their MUST/MAY levels
- Which sections are required per Type
- Which markers are allowed in which Types
- Valid enum values for Status, Type, etc.

If a field/section appears in the document but NOT in registry.jsonl,
the validator MUST accept it (Postel's Law) and MAY emit an INFO message.

## 8. Future Work

- [ ] Validator implementation (shell script: grep + awk)
- [ ] Lint integration (pre-commit hook, CI)
- [ ] VS Code extension (real-time validation)
- [ ] External tool (ductus pattern: static binary for advanced checks)

## References

- [CS-MD](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-MD.md) — Document specification
- [CS-INDEX](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-INDEX.md) — Index specification
- [CONTEXTUS-REGISTRY](https://github.com/lef/contextus/blob/main/specs/CONTEXTUS-REGISTRY.md) — Registry management
- [JSON Schema](https://json-schema.org/) — Schema concept (for JSON)
- [CommonMark](https://commonmark.org/) — Base markdown spec
- [RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174) — Keywords

## Authors

This specification was developed through collaborative sessions between a human author and AI agents.
