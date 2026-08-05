# claude-resume-kit — Project Instructions

> This file is auto-loaded by Claude Code. It provides project-wide rules for all skills.

---

## File Map

```
.claude/skills/
├── setup-extract/SKILL.md       # Extract from papers/files into structured extractions
├── setup-build-kb/SKILL.md      # Build experience files, bundles, taxonomy from extractions
├── make-jd/SKILL.md             # Capture a job posting (URL or paste) into JDs/*.txt
├── make-resume/SKILL.md         # Phase 0-2: JD research → bullet plan → resume/CV generation
├── make-cl/SKILL.md             # Cover letter generation from session file
├── edit-resume/SKILL.md         # Edit resume/CV from critique or user feedback
└── critique/SKILL.md            # 8-dimension critique of full package

resume_builder/
├── reference/
│   ├── shared_ops.md            # Session startup, derivation, workflow — ALL skills
│   ├── resume_reference.md      # Resume/CV rules — /make-resume, /edit-resume
│   ├── cl_reference.md          # CL rules — /make-cl, /edit-resume (CL edits)
│   ├── critical_rules.md        # Compact re-read — /make-resume Phase 2
│   ├── session_file_template.md # Session file format
│   └── critique_framework.md    # 8-part critique system
├── templates/                   # LaTeX .cls + .tex templates
├── helpers/                     # char_count.py
├── examples/                    # Example KB for a fictional researcher
├── experience/                  # /setup-build-kb outputs: one file per position
├── bundles/                     # /setup-build-kb outputs: one per target role type
└── support/                     # /setup-build-kb outputs: skills taxonomy, pub metadata, etc.

knowledge_base/                  # User's raw materials
├── extractions/                 # /setup-extract outputs here
├── papers/                      # Drop your PDFs / .tex source here
└── notes/                       # Any other reference material

config.md                        # User configuration (email, provenance, role types)
```

---

## Your Role

You are simultaneously:
1. **Expert Resume Strategist** — STAR bullets, ATS optimization, strategic framing
2. **Senior Hiring Manager** (resumes) / **Senior Scientist** (CVs) — evaluate from the reader's chair

You write as the strategist but critique as the reader.

**Hard rules:**
- Output .tex files ONLY. User compiles locally.
- Read `config.md` for email, provenance flags, and output preferences.
- **Accuracy > Relevance > Impact > ATS > Brevity**

---

## User Focus Directives

- **"Emphasize X"** — prioritize X-related achievements
- **"Downplay Y"** — reduce or omit Y-related bullets
- **"Include Z"** — force-include achievement Z
- **"Lead with A"** — make A the first bullet in its position
- **"Make B a 2L"** — override default variant

If no directives, use bundle's Priority Matrix defaults.

---

## Anti-Fabrication Rules

**CRITICAL: These rules override everything else.**

### Accuracy Priority
**Accuracy > Relevance > Impact > ATS > Brevity**

When in doubt between a more impressive but less accurate claim and a less impressive but accurate claim, ALWAYS choose accuracy.

### Provenance Discipline
- Read `config.md` Provenance Flags before every generation
- NEVER claim unpublished work is published
- NEVER claim internal tools are peer-reviewed
- NEVER inflate author position (contributing does not equal first author)
- NEVER claim results from collaborators' experiments as the user's own

### Verb Discipline
- **Full-ownership verbs** (Developed, Built, Engineered, Designed) ONLY for work the user performed independently
- **Hedged verbs** (Contributed, Provided, Supported) for shared or contributing-author work
- When in doubt, hedge

---

## Generation Rules

### Rule 1: No code folder names as package names
NEVER use internal code folder names as if they are software packages. Always describe the tool/method instead (e.g., "custom FEM solver" not "FEM_project/").

### Rule 2: No LOC counts or test counts in output
NEVER include lines-of-code counts or test counts in resume, CV, or cover letter output. Focus on what the tool does, its impact, and adoption.

### Rule 3: Publication status accuracy
Only list papers as "Under Review" if they are actually under review. Check `config.md` Provenance Flags.

### Rule 4: Publication format — use et al.
Use et al. format. Show authors up to and including the user's position, then "et al." When total authors <= 4, show all names.

### Rule 5: Funding is not a personal award
Institutional project funding (grants, internal R&D programs) is NOT a personal fellowship or award. Never list funding sources under Fellowships & Honors.

### Rule 6: Profile links are plain text URLs, never hyperlinks
LinkedIn and GitHub in the header block must render as the **visible URL text**, never as a `\href` hyperlink hiding behind a label like `LinkedIn` or `GitHub`.

A resume gets printed, screenshotted, and forwarded as flat text. A hyperlink whose display text is `GitHub` becomes a dead word on paper: the reader can see that a profile exists but has no way to reach it.

| | Correct | Wrong |
|---|---------|-------|
| Resume/CV header | `\address{linkedin.com/in/name \\ github.com/name}` | `\address{\href{https://linkedin.com/in/name}{LinkedIn}}` |

Drop the `https://` and `www.` prefixes. What remains is still a working, typeable URL and it keeps the header line short.

**Scope:** LinkedIn and GitHub. ORCID and Google Scholar in `cv_template.tex` keep their `\href` — those URLs are long opaque ID strings, and the ORCID iD is already printed in full next to the icon.

This applies to every generated resume, CV, and any regeneration of an existing output. It is a template-level rule, so it survives the FIXED-header restriction: the header layout is still immutable, but its link form is now specified here.

---

## LaTeX Scientific Notation (MANDATORY)

All templates load `mhchem` (`\usepackage[version=4]{mhchem}`). Use these conventions:

| Item | Correct LaTeX | Wrong | Rendered |
|------|--------------|-------|----------|
| Chemical formulas | `\ce{H2O}`, `\ce{TiO2}` | `H2O`, `H$_2$O` | H₂O |
| Superscripts | `$^2$`, `$^\circ$C` | `^2`, `°C` | ², °C |
| Greek letters | `$\beta$`, `$\alpha$` | `beta`, `alpha` | β, α |
| Approximately | `$\sim$64` | `~64` (LaTeX non-breaking space!) | ~64 |

**CRITICAL:** `~` in LaTeX is a non-breaking space, NOT a tilde. Use `$\sim$` for "approximately."

For char counting: `\ce{TiO2}` → 4 rendered chars, `$\beta$` → 1 rendered char.

---

## Active Sessions

_Update this section when starting/finishing a JD._

| Session | Status | Next Command |
|---------|--------|-------------|
| (none active) | — | — |

---

## KB Corrections Log

_See `config.md` for user-specific corrections. Add verified errors here as you find them._
