# Telecom Job Agent — n8n V0

A lightweight personal job-search assistant for one candidate. V0 focuses on the most important thing first: can the agent learn what a candidate considers a good or bad job and rank unseen jobs accordingly?

## V0 scope

- Candidate profile stored as simple JSON
- Positive and negative reference jobs
- Score a pasted job description from 0–100
- Explain strong matches, gaps, and red flags
- Store results in Google Sheets (setup schema included)
- Optional daily Telegram digest workflow

Not included yet: LinkedIn automation, scraping, recruiter enrichment, automated applications, complex vector databases, or multi-agent orchestration.

No real resume/CV is required for this V0. The included request uses synthetic sample data only.

## Architecture

```text
Resume/profile + preferences
            |
            v
   Positive/negative examples
            |
            v
       n8n workflow
            |
            v
 Google Gemini API
            |
            v
 score + explanation + verdict
            |
       +----+----+
       |         |
       v         v
 Google Sheet  Telegram digest
```

## Model

V0 defaults to `gemini-2.5-flash-lite` through the Gemini Developer API. It is a stable, fast model that supports structured JSON output and currently has free input/output token usage on Google's free developer tier.

The workflow sends a JSON schema with every scoring request, so downstream n8n nodes receive a predictable object rather than free-form model text.

The model is configurable with `GEMINI_MODEL` in `.env`.

> Privacy note: Google's Gemini Developer API free tier may use submitted content to improve Google products. V0 therefore uses synthetic candidate data only. Before processing a real CV/profile, review the current data-use terms and decide whether to move the project to a paid tier or another provider.

## Quick start

### 1. Run n8n locally

Install Docker Desktop, then from this folder run:

```bash
docker compose up -d
```

Open http://localhost:5678 and create the local n8n owner account.

### 2. Create a Gemini API key

Create an API key in Google AI Studio / Gemini Developer API, then copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Edit `.env`:

```text
GEMINI_API_KEY=your_real_key_here
GEMINI_MODEL=gemini-2.5-flash-lite
```

`.env` is gitignored. Never put the real API key in `.env.example`, workflow JSON, README, or GitHub.

Restart n8n after changing `.env`:

```bash
docker compose down
docker compose up -d
```

### 3. Import the workflow

In n8n:

1. Workflows → Import from File
2. Import `workflows/01-score-job-webhook.json`
3. Activate the workflow
4. Copy the production webhook URL

### 4. Test scoring

POST JSON like this to the webhook:

```json
{
  "candidate_profile": {
    "current_role": "Telecom Network Engineer",
    "years_experience": 6,
    "skills": ["LTE", "5G", "RAN", "RF Optimization", "Ericsson"],
    "preferred_locations": ["Bangalore", "Gurgaon", "Noida"],
    "preferred_roles": ["RAN Engineer", "RF Optimization Engineer", "5G Network Engineer"],
    "avoid_roles": ["L1 Support", "Field Technician", "Project Manager"]
  },
  "positive_examples": [
    {
      "title": "RAN Optimization Engineer",
      "reason": "Strong 4G/5G optimization and KPI analysis work"
    }
  ],
  "negative_examples": [
    {
      "title": "Network Support Engineer",
      "reason": "Mostly L1/L2 ticket support"
    }
  ],
  "job": {
    "title": "Senior RAN Optimization Engineer",
    "company": "Example Telecom",
    "location": "Gurgaon",
    "description": "5G/LTE RAN optimization, KPI analysis, Ericsson OSS, drive test analysis...",
    "url": "https://example.com/jobs/123"
  }
}
```

The response is structured JSON containing the match score, verdict, reasons, gaps, red flags, model version, and Gemini usage metadata.

You can also test with the included sample:

```bash
curl -X POST 'YOUR_N8N_WEBHOOK_URL' \
  -H 'Content-Type: application/json' \
  --data @samples/request.json
```

## Why Gemini for V0

For this one-person prototype, Gemini 2.5 Flash-Lite keeps model inference cost at zero while we validate whether the ranking logic is actually useful. The candidate/job data model is provider-independent, so switching models later does not require redesigning the application.

Do not commit real job inputs, personal profile data, or any future real resume to this repository; submit them to the running workflow at runtime.

## Suggested V0 validation

Before automating job discovery, give the workflow 20–30 job descriptions the candidate has never rated. Ask him to rate the top 10 independently.

A good signal to continue is if roughly 8 of the agent's top 10 are jobs he genuinely considers relevant.

## Next milestones

- V0.1 — Store scored jobs in Google Sheets
- V0.2 — Telegram bot for 👍 / 👎 / APPLIED feedback
- V0.3 — Daily digest
- V1 — Automated job discovery from selected company career pages / ATS feeds
- V1.1 — Learn from feedback automatically
- Later — resume tailoring and referral/recruiter research
