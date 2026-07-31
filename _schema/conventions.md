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

Inline, at the claim. These are the mechanism that flags
risk — the frontmatter percentages say *how much*, the
markers say *which*.

- Plain text = extracted directly from a source
- `^[inferred: reason]` = synthesised from held sources
- `^[generated: reason]` = a claim constructed here that
  **no source states**
- `^[imported: reason]` = brought from outside the held
  corpus — model knowledge, web search, general background
- `^[ambiguous: source A says X, source B says Y]` =
  sources disagree

### Choosing between inferred and generated

The test is whether an author of a held source would
recognise the claim as theirs.

- **Inferred** — a fair gloss, restatement, or comparison
  that follows from what sources say. *"VDEP and xT differ
  on whether risk is modelled"* is inferred: both papers
  would agree.
- **Generated** — a novel claim, reconciliation, or
  mechanism that exists nowhere but here. *"Encode
  structure the representation cannot recover and the data
  cannot support learning"* is generated: it was invented
  to reconcile three sources that never addressed each
  other, and none of them states it.

The boundary case is a **comparison that produces a new
conclusion**. Noting that two sources disagree is inferred.
Diagnosing *why* they disagree, or resolving it, is
generated.

### Why imported is separate

Imported claims look like knowledge and have no source in
`raw/` to check against. They are the hardest error to
catch, because nothing in the vault contradicts them.

Worked example: an entity page once stated an author was
"associated with Liverpool FC's research department".
That came from background knowledge, not from any held
source, and was wrong — the primary source, acquired
later, says Hudl. No amount of internal review would have
caught it, because the vault contained nothing to check
it against.

**Rule: never state an imported claim as fact.** Mark it,
or omit it.

## Frontmatter Provenance

Page-level proportions, summing to 100:

```yaml
provenance:
  extracted: 60%
  inferred: 25%
  generated: 10%
  imported: 0%
  ambiguous: 5%
```

`generated` and `imported` default to 0 if absent, so
pages written before this convention remain valid.

**A percentage is not sufficient on its own.** Risk is not
proportional to volume — one wrong generated claim that
propagates across pages does more damage than thirty
percent harmless glossing. Where `generated` is non-zero,
the claims should also carry inline markers, or be named
in the body.

## Supersession

When new info contradicts old, don't silently overwrite.
Record it:

> **Superseded**: This section previously stated X based
> on [[source-a]]. As of [[source-b]] (YYYY-MM), the
> current understanding is Y.

Generated and imported claims are **more likely** to need
supersession than extracted ones, and are the first place
to look when a contradiction surfaces.

## Lifecycle States

draft → reviewed → verified → stale → archived

- **draft**: Created from a single source. Low confidence. 
- **reviewed**: Human has read and confirmed it's reasonable.
- **verified**: Multiple sources confirm. High confidence. 
- **stale**: Not updated in 90+ days or superseded. 
- **archived**: Explicitly outdated. Kept for history.

Note that **verified is unavailable to a generated claim**
by definition — no source can confirm it, because no
source states it. A page whose central claim is generated
should not reach `verified` on the strength of its
extracted material. Where a generated claim can be
tested, a `question` page is the appropriate home for it.
