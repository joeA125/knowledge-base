# Knowledge Base

This repository is an Obsidian knowledge vault for AI, ML, data science, and adjacent topics. It is designed to preserve raw source material, maintain structured wiki pages, and compound useful work over time instead of leaving it in chat history.

The operating model for the vault is defined in `CLAUDE.md`: raw inputs stay immutable, wiki pages are maintained and cross-linked, tags are governed centrally, and substantive work is logged.

## Core Rules

1. `raw/` is source storage and must not be edited.
2. `wiki/` is the maintained knowledge layer and should be updated, cross-linked, and kept readable.
3. Tags should come from `_schema/tags.md`.
4. Substantive maintenance work should be recorded in `log.md`.

## Repository Layout

```text
.
|- README.md
|- CLAUDE.md
|- index.md
|- log.md
|- _schema/
|  |- conventions.md
|  |- tags.md
|  |- templates/
|     |- concept.md
|     |- entity.md
|     |- summary.md
|     \- synth.md
|- raw/
|  |- articles/
|  |- notes/
|  |- other/
|  |- papers/
|  \- transcripts/
|- wiki/
|  |- overview.md
|  |- concepts/
|  |- conversations/
|  |- dashboards/
|  |- entities/
|  |- summaries/
|  \- syntheses/
\- _quizzes/
```

## What Goes Where

- `raw/`: immutable source material such as papers, notes, transcripts, and copied articles.
- `wiki/summaries/`: source summaries extracted from raw inputs.
- `wiki/entities/`: pages for people, organizations, tools, models, and other named things.
- `wiki/concepts/`: pages for concepts, methods, and principles.
- `wiki/syntheses/`: cross-source analysis, comparisons, and higher-order takeaways.
- `wiki/conversations/`: durable write-backs from useful question-answering sessions.
- `wiki/dashboards/`: Dataview-driven operational views of vault health and topic coverage.
- `_schema/`: conventions, approved tags, and page templates.
- `index.md`: the catalog of maintained wiki pages.
- `log.md`: chronological record of maintenance operations.

## Working Model

Typical workflows in this vault follow the rules in `CLAUDE.md`:

- `INGEST`: read a source in `raw/`, create or update a summary, update affected concept and entity pages, refresh links, then update `index.md` and `log.md`.
- `QUERY`: answer questions from the maintained wiki first, not directly from raw source text when avoidable; file durable results into `wiki/conversations/` or another appropriate wiki location when the result adds value.
- `LINT`: check for orphans, dead links, stale pages, low-confidence pages, missing concepts, and tag drift.
- `WRITEBACK`: treat useful analysis as durable knowledge, not transient chat output.

## Page Standards

Wiki pages should:

- use Obsidian-compatible markdown and `[[wikilinks]]`
- follow lowercase-kebab-case filenames
- include the required frontmatter described in `CLAUDE.md`
- use tags defined in `_schema/tags.md`
- maintain a confidence score and lifecycle state when applicable

For exact formatting and naming rules, see `_schema/conventions.md` and the templates in `_schema/templates/`.

## Suggested Maintenance Loop

1. Start from `index.md`, `wiki/overview.md`, or a relevant dashboard.
2. Read or ingest from `raw/` without modifying source files.
3. Update the right wiki pages and cross-links.
4. Record the operation in `log.md`.
5. Keep the vault easier to navigate than it was before the change.

## Purpose

The goal of this repository is not just to store information. It is to turn scattered source material into a maintained, navigable, and compounding knowledge system.
