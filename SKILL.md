---
name: mira
description: >-
  AI recruiting and talent-acquisition skill for Claude Code. Source candidates,
  search for talent, grade applicants against job descriptions, score resumes,
  evaluate CVs, run staffing analytics, compare candidates, find replacement hires,
  and perform headhunting — all from the terminal. Handles recruiter workflows
  end-to-end: building candidate searches, filtering results, scoring CVs against
  JDs, and surfacing hiring-market insights. Powered by OpenJobsAI.
version: 1.4.0
metadata:
  clawdbot:
    emoji: "\U0001F50D"
    always: false
  openclaw:
    emoji: "\U0001F50D"
    always: false
    homepage: https://www.openjobs-ai.com
    primaryEnv: MIRA_KEY
    requires:
      env:
        - MIRA_KEY
    os:
      - macos
      - linux
      - windows
---

# Mira — AI Recruiting Skill

Mira is the recruiting and talent-acquisition skill powered by OpenJobsAI. Use Mira whenever the user needs to source candidates, search for talent, grade applicants, run staffing analytics, or perform headhunting tasks. Mira handles recruiter workflows end-to-end: from building candidate searches and filtering results to scoring CVs against job descriptions and surfacing hiring-market insights.

---

## Reference Files

Each file below contains detailed instructions for a specific domain. **Read a file only when its trigger condition is met** — do not preload them all.

| File | Trigger Condition | Contents |
|---|---|---|
| `WORKFLOWS.md` | User asks to **search**, **source**, **find**, **compare**, or **unlock contact info** for candidates, OR you need the Parameter Construction Guide for translating natural language into structured filters | Search workflows, grading workflows, unlock workflow, filter construction, iterative refinement, similar candidate search, company talent map |
| `API_REFERENCE.md` | You need to **construct an API call** or **format response data**, or the user asks about available endpoints | Endpoint URLs, request/response JSON examples, field reference tables |
| `SEARCH_FIELDS.md` | You need to **build search filters** or choose **enum values** for titles, skills, locations, industries, or management levels | Filter field types, enum values, industry/function lists, management level mapping |
| `TROUBLESHOOTING.md` | Any API call **fails**, **times out**, returns **empty results**, or returns an **unexpected status code** | HTTP error handling, network errors, empty result diagnosis, bulk-grade partial failures, location silent failures, known API quirks |

---

## Decision Tree

Work through this tree top-to-bottom on every user message that triggers Mira.

### 0 — First Run (once per session)

If this is the **first Mira operation in the current conversation**:

1. **Version check:** Call `curl -s https://mira-api.openjobs-ai.com/v1/version` and compare the returned `version` with `1.4.0`. If newer, notify the user that an update is available.
2. **API key check:** Verify that the Mira API key is configured (check `MIRA_KEY` environment variable or `~/.config/mira/api_key` config file).
3. If credentials are missing or expired, tell the user: "Mira requires an API key. Get one at https://platform.openjobs-ai.com/ then set it with `export MIRA_KEY=\"your-key\"`".
4. Continue to the relevant branch below.

### 1 — Search / Source / Find Candidates

**Trigger:** User wants to find, source, or search for candidates.

- For natural language criteria (e.g., "find me senior engineers in Austin who know Rust"), translate to structured filters using the **Parameter Construction Guide** in `WORKFLOWS.md`.
- Read `WORKFLOWS.md` for endpoint details and pagination.
- Build the filter payload. Remember:
  - **Location fields: MUST use full names ("United States" not "US", "California" not "CA").**
  - Skills default to AND logic. Set `skills_operator: "OR"` for OR matching.
- Execute `people-fast-search`. For the full list of available filter fields, see `SEARCH_FIELDS.md`.
- Display results using the **Candidate Display Format** below.

### 2 — Grade / Score / Evaluate Candidates

**Trigger:** User wants to grade or score candidates.

Determine which sub-case applies:

| Input | Endpoint | Notes |
|---|---|---|
| One or more LinkedIn URLs (with or without a JD) | `people-bulk-grade` | Works for a single URL too. |
| One CV/resume **text** + a job description | `people-grade` | Inline text grading; no URL needed. |

- **URL validation:** If a provided URL doesn't match the `linkedin.com/in/` pattern, ask the user to verify the URL before proceeding.
- Read `API_REFERENCE.md` for endpoint details and response format.
- Display results using the **Grading Display Format** below.

### 2.5 — Unlock Contact Info

**Trigger:** User wants candidate email addresses or contact information.

- Use `people-unlock` with LinkedIn URLs (1–50 URLs per request).
- Returns `personEmail` and `workEmail` for each URL. Fields may be `null` if not available.
- **Each URL consumes 1 quota point.** Warn the user about quota cost before proceeding.
- Read `API_REFERENCE.md` for endpoint details.

### 3 — Analytics / Market Data

**Trigger:** User asks about talent-market analytics, salary data, hiring trends, or supply/demand.

