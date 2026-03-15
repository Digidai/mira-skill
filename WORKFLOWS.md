# Mira — Workflows

This file contains all search, sourcing, and grading workflows, plus the Parameter Construction Guide for translating natural-language requests into structured API filters.

---

## Workflow 0: First Run / Onboarding

**Trigger:** This is the user's very first interaction with Mira, or the user asks "what can you do?" / "how do I get started?"

### Steps

1. **Version check.** Call `curl -s https://mira-api.openjobs-ai.com/v1/version` and compare with `1.4.0`. If newer, notify the user.
2. **Verify API credentials.** Check that the Mira API key is configured (`MIRA_KEY` environment variable or `~/.config/mira/api_key`). If not, tell the user to get a key at https://platform.openjobs-ai.com/ and set it with `export MIRA_KEY="your-key"`.
3. **Explain capabilities.** Briefly tell the user what Mira can do:
   - **Search** for candidates by title, skills, location, experience, company, and more.
   - **Grade** candidates against a job description (by LinkedIn URL or pasted CV text).
   - **Analytics** on talent pools, salary benchmarks, and hiring-market trends.
   - **Compare** candidates side-by-side.
   - **Unlock contact info** — get candidate email addresses.
   - **Find similar** candidates to a reference person (replacement hire).
   - **Map company talent** to understand an organization's workforce.
4. **Suggest a demo.** Offer a quick example the user can try right now:
   > "Want to try it out? Give me a job title and location (e.g., 'Find senior backend engineers in Berlin') and I'll run a search."
5. **Proceed.** Once credentials are confirmed and the user provides a task, continue to the relevant workflow below.

---

## Workflow 1: Basic Candidate Search

**Trigger:** User wants to find or source candidates using straightforward criteria.

### Steps

1. Parse the user's request and translate natural-language criteria into structured filters using the **Parameter Construction Guide** (see below).
2. Build the `people-fast-search` request payload. Remember:
   - **Location fields MUST use full names** ("United States" not "US", "California" not "CA").
   - All filter fields are optional; omit fields that the user did not specify.
3. Execute `people-fast-search`.
4. Display results using the **Candidate Display Format** from `SKILL.md`.
5. If fewer than 3 results are returned, suggest broadening filters (see Workflow 5).

---

## Workflow 2: Search + Grade (Multi-Step)

**Trigger:** User wants to find AND rank/score candidates in a single request (e.g., "find and rank the top 10 backend engineers in Berlin").

### Steps

1. Run `people-fast-search` with the user's criteria (same as Workflow 1, steps 1-3).
2. Collect the LinkedIn URLs from the search results.
3. Run `people-bulk-grade` with those URLs and the job description (the field is `jd`, not `job_description`).
   - If the user did not provide a JD, ask for one before grading. A JD is required for meaningful scoring.
4. Sort results by `total_score.rating` (highest first, scale is 0-100) and display using the **Grading Display Format** from `SKILL.md`.

---

## Workflow 3: Grade Candidates

**Trigger:** User provides candidate(s) and wants them graded/scored against a job description.

### Routing — Single Candidate

Determine which sub-case applies:

| Input | Endpoint | Notes |
|---|---|---|
| One CV/resume **text** + a job description | `people-grade` | Inline text grading; no URL needed. |
| One LinkedIn URL + a job description | `people-bulk-grade` | Works for a single URL too. |

This matches the decision tree in `SKILL.md` exactly: CV text goes to `people-grade`; LinkedIn URLs (one or many) go to `people-bulk-grade`.

### Routing — Multiple Candidates

| Input | Endpoint | Notes |
|---|---|---|
| Multiple LinkedIn URLs (with or without a JD) | `people-bulk-grade` | Batch grading up to the API limit. |

### Steps

1. Identify which inputs the user provided (CV text vs. LinkedIn URLs) and select the correct endpoint per the table above.
2. **URL validation:** If a provided URL does not match the `linkedin.com/in/` pattern, ask the user to verify before proceeding.
3. If no job description was provided, ask the user for one. A JD is required for grading.
4. Execute the appropriate endpoint.
5. Display results using the **Grading Display Format** from `SKILL.md`.

---

## Workflow 4: Candidate Lookup & Comparison

**Trigger:** User wants to look up a single candidate's profile, or compare two or more candidates side-by-side.

