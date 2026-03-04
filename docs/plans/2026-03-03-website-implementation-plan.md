# Chan-Ye Ohh Personal Website — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build and deploy a single-page academic website for Chan-Ye (Chris) Ohh using Astro, with a LaTeX CV pipeline, hosted on GitHub Pages.

**Architecture:** Astro static site with component-per-section design. All text content lives in Astro components (no CMS). Publication data is read from `publications.json` at build time. CV is built from LaTeX via a shell script and deployed alongside the site via GitHub Actions.

**Tech Stack:** Astro 5, vanilla CSS (custom properties), GitHub Pages, pdflatex, pandoc

**Design doc:** `docs/plans/2026-03-03-personal-website-design.md`
**Draft text:** `docs/plans/2026-03-03-draft-text.md`

---

## Cross-Cutting: TODO Display System

Several sections contain placeholder text that Chris needs to fill in. These **must be visually obvious on the rendered site** so she knows exactly what to edit and where.

### Implementation

Add this to `src/styles/global.css`:

```css
/* === Editable TODO callouts === */
.todo {
  background: #fff3cd;
  border-left: 4px solid #d4a574;
  border-radius: 0 var(--radius) var(--radius) 0;
  padding: 0.75rem 1rem;
  margin: 1rem 0;
  font-size: 0.9rem;
}

.todo__file {
  display: inline-block;
  font-family: 'SF Mono', 'Fira Code', monospace;
  font-size: 0.8rem;
  background: #f0e6d3;
  padding: 0.1rem 0.45rem;
  border-radius: 3px;
  color: #7d6340;
  margin-bottom: 0.3rem;
}

.todo__text {
  color: #664d03;
}
```

### Usage in Astro Components

Wherever a TODO exists, render a visible callout that tells Chris **which source file to edit** and **what to add**. Use this pattern:

```astro
<div class="todo">
  <span class="todo__file">src/components/About.astro</span>
  <p class="todo__text">
    Add a forward-looking paragraph about your research program for faculty applications.
  </p>
</div>
```

The `todo__file` span must contain the **exact relative path** from the project root to the source file she needs to edit. This way she can open the file directly.

### Where TODOs Exist

| Source File | What Chris Needs to Add | How It Appears |
|-------------|------------------------|----------------|
| `src/components/About.astro` | Verify/expand Scripps research paragraph (mesoscale eddies) | Yellow callout on page with file path |
| `src/components/About.astro` | Forward-looking research vision paragraph | Yellow callout on page with file path |
| `src/components/Research.astro` | Verify/expand Mesoscale Ocean Dynamics theme + add figures | Yellow callout on page with file path |
| `cv/cv.tex` | Education department name, additional degrees | Orange-bordered box in PDF with `cv/cv.tex` path |
| `cv/cv.tex` | Employment date ranges | Orange-bordered box in PDF with `cv/cv.tex` path |
| `cv/cv.tex` | Additional publications, invited seminars, teaching, awards | Orange-bordered box in PDF with `cv/cv.tex` path |

### Production Removal

These callouts are intentionally **not hidden** — they should be visible during development so nothing gets missed. When Chris is ready to publish, she fills in the content and deletes the `<div class="todo">` blocks. No build flag or environment variable needed.

---

## Task 1: Project Scaffolding

**Files:**
- Create: `package.json`
- Create: `astro.config.mjs`
- Create: `tsconfig.json`
- Create: `.gitignore`
- Copy assets into: `public/images/`, `public/pubs/`, `src/data/`

**Step 1: Initialize git repo**

```bash
cd /Users/ollie/Documents/website2
git init
```

**Step 2: Create Astro project**

```bash
npm create astro@latest . -- --template minimal --no-install --no-git
npm install
```

If `create astro` prompts or fails because the directory isn't empty, manually create the minimal files:

```bash
npm init -y
npm install astro
```

Then create `astro.config.mjs`:

```js
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://chanyeohh.github.io', // Update with actual GitHub username
});
```

And `tsconfig.json`:

```json
{
  "extends": "astro/tsconfigs/strict"
}
```

**Step 3: Create .gitignore**

```
node_modules/
dist/
.astro/
.DS_Store
cv/*.aux
cv/*.log
cv/*.out
cv/*.synctex.gz
```

**Step 4: Copy assets into project structure**

```bash
mkdir -p src/data src/pages src/components src/layouts src/styles
mkdir -p public/images/figures public/pubs

# Headshot
cp cy_ohh_assets/headshots/citations.jpeg public/images/headshot.jpeg

# Research figures
cp cy_ohh_assets/figures/JFM_2024_997_A43_Fig2_experimental_setup.png public/images/figures/
cp cy_ohh_assets/figures/JFM_2024_997_A43_Fig10_vorticity_trajectory.png public/images/figures/
cp cy_ohh_assets/figures/PRF_2022_024801_Fig1_regime_diagrams.png public/images/figures/
cp cy_ohh_assets/figures/PRF_2022_024801_Fig6_snapshots_DMD_modes.png public/images/figures/
cp cy_ohh_assets/figures/PRF_2022_024801_Fig13_TDMD_example.png public/images/figures/

# Publication PDFs
cp cy_ohh_assets/pubs/*.pdf public/pubs/

# Data files
cp cy_ohh_assets/metadata/publications.json src/data/
cp cy_ohh_assets/metadata/profile.json src/data/
```

**Step 5: Verify the project builds**

```bash
npx astro build
```

Expected: Build succeeds (empty site, no pages yet).

**Step 6: Commit**

```bash
git add -A
git commit -m "feat: scaffold Astro project and copy assets"
```

