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
 Anthropic Claude Haiku API
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

V0 defaults to `claude-haiku-4-5` through the Anthropic Messages API. The workflow forces Claude to return the assessment through a structured tool schema so the downstream n8n nodes receive predictable JSON.

The model is configurable with `ANTHROPIC_MODEL` in `.env`.

## Quick start

### 1. Run n8n locally

Install Docker Desktop, then from this folder run:

```bash
docker compose up -d
```

Open http://localhost:5678 and create the local n8n owner account.

### 2. Configure environment

Copy `.env.example` to `.env` and add your Anthropic API key.

```bash
cp .env.example .env
```

Then edit `.env`:

```text
ANTHROPIC_API_KEY=your_real_key_here
ANTHROPIC_MODEL=claude-haiku-4-5
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

The response is structured JSON containing the match score, verdict, reasons, gaps, red flags, model name, and Anthropic token usage.

You can also test with the included sample:

```bash
curl -X POST 'YOUR_N8N_WEBHOOK_URL' \
  -H 'Content-Type: application/json' \
  --data @samples/request.json
```

## About free model APIs

The workflow currently uses Anthropic because it is already configured for the project and is inexpensive at this scale. The architecture can later be adapted to Gemini or an Anthropic/OpenAI-compatible gateway without changing the candidate/job data model.

For real CV/profile data, review each provider's data-use policy before switching to a free tier. Free developer tiers can have different privacy terms from paid API usage.

## Suggested V0 validation

Before automating job discovery, give the workflow 20–30 real job descriptions your friend has never rated. Ask him to rate the top 10 independently.

A good signal to continue is if roughly 8 of the agent's top 10 are jobs he genuinely considers relevant.

Do not commit those job inputs or any future real resume to this repository; submit them to the running workflow at runtime.

## Next milestones

- V0.1 — Store scored jobs in Google Sheets
- V0.2 — Telegram bot for 👍 / 👎 / APPLIED feedback
- V0.3 — Daily digest
- V1 — Automated job discovery from selected company career pages / ATS feeds
- V1.1 — Learn from feedback automatically
- Later — resume tailoring and referral/recruiter research
