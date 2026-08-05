---
description: Upload a finished application package (resume, CV, cover letter PDFs) to the Google Drive cv-and-resume folder. Use when the user asks to sync, upload, back up, archive, or share their resume, CV, or application package to Drive, or right after /critique approves a package.
user-invocable: true
---

# /sync-drive

**User input:** `$ARGUMENTS`

Parse `$ARGUMENTS`:
- Output folder path (e.g. `output/Acme/`) -> sync that package
- Session file path -> derive its output folder
- Company name -> resolve to `output/<Company>/`
- Empty -> read `CLAUDE.md` Active Sessions and use the most recently completed package

---

## Why this is a skill and not a hook

A hook would fire on every file write. The generation pipeline rewrites the same `.tex` many
times - once per char-count gate, again after each compile, again after `/critique` and every
edit pass. Syncing on write would fill Drive with dozens of near-identical drafts and leave you
unable to tell which one you actually sent to the employer.

"Finished" is a judgment call, not a filesystem event. It happens when you approve the package
at the `/critique` stop. Run this skill at that moment and Drive gets exactly one clean copy
per role you applied to.

Uploading also sends documents carrying your full name, phone, email, and employment history to
an external service, which is a thing to do deliberately rather than in the background.

---

## Transport: rclone, not an MCP upload call

Upload with `rclone`, which is already configured with a `gdrive:` remote pointing at the
account that owns `cv-and-resume`.

The reason matters: MCP file-creation tools take file content inline as base64, so pushing a
100 KB PDF through one costs roughly 30k tokens of context per file and scales linearly with
every re-sync. `rclone` streams straight from disk at zero context cost and handles binaries
natively. Verify the remote responds before doing anything else:

```bash
rclone lsd gdrive:cv-and-resume
```

If that fails, stop and report it. Do not fall back to base64 uploads without telling the user
what it will cost, and never ask them to paste credentials or tokens into the chat.

The Google Drive **connector** (`search_files`, `get_file_metadata`) is the companion tool for
metadata: use it to fetch a file's `viewUrl` after upload. Do **not** use `rclone link` for
this - it changes the file's sharing permissions, and these documents should stay private
unless the user asks otherwise.

---

## Phase 1: Collect the local files

Read the session file and confirm the package is finished. If `Critique: PENDING` or the resume
status is not DONE, say so and ask whether to sync anyway. An unfinished package in Drive is
worse than none, because six months later it is indistinguishable from the final one.

Upload **only the compiled PDFs** - the resume or CV, and the cover letter if one exists.

Never upload `.tex` sources, session files, critiques, or LaTeX build artifacts (`.aux`, `.log`,
`.out`). Those are working files carrying draft history and reviewer notes, and they do not
belong in a folder you might later share.

If no PDFs exist, stop and tell the user to compile first.

### Name the destination, not the source

Local files are named for the pipeline (`e2e_acme_engineer_resume.pdf`). That name is useless to
a recruiter and meaningless in a shared folder. Upload under a submission name instead -
`rclone copyto` sets the destination name independently, so the local working name stays
untouched.

Take the person's name from `config.md` Personal Info and the role from the session file's JD
Info (the same `ROLE:` value the JD capture recorded):

| Document | Drive name |
|----------|-----------|
| Resume or CV | `<First>-<Last>-CV-<Company>-<Job-Title>.pdf` |
| Cover letter | `<First>-<Last>-Cover-Letter-<Company>-<Job-Title>.pdf` |

Both the company and the job title belong in the name. One company often has several openings,
and a company-only name silently overwrites your first application the moment you apply to a
second role there.

Slug rules for the job title: drop parenthetical qualifiers, replace spaces with hyphens, keep
the original capitalization so acronyms stay readable.

- "AI Solutions Engineer" at Acme -> `Jordan-Chen-CV-Acme-AI-Solutions-Engineer.pdf`
- "Software Engineer (Network Security)" at Globex -> `Jordan-Chen-CV-Globex-Software-Engineer.pdf`

This is the same qualifier-dropping rule `/make-jd` uses for JD filenames, so the JD file, the
`output/` folder, and the Drive copy stay recognizably the same job.

**If the folder already uses a different pattern**, say so in the confirmation rather than
silently mixing two schemes. In particular, a file named for the company alone is an earlier,
looser version of this convention: your upload does not replace it, so point it out and let the
user decide whether to delete the old one. Never delete it yourself.

---

## Phase 2: Resolve the destination

Files go flat in `cv-and-resume/`, one set per role, kept distinct by the company and job title
in the filename. Do not create subfolders unless the folder is already organized that way.

```bash
rclone lsl gdrive:cv-and-resume/
```

Read that listing to work out which of your uploads replace an existing file and which are new,
and to spot any file under an older naming pattern. The confirmation step below needs both.

If a subfolder like `old/` exists, treat it as the user's own archive. Never write into it, and
never move or delete anything already in Drive - archiving is their call, not yours.

---

## Phase 3: Upload

### >>>>>> CONFIRM BEFORE UPLOADING <<<<<<

Uploading sends personal documents to an external service. Show exactly what will happen and
wait for a clear yes:

```
About to upload to Drive folder cv-and-resume/:
  Jordan-Chen-CV-Acme-Backend-Engineer.pdf            (new)
  Jordan-Chen-Cover-Letter-Acme-Backend-Engineer.pdf  (replaces existing, modified 2026-08-01)

Already there, not touched: Jordan-Chen-CV-Acme.pdf (older company-only name, likely superseded)
```

If the user declines, stop cleanly and change nothing.

Then, per file:

```bash
rclone copyto "output/<Company>/<local>.pdf" "gdrive:cv-and-resume/<Submission-Name>.pdf"
```

`copyto` writes to an exact destination path, so re-syncing overwrites in place rather than
accumulating `Resume (1).pdf` copies, and any link already shared keeps working.

Verify what landed:

```bash
rclone lsl gdrive:cv-and-resume/
```

---

## Wrap-Up

Report per file: destination name, whether it was new or replaced, and size. Fetch the folder or
file `viewUrl` through the Drive connector so the user gets a clickable link.

Flag anything worth knowing - a package synced before critique, a missing cover letter, or a
file under an older naming pattern that your upload did not replace.

Do not modify the session file or `CLAUDE.md`. This skill copies finished work outward; it is
not part of the generation state machine.