---

## Task 2: Global Styles & Base Layout

**Files:**
- Create: `src/styles/global.css`
- Create: `src/layouts/BaseLayout.astro`
- Create: `src/pages/index.astro` (minimal shell)

**Step 1: Create global.css**

Create `src/styles/global.css` with the full design system:

```css
/* === Design Tokens === */
:root {
  --color-primary: #1a5276;
  --color-bg: #fafafa;
  --color-accent: #d4a574;
  --color-section-alt: #e8f4f8;
  --color-text: #2c3e50;
  --color-subtle: #5d6d7e;
  --color-white: #ffffff;

  --font-body: 'Inter', system-ui, -apple-system, sans-serif;
  --font-heading: 'Lora', Georgia, serif;

  --max-width: 900px;
  --radius: 8px;
  --nav-height: 64px;
}

/* === Reset === */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  scroll-behavior: smooth;
  scroll-padding-top: var(--nav-height);
}

body {
  font-family: var(--font-body);
  font-size: 18px;
  line-height: 1.7;
  color: var(--color-text);
  background: var(--color-bg);
}

img {
  max-width: 100%;
  height: auto;
  display: block;
}

a {
  color: var(--color-primary);
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

/* === Typography === */
h1, h2, h3 {
  font-family: var(--font-heading);
  color: var(--color-primary);
  line-height: 1.3;
}

h1 { font-size: 2.2rem; }
h2 { font-size: 1.6rem; margin-bottom: 1rem; }
h3 { font-size: 1.25rem; }

p {
  margin-bottom: 1rem;
}

/* === Layout === */
.container {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: 0 1.5rem;
}

section {
  padding: 4rem 0;
}

section:nth-child(even) {
  background: var(--color-section-alt);
}

/* === Utilities === */
.badge {
  display: inline-block;
  font-size: 0.75rem;
  font-weight: 600;
  padding: 0.15rem 0.5rem;
  border-radius: 4px;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}

.badge--accent {
  background: var(--color-accent);
  color: var(--color-white);
}

.badge--primary {
  background: var(--color-primary);
  color: var(--color-white);
}

/* === Editable TODO callouts (see Cross-Cutting section) === */
.todo {
  background: #fff3cd;
  border-left: 4px solid #d4a574;
  border-radius: 0 var(--radius) var(--radius) 0;
  padding: 0.75rem 1rem;
  margin: 1rem 0;
  font-size: 0.9rem;
}

.todo__file {
  display: inline-block;
  font-family: 'SF Mono', 'Fira Code', monospace;
  font-size: 0.8rem;
  background: #f0e6d3;
  padding: 0.1rem 0.45rem;
  border-radius: 3px;
  color: #7d6340;
  margin-bottom: 0.3rem;
}

.todo__text {
  color: #664d03;
  margin-bottom: 0;
}

/* === Responsive === */
@media (max-width: 768px) {
  body { font-size: 16px; }
  h1 { font-size: 1.8rem; }
  h2 { font-size: 1.4rem; }
  section { padding: 2.5rem 0; }
}
```

**Step 2: Create BaseLayout.astro**

Create `src/layouts/BaseLayout.astro`:

```astro
---
interface Props {
  title: string;
  description: string;
}

const { title, description } = Astro.props;
---

<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content={description} />
    <title>{title}</title>

    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Lora:wght@500;600;700&display=swap"
      rel="stylesheet"
    />

    <!-- Favicon (placeholder) -->
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
  </head>
  <body>
    <slot />
  </body>
</html>

<style is:global>
  @import '../styles/global.css';
</style>
```

**Step 3: Create minimal index.astro**

Create `src/pages/index.astro`:

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout
  title="Chan-Ye (Chris) Ohh — Postdoctoral Scholar"
  description="Personal website of Chan-Ye (Chris) Ohh, postdoctoral researcher at Scripps Institution of Oceanography studying stratified ocean dynamics."
>
  <main>
    <div class="container">
      <h1>Site under construction</h1>
      <p>Sections will be added one at a time.</p>
    </div>
  </main>
</BaseLayout>
```

**Step 4: Create a placeholder favicon**

Create `public/favicon.svg`:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <text x="4" y="26" font-size="28" font-family="serif">C</text>
</svg>
```

**Step 5: Build and verify**

```bash
npx astro build && npx astro preview
```

Expected: Site loads at localhost, shows "Site under construction" with correct fonts and styles.

**Step 6: Commit**

```bash
git add -A
git commit -m "feat: add base layout, global styles, and font loading"
```

---

## Task 3: Nav Component

**Files:**
- Create: `src/components/Nav.astro`
- Modify: `src/pages/index.astro` (add Nav import)

**Step 1: Create Nav.astro**

Create `src/components/Nav.astro`:

