# Project Context

This document is a handoff guide for restoring project context in another AI coding environment. It summarizes the portfolio's architecture, design intent, fragile implementation contracts, recent changes, and likely future work.

## Current Project Shape

This is Chen Zai's static portfolio for GitHub Pages. The main site is a single-file app:

- `index.html` is the shipped entry point and contains all HTML, CSS, and JavaScript inline.
- There is no build system, bundler, package manager, or external JavaScript dependency.
- Project detail pages are standalone HTML files: `p_urban.html`, `p_cubik.html`, `p_forest.html`, `p_silent.html`, and `p_pet.html`.
- Lab experiments are standalone pages: `lab_brain_mom.html`, `blurdemo.html`, and `graph_test.html`.
- Static assets live in `images/`, `fonts/`, and `files/`.

The repository is prepared for GitHub Pages publishing from the `main` branch root. `README.md` intentionally stays minimal.

## Operating Constraints

Read `docs/ARCHITECTURE.md` before editing the site.

All normal site work should happen in `index.html` unless the user explicitly asks otherwise. Do not create separate CSS or JS files. Use vanilla JavaScript only.

After changing the site, open `index.html` in a browser and verify the affected section at both desktop width around `1280px` and mobile width around `375px`.

Do not casually change these load-bearing parts:

- `scroll-snap-type` on `html`
- `scroll-snap-align` on `section`
- `height: 100vh` on `.hero`, `.cv-section`, and `.projects-section`
- the module-level `mobileProjectsQuery` declaration
- the canvas repulsion radius formula: `Math.min(width, height) * 0.25`
- CSS custom property names under the `--projects-exp-*` project theme scope

## Section Architecture

The main scroll order is:

1. Hero: `.hero`, `100vh`
2. CV: `.cv-section`, `100vh`
3. Projects: `.projects-section`, `100vh`
4. Lab: `.lab-section`, currently styled as a full-height design lab, with responsive `auto` height below `1024px`

The page uses vertical scroll snapping:

- `html { scroll-snap-type: y mandatory; scroll-behavior: smooth; }`
- every `section` has `scroll-snap-align: start` and `scroll-snap-stop: always`
- `.cv-col-main` scrolls internally and explicitly disables snap inside itself

This means section heights, internal scrolling, and navigation behavior are coupled. Any change that makes full-screen sections taller than the viewport can create unpleasant snapping or unreachable content.

## Design Direction

The visual style is quiet, editorial, and portfolio-focused: mostly white/gray surfaces, restrained typography, and project-specific atmospheres. Avoid making it feel like a marketing landing page.

Important design choices:

- `Inter` is used for body and UI text.
- `Newsreader` is used for the hero/title voice.
- Local fonts `MonoMatrix` and `Cubic 11` exist in `fonts/`; `Cubic 11` supports decorative CJK moments.
- The hero is centered and minimal, with an interactive canvas graph behind it.
- The projects section is intentionally immersive: background image, translucent gradient overlays, and a right-side "tooth wall" project picker.
- The lab section uses a mosaic grid, not ordinary cards in a list.

Do not introduce large decorative gradient blobs/orbs or a generic SaaS-style card layout. The design should preserve the portfolio's sparse, tactile, research-design tone.

## Key JavaScript Systems

### Hero Typewriter

The hero title writes and cycles through:

- `Chen Zai`
- `陳在`
- a pinyin conversion sequence: `chen'` to `陳`, then `zai` to `在`

The `.is-latin` and `.is-cjk` classes switch typography and spacing. CJK variants need separate metrics, especially `letter-spacing`.

Reduced motion users get static text.

The tagline is separately typewritten:

`Ideas begin where disciplines meet in motion.`

### Hero Particle Graph

`<canvas id="hero-graph">` draws 27 floating keyword nodes and connection lines.

Behavior:

- nodes float and bounce inside the viewport
- nodes connect when closer than `250px`
- desktop hover within `50px` enlarges a node and shows its keyword label
- center repulsion keeps nodes away from the name
- on touch devices, `mouse.x` and `mouse.y` remain null, so the graph floats without hover labels

The repulsion radius is tuned and should remain `Math.min(width, height) * 0.25` unless a broader hero redesign is requested.

### Projects Experience

The projects section is controlled by a two-part contract:

- markup contains `<img class="projects-exp-bg-img" id="project-img-N">` elements and matching `.projects-exp-tooth.item` links with `data-target="project-img-N"`
- `projectsExpData` contains metadata and theme values keyed by those same image IDs

Each project data entry has:

- `kicker`
- `title`
- `summary`
- `type`
- `focus`
- `time`
- `tags`
- optional `github` or `youtube`
- `theme`

Each `theme` must include all 13 current keys:

- `wall`
- `overlayLeft`
- `overlayMid`
- `overlayFade`
- `overlayBottom`
- `overlayTop`
- `text`
- `textSoft`
- `textMuted`
- `tagBg`
- `tagBorder`
- `toothText`
- `filter`

`applyProjectsExpTheme()` maps those values to CSS variables on `.projects-exp-wrapper`. `renderProjectsExpInfo()` updates the visible background, text, tags, external link icon, and wrapper visibility.