### Single Lookup

1. Use `people-lookup` with the candidate's LinkedIn URL (pass as `linkedin_urls` array, e.g., `{"linkedin_urls": ["https://..."]}`).
2. Extract the profile from `data.results[0]`. Present the full profile: experience, skills, education, certifications, languages. Note: current company is in the `experience` array entry where `is_current: true`, not a top-level field.

### Comparison

1. Collect two or more LinkedIn URLs from the user.
2. Use `people-compare` to get side-by-side profiles.
3. Present a comparison table highlighting differences in title, company, skills, education, and languages.

---

## Workflow 5: Search with Iterative Refinement (Default Pattern)

> **Almost every search requires iteration. Treat the first search as calibration, not a final result.**

This is the default pattern for all candidate searches. Workflows 1 and 2 should flow into this workflow whenever results need improvement.

**Trigger:** The initial search returned too few, too many, or poorly matched candidates — or the user wants to explore variations.

### Steps

1. **Run the initial search** using `people-fast-search` with the user's stated criteria.
2. **Evaluate results** with the user:
   - Too few results? → Broaden filters: remove one skill, widen experience range, expand location to state or country level.
   - Too many irrelevant results? → Narrow filters: add a required skill, tighten experience range, add a title filter.
   - Wrong type of candidate? → Adjust the title or skills filter, or add a `company_name` filter for industry targeting.
3. **Refine and re-run.** Adjust filters based on the evaluation and execute `people-fast-search` again.
4. **Repeat** steps 2-3 as needed, up to 3 rounds of refinement.
5. **Terminal condition:** If after 3 rounds of refinement results are still poor, tell the user:
   > "The OpenJobsAI database may have limited coverage for this specific combination of criteria. Consider broadening your search significantly or trying different filter dimensions."

### Refinement Strategies

| Problem | Strategy |
|---|---|
| 0 results | Remove the most restrictive filter (often `skills` or `city`). Try state-level or country-level location. |
| Results lack a key skill | Add that skill to the `skills` array and re-run. |
| Experience levels are off | Adjust `experience_months_min` / `experience_months_max`. See **Experience Range Translation** in `SKILL.md`. |
| Wrong industry/domain | Add or change the `active_title` filter to be more specific (e.g., "Backend Engineer" instead of "Engineer"). |
| Need more variety | Run multiple searches with slight filter variations and combine the unique results. |

---

## Workflow 6: Find Similar Candidates (Replacement Hire)

**Trigger:** User provides a LinkedIn URL of a reference person and wants to find similar candidates (e.g., "find me people like this person", "replacement hire", "find similar candidates").

### Steps

1. **Look up the reference profile.** Use `people-lookup` with the provided LinkedIn URL (pass as `linkedin_urls` array). Extract the profile from `data.results[0]`.
2. **Extract key attributes** from the profile:
   - **Skills:** Take the top 3-5 most relevant skills from the profile.
   - **Experience level:** Note the `total_experience_duration_months` value; use a range of +/- 24 months.
   - **Location:** Note `address.country`, `address.state`, and `address.city`.
   - **Title/role:** Note the `active_experience_title`.
   - **Current company:** Find the entry in `experience` where `is_current: true` and read its `company_name`.
   - **Level:** Infer seniority from the title (e.g., "Senior", "Staff", "Lead", "Principal").
3. **Search for similar candidates.** Use `people-fast-search` with the extracted attributes as filters:
   - Set `active_title` to the reference person's role type (e.g., "Software Engineer").
   - Set `skills` to 2-3 of the most distinctive skills (not generic ones like "Communication").
   - Set `experience_months_min` and `experience_months_max` to a range around the reference person's experience.
   - Set location filters as appropriate (broaden from city to state or country if needed).
4. **Optionally grade results.** If the user also provided a job description, run `people-bulk-grade` on the search results to rank them by fit.
5. **Present results.** Display candidates using the Candidate Display Format, noting which attributes they share with the reference person.
6. **Iterate if needed.** Follow Workflow 5 (Iterative Refinement) to adjust filters based on the results.

### Example

User says: "Find me someone like linkedin.com/in/janepark for a replacement hire."

