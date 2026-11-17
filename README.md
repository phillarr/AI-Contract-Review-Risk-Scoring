# AI Contract Review + Risk Scoring

A scenario that picks up new contracts dropped in a Google Drive folder, reads them with OCR, asks an AI agent (Claude) to review each clause and score its risk, then writes a risk report in Google Docs from a template.

## The scenario

```
Google Drive (new file) ──► Get file ──► Google Cloud Vision (OCR) ──► AI agent (review + risk scoring) ──► Parse JSON ──► Google Docs (report from template)
```

## What it does

1. **Watch files** (Google Drive): checks a folder for newly created files.
2. **Get the file** (Google Drive): downloads the contract.
3. **Detect text** (Google Cloud Vision): extracts the text of the contract with OCR.
4. **Review the contract** (Make AI agent, Claude Sonnet): sends the text to the agent with a system prompt (see [`prompt.md`](prompt.md)). The agent returns a JSON object with:
   - `title`, `contract_id`, `parties`, `effective_date`, `expiration_date` (`"Not present"` when the contract does not state them)
   - `clauses`: each clause found (Termination, Auto-renewal, Payment terms, Liability, Indemnification, Confidentiality, Governing law, Unusual clauses) with a `finding`, a `risk_level` (`Low`, `Medium`, `High`) and a `recommendation`
   - `overall_risk_score`: `Low`, `Medium` or `High`, based on all findings together
   - `executive_summary`: 2 to 3 sentences on the main risk areas
   - `clauses_text`: the same clauses as plain text, ready to drop into a document
5. **Parse JSON**: removes any stray markdown fences and turns the reply into structured data.
6. **Create a document from a template** (Google Docs): fills a report template and saves it in an output folder, named after the contract title.

> **The AI can make mistakes.** It can miss a clause, misread the OCR text, or rate a risk too high or too low. This report is not legal advice: always read the contract itself and have important contracts reviewed by a qualified lawyer before signing.

## Requirements

- A Make.com account with access to the **AI agent** module
- A Google account (Drive + Docs)
- A Google Cloud project with the **Cloud Vision API** enabled

## Google Docs template setup

Create a Google Doc to use as the report template, with these placeholders (Make replaces each `{{...}}` tag):

| Placeholder              | Filled with                         |
|--------------------------|-------------------------------------|
| `{{title}}`              | Contract title                      |
| `{{contract_id}}`        | Contract ID                         |
| `{{parties}}`            | Party names, comma-separated        |
| `{{effective_date}}`     | Effective date                      |
| `{{expiration_date}}`    | Expiration date                     |
| `{{overall_risk_score}}` | Overall risk score                  |
| `{{executive_summary}}`  | Executive summary                   |
| `{{clauses}}`            | All clauses with risk and advice    |

You also need two Drive folders: one where you drop contracts, one where the reports are saved.

## Installation

1. Download [`AI Contract Review + Risk Scoring.blueprint.json`](AI%20Contract%20Review%20%2B%20Risk%20Scoring.blueprint.json).
2. In Make, create a new scenario, open the **⋯** menu, choose **Import Blueprint** and select the file.
3. Reconnect each module to your own accounts:
   - **Google Drive** and **Google Docs**: add your Google connection, then pick **your** input folder, template and output folder.
   - **Google Cloud Vision**: add your connection.
   - **AI agent**: check the model and connection available in your Make account.
4. In the **Parse JSON** module, rebuild the data structure if Make asks for it.
5. Click **Run once** and drop a test contract in the input folder.
6. Turn on scheduling when you are happy with the result.

## Customizing

- **Clause categories and risk rules**: edit them in the agent's system prompt. [`prompt.md`](prompt.md) has a readable copy.
- **Negotiation playbook**: the prompt uses a negotiation playbook from the agent's knowledge if one is available. Add your own thresholds (for example, maximum liability cap or payment terms) to make the risk scores match your company's standards.
- **Output**: send the report by email or post a summary in a chat tool when the overall risk is `High`.

## Privacy and security

- **Contract contents are sent to Google Cloud Vision and to the AI provider** for processing. Only use this with documents you are allowed to share with these services.
- **API keys and OAuth tokens stay in your Make connections** and are never exported in the blueprint.
- **Contracts are untrusted input.** A document could contain text designed to trick the AI (prompt injection) and lower a risk score. Never rely on the score alone.
