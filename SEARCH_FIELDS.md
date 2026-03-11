# Mira — Search Fields Reference

Complete reference for all filter fields accepted by the `people-fast-search` endpoint. Every field is optional — omit or set to `null` to skip that filter.

---

## Quick Reference

| Field | Type | Example Value |
|---|---|---|
| `title` | string | `"Backend Engineer"` |
| `skills` | string[] | `["Python", "AWS"]` |
| `country` | string | `"United States"` |
| `state` | string | `"California"` |
| `city` | string | `"San Francisco"` |
| `min_experience_months` | integer | `60` |
| `max_experience_months` | integer | `180` |
| `is_working` | boolean | `true` |
| `management_level` | string | `"Director"` |
| `company_name` | string | `"Stripe"` |
| `company_size` | string | `"51-200"` |
| `company_industry` | string | `"Information Technology"` |
| `company_type` | string | `"Privately Held"` |
| `industry` | string | `"Finance & Accounting"` |
| `function` | string | `"Engineering"` |
| `employment_type` | string | `"Full-Time"` |

---

## Title

**Type:** string (partial match)

The API performs partial, case-insensitive matching on the candidate's current job title. Use the canonical form from the table below when possible.

### Common Title Values

| User Intent | `title` Value |
|---|---|
| Software engineer (general) | `"Software Engineer"` |
| Backend engineer / developer | `"Backend Engineer"` |
| Frontend engineer / developer | `"Frontend Engineer"` |
| Full-stack engineer / developer | `"Full Stack Engineer"` |
| Data scientist | `"Data Scientist"` |
| Data engineer | `"Data Engineer"` |
| ML / machine learning engineer | `"Machine Learning Engineer"` |
| DevOps engineer | `"DevOps Engineer"` |
| SRE / site reliability engineer | `"Site Reliability Engineer"` |
| Product manager / PM | `"Product Manager"` |
| Engineering manager / EM | `"Engineering Manager"` |
| UX designer / designer | `"UX Designer"` |
| Data analyst | `"Data Analyst"` |
| Business analyst | `"Business Analyst"` |
| Solutions architect | `"Solutions Architect"` |
| QA / test engineer | `"QA Engineer"` |
| Security engineer | `"Security Engineer"` |
| Mobile engineer | `"Mobile Engineer"` |
| iOS engineer | `"iOS Engineer"` |
| Android engineer | `"Android Engineer"` |
| Technical program manager | `"Technical Program Manager"` |
| Recruiter | `"Recruiter"` |
| Account executive | `"Account Executive"` |

**Notes:**
- There is no dedicated "Lead" title filter. For lead-level candidates, use `title: "Senior Engineer"` or `title: "Staff Engineer"` and combine with experience range filters.
- Seniority prefixes (Senior, Staff, Principal) can be included in the title string — the partial match will work (e.g., `"Senior Software Engineer"`).
- If in doubt, use the broader form (e.g., `"Engineer"` rather than `"Backend Engineer"`) and rely on `skills` to narrow results.

---

## Skills

**Type:** string[] (AND matching — all listed skills must be present)

Skills are matched against the candidate's skill list. Matching is case-insensitive but otherwise exact per skill value. All skills in the array must be present on the candidate's profile for a match.

### Common Skill Values

**Languages & Frameworks:**
`"Python"`, `"JavaScript"`, `"TypeScript"`, `"Java"`, `"Go"`, `"Rust"`, `"C++"`, `"C#"`, `"Ruby"`, `"PHP"`, `"Swift"`, `"Kotlin"`, `"Scala"`, `"R"`, `"SQL"`, `"React"`, `"Angular"`, `"Vue.js"`, `"Node.js"`, `"Django"`, `"Flask"`, `"Spring"`, `"Rails"`, `".NET"`, `"Next.js"`

**Cloud & Infrastructure:**
`"AWS"`, `"GCP"`, `"Azure"`, `"Docker"`, `"Kubernetes"`, `"Terraform"`, `"Ansible"`, `"Jenkins"`, `"CI/CD"`, `"Linux"`

**Data & ML:**
`"Machine Learning"`, `"Deep Learning"`, `"PyTorch"`, `"TensorFlow"`, `"NLP"`, `"Computer Vision"`, `"Spark"`, `"Hadoop"`, `"Airflow"`, `"dbt"`, `"Snowflake"`, `"Databricks"`

