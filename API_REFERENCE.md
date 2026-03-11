# Mira Recruiting Skill API Reference

Base URL: `https://mira-api.openjobs-ai.com/v1/`

All endpoints accept JSON request bodies and return JSON responses. Authentication is required via an `Authorization: Bearer <token>` header on every request.

---

## Endpoints Overview

| Endpoint | Method | Path | Description |
|---|---|---|---|
| **people-lookup** | POST | `/people-lookup` | Retrieve a detailed profile for a single person by LinkedIn URL or email |
| **people-compare** | POST | `/people-compare` | Compare two or more candidates side-by-side |
| **people-bulk-grade** | POST | `/people-bulk-grade` | Grade a batch of candidates against a job description |
| **people-fast-search** | POST | `/people-fast-search` | Search for candidates matching specific filters |

---

## people-lookup

Retrieve a detailed profile for a single person by LinkedIn URL or email.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-lookup`

### Request Example

```json
{
  "linkedin_url": "https://www.linkedin.com/in/johndoe",
  "email": null
}
```

You may provide `linkedin_url`, `email`, or both. At least one identifier is required.

### Response Example

```json
{
  "full_name": "John Doe",
  "headline": "Senior Software Engineer at Acme Corp",
  "linkedin_url": "https://www.linkedin.com/in/johndoe",
  "country": "United States",
  "state": "California",
  "city": "San Francisco",
  "experience_months": 96,
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
  "active_title": "Senior Software Engineer",
  "company_name": "Acme Corp",
  "work_history": [
    {
      "title": "Senior Software Engineer",
      "company": "Acme Corp",
      "start_date": "2022-03",
      "end_date": null,
      "duration_months": 48
    },
    {
      "title": "Software Engineer",
      "company": "Globex Inc",
      "start_date": "2018-06",
      "end_date": "2022-02",
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
    "AWS Solutions Architect – Associate",
    "Certified Kubernetes Administrator (CKA)"
  ],
  "languages": [
    "English",
    "Spanish"
  ]
}
```

### Field Reference

| Field | Type | Description |
|---|---|---|
| `full_name` | string | Candidate's full name |
| `headline` | string | LinkedIn headline |
| `linkedin_url` | string | LinkedIn profile URL |
| `country` | string | Country of residence |
| `state` | string | State or region |
| `city` | string | City of residence |
| `experience_months` | integer | Total professional experience in months |
| `is_working` | boolean | Whether the candidate is currently employed |
| `skills` | string[] | List of professional skills |
| `active_title` | string | Current job title |
| `company_name` | string | Current employer |
| `work_history` | object[] | Employment history (see below) |
| `education` | object[] | Education records (see below) |
| `certifications` | string[] | Professional certifications |
| `languages` | string[] | Spoken languages |

**work_history[] object:**

| Field | Type | Description |
|---|---|---|
| `title` | string | Job title |
| `company` | string | Company name |
| `start_date` | string | Start date (YYYY-MM format) |
| `end_date` | string or null | End date (null if current position) |
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
  "candidates": [
    {
      "full_name": "John Doe",
      "linkedin_url": "https://www.linkedin.com/in/johndoe",
      "active_title": "Senior Software Engineer",
      "company_name": "Acme Corp",
      "highest_education": {
        "degree": "Master of Science",
        "institution": "Stanford University"
      },
      "skills": [
        "Python",
        "TypeScript",
        "React",
        "PostgreSQL",
        "AWS",
        "Docker",
        "Kubernetes"
      ],
      "languages": [
        "English",
        "Spanish"
      ]
    },
    {
      "full_name": "Jane Smith",
      "linkedin_url": "https://www.linkedin.com/in/janesmith",
      "active_title": "Engineering Manager",
      "company_name": "Initech",
      "highest_education": {
        "degree": "Bachelor of Science",
        "institution": "MIT"
      },
      "skills": [
        "Java",
        "Go",
        "System Design",
        "Team Leadership",
        "Agile",
        "AWS",
        "Terraform"
      ],
      "languages": [
        "English",
        "Mandarin",
        "French"
      ]
    }
  ]
}
```

### Field Reference

Each object in the `candidates` array contains:

| Field | Type | Description |
|---|---|---|
| `full_name` | string | Candidate's full name |
| `linkedin_url` | string | LinkedIn profile URL |
| `active_title` | string | Current job title |
| `company_name` | string | Current employer |
| `highest_education` | object | Highest degree obtained (`degree` and `institution`) |
| `skills` | string[] | List of professional skills |
| `languages` | string[] | Spoken languages |

---

## people-bulk-grade

Grade a batch of candidates against a job description. Each candidate receives a rating and a short description of their fit.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-bulk-grade`

### Request Example

```json
{
  "job_description": "We are looking for a Senior Backend Engineer with 5+ years of experience in Python and cloud infrastructure (AWS or GCP). Must have strong SQL skills and experience with microservices.",
  "linkedin_urls": [
    "https://www.linkedin.com/in/johndoe",
    "https://www.linkedin.com/in/janesmith",
    "https://www.linkedin.com/in/alexunknown"
  ]
}
```

### Response Example

```json
{
  "results": [
    {
      "linkedin_url": "https://www.linkedin.com/in/johndoe",
      "full_name": "John Doe",
      "rating": 9,
      "description": "Strong match. 8 years of Python experience, AWS certified, extensive work with PostgreSQL and microservices at Acme Corp.",
      "error": null
    },
    {
      "linkedin_url": "https://www.linkedin.com/in/janesmith",
      "full_name": "Jane Smith",
      "rating": 6,
      "description": "Partial match. Engineering management background with solid AWS and system design skills, but primary languages are Java and Go rather than Python.",
      "error": null
    },
    {
      "linkedin_url": "https://www.linkedin.com/in/alexunknown",
      "full_name": null,
      "rating": null,
      "description": null,
      "error": "Profile not found in database"
    }
  ]
}
```

### Field Reference

Each object in the `results` array contains:

| Field | Type | Description |
|---|---|---|
| `linkedin_url` | string | The LinkedIn URL that was submitted |
| `full_name` | string or null | Candidate's name (null on failure) |
| `rating` | integer or null | Fit score from 1 (poor) to 10 (excellent), null on failure |
| `description` | string or null | Brief explanation of the rating, null on failure |
| `error` | string or null | Error message if the profile could not be graded, otherwise null |

---

## people-fast-search

Search for candidates matching specific filters such as job title, skills, location, and experience level.

**URL:** `POST https://mira-api.openjobs-ai.com/v1/people-fast-search`

> **Note:** Returns up to 20 results per request. No pagination — to see more candidates, refine your filters.

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
  "total": 3,
  "candidates": [
    {
      "full_name": "John Doe",
      "linkedin_url": "https://www.linkedin.com/in/johndoe",
      "active_title": "Senior Software Engineer",
      "company_name": "Acme Corp",
      "city": "San Francisco",
      "state": "California",
      "country": "United States",
      "experience_months": 96,
      "skills": ["Python", "TypeScript", "React", "PostgreSQL", "AWS"]
    },
    {
      "full_name": "Alice Johnson",
      "linkedin_url": "https://www.linkedin.com/in/alicejohnson",
      "active_title": "Backend Engineer",
      "company_name": "Widgets LLC",
      "city": "Los Angeles",
      "state": "California",
      "country": "United States",
      "experience_months": 72,
      "skills": ["Python", "Django", "AWS", "Redis", "Kafka"]
    },
    {
      "full_name": "Carlos Rivera",
      "linkedin_url": "https://www.linkedin.com/in/carlosrivera",
      "active_title": "Staff Engineer",
      "company_name": "NextGen AI",
      "city": "San Jose",
      "state": "California",
      "country": "United States",
      "experience_months": 108,
      "skills": ["Python", "Go", "AWS", "Terraform", "gRPC"]
    }
  ]
}
```

### Filter Reference

| Field | Type | Description |
|---|---|---|
| `title` | string | Job title to search for (partial match) |
| `skills` | string[] | Required skills (all must be present) |
| `country` | string | Country filter |
| `state` | string | State or region filter |
| `city` | string | City filter |
| `min_experience_months` | integer | Minimum total experience in months |
| `max_experience_months` | integer | Maximum total experience in months |
| `is_working` | boolean | Filter by current employment status |

### Response Fields

| Field | Type | Description |
|---|---|---|
| `total` | integer | Number of candidates returned |
| `candidates` | object[] | Array of matching candidate summaries |

Each candidate object includes: `full_name`, `linkedin_url`, `active_title`, `company_name`, `city`, `state`, `country`, `experience_months`, and `skills`.

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
