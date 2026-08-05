# Paper Brief AI

Build a research paper analyzer web app with these requirements:

PURPOSE

Frontend for a multi-agent AI system that analyzes academic papers. User submits a paper (PDF upload, URL, or pasted text), backend (n8n webhook) processes it through multiple AI agents, returns a structured research brief.

INPUT SCREEN

Three input modes, shown as tabs or toggle buttons:

1. "Upload PDF"

   - Drag-and-drop zone + click-to-browse, accept only .pdf files, max 20MB

   - On file select: read file as base64 using FileReader.readAsDataURL, strip the "data:application/pdf;base64," prefix, keep only the base64 string

   - Show file name, size, and a remove/replace button after selection

   - On submit: POST body = { "pdf_base64": "<base64 string>" }

2. "Paper URL"

   - Text field for an arXiv/PDF URL

   - On submit: POST body = { "pdf_url": "<value>" }

3. "Paste Text"

   - Textarea for pasting paper text directly

   - On submit: POST body = { "paper_text": "<value>" }

All three POST to: https://arjunn1729.app.n8n.cloud/webhook/research-paper-analyzer

Submit button label: "Analyze Paper" — disabled until valid input given for the active tab.

LOADING / PROGRESS STATE

After submit, show a progress view simulating the agent pipeline (backend doesn't stream progress, so animate through these labels with a few seconds delay each, or show a spinner with rotating status text):

1. "Analyzing methodology and findings..."

2. "Reviewing analysis quality..."

3. "Generating executive summary..."

4. "Extracting citations..."

5. "Combining final research brief..."

For PDF uploads specifically, show an upload progress bar first (file → base64 conversion can take a moment for large files) before the analysis stages begin.

RESULTS SCREEN

Display the JSON response in a structured, readable layout:

1. Research Analysis card — problem_statement, methodology, experiments, findings (from research_analysis object)

2. Executive Summary card — executive_summary (single paragraph, 150-200 words)

3. Citations & References card — list from citations array

4. Quality Scores panel — quality_scores.analysis, quality_scores.summary, quality_scores.citations as scores out of 10 (visual badges/progress bars), plus retries.analysis, retries.summary, retries.citations showing review iterations per agent — demonstrates the quality-control loop transparency

DESIGN

Clean, academic/research tool aesthetic — Notion or Linear style, not flashy. Card-based layout, generous whitespace, serif font for paper content (readability), sans-serif UI chrome. Handle loading/error states gracefully (webhook timeout, invalid URL, corrupt PDF, file too large) with clear error messages and a retry button.

TECH

React + Tailwind. No backend needed — pure frontend calling the external n8n webhook directly via fetch/axios. Handle large base64 payloads without blocking the UI thread (show progress during FileReader conversion).

This project was built with [Lovable](https://lovable.dev).

**Live app**: https://research-paper-analyzeragent.lovable.app

## Build with Lovable

Continue developing this project in the [Lovable editor](https://lovable.dev/projects/4dcb32f5-aff5-41b8-a8f8-fcfa0f43bdaf).

- **Ship faster**: describe what you want to build and Lovable handles the code.
- **Stay in sync**: every change made in Lovable is committed straight to this repository.
- **Full ownership**: this code is yours. Push to `main` on GitHub and your changes sync back into Lovable, ready for your next prompt.

## Development

Prefer working locally? You need Node.js and npm — [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating).

```sh
git clone <this-repository-url>
cd <repository-name>
npm i
npm run dev
```
