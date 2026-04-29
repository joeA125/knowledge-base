# Conventions

## Naming

- Filenames: `lowercase-kebab-case.md`
- One concept per page
- Split pages that grow beyond ~1000 words covering
  multiple distinct ideas

## Linking

- Use `[[wikilinks]]` for all cross-references
- Every wiki page should link:
  - Downward to its sources
  - Sideways to related wiki pages
- Prefer `[[page-name|display text]]` when the link
  text should differ from the filename

## Provenance Markers

- Plain text = extracted directly from a source
- `^[inferred: reason]` = synthesised by the LLM
- `^[ambiguous: source A says X, source B says Y]` =
  sources disagree

## Supersession

When new info contradicts old, don't silently overwrite.
Record it:

> **Superseded**: This section previously stated X based
> on [[source-a]]. As of [[source-b]] (YYYY-MM), the
> current understanding is Y.

## Lifecycle States

draft → reviewed → verified → stale → archived

- **draft**: Created from a single source. Low confidence. 
- **reviewed**: Human has read and confirmed it's reasonable.
- **verified**: Multiple sources confirm. High confidence. 
- **stale**: Not updated in 90+ days or superseded. 
- **archived**: Explicitly outdated. Kept for history.