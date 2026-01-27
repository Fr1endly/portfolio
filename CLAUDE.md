# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Astro-based portfolio website showcasing animated visual effects. The project uses Astro 5 with Tailwind CSS for styling and custom animations. Key features include animated gradients, transparent text effects, and image cutout effects.

**Tech Stack:**
- Astro 5.16+ (static site generator)
- Tailwind CSS 4 (styling)
- TypeScript (strict mode)
- Custom CSS animations and SVG filters

## Development Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start dev server at localhost:4321 with hot reload |
| `npm run build` | Build production-ready site to `./dist/` |
| `npm run preview` | Preview the built site locally before deployment |
| `npm run astro check` | Run TypeScript type checking |
| `npm run astro add [integration]` | Add Astro integrations |

## Architecture & Key Concepts

### Project Structure

```
src/
├── pages/           # Astro pages (each .astro file becomes a route)
│   ├── index.astro         # Home page with scroll overlap sections
│   ├── gradient.astro       # Animated gradient background demo
│   ├── transparent-text.astro # Text knockout effect demo
│   ├── combined-effect.astro # Gradient + text effect component demo
│   └── image-cutout.astro   # Gradient + image effect component demo
├── components/      # Reusable Astro components
│   ├── Layout.astro         # Root layout with SEO meta tags
│   ├── HeroSection.astro    # Animated gradient hero with image cutouts
│   ├── ProjectCard.astro    # Project card for carousel
│   ├── GradientTransparentText.astro   # Combines gradient bg + text cutout
│   ├── GradientTransparentImage.astro  # Combines gradient bg + image cutout
│   └── Container.astro      # Basic container component
├── layouts/         # Layout components (Layout.astro is the main one)
├── styles/          # Global CSS
│   └── global.css   # Font definitions and base animations
└── public/          # Static assets (images, fonts, etc.)
```

### Page Layout & Scroll Behavior

**Index Page Structure (index.astro:13-137)**

The home page uses a unique two-section layout with scroll-based overlap effect:

**Section 1: Sticky Hero** (`index.astro:13-15`)
- Wrapped in `<div class="sticky top-0 h-screen z-10">`
- Contains `<HeroSection />` component
- Uses `position: sticky` to stay fixed at viewport top during scroll
- Full viewport height (`h-screen`)
- Z-index of 10 (lower layer)

**Section 2: Projects Section** (`index.astro:18-137`)
- `<section class="relative ... z-20 mt-[100vh] ...">`
- Uses `position: relative` with z-index of 20 (higher than hero)
- `mt-[100vh]` (margin-top: 100vh) positions it exactly one viewport height down
- Dark stone background (`bg-stone-900`)
- Contains header text and project carousel

**Scroll Overlap Mechanism:**
1. Initially, hero fills viewport and projects section is below (100vh down)
2. As user scrolls down, the hero stays "stuck" at top due to `position: sticky`
3. Projects section (higher z-index: 20) slides up and covers the hero (z-index: 10)
4. Creates smooth "slide over" transition effect

**HeroSection Component Layers** (`HeroSection.astro:5-79`)

Three stacked layers with `isolation: isolate` on root container:

1. **Layer 1: Animated Gradient Background** (lines 10-36)
   - Base gradient: `linear-gradient(40deg, rgb(108, 0, 162), rgb(0, 17, 82))`
   - Three animated blob gradients (`.blob--g1`, `.blob--g2`, `.blob--g3`)
   - Interactive bubble (`.blob--interactive`) that follows mouse cursor
   - SVG goo filter (`#goo-image`) creates blob merging effect

2. **Layer 2: Image Cutout Effect** (lines 39-58)
   - Semi-transparent black overlay (`bg-stone-950/95`) with `mix-blend-multiply`
   - Three floating images: sticky notes, laptop, coffee cup
   - Blend mode reveals gradient through image areas
   - Images use `.floating` and `.floating-slow` animations

3. **Layer 3: Text Content** (lines 61-78)
   - No blend mode - renders normally with white text
   - Name heading and welcome message
   - Pointer events disabled on wrapper, enabled on content div
   - Animated arrow at bottom using `animate-bounce`

**Project Carousel** (`index.astro:50-134`)

- Horizontal scrolling container with drag-to-scroll functionality
- Uses `snap-x snap-mandatory` for snap scrolling
- Mouse and touch event handlers for drag interaction (lines 214-324)
- Momentum scrolling with friction physics (lines 327-347)
- Navigation arrows that auto-hide at start/end positions
- Prevents click events during drag gestures (dragDistance check)

### Key Animation Systems

**SVG Filter (goo effect):**
- Applied to gradient animations in `GradientTransparentText.astro` and `GradientTransparentImage.astro`
- Uses Gaussian blur + color matrix for blob merging effect
- Filter ID: `goo` or `goo-image`

**CSS Animations in global.css:**
- `moveInCircle` - Rotate 360° (used by g2, g3, g5 gradient layers)
- `moveVertical` - Vertical oscillation (used by g1 gradient layer)
- `moveHorizontal` - Horizontal + slight vertical oscillation (used by g4 layer)
- `float` - Simple vertical bounce (3s, used for images)
- `floatSlow` - Slower vertical bounce (4s)

