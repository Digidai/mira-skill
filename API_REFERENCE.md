# Mira Recruiting Skill API Reference

Base URL: `https://mira-api.openjobs-ai.com/v1/`

All endpoints accept JSON request bodies and return JSON responses. Authentication is required via an `Authorization: Bearer $MIRA_KEY` header on every request.

All successful responses are wrapped in a standard envelope:

```json
{
  "code": 200,
  "message": "ok",
  "data": { ... }
}
```

The `data` field contains the endpoint-specific payload described in each section below.

---

## Endpoints Overview

| Endpoint | Method | Path | Description |
|---|---|---|---|
| **people-lookup** | POST | `/people-lookup` | Retrieve detailed profiles for one or more people by LinkedIn URL |
| **people-compare** | POST | `/people-compare` | Compare two or more candidates side-by-side |
| **people-bulk-grade** | POST | `/people-bulk-grade` | Grade a batch of candidates against a job description |
| **people-grade** | POST | `/people-grade` | Grade a single CV/resume text against a job description |
| **people-fast-search** | POST | `/people-fast-search` | Search for candidates matching specific filters |
| **people-stats** | POST | `/people-stats` | Get aggregate statistics for a talent pool |

---

## people-lookup

Retrieve detailed profiles for one or more people by LinkedIn URL. Takes an array of URLs and returns structured results.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-lookup`

### Request Example

```json
{
  "linkedin_urls": [
    "https://www.linkedin.com/in/johndoe"
  ]
}
```

The `linkedin_urls` field is an **array** (not a single string). You may look up multiple profiles in one request.

### Response Example

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "total": 1,
    "found": 1,
    "not_found": [],
    "results": [
      {
        "linkedin_url": "https://www.linkedin.com/in/johndoe",
        "full_name": "John Doe",
        "first_name": "John",
        "last_name": "Doe",
        "headline": "Senior Software Engineer at Acme Corp",
        "summary": "Experienced software engineer specializing in backend systems and cloud infrastructure.",
        "address": {
          "city": "San Francisco",
          "state": "California",
          "country": "United States",
          "full": "San Francisco, California, United States"
        },
        "age": null,
        "total_experience_duration_months": 96,
        "is_working": true,
        "active_experience_title": "Senior Software Engineer",
        "active_experience_description": "Building scalable backend services for Acme Corp's platform.",
        "active_experience_department": "Engineering",
        "active_experience_management_level": "Individual Contributor",
        "is_decision_maker": false,
        "skills": [
          "Python",
          "TypeScript",
          "React",
          "PostgreSQL",
          "AWS",
          "Docker",
          "Kubernetes"
        ],
        "experience": [
          {
            "title": "Senior Software Engineer",
            "company_name": "Acme Corp",
            "start_date": "2022-03",
            "end_date": null,
            "is_current": true,
            "duration_months": 48
          },
          {
            "title": "Software Engineer",
            "company_name": "Globex Inc",
            "start_date": "2018-06",
            "end_date": "2022-02",
            "is_current": false,
            "duration_months": 44
          }
        ],
        "education": [
          {
            "degree": "Master of Science",
            "institution": "Stanford University",
            "major": "Computer Science",
            "gpa": 3.9
          },
          {
            "degree": "Bachelor of Science",
            "institution": "University of California, Berkeley",
            "major": "Computer Science",
            "gpa": 3.7
          }
        ],
        "certifications": [
          "AWS Solutions Architect - Associate",
          "Certified Kubernetes Administrator (CKA)"
        ],
        "awards": [],
        "courses": [],
        "publications": [],
        "patents": [],
        "languages_nested": [
          "English",
          "Spanish"
        ]
      }
    ]
  }
}
```

### Field Reference

The `data` object contains:

| Field | Type | Description |
|---|---|---|
| `total` | integer | Number of URLs requested |
| `found` | integer | Number of profiles successfully found |
| `not_found` | string[] | LinkedIn URLs that could not be found |
| `results` | object[] | Array of profile objects |

Each object in the `results` array contains:

