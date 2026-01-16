# Research Analysis Patterns

Advanced patterns for using Zotero search as a research assistant tool, going beyond basic search to provide analysis, synthesis, and strategic guidance.

## Thematic Clustering

**When:** User has 10+ papers and wants to understand the landscape

**Workflow:**
1. Get abstracts: `pyzotero search -q "topic" --json | jq '.items[] | {title, abstractNote, date}'`
2. Read and identify recurring themes, theoretical approaches, or methods
3. Group papers by theme with counts
4. Describe each cluster with representative papers
5. Note relationships between clusters

**Example output format:**
```
Found 3 main themes in 18 papers:

Theme 1: Adaptation through policy (7 papers)
- Focus: Top-down institutional approaches
- Key papers: [Author1 2021], [Author2 2020]
- Methods: Policy analysis, case studies

Theme 2: Community-based adaptation (8 papers)
- Focus: Bottom-up local initiatives
- Key papers: [Author3 2022], [Author4 2019]
- Methods: Participatory research, ethnography

Theme 3: Techno-managerial solutions (3 papers)
- Focus: Engineering and infrastructure
- Key papers: [Author5 2023]
- Methods: Modeling, quantitative assessment
```

## Gap Analysis

**When:** User wants to identify what's missing or underexplored

**Workflow:**
1. Map what's well-covered (themes, methods, periods, regions)
2. Identify sparse areas
3. Check for methodological imbalances
4. Note missing perspectives or voices
5. Suggest targeted searches to fill gaps

**Questions to ask:**
- What time periods are underrepresented?
- What methods are dominant vs. absent?
- What geographic regions are covered?
- What populations or cases are studied?
- What theoretical perspectives are present?

**Comparing local library against broader literature:**

Use Semantic Scholar to find important papers not yet in the library:

```bash
# Search S2 for highly-cited papers on a topic
pyzotero s2search -q "topic" --sort citations --min-citations 200 --limit 50

# Results include inLibrary: true/false
# Papers with inLibrary: false and high citations are gaps worth filling
```

**Gap analysis workflow:**
1. Search local Zotero for a topic: `pyzotero search -q "topic" --json`
2. Search Semantic Scholar for the same topic, sorted by citations
3. Compare: which highly-cited papers are missing locally?
4. For papers in library, check their references for foundational works:
   `pyzotero references --doi "..." --min-citations 100`
5. Report: "You have 15 papers on X, but are missing these 5 highly-cited works..."

**Finding what you should have cited:**
```bash
# For a paper you're writing about, find highly-cited related work
pyzotero related --doi "10.1234/your-paper" --min-citations 100
# Any with inLibrary: false might be important omissions
```

## Research Trajectory Analysis

**When:** User wants to understand how thinking evolved

**Workflow:**
1. Sort by date: `jq '.items | sort_by(.date)' results.json`
2. Identify early foundational work
3. Track how questions/methods shifted over time
4. Note paradigm shifts or turning points
5. Identify current frontier

**Output format:**
```
Evolution of [topic] research:

Early period (2010-2015): 5 papers
- Focus: Problem definition and measurement
- Seminal: [Author1 2012]

Middle period (2016-2019): 12 papers
- Focus: Causal mechanisms and theory-building
- Shift: From correlation to causation
- Key debate: [X vs Y perspective]

Recent period (2020-2024): 15 papers
- Focus: Solutions and interventions
- Methods: More experimental, mixed-methods
- Current frontier: [emerging questions]
```

**Tracing intellectual lineages with Semantic Scholar:**

Use citations and references to trace how ideas developed:

```bash
# Find what a seminal paper built upon (its foundations)
pyzotero references --doi "10.1234/seminal-paper" --min-citations 100

# Find influential work that built on the seminal paper
pyzotero citations --doi "10.1234/seminal-paper" --min-citations 50

# Trace a chain: paper -> its most-cited references -> their references
```

**Workflow for tracing a research lineage:**
1. Start with a key paper in the user's library
2. Use `references` to find its intellectual foundations (filter by citations for the important ones)
3. Use `citations` to find subsequent influential work
4. Check `inLibrary` to see which papers the user already has
5. Recommend filling gaps in the lineage

## Methodological Survey

**When:** User needs to understand research approaches in the field

**Workflow:**
1. Extract methods information from abstracts/titles
2. Categorise: qualitative/quantitative/mixed, experimental/observational, etc.
3. Note which approaches are dominant
4. Identify methodological gaps or opportunities
5. Find exemplar papers for each approach

## Reading List Prioritisation

**When:** Results are too large to read everything

**Criteria for ranking:**
1. Direct relevance to user's specific question
2. Foundational vs. recent (balance both)
3. Highly cited or influential work
4. Methodological fit for user's needs
5. Diversity of perspectives

**Workflow:**
1. Analyse all results against criteria
2. Create tiered reading list:
   - **Essential (5-7 papers)**: Start here
   - **Important (8-12 papers)**: Read next
   - **Supplementary (rest)**: For completeness
3. Within each tier, suggest reading order
4. Note why each paper matters

**Using citation counts to identify influential work:**

```bash
# Find highly-cited papers on a topic (sorted by citations)
pyzotero s2search -q "topic" --sort citations --min-citations 100

# For papers already in library, check their citation impact via Semantic Scholar
# by looking up individual DOIs
pyzotero citations --doi "10.1234/example" --min-citations 50
```

