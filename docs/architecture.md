# Architecture

## Components
- **Orchestrator** — routes a user's question + files to relevant pipelines.
- **Text Q&A Flow** — extracts answers from historical text; returns confidence.
- **Document Analysis Flow** — processes PDFs/Images; OCR + tamper checks.

## Data Flow (overview)
1. Ingest question and optional files.
2. Select flow(s) based on inputs.
3. Aggregate evidence; compute scores; return final JSON response.
