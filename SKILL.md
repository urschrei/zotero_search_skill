---
name: zotero-search
description: Search and retrieve documents from the user's local Zotero library using the Pyzotero CLI. **Claude code only** - requires connection to local Zotero instance. This skill should be used when the user asks to search their Zotero library, find papers or references, or query their personal research collection. Returns rich metadata including titles, authors, dates, abstracts, and bibliographic information with local file paths.
---

# Zotero Search

## Overview

**⚠️ CLAUDE CODE ONLY** - This skill requires Claude code as it connects to a local Zotero instance running on localhost. It will not work in web Claude at claude.ai.

This skill enables searching and retrieving documents from the user's local Zotero library using the Pyzotero command-line interface. It provides comprehensive search capabilities across titles, full-text content, metadata, authors, collections, and item types.

**Future possibility:** The skill could be extended to support Zotero's Web API for use in web Claude, but this would require API credentials (library ID and API key).

## Prerequisites

**Requirements:**
- Python ≥3.9
- Pyzotero ≥1.7.3 with CLI support
- Claude Code (this skill does not work in web Claude)
- Zotero desktop application running locally with local API enabled

**Installation check:**
```bash
pyzotero test
```

If Pyzotero is not installed, install using:
```bash
uv tool install "pyzotero[cli]"
```

**Alternative:** Run without permanent installation using:
```bash
uvx --from "pyzotero[cli]" pyzotero search -q "your query"
pipx run --spec "pyzotero[cli]" pyzotero search -q "your query"
```
The second prerequsite is the Zotero desktop application running locally, with local API enabled. Test by calling:

```bash
pyzotero test
```
The response will be either success:

```text
✓ Connection successful: Zotero is running and listening locally.
  Received expected empty settings response.
```
or failure:

```text
✗ Connection failed: Could not connect to Zotero.

Possible causes:
  • Zotero might not be running
  • Local connections might not be enabled

To enable local connections:
  Zotero > Settings > Advanced > Allow other applications on this computer to communicate with Zotero
```

In case of failure, **pass the failure message on the user and ask them to tell you when they've fixed the problem, then try again**. In case of success, the skill is ready to use.

If the user is on web Claude, inform them that this skill requires Claude Code to access the local Zotero library. Do not continue after this as the skill cannot run.

## Quick Start

When the user asks to search their Zotero library, use the `pyzotero` CLI with appropriate commands. The CLI can output either human-readable text or JSON format. It will output a 'count' field showing the number of returned results.

**Basic search example:**
```bash
pyzotero search -q "climate adaptation"
```

**JSON output for structured data:**
```bash
pyzotero search -q "climate adaptation" --json
```

## Search Capabilities

### 1. Basic Query Search

Search across titles and metadata using the `-q` parameter.

**When to use:** User provides a topic, keyword, or phrase to search for.

**Examples:**
```bash
# Basic search
pyzotero search -q "machine learning"

# Get JSON output for structured processing
pyzotero search -q "machine learning" --json
```

### 2. Full-Text Search

Search across all content including full-text of PDFs using the `--fulltext` flag.

**When to use:** User wants to search within document contents, not just titles and metadata.

**Example:**
```bash
pyzotero search -q "climate change" --fulltext
```

**Note:** Full-text search is more comprehensive but may be slower and requires Zotero to have indexed the document contents.

### 3. Filter by Item Type

Restrict search to specific document types using `--itemtype` (can be specified multiple times).

**When to use:** User specifies "articles", "books", "papers", or other document types.

**Common item types:**
- `journalArticle` - Journal articles
- `book` - Books
- `bookSection` - Book chapters
- `conferencePaper` - Conference papers
- `report` - Technical reports
- `thesis` - Theses and dissertations

**List all available item types:**
```bash
pyzotero itemtypes
```

**Examples:**
```bash
# Search only journal articles
pyzotero search -q "neural networks" --itemtype journalArticle

# Search books and book chapters
pyzotero search -q "methodology" --itemtype book --itemtype bookSection

# Multiple types with JSON output
pyzotero search -q "climate" --itemtype journalArticle --itemtype conferencePaper --json
```

**Important:** If the user doesn't specify an item type, prompt them whether they want to search all items or filter by type (articles, books, etc.). Consider running `pyzotero itemtypes` first to show available options.

### 4. Search Within Collections

Search within a specific Zotero collection using `--collection`.

**When to use:** User mentions searching within a specific collection or folder.

**Example:**
```bash
# Search within a collection (requires collection key)
pyzotero search --collection ABC123 -q "adaptation"
```

**Finding collection keys:**
```bash
# List all collections with their keys
pyzotero listcollections
```

**Workflow:** If user mentions a collection by name, first run `pyzotero listcollections` to find the collection key, then search within it.

### 5. Limiting Results

Control the number of results with the `--limit` parameter.

**When to use:** User wants more/fewer results, or initial search returns too many items.

**Example:**
```bash
pyzotero search -q "adaptation" --limit 10
```

### 6. Combining Search Parameters

All search parameters can be combined for precise queries.

**Examples:**
```bash
# Full-text search in journal articles, limited results
pyzotero search -q "CRISPR" --fulltext --itemtype journalArticle --limit 20

# Search within collection with item type filter
pyzotero search --collection ABC123 -q "climate" --itemtype book --json

# Multiple item types with full-text search
pyzotero search -q "renewable energy" --fulltext --itemtype journalArticle --itemtype conferencePaper
```

## Output Format

### Human-Readable Output (Default)

