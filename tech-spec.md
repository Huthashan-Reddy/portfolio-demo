# Technical Specification — Aarav Sharma Student Portfolio

## Dependencies

### Runtime

| Package | Version | Purpose |
|---------|---------|---------|
| react | ^19.1.0 | UI framework |
| react-dom | ^19.1.0 | React DOM renderer |
| three | ^0.175.0 | 3D engine (particle simulation) |
| @react-three/fiber | ^9.1.0 | React renderer for Three.js |
| @react-three/drei | ^10.0.0 | R3F helpers (OrbitControls, Environment, RoundedBox, ContactShadows, etc.) |
| @react-three/cannon | ^6.6.0 | Physics engine (Cannon.js bindings) |
| gsap | ^3.13.0 | Animation engine (ScrollTrigger, timelines, quickTo) |
| lenis | ^1.3.0 | Smooth scrolling |
| lucide-react | ^0.510.0 | Icons (Mail, MapPin, Linkedin, Instagram, Youtube, Calculator, Sigma, Code, Mic, Flask) |

### Dev

| Package | Version | Purpose |
|---------|---------|---------|
| typescript | ^5.8.0 | Type safety |
| vite | ^6.3.0 | Build tool |
| @vitejs/plugin-react | ^4.5.0 | React Fast Refresh for Vite |
| tailwindcss | ^4.1.0 | Utility CSS |
| @tailwindcss/vite | ^4.1.0 | Tailwind Vite integration |
| @types/react | ^19.1.0 | React type definitions |
| @types/react-dom | ^19.1.0 | ReactDOM type definitions |
| @types/three | ^0.175.0 | Three.js type definitions |

### GSAP Plugins (included in gsap package)

- **ScrollTrigger** — Scroll-triggered animations, scrub-linked timelines
- **SplitText** — Character-level text splitting for flip reveal (premium, see Notes)
- **Draggable** — Drag interaction for gallery (premium, see Notes)
- **InertiaPlugin** — Momentum scrolling for gallery (premium, see Notes)

> **Note on premium GSAP plugins**: SplitText, Draggable, and InertiaPlugin require GSAP Club membership. Implement fallbacks using open-source alternatives: custom character splitting for SplitText, and a manual velocity-based drag system for Draggable + InertiaPlugin.

---

## Component Inventory

### Layout

| Component | Source | Notes |
|-----------|--------|-------|
| Header | Custom | Fixed, frosted glass, scroll-aware transparency |
| Footer | Custom | Simple centered footer |
| SmoothScrollProvider | Custom (Lenis) | Wraps entire app, initializes Lenis instance |

### Sections

| Component | Key Features |
|-----------|-------------|
| HeroSection | Split-screen layout, animated name, stats counter, particle canvas |
| AboutSection | Two-column (photo + details grid), interest pills |
| AcademicSection | Scroll flip reveal title, grade table |
| ClubsSection | Scroll flip reveal title, 2-column club cards |
| CompetitionsSection | Scroll flip reveal title, vertical timeline |
| ProjectsSection | Dark inset card, scroll flip reveal, 3D flip card grid |
| SkillsSection | Scroll flip reveal, 3-column skill cards with progress bars |
| CertificatesSection | Scroll flip reveal, smooth scroll gallery |
| BlogSection | Scroll flip reveal, 3-column blog cards |
| TestimonialsSection | Scroll flip reveal, 2-column testimonial cards |
| ContactSection | Two-column (contact info + form) |

### Reusable Components

| Component | Source | Used By |
|-----------|--------|---------|
| ParticleCanvas | Custom (R3F) | HeroSection |
| ScrollFlipReveal | Custom (GSAP) | AcademicSection, ClubsSection, CompetitionsSection, ProjectsSection, SkillsSection, CertificatesSection, BlogSection, TestimonialsSection |
| FlipCard | Custom (CSS 3D) | ProjectsSection |
| SmoothScrollGallery | Custom (GSAP) | CertificatesSection |
| SectionOverline | Custom | All sections ("ABOUT ME", "ACADEMICS", etc.) |
| SectionHeading | Custom | All sections (wraps ScrollFlipReveal) |
| StatCounter | Custom (GSAP) | HeroSection |
| PillBadge | Custom | AboutSection, CompetitionsSection, SkillsSection |

