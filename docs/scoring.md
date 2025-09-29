# Scoring & Final Output

We compute:
- **AnswerScore** — accuracy/consistency of the produced answer.
- **TamperScore** — likelihood of manipulation in supplied documents.
- **SupportScore** — strength of evidence supporting the answer.

**Final JSON (example fields):**
`answer`, `confidence`, `answer_score`, `tamper_score`, `support_score`, `evidence[]`, `diagnostics{}`
