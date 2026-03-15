# Mira — Search Fields Reference

Complete reference for all filter fields accepted by the `people-fast-search` endpoint. Every field is optional — omit or set to `null` to skip that filter.

---

## Quick Reference

| Field | Type | Example Value |
|---|---|---|
| `full_name` | string | `"John Smith"` |
| `headline` | string | `"Senior Engineer at Google"` |
| `active_title` | string | `"Backend Engineer"` |
| `active_department` | string | `"Engineering"` |
| `skills` | string[] | `["Python", "AWS"]` |
| `skills_operator` | string | `"AND"` or `"OR"` |
| `country` | string | `"United States"` |
| `state` | string | `"California"` |
| `city` | string | `"San Francisco"` |
| `experience_months_min` | integer | `60` |
| `experience_months_max` | integer | `84` |
| `is_working` | boolean | `true` |
| `is_decision_maker` | boolean | `true` |
| `management_level` | string | `"Director"` |
| `level` | string | `"Senior"` |
| `role` | string | `"Engineering and Technical"` |
| `company_name` | string | `"Stripe"` |
| `industry` | string | `"Technology, Information and Media"` |
| `company_type` | string | `"Privately Held"` |
| `certifications` | string | `"AWS"` |
| `languages` | string[] | `["English", "Spanish"]` |
| `degree_level_min` | integer | `2` (Master) |
| `institution_name` | string | `"Stanford University"` |
| `major` | string | `"Computer Science"` |
| `institution_ranking_max` | integer | `100` |

---

## Full Name

**Type:** string (partial match)

Search by candidate's name. Useful when the user asks for a specific person by name (e.g., "find John Smith in California").

```json
{ "full_name": "John Smith", "country": "United States" }
```

**Notes:**
- Combine with location or company filters to narrow results when the name is common.
- Matching is case-insensitive and partial (e.g., `"John"` will match "John Smith", "John Doe", etc.).

---

## Headline

**Type:** string (fuzzy match)

Search by the candidate's LinkedIn headline text. Useful for broad keyword matching across the entire headline.

```json
{ "headline": "Machine Learning" }
```

---

## Active Title

**Field:** `active_title` (alias: `title`)

**Type:** string (fuzzy match)

The API performs partial, case-insensitive matching on the candidate's current job title. Use the canonical form from the table below when possible.

### Common Title Values

| User Intent | `active_title` Value |
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
- There is no dedicated "Lead" title filter. For lead-level candidates, use `active_title: "Senior Engineer"` or `active_title: "Staff Engineer"` and combine with experience range filters.
- Seniority prefixes (Senior, Staff, Principal) can be included in the title string — the partial match will work (e.g., `"Senior Software Engineer"`).
- If in doubt, use the broader form (e.g., `"Engineer"` rather than `"Backend Engineer"`) and rely on `skills` to narrow results.

---

## Active Department

**Field:** `active_department`

**Type:** string (fuzzy match)

Filter by the candidate's current department (e.g., "Engineering", "Sales", "Marketing").

```json
{ "active_department": "Engineering" }
```

---

## Skills

**Type:** string[] (default AND matching — all listed skills must be present)

Skills are matched against the candidate's skill list. Matching is case-insensitive but otherwise exact per skill value. Each skill must be atomic (e.g., `"Python"`, not `"Python backend development"`).

Use `skills_operator` to control matching logic:
- `"AND"` (default) — all skills must be present
- `"OR"` — any skill can match

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

- **AND logic** (default): `["Python", "AWS"]` with `skills_operator: "AND"` returns only candidates who have both Python AND AWS.
- **OR logic**: `["Python", "Go"]` with `skills_operator: "OR"` returns candidates who have Python OR Go (or both).
- **Exact skill names**: `"React"` will not match `"React.js"` or `"ReactJS"`. Use the canonical form listed above.
- **Limit to 2-5 skills** per search. Too many required skills with AND logic will return zero results. Start with 2-3 core skills and add more only to narrow an overly broad result set.