```astro
---
const navItems = [
  { label: 'About', href: '#about' },
  { label: 'Research', href: '#research' },
  { label: 'Publications', href: '#publications' },
  { label: 'Contact', href: '#contact' },
];
---

<nav class="nav" id="nav">
  <div class="nav__inner container">
    <a class="nav__logo" href="#">C.-Y. Ohh</a>

    <button class="nav__toggle" aria-label="Toggle menu" aria-expanded="false">
      <span class="nav__toggle-bar"></span>
      <span class="nav__toggle-bar"></span>
      <span class="nav__toggle-bar"></span>
    </button>

    <ul class="nav__links" id="nav-links">
      {navItems.map((item) => (
        <li><a href={item.href}>{item.label}</a></li>
      ))}
      <li><a href="/pubs/cv.pdf" class="nav__cv-link">CV</a></li>
    </ul>
  </div>
</nav>

<style>
  .nav {
    position: sticky;
    top: 0;
    z-index: 100;
    background: var(--color-white);
    border-bottom: 1px solid #e0e0e0;
    height: var(--nav-height);
    display: flex;
    align-items: center;
  }

  .nav__inner {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
  }

  .nav__logo {
    font-family: var(--font-heading);
    font-size: 1.15rem;
    font-weight: 600;
    color: var(--color-primary);
  }

  .nav__logo:hover {
    text-decoration: none;
  }

  .nav__links {
    display: flex;
    list-style: none;
    gap: 1.5rem;
    align-items: center;
  }

  .nav__links a {
    font-size: 0.95rem;
    font-weight: 500;
    color: var(--color-text);
    transition: color 0.2s;
  }

  .nav__links a:hover {
    color: var(--color-primary);
    text-decoration: none;
  }

  .nav__cv-link {
    background: var(--color-primary);
    color: var(--color-white) !important;
    padding: 0.35rem 1rem;
    border-radius: var(--radius);
    font-weight: 600;
  }

  .nav__cv-link:hover {
    opacity: 0.9;
  }

  .nav__toggle {
    display: none;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0.5rem;
    flex-direction: column;
    gap: 5px;
  }

  .nav__toggle-bar {
    display: block;
    width: 24px;
    height: 2px;
    background: var(--color-text);
    transition: transform 0.2s;
  }

  @media (max-width: 768px) {
    .nav__toggle {
      display: flex;
    }

    .nav__links {
      display: none;
      position: absolute;
      top: var(--nav-height);
      left: 0;
      right: 0;
      background: var(--color-white);
      flex-direction: column;
      padding: 1rem 1.5rem;
      border-bottom: 1px solid #e0e0e0;
      gap: 0.75rem;
    }

    .nav__links.is-open {
      display: flex;
    }
  }
</style>

<script>
  const toggle = document.querySelector('.nav__toggle');
  const links = document.getElementById('nav-links');

  toggle?.addEventListener('click', () => {
    const isOpen = links?.classList.toggle('is-open');
    toggle.setAttribute('aria-expanded', String(isOpen));
  });

  // Close mobile menu on link click
  links?.querySelectorAll('a').forEach((link) => {
    link.addEventListener('click', () => {
      links.classList.remove('is-open');
      toggle?.setAttribute('aria-expanded', 'false');
    });
  });
</script>
```

**Step 2: Add Nav to index.astro**

Modify `src/pages/index.astro` to import and use Nav:

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
import Nav from '../components/Nav.astro';
---

<BaseLayout
  title="Chan-Ye (Chris) Ohh — Postdoctoral Scholar"
  description="Personal website of Chan-Ye (Chris) Ohh, postdoctoral researcher at Scripps Institution of Oceanography studying stratified ocean dynamics."
>
  <Nav />
  <main>
    <div class="container">
      <p>Sections coming next.</p>
    </div>
  </main>
</BaseLayout>
```

**Step 3: Build and verify**

```bash
npx astro build
```

Expected: Builds without errors. Sticky nav renders at top.

**Step 4: Commit**

```bash
git add src/components/Nav.astro src/pages/index.astro
git commit -m "feat: add sticky nav with mobile hamburger menu"
```

---

## Task 4: Hero Component

**Files:**
- Create: `src/components/Hero.astro`
- Modify: `src/pages/index.astro` (add Hero import)

**Step 1: Create Hero.astro**

Create `src/components/Hero.astro`:

```astro
---
import profile from '../data/profile.json';
---

<section class="hero" id="hero">
  <div class="hero__inner container">
    <div class="hero__text">
      <h1 class="hero__name">Chan-Ye (Chris) Ohh</h1>
      <p class="hero__title">Postdoctoral Scholar</p>
      <p class="hero__affiliation">Marine Physical Laboratory, Scripps Institution of Oceanography, UC San Diego</p>
      <p class="hero__tagline">
        Experimental fluid mechanics &amp; data-driven methods for understanding ocean flows
      </p>
      <div class="hero__links">
        <a href={profile.google_scholar} target="_blank" rel="noopener">Google Scholar</a>
        <a href={`https://orcid.org/${profile.orcid}`} target="_blank" rel="noopener">ORCID</a>
        <a href={`mailto:${profile.emails_public[1].email}`}>Email</a>
        <a href={profile.links.linkedin} target="_blank" rel="noopener">LinkedIn</a>
      </div>
    </div>
    <div class="hero__photo">
      <img
        src="/images/headshot.jpeg"
        alt="Chan-Ye (Chris) Ohh"
        width="280"
        height="280"
      />
    </div>
  </div>
</section>

<style>
  .hero {
    padding: 4rem 0 3rem;
    background: var(--color-white);
  }

  .hero__inner {
    display: flex;
    align-items: center;
    gap: 3rem;
  }

  .hero__text {
    flex: 1;
  }

  .hero__name {
    margin-bottom: 0.25rem;
  }

  .hero__title {
    font-size: 1.15rem;
    font-weight: 500;
    color: var(--color-primary);
    margin-bottom: 0.1rem;
  }

  .hero__affiliation {
    font-size: 1rem;
    color: var(--color-subtle);
    margin-bottom: 0.75rem;
  }

  .hero__tagline {
    font-size: 1.05rem;
    font-style: italic;
    color: var(--color-text);
    margin-bottom: 1.25rem;
  }

  .hero__links {
    display: flex;
    flex-wrap: wrap;
    gap: 0.75rem;
  }

  .hero__links a {
    font-size: 0.9rem;
    font-weight: 500;
    padding: 0.3rem 0.85rem;
    border: 1.5px solid var(--color-primary);
    border-radius: var(--radius);
    transition: background 0.2s, color 0.2s;
  }

  .hero__links a:hover {
    background: var(--color-primary);
    color: var(--color-white);
    text-decoration: none;
  }

  .hero__photo img {
    width: 280px;
    height: 280px;
    border-radius: 50%;
    object-fit: cover;
    border: 4px solid var(--color-section-alt);
  }

  @media (max-width: 768px) {
    .hero__inner {
      flex-direction: column-reverse;
      text-align: center;
      gap: 1.5rem;
    }

    .hero__links {
      justify-content: center;
    }

    .hero__photo img {
      width: 200px;
      height: 200px;
    }
  }
