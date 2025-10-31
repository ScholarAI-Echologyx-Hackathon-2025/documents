# ScholarAI Post-Research Use Cases

## UC-PO01: Impact Tracking & Metrics Dashboard

### Actors
- Researcher
- System
- External APIs (Semantic Scholar, Crossref, Altmetrics)

### Goal
Track performance of published research over time.

### Main Flow
1. Researcher opens the “Impact Dashboard”.
2. System fetches and displays:
   - Citation counts (Semantic Scholar / Crossref)
   - Downloads (if available)
   - Mentions or social media shares (Altmetrics)
3. Graph shows trends over time (monthly/quarterly).
4. Researcher can filter by project or paper.

---

## UC-PO02: Citation Alert Notifications

### Actors
- Researcher
- Notification Service

### Goal
Notify researchers when one of their papers is cited.

### Main Flow
1. System runs daily/weekly check via APIs.
2. If a new citation is detected:
   - Sends in-site notification
   - Sends email (if enabled)
3. Notification includes:
   - Citing paper title
   - Link to source
   - Citation context (if available)

---

## UC-PO03: Download & Share Paper Metrics Report

### Actors
- Researcher

### Goal
Download a shareable summary of a paper or project’s performance.

### Main Flow
1. Researcher clicks “Export Metrics” on a paper/project.
2. System generates a PDF/CSV report with:
   - Graphs
   - Tables of citation, download, and view data
   - Summary of trends and top sources
3. File is downloaded or shared via email

---

## UC-PO04: Visual Timeline of Impact

### Actors
- Researcher

### Goal
Visualize the lifecycle of paper impact across platforms.

### Main Flow
1. System aggregates metrics by time and source.
2. Timeline is plotted with key milestones:
   - Published
   - Cited first time
   - Trending on Twitter
   - Mentioned in blog
3. Timeline allows drill-down on events