---

## Location Fields

**CRITICAL: All location fields MUST use full, unabbreviated names. Abbreviations will NOT cause an error — the API silently returns zero results, making the problem hard to diagnose.**

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

### experience_months_min

**Type:** integer

Minimum total professional experience in months. A value of `60` means "at least 5 years."

### experience_months_max

**Type:** integer

Maximum total professional experience in months. A value of `84` means "at most 7 years."

### Interpretation Guide

| User Intent | `min` | `max` |
|---|---|---|
| Entry-level / junior / 0-2 years | `0` | `24` |
| Junior / under 3 years | `0` | `36` |
| Mid-level / 3-5 years | `36` | `60` |
| Mid-level (broad) / 3-8 years | `36` | `96` |
| Senior / 5+ years (IC context) | `60` | `84` |
| Senior / 5+ years (with Lead/Manager signals) | `60` | `180` |
| Staff / principal | `120` | `180` |
| 10+ years (IC context) | `120` | `144` |
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

## is_decision_maker

**Type:** boolean

Filter for candidates who are decision makers within their organization.

| Value | Meaning |
|---|---|
| `true` | Decision maker (hiring authority, budget control, etc.) |
| `false` | Not a decision maker |
| omitted / `null` | No filter |

---

## management_level

**Type:** string (enum)

Filter by the candidate's management/seniority level.

### Accepted Values

| Value | Description |
|---|---|
| `"Specialist"` | Individual contributors, specialists, and standard roles |
| `"Senior"` | Senior-level ICs (Senior Engineer, Senior Analyst, etc.) |
| `"Manager"` | First-line managers, team leads with direct reports |
| `"Director"` | Directors, senior directors |
| `"Head"` | Head of department (Head of Engineering, Head of Product, etc.) |
| `"President/Vice President"` | VPs, SVPs, EVPs, Presidents |
| `"Vice President"` | Vice President (alternative to President/Vice President) |
| `"C-Level"` | CTO, CEO, CFO, CIO, CPO, etc. |
| `"Founder"` | Founders, co-founders |
| `"Owner"` | Business owners |
| `"Partner"` | Partners (law firms, consulting, VC, etc.) |
| `"Intern"` | Interns and trainees |

**Mapping from user intent:**
- "junior" / "entry level" → `"Specialist"` combined with `experience_months_max: 36`
- "senior engineer" → `"Senior"`
- "manager" / "team lead" → `"Manager"`
- "director" → `"Director"`
- "VP" / "vice president" → `"President/Vice President"`
- "C-level" / "executive" → `"C-Level"`
- "founder" / "co-founder" → `"Founder"`
- "intern" → `"Intern"`

---

## Level

**Type:** string (exact match)

Position level filter. Uses the same enum values as `management_level` — both fields are accepted by the API.

### Accepted Values

| Value |
|---|
| `"C-Level"` |
| `"Director"` |
| `"Founder"` |
| `"Head"` |
| `"Intern"` |
| `"Manager"` |
| `"Owner"` |
| `"Partner"` |
| `"President/Vice President"` |
| `"Senior"` |
| `"Specialist"` |

---

## Role

**Type:** string (exact match)

Professional function/role classification. Different from the `function` field — use the values below.

### Accepted Values

| Value |
|---|
| `"Administrative"` |
| `"C-Suite"` |
| `"Consulting"` |
| `"Customer Service"` |
| `"Design"` |
| `"Education"` |
| `"Engineering and Technical"` |
| `"Finance & Accounting"` |
| `"Human Resources"` |
| `"Legal"` |
| `"Marketing"` |
| `"Medical"` |
| `"Operations"` |
| `"Other"` |
| `"Product"` |
| `"Project Management"` |
| `"Real Estate"` |
| `"Research"` |
| `"Sales"` |
| `"Trades"` |