| Field | Type | Description |
|---|---|---|
| `linkedin_url` | string | LinkedIn profile URL |
| `full_name` | string | Candidate's full name |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `headline` | string | LinkedIn headline |
| `summary` | string | Profile summary / about section |
| `address` | object | Location object with `city`, `state`, `country`, and `full` |
| `age` | integer or null | Age (often null) |
| `total_experience_duration_months` | integer | Total professional experience in months |
| `is_working` | boolean | Whether the candidate is currently employed |
| `active_experience_title` | string | Current job title |
| `active_experience_description` | string | Description of current role |
| `active_experience_department` | string | Department of current role |
| `active_experience_management_level` | string | Management level of current role |
| `is_decision_maker` | boolean | Whether the candidate is a decision maker |
| `skills` | string[] | List of professional skills |
| `experience` | object[] | Employment history (see below) |
| `education` | object[] | Education records (see below) |
| `certifications` | string[] | Professional certifications |
| `awards` | string[] | Awards received |
| `courses` | string[] | Courses completed |
| `publications` | string[] | Publications authored |
| `patents` | string[] | Patents held |
| `languages_nested` | string[] | Spoken languages |

**Note:** There is no top-level `company_name` field. To get the current company, find the entry in `experience` where `is_current: true` and read its `company_name`.

**experience[] object:**

| Field | Type | Description |
|---|---|---|
| `title` | string | Job title |
| `company_name` | string | Company name |
| `start_date` | string | Start date (YYYY-MM format) |
| `end_date` | string or null | End date (null if current position) |
| `is_current` | boolean | Whether this is the current position |
| `duration_months` | integer | Duration of the role in months |

**education[] object:**

| Field | Type | Description |
|---|---|---|
| `degree` | string | Degree earned |
| `institution` | string | School or university name |
| `major` | string | Field of study |
| `gpa` | number or null | Grade point average (null if unavailable) |

---

## people-compare

Compare two or more candidates side-by-side. Useful for shortlist evaluations.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-compare`

### Request Example

```json
{
  "linkedin_urls": [
    "https://www.linkedin.com/in/johndoe",
    "https://www.linkedin.com/in/janesmith"
  ]
}
```

### Response Example

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "total_requested": 2,
    "total_found": 2,
    "not_found": [],
    "comparisons": [
      {
        "linkedin_url": "https://www.linkedin.com/in/johndoe",
        "full_name": "John Doe",
        "first_name": "John",
        "last_name": "Doe",
        "headline": "Senior Software Engineer at Acme Corp",
        "summary": "Experienced software engineer specializing in backend systems.",
        "address": {
          "city": "San Francisco",
          "state": "California",
          "country": "United States",
          "full": "San Francisco, California, United States"
        },
        "active_experience_title": "Senior Software Engineer",
        "total_experience_duration_months": 96,
        "is_working": true,
        "skills": [
          "Python",
          "TypeScript",
          "React",
          "PostgreSQL",
          "AWS",
          "Docker",
          "Kubernetes"
        ],
        "experience": [
          {
            "title": "Senior Software Engineer",
            "company_name": "Acme Corp",
            "start_date": "2022-03",
            "end_date": null,
            "is_current": true,
            "duration_months": 48
          }
        ],
        "education": [
          {
            "degree": "Master of Science",
            "institution": "Stanford University",
            "major": "Computer Science",
            "gpa": 3.9
          }
        ],
        "languages_nested": [
          "English",
          "Spanish"
        ]
      },
      {
        "linkedin_url": "https://www.linkedin.com/in/janesmith",
        "full_name": "Jane Smith",
        "first_name": "Jane",
        "last_name": "Smith",
        "headline": "Engineering Manager at Initech",
        "summary": "Engineering leader with a focus on distributed systems.",
        "address": {
          "city": "New York",
          "state": "New York",
          "country": "United States",
          "full": "New York, New York, United States"
        },
        "active_experience_title": "Engineering Manager",
        "total_experience_duration_months": 120,
        "is_working": true,
        "skills": [
          "Java",
          "Go",
          "System Design",
          "Team Leadership",
          "Agile",
          "AWS",
          "Terraform"
        ],
        "experience": [
          {
            "title": "Engineering Manager",
            "company_name": "Initech",
            "start_date": "2021-01",
            "end_date": null,
            "is_current": true,
            "duration_months": 62
          }
        ],
        "education": [
          {
            "degree": "Bachelor of Science",
            "institution": "MIT",
            "major": "Computer Science",
            "gpa": null
          }
        ],
        "languages_nested": [
          "English",
          "Mandarin",
          "French"
        ]
      }
    ]
  }
}
```

