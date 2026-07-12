# Templates

System-layer template files used by career-ops scripts and modes. These files are auto-updated when you run `npm run update` -- put user customizations in the user-layer files instead (see DATA_CONTRACT.md).

## Files

| File | Used By | Purpose |
|------|---------|---------|
| `cv-template.html` | `generate-pdf.mjs` | HTML/CSS template for ATS-optimized CV PDFs |
| `resume-template.html` | `generate-pdf.mjs` (via `--template`) | Resume-branded variant of `cv-template.html`. Same layout and placeholder tokens; differs in: `<title>` reads "Resume" instead of "CV", omits Certifications section, targets 1–2 page US/industry format. See detailed section below. |
| `cv-template-alt.html` | `generate-pdf.mjs` | Alternate HTML/CSS design for ATS-optimized CV PDFs |
| `cv-template-classic.html` | `generate-pdf.mjs` | Compact, centered-header HTML design tuned to fit 2 pages |
| `cv-template.tex` | `generate-latex.mjs` | LaTeX/Overleaf template for ATS-optimized CV PDFs |
| `portals.example.yml` | Onboarding | Example portal scanner configuration (copy to `portals.yml` to activate) |
| `states.yml` | `verify-pipeline.mjs`, `normalize-statuses.mjs`, `merge-tracker.mjs` | Canonical application states and their aliases |

### cv-template.html

The HTML template rendered by Playwright into PDF. Uses placeholder tokens (`{{NAME}}`, `{{SUMMARY_TEXT}}`, `{{EXPERIENCE}}`, etc.) that the PDF pipeline fills at generation time.

**Design:** Space Grotesk headings + DM Sans body, single-column ATS-safe layout, self-hosted fonts from `fonts/`. Cyan/purple gradient accent, underlined section headers, filled competency pills.

**Customization:** Edit this file to change colors, spacing, or section order. The placeholder tokens are documented in `batch/batch-prompt.md` under "Template placeholders."

### resume-template.html

Resume-branded variant of `cv-template.html` for US/industry job applications. Key differences from the CV template:

- **Title** reads "Resume" instead of "CV"
- **No Certifications section** — resumes focus on recent, relevant experience
- **Designed for 1–2 pages** — omits academic-style sections

Otherwise uses the same placeholder tokens (`{{NAME}}`, `{{SUMMARY_TEXT}}`, etc.) and is fully compatible with the existing PDF pipeline.

**Keep in sync:** When updating `cv-template.html`, apply matching changes to `resume-template.html` (preserving the differences noted above).

### cv-template-alt.html

A second, drop-in HTML design using the exact same placeholder tokens and CSS class names as `cv-template.html` (same `{{...}}` contract, same `.job`/`.project`/`.edu-item`/`.cert-item`/`.skills-grid` structure) — only the visual theme differs, so any content-generation logic that fills one template fills the other without changes.

**Design:** Same self-hosted fonts (Space Grotesk + DM Sans), single-column ATS-safe layout. Distinct look: slate-blue + amber palette, uppercase name with a left accent bar in the header, "tab"-style section headers (short accent underline instead of a full-width rule), outlined (not filled) competency/badge pills, and a thin left rule connecting work-experience and project entries.

**Usage:** To generate a CV with this design instead of the default, tell the agent to use `cv-template-alt.html` as the base when running the `pdf` mode (step 12 of `modes/pdf.md` — "Generate full HTML from template"). Everything else in the pipeline (`generate-pdf.mjs`, keyword injection, tracker update) is unchanged.

**Customization:** Edit this file the same way as `cv-template.html` — change the `hsl(222, 47%, ...)` / `hsl(20, 80%, ...)` color tokens, border styles, or section-title treatment to create further variants.

### cv-template-classic.html

A third HTML design, built to reproduce a traditional single-column, centered-header resume layout (uppercase name, italic location, section titles with a trailing horizontal rule) at a tighter type scale so a full career history — 4+ jobs, multiple sub-headed bullet groups, projects, education, certifications — fits in **2 pages max** at 0.6in margins. Verified against `generate-pdf.mjs` with a full 4-job / 2-project / 3-cert dataset → 2 pages.

