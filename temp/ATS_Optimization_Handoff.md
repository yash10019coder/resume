# Context Handoff — Resume ATS Optimization

**Purpose:** Hand off an in-progress resume ATS-optimization task to another agent. This document is self-contained; you do not need the prior conversation to continue.

**Suggested opening instruction for the receiving agent:**
> You are continuing an ATS-optimization task for a software engineer's resume. Read this handoff fully, then continue from the "Open Items / Next Steps" section. Verify every formatting change by re-running a text-extraction parser on the compiled PDF — do not trust visual appearance.

---

## 1. Candidate & Goal

- **Candidate:** Yash Verma — backend engineer, ~4 years experience. B.Tech Information Technology, IIIT Lucknow (Apr 2020 – Apr 2024), CGPA 8.32.
- **Current/most recent role:** Backend SDE at BrowserStack (Apr 2025 – present). Prior: Zus/0chain (blockchain storage & mobile), Oppia Foundation (OSS + GSoC), GNOME Foundation (OSS).
- **Goal:** Maximize shortlisting odds at large MNCs whose first screen is an Applicant Tracking System (ATS). The candidate wants the resume to (a) parse cleanly and (b) read well to the modern AI screening layer.

## 2. Files & Source of Truth

- **Repo:** `github.com/yash10019coder/resume`
- **Branch:** `post-browserstack-resume-short`
- **Primary file:** `Yash_Verma_CV_Remote_Short.tex` (LaTeX; compiled with pdfTeX/`pdflatex`).
- **Important:** The LaTeX source on this branch is a NEWER revision than an earlier PDF the candidate shared. Treat the repo `.tex` as the source of truth, not any older PDF.
- **Delivered optimized files (already produced):**
  - `Yash_Verma_CV_Remote_Short_ATS.tex` — corrected source
  - `Yash_Verma_CV_Remote_Short_ATS.pdf` — compiled, 1 page

## 3. The ATS Model We're Optimizing Against

Two readers must both be satisfied:

1. **Parser/scorer (literal):** Strips formatting, reads the text stream, segments it into sections, extracts structured fields (job title, company, dates, skills), then keyword-matches/scores against the job description. **If parsing fails, the score is zero regardless of qualifications.** Job-title match is heavily weighted (notably in Workday, common at large MNCs).
2. **AI co-pilot (semantic):** A language model reads the resume as prose and assesses career trajectory, coherence, stability, and growth. A resume that scores well mechanically but reads incoherently can still be flagged weak.

Practical implications used in this task:
- Single-column, standard section headings, no tables/multi-column for parse-critical content.
- `.docx` often parses more reliably than PDF across ATS variants; keep a PDF for humans, a `.docx` for portals.
- Keyword alignment with the target JD ~65–75% (enough to rank; copy-pasting the JD triggers manipulation flags).
- No manipulation tactics (white-text keyword stuffing, invisible JD paste) — these are detected and penalized.

## 4. Diagnostics Already Run (baseline)

Using poppler tools on the compiled PDF:

- `pdfinfo`: 1 page, US Letter, **untagged** (`Tagged: no`), no forms, no JavaScript, Creator = "LaTeX with hyperref", Producer = pdfTeX.
- `pdffonts`: Computer Modern Type-1 fonts, all embedded, Unicode mapping present.
- `pdfimages -list`: no raster images (good — nothing unparseable).
- Text extraction (`pdftotext`, default + `-layout`): reading order is correct top-to-bottom. **Special characters extract correctly** — `208×`, `→`, `∼1TB`, en-dashes — no mojibake. This is notable because Computer Modern PDFs sometimes garble these.

## 5. Issues Identified

1. **Skills section split (FIXED).** The section used manual `\hspace{...}` alignment after each bold label, creating a pseudo two-column layout. Naive parsing read all labels first, then all values, detaching each category from its skills.
2. **Job titles fused with tech stack (FIXED).** Each role's title line (arg #3 of the `\resumeSubheading` macro) appended the full tech stack, e.g. `Backend SDE - Ruby on Rails, Java/Spring Boot, Node.js, ClickHouse, Kafka, AWS`. The parser grabs that entire string as the "Job Title" field, polluting a heavily-weighted signal.
3. **Untagged PDF (NOT addressed; low priority).** No logical reading-order tags. Poppler recovered order fine, but tagged docs are more robust across the ~200 ATS parsers. `.docx` sidesteps this.
4. **Unused dependency (FIXED).** `\usepackage{fontawesome}` was loaded but unused (icon header is commented out).

