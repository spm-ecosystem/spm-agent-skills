---
name: component-developer
description: Component development guidelines and strict rules for building robust, configurable React components for the SPM ecosystem (spm-components). Focuses on Shadow DOM encapsulation, custom theme design, data-driven prop contracts, responsive layouts, fallback handling, and performance optimization.
---

# SPM Component Developer Guidelines

ROLE: Senior Frontend & Component Architect  
CONTEXT: Site Package Manager Component Library (`spm-components`)  
GOAL: Guide developers in building fully configurable, responsive, and robust React components that can be dynamically mounted and themed via Veneer Spec (`.vnr`) declarations inside a Shadow DOM web component context.

---

## Core Development Principles

Every component in the `spm-components` library must adhere to these six pillars of development:

1. **Extreme Configurability** (Zero Hardcoded Content/Assumptions)
2. **Shadow DOM Compatibility** (Host Styling & CSS Variables)
3. **Data-Driven Prop Contracts** (Uniform Key Naming & Type Safety)
4. **Responsive Sizing & Sane Typographics** (Viewport Agnosticism)
5. **Graceful Fallbacks & Robust Error Handling** (Fault Tolerance)
6. **Optimized Slicing & Performance** (Lazy Loading & Pagination)

---

## 1. Extreme Configurability

Veneer-reconstructed elements are built entirely from dynamic page-scraped JSON properties. Hardcoding layout assumptions, text labels, or margins within components restricts their reusability across different websites.

- **Labels & Strings:** Every text label, prefix, search placeholder, or fallback text must be a prop.
- **Dangling Prefixes:** Do not render prefix labels (e.g. `Posted by: ` or `Tags: `) if the corresponding prop value is empty or null. Always wrap the label and the value in a single conditional statement:
  ```tsx
  // BAD: Renders "Posted by: " with nothing if author is null
  <span>Posted by: {author}</span>

  // GOOD: Conditionally renders the entire block
  {author && <span>Posted by: {author}</span>}
  ```
- **Layout Variations:** If a layout has alternate styles (e.g., sidebar on the left vs. right, grid column count, sticky navigation), expose these settings as props (e.g. `sidebarSide: "left" | "right"`, `sticky: boolean`).

---

## 2. Shadow DOM Compatibility & Sizing

SPM injects React components inside a custom element Shadow Root (`[class^="modern-reconstruct-host-"]`) to isolate styling from the target page. This adds specific layout and styling requirements:

### Host Layout Rules
- **Host Element Sizing:** By default, custom elements are rendered with `display: inline` by browsers. Always specify the default layout of the host using `:host` selectors inside the component stylesheet:
  ```css
  :host {
    display: block;
    width: 100%;
    box-sizing: border-box;
  }
  ```
- **Sticky Host Positioning:** Standard `position: sticky` inside the Shadow Root will fail to stick to the viewport if the host custom element itself is not sticky. Expose a `sticky` prop and apply sticky styling directly to the host container using attribute selectors:
  ```css
  :host([sticky="true"]) {
    position: sticky;
    top: 0;
    z-index: 1000;
  }
  ```

### Centralized Styling via CSS Variables
Never hardcode colors, border-radii, gaps, or fonts. Use SPM CSS variable tokens with neutral fallback values to ensure they map cleanly to the theme configuration:

| Token | Purpose | Example |
| :--- | :--- | :--- |
| `var(--spm-bg-primary)` | Base background color | `#ffffff` / `#000000` |
| `var(--spm-bg-surface)` | Card/section background | `#f4f4f5` / `#18181b` |
| `var(--spm-bg-element)` | Button/input background | `#e4e4e7` / `#27272a` |
| `var(--spm-text-primary)`| Primary text color | `#18181b` / `#f4f4f5` |
| `var(--spm-text-muted)` | Subtext/meta color | `#71717a` / `#a1a1aa` |
| `var(--spm-accent)` | Primary color for links/active states | `#3b82f6` (blue) |
| `var(--spm-border-contrast)`| Border color for cards/tables | `rgba(0,0,0,0.08)` |
| `var(--spm-radius)` | Standard border radius | `8px` |

---

## 3. Data-Driven Prop Contracts

Keep prop names uniform across the library to prevent integration and compiler errors.

- **Uniform Link Properties:** Use `url` instead of mixing `href` and `url`. Every link configuration should follow this contract:
  ```typescript
  interface LinkItem {
    label: string;
    url: string;
  }
  ```
- **Array Checks:** Never assume arrays are populated. Always perform `Array.isArray()` and length checks before rendering list headers, grids, or divider lines. If a tag list is empty, the entire container should unmount to prevent rendering empty boxes or raw margins.

---

## 4. Responsive Sizing & Sane Typographics

Components will be mounted on different viewports and column sizes. They must adapt dynamically.

- **Flex Wrapping:** Never use fixed widths for internal layout elements. Always use `flex-wrap: wrap` or CSS Grid with `auto-fit`/`auto-fill` templates.
- **Squeezing Guard:** On viewports under `400px`, layout columns should stack vertically. For example, search metrics, comment columns, and split sidebars must drop to a single-column layout on mobile.
- **Word Wrapping:** Prevent text overflow caused by long strings (such as unspaced tags, URLs, or usernames) by applying breaking rules to all text containers:
  ```css
  .text-container {
    word-break: break-word;
    overflow-wrap: break-word;
  }
  ```
- **Ellipsis:** For single-line text overflows (such as card titles), use ellipsis:
  ```css
  .title {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  ```

---

## 5. Graceful Fallbacks & Robust Error Handling

Modernized targets may contain missing or corrupt data. The component must degrade gracefully.

- **Broken Image Fallback:** Image components (and cards containing images like `UiImageCard`) must handle image load failures. Provide an inline placeholder SVG or default icon using the `onError` event handler:
  ```tsx
  const [imgSrc, setImgSrc] = useState(imageUrl);
  
  return (
    <img 
      src={imgSrc} 
      onError={() => setImgSrc(PLACEHOLDER_SVG_URL)}
      alt={title}
    />
  );
  ```
- **Metadata Fallbacks:** Provide sensible default text if metadata strings are empty (e.g. `title || 'Untitled'`, `author || 'Anonymous'`).
- **Null Safety:** Deeply nested structures must use optional chaining (`?.`) to prevent runtime crashes when props are partially populated.

---

## 6. Optimized Slicing & Performance

When rendering galleries, lists, or tables:

- **Lazy Loading Images:** By default, all images inside list components must use native browser lazy loading to save bandwidth and improve page load metrics:
  ```tsx
  <img src={imageUrl} loading="lazy" ... />
  ```
- **Overflow Navigation:** If an array (like navigation links) contains a high number of items (e.g., 10+ links), the component must handle layout overflow. Add an `overflow-x: auto` style with thin scrollbar support or implement a `maxVisibleLinks` option that collapses remaining items into a "More" dropdown.
- **Infinite Scrolling Sentinels:** Table lists and metrics components should provide hooks for loading dynamic pagination. Place an `IntersectionObserver` trigger/sentinel block at the bottom of the list that dispatches a custom window/DOM event (e.g. `spm-load-next-page`) when visible, allowing external runners to hydrate more content.
