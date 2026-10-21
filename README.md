# Candidate Screening Orchestrator

A multi-agent recruitment workflow built in n8n. A supervisor agent routes each candidate to a department, a skill-match agent scores the resume against the job, and a human approves shortlisted candidates on Telegram before any interview invite is sent.

## Architecture

```
Webhook → Supervisor Agent → Parse Route → Route Candidate (Switch)
   → Skill Match Agent → Parse Match Score → Check Score
        ├── Score < 70  → Save Rejected Candidate → Rejected Response
        └── Score ≥ 70  → Save Pending Candidate → Telegram Approval
                              ├── Approved → Send Interview Email → Mark Interview Invited → Approved Response
                              └── Rejected → Mark Not Selected → Not Selected Response
```

## Features

- **Supervisor agent** routes candidates to `technical`, `sales`, or `general` based on job title
- **Skill-match agent** scores each resume from 0 to 100 with a one-line reason
- **Automatic rejection** for scores below 70, with the record saved to Supabase
- **Human-in-the-loop approval** on Telegram for shortlisted candidates
- **Pending-first saving:** candidates are saved as `SHORTLIST_PENDING` before approval, so no data is lost if the workflow stops while waiting
- **Status tracking** in Supabase: `REJECTED`, `SHORTLIST_PENDING`, `INTERVIEW_INVITED`, `NOT_SELECTED`
- **Webhook API** returns a JSON response for every path

## Tech Stack

- n8n (workflow automation)
- OpenAI `gpt-5-mini` (supervisor and skill-match agents)
- Gmail API (interview invitations)
- Supabase (candidate records and status tracking)
- Telegram Bot API (approval requests)

## Setup

1. Import the workflow JSON into n8n.
2. Add credentials: OpenAI, Gmail OAuth2, Supabase, and Telegram.
3. Create the `candidates` table in Supabase:

```sql
create table candidates (
  id bigint generated always as identity primary key,
  name text,
  email text,
  job_title text,
  route text,
  match_score int,
  match_reason text,
  status text,
  created_at timestamptz default now()
);
```

4. Replace the Telegram `chatId` with your own chat ID.
5. Activate the workflow and send a POST request to the webhook.

## Sample Request

```json
{
  "name": "Ayesha Khan",
  "email": "ayesha.test@example.com",
  "job_title": "Sales Development Representative",
  "resume_text": "3 years of B2B sales experience. Managed outbound cold calling and email campaigns. Used HubSpot CRM and Google Sheets to track pipeline. Closed 25 deals in 2025. Strong communication skills in English and Urdu."
}
```

## Project Notes

- Built as part of my AI Automation portfolio.
- Telegram messages use underscore-free text (`.replace(/_/g, ' ')`) to avoid Markdown parsing errors.