**Databases:**
`"PostgreSQL"`, `"MySQL"`, `"MongoDB"`, `"Redis"`, `"Elasticsearch"`, `"DynamoDB"`, `"Cassandra"`

**Other:**
`"Kafka"`, `"gRPC"`, `"GraphQL"`, `"REST"`, `"Microservices"`, `"System Design"`, `"Agile"`, `"Scrum"`, `"Figma"`, `"Tableau"`, `"Power BI"`, `"Salesforce"`, `"SAP"`

### Matching Behavior

- **AND logic**: `["Python", "AWS"]` returns only candidates who have both Python AND AWS.
- **No OR logic**: To search for candidates with Python OR Go, run two separate searches and combine the results.
- **Exact skill names**: `"React"` will not match `"React.js"` or `"ReactJS"`. Use the canonical form listed above.
- **Limit to 2-5 skills** per search. Too many required skills will return zero results. Start with 2-3 core skills and add more only to narrow an overly broad result set.

---

## Location Fields

**CRITICAL: All location fields MUST use full, unabbreviated names. The API will reject or misinterpret abbreviations.**

### country

**Type:** string

| User Says | Correct Value | WRONG |
|---|---|---|
| US, USA, America | `"United States"` | ~~"US"~~, ~~"USA"~~ |
| UK, Britain | `"United Kingdom"` | ~~"UK"~~, ~~"GB"~~ |
| UAE | `"United Arab Emirates"` | ~~"UAE"~~ |
| South Korea | `"South Korea"` | ~~"KR"~~ |
| The Netherlands | `"Netherlands"` | ~~"Holland"~~ |

Other countries use their standard English full name: `"Germany"`, `"France"`, `"Canada"`, `"Australia"`, `"India"`, `"Japan"`, `"Brazil"`, `"Singapore"`, `"Israel"`, etc.

### state

**Type:** string

US states must use full names. This applies to all countries with state/province/region subdivisions.

| User Says | Correct Value | WRONG |
|---|---|---|
| CA, Cali | `"California"` | ~~"CA"~~ |
| NY | `"New York"` | ~~"NY"~~ |
| TX | `"Texas"` | ~~"TX"~~ |
| WA | `"Washington"` | ~~"WA"~~ |
| MA, Mass | `"Massachusetts"` | ~~"MA"~~ |
| IL | `"Illinois"` | ~~"IL"~~ |
| CO | `"Colorado"` | ~~"CO"~~ |
| ON | `"Ontario"` | ~~"ON"~~ |
| BC | `"British Columbia"` | ~~"BC"~~ |
| NSW | `"New South Wales"` | ~~"NSW"~~ |
| Bavaria | `"Bavaria"` | ~~"Bayern"~~ |

### city

**Type:** string

Use the common English name of the city.

| User Says | Correct Value | Notes |
|---|---|---|
| SF | `"San Francisco"` | Always set `state: "California"` too |
| NYC, New York City | `"New York"` | Always set `state: "New York"` too |
| LA | `"Los Angeles"` | |
| DC | `"Washington"` | Set `state: "District of Columbia"` |
| the Bay Area | Omit `city`; set `state: "California"` | Bay Area is not a single city — use state-level |
| Bangalore | `"Bengaluru"` | Use official city name |

### Location Hierarchy

Always set the parent location fields when specifying a child:
- If you set `city`, also set `state` and `country`.
- If you set `state`, also set `country`.
- If user says "remote" or "anywhere", omit all location fields.

---

## Experience Months

### min_experience_months

**Type:** integer

Minimum total professional experience in months. A value of `60` means "at least 5 years."

### max_experience_months

**Type:** integer

Maximum total professional experience in months. A value of `180` means "at most 15 years."

### Interpretation Guide