</style>
```

**Step 2: Add Hero to index.astro**

Update the `<main>` in `src/pages/index.astro` to include Hero after Nav:

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
import Nav from '../components/Nav.astro';
import Hero from '../components/Hero.astro';
---

<BaseLayout
  title="Chan-Ye (Chris) Ohh — Postdoctoral Scholar"
  description="Personal website of Chan-Ye (Chris) Ohh, postdoctoral researcher at Scripps Institution of Oceanography studying stratified ocean dynamics."
>
  <Nav />
  <main>
    <Hero />
  </main>
</BaseLayout>
```

**Step 3: Build and verify**

```bash
npx astro build
```

Expected: Builds. Hero shows name, title, tagline, circular headshot, link buttons.

**Step 4: Commit**

```bash
git add src/components/Hero.astro src/pages/index.astro
git commit -m "feat: add hero section with headshot and quick links"
```

---

## Task 5: About Component

**Files:**
- Create: `src/components/About.astro`
- Modify: `src/pages/index.astro` (add About import)

**Step 1: Create About.astro**

Create `src/components/About.astro`:

```astro
<section class="about" id="about">
  <div class="container">
    <h2>About</h2>

    <p>
      I am a postdoctoral researcher in the Air-Sea Interaction Laboratory at
      Scripps Institution of Oceanography, UC San Diego. My research combines
      laboratory experiments and data-driven analysis to understand how density
      stratification shapes fluid flows in the ocean.
    </p>

    <p>
      I received my PhD from the University of Southern California in 2023,
      where I worked with Prof. Geoffrey Spedding in the Stratified Fluids
      Laboratory. My dissertation focused on the wakes generated by bluff
      bodies moving through stratified environments &mdash; a problem with
      direct connections to underwater vehicle hydrodynamics and oceanic
      turbulence. Using towed experiments in a large stratified water tank
      and dynamic mode decomposition (DMD), I developed new approaches for
      identifying and classifying wake regimes from limited flow measurements.
    </p>

    <p>
      At Scripps, I work on mesoscale ocean eddy characterization, integrating
      satellite altimetry observations with modeling techniques to advance our
      understanding of ocean circulation and transport.
    </p>

    <div class="todo">
      <span class="todo__file">src/components/About.astro</span>
      <p class="todo__text">
        Verify and expand the Scripps research paragraph above &mdash; does it
        accurately describe your current work?
      </p>
    </div>

    <div class="todo">
      <span class="todo__file">src/components/About.astro</span>
      <p class="todo__text">
        Add a forward-looking paragraph for faculty applications, e.g.:
        "My research program aims to bridge lab-scale stratified flow experiments
        with field observations, developing new measurement and data-driven
        techniques for understanding ocean mixing and transport processes."
      </p>
    </div>
  </div>
</section>

<style>
  .about {
    background: var(--color-section-alt);
  }
</style>
```

**Step 2: Add About to index.astro**

Add `import About from '../components/About.astro';` to the frontmatter and `<About />` after `<Hero />`.

**Step 3: Build and verify**

```bash
npx astro build
```

Expected: Builds. About section renders below hero with alt background color.

**Step 4: Commit**

```bash
git add src/components/About.astro src/pages/index.astro
git commit -m "feat: add about section with bio text"
```

---

## Task 6: Research Section

**Files:**
- Create: `src/components/ResearchTheme.astro`
- Create: `src/components/Research.astro`
- Modify: `src/pages/index.astro` (add Research import)

**Step 1: Create ResearchTheme.astro**

Create `src/components/ResearchTheme.astro` — a reusable card for each research theme:

```astro
---
interface Props {
  title: string;
  figures: { src: string; alt: string }[];
  reverse?: boolean;
}

const { title, figures, reverse = false } = Astro.props;
---

<div class:list={['theme', { 'theme--reverse': reverse }]}>
  <div class="theme__text">
    <h3>{title}</h3>
    <slot />
  </div>
  <div class="theme__figures">
    {figures.map((fig) => (
      <img src={fig.src} alt={fig.alt} loading="lazy" />
    ))}
  </div>
</div>

<style>
  .theme {
    display: flex;
    gap: 2rem;
    align-items: flex-start;
    margin-bottom: 3rem;
  }

  .theme--reverse {
    flex-direction: row-reverse;
  }

  .theme__text {
    flex: 1;
  }

  .theme__text h3 {
    margin-bottom: 0.75rem;
  }

  .theme__figures {
    flex: 0 0 340px;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .theme__figures img {
    border-radius: var(--radius);
    border: 1px solid #e0e0e0;
  }

  @media (max-width: 768px) {
    .theme,
    .theme--reverse {
      flex-direction: column;
    }

    .theme__figures {
      flex: none;
      width: 100%;
    }
  }
</style>
```

**Step 2: Create Research.astro**

Create `src/components/Research.astro`:

```astro
---
import ResearchTheme from './ResearchTheme.astro';
---

<section class="research" id="research">
  <div class="container">
    <h2>Research</h2>

    <ResearchTheme
      title="Stratified Ocean Wakes"
      figures={[
        {
          src: '/images/figures/JFM_2024_997_A43_Fig2_experimental_setup.png',
          alt: 'Tow-tank experimental setup for stratified wake measurements',
        },
        {
          src: '/images/figures/JFM_2024_997_A43_Fig10_vorticity_trajectory.png',
          alt: 'Streamwise vorticity and vortex trajectory measurements',
        },
      ]}
    >
      <p>
        When objects move through density-stratified water &mdash; as submarines,
        autonomous underwater vehicles, and marine organisms do in the ocean &mdash;
        their wakes behave fundamentally differently than in uniform fluid. Buoyancy
        forces suppress vertical motions, reorganize vortex structures, and create
        distinctive internal wave signatures that can persist far downstream.
      </p>
      <p>
        My experimental work uses a large tow tank with precisely controlled density
        stratification to study these wakes systematically. By towing scaled models
        through stably stratified water and measuring the resulting flow fields with
        particle image velocimetry (PIV), I have mapped how wake structure transitions
        across the Reynolds number and Froude number parameter space.
      </p>
      <p class="theme__papers">
        <strong>Related:</strong>
        <a href="https://doi.org/10.1017/jfm.2024.829" target="_blank" rel="noopener">Ohh &amp; Spedding (2024) <em>JFM</em></a>,
        <a href="https://doi.org/10.1103/PhysRevFluids.7.024801" target="_blank" rel="noopener">Ohh &amp; Spedding (2022) <em>PRF</em></a>
      </p>
    </ResearchTheme>

    <ResearchTheme
      title="Data-Driven Flow Identification"
      reverse
      figures={[
        {
          src: '/images/figures/PRF_2022_024801_Fig6_snapshots_DMD_modes.png',
          alt: 'DMD mode comparison between simulation and experiment',
        },
        {
          src: '/images/figures/PRF_2022_024801_Fig13_TDMD_example.png',
          alt: 'Time-resolved dynamic mode decomposition analysis',
        },
      ]}
    >
      <p>
        Real ocean measurements are sparse &mdash; sensors are expensive, and you
        rarely have the luxury of full-field data. How do you identify what kind of
        flow you are observing from just a handful of measurement points?
      </p>
      <p>
        I develop data-driven methods that extract physically meaningful flow
        structures from limited experimental data. My primary tool is dynamic mode
        decomposition (DMD), a technique that decomposes unsteady flow fields into
        spatiotemporal modes. Applied to stratified wake experiments, DMD reveals the
        dominant oscillatory structures that characterize different wake regimes.
      </p>
      <p>
        My 2022 paper on this topic was selected as an
        <span class="badge badge--accent">Editor's Suggestion</span>
        in <em>Physical Review Fluids</em>.
      </p>
      <p class="theme__papers">
        <strong>Related:</strong>
        <a href="https://doi.org/10.1103/PhysRevFluids.7.024801" target="_blank" rel="noopener">Ohh &amp; Spedding (2022) <em>PRF</em></a>,
        <a href="https://doi.org/10.1103/PhysRevFluids.7.033803" target="_blank" rel="noopener">Chinta, Ohh et al. (2022) <em>PRF</em></a>
      </p>
    </ResearchTheme>

    <ResearchTheme
      title="Mesoscale Ocean Dynamics"
      figures={[]}
    >
      <p>
        At Scripps Institution of Oceanography, I am advancing methods for
        characterizing mesoscale ocean eddies &mdash; rotating structures
        spanning tens to hundreds of kilometers that play a central role in
        ocean heat transport, nutrient distribution, and carbon cycling. By
        integrating satellite altimetry observations with modeling techniques,
        I aim to improve how we detect, track, and understand these features
        across the global ocean.
      </p>
      <div class="todo">
        <span class="todo__file">src/components/Research.astro</span>
        <p class="todo__text">
          Verify and expand this section. Add specific methods, results, or
          broader impacts. Add figures when available.
        </p>
      </div>
      <p class="theme__papers">
        <strong>Lab:</strong>
        <a href="https://airsea.ucsd.edu/people/" target="_blank" rel="noopener">Air-Sea Interaction Laboratory</a>
      </p>
    </ResearchTheme>
  </div>
</section>
```

**Step 3: Add Research to index.astro**

Add `import Research from '../components/Research.astro';` and `<Research />` after `<About />`.

**Step 4: Build and verify**

```bash
npx astro build
```

Expected: Builds. Three research theme cards render with alternating figure placement.

**Step 5: Commit**

```bash
git add src/components/ResearchTheme.astro src/components/Research.astro src/pages/index.astro
git commit -m "feat: add research section with three themed blocks and figures"
```

---

## Task 7: Publications Component

**Files:**
- Create: `src/components/Publications.astro`
- Modify: `src/pages/index.astro` (add Publications import)

**Step 1: Create Publications.astro**

Create `src/components/Publications.astro`:

```astro
---
import publications from '../data/publications.json';

function formatAuthors(authors: string[]): string[] {
  return authors.map((a) => (a === 'Chan-Ye Ohh' ? `<strong>${a}</strong>` : a));
}

function getBadges(pub: any): { label: string; class: string }[] {
  const badges: { label: string; class: string }[] = [];
  for (const note of pub.notes || []) {
    if (note.toLowerCase().includes('editor')) badges.push({ label: "Editor's Suggestion", class: 'badge--accent' });
    if (note.toLowerCase().includes('open access')) badges.push({ label: 'Open Access', class: 'badge--primary' });
  }
  return badges;
}
---

<section class="publications" id="publications">
  <div class="container">
    <h2>Publications</h2>

    <ol class="pub-list">
      {publications.map((pub) => (
        <li class="pub-item">
          <p class="pub-item__authors" set:html={formatAuthors(pub.authors).join(', ')} />
          <p class="pub-item__title">
            <a href={pub.url || `https://doi.org/${pub.doi}`} target="_blank" rel="noopener">
              {pub.title}
            </a>
          </p>
          <p class="pub-item__venue">
            <em>{pub.venue}</em>
            {pub.volume && `, ${pub.volume}`}
            {pub.pages_or_article && `, ${pub.pages_or_article}`}
            {` (${pub.year})`}
          </p>
          <div class="pub-item__meta">
            {getBadges(pub).map((b) => (
              <span class={`badge ${b.class}`}>{b.label}</span>
            ))}
            {pub.doi && (
              <a href={`https://doi.org/${pub.doi}`} target="_blank" rel="noopener" class="pub-item__link">
                DOI
              </a>
            )}
            {pub.pdf && (
              <a href={`/${pub.pdf}`} class="pub-item__link">PDF</a>
            )}
            {pub.preprint?.url && (
              <a href={pub.preprint.url} target="_blank" rel="noopener" class="pub-item__link">
                arXiv
              </a>
            )}
          </div>
        </li>
      ))}
    </ol>
  </div>
</section>

<style>
  .publications {
    background: var(--color-section-alt);
  }

  .pub-list {
    list-style: none;
    counter-reset: pub-counter;
  }

  .pub-item {
    counter-increment: pub-counter;
    padding: 1.25rem 0;
    border-bottom: 1px solid #d5e8ee;
  }

  .pub-item:last-child {
    border-bottom: none;
  }

  .pub-item__authors {
    font-size: 0.95rem;
    margin-bottom: 0.2rem;
  }

  .pub-item__title {
    margin-bottom: 0.2rem;
  }

  .pub-item__title a {
    font-weight: 500;
  }

  .pub-item__venue {
    font-size: 0.9rem;
    color: var(--color-subtle);
    margin-bottom: 0.4rem;
  }

  .pub-item__meta {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    align-items: center;
  }

  .pub-item__link {
    font-size: 0.85rem;
    font-weight: 500;
    padding: 0.15rem 0.5rem;
    border: 1px solid var(--color-primary);
    border-radius: 4px;
  }

  .pub-item__link:hover {
    background: var(--color-primary);
    color: var(--color-white);
    text-decoration: none;
  }
</style>
```

**Step 2: Add Publications to index.astro**

Add `import Publications from '../components/Publications.astro';` and `<Publications />` after `<Research />`.

**Step 3: Build and verify**

```bash
npx astro build
```

Expected: Builds. Publications render as a styled list with badges and links.

**Step 4: Commit**

```bash
git add src/components/Publications.astro src/pages/index.astro
git commit -m "feat: add publications section driven by publications.json"
```

---

## Task 8: Contact Component

**Files:**
- Create: `src/components/Contact.astro`
- Modify: `src/pages/index.astro` (add Contact import)

**Step 1: Create Contact.astro**

Create `src/components/Contact.astro`:

```astro
---
import profile from '../data/profile.json';

const email = profile.emails_public[1].email; // cohh@ucsd.edu
---

<section class="contact" id="contact">
  <div class="container contact__inner">
    <h2>Contact</h2>

    <div class="contact__grid">
      <div class="contact__block">
        <h3>Email</h3>
        <p><a href={`mailto:${email}`}>{email}</a></p>
      </div>

      <div class="contact__block">
        <h3>Institution</h3>
        <p>
          <a href={profile.links.scripps_profile} target="_blank" rel="noopener">Scripps Institution of Oceanography</a>
        </p>
        <p>
          <a href={profile.links.airsea_lab_people} target="_blank" rel="noopener">Air-Sea Interaction Laboratory</a>
        </p>
      </div>

      <div class="contact__block">
        <h3>Profiles</h3>
        <p><a href={profile.google_scholar} target="_blank" rel="noopener">Google Scholar</a></p>
        <p><a href={`https://orcid.org/${profile.orcid}`} target="_blank" rel="noopener">ORCID</a></p>
        <p><a href={profile.links.researchgate} target="_blank" rel="noopener">ResearchGate</a></p>
        <p><a href={profile.links.linkedin} target="_blank" rel="noopener">LinkedIn</a></p>
      </div>
    </div>
  </div>
</section>

<footer class="footer">
  <div class="container">
    <p>&copy; {new Date().getFullYear()} Chan-Ye Ohh</p>
  </div>
</footer>

<style>
  .contact__grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
    margin-top: 1.5rem;
  }

  .contact__block h3 {
    margin-bottom: 0.5rem;
    font-size: 1.05rem;
  }

  .contact__block p {
    margin-bottom: 0.3rem;
    font-size: 0.95rem;
  }

  .footer {
    padding: 1.5rem 0;
    text-align: center;
    font-size: 0.85rem;
    color: var(--color-subtle);
    border-top: 1px solid #e0e0e0;
  }

  @media (max-width: 768px) {
    .contact__grid {
      grid-template-columns: 1fr;
      gap: 1.5rem;
    }
  }
