---
Type: SPEC
Version: 0.0.1
Updated: 2026-06-29T02:05+09:00
Status: draft
Provenance: L0/contextus
Tags: index, spec, cs-md, contextus
---

# CS-INDEX: Contextus Structured Markdown Index


## Abstract

CS-INDEX defines an optional JSONL-based cache format for CS-MD documents.
Git remains the primary index. This generated cache provides machine-readable
lookup, bidirectional reference tracking, and implicit graph structure when a
tool needs a precomputed representation.

CS-INDEX は CS-MD 文書の optional JSONL cache format を定義する。
primary index は Git である。この generated cache は、tool が precomputed
representation を必要とするときに、検索、双方向参照追跡、暗黙的グラフ構造を提供する。

## Status of This Document

This is a draft specification, dependent on CS-MD (specs/CS-MD.md).

## 1. Introduction

### 1.1 Relationship to CS-MD

CS-INDEX depends on CS-MD. CS-MD does NOT depend on CS-INDEX.

```
CS-MD (independent) ← CS-INDEX (depends on CS-MD)
```

CS-MD documents are the source of truth. CS-INDEX is a **derived cache**
— it can be regenerated at any time from the source .md files.

The design rationale is specified in
[GIT-IS-THE-INDEX](https://github.com/lef/context-structured-markdown/blob/main/specs/GIT-IS-THE-INDEX.md).

### 1.2 Two Indexes: Machine and Human

A project has **two** indexes for its CS-MD documents:

| Index | Format | Generated | Audience | Role |
|---|---|---|---|---|
| CP-INDEX.jsonl | JSONL | auto-generated from CS-MD headers | machine | Search, graph, validation |
| KNOWLEDGE.md | CS-MD | human-curated | human + LLM | Quick capture inbox, importance-ranked entry list |

CP-INDEX.jsonl is the machine index — complete, mechanical, regenerable.
KNOWLEDGE.md is the human index — curated, editorial, importance-ranked.

Both are derived from the individual CS-MD files in `knowledge/`,
but they serve different purposes and different audiences.

KNOWLEDGE.md is itself a CS-MD document (`Type: KNOWLEDGE` in frontmatter).
It contains:
- **Quick Notes** section: rapid capture of small findings (inbox)
- **Entries** section: curated links to individual `knowledge/*.md` files (index)

When a Quick Note matures, it is promoted to an individual CS-MD file.
When it becomes stale, it is archived or deleted.

### 1.3 Design Principles

- **Cache, not source of truth.** CP-INDEX.jsonl is derived from .md files.
  If it breaks, regenerate it (`make index`).
- **grep-friendly.** JSONL (one JSON object per line) is greppable.
- **No database.** For project-scoped knowledge (< 10,000 entries),
  flat files + grep are sufficient. SQLite is the escalation path.
- **Postel's Law.** Unknown fields in index entries MUST be ignored.

### 1.4 Scan Scope

Default scan target: all `*.md` files in the repository (`git ls-files -- '*.md'`).
Overridable via `CSMD_SCAN_PATTERNS` environment variable.

All CS-MD files (knowledge, specs, rules, tasks, designs, procedures, etc.)
are equally searchable. Directory names do not restrict search scope.

## 2. File Format

### 2.1 Location

```
$FLOW_DIR/CP-INDEX.jsonl
```

The file MAY be git-tracked or gitignored (it is a regenerable cache).

### 2.2 Format

One JSON object per line (JSONL). Each line represents one CS-MD document
**or one section within a multi-entry document** (see §2.4).

```jsonl
{"id":"fetch-mcp-e2e","file":"knowledge/fetch-mcp-e2e.md","title":"fetch MCP e2e","tags":["finding","fetch-mcp"],"provenance":"example-repo@main:abc1234","context":"MINUTES:2026-03-21#fetch","refs_out":["CONSTITUTION#security"],"refs_in":["audit-2026-03"],"updated":"2026-03-21","status":"active"}
```

### 2.3 Fields

#### MUST Fields

| Field | Type | Description |
|---|---|---|
| id | string | Unique identifier (descriptive slug) |
| file | string | Relative path to the .md file, optionally with `#heading` anchor |
| title | string | H1 title of the document, or H2 title of the section |
| updated | string | ISO 8601 date from CS-MD header |

#### SHOULD Fields

| Field | Type | Description |
|---|---|---|
| tags | string[] | From CS-MD `Tags:` frontmatter field |
| status | string | From CS-MD `Status:` frontmatter field |
| refs_out | string[] | Documents this entry references (outgoing edges) |
| refs_in | string[] | Documents referencing this entry (incoming edges, derived) |

#### MAY Fields

| Field | Type | Description |
|---|---|---|
| provenance | string | From CS-MD `Provenance:` frontmatter field |
| context | string | From CS-MD `Context:` frontmatter field (link to MINUTES) |
| description | string | From CS-MD `Description:` frontmatter field |
| class | string | KNOWLEDGE class: `decision`, `finding`, or `lesson` |

### 2.4 Sub-Document Indexing (#heading References)

A single CS-MD file MAY contain multiple entries as H2 sections
(e.g., weekly findings grouped in one file). CP-INDEX.jsonl can reference
individual sections using `#heading` anchors in the `file` field.

```jsonl
{"id":"bwrap-path","file":"knowledge/2026-03-w3.md#bwrap-path","title":"bwrap PATH に /sbin 必要","tags":["finding","bwrap"],...}
{"id":"socat-timeout","file":"knowledge/2026-03-w3.md#socat-timeout","title":"socat -t120 タイムアウト","tags":["finding","socat"],...}
```

**Rules for sub-document indexing:**

- `file` field includes `#heading-slug` (GitHub-style heading anchor)
- `title` is the H2 heading text, not the H1
- `tags` MAY be document-level (from `> Tags:` header) or entry-specific
- `id` is unique per entry, not per file
- Standard markdown `[text](file.md#heading)` links can point to individual entries
- `#heading` anchors work in GitHub, Obsidian, and most markdown renderers

**When to use sub-document indexing:**

- Small findings that don't warrant individual files (2-3 lines each)
- Grouped by time (weekly), context (session), or topic
- The document itself is CS-MD compliant (H1 + header block)
- INDEX provides per-entry searchability despite grouping

## 3. Generation

### 3.1 Source Extraction

CP-INDEX.jsonl is generated by reading CS-MD headers and body references:

```bash
for f in $(git ls-files -- '*.md'); do
    [ -f "$f" ] || continue
    # Extract YAML frontmatter fields
    # Extract [text](path) references → refs_out
    # Output JSONL line
done
# Derive refs_in by inverting all refs_out
```

### 3.2 Regeneration

```bash
make index    # regenerate CP-INDEX.jsonl from .md files
```

If CP-INDEX.jsonl is missing or corrupt, regeneration recovers all data.
No information is lost because the .md files are the source of truth.

## 4. Graph Structure

### 4.1 Implicit Graph

The combination of `refs_out` and `refs_in` defines a directed graph:

- **Nodes**: CS-MD documents (each JSONL entry)
- **Outgoing edges**: `refs_out` (what this document references)
- **Incoming edges**: `refs_in` (what references this document)

No explicit graph database is needed. The graph is implicit in the index.

### 4.2 Bidirectional Lookup

```bash
# What does "gh-oauth-fallback" reference?
grep '"gh-oauth-fallback"' CP-INDEX.jsonl    # read refs_out

# What references "gh-oauth-fallback"?
grep '"gh-oauth-fallback"' CP-INDEX.jsonl    # read refs_in (or grep all refs_out)
```

### 4.3 Provenance Chains

Provenance forms a DAG (directed acyclic graph):

```
KNOWLEDGE: "gh OAuth fallback"
  ← MINUTES: 2026-03-21 (context)
  ← CONSTITUTION: security (refs_out)
  ← KNOWLEDGE: "auth setup" (refs_out)
```

## 5. KNOWLEDGE-Specific Features

### 5.1 Three-Class System

KNOWLEDGE entries are classified by the `class` field:

| Class | Stability | Archive Strategy |
|---|---|---|
| decision | High (ADR-like) | Archive directly |
| finding | Low (may stale) | Needs validity check |
| lesson | High (transferable) | L0/L2 promotion candidate |

### 5.2 MINUTES Linkage

The `context` field links a KNOWLEDGE entry to the MINUTES discussion
that produced it. This provides the "WHY" behind decisions without
bloating the KNOWLEDGE entry itself.

## 6. Security Considerations

- CP-INDEX.jsonl is a cache. Tampering with it does not affect source of truth.
- Regeneration from .md files is the recovery mechanism.
- Graph traversal should not follow external URLs automatically
  (prompt injection risk via fetched content).

## References

- [CS-MD](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-MD.md) — The document format this index is built on
- [JSONL](https://jsonlines.org/) — JSON Lines format
- [ADR](https://adr.github.io/) — Architecture Decision Records (KNOWLEDGE class inspiration)

## Authors

This specification was developed through collaborative sessions between a human author and AI agents.
