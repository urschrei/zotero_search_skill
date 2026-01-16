# Output and Analysis Guidelines

Guidelines for presenting search results and providing proactive research assistance.

## Presenting Search Results

Format search results for readability based on result count:

### For 1-10 Results: Full Details

Show complete information for each paper:
- Title (with author, year)
- Publication venue
- Abstract (first 2-3 sentences if lengthy)
- DOI or URL
- Local file path (if available)
- If it wasn't a full-text search, point out that this may yield more results

### For 11-20 Results: Condensed Format

Show essential information:
- Title, authors, year
- Publication venue
- Local file path

### For 21+ Results: Overview

Provide summary and selective detail:
- Total count and brief summary
- Group by theme/category if applicable
- Show top 5-10 most relevant results
- Offer to show more details or refine search
- Any publication mentioned should include author and year

### Always Include

1. Total result count at the start ("Found 12 journal articles")
2. Local file paths when PDFs are attached (highly useful for user)
3. Offer to refine search if results seem too broad or narrow

Adapt format based on context and user needs - these are sensible defaults.

## Proactive Analysis

After presenting search results, analyse findings and suggest productive next steps. Act as a research assistant, not just a command executor.

### Pattern Recognition (Always Do This)

1. Identify themes or clusters in results
2. Notice temporal patterns (date ranges, gaps)
3. Check for missing expected work or authors
4. Assess coverage (methodologies, perspectives, geographies)

### Suggest Next Steps Based on Findings

**If results show clear themes:**
- "These 15 papers cluster into 3 approaches: [A], [B], [C]. Want me to break down each cluster?"
- "I notice 2 distinct debates here. Want to explore each separately?"

**If temporal patterns emerge:**
- "Most papers are from 2020-2023. Want to search for earlier foundational work?"
- "There's a gap from 2015-2018. Want to search that period specifically?"

**If coverage seems incomplete:**
- "These are all quantitative studies. Want to search for qualitative approaches?"
- "All papers are from Western contexts. Want to search for [other region] research?"
- "I see methodological papers but few empirical applications. Want to find applications?"

**If specific authors dominate:**
- "5 papers by [Author]. Want to find other perspectives or critiques?"
- "Several papers cite [Key Author]. Want to search for their work?"

**If PDFs are missing:**
- "Only 4 of 12 papers have PDFs attached. Want me to identify which are missing?"

**If results are too broad:**
- "Got 47 results. Want to filter by time period, item type, or add more specific terms?"

**If results are too narrow:**
- "Only 2 results. Try broader terms, remove filters, or use full-text search?"

### Research Workflow Suggestions

Offer to go deeper when appropriate:
- "Ready to do a thematic analysis of these results?"
- "Want me to identify gaps in this literature?"
   - You can use the semantic scholar integration for this
- "Should we create a reading priority list?"
- "Want to track how thinking evolved over time?"

### Tone Guidelines

- Don't be too pushy, overeager, or obsequious
- Act like a diligent postdoc
- Respect user preferences: they may want "just" search results, extensive analysis, or anything in between
- Allow users to change their mind about the level of assistance

## Common Usage Patterns

### Pattern 1: Basic Exploratory Search

```
User: "Search my Zotero for papers about ocean acidification"

1. Run: pyzotero search -q "ocean acidification" --itemtype journalArticle --json
2. Present results
3. Analyse: "Found 12 papers. I notice 3 focus on coral reefs, 8 on broader marine
   ecosystems, and 1 on economic impacts. Most are from 2018-2023. Want me to
   break down each cluster?"
```

### Pattern 2: Full-Text Search with Gap Analysis

```
User: "Find everything in my library that mentions 'adaptation strategies'"

1. Run: pyzotero search -q "adaptation strategies" --fulltext --json
2. Present results
3. Analyse: "Found 23 papers, but all are from developed countries. Want to search
   for developing world contexts? Also, these are all climate-focused - want to
   broaden to other risks?"
```

### Pattern 3: Advanced Research Workflow

```
User: "What do I have about neural networks? I need to understand the landscape."

1. Run: pyzotero search -q "neural networks" --json
2. Present results with initial analysis
3. Suggest: "Found 18 papers. Want me to do a thematic clustering? Or track how
   thinking evolved over time? Or create a prioritised reading list?"
4. If user wants advanced analysis, use patterns from references/research-patterns.md
```

### Pattern 4: Combining Local and External Search

```
User: "Am I missing important papers on topic X?"

1. Run local search: pyzotero search -q "topic X" --json
2. Run Semantic Scholar search: pyzotero s2search -q "topic X" --sort citations --min-citations 50
3. Compare: identify highly-cited papers with inLibrary: false
4. Report: "You have 12 papers locally. Semantic Scholar shows 8 highly-cited papers
   you're missing. Here are the top 3 by citation count..."
```

## Semantic Scholar Result Presentation

When presenting Semantic Scholar results, highlight the cross-reference information:

**Example format:**
```
Found 20 related papers (5 already in your library):

IN LIBRARY:
1. [Author 2021] "Title" - 234 citations
   You have this paper

NOT IN LIBRARY (potential additions):
2. [Author 2022] "Title" - 567 citations - HIGHLY CITED
   Open access PDF available

3. [Author 2020] "Title" - 123 citations
   ...
```

Prioritise papers by:
1. Citation count (highly-cited first)
2. Open access availability
3. Recency (for emerging topics)
