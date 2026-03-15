# Mira Recruiting Skill -- Troubleshooting Guide

Use this document when an API call to Mira fails or returns unexpected results. Each section tells you exactly what to do.

---

## 1. HTTP Error Handling

When an API call returns a non-2xx status code, take the action listed below. Do NOT blindly retry every error -- some are permanent.

| Status Code | Meaning | Action |
|---|---|---|
| **400** Bad Request | The request body or query parameters are malformed. | Read the error message carefully. Fix the request payload (wrong field name, invalid enum value, missing required field) and retry once. Do NOT retry with the same payload. |
| **401** Unauthorized | The API key or auth token is missing, expired, or invalid. | Do NOT retry. Inform the user that authentication has failed and ask them to verify their Mira API credentials. |
| **402** Quota Exhausted | API quota has been depleted. | Do NOT retry. Inform the user their quota is exhausted and suggest they check their plan at https://platform.openjobs-ai.com/ |
| **403** Forbidden | The credentials are valid but the API key is disabled, expired, or has insufficient scope. | Do NOT retry. Tell the user they do not have permission for the requested operation and suggest they check their account role or plan tier. |
| **404** Not Found | The endpoint path or a referenced resource ID does not exist. | Verify the URL path is correct. If a candidate or job ID was passed, confirm it exists. Do NOT retry with the same ID -- inform the user the resource was not found. |
| **422** Unprocessable Entity | The request is syntactically valid but semantically wrong. | Read the error body for field-level details. Fix the offending field value and retry once. **Note:** Location abbreviations do NOT cause a 422 — they silently return empty results. See Section 6. |
| **429** Too Many Requests | Rate limit exceeded. | Follow the rate-limiting strategy in Section 7. Wait for the duration specified in the `Retry-After` header before retrying. |
| **500** Internal Server Error | An unexpected error on Mira's servers. | Retry the exact same request up to 2 times with exponential backoff (2s, then 4s). If it still fails, inform the user that the Mira service is experiencing an internal error. |
| **502** Bad Gateway | An upstream server returned an invalid response to Mira. | Retry up to 2 times with exponential backoff (2s, then 4s). If it persists, inform the user of a temporary service issue. |
| **503** Service Unavailable | Mira is temporarily down for maintenance or overloaded. | Check for a `Retry-After` header. If present, wait that long. Otherwise, retry up to 2 times with exponential backoff (5s, then 10s). If it still fails, tell the user the service is temporarily unavailable. |

### General Rules

- **4xx errors (except 429):** Fix the request before retrying. Repeating the same request will produce the same error.
- **5xx errors:** These may be transient. Retry with backoff, but cap retries at 2 attempts.
- **Always surface the error message from the response body** -- it often contains actionable detail.

---

## 2. Network Errors

These errors occur before any HTTP status code is received.

| Error Type | Symptoms | Action |
|---|---|---|
| **Timeout (>30s)** | The request hangs and eventually times out. | Retry once with the same request. If it times out again, inform the user that the Mira API is not responding and suggest trying again later. Do NOT increase the timeout beyond 30 seconds. |
| **DNS Failure** | Cannot resolve the API hostname. Error messages mention `ENOTFOUND`, `getaddrinfo`, or similar. | Do NOT retry immediately. Inform the user that the API hostname could not be resolved. This usually indicates a network configuration problem or that the service domain has changed. |
| **Connection Refused** | The server actively refused the connection. Error messages mention `ECONNREFUSED`. | Retry once after 5 seconds. If it fails again, inform the user that the Mira service is not accepting connections and may be down. |
| **SSL/TLS Errors** | Certificate validation failures, handshake errors, or `UNABLE_TO_VERIFY_LEAF_SIGNATURE`. | Do NOT retry. Do NOT disable certificate verification. Inform the user that there is a TLS certificate problem with the Mira API. This may indicate a configuration issue or a man-in-the-middle concern. |

---

## 3. Empty Results Troubleshooting

When a search or list call returns zero results (an empty `data` array), do NOT immediately tell the user "no results found." First consider these common causes and attempt fixes.

### Common Causes and Fixes

