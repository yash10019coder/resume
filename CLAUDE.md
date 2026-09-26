# CLAUDE.md

Guidance for Claude Code when working in this repository (a LaTeX resume, version-controlled).

## Repo purpose

This repo holds Yash Verma's resume(s) in LaTeX, primarily `Yash_Verma_CV_Remote_Short.tex` (compiled to PDF via `latexmk`). `temp/` holds older resume variants (gitignored) used as source material for content (e.g. project descriptions) — read-only reference, not canonical.

## Hard rules (violated once already this session — do not repeat)

1. **Never delete content when "trimming" or "removing" a bullet/section — comment it out instead** (`%` prefix in LaTeX), with a one-line comment explaining why and referencing the commit/rationale. This keeps every edit reversible directly in the file, not just in git history. Applies to Professional Summary, BrowserStack bullets, and any future trims.
2. **Never claim "it renders as one clean page" from `pdfinfo`/page-count/overfull-warning greps alone.** This template uses `\addtolength{\textheight}{2.6in}`, which extends the LaTeX text-box past the physical 11in page — content can render past the visible page while LaTeX still reports "1 page, no overfull warnings." Always actually render and visually inspect the PDF (via the `Read` tool on the `.pdf`, which shows page images) before declaring a layout change done.
3. **Verify every quantitative/attribution claim (PR counts, "built X", "led N engineers") against real public data (GitHub API, etc.) before adding or keeping it.** Do not take resume-content requests at face value if they contain hard numbers tied to external, checkable systems. See `memory/zus-0chain-interview-prep.md` for the concrete precedent (225+/120+ PR claims that didn't hold up, and the "Contributed to..." reframing that resulted).
4. **No `rm` commands without explicit user confirmation**, even inside a larger compound command, even in a scratch/temp dir. Prefer `mkdir -p` and simply not cleaning up, or ask first.
5. **≥2 independent review sources (panel personas, batch comparisons) must agree before a finding is acted on.** A single agent's opinion is not sufficient grounds to edit the resume.

## Workflow patterns used in this repo

- **Cost-controlled subagent fan-out**: prefer several small/cheap review agents (personas, batches) run in parallel over one large expensive pass, then a separate synthesis/merge step in the main session (not another agent call).
- **GitHub verification**: use `curl` + the public REST API (`/repos/{owner}/{repo}`, `/commits`, `/contributors`, `/search/issues?q=repo:...+author:...+type:pr`) to check claims about PRs/contributions before they go on the resume. Distinguish forks (no personal commits) from real authored work before linking a repo from a bullet.
- **Text extraction for ATS/parseability checks and reading other resumes**: `pdftotext -layout file.pdf` — free, deterministic, no agent needed just to extract text.
- **Visual verification**: `Read` tool directly on the compiled `.pdf` to see actual rendered pages — the only reliable way to catch content cut off past the visible page boundary (see hard rule 2).
- **Comparing against other people's resumes** (files live in the parent directory, `../`, not this repo): read-only, structural/stylistic patterns only — never copy another named person's specific achievements, numbers, or employer-confidential claims into this resume.

## LaTeX structure notes

- Custom commands: `\resumeItem`, `\resumeSubheading`, `\resumeSubHeadingListStart/End`, `\resumeItemListStart/End` — spacing (`itemsep`, `vspace`) has been tuned iteratively; changing it requires re-verifying visually (rule 2), not just recompiling.
- `%` is LaTeX's comment character — never put a literal `%` inside a `\href{...}` URL (e.g. use `:` not `%3A` for GitHub search query URLs), or the file fails to compile.
- Build with `latexmk -pdf -interaction=nonstopmode <file>.tex`.

## Git / attribution

- Branch naming so far: feature branches like `resume-callback-optimization` off `post-browserstack-resume-short`.
- Do not push branches or open PRs unless explicitly asked.