By default, the CLI outputs formatted text with:
- Title, authors, date, publication
- Volume, issue, DOI, URL
- PDF attachments with **local file paths**
- Key metadata fields

**When to use:** For casual queries or when user wants readable output.

### JSON Output (`--json` flag)

Structured JSON with complete metadata for each item. It has the following structure:

```
{
    "count": len(output_items),
    "items": output_items
}
```

**When to use:** 
- Need to process results programmatically
- Want complete metadata
- Need to extract specific fields
- Building reports or analyses

**Example JSON structure:**
```json
{
  "key": "ABCD1234",
  "itemType": "journalArticle",
  "title": "Article Title",
  "creators": [
    {"creatorType": "author", "firstName": "John", "lastName": "Doe"}
  ],
  "date": "2023-05-15",
  "publicationTitle": "Journal Name",
  "volume": "42",
  "issue": "3",
  "pages": "123-145",
  "DOI": "10.1234/example",
  "abstractNote": "Full abstract text...",
  "tags": [{"tag": "keyword1"}, {"tag": "keyword2"}],
  "url": "https://...",
  "attachments": [
    {"path": "/path/to/local/file.pdf", "title": "Full Text PDF"}
  ]
}
```

## Presenting Results to User

Format search results for readability based on result count:

**For 1-10 results:** Show full details for each
- Title (with author, year)
- Publication venue
- Abstract (first 2-3 sentences if lengthy)
- DOI and URL
- Full local file path (if available)

**For 11-20 results:** Show condensed format
- Title, authors, year
- Publication venue
- Full local file path (if available)
- Offer to show abstract (first 2-3 sentences if lengthy)

**For 21+ results:** Provide overview
- Total count and brief summary
- Group by theme/category if applicable, with count
- Show top 5-10 most relevant results. If grouping, show most relevant results _per group_
- Offer to show more details or refine search

**Always include:**
1. Total result count at the start ("Found 12 journal articles")
2. Local file paths when PDFs are attached (highly useful for user)
3. Offer to refine search if results seem too broad or narrow

Adapt format based on context and user needs—these are sensible defaults.

## Workflow Decision Tree

1. **User mentions searching Zotero?** → Use this skill
2. **Verify Desktop Claude** → If web Claude, inform limitation
3. **Check if Pyzotero CLI is available** → If not, inform user about installation
4. **Determine search parameters:**
   - What is the query/topic?
   - Should search be full-text or metadata-only?
   - Should results be filtered by item type? (Prompt if not specified, show options with `pyzotero itemtypes`)
   - Is this within a specific collection? (Use `pyzotero listcollections` to find key)
   - How many results needed?
   - JSON output or human-readable?
5. **Execute search** using `pyzotero search` with appropriate flags
6. **Present results** in a readable format, highlighting:
   - Number of results found. This is also available for JSON output. This count is highly informative for the user and should always be shown
   - For each result: title, authors, publication info, abstract (if relevant)
   - Local file paths for PDFs (very useful!). Always show full file paths
7. **Offer refinement** if results are too broad or too narrow

## Common Usage Patterns

**Pattern 1: Exploratory topic search**
```
User: "Search my Zotero for papers about ocean acidification"
→ pyzotero search -q "ocean acidification" --itemtype journalArticle --json
```

**Pattern 2: Full-text search across all documents**
```
User: "Find everything in my library that mentions 'adaptation strategies'"
→ pyzotero search -q "adaptation strategies" --fulltext
```

**Pattern 3: Finding items in a specific collection**
```
User: "What do I have about renewable energy in my Climate Research collection?"
→ pyzotero listcollections  # Find collection key
→ pyzotero search --collection <KEY> -q "renewable energy"
```

**Pattern 4: Comprehensive filtered search**
```
User: "Find recent journal articles and conference papers about neural networks"
→ pyzotero search -q "neural networks" --itemtype journalArticle --itemtype conferencePaper --fulltext --json
```

**Pattern 5: Discovering available options**
```
User: "What types of documents can I search for?"
→ pyzotero itemtypes

User: "What collections do I have?"
→ pyzotero listcollections
```

## Helper Commands

### List Collections
```bash
pyzotero listcollections
```
Shows all collections with their keys and names. Use this when user refers to a collection by name to find its key.

### List Item Types
```bash
pyzotero itemtypes
```
Shows all available item types that can be used with `--itemtype`. Use this when user is unsure what types to search for.

## Error Handling

If commands fail:
- **"Connection error"** → Check if Zotero desktop application is running
- **"Command not found"** → Verify Pyzotero CLI is installed: `uv tool install --with click "pyzotero"`
- **"No results"** → Try:
  - Removing item type filters
  - Using full-text search with `--fulltext`
  - Broadening the query
  - Checking collection key with `pyzotero listcollections`

## Tips for Best Results

1. **Start broad, then narrow:** Begin with a simple query, then add filters based on results
2. **Use full-text for comprehensive searches:** Add `--fulltext` when searching for specific concepts that might not be in titles
3. **Always use `--json` for processing:** When you need to extract specific information or present structured data
4. **Leverage local file paths:** The output includes paths to local PDFs, which is very useful for file operations
5. **Show item type options:** When user asks about filtering, show them `pyzotero itemtypes` output first
6. **List collections proactively:** If user mentions collections, show them `pyzotero listcollections` to see what's available
7. **Keep track of counts:** When grouping or categorising results, display a count per category
8. **Use jq, sed, and awk for counting, mixing, and matching:** Result sets can potentially be very large. Use standard tools at your disposal to manage this