**Design:** System font stack (`Calibri, Carlito, Arial, Helvetica Neue, sans-serif` — no custom `fonts/` files needed), ~9.5px body type, single accent color (`hsl(212, 70%, 32%)`) on job titles only. Section titles use a flex row with a `border-top` spacer filling the remaining width (a real element, not a `::before`/`::after` with `content:` text, so nothing gets pulled out of PDF reading order).

**Placeholder contract differences from `cv-template.html`:** no `{{COMPETENCIES}}`/`{{SECTION_COMPETENCIES}}` (skills live only in `{{SKILLS}}` as full-width category lines, not pill tags); adds `{{GITHUB_URL}}` in the contact row (same token name used in `cv-template.tex`). All other tokens (`{{NAME}}`, `{{SUMMARY_TEXT}}`, `{{EXPERIENCE}}`, `{{PROJECTS}}`, `{{EDUCATION}}`, `{{CERTIFICATIONS}}`, etc.) match `cv-template.html`.

**New structure for `{{EXPERIENCE}}`:** each `.job` may include an optional italic `.job-intro` paragraph (a one-line role summary) and zero or more `.job-subsection` blocks, each with an italic bold `.job-subsection-title` (e.g. "Transaction Monitoring") followed by its own `<ul>`. Roles with no bullets (e.g. short/part-time stints) can be just `.job-header` + `.job-intro`, no subsections.

**New structure for `{{PROJECTS}}`:** each `.project` is a hanging-indent bullet — put a literal `<span class="project-bullet">&bull;</span>` before `.project-title`, **not** a CSS `::before` pseudo-element. Chromium's PDF text stream follows paint/z-order for absolutely-positioned pseudo-content, so an absolutely-positioned bullet marker gets extracted out of sequence by ATS parsers and text-extraction tools, even though it renders correctly on screen. Keep bullet markers as real inline elements in the DOM.

**Customization:** Edit this file to change the accent color, type scale, or section order. If you tighten spacing further to fit more content, re-verify page count with `node generate-pdf.mjs <filled.html> <out.pdf> --format=letter` before relying on it — 2-page fit depends on content volume as much as CSS (the `pdf` mode's project/bullet selection already trims content to the most relevant items, which does most of the work).

### cv-template.tex

LaTeX template for Overleaf-compatible CV generation. Based on the [sb2nov/resume](https://github.com/sb2nov/resume) format. Uses placeholder tokens (`{{NAME}}`, `{{EXPERIENCE}}`, `{{PROJECTS}}`, etc.) that the LaTeX pipeline fills at generation time.

**Design:** Single-column ATS-safe layout using standard CTAN packages (`fontawesome5`, `enumitem`, `hyperref`, `titlesec`). No custom fonts or external dependencies — uploads directly to Overleaf.

**Usage:**
```bash
# Validate and compile .tex → .pdf (requires pdflatex on PATH)
node generate-latex.mjs output/cv-name-company-date.tex

# Or specify a custom output path
node generate-latex.mjs output/cv-name-company-date.tex output/custom-name.pdf
```

**Prerequisites:** `pdflatex` via [MiKTeX](https://miktex.org/) (Windows) or TeX Live (Linux/macOS). First compilation may auto-install missing LaTeX packages. Alternatively, upload the `.tex` file directly to [Overleaf](https://www.overleaf.com) — no local install needed.

**Customization:** Edit this file to change margins, section order, or formatting commands. The placeholder tokens are documented in `modes/latex.md` under "Template Placeholders."

### portals.example.yml

Pre-configured portal scanner with 45+ tracked companies and search queries. Contains title filters, company career page URLs, Greenhouse API endpoints, and WebSearch queries.

**To activate:** Copy to project root as `portals.yml` and customize `title_filter.positive` keywords for your target roles. Add or remove companies as needed.

### states.yml

Defines the 9 canonical application states (`Evaluated`, `Applied`, `Responded`, `Interview`, `Offer`, `Hired`, `Rejected`, `Discarded`, `SKIP`) with aliases for common variants. All pipeline scripts validate statuses against this file.

**Do not rename states** -- the dashboard and all scripts depend on these exact IDs. You can add aliases if you encounter new variants that should map to an existing state.