| User Intent | `min` | `max` |
|---|---|---|
| Entry-level / junior / 0-2 years | `0` | `24` |
| Junior / under 3 years | `0` | `36` |
| Mid-level / 3-5 years | `36` | `60` |
| Mid-level (broad) / 3-8 years | `36` | `96` |
| Senior / 5+ years (IC context) | `60` | `180` |
| Senior / 5+ years (with Lead/Manager signals) | `60` | `300` |
| Staff / principal | `120` | `300` |
| 10+ years (IC context) | `120` | `240` |
| 10+ years (with senior/lead signals) | `120` | `300` |
| About 5 years | `36` | `84` |
| About 10 years | `96` | `144` |
| Exactly 5 years | `48` | `72` |
| 3-5 years (explicit range) | `36` | `60` |
| 7-10 years (explicit range) | `84` | `120` |

See also the **Experience Range Translation** table in `SKILL.md` for the full set of role-level-aware conversions.

---

## is_working

**Type:** boolean

| Value | Meaning |
|---|---|
| `true` | Currently employed |
| `false` | Not currently employed (may indicate open to work) |
| omitted / `null` | No filter on employment status |

**Usage notes:**
- "Currently employed" / "actively working" -> `true`
- "Open to work" / "not currently employed" / "available" -> `false`
- Default: omit. Only set when the user specifically mentions employment status.

---

## management_level

**Type:** string (enum)

Filter by the candidate's management/seniority level.

### Accepted Values

| Value | Description |
|---|---|
| `"Individual Contributor"` | IC roles with no direct reports |
| `"Manager"` | First-line managers, team leads with direct reports |
| `"Director"` | Directors, senior directors |
| `"VP"` | Vice presidents, SVPs |
| `"C-Suite"` | CTO, CEO, CFO, CIO, CPO, etc. |
| `"Owner"` | Founders, co-founders, business owners |

**There is NO "Junior" or "Entry-Level" enum value.** To find junior candidates, use `management_level: "Individual Contributor"` combined with `max_experience_months: 36` (or an appropriate low range).

**There is NO "Senior" or "Staff" enum value.** Senior ICs are still `"Individual Contributor"`. Distinguish them by experience range:
- Senior IC: `management_level: "Individual Contributor"`, `min_experience_months: 60`
- Staff/Principal IC: `management_level: "Individual Contributor"`, `min_experience_months: 120`

---

## Company Fields

### company_name

**Type:** string

Filter by the candidate's current employer. Use the company's common name.

| User Says | `company_name` Value |
|---|---|
| Google / Alphabet | `"Google"` |
| Meta / Facebook | `"Meta"` |
| Amazon / AWS (the company) | `"Amazon"` |
| Microsoft | `"Microsoft"` |
| Apple | `"Apple"` |
| FAANG engineers | Run 5 separate searches: `"Google"`, `"Meta"`, `"Apple"`, `"Amazon"`, `"Netflix"` |

**Note:** This filters on current employer only. To find ex-employees of a company, search by relevant title/skills and then use `people-lookup` to check `work_history` for past employers.

### company_size

**Type:** string (enum)

Filter by the size of the candidate's current employer (employee count).

| Value | Meaning |
|---|---|
| `"1-10"` | Micro startup |
| `"11-50"` | Small startup |
| `"51-200"` | Growth-stage / small company |
| `"201-500"` | Mid-size company |
| `"501-1000"` | Upper mid-size |
| `"1001-5000"` | Large company |
| `"5001-10000"` | Enterprise |
| `"10001+"` | Major enterprise / big tech |

**Usage notes:**
- "Startup" -> `"1-10"` or `"11-50"` (run both if user is vague)
- "Small company" -> `"11-50"` or `"51-200"`
- "Big tech" / "enterprise" -> `"10001+"`
- "Mid-size" -> `"201-500"` or `"501-1000"`

### company_industry

**Type:** string

Industry of the candidate's current employer. This is distinct from the candidate-level `industry` field. Common values include:

`"Information Technology"`, `"Financial Services"`, `"Healthcare"`, `"Manufacturing"`, `"Retail"`, `"Education"`, `"Telecommunications"`, `"Media & Entertainment"`, `"Consulting"`, `"Energy"`, `"Real Estate"`, `"Government"`, `"Nonprofit"`, `"Aerospace & Defense"`, `"Automotive"`, `"Pharmaceuticals"`, `"Insurance"`, `"Legal Services"`, `"Transportation"`, `"Agriculture"`

### company_type

**Type:** string (enum)