### Field Reference

The `data` object contains:

| Field | Type | Description |
|---|---|---|
| `total_requested` | integer | Number of candidates requested |
| `total_found` | integer | Number of candidates successfully found |
| `not_found` | string[] | LinkedIn URLs that could not be found |
| `comparisons` | object[] | Array of candidate profile objects |

Each object in the `comparisons` array has the same structure as a `people-lookup` result (see the people-lookup field reference above). Key fields: `full_name`, `linkedin_url`, `active_experience_title`, `address`, `total_experience_duration_months`, `skills`, `experience`, `education`, `languages_nested`.

**Note:** There is no top-level `company_name` field. To get a candidate's current company, find the entry in `experience` where `is_current: true` and read its `company_name`.

---

## people-bulk-grade

Grade a batch of candidates against a job description. Each candidate receives a score (0-100) and a description of their fit.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-bulk-grade`

### Request Example

```json
{
  "jd": "We are looking for a Senior Backend Engineer with 5+ years of experience in Python and cloud infrastructure (AWS or GCP). Must have strong SQL skills and experience with microservices.",
  "linkedin_urls": [
    "https://www.linkedin.com/in/johndoe",
    "https://www.linkedin.com/in/janesmith",
    "https://www.linkedin.com/in/alexunknown"
  ]
}
```

**Note:** The job description field is `jd` (not `job_description`).

### Response Example

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "jd_preview": "We are looking for a Senior Backend Engineer with 5+ years of experience in Python and cloud infrastructure...",
    "total_requested": 3,
    "total_graded": 2,
    "total_failed": 1,
    "not_found": [
      "https://www.linkedin.com/in/alexunknown"
    ],
    "rankings": [
      {
        "linkedin_url": "https://www.linkedin.com/in/johndoe",
        "total_score": {
          "rating": 92,
          "description": "Strong match. 8 years of Python experience, AWS certified, extensive work with PostgreSQL and microservices at Acme Corp."
        },
        "error": null
      },
      {
        "linkedin_url": "https://www.linkedin.com/in/janesmith",
        "total_score": {
          "rating": 61,
          "description": "Partial match. Engineering management background with solid AWS and system design skills, but primary languages are Java and Go rather than Python."
        },
        "error": null
      }
    ]
  }
}
```

### Field Reference

The `data` object contains:

| Field | Type | Description |
|---|---|---|
| `jd_preview` | string | Truncated preview of the submitted job description |
| `total_requested` | integer | Number of candidates submitted for grading |
| `total_graded` | integer | Number of candidates successfully graded |
| `total_failed` | integer | Number of candidates that could not be graded |
| `not_found` | string[] | LinkedIn URLs that could not be found in the database |
| `rankings` | object[] | Array of grading result objects |

Each object in the `rankings` array contains:

| Field | Type | Description |
|---|---|---|
| `linkedin_url` | string | The LinkedIn URL that was submitted |
| `total_score` | object or null | Score object with `rating` and `description` (null on failure) |
| `total_score.rating` | integer or null | Fit score from 0 (poor) to 100 (excellent) |
| `total_score.description` | string or null | Brief explanation of the rating |
| `error` | string or null | Error message if the profile could not be graded, otherwise null |

---

## people-grade

Grade a single CV/resume text against a job description. Use this when you have the candidate's resume text rather than a LinkedIn URL.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-grade`

### Request Example

```json
{
  "cv": "John Doe\nSenior Software Engineer\n8 years experience in Python, AWS, PostgreSQL...",
  "jd": "We are looking for a Senior Backend Engineer with 5+ years of experience in Python and cloud infrastructure."
}
```

**Note:** The fields are `cv` (resume text) and `jd` (job description text).