</style>
```

**Step 2: Add Contact to index.astro**

Add `import Contact from '../components/Contact.astro';` and `<Contact />` after `<Publications />`.

**Step 3: Build and verify**

```bash
npx astro build
```

Expected: Builds. Contact section with three columns and footer at bottom.

**Step 4: Commit**

```bash
git add src/components/Contact.astro src/pages/index.astro
git commit -m "feat: add contact section and footer"
```

---

## Task 9: LaTeX CV Source & Build Script

**Files:**
- Create: `cv/cv.tex`
- Create: `scripts/build-cv.sh`

**Step 1: Create cv/cv.tex**

Create the directory and LaTeX source:

```bash
mkdir -p cv scripts
```

Create `cv/cv.tex`:

```latex
\documentclass[11pt, letterpaper]{article}

% --- Packages ---
\usepackage[margin=0.8in]{geometry}
\usepackage[T1]{fontenc}
\usepackage{palatino}
\usepackage{enumitem}
\usepackage{titlesec}
\usepackage[hidelinks]{hyperref}
\usepackage{xcolor}

% --- Colors ---
\definecolor{heading}{HTML}{1a5276}

% --- Section formatting ---
\titleformat{\section}{\large\bfseries\color{heading}}{}{0em}{}[\titlerule]
\titlespacing*{\section}{0pt}{1.2em}{0.6em}

% --- List formatting ---
\setlist[itemize]{left=0pt, nosep, label={}}