| Value |
|---|
| `"Public Company"` |
| `"Privately Held"` |
| `"Nonprofit"` |
| `"Government Agency"` |
| `"Educational Institution"` |
| `"Partnership"` |
| `"Sole Proprietorship"` |

---

## Industry

**Type:** string (enum)

The candidate's professional industry classification. This describes the candidate's domain, not necessarily the employer's industry.

### Accepted Values

| Value | Notes |
|---|---|
| `"Accounting"` | |
| `"Administrative"` | |
| `"Advertising & Marketing"` | Uses `&` |
| `"Aerospace & Defense"` | Uses `&` |
| `"Agriculture"` | |
| `"Arts & Design"` | Uses `&` |
| `"Automotive"` | |
| `"Banking"` | |
| `"Biotechnology"` | |
| `"Business Development"` | |
| `"Consulting"` | |
| `"Consumer Goods"` | |
| `"Education"` | |
| `"Energy & Utilities"` | Uses `&` |
| `"Engineering"` | |
| `"Entertainment"` | |
| `"Environmental Services"` | |
| `"Finance & Accounting"` | Uses `&` not "and" |
| `"Financial Services"` | |
| `"Food & Beverage"` | Uses `&` |
| `"Government"` | |
| `"Healthcare"` | |
| `"Hospitality"` | |
| `"Human Resources"` | |
| `"Information Technology"` | |
| `"Insurance"` | |
| `"Legal"` | |
| `"Logistics & Supply Chain"` | Uses `&` |
| `"Manufacturing"` | |
| `"Media & Communications"` | Uses `&` |
| `"Mining & Metals"` | Uses `&` |
| `"Nonprofit"` | |
| `"Pharmaceuticals"` | |
| `"Real Estate"` | |
| `"Retail"` | |
| `"Sales"` | |
| `"Telecommunications"` | |
| `"Transportation"` | |

**Important:** Multi-word industries that contain a conjunction always use `&` (ampersand), never `and`. For example, `"Finance & Accounting"` is correct; `"Finance and Accounting"` will not match.

---

## Function

**Type:** string (enum)

The candidate's professional function or department. Describes what the candidate does, regardless of industry.

### Accepted Values

| Value |
|---|
| `"Accounting"` |
| `"Administrative"` |
| `"Advertising"` |
| `"Analyst"` |
| `"Arts & Design"` |
| `"Business Development"` |
| `"Community & Social Services"` |
| `"Consulting"` |
| `"Customer Service"` |
| `"Education"` |
| `"Engineering"` |
| `"Finance"` |
| `"Healthcare"` |
| `"Human Resources"` |
| `"Information Technology"` |
| `"Legal"` |
| `"Marketing"` |
| `"Media & Communications"` |
| `"Military & Protective Services"` |
| `"Operations"` |
| `"Product Management"` |
| `"Program & Project Management"` |
| `"Purchasing"` |
| `"Quality Assurance"` |
| `"Real Estate"` |
| `"Research"` |
| `"Sales"` |
| `"Support"` |
| `"Writing & Editing"` |

**Notes:**
- Multi-word values with conjunctions use `&` (ampersand), never `and`.
- Software engineers, data engineers, DevOps engineers, etc. all fall under `"Engineering"`.
- Product managers fall under `"Product Management"`.
- TPMs fall under `"Program & Project Management"`.

---

## Employment Type

**Type:** string (enum)

Filter by the candidate's current employment arrangement.

### Accepted Values

| Value | Description |
|---|---|
| `"Full-Time"` | Standard full-time employment |
| `"Part-Time"` | Part-time employment |
| `"Contract"` | Contract / freelance work |
| `"Internship"` | Intern positions |
| `"Temporary"` | Temporary / seasonal roles |
| `"Volunteer"` | Volunteer positions |
| `"Self-Employed"` | Independent freelancer or consultant (works for themselves, no company) |

**"Self-Employed" vs. "Self-Owned" distinction:**
- `"Self-Employed"` is the correct API value. It describes individuals who work independently as freelancers, consultants, or sole practitioners.
- There is NO `"Self-Owned"` value. If a user asks for business owners or founders, use `management_level: "Owner"` instead, not `employment_type`.
- `"Self-Employed"` = works for themselves (freelancer/consultant). `management_level: "Owner"` = owns/founded a company (may have employees).