### Response Example

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "total_score": {
      "rating": 89,
      "description": "Strong match. Candidate has 8 years of Python experience with extensive AWS and PostgreSQL expertise. Cloud infrastructure skills align well with the role requirements."
    }
  }
}
```

### Field Reference

The `data` object contains:

| Field | Type | Description |
|---|---|---|
| `total_score` | object | Score object with `rating` and `description` |
| `total_score.rating` | integer | Fit score from 0 (poor) to 100 (excellent) |
| `total_score.description` | string | Brief explanation of the rating |

---

## people-fast-search

Search for candidates matching specific filters such as job title, skills, location, and experience level.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-fast-search`

> **Note:** Returns up to 20 results per request. No pagination -- to see more candidates, refine your filters.

### Request Example

```json
{
  "title": "Backend Engineer",
  "skills": ["Python", "AWS"],
  "country": "United States",
  "state": "California",
  "min_experience_months": 60,
  "max_experience_months": 144,
  "is_working": true
}
```

All filter fields are optional. Omit a field or set it to `null` to skip that filter.

### Response Example

```json
{
  "code": 200,
  "message": "ok",
  "data": [
    {
      "linkedin_url": "https://www.linkedin.com/in/johndoe",
      "full_name": "John Doe",
      "first_name": "John",
      "last_name": "Doe",
      "headline": "Senior Software Engineer at Acme Corp",
      "summary": "Experienced backend engineer focused on distributed systems.",
      "address": {
        "city": "San Francisco",
        "state": "California",
        "country": "United States",
        "full": "San Francisco, California, United States"
      },
      "age": null,
      "total_experience_duration_months": 96,
      "is_working": true,
      "active_experience_title": "Senior Software Engineer",
      "active_experience_description": "Building scalable backend services.",
      "active_experience_department": "Engineering",
      "active_experience_management_level": "Individual Contributor",
      "is_decision_maker": false,
      "skills": ["Python", "TypeScript", "React", "PostgreSQL", "AWS"],
      "experience": [
        {
          "title": "Senior Software Engineer",
          "company_name": "Acme Corp",
          "start_date": "2022-03",
          "end_date": null,
          "is_current": true,
          "duration_months": 48
        }
      ],
      "education": [
        {
          "degree": "Master of Science",
          "institution": "Stanford University",
          "major": "Computer Science",
          "gpa": 3.9
        }
      ],
      "awards": [],
      "courses": [],
      "certifications": [],
      "publications": [],
      "patents": [],
      "languages_nested": ["English", "Spanish"]
    },
    {
      "linkedin_url": "https://www.linkedin.com/in/alicejohnson",
      "full_name": "Alice Johnson",
      "first_name": "Alice",
      "last_name": "Johnson",
      "headline": "Backend Engineer at Widgets LLC",
      "summary": "Backend engineer with expertise in Python and event-driven architectures.",
      "address": {
        "city": "Los Angeles",
        "state": "California",
        "country": "United States",
        "full": "Los Angeles, California, United States"
      },
      "age": null,
      "total_experience_duration_months": 72,
      "is_working": true,
      "active_experience_title": "Backend Engineer",
      "active_experience_description": "Developing microservices and event pipelines.",
      "active_experience_department": "Engineering",
      "active_experience_management_level": "Individual Contributor",
      "is_decision_maker": false,
      "skills": ["Python", "Django", "AWS", "Redis", "Kafka"],
      "experience": [
        {
          "title": "Backend Engineer",
          "company_name": "Widgets LLC",
          "start_date": "2020-01",
          "end_date": null,
          "is_current": true,
          "duration_months": 74
        }
      ],
      "education": [],
      "awards": [],
      "courses": [],
      "certifications": [],
      "publications": [],
      "patents": [],
      "languages_nested": ["English"]
    },
    {
      "linkedin_url": "https://www.linkedin.com/in/carlosrivera",
      "full_name": "Carlos Rivera",
      "first_name": "Carlos",
      "last_name": "Rivera",
      "headline": "Staff Engineer at NextGen AI",
      "summary": "Staff engineer specializing in infrastructure and platform engineering.",
      "address": {
        "city": "San Jose",
        "state": "California",
        "country": "United States",
        "full": "San Jose, California, United States"
      },
      "age": null,
      "total_experience_duration_months": 108,
      "is_working": true,
      "active_experience_title": "Staff Engineer",
      "active_experience_description": "Leading platform infrastructure initiatives.",
      "active_experience_department": "Engineering",
      "active_experience_management_level": "Individual Contributor",
      "is_decision_maker": false,
      "skills": ["Python", "Go", "AWS", "Terraform", "gRPC"],
      "experience": [
        {
          "title": "Staff Engineer",
          "company_name": "NextGen AI",
          "start_date": "2019-06",
          "end_date": null,
          "is_current": true,
          "duration_months": 81
        }
      ],
      "education": [],
      "awards": [],
      "courses": [],
      "certifications": [],
      "publications": [],
      "patents": [],
      "languages_nested": ["English", "Spanish"]
    }
  ]
}
```