1. `people-lookup` on Jane Park (pass `{"linkedin_urls": ["https://linkedin.com/in/janepark"]}`) → `data.results[0]`: `active_experience_title` = "Staff Engineer", `total_experience_duration_months` = 108, `address.city` = "San Francisco", skills: Rust, Go, distributed systems, Kubernetes.
2. Extract: title = "Engineer", skills = ["Rust", "Kubernetes"], experience = 84-132 months, state = "California".
3. `people-fast-search` with those filters.
4. Present similar candidates.

---

## Workflow 7: Company Talent Map

**Trigger:** User wants to understand the talent composition of a specific company (e.g., "who works at Stripe?", "show me the engineering team at Acme Corp", "talent map for Google").

### Steps

1. **Search for employees.** Use `people-fast-search` with the `company_name` filter set to the target company.
   - Optionally add an `active_title` filter if the user is interested in a specific function (e.g., "engineers at Stripe").
2. **Get workforce statistics.** Use `people-stats` with the `company_name` filter and `group_by` set to relevant dimensions (note: `group_by` takes an **array**, max 5):
   - `group_by: ["state"]` — breakdown by geography (state level).
   - `group_by: ["country"]` — breakdown by country.
   - `group_by: ["management_level"]` — breakdown by seniority.
   - `group_by: ["active_title"]` — breakdown by job title.
   - `group_by: ["active_department"]` — breakdown by department.
   - `group_by: ["industry"]` — breakdown by industry.
   - `group_by: ["state", "city"]` — multi-level geographic breakdown.
   - Optionally add `stats_fields: ["experience_months"]` for min/max/avg experience.
   - Optionally add `histogram_fields: [{"field": "age", "interval": 10}]` for age distribution.
   - See `API_REFERENCE.md` for the full list of group_by dimensions, stats_fields, and histogram_fields.
3. **Present the org overview.** Combine the search results and statistics into a summary:
   - Total employee count in the database.
   - Distribution by role/function.
   - Distribution by seniority level.
   - Key locations.
   - Notable individuals (if the search returned specific profiles).
4. **Drill down.** If the user wants more detail on a specific segment (e.g., "show me the senior engineers"), re-run `people-fast-search` with additional filters.

---

## Workflow 8: Unlock Candidate Contact Info

**Trigger:** User wants to get candidate email addresses or contact information.

### Steps

1. Collect LinkedIn URLs from the user (or use URLs from a previous search).
2. **Warn the user about quota cost:** Each URL consumes 1 quota point. Confirm before proceeding if the list is large.
3. Use `people-unlock` with the LinkedIn URLs (1–50 per request).
4. Present results:

```
**[Name]** — personEmail: xxx@gmail.com | workEmail: xxx@company.com
```

5. Note any URLs where both emails are `null` — contact info is not available for everyone.

---

## Parameter Construction Guide

Use this guide to translate natural-language recruiting requests into structured `people-fast-search` filter payloads.

### Title Mapping

| User says | `active_title` value |
|---|---|
| "software engineer" | `"Software Engineer"` |
| "backend engineer" / "backend developer" | `"Backend Engineer"` |
| "frontend engineer" / "frontend developer" | `"Frontend Engineer"` |
| "full-stack engineer" / "full-stack developer" | `"Full Stack Engineer"` |
| "data scientist" | `"Data Scientist"` |
| "data engineer" | `"Data Engineer"` |
| "ML engineer" / "machine learning engineer" | `"Machine Learning Engineer"` |
| "DevOps engineer" | `"DevOps Engineer"` |
| "SRE" / "site reliability engineer" | `"Site Reliability Engineer"` |
| "product manager" / "PM" | `"Product Manager"` |
| "engineering manager" / "EM" | `"Engineering Manager"` |
| "designer" / "UX designer" | `"UX Designer"` |
| "lead engineer" | Use `active_title: "Engineer"` combined with experience filters for senior-level (see below). The API does not have a dedicated "Lead" title; filter by experience range or look for "Staff Engineer" / "Senior Engineer" titles. |

### Skills Mapping

| User says | `skills` value | `skills_operator` |
|---|---|---|
| "knows Python and AWS" | `["Python", "AWS"]` | `"AND"` (default) |
| "knows Python or Go" | `["Python", "Go"]` | `"OR"` |
| "React developer" | `["React"]` | — (also set `active_title` to a frontend role) |
| "cloud infrastructure" | `["AWS"]` or `["GCP"]` or `["Azure"]` | — (pick based on context) |
| "full-stack with React and Node" | `["React", "Node.js"]` | `"AND"` |
| "AI/ML skills" | `["Machine Learning"]` or `["PyTorch"]` or `["TensorFlow"]` | — (pick the most specific) |

