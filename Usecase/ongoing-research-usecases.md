# ScholarAI Ongoing Research Use Cases

## UC-OR01: Topic Suggestion to Writing Template

### Actors
- Researcher
- Topic Agent

### Goal
Convert selected research topics into structured writing templates.

### Main Flow
1. Researcher selects a topic from suggestions or enters custom topic.
2. System generates a template with section headings and brief guides:
   - Introduction
   - Related Work
   - Methodology
   - Experiments
   - Results
   - Conclusion
3. Template is stored and editable in the project workspace.

---

## UC-OR02: AI-Powered LaTeX Editor

### Actors
- Researcher

### Goal
Write papers using a LaTeX editor with AI-assisted utilities and live preview.

### Main Flow
1. Researcher opens the writing workspace.
2. LaTeX editor with live preview loads.
3. AI utilities include:
   - Inline suggestions and improvements
   - Refactoring (rewrite section for clarity)
   - Insertion of boilerplate sections
4. System autosaves progress.

---

## UC-OR03: Inline Citation & Auto-cite Missing References

### Actors
- Researcher
- Citation Agent

### Goal
Cite papers directly while writing, and auto-suggest missing citations.

### Main Flow
1. Researcher selects text and clicks “Cite”
2. System recommends papers from library based on semantic match
3. Researcher clicks to insert citation
4. Optionally, “Scan for Missing Citations” suggests additions

---

## UC-OR04: AI Paper Review with Rule Enforcement

### Actors
- Researcher
- Review Agent

### Goal
Automatically review written paper for quality, grammar, and conference/journal rule compliance.

### Main Flow
1. Researcher clicks “AI Review”
2. System checks:
   - Grammar & clarity
   - Section completeness
   - Figure/table captions
   - Citation format
   - Compliance with selected ruleset (e.g., IEEE)
3. Feedback is shown inline or as a report

---

## UC-OR05: AI Corrections (Optional)

### Actors
- Researcher

### Goal
Automatically apply AI suggestions to fix issues found in review.

### Main Flow
1. Researcher clicks “Apply Suggestions”
2. System updates LaTeX document with inline diffs shown