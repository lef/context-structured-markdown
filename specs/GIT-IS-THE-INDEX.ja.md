---
Type: SPEC
Version: 0.0.1
Updated: 2026-06-30T08:06+09:00
Status: draft
Tags: cs-md, git, index, rationale, public-snapshot
Description: "CS-MD の設計根拠: Git が primary index であり、生成 index file は optional cache である。"
---

# Git Is the Index

## 概要

CS-MD は専用の index database や生成済み index file を必須にしない。

primary index は Git と、見える YAML frontmatter の組み合わせである。Git は、tracked files、content identity、history、diff、grep、status、branch、tag、commit atomicity という、堅く、正確で、review 可能な層をすでに提供している。CS-MD はその上に小さな document convention を足し、人間と LLM agent が record を一貫して解釈しやすくする。

中心にある分離はこれである。

- 堅く durable な事実は Git に置く
- 柔らかい解釈は人間、tool、LLM agent に任せる

## Git が提供するもの

| 必要な機能 | Git-backed implementation | 専用 index は必要か |
|---|---|---|
| file list | `git ls-files` | no |
| tag search | `git grep '^Tags:.*security'` | no |
| status filtering | `git grep '^Status: active'` | no |
| outgoing references | link pattern による `git grep` | no |
| incoming references | `git grep -l 'target.md'` | no |
| stale / uncommitted records | `git status` | no |
| content integrity | Git object identity | no |
| atomic write boundary | `git commit` | no |
| recovery | Git history | no |

## CS-MD が足すもの

CS-MD は Git indexing を置き換えない。Git-indexed Markdown を解釈しやすくするため、次のものを標準化する。

- 見える YAML frontmatter
- document type
- update timestamp
- tags
- status
- references
- predictable sections

結果として残るのは、通常の Git repository に置かれた通常の Markdown である。

## 生成 index file

生成 index file が役に立つ場面はある。ただし、それは source of truth ではなく cache である。

生成する理由の例:

- session startup で compact な structured context が必要
- external tool が Git を直接使えない
- repository が大きくなり、毎回の `git grep` が遅い
- graph traversal に precomputed form が必要

生成 index が存在する場合でも、それは Git-tracked CS-MD documents から再生成できなければならない。canonical state は Git repository に残る。

## tooling への帰結

最初に必要な tool は database ではない。Git と frontmatter の上に置く薄い query layer である。

- tracked CS-MD files を列挙する
- tags / status で検索する
- references を辿る
- exact blob / commit を表示する
- frontmatter と links を validate する

tool output format は別に進化してよい。それは core data model を定義しない。

## Non-Goals

この rationale は以下を要求しない。

- JSONL index file
- background indexing daemon
- database server
- specific query command
- Contextus adoption

CS-MD は Markdown、Git、小さな validator だけでも役に立つ。

## References

- [CS-MD](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-MD.md)
- [CS-INDEX](https://github.com/lef/context-structured-markdown/blob/main/specs/CS-INDEX.md)
