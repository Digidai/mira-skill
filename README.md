# Mira — AI Recruiting Skill for Claude Code

Mira is a Claude Code Skill that gives your AI coding assistant recruiting superpowers. Search for candidates, grade resumes against job descriptions, compare talent side-by-side, and analyze hiring markets — all without leaving your terminal.

Powered by [OpenJobsAI](https://www.openjobs-ai.com).

## Quick Start

### Install

```bash
# Clone the repo into your Claude Code skills directory
git clone https://github.com/Digidai/mira-skill.git ~/.claude/skills/mira
```

Or manually:

```bash
mkdir -p ~/.claude/skills/mira
cp SKILL.md API_REFERENCE.md SEARCH_FIELDS.md WORKFLOWS.md TROUBLESHOOTING.md ~/.claude/skills/mira/
```

### Use

Once installed, just talk to Claude Code naturally:

```
Find me senior backend engineers in San Francisco who know Python and Kubernetes
```

Or invoke explicitly:

```
/mira search for ML engineers in Berlin with PyTorch experience
```

## What Can Mira Do?

| Capability | Example | API Endpoint |
|---|---|---|
| **Search candidates** | "Find React engineers in NYC" | `people-fast-search` |
| **Grade against JD** | "Score this LinkedIn profile against our JD" | `people-bulk-grade` / `people-grade` |
| **Lookup profiles** | "Show me details for linkedin.com/in/johndoe" | `people-lookup` |
| **Compare candidates** | "Compare these two candidates side-by-side" | `people-compare` |
| **Find similar** | "Find someone like this person for a replacement hire" | `people-lookup` + `people-fast-search` |
| **Company talent map** | "Show me engineers at Stripe" | `people-fast-search` + `people-stats` |
| **Market analytics** | "What's the AI/ML talent market like in Austin?" | `people-stats` |

## File Structure

```
mira/
├── SKILL.md              # Core instructions — loaded on every activation (~2,000 tokens)
├── API_REFERENCE.md      # Endpoint URLs, request/response JSON examples
├── SEARCH_FIELDS.md      # Filter field types, enum values, industry/function lists
├── WORKFLOWS.md          # 8 recruiting workflows + parameter construction guide
└── TROUBLESHOOTING.md    # Error handling, empty results, location 422 fixes
```

The skill uses **progressive disclosure** — only `SKILL.md` is loaded on activation. Other files are read on-demand by Claude when needed, saving context window tokens.

## Key Features

- **Natural language to API translation** — Say "find senior Python devs in California" and Mira translates it to structured filters automatically
- **Smart experience ranges** — "5+ years" for a Senior role searches 5-25 years; for an IC role, 5-15 years. No more excluding overqualified candidates
- **Decision tree routing** — Automatically picks the right API endpoint based on your input (LinkedIn URL vs CV text vs search criteria)
- **Iterative refinement** — If results aren't great, Mira suggests filter adjustments and re-runs automatically (up to 3 rounds)
- **Error self-healing** — Auto-expands location abbreviations ("CA" → "California"), retries on transient errors, handles partial batch failures gracefully
- **Candidate display format** — Clean, scannable output with name, role, company, experience, and match reason

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- OpenJobsAI API key (configured via environment variable or config file)

## Examples

### Search + Grade in one step

```
Find and rank the top 10 backend engineers in Berlin for this role:

JD: Senior Backend Engineer, 5+ years Python, AWS, microservices, strong SQL
```

Mira will search, collect LinkedIn URLs from results, grade them all against the JD, and present a ranked list.

### Replacement hire

```
Our lead engineer is leaving. Find similar candidates:
https://www.linkedin.com/in/janepark
```

Mira looks up Jane's profile, extracts her key skills/experience/location, and searches for matching candidates.

### Batch grading

```
Grade these candidates for our Senior Python Engineer role:

https://www.linkedin.com/in/candidate1
https://www.linkedin.com/in/candidate2
https://www.linkedin.com/in/candidate3

JD: ...
```

## Design Principles

This skill follows [Anthropic's Skill design best practices](https://docs.anthropic.com/en/docs/claude-code/skills):

1. **Progressive disclosure** — Main file < 1,500 words; reference files loaded on-demand
2. **Decision tree routing** — Structured if-then branches, not paragraph instructions
3. **Verification loops** — Every API call has error handling with retry/escalation strategy
4. **MUST/NEVER language** — Directive language for critical rules (location format, URL validation)
5. **Concrete examples** — JSON request/response examples for every endpoint

## License

Proprietary — [OpenJobsAI](https://www.openjobs-ai.com)
