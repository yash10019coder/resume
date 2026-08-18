---
name: tailor-resume-to-jd
description: Use when the user gives a job description (URL or text) and asks for a tailored resume, a new resume branch, or to "optimize my resume for this role" in this repo.
---

# Tailor Resume to JD

## Overview
Generates a new git branch + single-page LaTeX resume tailored to a job description, built from real experience (existing branches, LinkedIn export, GitHub repos) — never fabricated.

## Steps

1. **Get the JD.** Fetch the URL or use pasted text. Extract: required skills/tools, responsibilities, seniority band (years of experience), and any explicitly-named preferred qualifications.

2. **Pick the base branch.** Default to `post-browserstack-resume-short` unless the user names another. Confirm with the user if ambiguous.

3. **Gather evidence — don't invent keywords.** A JD keyword only goes on the resume if it's backed by evidence from one of:
   - `git log --all -p -- '*.tex'` across existing branches (grep for the keyword/synonyms)
   - The user's LinkedIn PDF export (ask them to download it from their profile page in the default "Resume" format if not already provided; note its date — if it looks stale, ask if a fresher copy exists)
   - GitHub: `gh api users/<user>/repos` for description/languages/topics of public repos, prioritizing pinned/starred repos as a proxy for what the user considers best work; fetch README via `gh api repos/<owner>/<repo>/readme` or WebFetch
   - Explicit confirmation from the user in-conversation (e.g. "I used Terraform at my last company")
   
   If a JD keyword (e.g. Datadog, Ansible-in-production) has no evidence anywhere, **do not add it**. Say so instead of guessing.

4. **Check for duplicate branches.** List existing branches (`git branch -a`) and skim their `.tex` filenames/content for near-duplicates of the role you're about to build (same company/role already tried). Flag it to the user rather than silently creating a redundant branch.

5. **Create the branch and file.** Naming convention: branch `<company>-<role-slug>-resume`, file `Yash_Verma_CV_<Company>.tex`. **Exactly one `.tex` file per branch** — the CI pipeline compiles and uploads every `.tex` file it finds in a branch to Drive, so a leftover file from the base branch must be `git rm`'d, not left behind.

6. **Write the resume**, reordering sections/bullets to foreground JD-relevant work, weaving in evidenced keywords naturally into existing bullets (don't just append a keyword-soup line).

7. **Fit to one page — verify by rendering, not by trusting `pdflatex`.** This template inflates `\textheight` beyond the physical page, so `pdflatex` reports "1 page" even when content is silently clipped past the visible bottom margin. After every compile:
   ```
   pdflatex -interaction=nonstopmode <file>.tex
   pdftoppm -png -r 150 <file>.pdf /tmp/resume_check
   ```
   Then Read the PNG and visually confirm the last section (Education) is fully visible, not cut off. Trust the rendered image, never the page count alone.

8. **Trim order when overflowing** (cut in this order, stop as soon as it fits):
   1. Oldest/least JD-relevant experience bullets
   2. Project bullets
   3. Skills/Concepts line wording (condense, don't drop categories)
   4. Never cut: name/contact header, Education, or any bullet that's the *only* evidence for a JD-required keyword

9. **Clean build artifacts** (`.aux .log .out .fls .fdb_latexmk .synctex.gz`) before committing. Commit only the `.tex` and `.pdf`.

10. **Report to the user** what JD keywords were matched with evidence, and which JD-requested skills were deliberately left off due to no evidence — so they can correct you if you missed something (e.g. they secretly do have Datadog experience).

## Common Mistakes
- Trusting `pdflatex`'s reported page count instead of rendering and looking — the #1 failure mode this session.
- Adding a JD keyword because it "sounds plausible" for a backend engineer, without checking any evidence source.
- Leaving the base branch's original `.tex` file in the new branch, causing CI to compile/upload two resumes.
- Cutting Education or contact info to fix overflow instead of trimming bullets.
