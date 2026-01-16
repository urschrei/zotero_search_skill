# jq Processing Recipes

Technical jq patterns for processing and combining pyzotero search results.

## Set Operations

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

### Comparing Local Library vs External Literature

**Use case:** Finding important papers you don't have yet

The Semantic Scholar commands automatically include `inLibrary: true/false` for each result, making it easy to identify gaps.

**Workflow 1: Find missing highly-cited papers**
```bash
# Search S2 for highly-cited papers on a topic
pyzotero s2search -q "topic" --sort citations --min-citations 100 --limit 50

# Filter results to show only papers NOT in library
# (parse JSON output and filter where inLibrary is false)
```

**Workflow 2: Compare coverage systematically**
```bash
# 1. Count papers in local library on topic
pyzotero search -q "topic" --json | jq '.count'

# 2. Search S2 for highly-cited papers
pyzotero s2search -q "topic" --sort citations --min-citations 50 --limit 100

# 3. Count how many of those you have vs don't have
# Results show inLibrary field for each paper
```

**Workflow 3: Find related papers not in library**
```bash
# For a key paper you have, find related work you're missing
pyzotero related --doi "10.1234/paper-you-have" --min-citations 50

# Papers with inLibrary: false are potential additions
```

## Processing Recipes

Common jq patterns for analysing search results.

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

### Sort by Date (Newest First)

```bash
pyzotero search -q "topic" --json | \
  jq '.items | sort_by(.date) | reverse | .[:10]'
```

### Extract DOIs

```bash
pyzotero search -q "topic" --json | \
  jq '[.items[] | select(.doi != null and .doi != "") | .doi]'
```

### Find Papers with Abstracts

```bash
pyzotero search -q "topic" --json | \
  jq '[.items[] | select(.abstractNote != null and .abstractNote != "") |
      {title, date, abstract: .abstractNote[:200]}]'
```

### Count Papers by Publication

```bash
pyzotero search -q "topic" --json | \
  jq '[.items | group_by(.publication)[] |
      {publication: .[0].publication, count: length}] |
      sort_by(-.count)'
```

### Extract Papers from Date Range with Specific Author

```bash
pyzotero search -q "topic" --json | \
  jq '[.items[] |
      select(.date >= "2020" and .date <= "2024") |
      select(.creators[]? | .lastName == "Smith")]'
```

## Combining Operations

### Multi-Step Analysis Pipeline

```bash
# Find recent papers without PDFs by a specific author
pyzotero search -q "topic" --json | \
  jq '[.items[] |
      select(.date >= "2020") |
      select((.attachments | length) == 0) |
      select(.creators[]? | .lastName == "Smith") |
      {title, date, doi}]'
```

### Create Citation-Ready List

```bash
pyzotero search -q "topic" --json | \
  jq '.items | sort_by(.date) | .[] |
      "\(.creators[0].lastName // "Unknown") (\(.date[:4] // "n.d.")). \(.title)"'
```

### Export for Further Processing

```bash
# Export to CSV-like format
pyzotero search -q "topic" --json | \
  jq -r '.items[] | [.date[:4], .title, .doi] | @tsv'
```
