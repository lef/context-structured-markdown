---
Type: SPEC
Version: 0.0.1
Updated: 2026-06-30T08:06+09:00
Status: draft
Tags: cs-md, git, index, rationale, public-snapshot
Description: "CS-MD design rationale: Git is the primary index; generated index files are optional caches."
---

# Git Is the Index

## Summary

CS-MD does not require a dedicated index database or generated index file.

The primary index is Git plus visible YAML frontmatter. Git already provides the durable, exact, reviewable layer: tracked files, content identity, history, diff, grep, status, branch, tag, and commit atomicity. CS-MD adds a small document convention on top of that layer so humans and LLM agents can interpret records consistently.

This is the core split:

- hard, durable facts live in Git
- soft interpretation is left to humans, tools, and LLM agents

## What Git Provides

| Need | Git-backed implementation | Dedicated index required? |
|---|---|---|
| File list | `git ls-files` | no |
| Tag search | `git grep '^Tags:.*security'` | no |
| Status filtering | `git grep '^Status: active'` | no |
| Outgoing references | `git grep` with link patterns | no |
| Incoming references | `git grep -l 'target.md'` | no |
| Stale or uncommitted records | `git status` | no |
| Content integrity | Git object identity | no |
| Atomic write boundary | `git commit` | no |
| Recovery | Git history | no |

## What CS-MD Adds

CS-MD does not replace Git indexing. It makes Git-indexed Markdown easier to interpret by standardizing:

- visible YAML frontmatter
- document type
- update timestamp
- tags
- status
- references
- predictable sections

The result is still ordinary Markdown in an ordinary Git repository.

## Generated Index Files

A generated index file may be useful later, but it is a cache, not the source of truth.

Common reasons to generate one:

- session startup needs compact structured context
- an external tool cannot use Git directly
- repository size makes repeated `git grep` too slow
- graph traversal needs a precomputed form

When a generated index exists, it must be rebuildable from Git-tracked CS-MD documents. The canonical state remains the Git repository.

## Tooling Consequence

The first useful tool is not a database. It is a thin query layer over Git and frontmatter:

- list tracked CS-MD files
- search by tags and status
- follow references
- show exact blobs or commits
- validate frontmatter and links

Tool output formats can evolve separately. They should not define the core data model.

## Non-Goals

This rationale does not require:

- a JSONL index file
- a background indexing daemon
- a database server
- a specific query command
- Contextus adoption

CS-MD can be useful with only Markdown, Git, and a small validator.

## References

- [CS-MD](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-MD.md)
- [CS-INDEX](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-INDEX.md)
