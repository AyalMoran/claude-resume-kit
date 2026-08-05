---
description: Capture a job posting into a JDs/*.txt file from a URL or pasted text - the entry point before /make-resume. Use whenever the user shares a job posting link, pastes a job ad, or mentions a new role they want to apply for, even if they never say "JD" or "file".
user-invocable: true
---

# /make-jd

**User input:** `$ARGUMENTS`

Parse `$ARGUMENTS` (plus any posting text in the surrounding message):
- URL (http/https) -> fetch the posting (see Getting the Text)
- Pasted text that reads like a job posting -> use it directly
- URL + pasted text -> the text is the posting; keep the URL as SOURCE
- Empty -> ask the user for a link or the pasted posting
- Multiple postings -> produce one JD file per posting

---

## Purpose

Turn a job posting into `JDs/<company>_<role>.txt` in the exact format the rest of the pipeline consumes.

`/make-resume` Phase 0 mines this file for ATS keywords and classifies every requirement as Direct / Bridge / Gap.
That analysis operates on the posting's exact wording, so this skill is a **capture tool, not a summarizer**.
The rule that overrides everything else: requirement and responsibility lines are copied word-for-word.

---

## Getting the Text (URL input)

Try in order; stop at the first method that yields the full posting:
1. WebFetch (load it via ToolSearch if deferred)
2. `curl -sL <url>` via Bash, then extract the posting text from the HTML
3. A browser tool, if one is available (for JS-rendered pages)

Signs you do NOT have the full posting: a login wall, truncated text, a search/listing page with several jobs, or a consent page with no body.
In that case - or if all methods fail - stop and ask the user to paste the posting text.
NEVER reconstruct a JD from memory, web-search snippets, or a cached summary.
A fabricated or paraphrased requirement poisons every downstream artifact (bullet plan, ATS scoring, cover letter hooks).

If the URL is a listing page with several roles, ask which one before proceeding.

---

## What to Keep, What to Drop

**Keep verbatim:**
- Role title, team, location, employment type
- Role summary / "about the role" paragraph
- Every responsibilities bullet
- Every requirements bullet
- Every nice-to-have / advantages bullet
- Compensation info and unusual notes (application instructions, AI-use policies, culture statements) - these feed cover-letter hooks
- EEO statement

**Keep, light condensing allowed:**
- Company about/marketing copy -> goes in COMPANY CONTEXT.
  Attribute marketing claims instead of asserting them ("States 300% revenue growth", not "300% revenue growth") - downstream skills must not mistake company marketing for verified fact.

**Drop (page chrome only):**
- Navigation menus, cookie banners, apply/save/share buttons, applicant counts, "promoted" tags, similar-jobs lists, job alerts, footers

Never rewrite, reorder, summarize, or translate the posting body.
Only include sections the posting actually has - never invent one.
If the posting is not in English, capture it as posted and flag that to the user.

---

## File Format

This spec is the authority - `JDs/` may be empty on a fresh clone, so do not depend on finding a
sample there:

```
COMPANY:  <Company Name>
ROLE:     <Full role title as posted>
TEAM:     <Team/department, if stated>
LOCATION: <Location, if stated>
SOURCE:   <URL, or: pasted by user - no URL provided>
CAPTURED: <today, YYYY-MM-DD>

================================================================================
COMPANY CONTEXT
================================================================================

<about-company text>

================================================================================
ROLE SUMMARY
================================================================================

<role intro paragraph>

================================================================================
<POSTING'S OWN SECTION HEADER, UPPERCASED>
================================================================================

- <bullet, word-for-word>

- <bullet, word-for-word>
```

- Omit the TEAM / LOCATION lines when the posting does not state them.
- SOURCE is honest provenance: the real URL, or `pasted by user - no URL provided`. Never invent a URL.
- Section dividers are 80 `=` characters. Wrap prose at ~80 columns. One blank line between bullets.
- Use the posting's own section names uppercased (e.g. WHAT YOU'LL BE DOING, THE IDEAL CANDIDATE).
  Fall back to RESPONSIBILITIES / REQUIREMENTS / ADVANTAGES only when the posting has no usable headers.

---

## Naming

`JDs/<company>_<role>.txt` - lowercase snake_case, ASCII only.

- Company first: `/make-resume` derives the output folder (`output/<Company>/`) and the session name from this filename, so the company must be the leading token.
- Drop parenthetical qualifiers from the filename but keep them in ROLE.
  Example: role "Software Engineer (Network Security)" at Acme -> `acme_software_engineer.txt`.
- If `JDs/` already holds captures from earlier runs, match their conventions where they agree with
  this spec - consistency across the folder is what makes the filenames scannable.
- If the target filename already exists: show the user the existing file's header and ask whether to overwrite, suffix (`_2`), or abort. Never overwrite silently - it may be a different req at the same company.

---

## Self-Check (before reporting)

Re-read the file you wrote and verify:
1. Header block is complete and accurate - SOURCE especially.
2. Every responsibilities / requirements / advantages bullet from the source appears word-for-word (line re-wrapping at 80 columns is the only allowed change).
3. No page chrome survived.
4. The filename starts with the company name and derives cleanly to an `output/<Company>/` folder.

---

## Wrap-Up

Report to the user:
- File path, company, role
- Sections captured, plus anything unusual worth knowing before tailoring (AI-use note, comp info, application instructions)
- Anything you could NOT capture (blocked URL, missing sections) - stated plainly, never papered over

Then hand off:

```
Next: /make-resume JDs/<filename>.txt
```

Do not start `/make-resume` yourself, and do not touch `CLAUDE.md` Active Sessions - that bookkeeping belongs to `/make-resume` Phase 0.
