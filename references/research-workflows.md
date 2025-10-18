# Research Workflows and Analysis Patterns

This reference provides advanced patterns for using Zotero search as a research assistant tool, going beyond basic search to provide analysis, synthesis, and strategic guidance.

## Table of Contents

1. Research Analysis Patterns
   - Thematic Clustering
   - Gap Analysis
   - Research Trajectory Analysis
   - Methodological Survey
   - Reading List Prioritization

2. Set Operations and Combining Searches
   - Intersection (AND)
   - Union (OR)
   - Difference (NOT)
   - Temporal Filtering

3. Processing Recipes (jq patterns)
   - Group by Year
   - Extract Unique Authors
   - Find Papers Without PDFs
   - Count by Item Type
   - Extract Papers by Specific Author
   - Get Papers with Most Authors

4. Integrating Search with Full-Text Reading
   - Evidence Extraction
   - Comparative Deep Dive
   - Methodology Learning
   - When to Read vs. When to Stay with Metadata

## Research Analysis Patterns

Beyond basic search, use these patterns to analyze and synthesize literature.

### Thematic Clustering

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

### Gap Analysis

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

### Research Trajectory Analysis

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

### Methodological Survey

**When:** User needs to understand research approaches in the field

**Workflow:**
1. Extract methods information from abstracts/titles
2. Categorize: qualitative/quantitative/mixed, experimental/observational, etc.
3. Note which approaches are dominant
4. Identify methodological gaps or opportunities
5. Find exemplar papers for each approach

### Reading List Prioritization

**When:** Results are too large to read everything

**Criteria for ranking:**
1. Direct relevance to user's specific question
2. Foundational vs. recent (balance both)
3. Highly cited or influential work
4. Methodological fit for user's needs
5. Diversity of perspectives

**Workflow:**
1. Analyze all results against criteria
2. Create tiered reading list:
   - **Essential (5-7 papers)**: Start here
   - **Important (8-12 papers)**: Read next
   - **Supplementary (rest)**: For completeness
3. Within each tier, suggest reading order
4. Note why each paper matters

## Set Operations and Combining Searches

Use multiple searches with jq to find intersections, unions, and differences.

### Finding Papers About Multiple Topics (Intersection)

**Use case:** Papers discussing BOTH concept A AND concept B

```bash
# Search for each concept
pyzotero search -q "adaptation" --json > adapt.json
pyzotero search -q "vulnerability" --json > vuln.json

# Find intersection (papers in both)
jq -s '.[0].items as $a | .[1].items as $b | 
  ($a | map(.key)) as $keys_a | 
  {count: ($b | map(select(.key | IN($keys_a[]))) | length),
   items: ($b | map(select(.key | IN($keys_a[]))))}' \
  adapt.json vuln.json
```

### Combining Multiple Searches (Union)

**Use case:** Papers about ANY of several topics

```bash
pyzotero search -q "climate adaptation" --json > set1.json
pyzotero search -q "resilience" --json > set2.json

# Combine and deduplicate
jq -s '.[0].items + .[1].items | unique_by(.key) | {count: length, items: .}' \
  set1.json set2.json
```

### Finding Differences (Papers in A but not B)

**Use case:** Papers about A that don't mention B

```bash
pyzotero search -q "climate" --json > climate.json
pyzotero search -q "mitigation" --json > mitigation.json

# Papers about climate but not mitigation
jq -s '.[0].items as $a | .[1].items as $b |
  ($b | map(.key)) as $keys_b |
  {count: ($a | map(select(.key | IN($keys_b[]) | not)) | length),
   items: ($a | map(select(.key | IN($keys_b[]) | not)))}' \
  climate.json mitigation.json
```

### Temporal Filtering

**Use case:** Papers from specific date range or recent work only

```bash
# Papers from 2020 onwards
pyzotero search -q "topic" --json | \
  jq '{count: ([.items[] | select(.date >= "2020")] | length),
       items: [.items[] | select(.date >= "2020")]}'

# Papers from specific range
pyzotero search -q "topic" --json | \
  jq '{count: ([.items[] | select(.date >= "2015" and .date <= "2019")] | length),
       items: [.items[] | select(.date >= "2015" and .date <= "2019")]}'
```

## Processing Recipes

Common jq patterns for analyzing search results.

### Group by Year
```bash
pyzotero search -q "topic" --json | \
  jq '[.items | group_by(.date[:4])[] | 
      {year: .[0].date[:4], count: length, 
       titles: map(.title)}]'
```

### Extract Unique Authors
```bash
pyzotero search -q "topic" --json | \
  jq '[.items[].creators[]? | 
      select(.creatorType=="author") | 
      .firstName + " " + .lastName] | 
      sort | unique'
```

### Find Papers Without PDFs
```bash
pyzotero search -q "topic" --json | \
  jq '{missing: [.items[] | 
      select((.attachments | length) == 0) | 
      {title, date, key}],
      count: [.items[] | 
      select((.attachments | length) == 0)] | length}'
```

### Count by Item Type
```bash
pyzotero search -q "topic" --json | \
  jq '[.items | group_by(.itemType)[] | 
      {type: .[0].itemType, count: length}]'
```

### Extract Papers by Specific Author
```bash
pyzotero search -q "topic" --json | \
  jq '{items: [.items[] | 
      select(.creators[]? | 
      select(.lastName == "Smith"))],
      count: [.items[] | 
      select(.creators[]? | 
      select(.lastName == "Smith"))] | length}'
```

### Get Papers with Most Authors (Collaborative Work)
```bash
pyzotero search -q "topic" --json | \
  jq '[.items | 
      map({title, date, author_count: (.creators | length)}) | 
      sort_by(-.author_count)][:10]'
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
4. Synthesize across papers
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