Desktop interaction:

- `mouseenter` and focus reveal project info
- mouseleave hides the info panel

Mobile interaction:

- `mobileProjectsQuery = window.matchMedia('(max-width: 760px), (hover: none), (pointer: coarse)')`
- tapping a project prevents navigation on the first interaction and reveals details
- the first project is auto-activated on mobile
- when switching back to desktop, active mobile state is cleared

The external link control uses the same anchor for GitHub and YouTube. The `.is-youtube` class swaps which inline SVG icon is visible.

### CV Tabs

CV tabs use inline handlers like `onclick="toggleTab('education')"`.

`toggleTab(id)`:

- collapses all panels
- opens the matching `#tab-{id}` panel
- marks the active tab by reading each button's `onclick` attribute string
- waits for the `max-height` transition to finish before scrolling the internal CV container

This is fragile by design. If the inline `onclick` format changes, the active tab logic must also change.

### Navigation

Navigation links use smooth `scrollIntoView()` for hash anchors. An `IntersectionObserver` with threshold `0.4` updates active nav state.

There is no mobile hamburger menu yet. Below narrow widths, the nav can become crowded.

## Content Inventory

Current project entries:

- Project 01: Urban Mirage, `p_urban.html`, `images/urban.png`, GitHub `GhostTown`
- Project 02: CUBIK, `p_cubik.html`, `images/vr.png`, YouTube link
- Project 03: Fragments of the Forest, `p_forest.html`, `images/forest.png`, YouTube link
- Project 04: Silent Signal, `p_silent.html`, `images/silent.png`, GitHub `Facial-EMG-Stroke-Monitor`
- Project 05: Pet Space, `p_pet.html`, `images/pet.png`, GitHub `PetSpace`

Current lab cards:

- AI Context Sharing: links to `lab_brain_mom.html`
- Material Test: placeholder
- Sketch: placeholder
- Concept Art: placeholder
- Icon: placeholder
- 3D Render: placeholder
- Photography: placeholder

## Recent Change History

Recent commits on `main` describe the current direction:

- `cb6732a` refined mobile projects and lab visuals.
- `fe6650f` updated the portfolio bio and project links, including detail pages for Urban Mirage, Silent Signal, and Pet Space.
- `29928ed` prepared the portfolio for GitHub Pages.
- `3958350` added hero typewriter typography and local fonts.
- `f5aa014` refined project hover theming.
- `3b44752` added the contextual sharing lab page and reworked the lab/index presentation.
- `4ef0439` archived the earlier projects hover demo.
- `f118a02` was the initial commit.

There are also local context files (`AGENTS.md`, `CLAUDE.md`, `docs/ARCHITECTURE.md`) that encode editing rules and architectural warnings. Treat them as project documentation, not shipped site code.

## Known Incomplete Areas

These are known gaps and good candidates for future work if specifically requested:

- Add a mobile hamburger/collapsed nav pattern below roughly `600px`.
- Fix hero education card overflow below roughly `480px`; `min-width: 280px` can be too wide, and the cards likely need column behavior or a tighter min width.
- Investigate whether scroll snapping should be softened or disabled on mobile. Do this carefully because it affects the whole navigation model.
- Replace placeholder lab cards with real experiments, images, or standalone pages.
- Consider adding keyboard/ARIA polish to project tooth interactions so the immersive project picker is as strong for non-pointer navigation as it is visually.
- Audit performance on low-power mobile devices; the full-screen canvas animation runs continuously.

## Safe Edit Recipes

### Add A Project

1. Add a new `projectsExpData` entry keyed by the new image ID.
2. Include all current theme keys listed above.
3. Add a matching `<img id="project-img-N" class="projects-exp-bg-img">` inside `.projects-exp-bg-layer`.
4. Add a matching `.projects-exp-tooth.item` link inside `.projects-exp-list` with `data-target="project-img-N"`.
5. Do not renumber existing projects.
6. Test desktop hover/focus and mobile tap behavior.

### Add A Lab Card

1. Add the card inside `.lab-grid`.
2. Include `class="lab-card"`.
3. Use `grid-column: span N` or an `nth-child` rule if the mosaic layout needs a new span.
4. Test both the desktop grid and the flex-column layout below `1024px`.

### Edit CV Entries

1. Edit only the relevant `.cv-tab-panel`.
2. Preserve the existing `onclick="toggleTab('...')"` tab button format.
3. Check that the expanding panel scrolls correctly inside `.cv-col-main`.

### Add Or Change A Breakpoint

Existing breakpoints are `1024px`, `900px`, and `760px`.

If changing behavior at `760px`, check whether the JavaScript `mobileProjectsQuery` must change too. Keep related media queries near the existing block for the same breakpoint.

## Verification Checklist

Before handing off site changes:

- Open `index.html` at desktop width around `1280px`.
- Open `index.html` at mobile width around `375px`.
- Check scroll snapping between Hero, CV, Projects, and Lab.
- Check project hover on desktop and project tap on mobile.
- Check CV tabs open, collapse, and scroll internally.
- Check hero title/tagline animation or reduced-motion fallback if relevant.
- Check that no text overlaps or overflows narrow containers.

