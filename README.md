# Chan-Ye (Chris) Ohh — Personal Website

Academic website built with [Astro](https://astro.build/), deployed to GitHub Pages at [chrisohh.github.io](https://chrisohh.github.io/).

## Quick Start

```bash
npm install
npm run dev        # http://localhost:4321/cysite1/
npm run build      # output → dist/
npm run preview    # preview production build
```

## Project Structure

```
src/
  pages/index.astro          # Single-page entry point
  layouts/BaseLayout.astro   # HTML shell, fonts, meta
  components/
    Nav.astro                # Sticky nav + mobile hamburger
    Hero.astro               # Name, title, headshot, links
    About.astro              # Bio paragraphs
    Research.astro           # Three research themes
    ResearchTheme.astro      # Reusable research card
    Publications.astro       # Driven by publications.json
    Contact.astro            # Contact grid + footer
  styles/global.css          # Design tokens, reset, utilities
  data/
    profile.json             # Links, emails, affiliations
    publications.json        # Publication entries
public/
  images/                    # Headshot + research figures
  pubs/                      # Publication PDFs
  cv.pdf                     # Built from LaTeX (see below)
  cv.html                    # HTML fallback
cv/
  cv.tex                     # LaTeX CV source
scripts/
  build-cv.sh                # Builds cv.tex → PDF + HTML
```

## TODO System

The site includes placeholder content marked with yellow TODO callouts. Each callout shows the **source file to edit** and a description of what's needed. Search for `class="todo"` in `.astro` files or `\cvtodo` in `cv/cv.tex` to find them all.

## CV Build

The CV is a LaTeX document at `cv/cv.tex`. To build locally:

```bash
# Requires pdflatex and pandoc
bash scripts/build-cv.sh
```

This runs `pdflatex` twice (for references), copies the PDF to `public/cv.pdf`, and generates an HTML fallback at `public/cv.html`.

## Editing Content

- **Bio text**: `src/components/About.astro`
- **Research descriptions**: `src/components/Research.astro`
- **Publications**: `src/data/publications.json`
- **Profile links/emails**: `src/data/profile.json`
- **Headshot**: replace `public/images/headshot.jpeg`
- **Research figures**: add to `public/images/figures/`
- **CV**: `cv/cv.tex`

## Deployment

Pushes to `main` trigger the GitHub Actions workflow (`.github/workflows/deploy.yml`) which:

1. Installs Node 22 + LaTeX + pandoc
2. Builds the CV (PDF + HTML)
3. Builds the Astro site
4. Deploys `dist/` to GitHub Pages

The site is configured for a project site at `/cysite1` (set in `astro.config.mjs`).

## Adding Images/Figures

Place files in `public/images/figures/`. Reference them in `Research.astro` using:

```astro
src: `${import.meta.env.BASE_URL}/images/figures/your-figure.png`
```

The `BASE_URL` prefix is required for the `/cysite1` subpath to work correctly.
