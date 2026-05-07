# Literature Synthesis Routine

A Claude Code routine that runs weekly, scans recent scientific literature across multiple fields, tracks specific authors, and surfaces non-obvious cross-domain connections as a structured digest.

## What it does

Each week, the routine:

1. Fetches recent papers from configured arXiv categories via the arXiv API
2. Checks for new publications from a watchlist of specific authors via Semantic Scholar
3. Enriches top candidates with citation and venue data
4. Scores papers on cross-domain reach, citation velocity, venue prestige, and recency
5. Identifies high-surprise pairs of papers from different fields with no shared citations but overlapping concepts
6. Generates a hypothesis for each connection and a concrete research question
7. Appends a structured section to `digest.md` and opens a pull request

## Output example

```markdown
## Week of 2026-05-04 — 2026-05-08

### Watched Author Updates

**Carl Friston** (Active inference, free energy principle)
- [Variational message passing for hierarchical models](https://arxiv.org/abs/...) — one-sentence summary
  Venue: PLOS Computational Biology | Citations: 3 | Fields: Neuroscience, CS

### Cross-Domain Connections

**1. cond-mat.soft ↔ q-bio.NC** | Score: 0.87 | Confidence: High

> If the topological defect dynamics described in active nematics (Paper A)
> were mapped onto cortical travelling waves in neural tissue (Paper B),
> one would predict that wave termination events correspond to ±½ defect
> annihilation, testable with voltage imaging.

- **Paper A:** [Defect dynamics in active nematics...](https://arxiv.org/abs/...)
- **Paper B:** [Travelling waves during slow-wave sleep...](https://arxiv.org/abs/...)
- **Research question:** Do cortical wave termination sites show the spatial
  signatures of nematic defect annihilation under optical imaging?
- **Why non-obvious:** No paper in either literature cites the other;
  the connection requires mapping an order parameter from soft matter
  onto a field variable in systems neuroscience.
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

- **`arxiv_categories`** — arXiv category codes to monitor. Browse categories at [arxiv.org/help/api/user-manual](https://info.arxiv.org/help/api/user-manual.html#subject_classifications). Mix top-level categories (`cs.LG`) with cross-listed ones for broader coverage.

- **`watch_authors`** — authors whose new papers always appear in the digest, regardless of score. Find a Semantic Scholar author ID by searching [semanticscholar.org](https://www.semanticscholar.org), opening the author's profile, and copying the numeric ID from the URL (`/author/Name/1234567` → `"1234567"`).

- **`lookback_days`** — how many days back to scan. `7` matches a weekly cadence; increase to `14` if you run bi-weekly.

- **`ranking`** — adjust weights if you want to prioritise citation velocity over cross-domain surprise, or vice versa. Weights must sum to 1.0.

- **`max_connections_in_digest`** — number of connection pairs per weekly section. 5–8 is readable; more than 10 becomes noise.

### 3. Create the Claude routine

**Option A — Web UI**

1. Go to [claude.ai/code/routines](https://claude.ai/code/routines) and click **New routine**
2. **Name:** `Weekly Literature Synthesis`
3. **Prompt:** Copy the entire contents of [`routine-prompt.md`](./routine-prompt.md) into the prompt field
4. **Repository:** Add this repo (or your fork)
5. **Trigger:** Select **Weekly** — pick a day and time (Monday morning works well)
6. **Branch push permission:** Leave default (`claude/`-prefixed branches only)
7. Click **Create**

**Option B — CLI**

```bash
# From inside a Claude Code session
/schedule weekly literature synthesis every Monday at 8am
```

Claude will walk through the same fields interactively. When asked for the prompt, paste the contents of `routine-prompt.md`. When asked for the repository, provide this repo's URL.

To customise the cron expression (e.g. every other week):

```bash
/schedule update
# Claude will prompt for the new cron expression
# Bi-weekly example: 0 8 * * 1/2
```

### 4. Configure Semantic Scholar API access (recommended)

The routine works without an API key but is rate-limited to ~100 requests/5 minutes. For reliable weekly runs across large category lists, set a free API key as an environment variable in the routine's environment settings:

1. Request a key at [semanticscholar.org/product/api](https://www.semanticscholar.org/product/api)
2. In the routine's **Environment** tab, add: `SEMANTIC_SCHOLAR_API_KEY=your_key_here`

The routine's prompt includes the header `x-api-key: {SEMANTIC_SCHOLAR_API_KEY}` in all Semantic Scholar calls automatically.

### 5. Trigger a test run

After creating the routine, click **Run now** on the routine's detail page to verify the first run produces a valid PR before the first scheduled Monday.

## Routine mechanics

### How papers are scored

Each candidate paper receives a composite score:

| Component | Weight | What it measures |
|---|---|---|
| Cross-domain reach | 40% | How many distinct fields cite this paper |
| Citation velocity | 25% | Citations per day since publication |
| Venue prestige | 15% | SJR quartile of the publishing venue |
| Recency | 20% | Exponential decay, half-life 180 days |

### How cross-domain pairs are found

The routine looks for papers that are:
- From different arXiv top-level namespaces (`cs.*` vs `q-bio.*` vs `physics.*`, etc.)
- Citation-disjoint — no shared citing papers
- Semantically overlapping — Claude judges concept similarity directly from the abstracts

Each pair gets a **surprise bonus** (2×) the first time two top-level namespaces appear together in this repo's digest history, rewarding genuinely novel field combinations.

### Watched author papers

Watched author papers bypass the credibility filter and always appear in the digest if published within `lookback_days`. They are scored normally for the connections section — a watched author's paper can appear in a cross-domain pair.

### Credibility filtering

Before scoring, the routine discards:
- Papers with no known venue and zero citations (likely noise or auto-generated)
- Venues flagged as predatory by Beall's criteria
- Retraction notices

Preprints are allowed through if `allow_preprints_if_cited_min` is satisfied (default: 0 — all preprints pass). Set to `3` to only include preprints with at least 3 citations.

## Extending the config

### Adding author watchlists

To add a new watched author, append to the `watch_authors` array in `config.json`:

```json
{
  "name": "Ada Yonath",
  "semantic_scholar_id": "2157392983",
  "note": "Ribosome crystallography, structural biology"
}
```

The `note` field appears in the digest next to the author's name.

### Changing field coverage

The default categories span machine learning, computational biology, biological physics, soft condensed matter, dynamical systems, and economics. To focus on a narrower interdisciplinary pair — say, materials science and synthetic biology — replace `arxiv_categories` with:

```json
["cond-mat.mtrl-sci", "q-bio.BM", "q-bio.SC", "physics.chem-ph"]
```

A full list of arXiv category codes is at [arxiv.org/category_taxonomy](https://arxiv.org/category_taxonomy).

## Repository structure

```
.
├── README.md              # this file
├── routine-prompt.md      # prompt to paste into the Claude routine
├── config.example.json    # template config — copy to config.json and edit
├── config.json            # your config (gitignored if sensitive)
└── digest.md              # running weekly digest, appended by the routine
```

`config.json` is listed in `.gitignore` if you add private author notes or API keys inline. Use the routine's Environment tab for secrets instead of committing them.

## Limitations

- **arXiv only** for paper discovery. PubMed, bioRxiv, and SSRN are not queried in the default prompt — add fetch steps to `routine-prompt.md` to include them.
- **Abstract-level analysis only.** The routine reads titles and abstracts, not full text. Deep methodological connections that are only visible in the methods section will be missed.
- **No persistent memory across runs.** Each weekly run starts fresh. The "surprise bonus" for novel field pairs requires the routine to read the existing `digest.md` from the repo, which it does — but it has no vector index of prior papers.
- **Semantic Scholar coverage gaps.** Some venues (especially non-English journals and grey literature) have incomplete metadata. Venue quartile will be missing for ~20% of papers.

## License

MIT
