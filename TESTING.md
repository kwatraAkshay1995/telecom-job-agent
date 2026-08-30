# Testing V0

## Minimal test without Telegram or Google Sheets

Once the scoring workflow is active, copy its production webhook URL and run:

```bash
curl -X POST 'YOUR_WEBHOOK_URL' \
  -H 'Content-Type: application/json' \
  --data @samples/request.json
```

Expected shape:

```json
{
  "score": 0,
  "verdict": "EXCELLENT|STRONG|POSSIBLE|WEAK|REJECT",
  "matching_reasons": [],
  "gaps": [],
  "red_flags": [],
  "inferred_role_family": "...",
  "recommended_action": "APPLY|REVIEW|SKIP",
  "scored_at": "..."
}
```

## Validation set

Use about 20–30 real jobs. Do not tell the candidate the model score before they independently label each job.

Record:
- Candidate verdict: GOOD / MAYBE / BAD
- Agent score
- Agent action
- Where they disagree

The main metric for V0 is precision in the top 10: how many of the agent's ten highest-ranked jobs the candidate actually likes.

Target before automating discovery: around 8/10 or better.