### Filter Reference

| Field | Type | Description |
|---|---|---|
| `full_name` | string | Candidate's full name (partial match) |
| `title` | string | Job title to search for (partial match) |
| `skills` | string[] | Required skills (all must be present) |
| `country` | string | Country filter (**MUST use full names**, e.g., "United States" not "US") |
| `state` | string | State or region filter (**MUST use full names**, e.g., "California" not "CA") |
| `city` | string | City filter |
| `min_experience_months` | integer | Minimum total experience in months |
| `max_experience_months` | integer | Maximum total experience in months |
| `is_working` | boolean | Filter by current employment status |
| `management_level` | string | Seniority level (e.g., "Specialist", "Senior", "Manager", "Director", "C-Level") |
| `company_name` | string | Current employer name |
| `company_size` | string | Employer size range (e.g., "51-200", "10001+") |
| `company_industry` | string | Employer industry (e.g., "Information Technology") |
| `company_type` | string | Employer type (e.g., "Public Company", "Privately Held") |
| `industry` | string | Candidate's professional industry classification |
| `function` | string | Candidate's professional function (e.g., "Engineering", "Product Management") |
| `employment_type` | string | Employment arrangement (e.g., "Full-Time", "Contract") |

For the complete list of enum values for each field, see `SEARCH_FIELDS.md`.

### Response Fields

The `data` field is an **array** of candidate objects. Each candidate object has the same structure as `people-lookup` results (see people-lookup field reference).

Key fields per candidate: `linkedin_url`, `full_name`, `first_name`, `last_name`, `headline`, `summary`, `address` (object with `city`, `state`, `country`, `full`), `total_experience_duration_months`, `is_working`, `active_experience_title`, `active_experience_description`, `active_experience_department`, `active_experience_management_level`, `is_decision_maker`, `skills`, `experience` (array), `education` (array), `certifications`, `awards`, `courses`, `publications`, `patents`, `languages_nested`.

**Note:** There is no top-level `company_name` field on candidates. To get the current company, find the entry in `experience` where `is_current: true` and read its `company_name`.

---

## people-stats

Get aggregate statistics for a talent pool matching specific filters. Useful for market analysis, workforce composition, and talent supply insights.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-stats`

### Request Example

```json
{
  "title": "Software Engineer",
  "country": "United States",
  "group_by": ["state"]
}
```

**Note:** The `group_by` field is an **array** of dimension strings (e.g., `["state"]`, `["state", "city"]`).

### Response Example

```json
{
  "code": 200,
  "message": "ok",
  "data": {
    "total_matched": 10000,
    "aggregations": {
      "state": [
        {
          "key": "California",
          "count": 6653415
        },
        {
          "key": "New York",
          "count": 3241022
        },
        {
          "key": "Texas",
          "count": 2105893
        }
      ]
    }
  }
}
```

### Field Reference

The `data` object contains:

| Field | Type | Description |
|---|---|---|
| `total_matched` | integer | Total number of candidates matching the filters |
| `aggregations` | object | Breakdown by each dimension in `group_by` |

Each key in `aggregations` corresponds to a `group_by` dimension and contains an array of `{ "key": string, "count": integer }` objects.

---

## Error Responses

All endpoints return standard error responses in this format:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "At least one of linkedin_url or email must be provided."
  }
}
```

Common error codes:

| HTTP Status | Code | Description |
|---|---|---|
| 400 | `invalid_request` | Missing or malformed parameters |
| 401 | `unauthorized` | Missing or invalid API token |
| 404 | `not_found` | Requested resource does not exist |
| 429 | `rate_limited` | Too many requests; retry after the indicated interval |
| 500 | `internal_error` | Unexpected server error |