**Mapping from user intent:**
- "engineer" / "developer" → `"Engineering and Technical"`
- "product manager" → `"Product"`
- "designer" → `"Design"`
- "recruiter" / "HR" → `"Human Resources"`
- "sales" / "account executive" → `"Sales"`
- "marketing" → `"Marketing"`

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

**Note:** This filters on current employer only. To find ex-employees of a company, search by relevant title/skills and then use `people-lookup` to check `experience` for past employers.

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

**Type:** string (exact match)

| Value |
|---|
| `"Educational"` |
| `"Government Agency"` |
| `"Nonprofit"` |
| `"Partnership"` |
| `"Privately Held"` |
| `"Public Company"` |
| `"Self-Employed"` |
| `"Self-Owned"` |

---

## Industry

**Type:** string (exact match)

The industry classification. Used in both `people-fast-search` and `people-stats`.

### Accepted Values

| Value |
|---|
| `"Accommodation Services"` |
| `"Administrative and Support Services"` |
| `"Construction"` |
| `"Consumer Services"` |
| `"Education"` |
| `"Entertainment Providers"` |
| `"Farming, Ranching, Forestry"` |
| `"Financial Services"` |
| `"Government Administration"` |
| `"Holding Companies"` |
| `"Hospitals and Health Care"` |
| `"Manufacturing"` |
| `"Oil, Gas, and Mining"` |
| `"Professional Services"` |
| `"Real Estate and Equipment Rental Services"` |
| `"Retail"` |
| `"Technology, Information and Media"` |
| `"Transportation, Logistics, Supply Chain and Storage"` |
| `"Utilities"` |
| `"Wholesale"` |

**Important:**
- These are the official industry values from the OpenJobs AI API.
- Use commas and "and" exactly as shown (e.g., `"Farming, Ranching, Forestry"`, not `"Agriculture"`).
- `"Technology, Information and Media"` covers IT, software, internet, telecom, and media companies.
- `"Professional Services"` covers consulting, legal, accounting, and similar service firms.

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

**"Self-Employed" in Employment Type vs. "Self-Owned" in Company Type:**
- `employment_type: "Self-Employed"` = individual who works independently as a freelancer, consultant, or sole practitioner.
- `company_type: "Self-Owned"` = company ownership classification (the company itself is self-owned).
- These are different fields. If a user asks for business owners or founders, use `management_level: "Owner"` or `level: "Owner"`, not `employment_type`.

---

## Certifications

**Type:** string (fuzzy match)

Filter by professional certifications. Uses fuzzy matching.

```json
{ "certifications": "AWS" }
```

Common values: `"AWS"`, `"PMP"`, `"CPA"`, `"CISSP"`, `"CFA"`, `"Google Cloud"`, `"Azure"`, `"Scrum Master"`, `"Six Sigma"`

---

## Languages

**Type:** string[] (all must match)

Filter by spoken languages. All specified languages must be present on the candidate's profile.

```json
{ "languages": ["English", "Mandarin"] }
```

Common values: `"English"`, `"Spanish"`, `"Mandarin"`, `"French"`, `"German"`, `"Japanese"`, `"Portuguese"`, `"Arabic"`, `"Hindi"`, `"Korean"`

---

## Education Fields

### degree_level_min

**Type:** integer

Minimum degree level:

| Value | Meaning |
|---|---|
| `0` | Other / Unclear |
| `1` | Bachelor's degree |
| `2` | Master's degree |
| `3` | PhD / Doctorate |

```json
{ "degree_level_min": 2 }
```

This returns candidates with a Master's degree or higher.

### institution_name

**Type:** string (fuzzy match)

Filter by university or institution name.

```json
{ "institution_name": "Stanford University" }
```

### major

**Type:** string (fuzzy match)

Filter by field of study / major.

```json
{ "major": "Computer Science" }
```

### institution_ranking_max

**Type:** integer

Filter by institution ranking (e.g., `100` means Top 100 universities).

```json
{ "institution_ranking_max": 50 }
```

This returns candidates who graduated from a Top 50 ranked university.
