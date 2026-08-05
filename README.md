# Vilambo Research Paper Analyzer

A multi-agent AI system that analyzes academic research papers automatically —
extracting methodology, generating summaries, organizing citations, and
running everything through an automated quality-control review loop before
combining it all into a final research brief. Includes a bonus web UI.

Built for the Vilambo Private Limited AI Agent Developer Intern technical
assignment.

**Live UI:** https://research-paper-analyzeragent.lovable.app
**Demo video:** https://drive.google.com/file/d/1GJv1XUAdNjZkwwqLjBOkGelAhFiDw4DD/view?usp=drive_link

---

## Repo structure

```
/                     — n8n multi-agent workflow + docs (this README)
  workflow.json       — importable n8n workflow export
  sample-input.txt    — example arXiv PDF URL used for testing
  sample-output.json  — full research brief returned for that paper
  AI AGENT WORKFLOW.png — architecture diagram
/ui                   — React + Tailwind frontend (built with Lovable),
                         calls the n8n webhook directly from the browser
```

## Architecture

```
Input (PDF URL or pasted text)
        |
   Webhook Trigger
        |
   Init State (Boss Agent — orchestrator, sets shared workflow state)
        |
   Has PDF URL? ──yes──> Download PDF ──> Extract PDF Text ──┐
        |no                                                   |
        └───────────────────────────────────────────────────┘
        |
   Build Analyzer Prompt
        |
   Gemini — Paper Analyzer  ──>  Gemini — Review Agent
        |                              |
        └───── score < 7 & retries < 2 (loop back) ──┘
        | score >= 7 (approved)
        v
   Build Summary Prompt
        |
   Gemini — Summary Generator  ──>  Gemini — Review Agent
        |                                  |
        └───── score < 7 & retries < 2 (loop back) ──┘
        | score >= 7 (approved)
        v
   Build Citation Prompt
        |
   Gemini — Citation Extractor  ──>  Gemini — Review Agent
        |                                    |
        └───── score < 7 & retries < 2 (loop back) ──┘
        | score >= 7 (approved)
        v
   Combine Final Brief (Boss Agent — merges all approved outputs)
        |
   Respond to Webhook (returns complete research brief JSON)
```

**Agent roles:**
- **Boss Agent (Init State + Combine Final Brief):** orchestrates the whole
  run, tracks shared state (paper text, retry counters, scores), and
  assembles the final combined output.
- **Paper Analyzer:** extracts problem statement, methodology, experiments,
  and findings.
- **Summary Generator:** writes a 150–200 word executive summary from the
  approved analysis.
- **Citation Extractor:** pulls formatted references from the source text.
- **Review Agent:** scores every sub-agent output 1–10 with feedback;
  anything below 7 is regenerated with that feedback appended, capped at 2
  retries per agent to avoid infinite loops.

## Tech stack

- **Orchestration:** n8n (workflow automation, state-based multi-agent flow)
- **LLM:** Google Gemini API (`gemini-3.5-flash-lite`), structured JSON
  output enforced via `responseSchema`
- **PDF processing:** n8n's built-in Extract From File node
- **UI:** React + Tailwind (`/ui`), built with Lovable, calls the n8n
  webhook directly from the browser — supports PDF upload, paper URL, and
  pasted text as input

## Setup instructions — agent (n8n)

### 1. Import the workflow
- Open n8n → Workflows → Import from File → select `workflow.json`

### 2. Add your Gemini API credential
- Get a free key: [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
- In n8n: Credentials → New → Google Gemini(PaLM) Api → paste key → Save
- Attach this credential on all 6 Gemini nodes (Analyzer, Review Analyzer,
  Summary Generator, Review Summary, Citation Extractor, Review Citation)

### 3. Activate the workflow
- Toggle "Active"/"Published" in the top right of the n8n canvas
- Copy the **production** Webhook URL from the Webhook node

### 4. Test it
```bash
curl -X POST https://YOUR-N8N-URL/webhook/research-paper-analyzer \
  -H "Content-Type: application/json" \
  -d "{\"pdf_url\": \"https://arxiv.org/pdf/2608.03495"}"
```

Or with pasted text instead of a PDF URL:
```json
{"paper_text": "your paper text here"}
```

If you see CORS errors calling the webhook from a browser, add a Response
Header on the Webhook node: `Access-Control-Allow-Origin: *`.

## Setup instructions — UI (`/ui`)

The UI is a React + Tailwind app (built with Lovable) with three input
modes — PDF upload, paper URL, or pasted text — and a results screen
showing the research analysis, executive summary, citations, and per-agent
quality scores/retry counts for transparency into the review loop.

To run it yourself:

```sh
cd ui
npm i
npm run dev
```

Update the webhook URL constant in the app to point at your own n8n
production URL before running, or use the hosted version linked above.

## Sample input / output

- `sample-input.txt` — arXiv PDF URL used for testing
- `sample-output.json` — full research brief returned by the system for
  that paper

## Known limitations

- Sub-agents run **sequentially**, not in parallel — simpler to debug and
  demo, at the cost of total runtime (roughly 30–90 seconds per paper
  depending on retries).
- Key Insights agent (bonus 4th sub-agent) is not implemented — analysis,
  summary, and citations only.
- Citation quality depends on the source paper's reference formatting;
  papers with non-standard reference sections may need the Citation
  Extractor prompt tuned further.
- PDF extraction handles standard text-based PDFs (e.g. arXiv); scanned or
  image-only PDFs are not supported without adding OCR.
- Review Agent occasionally exhausts its 2-retry cap without reaching the
  score-7 threshold (e.g. a summary scoring 6/10) — the workflow falls back
  to the best available attempt rather than blocking the pipeline
  indefinitely.

## Environment variables

See `.env.example`. The n8n workflow's LLM calls are configured via n8n
credentials rather than a `.env` file directly, but the Gemini API key
should never be hardcoded in the workflow JSON or committed to this repo.