## 6. Fixes Already Applied & Verified

All changes are in `Yash_Verma_CV_Remote_Short_ATS.tex` / `.pdf`. Each was verified by re-running `pdftotext` on the recompiled PDF.

- **Skills:** Removed `\hspace` alignment so each line is one continuous run: `Programming & Scripting: Java, Go, Kotlin, ...`. Tradeoff: values no longer align in a vertical column visually (acceptable for ATS). Verified: each label now parses on the same line as its values.
- **Titles:** Stripped tech stack from every title line; moved it to a `Tech:` first bullet under each role so keywords are still matched but the Title field is clean. Expanded abbreviations: `SDE → Software Development Engineer`, `GSoC → Google Summer of Code`. Verified: titles now parse as clean strings; `Tech:` lines parse separately.
- **Dependency:** Commented out `\usepackage{fontawesome}` (re-enable if the icon header is restored). NOTE: the candidate's CI presumably has `fontawesome.sty`; it was only missing in the build sandbox.
- **Result:** Still compiles to 1 page; special characters intact.

## 7. Open Items / Next Steps

1. **`.docx` version (offered, NOT yet produced).** Generate a plain `.docx` from the optimized content for application portals that parse Word more reliably. Keep the LaTeX PDF for human readers.
2. **Per-job keyword tailoring.** For each target role, mirror the exact terminology of that job description (e.g., "cross-functional collaboration" vs "teamwork" are different tokens). Aim ~65–75% alignment. For MNCs, watch US/UK spelling ("organization" vs "organisation"). Identify terms that repeat in a posting — those are weighted most.
3. **"4+ years in production" coherence flag.** The professional summary claims 4+ years, but the degree ran 2020–2024 and some experience overlaps with school / is OSS or mobile rather than backend distributed systems. Parses fine, but the AI semantic reader (and interviewers) may probe it. Consider rewording for precision or be ready to explain the timeline. NOT yet changed — candidate's call.
4. **Title-to-target alignment.** If applying to roles posted as "Backend Engineer" or "Software Engineer," consider matching the resume's lead title to that exact phrasing for stronger title-match scoring.
5. **Re-verify after any edit.** Always recompile and re-parse; never trust visual layout alone.

## 8. Reproduction & Verification Commands

```bash
# Clone the source of truth
git clone --depth 1 --branch post-browserstack-resume-short \
  https://github.com/yash10019coder/resume.git

# Compile (needs fontawesome.sty in the TeX distro, OR keep it commented out)
pdflatex -interaction=nonstopmode -halt-on-error Yash_Verma_CV_Remote_Short.tex

# Simulate a NAIVE ATS parser (raw stream order) — the key test
pdftotext Yash_Verma_CV_Remote_Short.pdf -

# Check spatial/column structure (catches hidden multi-column layouts)
pdftotext -layout Yash_Verma_CV_Remote_Short.pdf -

# Sanity checks
pdfinfo  Yash_Verma_CV_Remote_Short.pdf   # page count, tagged?, forms?
pdffonts Yash_Verma_CV_Remote_Short.pdf   # embedded? unicode? (garbling risk)
```

**Pass criteria when re-parsing:** each skills category sits on the same line as its values; each job title parses as a clean string without the tech stack; dates and companies are recoverable per role; special characters are not garbled; document stays one page.

## 9. Things NOT To Do

- Do not add white/invisible keyword text or paste the job description verbatim — detected and penalized by modern ATS.
- Do not reintroduce multi-column or `\hspace`/tabular alignment for parse-critical content (skills, titles, dates).
- Do not claim skills the candidate can't back up in a technical interview — keywords pass the screen, but the interview is where they're tested.
