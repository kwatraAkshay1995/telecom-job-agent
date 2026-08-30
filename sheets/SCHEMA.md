# Google Sheets schema

Create one spreadsheet named `Telecom Job Agent` with these tabs.

## Profile

| key | value |
|---|---|
| current_role | Telecom Network Engineer |
| years_experience | 6 |
| preferred_locations | Bangalore,Gurgaon,Noida |
| preferred_roles | RAN Engineer,RF Optimization Engineer,5G Network Engineer |
| avoid_roles | L1 Support,Field Technician,Project Manager |
| skills | LTE,5G,RAN,RF Optimization,Ericsson |

## Examples

| id | verdict | title | company | description_or_notes | reason | created_at |
|---|---|---|---|---|---|---|
| 1 | POSITIVE | RAN Optimization Engineer | Nokia | ... | Good optimization role | ... |
| 2 | NEGATIVE | Network Support Engineer | Example | ... | Too support-heavy | ... |

## Jobs

| id | discovered_at | title | company | location | url | description | score | verdict | matching_reasons | gaps | red_flags | status | user_feedback |
|---|---|---|---|---|---|---|---:|---|---|---|---|---|---|

Suggested `status` values: NEW, SAVED, APPLIED, SKIPPED.
Suggested `user_feedback` values: LIKE, DISLIKE, NONE.