| Cause | How to Detect | Fix |
|---|---|---|
| **Filters too restrictive** | Multiple filters were applied (e.g., location + title + skills + years of experience all at once). | Remove or relax one filter at a time, starting with the most restrictive. Re-run the query after each change to see if results appear. Report back to the user what filter combination yields results. |
| **Location abbreviations used** | The location filter contains abbreviations like "US", "CA", "NY", "SF", "UK". | Replace with full names: "United States", "California", "New York", "San Francisco", "United Kingdom". See Section 6 for details. |
| **Invalid enum value** | A filter uses a value that is not in the API's allowed enum set (e.g., invalid `management_level` or `industry`). The API does NOT return a 4xx error — it silently returns zero results. | Check `SEARCH_FIELDS.md` for valid enum values. Correct the value and retry. This is a common silent failure. |
| **Typos in company or school names** | The company or school name does not exactly match what Mira has indexed. | Try a partial match or broader search term. For example, use "Google" instead of "Google LLC" or "Alphabet Inc." |
| **Date range too narrow** | A date-based filter (e.g., last active) excludes most candidates. | Widen the date range or remove the date filter entirely. |

### What to Tell the User

After attempting fixes, if results are still empty, tell the user:
- Which filters were applied
- Which filters were relaxed during troubleshooting
- That the combination of criteria did not match any candidates in Mira's database
- Suggest specific filter relaxations they might try

---

## 4. Malformed Response Handling

Sometimes the API returns a 2xx status but the response body is unexpected.

### Incomplete or Truncated JSON

- **Detection:** JSON parsing fails with a syntax error.
- **Action:** Retry the request once. If the response is still malformed, inform the user that the API returned an invalid response and report the first 200 characters of the raw body for diagnostic purposes.

### Unexpectedly Null Fields

- **Detection:** Fields that should contain data (e.g., `candidate.name`, `candidate.email`, `grade.score`) are `null` or missing.
- **Action:** Do NOT treat null fields as errors that block the entire response. Present the data that IS available to the user. Note which fields are missing. For example: "I found 5 candidates but email addresses are unavailable for 2 of them."

### Response Structure Mismatch

- **Detection:** The top-level keys or nesting of the response do not match the expected schema (e.g., missing the `{ "code": 200, "message": "ok", "data": ... }` wrapper).
- **Action:** Attempt to locate the relevant data by checking common wrapper patterns (`data`, `results`, `items`, `records`). If the data can be found, proceed and use it. If the structure is entirely unrecognizable, inform the user that the API response format may have changed and include the top-level keys you received.

---

## 5. Bulk Grade Partial Failures

When using `people-bulk-grade` (or similar bulk endpoints), some candidates in the batch may succeed while others fail. Handle this gracefully.

### How to Handle

1. **Always present the successful results first.** Do not let failures in some candidates block the user from seeing the candidates that were graded successfully.

2. **Group and report failures separately.** After showing successful results, list the failed candidates with:
   - The candidate identifier (name or ID)
   - The error message or reason for failure
   - Whether the error is retryable

3. **Offer to retry failed candidates.** If the errors are transient (5xx, timeout), offer to retry just the failed subset. Do NOT re-submit the entire batch -- only the failed items.

### Example Output Format

```
Successfully graded 8 of 10 candidates:

| Candidate | Grade | Score |
|---|---|---|
| Jane Smith | A | 92 |
| ... | ... | ... |

2 candidates could not be graded:
- John Doe: 500 Internal Server Error (retryable)
- Alex Park: 422 Invalid candidate ID "xyz-000" (not retryable -- ID may be incorrect)

Would you like me to retry the 1 retryable failure?
```

### Important Rules

- Never silently drop failed candidates from the results.
- Never report the entire batch as failed if any candidates succeeded.
- If ALL candidates fail with the same error, report it as a single issue rather than repeating the same error N times.

---

## 6. Location Empty Results (Silent Failure)

This is the **single most common issue** when working with the Mira API. The API requires full location names — abbreviations will NOT cause a 422 error. Instead, the API **silently returns zero results**, making the problem hard to diagnose.

### The Rule

Always use **full, unabbreviated location names** in every API call that accepts a location parameter.

### Common Mistakes and Corrections

| Wrong (silently returns 0 results) | Correct |
|---|---|
| `US` | `United States` |
| `USA` | `United States` |
| `UK` | `United Kingdom` |
| `CA` (state) | `California` |
| `CA` (country) | `Canada` |
| `NY` | `New York` |
| `SF` | `San Francisco` |
| `LA` | `Los Angeles` |
| `DC` | `Washington, D.C.` |
| `TX` | `Texas` |
| `MA` | `Massachusetts` |
| `WA` | `Washington` |
| `IL` | `Illinois` |
| `CO` | `Colorado` |
| `GA` | `Georgia` |
| `UAE` | `United Arab Emirates` |
| `KSA` | `Saudi Arabia` |

### What to Do When Location Filters Return Zero Results

