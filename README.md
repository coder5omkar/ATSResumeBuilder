# ATS Resume Builder — Multi-Template Editor

A single-file HTML application that lets you enter resume data **once** and preview it across **5 different ATS-optimised templates** simultaneously. Pick the best look, then download as HTML or PDF.

---

## Architecture

```
editor-5tpl.html
│
├── HTML  ──  Editor form (sidebar) + preview panel + toolbar
│
├── CSS   ──  Editor UI layout, template selector cards,
│             responsive/mobile breakpoints, print styles
│
└── JS    ──  Everything below lives in one <script> block
     │
     ├── DATA LAYER (pure functions, no DOM)
     │   pushSkill(), pushExp(), pushEdu(), pushProj(),
     │   pushCert(), pushLang()
     │   └── Only mutate resumeData[] arrays
     │
     ├── EDITOR RENDERERS (DOM from data, no mutation)
     │   renderSkillsEditor(), renderExpEditor(),
     │   renderEduEditor(), renderProjEditor(),
     │   renderCertEditor(), renderLangEditor()
     │   └── Read from resumeData[], build form controls
     │
     ├── USER ACTIONS (bridge data → DOM)
     │   addSkill(), addExperience(), addEducation(), etc.
     │   └── Call push*() then render*Editor() + renderPreview()
     │
     ├── TEMPLATE RENDERERS (5 resume layouts)
     │   R.classic(d)   –  Single column, black/white, 98% ATS
     │   R.sidebar(d)   –  Two-column, navy sidebar, blue accent
     │   R.modern(d)    –  Green header bar, pill skill tags
     │   R.compact(d)   –  Dense single column, minimal borders
     │   R.executive(d) –  Two-column, brown/gold, serif headings
     │   └── Each returns an HTML string; all share the same `d` data
     │
     ├── TEMPLATE SELECTOR
     │   buildTemplateSelector() – 5 clickable cards with ATS score
     │
     └── EXPORT
         downloadHTML() – Wraps selected template in standalone HTML
         downloadPDF()  – Renders preview via html2pdf.js
```

### Data Flow

```
User edits form ──→ renderPreview() ──→ R[selectedTemplate](getData()) ──→ innerHTML
                                                ↑
User adds/removes items ──→ push*() ──→ render*Editor() ──→ rebuilds form DOM
                                                ↓
                                           renderPreview()
```

Data and DOM are **strictly separated** — the push functions never touch the DOM, and the editor renderers never mutate data. This prevents the infinite-recursion bug that plagued earlier versions.

---

## File Structure

| File | Description |
|------|-------------|
| `editor.html` | v1 — Single template, two-column with sidebar |
| `editor-v2.html` | v2 — One-page compact, colored header bar |
| `editor-v3.html` | v3 — Project Manager themed, two-column design |
| **`editor-5tpl.html`** | **Latest — 5 templates, same data, best of all** |

---

## Technologies Used

| Technology | Role |
|------------|------|
| **Vanilla HTML5** | Document structure, semantic elements |
| **Vanilla CSS3** | Layout (flexbox, grid), responsive design, print styles |
| **Vanilla JavaScript ES6** | All logic — no frameworks, no build tools, no dependencies |
| **html2pdf.js** (CDN) | Client-side HTML → PDF conversion (wraps html2canvas + jsPDF) |
| **html2canvas** (via html2pdf) | Captures DOM element as canvas image |
| **jsPDF** (via html2pdf) | Generates the PDF file |

Zero frameworks. Zero build step. Open in any browser and it works.

---

## Template System

Each template is a function `R.id = function(d) { … }` that returns a raw HTML string. The `d` parameter contains all user data:

```js
d = {
  name, title, email, phone, location, linkedin, website,
  summary,
  skills:          [{ name, level }, ...],
  experiences:     [{ title, company, location, dates, bullets }, ...],
  education:       [{ degree, institution, location, dates, details }, ...],
  projects:        [{ name, tech, dates, bullets }, ...],
  certifications:  [{ name, issuer, date }, ...],
  languages:       [{ name, proficiency }, ...],
  photo            // data:image/… URL or null
}
```

### Adding a New Template

1. Add a render function:
   ```js
   R.myNew = function(d) {
     // Build and return HTML string from d.xxx
     return `<div style="...">…</div>`;
   };
   ```
2. Register it in the `TEMPLATES` array:
   ```js
   { id:'myNew', name:'My Template', desc:'Description', ats:90,
     colors:['#hex1','#hex2','#hex3'] }
   ```
3. Done — it appears as a selectable card.

---

## ATS Compliance

Each template carries a score (86–98%). Factors considered:

- **Single vs multi-column** — single column is safest for ATS parsers
- **Font choice** — standard web-safe fonts (Calibri, Arial, Georgia)
- **No tables** — all layout uses CSS flexbox/grid
- **Semantic hierarchy** — `<h1>` for name, `<h2>` / `<h3>` for section headings
- **No critical info in images** — photo is optional and decorative
- **Bullet lists** — use proper `<ul>` / `<li>` elements
- **Clean text extraction** — no CSS pseudo-content for important text

---

## Editing the Code

Open `editor-5tpl.html` in any text editor.

- **Colors** — search for hex values (`#1a2a3a`, `#4a90d9`, etc.) in the template renderers
- **Fonts** — change `font-family` in the outer wrapper divs
- **Spacing** — adjust `padding`, `margin`, `line-height` values in inline styles
- **Sections** — add/remove `if(d.xxx)` blocks inside a template function
- **Template selector** — the `TEMPLATES` array near the top of the `<script>` block

The file is intentionally self-contained — no build steps, no config files.

---

## Browser Support

Chrome, Firefox, Safari, Edge (latest 2 versions). PDF generation requires a browser that supports `Promise` and `canvas` (all modern browsers).

---

## Licence

Free to use, modify, and distribute.
