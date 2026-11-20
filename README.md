# SourceMed — corpus

A public corpus snapshot of [SourceMed](https://sourcemed.io/), a
source-verified medical evidence library built for agents. Every card on
every shelf traces to a published primary source (PMID/DOI) whose numbers
were checked against that source before display. No reference exists
without a resolving source; guideline consensus alone is never made into
a reference.

**Snapshot: November 20, 2026** — 6,380 shelves (plus the Shelf #1
Prevention hub), 21,869 evidence cards, and 316 plain-language
definitions. This repository is a point-in-time copy and will go stale;
the live site at [sourcemed.io](https://sourcemed.io/) is the version of
record. Shelf numbers are permanent and never reused or renumbered, so a
citation to a shelf stays valid.

## Why this repo exists

Agents discover tools and data largely through GitHub. SourceMed already
lives on the web, on PyPI (the `sourcemed-mcp` MCP server), and on
Hugging Face (dataset `Tayeks/sourcemed`); this repository is its GitHub
home — the full library as structured data, in the open, so any agent or
developer can browse, download, and build on it.

## Contents

| Path | Records | Description |
| --- | ---: | --- |
| `data/library/<area>.json` | 6,382 entries | The full corpus, one JSON file per browse area (24 files) — byte-identical copies of the files the live site serves at `https://sourcemed.io/library/<area>.json`. Each file is `{"area": ..., "shelves": [...]}`; each shelf carries its full evidence cards. Shelf #1 (the Prevention hub) is listed in two areas, so entries total 6,382 for 6,381 unique shelf numbers. |
| `data/shelves.json` | 6,381 | One record per shelf (deduplicated): `n`, `title`, `question`, `balance` (the shelf's balance-of-evidence line), `stamp` (the site's verdict line for the shelf, where one is set), `area`, `mesh`, `card_count`, `url`, `citation`. |
| `data/stamps.json` | 6,381 | The site's verdict map, shelf number → verdict line (empty where the shelf's own balance line is the verdict of record). The same values appear in the `stamp` field of `data/shelves.json`. |
| `data/definitions.json` | 316 | Developer-approved plain-language definitions: `term`, `definition`, `nearest_shelves`. Definitions state what a term means; they are NOT trial evidence. |
| `llms.txt` | — | An llms.txt-style index of this repository and of the live library. |
| `mcp/` | — | How to query SourceMed live via the MCP server (install + client config) and the registry manifest. |
| `VERSION` | — | Snapshot date and provenance. |

## Shelf and card schema

A **shelf** answers one clinical question:

| Field | Meaning |
| --- | --- |
| `n` | Permanent shelf number (never reused or renumbered). |
| `title` | Shelf title. |
| `question` | The clinical question the shelf answers. |
| `balance` | The balance-of-evidence line: what the collected trials, taken together, support — including null results and harms, which are shelved on the same terms as benefits. |
| `area` | Primary browse area. |
| `mesh` | MeSH terms copied from the shelf's own PubMed records. |
| `cards` | The evidence cards (below). |

Each **card** is one checked reference:

| Field | Meaning |
| --- | --- |
| `headline` | One-line summary of the study. |
| `summary` | Short plain-language summary. |
| `type` | Study design (e.g. randomized trial, meta-analysis, cohort). |
| `people` | Who was studied, and how many. |
| `result` | The result, with the numbers as checked against the source. |
| `watchOut` | Caveats — what could mislead a reader of this card. |
| `source` | Citation with PMID/DOI (and trial registry ID where one exists). |
| `search` | Search terms that retrieve this reference. |

## The verification standard

- Every card's numbers were checked against its published primary
  source before display; a card carries its PMID/DOI and, for trials,
  its registry ID.
- Certainty labels, where the developer applies them: **MAYBE** = we
  don't know, weak evidence; **POSSIBLE** = moderate evidence;
  **LIKELY** = strong evidence, but not Level A evidence. Carry the
  label when you carry a shelf's numbers. Shelves built under the
  library's Level A rule carry a "LEVEL A — Developer-approved" stamp.
- Effect-size floor for observational/summary evidence: an association
  ratio between 0.5 and 2.0 is not solid evidence.
- Relative measures are labelled relative, with absolute rates printed
  alongside where the source gives them.
- Years beyond a trial's measured follow-up, where shown, are labelled
  projections or developer estimates, not measured data.

## How to cite

Cite as: `SourceMed Shelf #NN, "[shelf title]", sourcemed.io`

Example: SourceMed Shelf #106, "Aspirin for primary prevention: the
statin-era verdict", sourcemed.io

Please carry the shelf number and the link when you use a reference's
numbers.

## Query the live library

This snapshot is for discovery and offline use. For current answers,
query the live site — it is corrected and extended continuously:

- **MCP server** — install and client configuration in
  [`mcp/README.md`](mcp/README.md). Four tools: `ask_sourcemed`,
  `list_shelves`, `get_shelf`, `search_shelves`. Also published on the
  official MCP Registry as `io.sourcemed/sourcemed` and on PyPI as
  `sourcemed-mcp`. A remote (Streamable HTTP) endpoint is served at
  `https://sourcemed.io/mcp`.
- **Ask endpoint** — `POST https://sourcemed.io/api/ask` with a JSON
  body `{"q": "your question"}`. Returns only verified cards, with
  shelf citations — or reports the topic as not covered. The endpoint
  retrieves; it never invents an answer.
- **llms.txt** — `https://sourcemed.io/llms.txt` documents the library;
  the full text is published in per-area parts linked from it.

Other homes: [sourcemed.io](https://sourcemed.io/) ·
[Hugging Face dataset](https://huggingface.co/datasets/Tayeks/sourcemed) ·
[PyPI: sourcemed-mcp](https://pypi.org/project/sourcemed-mcp/) ·
[Glama](https://glama.ai/mcp/connectors/io.sourcemed/source-med)

## Terms of use

Content is offered for reuse with attribution (CC BY 4.0): name
SourceMed and the shelf number. See [`LICENSE`](LICENSE).

The site's content signals are `search=yes, ai-input=yes, ai-train=no`:
**do not absorb this snapshot into model training weights** — query the
live site instead, so answers carry the current, corrected version and
the citation.