### Component Props Pattern

All effect components accept props via Astro's `interface Props`:
- Components validate props with TypeScript
- Default values are set for all props
- Props are destructured with `Astro.props`

Example:
```astro
interface Props {
  text?: string;
  class?: string;
}

const { text = "DEFAULT", class: className = "" } = Astro.props;
```

## Important Implementation Details

### Sticky Scroll & Z-Index Stacking

The index page uses CSS positioning and z-index for the scroll overlap effect:
- **Hero section**: `position: sticky`, `top: 0`, `z-index: 10`
- **Projects section**: `position: relative`, `z-index: 20`, `margin-top: 100vh`
- Key: Higher z-index (20) allows projects to slide over sticky hero (10)
- `isolation: isolate` on HeroSection prevents blend mode conflicts with parent stacking context

### Gradient Effects

The animated gradient backgrounds use:
1. **Base background** - Linear gradient: `linear-gradient(40deg, rgb(108, 0, 162), rgb(0, 17, 82))`
2. **Three to five gradient layers** (`.blob--g1` through `.blob--g3` in HeroSection) - Radial gradients with different colors and animation timings
3. **Interactive bubble** (`.blob--interactive`) - Follows mouse cursor using `requestAnimationFrame`
4. **Goo filter** - SVG filter (`#goo-image`) for blob-like merging effect

### Text/Image Cutout Effects

Uses `mix-blend-mode: multiply` on a black overlay to create knockout effect:
- Image cutout applies `filter: brightness(1) invert(1)` to invert image colors
- Text remains white on black background
- The blend mode reveals animated gradient through transparent areas

### TypeScript Strictness

Project uses `astro/tsconfigs/strict`. Key patterns for null-safety:
- Use generic type parameters: `document.querySelector<HTMLElement>(".class")`
- Add non-null assertion (`!`) when TypeScript can't infer safety: `element!.style.property`
- Always check null before using in nested functions

### Tailwind Configuration

Custom fonts defined in `tailwind.config.mjs`:
- `font-lato` - Lato family (loaded in global.css)
- `font-raleway` - Raleway family (loaded in global.css)
- Arbitrary width values used: `w-140`, `w-230`, `w-100` (these need to be defined or use inline styles)

## Common Development Tasks

### Adding a New Effect Component

1. Create `.astro` file in `src/components/`
2. Define `Props` interface with optional properties
3. Include `<style>` block with scoped styles
4. Include `<script>` block for interactivity (browser-only)
5. Create demo page in `src/pages/` that imports the component

### Modifying Gradient Animations

- Edit animation durations in `.g1` through `.g5` classes
- Adjust colors in the radial gradient values
- Modify animation timing functions (ease, linear, reverse)
- All animations are in `global.css` or component `<style>` blocks

### Adding Images or Assets

1. Place files in `public/` directory
2. Reference as `/filename` in Astro components
3. Images are used in index.astro and can be reused in other components

### TypeScript Checking

Run `npm run astro check` to catch type errors. Common fixes:
- Add generic to `querySelector`: `document.querySelector<HTMLElement>(...)`
- Add non-null assertion for checked nullability: `element!.property`
- Ensure component Props are properly typed

### Implementing Drag-to-Scroll Carousels

The project carousel in `index.astro` demonstrates advanced drag-to-scroll:
1. **Event Handlers**: Mouse (mousedown/up/move) and touch (touchstart/end/move) events
2. **Velocity Tracking**: Calculate drag speed for momentum scrolling
3. **Momentum Physics**: Apply friction (`0.92`) to velocity over time using `requestAnimationFrame`
4. **Drag State**: `.dragging` class disables scroll-snap and smooth-scroll during drag
5. **Click Prevention**: Track `dragDistance` to prevent clicks when dragging > 5px
6. **Button Visibility**: Navigation arrows show/hide based on scroll position and hover state

Key considerations:
- Use `passive: true` for touch events to improve scroll performance
- Cancel momentum animation on new drag start
- Disable pointer-events on children during drag to prevent interference

## Notes for Future Development

- **Scroll Overlap Pattern**: The sticky + z-index technique can be reused for other section transitions. Always ensure the overlaying section has higher z-index and sufficient margin-top (typically 100vh)
- **Z-Index Management**: Hero (z-10), Projects (z-20). Keep z-index values well-spaced for future insertions
- **Isolation Context**: `isolation: isolate` on components with blend modes prevents them from affecting parent stacking context
- **SVG Filters**: Each component using effects has unique filter IDs (goo, goo-image) to avoid conflicts when multiple components render
- **Mouse Interactivity**: The interactive bubble is scoped to its component with proper selectors (e.g., `.blob--interactive`)
- **Blend Modes**: `mix-blend-mode: hard-light` is used for gradient layers; `multiply` for overlay effects in HeroSection
- **Performance**: `requestAnimationFrame` is used for smooth mouse tracking and momentum scrolling
- **Responsive**: Media queries at 768px breakpoint adjust font sizes and image dimensions
- **Carousel Scrollbar**: Hidden via CSS (`scrollbar-width: none` and `::-webkit-scrollbar`) for cleaner aesthetic
