---
name: taste
description: Apply refined aesthetic, anti-slop guidelines, and UI/UX taste to prevent generic "AI-generated" designs.
globs: "**/*.{tsx,jsx,vue,svelte,html,css,scss,tailwind.config.*}"
---

# Taste Skill Directive for Antigravity (Anti-Slop UI)

When creating, refactoring, or editing front-end interfaces, UI components, or styles, enforce these core aesthetic principles to eliminate generic "AI-generated" patterns and deliver high-end, human-crafted UI/UX quality.

---

## 1. Typography & Hierarchy
- **Typography Selection**: Avoid default system stacks or generic fonts like plain Arial/un-styled Inter. In KalySync, **strictly use the centralized typography tokens**: `var(--font-heading)` (*Playfair Display*) for titles, `var(--font-body)` (*Roboto*) for text/forms, and `var(--font-number)` (*Montserrat*) for numbers/counters.
- **Font Size Scale**: Never hardcode arbitrary pixel/rem font sizes. Always rely on `var(--font-size-title-page)`, `var(--font-size-title-modal)`, `var(--font-size-base)`, `var(--font-size-md)`, `var(--font-size-sm)`, `var(--font-size-xs)`, `var(--font-size-badge)`.
- **Display vs Body**: Use high-contrast font weights and tight negative letter-spacing (`tracking-tight`) for large display headings. Apply subtle expanded tracking (`tracking-wider` + uppercase) for labels, badges, or kicker text.
- **Rhythm & Formatting**: Maintain strict vertical rhythm and clear line-height scaling. Avoid overuse of em-dashes (`—`) in UI copy.

## 2. Color, Depth & Atmosphere
- **Monotone & Accents**: Avoid pure black (`#000000`) or pure white (`#ffffff`). Prefer rich, nuanced neutral backgrounds (warm slate, deep zinc, rich off-white) paired with a single intentional accent color.
- **No Boring Gradients**: Avoid standard two-stop linear gradients (e.g., standard blue-to-purple). Use high-chroma subtle glows or background blurs (`backdrop-blur-md`).
- **Elevation**: Build depth using layered backdrop surfaces, subtle translucent borders (`border-white/10` or `border-black/5`), and soft multi-layered drop shadows instead of harsh, heavy outlines.

## 3. Layout & Micro-Spacing
- **Generous Spacing**: Prioritize breathing room over compact, crowded boxes. Use generous padding (`p-6` to `p-12`) for card containers and sections.
- **Asymmetry & Intent**: Avoid flat, static, or perfectly symmetric grid blocks. Incorporate subtle asymmetric accents, offset status indicators, or dynamic flex/grid arrangements.
- **Nested Radii Math**: Maintain mathematical corner alignment for nested elements (outer radius = inner radius + padding).

## 4. Polishing, Motion & Micro-Interactions
- **State Feedback**: Always design explicit, fluid micro-interactions for `:hover`, `:active`, and `:focus-visible` states (e.g., `active:scale-[0.98]`, smooth opacity transitions).
- **Intentional Easing**: Ensure motion feels tactile, fluid, and subtle rather than jarring or decorative for the sake of it.

## 5. Output Integrity
- **Full Deliverables**: Never truncate generated UI components or use placeholder comments (e.g., `// ... rest of component`). Always provide complete, production-ready code.