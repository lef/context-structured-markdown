---
Type: README
Updated: 2026-06-29T02:05+09:00
Status: active
Tags: cs-md, markdown, contextus, public-snapshot
Description: Public snapshot of Context Structured Markdown specifications
---

# Context Structured Markdown (CS-MD)

Context Structured Markdown (CS-MD) is a small convention for writing git-backed Markdown records that both humans and LLM agents can read.

The core idea is simple: keep the hard, durable parts in Git; let LLMs work with the softer interpretive layer. A CS-MD document is still normal Markdown. It adds visible YAML frontmatter, predictable sections, and references so documents can be searched, reviewed, linked, and reconstructed from Git history.

CS-MD was developed inside Contextus, but it can be used independently of the full Contextus workflow, agent orchestration, or enforcement model.

## 日本語概要

CS-MD は、人間と LLM Agent の両方が読める git-backed Markdown record のための小さな規約です。

中心にある考えは、「堅い部分を Git に、柔らかい部分を LLM に」置くことです。CS-MD 文書は通常の Markdown のままです。その上で、見える YAML frontmatter、予測可能な section、参照規約を加え、検索・レビュー・リンク・Git history からの再構成をしやすくします。

CS-MD は Contextus の内部で生まれましたが、Contextus 全体の workflow、agent orchestration、enforcement model を採用しなくても使えます。

## Current Status

This repository is an early public snapshot for CS-MD 0.0.1.

The specifications are drafts. The format is already dogfooded in Contextus-based projects, but names, validation rules, and tooling contracts may still change before a stable 0.1.0 release.

## Specifications

- [CS-MD](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-MD.md) — document format: how to write structured Markdown records
- [CS-SCHEMA](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-SCHEMA.md) — validation model: what makes a CS-MD document valid
- [GIT-IS-THE-INDEX](https://github.com/lef/context-structured-markdown/blob/main/specs/GIT-IS-THE-INDEX.md) / [日本語](https://github.com/lef/context-structured-markdown/blob/main/specs/GIT-IS-THE-INDEX.ja.md) — design rationale: Git is the primary index
- [CS-INDEX](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-INDEX.md) — optional derived cache format for lookup and graph traversal

The [canonical vocabulary registry](https://github.com/lef/contextus/blob/main/registry.jsonl) is owned by Contextus.

`registry.jsonl` is intentionally not copied into this repository for 0.0.1. CS-MD defines the document convention; Contextus owns the current registry data and governance.

## Minimal Example

```markdown
---
Type: KNOWLEDGE
Updated: 2026-06-28T00:00+09:00
Tags: cs-md, git, llm
Status: active
---

# Git-backed context records

## Finding

Durable context should be reviewable as Git history, not trapped in chat logs.

## References

- [CS-MD](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-MD.md)
```

## Non-Goals

CS-MD is not:

- a replacement for Markdown
- a chat transcript format
- a protocol for agent orchestration
- a database
- a requirement to generate an index file
- a requirement to adopt Contextus

## License

MIT License. See [LICENSE](https://github.com/lef/context-structured-markdown/blob/main/LICENSE).