% --- Commands ---
\newcommand{\cventry}[4]{%
  \noindent\textbf{#1} \hfill #2 \\
  \textit{#3} \hfill \textit{#4} \\[0.4em]
}

\newcommand{\cvpub}[1]{\noindent #1 \\[0.4em]}

% --- TODO marker (renders as highlighted box in PDF, edit cv/cv.tex to resolve) ---
\newcommand{\cvtodo}[1]{%
  \noindent\fcolorbox{orange}{yellow!20}{%
    \parbox{\dimexpr\textwidth-2\fboxsep-2\fboxrule}{%
      \small\textbf{TODO} \textit{(edit \texttt{cv/cv.tex}):} #1%
    }%
  }\\[0.6em]%
}

% --- Document ---
\begin{document}

\begin{center}
  {\LARGE\bfseries Chan-Ye (Chris) Ohh} \\[0.3em]
  Postdoctoral Scholar, Marine Physical Laboratory, Scripps Institution of Oceanography \\
  \href{mailto:cohh@ucsd.edu}{cohh@ucsd.edu} \quad
  \href{https://orcid.org/0000-0002-7313-4291}{ORCID: 0000-0002-7313-4291} \\
  \href{https://scholar.google.com/citations?user=3VvK59oAAAAJ}{Google Scholar}
\end{center}

\section{Education}

\cventry{University of Southern California}{Los Angeles, CA}{Ph.D., [Department]}{2023}

\cvtodo{Fill in department name and any additional degrees (B.S., M.S., etc.)}

\section{Research Experience}

\cventry{Postdoctoral Researcher}{La Jolla, CA}
  {Air-Sea Interaction Laboratory, Scripps Institution of Oceanography, UC San Diego}{[Year]--Present}

\cventry{Graduate Research Assistant}{Los Angeles, CA}
  {Stratified Fluids Laboratory, University of Southern California}{[Year]--[Year]}

\cvtodo{Fill in start/end years for both positions above.}

\section{Publications}

\cvpub{%
  \textbf{C.-Y. Ohh} and G.\,R. Spedding (2024).
  ``The effects of stratification on the near wake of 6:1 prolate spheroid.''
  \textit{Journal of Fluid Mechanics}, 997, A43.
  \href{https://doi.org/10.1017/jfm.2024.829}{doi:10.1017/jfm.2024.829}.
  \textit{Open Access (CC BY 4.0).}
}

\cvpub{%
  \textbf{C.-Y. Ohh} and G.\,R. Spedding (2022).
  ``Wake identification of stratified flows using dynamic mode decomposition.''
  \textit{Physical Review Fluids}, 7, 024801.
  \href{https://doi.org/10.1103/PhysRevFluids.7.024801}{doi:10.1103/PhysRevFluids.7.024801}.
  \textit{Editor's Suggestion.}
}

\cvpub{%
  V.\,K. Chinta, \textbf{C.-Y. Ohh}, G. Spedding, and M. Luhar (2022).
  ``Regime identification for stratified wakes from limited measurements:
    a library-based sparse regression formulation.''
  \textit{Physical Review Fluids}, 7, 033803.
  \href{https://doi.org/10.1103/PhysRevFluids.7.033803}{doi:10.1103/PhysRevFluids.7.033803}.
}

\cvtodo{Add any additional publications not listed above.}

\section{Presentations}

\cvpub{%
  \textbf{C.-Y. Ohh} (2024).
  ``Advancing mesoscale ocean eddy characterization: integrating altimetry observations with modeling techniques.''
  APS Division of Fluid Dynamics Meeting (DFD24), Salt Lake City, UT. \textit{(Oral)}
}

\cvpub{%
  \textbf{C.-Y. Ohh} and G.\,R. Spedding (2023).
  ``The near-wake of an inclined 6:1 prolate spheroid in uniform and stratified flows.''
  APS Division of Fluid Dynamics Meeting (DFD23), Washington, DC.
}

\cvpub{%
  \textbf{C.-Y. Ohh} and G.\,R. Spedding (2021).
  ``Regime classification for stratified wakes using a convolutional neural network.''
  APS Division of Fluid Dynamics Meeting (DFD21), Phoenix, AZ. \textit{(Poster)}
}

\cvpub{%
  \textbf{C.-Y. Ohh} and G.\,R. Spedding (2020).
  ``Automated stratified wake classification using dynamic mode decomposition.''
  APS Division of Fluid Dynamics Meeting (DFD20), Virtual. \textit{(Poster)}
}

\cvpub{%
  \textbf{C.-Y. Ohh} and G.\,R. Spedding (2019).
  ``Wake identification of stratified flows using dynamic mode decomposition.''
  APS Division of Fluid Dynamics Meeting (DFD19), Seattle, WA. \textit{(Oral)}
}

\cvpub{%
  \textbf{C.-Y. Ohh}, Y. Huang, Z. Wen, E. Kanso, and M. Luhar (2016).
  ``Leveraging fluid-structure interaction for passive control of flapping locomotion.''
  APS Division of Fluid Dynamics Meeting (DFD16), Portland, OR.
}

\cvtodo{Add any invited seminars or additional presentations.}

\section{Awards \& Honors}

\noindent Student Poster Award, APS Division of Fluid Dynamics Meeting \hfill 2021 \\[0.4em]

\cvtodo{Add any additional awards, fellowships, or honors.}

\cvtodo{Add additional sections as needed: Teaching, Service \& Outreach, Skills.}

\end{document}
```

**Step 2: Create scripts/build-cv.sh**

Create `scripts/build-cv.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(dirname "$SCRIPT_DIR")"
CV_DIR="$PROJECT_ROOT/cv"
OUT_DIR="$PROJECT_ROOT/public"

echo "==> Building CV PDF..."
cd "$CV_DIR"
pdflatex -interaction=nonstopmode cv.tex
pdflatex -interaction=nonstopmode cv.tex  # second pass for references

echo "==> Copying PDF to public/"
cp cv.pdf "$OUT_DIR/cv.pdf"

echo "==> Building CV HTML fallback..."
pandoc cv.tex -s -o "$OUT_DIR/cv.html" \
  --metadata title="CV — Chan-Ye (Chris) Ohh"

echo "==> Done. Outputs:"
echo "    $OUT_DIR/cv.pdf"
echo "    $OUT_DIR/cv.html"
```

**Step 3: Make script executable and test**

```bash
chmod +x scripts/build-cv.sh
./scripts/build-cv.sh
```

Expected: `public/cv.pdf` and `public/cv.html` are created without errors.

**Step 4: Verify the PDF renders correctly**

```bash
ls -la public/cv.pdf public/cv.html
```

Expected: Both files exist and are non-empty.

**Step 5: Commit**

```bash
git add cv/cv.tex scripts/build-cv.sh public/cv.pdf public/cv.html
git commit -m "feat: add LaTeX CV source and build script (PDF + HTML)"
```

---

## Task 10: GitHub Actions Deploy Workflow

**Files:**
- Create: `.github/workflows/deploy.yml`

**Step 1: Create the workflow**

```bash
mkdir -p .github/workflows
```

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install dependencies
        run: npm ci

      - name: Install LaTeX
        run: |
          sudo apt-get update
          sudo apt-get install -y texlive-latex-base texlive-latex-recommended texlive-fonts-recommended pandoc

      - name: Build CV
        run: bash scripts/build-cv.sh

      - name: Build Astro site
        run: npx astro build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Step 2: Commit**

```bash
git add .github/workflows/deploy.yml
git commit -m "feat: add GitHub Actions workflow for Astro + CV build and deploy"
```

---

## Task 11: Final Assembly & Polish

**Files:**
- Modify: `src/pages/index.astro` (ensure all components assembled)
- Verify full build

**Step 1: Verify final index.astro has all sections**

`src/pages/index.astro` should look like:

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
import Nav from '../components/Nav.astro';
import Hero from '../components/Hero.astro';
import About from '../components/About.astro';
import Research from '../components/Research.astro';
import Publications from '../components/Publications.astro';
import Contact from '../components/Contact.astro';
---

<BaseLayout
  title="Chan-Ye (Chris) Ohh — Postdoctoral Scholar"
  description="Personal website of Chan-Ye (Chris) Ohh, postdoctoral researcher at Scripps Institution of Oceanography studying stratified ocean dynamics."
>
  <Nav />
  <main>
    <Hero />
    <About />
    <Research />
    <Publications />
    <Contact />
  </main>
</BaseLayout>
```

**Step 2: Full build**

```bash
npx astro build
```

Expected: Clean build, all pages generated, no warnings.

**Step 3: Preview and visual check**

```bash
npx astro preview
```

Verify:
- Nav sticks to top on scroll
- Hero renders with circular headshot and link buttons
- About section has alt background
- Research shows alternating figure/text cards
- Publications list with badges and DOI links
- Contact grid with three columns
- Footer at bottom
- Mobile: hamburger menu works, single-column layout

**Step 4: Commit**

```bash
git add -A
git commit -m "feat: assemble all sections, final polish"
```

---

## Summary

| Task | What | Depends On |
|------|------|-----------|
| 1 | Project scaffolding, asset copy | — |
| 2 | Global CSS, BaseLayout, fonts | 1 |
| 3 | Nav (sticky, mobile hamburger) | 2 |
| 4 | Hero (headshot, links, tagline) | 2 |
| 5 | About (bio text) | 2 |
| 6 | Research (3 themed cards + figures) | 2 |
| 7 | Publications (data-driven from JSON) | 2 |
| 8 | Contact + footer | 2 |
| 9 | LaTeX CV + build script | 1 |
| 10 | GitHub Actions deploy | 1 |
| 11 | Final assembly + polish | 3-10 |

Tasks 3-8 are independent of each other (all depend only on Task 2). Task 9-10 are independent of the component tasks. Task 11 is the final integration step.
