# Architecture

## Project shape

Single-file portfolio: all CSS and JS live inline inside `index.html`. No build system, no npm, no bundler. What you see is what ships.

Individual project detail pages are separate HTML files: `p_cubik.html`, `p_forest.html`, `p_pet.html`, `p_silent.html`, `p_urban.html`. Lab experiments are also standalone: `blurdemo.html`, `graph_test.html`, `lab_brain_mom.html`.

```
website_codex/
├── index.html          ← entire portfolio (CSS + JS inline)
├── p_*.html            ← project detail pages
├── fonts/              ← MonoMatrix.ttf, Cubic_11.ttf (local)
├── images/             ← project images referenced by JS data
└── files/              ← downloadable assets (CV PDF etc.)
```

## Sections (in scroll order)

| Section | Element | Height | Notes |
|---------|---------|--------|-------|
| Hero | `.hero` | 100vh | Canvas particle graph background |
| CV | `.cv-section` | 100vh | 3-col grid; only `.cv-col-main` scrolls internally |
| Projects | `.projects-section` | 100vh | Hover→info panel on desktop; tap on mobile |
| Lab | `.lab-section` | auto | Grid of experiment cards |

`scroll-snap-type: y mandatory` is set on `html`. Every `section` has `scroll-snap-align: start`. This is intentional — changing section heights breaks snap behavior.

## CSS variables

Global palette in `:root`:
```
--bg-primary, --bg-secondary, --text-primary, --text-secondary, --accent, --border
```

Projects section overrides these per-project via inline JS (`applyProjectsExpTheme`). Each project entry in `projectsExpData` carries its own `theme` object with 11 color keys (`wall`, `overlayLeft/Mid/Fade/Bottom/Top`, `text/textSoft/textMuted`, `tagBg/tagBorder`, `toothText`, `filter`).

## Fonts

- `Inter` — Google Fonts, body/UI
- `Newsreader` — Google Fonts, headings and taglines
- `MonoMatrix` — local TTF, decorative
- `Cubic 11` — local TTF, decorative CJK

## Key JS systems

### Hero particle graph (`<canvas id="hero-graph">`)
- 27 keyword nodes float and connect when within 250px
- Center repulsion keeps nodes away from the name (radius = 25% of min viewport dimension)
- Mouse hover within 50px enlarges node and shows tooltip
- Touch devices: `mouse.x/y` stay null → no hover state, nodes just float

### Typewriter (`typewriter-title`)
- Cycles through name variants (Latin + CJK) with slide-up exit animation
- CJK variant switches font-family and letter-spacing via `.is-cjk` class

### Projects experience
- Desktop: `mouseenter` on `.projects-exp-item` triggers `renderProjectsExpInfo()`
- Mobile (≤760px or `hover:none`): detected via `window.matchMedia(...)`, switches to tap-to-reveal
- `mobileProjectsQuery` is module-level — do not redeclare it
- First project auto-activates on mobile via `setActiveProjectItem(projectsExpItems[0])`

### CV tabs
- `toggleTab(id)` collapses all panels then opens the clicked one
- Uses `transitionend` on `max-height` before scrolling to avoid measuring collapsed height
- Button active state is read back from `onclick` attribute string — fragile, don't refactor without updating HTML

## Responsive breakpoints

| Breakpoint | What changes |
|------------|-------------|
| `≤1024px` | Projects info panel narrows; lab grid → flex column; CV item headers stack |
| `≤900px` | CV → single column; left decorative column hidden |
| `≤760px` | Projects switches to touch mode; info panel goes full-width bottom overlay |

## Constraints

- **No external JS dependencies** — vanilla only
- **No build step** — edits go directly to `index.html`
- **Inline everything** — do not create separate `.css` or `.js` files
- The `scroll-snap` + `height: 100vh` contract is load-bearing; changing either requires updating both together
