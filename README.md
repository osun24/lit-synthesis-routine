# Literature Synthesis Routine

A Claude Code routine that runs weekly, scans recent scientific literature across multiple sources and fields via the [Paperclip MCP](https://paperclip.gxl.ai), tracks specific authors, and surfaces non-obvious cross-domain connections as a structured digest.

## What it does

Each week, the routine:

1. Searches **5 sources** (PubMed Central, arXiv, bioRxiv, medRxiv, OpenAlex) using concept-based queries via Paperclip
2. Checks for new publications from a named author watchlist — no IDs required
3. Runs a parallel AI map pass over top candidates to extract method, phenomenon, and field metadata
4. Reads methods sections of high-interest papers for transferable technique detection
5. Scores papers on cross-domain reach, citation velocity, venue prestige, and recency
6. Identifies high-surprise pairs of papers from different fields with overlapping methods or structure
7. Generates a testable hypothesis and research question for each connection
8. Appends a structured section to `digest.md` and opens a pull request

## Output example

```markdown
## Week of 2026-05-04 — 2026-05-08

*Sources: pmc, arxiv, biorxiv, medrxiv | Queries: 10 | Papers scanned: 312 | Papers after filter: 187*

### Watched Author Updates

**Carl Friston** (Active inference, free energy principle)
- [Variational message passing for hierarchical models](https://doi.org/...) — one-sentence summary
  Source: PMC | Venue: PLOS Computational Biology | Fields: Neuroscience, Statistics

### Cross-Domain Connections

**1. Soft Matter ↔ Systems Neuroscience** | Score: 0.87 | Confidence: High

> If the topological defect dynamics described in active nematics (Paper A)
> were mapped onto cortical travelling waves in neural tissue (Paper B),
> one would predict that wave termination events correspond to ±½ defect
> annihilation, testable with voltage imaging.

- **Paper A:** [Defect dynamics in active nematics...](https://arxiv.org/abs/...)
- **Paper B:** [Travelling waves during slow-wave sleep...](https://doi.org/...)
- **Research question:** Do cortical wave termination sites show the spatial
  signatures of nematic defect annihilation under optical imaging?
- **Why non-obvious:** Different citation communities (soft matter vs. neurophysiology),
  different vocabulary (order parameter vs. field potential), different spatial scales —
  the connection requires recognising that both are described by the same tensor PDE.
```

## Setup

### 1. Fork or clone this repo

```bash
git clone https://github.com/osun24/lit-synthesis-routine
cd lit-synthesis-routine
```

### 2. Create your config

```bash
cp config.example.json config.json
```

Edit `config.json`:

- **`search_queries`** — concept-based search strings passed to Paperclip. Write these as natural language topics rather than database-specific category codes. Paperclip runs them in parallel across all configured sources and deduplicates results. Aim for 6–12 queries that cover your target fields from different angles.

- **`sources`** — Paperclip source names to search. Options: `biorxiv`, `medrxiv`, `pmc`, `arxiv`, `abstracts_only` (OpenAlex, 150M abstracts). Default covers all full-text sources.

- **`watch_authors`** — authors whose new papers always appear in the digest, regardless of score. Only `name` and `note` are required — Paperclip looks up authors by name. Use the full name as it appears on papers.

- **`lookback_days`** — how many days back to scan. `7` matches a weekly cadence; increase to `14` for bi-weekly.

- **`max_papers_per_search`** — total papers to process after merging all search queries. 60–100 is the practical range; above 150 the map pass becomes slow.

- **`ranking`** — adjust weights for scoring. `cross_domain_surprise_weight` should stay the largest; reducing `citation_velocity_weight` prevents bias toward hot topics in established fields. Weights must sum to 1.0.

- **`max_connections_in_digest`** — connection pairs per weekly section. 5–8 is readable.

### 3. Add Paperclip as a MCP connector

Paperclip is the MCP that gives the routine access to 11M+ full-text papers and 150M+ abstracts across five scientific sources. It must be connected before creating the routine.

**Add to Claude Code:**

```bash
claude mcp add --transport http paperclip https://paperclip.gxl.ai/mcp
```

Then authenticate: open Claude Code, run `/mcp`, and select **Authenticate** under the `paperclip` server. You'll need a free Paperclip API key from [paperclip.gxl.ai](https://paperclip.gxl.ai).

**Add to the routine:**

When creating the routine at `claude.ai/code/routines`, the Paperclip connector should appear in the **Connectors** tab (it syncs from your Claude Code config). Enable it for the routine.

Alternatively, add it directly from the routine form under **Connectors → Add connector**.

### 4. Create the Claude routine

**Option A — Web UI**

1. Go to [claude.ai/code/routines](https://claude.ai/code/routines) and click **New routine**
2. **Name:** `Weekly Literature Synthesis`
3. **Prompt:** Copy the entire contents of [`routine-prompt.md`](./routine-prompt.md) into the prompt field
4. **Repository:** Add this repo (or your fork)
5. **Connectors:** Enable **Paperclip**
6. **Trigger:** Select **Weekly** — Monday morning works well
7. **Branch push permission:** Leave default (`claude/`-prefixed branches only)
8. Click **Create**

**Option B — CLI**

```bash
# From inside a Claude Code session
/schedule weekly literature synthesis every Monday at 8am
```

When asked for the prompt, paste the contents of `routine-prompt.md`. When asked for the repository, provide this repo's URL. Add the Paperclip connector from the web UI after creation.

To set a custom cron expression (e.g. bi-weekly):

```bash
/schedule update
# Bi-weekly: 0 8 1-7,15-21 * 1
```

### 5. Trigger a test run

Click **Run now** on the routine's detail page. Check the session output to confirm Paperclip is accessible and the first digest section is appended and a PR opened.

## Routine mechanics

### How papers are scored

| Component | Weight | What it measures |
|---|---|---|
| Cross-domain reach | 40% | Distinct secondary fields from the map pass |
| Citation velocity | 25% | Citations per day since publication |
| Venue prestige | 15% | Source-inferred prestige (PMC peer-reviewed > preprints) |
| Recency | 20% | Exponential decay, half-life 180 days |

### How cross-domain pairs are found

The routine uses Paperclip's `map` command to extract per-paper metadata (method, phenomenon, primary field, secondary fields) across the full candidate set. It then identifies pairs where:

- Primary fields are from different scientific domains
- No shared citation lineage (different sources, no shared authors)
- Methods, phenomena, or mathematical structures overlap semantically

For the top 20 candidates, it also reads the **methods section** directly via `paperclip cat /papers/{id}/sections/Methods.lines`. This is what enables detection of connections that are only visible at the technique level — the layer where most genuine interdisciplinary transfer happens.

Each pair gets a **surprise bonus** (2×) the first time two domains appear together in this repo's digest history.

### Watched author papers

Add any researcher by name — no database ID required. Paperclip's `lookup author` command handles disambiguation. Watched-author papers:
- Bypass credibility filtering
- Always appear in the **Watched Author Updates** section
- Are still scored and eligible to appear in connection pairs

### What Paperclip adds over raw API calls

| Capability | Raw arXiv + Semantic Scholar | With Paperclip MCP |
|---|---|---|
| Sources | arXiv only | PMC, arXiv, bioRxiv, medRxiv, OpenAlex |
| Full text | No (abstracts only) | Yes — sections, figures, supplements |
| Author lookup | Requires numeric SS ID | By name |
| Parallel search | Manual multi-request | `searches` command, native |
| Methods reading | Not possible | `cat /papers/{id}/sections/Methods.lines` |
| Rate limits | Manual handling | Managed by Paperclip |

## Extending the config

### Adding authors to the watchlist

```json
{
  "name": "Ada Yonath",
  "note": "Ribosome crystallography, structural biology"
}
```

The `note` appears next to the author's name in the digest.

### Changing topic coverage

Replace `search_queries` with concepts relevant to your fields. Write as natural language — Paperclip uses hybrid BM25 + vector search, so topic descriptions outperform keyword lists:

```json
"search_queries": [
  "materials informatics crystal structure prediction",
  "synthetic biology genetic circuit design",
  "microfluidics organ on a chip",
  "topological materials quantum transport"
]
```

### Restricting to specific sources

To limit to peer-reviewed literature only (slower but higher average credibility):

```json
"sources": ["pmc"]
```

To include preprints for fastest coverage of emerging work:

```json
"sources": ["biorxiv", "medrxiv", "arxiv"]
```

## Repository structure

```
.
├── README.md              # this file
├── routine-prompt.md      # prompt to paste into the Claude routine
├── config.example.json    # template config — copy to config.json and edit
├── config.json            # your config (gitignored)
└── digest.md              # running weekly digest, appended by the routine
```

## Limitations

- **Paperclip API key required.** Free tier available at [paperclip.gxl.ai](https://paperclip.gxl.ai). Rate limits apply on the free tier; for high `max_papers_per_search` values, a paid key avoids throttling.
- **No persistent vector index.** Each run starts fresh. Cross-domain surprise detection reads the existing `digest.md` for prior pairings but has no semantic memory of past papers.
- **Author name disambiguation.** Paperclip looks up authors by name. Very common names (e.g. "John Smith") may match multiple researchers. Use the full name as it appears on papers, or add a distinctive co-author or institution in the `note` field as a hint.
- **PMC open-access only.** PMC full text is limited to open-access papers. Closed-access journals are not in the corpus — Paperclip falls back to abstract-only for those via OpenAlex.
- **Methods section parsing.** Section detection relies on standard headings. Some preprints use non-standard structure (e.g. no explicit "Methods" heading), in which case Paperclip returns the closest matching section.

## License

MIT
