# Screenshots

Diagrams in this repository are generated from each workflow's real canvas coordinates, so the
architecture is already documented. What screenshots add is evidence the pipeline actually ran.

Drop files here and link them from the relevant page. In rough order of value:

| File | Shows | Why it matters |
|---|---|---|
| `execution-history.png` | n8n executions list, successful batch runs | Proves real execution against the full ~5,600-lead dataset |
| `crm-sheet.png` | The live CRM with real rows | Structured output of the whole pipeline |
| `integrity-check-output.png` | A clean integrity-check run (delta 0, missing 0) | The verification discipline described in `engineering/build-log.md`, shown rather than claimed |
| `instantly-campaign.png` | Leads landed in an Instantly campaign via the direct push | The outreach connection actually working end to end |
| `child-canvas.png` | The enrichment child workflow in the n8n editor | 42 nodes, the largest workflow in this repository |

## Redact before adding

- Company names, phone numbers, email addresses. CRM rows and execution logs are full of them
- The n8n instance URL, visible in the address bar on every editor screenshot
- Webhook URLs, visible in webhook node panels
- API keys and credential names, visible in credential dropdowns
- Any client or commercial detail not already excluded from this repository's written docs

Blur or crop rather than relying on a small font. A 4K screenshot is readable when zoomed.