### Location Mapping

**Always use full names. Never use abbreviations.**

| User says | Filter fields |
|---|---|
| "in the US" / "in America" | `country: "United States"` |
| "in California" / "in CA" | `country: "United States", state: "California"` |
| "in SF" / "in San Francisco" | `country: "United States", state: "California", city: "San Francisco"` |
| "in NYC" / "in New York City" | `country: "United States", state: "New York", city: "New York"` |
| "in the UK" / "in Britain" | `country: "United Kingdom"` |
| "in London" | `country: "United Kingdom", city: "London"` |
| "in Germany" | `country: "Germany"` |
| "in Berlin" | `country: "Germany", city: "Berlin"` |
| "remote" / "anywhere" | Omit all location filters. |

### Experience Mapping

Refer to the **Experience Range Translation** table in `SKILL.md` for the full set of conversions. Key additions:

| User says | `experience_months_min` | `experience_months_max` |
|---|---|---|
| "mid-level" | `36` | `96` |
| "junior" / "entry level" | `0` | `36` |
| "senior" | `60` | `180` |
| "staff" / "principal" | `120` | `300` |
| "3-5 years" | `36` | `60` |
| "5+ years" | `60` | `180` (or `300` with senior/lead context) |

### Level and Role Mapping

| User says | Filter approach |
|---|---|
| "lead engineer" | Set `active_title: "Senior Engineer"` or `active_title: "Staff Engineer"`. Alternatively use experience filters: `experience_months_min: 72, experience_months_max: 180`. |
| "senior IC" | Set `level: "Senior"` or `management_level: "Senior"` |
| "engineering role" | Set `role: "Engineering and Technical"` |
| "sales people" | Set `role: "Sales"` |
| "FAANG engineers" | Search each company separately with `company_name` filter: `"Google"`, `"Meta"`, `"Apple"`, `"Amazon"`, `"Netflix"`. Combine the results from all five searches. |
| "no job hoppers" | Not directly filterable via the API. After retrieving results, review `experience` from `people-lookup` and note in the output if a candidate's average tenure is less than 2 years. Flag these candidates accordingly. |
| "currently employed" | `is_working: true` |
| "open to work" / "not currently employed" | `is_working: false` |

### Company-Specific Searches

| User says | Filter fields |
|---|---|
| "engineers at Google" | `company_name: "Google", active_title: "Engineer"` |
| "FAANG engineers" | Run 5 separate searches with `company_name` set to each of: `"Google"`, `"Meta"`, `"Apple"`, `"Amazon"`, `"Netflix"`. Combine and deduplicate. |
| "ex-Stripe" | Not directly filterable by past employers. Search with `active_title` and `skills` relevant to the role, then use `people-lookup` on results to check `experience` for Stripe. |

### Combining Filters

When the user specifies multiple criteria, combine them into a single `people-fast-search` payload:

**Example:** "Find senior backend engineers in California who know Python and Kubernetes"

```json
{
  "active_title": "Backend Engineer",
  "skills": ["Python", "Kubernetes"],
  "skills_operator": "AND",
  "country": "United States",
  "state": "California",
  "experience_months_min": 60,
  "experience_months_max": 180,
  "is_working": true
}
```

### Filter Priority

When too many filters produce zero results, relax them in this order (least important first):

1. `city` — broaden to state or country
2. `is_working` — remove employment status filter
3. `experience_months_max` — raise the upper bound
4. One skill from the `skills` array — remove the least critical skill
5. `state` — broaden to country-level
6. `active_title` — only remove as a last resort

---

## Notes

- `people-fast-search` returns a maximum of **20 results** per request. There is no pagination. To see more candidates, refine filters or run multiple searches with variations.
- When running multi-step workflows (search then grade), extract LinkedIn URLs from the search results to pass to `people-bulk-grade`.
- Always apply the **Location Format Rule** from `SKILL.md`: full names only, never abbreviations.
- For grading, a job description is always required. If the user has not provided one, ask before calling any grading endpoint.