**Tiering by citation count:**
- **Seminal (500+ citations)**: Foundational works everyone should know
- **Influential (100-500 citations)**: Important contributions shaping the field
- **Emerging (10-100 citations)**: Recent work gaining traction
- **New (<10 citations)**: Latest research, not yet widely cited

## Finding Seminal Works

**When:** User asks for "foundational", "seminal", "classic", or "most important" papers on a topic

**Workflow:**
1. Search Semantic Scholar sorted by citations:
   ```bash
   pyzotero s2search -q "topic" --sort citations --min-citations 500 --limit 20
   ```
2. Check which are already in library (`inLibrary` field)
3. For the top results, verify they're actually foundational (not just popular):
   - Check publication date (seminal works are often older)
   - Read abstracts to confirm they're theoretical/methodological contributions
4. Present tiered results with context

**Example output format:**
```
Seminal works on [topic] (sorted by citations):

Foundational (1000+ citations):
1. [Author 2005] "Title" - 3,421 citations - IN LIBRARY
   Established the theoretical framework for...

2. [Author 2008] "Title" - 2,156 citations - NOT IN LIBRARY
   Introduced the methodology widely used for...

Highly Influential (500-1000 citations):
3. [Author 2012] "Title" - 876 citations - IN LIBRARY
   ...
```

**Finding foundational works for a specific paper:**
```bash
# What did this influential paper build upon?
pyzotero references --doi "10.1234/influential" --min-citations 200
```

## Literature Expansion Workflows

**When:** User wants to systematically expand their reading from papers they already have

### Pattern 1: Snowball from a Seed Paper

Start with one important paper and expand outward:

```bash
# 1. Find semantically related papers
pyzotero related --doi "10.1234/seed-paper" --min-citations 50

# 2. Find what it cites (its foundations)
pyzotero references --doi "10.1234/seed-paper" --min-citations 100

# 3. Find what cites it (subsequent work)
pyzotero citations --doi "10.1234/seed-paper" --min-citations 50
```

Check `inLibrary` in results to identify gaps.

### Pattern 2: Expand from a Collection

For a set of papers (e.g., a Zotero collection):

1. Get DOIs from the collection:
   ```bash
   pyzotero search --collection COLLECTIONKEY --json | jq '.items[].doi'
   ```

2. For each DOI, find related papers:
   ```bash
   pyzotero related --doi "DOI" --min-citations 50 --limit 10
   ```

3. Aggregate results, deduplicate, and identify papers that appear related to multiple seed papers (these are likely central to the topic)

### Pattern 3: Citation Network Exploration

For understanding a research community:

1. Start with a highly-cited paper in the field
2. Get its citations (who builds on it):
   ```bash
   pyzotero citations --doi "..." --min-citations 20 --limit 100
   ```
3. Among citing papers, identify those that are themselves highly cited (influential followers)
4. These form the "core" of an active research programme

### Pattern 4: Finding Review Articles

Review articles synthesise a field and cite many foundational works:

```bash
# Search for reviews (often have "review" in title)
pyzotero s2search -q "topic review" --sort citations --min-citations 100

# Then get references from a good review to find key primary sources
pyzotero references --doi "10.1234/review-article" --min-citations 50
```

## Integrating Search with Full-Text Reading

After search identifies key papers, read PDFs directly for deeper analysis. The local file paths in search results enable seamless transition from discovery to reading.

### Pattern: Evidence Extraction

**When:** User needs specific evidence or quotes about a claim

**Workflow:**
1. Search for papers on the topic
2. Identify 5-10 most relevant papers
3. For each PDF at local path, read and extract:
   - Main arguments/claims
   - Supporting evidence
   - Methodology used
   - Key quotes (with page numbers)
   - Limitations acknowledged
4. Synthesise across papers
5. Build evidence map showing supporting/contradicting findings

### Pattern: Comparative Deep Dive

**When:** User wants detailed comparison of approaches/theories

**Workflow:**
1. Search identifies papers representing different perspectives
2. Read PDFs of representative papers (3-5 per perspective)
3. Extract for each:
   - Core theoretical assumptions
   - Methodological approach
   - Key findings
   - How they frame the problem
4. Create comparison table
5. Note where perspectives agree, disagree, and remain silent

### Pattern: Methodology Learning

**When:** User wants to understand how to apply a specific method

**Workflow:**
1. Search for papers using that method
2. Filter to exemplar papers (well-cited, clear methodology sections)
3. Read methods sections from 3-5 papers
4. Extract common patterns:
   - Research design choices
   - Data collection procedures
   - Analysis approaches
   - Common pitfalls and how addressed
5. Create methodology guide with examples

### Knowing When to Read Full Texts

**Read PDFs when:**
- User asks for detailed analysis of arguments or evidence
- Need to extract specific information not in abstracts
- Comparing theoretical frameworks or methodologies in depth
- User is writing and needs quotes or specific citations
- Abstracts don't provide enough detail to answer question

**Stay with metadata when:**
- User wants landscape overview or "what exists"
- Identifying themes across many papers (abstracts sufficient)
- Initial exploration phase
- User asks about counts, distributions, temporal patterns
