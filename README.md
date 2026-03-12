# Mira — AI Recruiting Skill for Claude Code

[![ClawHub](https://img.shields.io/badge/ClawHub-mira-blue?logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJ3aGl0ZSI+PHBhdGggZD0iTTEyIDJDNi40OCAyIDIgNi40OCAyIDEyczQuNDggMTAgMTAgMTAgMTAtNC40OCAxMC0xMFMxNy41MiAyIDEyIDJ6Ii8+PC9zdmc+)](https://clawhub.ai)
[![Version](https://img.shields.io/badge/version-1.3.0-green)](https://github.com/Digidai/mira-skill)
[![License](https://img.shields.io/badge/license-MIT--0-green)](https://opensource.org/license/mit-0)
[![Platform](https://img.shields.io/badge/platform-Claude%20Code%20%7C%20Codex%20%7C%20Cursor%20%7C%20Gemini%20CLI-purple)]()

Mira is a Claude Code Skill that gives your AI coding assistant recruiting superpowers. Search for candidates, grade resumes against job descriptions, compare talent side-by-side, and analyze hiring markets — all without leaving your terminal.

Powered by [OpenJobsAI](https://www.openjobs-ai.com).

---

## Installation

### Option 1: ClawHub CLI (Recommended)

```bash
# Install ClawHub CLI if you haven't
npm install -g clawhub

# Install Mira skill
clawhub install mira
```

The skill will be installed to your current project's `./skills/` directory and registered in `.clawhub/lock.json`.

### Option 2: ClawHub CLI from GitHub URL

```bash
clawhub install https://github.com/Digidai/mira-skill
```

### Option 3: Git Clone (Manual)

```bash
git clone https://github.com/Digidai/mira-skill.git ~/.claude/skills/mira
```

### Option 4: Direct Copy

```bash
mkdir -p ~/.claude/skills/mira
curl -sL https://github.com/Digidai/mira-skill/archive/refs/heads/master.tar.gz \
  | tar xz --strip-components=1 -C ~/.claude/skills/mira
```

### Setup API Key

Mira requires an API key. Set it as an environment variable:

```bash
export MIRA_KEY="your-api-key-here"
```

Or add it to your shell profile (`~/.zshrc`, `~/.bashrc`):

```bash
echo 'export MIRA_KEY="your-api-key-here"' >> ~/.zshrc
```

Alternatively, store the key in the config file at `~/.config/mira/api_key`.

On first use, if the key is not found, Mira will walk you through the setup process.

### Verify Installation

```
/mira
```

Claude will recognize the skill and show you available capabilities.

---

## Usage

Just talk to Claude Code naturally:

```
Find me senior backend engineers in San Francisco who know Python and Kubernetes
```

Or invoke explicitly with the slash command:

```
/mira search for ML engineers in Berlin with PyTorch experience
```

---

## Capabilities

| Capability | Example | API Endpoint |
|---|---|---|
| **Search candidates** | "Find React engineers in NYC" | `people-fast-search` |
| **Grade against JD** | "Score this LinkedIn profile against our JD" | `people-bulk-grade` / `people-grade` |
| **Lookup profiles** | "Show me details for linkedin.com/in/johndoe" | `people-lookup` |
| **Compare candidates** | "Compare these two candidates side-by-side" | `people-compare` |
| **Find similar** | "Find someone like this person for a replacement hire" | `people-lookup` + `people-fast-search` |
| **Company talent map** | "Show me engineers at Stripe" | `people-fast-search` + `people-stats` |
| **Market analytics** | "What's the AI/ML talent market like in Austin?" | `people-stats` |

---

## File Structure

```
mira-skill/
├── SKILL.md              # Core instructions — loaded on every activation (~1,300 words)
├── API_REFERENCE.md      # Endpoint URLs, request/response JSON examples
├── SEARCH_FIELDS.md      # 16 filter fields, enum values, industry/function lists
├── WORKFLOWS.md          # 8 recruiting workflows + parameter construction guide
├── TROUBLESHOOTING.md    # Error handling, empty results, location 422 fixes
├── .clawhubignore        # Files excluded from ClawHub publishing
├── LICENSE               # MIT-0 (required by ClawHub)
└── README.md             # This file
```

### Progressive Disclosure Architecture

The skill uses a **progressive disclosure** design to minimize context window usage:

```
Stage 1: description field (~100 tokens)
  → Claude reads ONLY the description to decide if Mira matches the user's request

Stage 2: SKILL.md body (~2,000 tokens)
  → On match, the full SKILL.md is loaded: decision tree, display formats, experience ranges

Stage 3: Reference files (on-demand)
  → Claude reads WORKFLOWS.md, API_REFERENCE.md, SEARCH_FIELDS.md, or TROUBLESHOOTING.md
    only when it needs specific details (endpoint schemas, enum values, error handling)
```

**Result:** A simple search query only costs ~2,000 tokens for skill instructions, instead of ~3,650 tokens if everything were in a single file (45% savings).

---

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

JD: We need a Senior Backend Engineer with 5+ years Python, AWS, microservices...
```

### Company talent map

```
Show me the engineering team at Stripe — breakdown by role and seniority
```

### Iterative search

```
Find senior data scientists in Germany who know PyTorch
```

If results are sparse, Mira automatically suggests broadening filters (removing a skill, expanding from city to state/country) and re-runs — up to 3 rounds of refinement.

---

## Key Features

- **Natural language to API translation** — Say "find senior Python devs in California" and Mira translates it to structured API filters automatically
- **Smart experience ranges** — "5+ years" for a Senior role searches 5-25 years; for an IC role, 5-15 years. Context-aware, never excludes overqualified candidates
- **Decision tree routing** — Automatically picks the right API endpoint based on input (LinkedIn URL vs CV text vs search criteria)
- **Iterative refinement** — Suggests filter adjustments and re-runs when results need improvement (up to 3 rounds)
- **Error self-healing** — Auto-expands location abbreviations ("CA" → "California"), retries on transient errors, handles partial batch failures gracefully
- **Candidate display format** — Clean, scannable output: `**Name** — Role @ Company, X yrs exp, Location · Match reason`

---

## ClawHub / OpenClaw Integration

### Frontmatter

Mira's `SKILL.md` includes both `clawdbot` (Claude Code native) and `openclaw` (ClawHub CLI) metadata namespaces. Only [officially supported fields](https://github.com/openclaw/clawhub/blob/main/docs/skill-format.md) are used:

```yaml
metadata:
  clawdbot:
    emoji: "\U0001F50D"
    always: false
  openclaw:
    emoji: "\U0001F50D"       # Display icon in CLI listings
    always: false              # Only activate when triggered, not on every message
    homepage: https://www.openjobs-ai.com
    primaryEnv: MIRA_KEY       # Key env var — CLI prompts for it on install
    requires:
      env:
        - MIRA_KEY             # Validated before skill activation
    os:
      - macos
      - linux
      - windows
```

**Official OpenClaw metadata fields:** `emoji`, `always`, `homepage`, `primaryEnv`, `requires` (env/bins/anyBins/config), `os`, `install`, `nix`, `config`, `skillKey`. Non-standard fields like `tags`, `category`, `author`, `license` are ignored by the CLI and should not be used.

### Publishing to ClawHub

If you fork and customize this skill, you can publish to ClawHub:

```bash
# Authenticate with ClawHub
clawhub login

# Navigate to your skill directory
cd ~/.claude/skills/mira

# Publish with explicit version (required — must be valid semver)
clawhub publish . --version 1.3.0
```

ClawHub will:
1. Validate the `SKILL.md` frontmatter against the [skill format spec](https://github.com/openclaw/clawhub/blob/main/docs/skill-format.md)
2. Index the `description` field using OpenAI `text-embedding-3-small` for semantic search
3. Generate `.clawhub/origin.json` automatically (do not create this file manually)
4. Make it discoverable via `clawhub search recruiting` or any related query

**License:** All skills published to ClawHub are distributed under the [MIT-0 license](https://opensource.org/license/mit-0) (MIT No Attribution). This is a ClawHub requirement — proprietary skills cannot be published to the registry.

### ClawHub CLI Commands

| Command | Description | Example |
|---|---|---|
| `clawhub install <name>` | Install a skill by name or GitHub URL | `clawhub install mira` |
| `clawhub search <query>` | Semantic search across all published skills | `clawhub search "recruiting talent sourcing"` |
| `clawhub inspect <name>` | View skill metadata and frontmatter details | `clawhub inspect mira` |
| `clawhub explore <name>` | Browse skill files and structure | `clawhub explore mira` |
| `clawhub list` | List installed skills in current project | `clawhub list` |
| `clawhub update <name>` | Update an installed skill to latest version | `clawhub update mira` |
| `clawhub sync` | Sync skill versions across environments | `clawhub sync --bump --changelog` |
| `clawhub publish <path>` | Publish to the ClawHub registry | `clawhub publish . --version 1.3.0` |
| `clawhub uninstall <name>` | Remove an installed skill | `clawhub uninstall mira` |
| `clawhub star <name>` | Star a skill on the registry | `clawhub star mira` |

### Semantic Search Optimization

The `description` field is optimized for ClawHub's embedding-based search. It contains 17+ trigger keywords covering all recruiting scenarios:

> recruiting, talent-acquisition, source candidates, search for talent, grade applicants, score resumes, evaluate CVs, staffing analytics, compare candidates, replacement hires, headhunting, recruiter workflows, candidate searches, filtering results, scoring CVs, hiring-market insights

This means users can find Mira by searching for any of these terms on ClawHub, not just exact keyword matches.

### Compatible Platforms

The `openclaw` metadata format is compatible with:

| Platform | Installation |
|---|---|
| **Claude Code** | `clawhub install mira` or `git clone` to `~/.claude/skills/mira` |
| **Codex** | `clawhub install mira` |
| **Cursor** | `clawhub install mira` |
| **Gemini CLI** | `clawhub install mira` |

---

## Design Principles

This skill follows [Anthropic's Skill design best practices](https://docs.anthropic.com/en/docs/claude-code/skills) and [ClawHub's format specification](https://github.com/openclaw/clawhub):

1. **Progressive disclosure** — Main file < 1,500 words; 4 reference files loaded on-demand
2. **Decision tree routing** — Structured if-then branches, not paragraph instructions
3. **Verification loops** — Every API call has error handling with retry/escalation strategy
4. **MUST/NEVER language** — Directive language for critical rules (location format, URL validation)
5. **Concrete examples** — JSON request/response examples for every endpoint
6. **Dual namespace** — `clawdbot` + `openclaw` metadata for maximum platform compatibility (only official fields used)
7. **Semantic search optimized** — Description packed with 17+ recruiting trigger keywords for ClawHub's embedding-based discovery

---

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI (or compatible platform: Codex, Cursor, Gemini CLI)
- Mira API key — set as `MIRA_KEY` environment variable or stored in `~/.config/mira/api_key`
- No additional binary dependencies

---

## OpenJobs AI Skills Ecosystem

Mira is the core skill in the OpenJobs AI suite. Planned extensions:

| Skill | Status | Description |
|---|---|---|
| **mira** | Released | Candidate search, grading, analytics |
| **mira-outreach** | Planned (P0) | Generate personalized outreach messages based on candidate profiles |
| **mira-pipeline** | Planned (P1) | Track candidates through hiring stages with persistent local files |
| **mira-similar** | Planned (P1) | Find similar candidates for replacement hires |
| **mira-talent-map** | Planned (P1) | Map company workforce by role, level, and geography |
| **mira-market-intel** | Planned (P1) | Deep talent market analysis with multi-dimensional insights |
| **mira-jd-writer** | Planned (P2) | Generate and optimize job descriptions |

---

## Contributing

Contributions welcome! If you'd like to improve Mira:

1. Fork this repository
2. Make your changes
3. Test with Claude Code: `cp -r . ~/.claude/skills/mira && claude`
4. Submit a pull request

### Skill Design Guidelines

When modifying the skill files, keep these rules in mind:

- **SKILL.md** must stay under 1,500 words (~2,000 tokens)
- Use **MUST/NEVER** for critical rules, not "should" or "try to"
- Every API endpoint needs a complete JSON response example
- Decision tree branches must cover all edge cases
- Location format rule must be repeated inline (most common error source)

---

## License

[MIT-0](https://opensource.org/license/mit-0) (MIT No Attribution) — Required by [ClawHub](https://clawhub.ai) for published skills.

API service powered by [OpenJobsAI](https://www.openjobs-ai.com).
