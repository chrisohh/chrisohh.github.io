# Design: Chan-Ye (Chris) Ohh — Personal Academic Website

**Date**: 2026-03-03
**Purpose**: Faculty application website for a postdoc at Scripps Oceanography
**Status**: Approved

---

## Decisions

| Decision | Choice |
|----------|--------|
| Tone | Warm & approachable |
| Structure | Single scrolling page, sticky nav |
| Tech stack | Astro (static output) |
| Hosting | GitHub Pages |
| Primary headshot | `citations.jpeg` |
| Research framing | Oceanography-oriented |
| CV format | LaTeX source → PDF + HTML fallback |
| CV template | Minimal custom (no moderncv) |

---

## Visual Design

### Color Palette

| Role | Color | Hex |
|------|-------|-----|
| Primary | Deep teal | `#1a5276` |
| Background | Soft white | `#fafafa` |
| Accent | Warm sand | `#d4a574` |
| Section alt bg | Light ocean blue | `#e8f4f8` |
| Text | Dark charcoal | `#2c3e50` |
| Subtle text | Slate | `#5d6d7e` |

### Typography

- **Body**: Inter or Source Sans Pro (clean sans-serif, highly legible)
- **Headings**: Optional serif pairing (e.g., Lora) for warmth, or bold Inter
- **Scale**: Generous — 18px body base, clear hierarchy

### Layout Principles

- Generous whitespace
- Rounded corners on cards (8px)
- Max content width ~900px, centered
- Research cards: figure left/right alternating with text
- Mobile-responsive (single column on narrow screens)

---

## Page Sections

### 1. Hero

**Layout**: Two-column on desktop (text left, headshot right). Single column on mobile.

**Content**:
- Name: **Chan-Ye (Chris) Ohh**
- Title: Postdoctoral Researcher, Scripps Institution of Oceanography
- Tagline: One line summarizing research focus
- Headshot: `citations.jpeg` (circular crop, ~250px)
- Quick-links row: Google Scholar | ORCID | Email | LinkedIn

### 2. About

**Layout**: Single column, 2-3 paragraphs.

**Content**: Bio bridging PhD work (experimental fluid mechanics, stratified wakes at USC) to current postdoc (air-sea interaction at Scripps). Forward-looking statement about research vision in physical oceanography.

See `draft-text.md` for full draft.

### 3. Research

**Layout**: 2-3 themed cards, each with:
- Theme title
- 1-2 paragraph description (oceanography-framed)
- 1-2 figures from publications
- Links to related papers

**Themes**:

#### Theme 1: Stratified Ocean Wakes
How density stratification in the ocean alters wake dynamics behind submerged bodies.
- Figures: `JFM_2024_Fig2_experimental_setup.png`, `JFM_2024_Fig10_vorticity_trajectory.png`, `PRF_2022_Fig1_regime_diagrams.png`
- Papers: Ohh & Spedding 2024 (JFM), Ohh & Spedding 2022 (PRF)

#### Theme 2: Data-Driven Flow Identification
Modal decomposition and sparse sensing for classifying wake regimes from limited measurements.
- Figures: `PRF_2022_Fig6_snapshots_DMD_modes.png`, `PRF_2022_Fig13_TDMD_example.png`
- Papers: Ohh & Spedding 2022 (PRF, Editor's Suggestion), Chinta et al. 2022 (PRF)

#### Theme 3: Air-Sea Interaction (Forward-Looking)
Current work at Scripps. Short narrative, no figures yet. Placeholder for future content.

### 4. Publications

**Layout**: Ordered list (reverse chronological), styled as a clean bibliography.

**Each entry**:
- Authors (Chris's name **bolded**)
- Title (linked to DOI)
- Venue, volume, year
- Badges: "Editor's Suggestion", "Open Access" where applicable
- PDF link where available

**Full list**:
1. Ohh & Spedding (2024) JFM 997, A43 — stratified near wake of prolate spheroid [Open Access]
2. Ohh & Spedding (2022) PRF 7, 024801 — DMD for stratified wake ID [Editor's Suggestion]
3. Chinta, Ohh, Spedding & Luhar (2022) PRF 7, 033803 — sparse regression for wake regime ID [arXiv]
4. Ohh, Huang, Wen, Kanso & Luhar (2016) APS DFD — flapping locomotion [conference abstract]

### 5. Talks & Media

**Layout**: Simple list or small cards.

**Content**:
- APS Physical Review Journal Club video (if available/linkable)
- Conference presentations (APS DFD 2016, others TBD)

### 6. Contact

**Layout**: Centered, clean.

**Content**:
- Email: cohh@ucsd.edu
- Institutional: Scripps profile link, Air-Sea Interaction Lab
- Profiles: Google Scholar, ORCID, ResearchGate, LinkedIn

### 7. Navigation (Sticky Top)

Items: **About** | **Research** | **Publications** | **Contact** | **CV** (PDF download link)

Behavior: Smooth scroll to sections. Active section highlighted. Collapses to hamburger on mobile.

---

## CV Pipeline

### Source
- `cv/cv.tex` — Minimal custom LaTeX
  - Serif font (Computer Modern or Palatino)
  - Clear section headers, date-aligned entries
  - Generous margins, scannable layout

### Build
- `scripts/build-cv.sh`:
  ```bash
  pdflatex cv.tex        # → cv.pdf
  pandoc cv.tex -o cv.html  # → cv.html (fallback)
  ```

### Automation
- GitHub Actions workflow triggered on changes to `cv/`
- Produces `cv.pdf` and `cv.html` as build artifacts
- Deployed alongside the Astro site to GitHub Pages

### Site Integration
- Nav "CV" button → downloads `cv.pdf`
- `/cv` route renders `cv.html` as an accessible inline fallback

---

## Asset Mapping

| Asset | Used In | Notes |
|-------|---------|-------|
| `citations.jpeg` | Hero headshot | Primary photo |
| `JFM_2024_Fig2_experimental_setup.png` | Research: Stratified Wakes | CC BY 4.0 |
| `JFM_2024_Fig10_vorticity_trajectory.png` | Research: Stratified Wakes | CC BY 4.0 |
| `PRF_2022_Fig1_regime_diagrams.png` | Research: Stratified Wakes | Author reuse rights |
| `PRF_2022_Fig6_snapshots_DMD_modes.png` | Research: Data-Driven | Author reuse rights |
| `PRF_2022_Fig13_TDMD_example.png` | Research: Data-Driven | Author reuse rights |
| `publications.json` | Publications section | Data source |
| `profile.json` | Contact, links | Data source |
| `publications.bib` | Publications (optional export) | — |

---

## Project Structure (Planned)

```
website2/
  src/
    pages/
      index.astro          # Main single-page layout
    components/
      Hero.astro
      About.astro
      ResearchTheme.astro
      Publications.astro
      TalksMedia.astro
      Contact.astro
      Nav.astro
    layouts/
      BaseLayout.astro     # HTML shell, meta tags, fonts
    styles/
      global.css           # Palette, typography, base styles
    data/
      publications.json    # Publication data (from assets)
      profile.json         # Profile data (from assets)
  public/
    images/
      headshot.jpeg        # citations.jpeg
      figures/             # Research figures
    pubs/                  # PDF downloads
    cv.pdf                 # Built CV
  cv/
    cv.tex                 # LaTeX source
  scripts/
    build-cv.sh            # LaTeX → PDF + HTML
  .github/
    workflows/
      deploy.yml           # Build Astro + CV, deploy to GH Pages
  astro.config.mjs
  package.json
```