1. Check if any location field uses an abbreviation (US, CA, NY, UK, etc.).
2. Replace the abbreviated location with its full name.
3. If the user said something like "candidates in CA," clarify whether they mean California (US state) or Canada (country) before retrying.
4. Retry the request with the corrected location.

### Proactive Prevention

When the user provides a location using an abbreviation, expand it to the full name BEFORE making the API call. The API will not warn you — it silently returns zero results.

---

## 7. Rate Limiting Strategy

When you receive a **429 Too Many Requests** response, follow this procedure exactly.

### Step-by-Step

1. **Check the `Retry-After` header.** If present, it contains the number of seconds to wait. Wait exactly that long before retrying.

2. **If no `Retry-After` header is present**, use exponential backoff:
   - 1st retry: wait 2 seconds
   - 2nd retry: wait 4 seconds
   - 3rd retry: wait 8 seconds
   - Stop after 3 retries.

3. **Do NOT attempt to parallelize requests to work around rate limits.** This will make the problem worse.

4. **If multiple API calls are needed** (e.g., grading 50 candidates), space them out by at least 1 second between calls to avoid hitting the limit in the first place.

5. **If rate limiting persists after 3 retries**, inform the user that the API rate limit has been reached and suggest waiting a few minutes before trying again.

### What NOT to Do

- Do not retry immediately without waiting.
- Do not increase the request volume.
- Do not make concurrent requests to "get ahead" of the limit.
- Do not treat 429 as a permanent failure -- it is always temporary.

---

## 8. Known API Quirks

These are confirmed behaviors discovered through testing. They are not bugs you can fix — document them so you handle them correctly.

### Duplicate URL Deduplication

`people-lookup`, `people-compare`, and `people-bulk-grade` all silently deduplicate URLs. If you pass the same URL twice:
- The server processes it once
- `data.total` (or `total_requested`) reflects the **deduplicated** count
- This is not an error, but do not rely on array length matching your input count

**Action:** Deduplicate URLs client-side before calling any endpoint. If the user provides duplicate URLs, remove them and note it.

### people-stats 500 on Invalid group_by

Passing an unsupported `group_by` value causes a **500 Internal Server Error** instead of a 400/422. This is a server-side bug.

**Verified safe values:** `country`, `city`, `state`, `active_title`, `active_department`, `management_level`, `job_title`, `company_name`, `industry`, `company_type`, `level`, `role`, `exp_country`, `exp_city`, `degree_level`, `degree_str`, `institution_name`, `major`, `institution_country`, `institution_city`, `skills`, `is_working`, `is_decision_maker`, `languages`.

**Action:** Only use values from the list above. Always wrap `people-stats` calls with error handling.

### Lenient Input Validation

The API does NOT strictly validate several input fields:
- **Bad LinkedIn URLs**: Non-LinkedIn URLs or malformed URLs do not cause errors — they appear in `not_found`.
- **Invalid enum values**: Wrong `management_level`, `industry`, `function`, or `employment_type` values do not cause errors — they silently return zero results.
- **Location abbreviations**: "US", "CA", "NY" etc. do not cause errors — they silently return zero results.

**The only strict validation:** Empty or missing `jd` in grading endpoints correctly returns 422.

**Action:** Always validate inputs client-side. Do not rely on the API to catch mistakes — most invalid inputs result in silent empty results rather than clear error messages.

---

## 9. Escalation Path

Stop retrying and inform the user when any of the following conditions are met.

### When to Stop

| Condition | Action |
|---|---|
| **3 consecutive retries of the same request have failed** | Stop retrying. Tell the user the operation failed and include the most recent error message. |
| **Authentication error (401 or 403)** | Stop immediately. Do not retry. Ask the user to verify credentials or permissions. |
| **The same 422 error repeats after you have corrected the request** | Stop. Show the user the exact error message and the request you sent, so they can identify the issue. |
| **The API returns a 404 for a known endpoint** | Stop. The API may have changed. Inform the user and suggest checking for API updates. |
| **Network errors persist across multiple different request types** | Stop. The issue is likely environmental (network, DNS, firewall), not request-specific. Tell the user. |
| **The response structure is unrecognizable and cannot be parsed** | Stop. Inform the user that the API response format may have changed and include the raw structure for diagnosis. |

### How to Escalate

When you stop retrying, always provide the user with:

1. **What you were trying to do** -- the operation and its parameters.
2. **What went wrong** -- the specific error code, message, or behavior.
3. **How many times you retried** -- and over what timespan.
4. **What the user can do next** -- check credentials, try later, contact Mira support, adjust their query, etc.

Never leave the user with just "an error occurred." Always give them enough context to take the next step.
