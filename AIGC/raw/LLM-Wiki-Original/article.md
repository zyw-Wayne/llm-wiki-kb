# LLM Wiki (Original English)

**Category**: Pattern  
**Tags**: Knowledge Base, RAG, Personal Knowledge Management, LLM

---

## The Core Idea

Most people's experience with LLMs and documents looks like RAG: you upload a collection of files, the LLM retrieves relevant chunks at query time, and generates an answer. This works, but the LLM is **rediscovering knowledge from scratch on every question**. There's no accumulation.

**The idea here is different.** Instead of just retrieving from raw documents at query time, the LLM incrementally builds and maintains a **persistent wiki** — a structured, interlinked collection of markdown files that sits between you and the raw sources.

**Key difference:** The wiki is a persistent, compounding artifact. The cross-references are already there. The contradictions have already been flagged. The synthesis already reflects everything you've read. The wiki keeps getting richer with every source you add and every question you ask.

**You never (or rarely) write the wiki yourself** — the LLM writes and maintains all of it. You're in charge of sourcing, exploration, and asking the right questions. The LLM does all the grunt work — the summarizing, cross-referencing, filing, and bookkeeping.

## Use Cases

| Context | Application |
|---------|-------------|
| **Personal** | Tracking goals, health, psychology — filing journal entries, articles, podcast notes |
| **Research** | Deep dives over weeks/months — reading papers, building comprehensive wiki with evolving thesis |
| **Reading a book** | Filing each chapter, building pages for characters, themes, plot threads (like fan wikis) |
| **Business/team** | Internal wiki fed by Slack threads, meeting transcripts, project documents |
| **Other** | Competitive analysis, due diligence, trip planning, course notes, hobby deep-dives |

## Architecture

Three layers:

1. **Raw sources** — Your curated collection of source documents. Immutable. LLM reads but never modifies. This is your source of truth.

2. **The wiki** — Directory of LLM-generated markdown files. Summaries, entity pages, concept pages, comparisons, synthesis. **The LLM owns this layer entirely.** You read it; the LLM writes it.

3. **The schema** — A document (e.g., `CLAUDE.md` or `AGENTS.md`) that tells the LLM how the wiki is structured, what conventions to follow, and what workflows to use. You and the LLM co-evolve this over time.

## Operations

### Ingest
Drop a new source into the raw collection and tell the LLM to process it.
- LLM reads the source, discusses key takeaways with you
- Writes summary page in wiki, updates index
- Updates relevant entity/concept pages across the wiki
- Appends entry to log
- A single source might touch 10-15 wiki pages

### Query
Ask questions against the wiki.
- LLM searches for relevant pages, reads them, synthesizes answer with citations
- Answers can be: markdown page, comparison table, slide deck (Marp), chart (matplotlib), canvas
- **Good answers can be filed back into the wiki as new pages** — explorations compound in the knowledge base

### Lint
Periodically health-check the wiki. Look for:
- Contradictions between pages
- Stale claims that newer sources have superseded
- Orphan pages with no inbound links
- Important concepts mentioned but lacking their own page
- Missing cross-references
- Data gaps that could be filled with web search

## Indexing and Logging

Two special files:

| File | Purpose |
|------|---------|
| **index.md** | Content-oriented catalog. Each page listed with link, one-line summary, metadata. Organized by category. LLM reads this first to find relevant pages. |
| **log.md** | Chronological, append-only record. Each entry with consistent prefix (e.g., `## [2026-04-02] ingest | Article Title`). Parseable with unix tools. Timeline of wiki's evolution. |

## Tips and Tricks

- **Obsidian Web Clipper**: Browser extension that converts web articles to markdown
- **Download images locally**: In Obsidian, set attachment folder, use hotkey to download all images from article
- **Graph view**: Best way to see shape of wiki — connections, hubs, orphans
- **Marp**: Markdown-based slide deck format (Obsidian plugin available)
- **Dataview**: Obsidian plugin for querying page frontmatter
- **Git repo**: Wiki is just markdown files — version history, branching, collaboration for free

## Why This Works

> The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping.

Humans abandon wikis because the maintenance burden grows faster than the value. **LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass.**

**The human's job**: Curate sources, direct analysis, ask good questions, think about what it all means.

**The LLM's job**: Everything else — summarizing, cross-referencing, filing, bookkeeping, maintaining consistency.

## Related Ideas

Related in spirit to **Vannevar Bush's Memex (1945)** — a personal, curated knowledge store with associative trails between documents. Bush's vision was private, actively curated, with connections between documents as valuable as the documents themselves. The part he couldn't solve was **who does the maintenance**. The LLM handles that.

---

**Source**: Original LLM Wiki idea document (English)