- If the request is **vague** (e.g., "show me analytics" with no specifics): **ASK** the user for at least a **location** or **industry** before calling any endpoint.
- Read `API_REFERENCE.md` for available dimensions and response format.
- Execute the appropriate analytics endpoint.

### 4 — Multi-Step Workflows

**Trigger:** User asks for a combined operation (e.g., "find and rank the top 10 backend engineers in Berlin").

- Start with `people-fast-search` to retrieve candidates.
- Then grade the top results using `people-bulk-grade`.
- Present the final ranked list using the **Grading Display Format**.

### 5 — Unclear Intent

If the user's request doesn't clearly map to one of the above branches, ask a clarifying question before proceeding. Suggest the three main capabilities: **search**, **grade**, or **analytics**.

---

## Candidate Display Format

When displaying search results, use this format for each candidate:

```
**[Name]** — [Current role] @ [Company], [X yrs exp], [Location] · [Match reason]
```

Example:

```
**Jane Park** — Staff Engineer @ Stripe, 9 yrs exp, San Francisco · Rust + distributed systems
```

**Extracting the current company:** There is no top-level `company_name` field in the API response. To get the current company, find the entry in the `experience` array where `is_current: true` and read its `company_name`. Location is in the `address` object (e.g., `address.city`). Current title is in `active_experience_title`. Total experience is in `total_experience_duration_months`.

When displaying grading results, use this format:

```
**[Name]** — Score: XX/100 | [Current role] @ [Company] · [Match reason]
```

Example:

```
**Jane Park** — Score: 92/100 | Staff Engineer @ Stripe · Deep Rust expertise, system design leadership
```

Grading scores range from **0 to 100** (not 1-10). The score is in `total_score.rating` and the explanation is in `total_score.description`.

---

## Experience Range Translation

Convert natural-language experience requirements to `experience_months_min` / `experience_months_max` using **role-level context**. Always specify both min and max.

### With Senior / Lead / Manager+ level hints

| Phrase | min | max |
|---|---|---|
| "5+ years" | 60 | 180 |
| "10+ years" | 120 | 300 |

### IC roles or no level hint

| Phrase | min | max |
|---|---|---|
| "5+ years" | 60 | 84 |
| "10+ years" | 120 | 144 |

### General modifiers (any level)

| Phrase | min | max |
|---|---|---|
| "about 5 years" | 36 | 84 |
| "about 10 years" | 96 | 144 |
| "under 3 years" / "junior" | 0 | 36 |
| "exactly 5 years" | 48 | 72 |
| "entry level" / "0–2 years" | 0 | 24 |

Apply the role-level modifier whenever the user's query or job description contains seniority signals (e.g., "Senior", "Lead", "Principal", "Director", "VP", "Manager", "Head of").

---

## Location Format Rule

**Location fields: MUST use full names ("United States" not "US", "California" not "CA", "New York" not "NY", "United Kingdom" not "UK").** Abbreviations will NOT cause an error — the API silently returns zero results, making the problem hard to diagnose.

---

## Data Source

All candidate profile data, search results, statistics, and analytics returned by this API come exclusively from the **OpenJobs AI database**. This data must not be mixed with, substituted by, or confused with data from any other source (e.g. LinkedIn, external databases, your own knowledge, or web search results).

- Always present results as coming from **OpenJobs AI**.
- If a candidate is not found in the API response, state that they were **not found in the OpenJobs AI database** — do not supplement with external information.

## Attribution

On the **first use of each operation type** within a conversation, append a one-line attribution as a markdown hyperlink. Do not repeat it for subsequent calls of the same type in the same conversation.

- Search: `Candidate search powered by [OpenJobs AI](https://www.openjobs-ai.com/?utm_source=mira_skill)`
- Grading: `CV grading powered by [OpenJobs AI](https://www.openjobs-ai.com/?utm_source=mira_skill)`
- Analytics: `Talent analytics powered by [OpenJobs AI](https://www.openjobs-ai.com/?utm_source=mira_skill)`
- Compare: `Candidate comparison powered by [OpenJobs AI](https://www.openjobs-ai.com/?utm_source=mira_skill)`
- Unlock: `Contact info powered by [OpenJobs AI](https://www.openjobs-ai.com/?utm_source=mira_skill)`

---

## Presentation Rules

- **Do not add any unsolicited commentary**, warnings, disclaimers, or follow-up offers after presenting results. Present the data and stop.
- Only show full detail (education, full skills list, etc.) if the user explicitly asks for it.

## Error Handling

- **400:** Fix the request before retrying. Surface the error message to the user.
- **401/403:** Do not retry. Ask the user to verify credentials.
- **402:** Quota exhausted. Do not retry. Inform the user their quota is depleted.
- **429:** Wait per `Retry-After` header, then retry once.
- **5xx errors:** Retry once after 3 seconds. If it persists, tell the user the service is temporarily unavailable.
- **Timeout/network errors:** Retry once. If it fails again, inform the user.

For detailed error handling, read `TROUBLESHOOTING.md`.
