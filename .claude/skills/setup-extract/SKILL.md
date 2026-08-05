---
description: Extract structured information from code repositories, project docs, or design documents into knowledge base extractions
user-invocable: true
---

# /setup-extract

**User input:** `$ARGUMENTS`

Parse `$ARGUMENTS`:
- Path to a code repository (e.g., `~/repos/some-service`) → read it as a project
- Path to a document (e.g., `knowledge_base/papers/design_doc.md`, a PDF, a `.tex`) → read that file
- Multiple paths separated by spaces → batch mode (process each sequentially)
- Empty → ask the user for the project path, or to describe the project directly

---

## Startup

1. Read `CLAUDE.md` — check KB Corrections Log for known issues
2. Read `config.md` — load Personal Info, Provenance Flags (ownership and scope framings)
3. Read `knowledge_base/extractions/_INVENTORY.md` — see what's already extracted, avoid duplicates

If the project is already in the inventory:
- Show the existing extraction path
- Ask: "This project is already extracted. Re-extract (overwrite) or skip?"
- Wait for user response before proceeding

---

## Phase 1: Read & Understand the Project

Read the source using the appropriate method:
- **Code repository:** read README, package manifests, CI config, top-level architecture,
  and `git log --author` to see the user's actual commit footprint. Do NOT read the whole
  tree — sample enough to characterize the system honestly.
- **Design/project docs (md, PDF, .tex):** read directly
- **Both:** prefer the code for what was built, the doc for why and at what scale

**While reading, collect:**
1. Project name, what it does in one sentence, timeframe, and whether it shipped
2. Scope signals — team size, the user's commit share, who else contributed
3. Status (check `config.md` Provenance Flags first, then infer: shipped / internal / prototype / archived)
4. Tech stack — languages, frameworks, datastores, infra, CI, deployment target
5. Quantitative results — throughput, latency, cost, uptime, users, adoption, test coverage,
   build time, incident reduction. Anything with a number and a measurement method.
6. Engineering-difficulty signals — scale, concurrency, correctness constraints, migrations
   under load, backwards compatibility, hard debugging
7. Collaboration indicators — other teams, external dependencies, code review ownership
8. Anything conspicuously absent (no tests, prototype-only, never deployed) — this matters
   for honest framing later

Progress: "Reading project... [name], [primary stack], [status]"

---

## Phase 2: Clarify the User's Role

Ownership is the main fabrication risk in engineering resumes. Repository access does not
establish authorship, and commit counts do not establish design ownership. Always ask:

**Questions to ask (skip any that are already clear from the source):**
1. "What did you personally build here, versus what the team built around you?"
2. "Did you own the design, the implementation, or both? Was there a tech lead above you?"
3. "Did this ship to real users? At what scale — and is that number measured or estimated?"
4. "Which numbers can you defend in an interview if someone asks how you measured them?"
5. "Anything here that should NOT appear on your resume? (NDA, someone else's work, a failure
   you would rather not open up)"

### >>>>>> MANDATORY STOP — DO NOT PROCEED <<<<<<
Present your understanding of the project and ask the clarifying questions above.
**You MUST wait for the user's explicit text response before continuing.**

---

## Phase 3: Write Extraction

Create the extraction file at `knowledge_base/extractions/<project_descriptor>.md`

**Naming convention:** `<year>_<project_2-3_word_descriptor>.md`
- Examples: `2025_payments_gateway.md`, `2024_k8s_migration.md`
- Normalize to lowercase with underscores

**Extraction format:**

```markdown
# [Project Name] — [one-line description]

## Metadata
- **Employer / context:** [company, side project, open source]
- **Timeframe:** [start — end]
- **Team:** [solo | N engineers | cross-team]
- **User's role:** [sole owner / design owner / implementer / contributor]
- **Status:** [shipped | internal | prototype | archived]
- **Repo:** [URL if public, else "private"]

## Stack & Architecture
- **Languages:** [e.g., Go, TypeScript, Python]
- **Frameworks/runtime:** [e.g., React, FastAPI, Node]
- **Data:** [e.g., Postgres, Redis, Kafka, S3]
- **Infra:** [e.g., AWS, Kubernetes, Terraform, GitHub Actions]
- **Architecture shape:** [monolith / microservices / event-driven / batch pipeline / library]

## Key Results
[Number each result. Include quantitative metrics AND how they were measured.]
1. [Result with numbers — e.g., "Cut p99 checkout latency 840ms → 210ms, measured via Datadog over 30d"]
2. [Result — e.g., "Migrated 40 services to the new CI pipeline, build time 11min → 3min"]
3. [...]

## Engineering Difficulty
[What was genuinely hard — this is what senior interviewers probe]
- [e.g., "Zero-downtime schema migration on a 400M-row table under live write load"]
- [e.g., "Exactly-once delivery across an at-least-once broker"]

## Collaboration & Scope
- **Other teams/dependencies:** [who else was involved]
- **User's specific contribution:** [from Phase 2 clarification]
- **Solo vs. shared work:** [what the user did alone vs. with others]

## Provenance Notes
- **Ownership status:** [matches config.md if listed there]
- **Safe to claim:** [what the user can put on a resume without hedging]
- **Needs hedging:** [claims that require "contributed to" or "supported" framing]
- **Do NOT claim:** [results from collaborators, claims that would be overclaiming]

## Resume Bullet Seeds
[3-5 draft bullets in STAR format. These are seeds, not final text.]
[Use full-ownership verbs only for sole-contributor work. Hedge for shared work.]
1. [Action verb] + [what was done] + [quantitative result/impact]
2. [Action verb] + [method/tool developed] + [what it enabled]
3. [Action verb] + [scope — e.g., "across N systems"] + [outcome]
4. [Optional: collaboration-framed bullet]
5. [Optional: tool/infrastructure bullet]
```

Save the file. Show the user the complete extraction.

Progress: "Writing extraction for [short title]... [N] results identified, [M] bullet seeds drafted"

---

## Phase 4: Update Inventory

Read and update `knowledge_base/extractions/_INVENTORY.md`.

Add a row to the inventory table:

```
| [filename] | [short title] | [user's role] | [status] | [primary methods] | [date extracted] |
```

Present the updated inventory entry to the user.

---

## Phase 5: Next Steps

After extraction is complete, present:

1. **Extraction summary:** [N] methods, [M] quantitative results, [K] bullet seeds
2. **Provenance flags:** Any items that need special handling
3. **Suggested next action:**
   - If more papers to extract: "Run `/setup-extract [next paper path]`"
   - If all papers done: "Run `/setup-build-kb` to synthesize extractions into experience files and bundles"

### >>>>>> MANDATORY STOP <<<<<<
Present extraction summary. Wait for user feedback or next paper.
**You MUST wait for the user's explicit text response before continuing.**

---

## Batch Mode

If `$ARGUMENTS` contains multiple file paths:
1. Process each paper through Phases 1-4 sequentially
2. Ask Phase 2 clarifying questions for ALL papers at once (grouped) before writing any extractions
3. After all extractions: present combined inventory update and summary
4. Single STOP at the end (not per paper)
