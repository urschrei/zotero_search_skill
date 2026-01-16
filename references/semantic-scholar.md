# Semantic Scholar Integration

The pyzotero CLI includes commands to search Semantic Scholar's database of over 200M papers. These commands can cross-reference results against the local Zotero library.

## Commands Overview

| Command | Purpose | Key Options |
|---------|---------|-------------|
| `related` | Find semantically similar papers | `--doi`, `--min-citations` |
| `citations` | Find papers that cite a given paper | `--doi`, `--min-citations` |
| `references` | Find papers a given paper cites | `--doi`, `--min-citations` |
| `s2search` | Search Semantic Scholar by keyword | `-q`, `--sort`, `--min-citations`, `--year` |

## Find Related Papers

Find papers semantically similar to a given paper using SPECTER2 embeddings:

```bash
pyzotero related --doi "10.1038/nature12373"
pyzotero related --doi "10.1038/nature12373" --limit 50
pyzotero related --doi "10.1038/nature12373" --min-citations 100
```

**When to use:** User wants to discover papers similar to one they already have, or wants to expand their reading in a particular direction.

## Find Citations (Who Cites This Paper)

Find papers that cite a given paper:

```bash
pyzotero citations --doi "10.1038/nature12373"
pyzotero citations --doi "10.1038/nature12373" --limit 200
pyzotero citations --doi "10.1038/nature12373" --min-citations 50
```

**When to use:** User wants to trace how a paper has influenced subsequent research, find more recent work building on a foundational paper, or assess a paper's impact.

## Find References (What This Paper Cites)

Find papers referenced by a given paper:

```bash
pyzotero references --doi "10.1038/nature12373"
pyzotero references --doi "10.1038/nature12373" --limit 200
pyzotero references --doi "10.1038/nature12373" --min-citations 100
```

**When to use:** User wants to understand the foundations of a paper, find earlier work in a research lineage, or explore the intellectual context of a paper.

## Search Semantic Scholar

Search across Semantic Scholar's full paper index:

```bash
pyzotero s2search -q "climate adaptation"
pyzotero s2search -q "machine learning" --year 2020-2024
pyzotero s2search -q "neural networks" --open-access --limit 50
pyzotero s2search -q "deep learning" --sort citations --min-citations 100
```

**Options:**
- `--year`: Filter by year (e.g., "2020", "2018-2022", "2020-")
- `--open-access`: Only return open access papers
- `--sort`: Sort by `citations` (most cited first) or `year` (most recent first)
- `--min-citations`: Only return papers with at least N citations
- `--limit`: Maximum results (default: 20, max: 100)

**When to use:** User wants to find papers beyond their local Zotero library, or wants to check what's available on a topic before adding to their collection.

## Finding Highly-Cited and Seminal Works

All Semantic Scholar commands support filtering by citation count using `--min-citations`:

```bash
# Find highly-cited related papers
pyzotero related --doi "10.1038/nature12373" --min-citations 100

# Find influential papers that cite a given work
pyzotero citations --doi "10.1038/nature12373" --min-citations 50

# Find well-cited foundational references
pyzotero references --doi "10.1038/nature12373" --min-citations 200

# Search for seminal works on a topic
pyzotero s2search -q "transformer architecture" --sort citations --min-citations 500
```

**When to use:**
- User asks for "important", "seminal", "foundational", or "highly-cited" papers
- User wants to identify the most influential works in a field
- User needs to prioritise reading the most impactful literature

**Tiering by citation count:**
- **Seminal (500+ citations)**: Foundational works everyone should know
- **Influential (100-500 citations)**: Important contributions shaping the field
- **Emerging (10-100 citations)**: Recent work gaining traction
- **New (<10 citations)**: Latest research, not yet widely cited

## Cross-Referencing with Local Zotero

All Semantic Scholar commands include an `inLibrary` field in their JSON output indicating whether each paper exists in the local Zotero library (matched by DOI). This is enabled by default.

To disable library checking (faster, but no cross-reference):
```bash
pyzotero related --doi "10.1038/nature12373" --no-check-library
```

**Interpreting results:**
- High `inLibrary: true` count = good coverage of the field
- Many highly-cited papers with `inLibrary: false` = significant gaps
- Use citation counts to prioritise which gaps to fill first

## Output Format

All Semantic Scholar commands output JSON:

```json
{
  "count": 20,
  "papers": [
    {
      "paperId": "abc123",
      "doi": "10.1234/example",
      "title": "Example Paper Title",
      "authors": ["John Doe", "Jane Smith"],
      "year": 2023,
      "venue": "Nature",
      "citationCount": 150,
      "referenceCount": 45,
      "isOpenAccess": true,
      "openAccessPdfUrl": "https://example.com/paper.pdf",
      "inLibrary": false
    }
  ]
}
```

## Common Workflows

### Literature Expansion from a Seed Paper

```
User: "I have this paper about coral bleaching (DOI: 10.1234/coral). What else should I read?"

1. Run: pyzotero related --doi "10.1234/coral"
2. Present related papers, highlighting those NOT in library (inLibrary: false)
3. Offer: "5 of these 20 related papers are already in your library. Want me to focus on the 15 new ones?"
```

### Finding Foundational and Follow-up Work

```
User: "I want to understand the impact of this key paper"

1. Run: pyzotero references --doi "..." (what it builds on)
2. Run: pyzotero citations --doi "..." (what built on it)
3. Summarise: "This paper cites 45 works (foundational literature) and has been cited 150 times. Of the citing papers, 3 are in your library."
```

### Checking Coverage on a Topic

```
User: "Am I missing anything important on topic X?"

1. Run: pyzotero s2search -q "topic X" --sort citations --min-citations 50 --limit 50
2. Compare with local search: pyzotero search -q "topic X" --json
3. Report: "Semantic Scholar shows 50 highly-cited papers on this topic. You have 12 in your library. Here are the most-cited ones you're missing..."
```

### Snowball from a Seed Paper

Start with one important paper and expand outward:

```bash
# 1. Find semantically related papers
pyzotero related --doi "10.1234/seed-paper" --min-citations 50

# 2. Find what it cites (its foundations)
pyzotero references --doi "10.1234/seed-paper" --min-citations 100

# 3. Find what cites it (subsequent work)
pyzotero citations --doi "10.1234/seed-paper" --min-citations 50
```

### Finding Review Articles

Review articles synthesise a field and cite many foundational works:

```bash
# Search for reviews (often have "review" in title)
pyzotero s2search -q "topic review" --sort citations --min-citations 100

# Then get references from a good review to find key primary sources
pyzotero references --doi "10.1234/review-article" --min-citations 50
```

## Error Handling

- **"Paper not found"**: The DOI may not be in Semantic Scholar's index. Try searching by title instead using `s2search`.
- **"Rate limit exceeded"**: Wait a moment and try again. Semantic Scholar has a shared rate limit of 1000 req/s for unauthenticated users.
- **Connection errors**: Check internet connectivity.
