# ScholarAI Pre-Research Use Cases

## UC-PR01: Paper Web Search & Retrieval

### Actors
- Researcher
- WebSearch Agent (FastAPI)

### Goal
Search for relevant papers across academic databases using project topics or user queries.

### Main Flow
1. Researcher clicks “Fetch Papers” or enters a custom search query.
2. Agent queries sources (Semantic Scholar, CrossRef, arXiv).
3. Papers are fetched with metadata and downloaded if accessible.
4. Duplicates (by DOI) are avoided.
5. Results are added to the Project Library.

---

## UC-PR02: PDF Viewer, Editor, Annotator

### Actors
- Researcher

### Goal
Read, highlight, annotate papers directly in-browser.

### Main Flow
1. Researcher opens a paper from Library.
2. System renders a PDF viewer with annotation tools.
3. Notes and highlights are saved in project DB, scoped to user.

---

## UC-PR03: Paper Extraction, Summarization, and Analysis

### Actors
- Researcher
- Scraper Agent
- Summarizer Agent

### Goal
Extract paper content, generate summaries (human & machine-friendly), enable quick insights.

### Main Flow
1. User selects one or more papers.
2. System extracts content and sends to summarizer.
3. Outputs:
   - Problem & Motivation
   - Key Contributions
   - Method Overview
   - Structured JSON for downstream agents

---

## UC-PR04: AI Critic Paper Scoring

### Actors
- Researcher
- Critic Agent

### Goal
Score each paper based on citation, venue prestige, relevance, recency, etc.

### Main Flow
1. User clicks "Score Papers"
2. System computes score for each item
3. Library is sorted or filtered based on scores

---

## UC-PR05: Gap Analysis & Topic Suggestion

### Actors
- Researcher
- Gap Analysis Agent

### Goal
Identify unaddressed areas and propose promising research topics.

### Main Flow
1. Researcher clicks “Analyze Gaps”
2. Agent processes `paper_summary.json` files
3. Suggests topics with rationales and severity/opportunity scores

---

## UC-PR06: Contextual Q&A Chat

### Actors
- Researcher
- QA Agent

### Goal
Ask questions and get concise, context-grounded answers from selected papers.

### Main Flow
1. Researcher selects docs + asks question
2. Agent returns targeted answer
3. Frontend shows chat thread

---

## UC-PR07: Citation Path Discovery (Optional)

### Actors
- Researcher

### Goal
Visualize how selected papers cite each other or share references.

### Main Flow
1. Researcher clicks “View Citation Graph”
2. System builds network graph from citation metadata
3. User explores paths and clusters