---

## Animation Implementation

| Animation | Library | Approach | Complexity |
|-----------|---------|----------|------------|
| Hero name entrance | GSAP | translateY(100%)→0 with overflow:hidden clip, staggered lines | Medium |
| Quick stats counter | GSAP | gsap.to with snap for counting effect, triggered after hero animation | Low |
| Tilted Bounce particles | R3F + @react-three/cannon | 55 instanced spheres with physics bodies bouncing off tilted planes, auto-rotating camera | 🔒 High |
| Scroll Flip Reveal | GSAP + SplitText | SplitText splits into chars, each char animates from rotateY(-80°) + opacity 0.2 to rotateY(0) + opacity 1, scrub-linked to scroll | 🔒 High |
| CSS 3D Flip Cards | Pure CSS | perspective + preserve-3d container, rotateY(180deg) on hover, backface-visibility hidden | Medium |
| Smooth Scroll Gallery | GSAP + Draggable | Drag-based horizontal scroll, quickTo setters interpolate items from random scattered positions to aligned row based on scroll progress | 🔒 High |
| Section entrance (global) | GSAP + ScrollTrigger | translateY(30px)→0, opacity 0→1, triggered at 85% viewport, 0.1s stagger | Low |
| Header scroll transition | CSS transition | backdrop-filter and border toggle based on scroll position (past 100vh) | Low |
| Club card hover | CSS transition | translateY(-4px), border-color change | Low |
| Testimonial quotation mark | Static | Decorative oversized quote character in accent color | Low |

---

## State & Logic

### Lenis ↔ GSAP ScrollTrigger Bridge

Lenis must drive GSAP's ScrollTrigger for all scroll-linked animations to work together. This requires:

1. Lenis instance stored in a shared ref/context
2. `lenis.on('scroll', ScrollTrigger.update)` to sync
3. GSAP ticker callback: `gsap.ticker.add((time) => lenis.raf(time * 1000))` with `gsap.ticker.lagSmoothing(0)`

### R3F ↔ DOM Bridge

The particle canvas runs in a separate React reconciler. No context sharing is possible. The hero section passes layout dimensions (container width/height) to the Canvas via props for responsive sizing.

### Gallery State (SmoothScrollGallery)

Internal to the gallery component:
- `minX` / `maxX` drag bounds calculated from total content width minus viewport width
- `targetState.x` — current scroll position (driven by Draggable)
- `quickSetters` — cached gsap.quickTo functions for each item's x, y, scale, rotateZ
- `progress` — normalized scroll progress (0 to 1) that drives the scatter→align interpolation

---

## Other Key Decisions

### Physics Engine: @react-three/cannon over @react-three/rapier

The spec specifies cannon-es physics. @react-three/cannon is the React wrapper for cannon-es. It provides `useSphere`, `usePlane`, `useBox` hooks that match the design's physics scene structure exactly. Rapier would require restructuring the physics setup.

### CSS 3D over R3F for Flip Cards

The project flip cards use CSS 3D transforms (perspective + rotateY) rather than a second R3F canvas. This avoids the overhead of a second WebGL context and keeps the cards lightweight and accessible.

### Premium GSAP Plugin Fallbacks

Since SplitText, Draggable, and InertiaPlugin are premium:
- **SplitText fallback**: Custom utility that splits text into individual `<span>` elements with `display: inline-block`, wraps each in an overflow-hidden container, then applies the same GSAP animation
- **Draggable + Inertia fallback**: Manual `pointerdown`/`pointermove`/`pointerup` handlers with `requestAnimationFrame` loop and velocity-based deceleration

### Mobile Strategy

- Hero: Stack text above canvas, reduce canvas to ~50vh
- Flip cards: Use IntersectionObserver to auto-flip when 50% visible, instead of hover
- Gallery: Remains draggable on touch
- Particles: Reduce count from 55 to 35 for performance
