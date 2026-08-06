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

**Check the target exists before linking.** Dead links are
usually plausible concept names produced mid-sentence and
bracketed without lookup — including names invented on the
spot, not only tag names that happen to exist. Run
`find_mentioned_but_missing` after any batch of page
creation; it catches roughly 2.5× what re-reading catches.

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

Any marker may carry a `rests-on:` clause recording what
the claim stands on — see [Claim Dependencies](#claim-dependencies).

### Choosing between inferred and generated

The test is whether an author of a held source would
recognise the claim as theirs.

- **Inferred** — a fair gloss, restatement, or comparison
  that follows from what sources say.
- **Generated** — a novel claim, reconciliation, or
  mechanism that exists nowhere but here.

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
later, says Hudl.

**Rule: never state an imported claim as fact.** Mark it,
or omit it.

## Claim Dependencies

A `^[generated]` marker records *that* a claim was invented
here. It does not record what it was invented **on top of**
— so when a premise is revised, nothing finds what else
moves.

### Claim IDs

A generated claim referenced from more than one page gets a
short kebab-case ID, declared once at its home page:

> **`counterfactual-individuates`** — the individuating
> ingredient is the counterfactual, not the data.

The ID is the claim's handle. Backtracking is then a text
search for it. Claims used on one page only do not need one.

### rests-on

Dependent claims record what they stand on, appended to the
marker:

```
^[generated: the fourth cause of offensive bias.
 rests-on: source:vdep-f1-zero]
```

Four dependency kinds, and they fail differently:

| Kind | Means | Fails when |
|---|---|---|
| `source:` | A held source states it | The source is misread, or superseded by a better one |
| `claim:` | Another generated claim | That claim is revised — **cascades** |
| `imported:` | Outside the corpus | Any time. Nothing here can check it |
| `absence:` | **No source does X** | **A source is acquired** |

### The two searches, and why they differ

`rests-on:` records dependencies **one way**, so finding
what a change affects needs the *opposite* search from
finding where a claim appears:

| To find | Search for | Answers |
|---|---|---|
| Where a claim is **used** | the claim ID | Which pages state it |
| What **depends on** a claim | `rests-on: claim:<id>` | What breaks if it is revised |
| All claims that **cascade** | `rests-on: claim:` | The blast radius, vault-wide |
| All claims with an **expiry date** | `absence:` | What a new source could overturn |

Searching the ID alone is the common mistake. It finds the
references and misses the dependents, which are exactly the
pages a revision needs to reach.

### Absence is the dangerous kind

Most of this vault's corrections have come from acquiring a
source, not from re-reading a held one. A claim resting on
absence is not merely unsupported — it has a **built-in
expiry date**, and the expiry is triggered by the ordinary
business of ingesting papers.

Examples that expired exactly this way:

- *"Individual defensive credit is unaddressed anywhere"* —
  rested on `absence:`. Propagated to three pages before a
  search found Umemoto & Fujii (2023).
- *"Spearman's OBSO factorises under an independence
  assumption"* — rested on a secondary description.
  Corrected on acquiring the primary source.

**Rule: a claim marked `absence:` must be re-checked
whenever a source is ingested in its area.** This belongs
in the ingest checklist, not only here.

Note that narrowing beats deleting. *"No source runs a
sensitivity analysis"* was falsified by one ingest, but
narrowed to *"no source sweeps a horizon or weighting
parameter"* it has survived three more. **A narrowed
absence claim locates the boundary; a deleted one loses
the finding.**

### Worked example

The claim *"offensive bias has four causes"* lives on
[[action-valuation]]. Its fourth cause — too few positives
to train a classifier — rests on the F1 = 0.000 finding on
[[vaep]].

That finding was later put in doubt: F1 is near-guaranteed
for a calibrated model at a 0.23% base rate, and VAEP never
thresholds. The fourth cause therefore weakened — but this
was noticed **by accident**, months after both claims were
written, because nothing connected them.

Under this convention the fourth cause carries
`rests-on: source:vaep-f1-zero`, and revising that finding
surfaces the dependent by search.

### What this does not solve

Finding dependents relies on the ID being used
consistently. There is no automatic cascade and no
integrity check.

Note also that the lint helper `_link_target` currently
strips heading anchors, so if claim IDs are ever expressed
as `[[page#claim]]` they will be invisible to
`find_backlinks` and `find_mentioned_but_missing`. Plain
text IDs avoid this.

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

`generated` and `imported` default to 0 if absent, so pages
written before this convention remain valid.

**A percentage is not sufficient on its own.** Risk is not
proportional to volume — one wrong generated claim that
propagates across pages does more damage than thirty
percent harmless glossing.

## Supersession

When new info contradicts old, don't silently overwrite.
Record it:

> **Superseded**: This section previously stated X based
> on [[source-a]]. As of [[source-b]] (YYYY-MM), the
> current understanding is Y.

Generated and imported claims are **more likely** to need
supersession than extracted ones, and are the first place
to look when a contradiction surfaces.

### Before closing a supersession, follow the dependencies

A superseded claim may be load-bearing elsewhere. Two
searches, not one:

1. **Search the claim ID** — finds pages that *state* it.
   Update each.
2. **Search `rests-on: claim:<id>`** — finds claims that
   *depend* on it. These are the ones a revision silently
   breaks, and they will not appear in the first search.

Then decide whether each dependent survives. A dependent
may weaken rather than fall: the fourth cause of offensive
bias did not disappear when its premise was questioned, it
became **the least secure of four** and is now marked so.

**Record the weakening rather than deleting the claim.**
The same reasoning as narrowing an absence claim — a
qualified claim keeps the finding and its caveat together;
a deleted one loses both.

## Lifecycle States

draft → reviewed → verified → stale → archived

- **draft**: Created from a single source. Low confidence. 
- **reviewed**: Human has read and confirmed it's reasonable.
- **verified**: Multiple sources confirm. High confidence. 
- **stale**: Not updated in 90+ days or superseded. 
- **archived**: Explicitly outdated. Kept for history.

Note that **verified is unavailable to a generated claim**
by definition — no source can confirm what no source
states. A page whose central claim is generated should not
reach `verified` on the strength of its extracted material.
Where a generated claim can be tested, a `question` page is
the appropriate home for it.

**Archived pages are intentionally orphaned.** They are
superseded duplicates kept for history, so they are
expected to have no inbound links, and `find_orphan_pages`
skips them. They remain link *sources* — their outbound
links still count — so a page whose only referrer is
archived is a genuine orphan.
