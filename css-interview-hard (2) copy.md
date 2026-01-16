/* BAD: Paint properties (causes repaint) */
/* color, background, border, box-shadow */

/* PERFORMANT ANIMATIONS */

/* ✓ GOOD: Transform for movement */
.slide {
  animation: slideGood 0.3s ease;
}

@keyframes slideGood {
  from { transform: translateX(-100px); }
  to { transform: translateX(0); }
}

/* ✗ BAD: Left for movement */
.slide-bad {
  animation: slideBad 0.3s ease;
}

@keyframes slideBad {
  from { left: -100px; } /* Causes reflow */
  to { left: 0; }
}

/* ✓ GOOD: Transform for scaling */
.grow {
  transform: scale(1);
  transition: transform 0.3s ease;
}

.grow:hover {
  transform: scale(1.2);
}

/* ✗ BAD: Width/Height for scaling */
.grow-bad {
  width: 100px;
  height: 100px;
  transition: width 0.3s, height 0.3s;
}

.grow-bad:hover {
  width: 120px;
  height: 120px;
}

/* OPTIMIZATION TECHNIQUES */

/* 1. Use will-change (sparingly) */
.button {
  will-change: transform;
  /* Tells browser to create composite layer */
}

.button:hover {
  transform: scale(1.1);
}

/* Remove will-change after use */
.button {
  transition: transform 0.3s;
}

.button:hover {
  will-change: transform;
  transform: scale(1.1);
}

.button:not(:hover) {
  will-change: auto;
}

/* 2. Use transform: translate3d() for hardware acceleration */
.element {
  transform: translate3d(0, 0, 0);
  /* Forces GPU acceleration */
}

/* 3. Contain layout changes */
.card {
  contain: layout; /* Isolate layout calculations */
  /* or */
  contain: paint; /* Isolate painting */
  /* or */
  contain: strict; /* All optimizations */
}

/* 4. Use CSS containment */
.container {
  content-visibility: auto; /* Lazy render */
  contain-intrinsic-size: 500px; /* Estimated size */
}

/* 5. Batch DOM changes */
/* BAD: Multiple reflows */
element.style.width = '100px';  /* Reflow */
element.style.height = '100px'; /* Reflow */
element.style.margin = '10px';  /* Reflow */

/* GOOD: Single reflow with CSS class */
.updated {
  width: 100px;
  height: 100px;
  margin: 10px;
}
element.classList.add('updated'); /* Single reflow */

/* MEASURING PERFORMANCE */

/* 1. Browser DevTools Performance tab */
/* Record during animation, look for: */
/* - Long frames (>16ms = dropped frames) */
/* - Layout/Reflow events */
/* - Paint events */
/* - Composite layers */

/* 2. Chrome DevTools Layers panel */
/* See what's being composited */

/* 3. Rendering > Paint flashing */
/* See what's being repainted */

/* PRACTICAL EXAMPLES */

/* Loading spinner (performant) */
.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Smooth modal entrance */
.modal {
  opacity: 0;
  transform: scale(0.9) translateY(-20px);
  transition: 
    opacity 0.3s ease,
    transform 0.3s ease;
}

.modal.is-visible {
  opacity: 1;
  transform: scale(1) translateY(0);
}

/* Skeleton loading (performant) */
.skeleton {
  background: linear-gradient(
    90deg,
    #f0f0f0 25%,
    #e0e0e0 50%,
    #f0f0f0 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

/* Parallax effect (use transform) */
.parallax {
  transform: translateZ(0); /* Force GPU */
}

.parallax-bg {
  transform: translateY(calc(var(--scroll) * -0.5px));
}

/* DEBUGGING PERFORMANCE ISSUES */

/* Mark composite layers with outline */
* {
  outline: 1px solid rgba(255, 0, 0, 0.2);
}

/* Check repaints */
* {
  background: rgba(255, 0, 0, 0.1);
}

/* Slow animations for debugging */
* {
  animation-duration: 10s !important;
  transition-duration: 10s !important;
}

/* 60 FPS = 16.67ms per frame */
/* Keep animations under 16ms for smooth 60fps */

/* WHAT NOT TO ANIMATE */
/* ✗ Never animate: */
/* - width, height (use transform: scale) */
/* - top, left, right, bottom (use transform: translate) */
/* - margin, padding */
/* - border-width */
/* - font-size (use transform: scale if needed) */

/* ✓ Safe to animate: */
/* - transform (translate, rotate, scale) */
/* - opacity */
/* - filter (with caution) */
/* - clip-path (with caution) */
```

---

## <a id="performance"></a>⚡ Performance Optimization

### Q13: How do you optimize CSS for performance? What are critical rendering path optimizations?

**Expected Answer:**
```css
/* CRITICAL RENDERING PATH OPTIMIZATION */

/* 1. Minimize CSS file size */
/* - Remove unused CSS */
/* - Minify in production */
/* - Use cssnano, PurgeCSS */

/* 2. Inline critical CSS */
/* Inline above-the-fold CSS in <head> */
<style>
  /* Critical styles for initial viewport */
  .header { height: 60px; background: white; }
  .hero { min-height: 400px; }
</style>

/* Load rest asynchronously */
<link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">

/* 3. Use CSS containment */
.widget {
  contain: layout; /* Isolate layout calculations */
  contain: paint;  /* Isolate paint */
  contain: size;   /* Fixed size, skip sizing */
}

/* 4. Lazy load non-critical CSS */
<link rel="stylesheet" href="non-critical.css" media="print" onload="this.media='all'">

/* 5. Use content-visibility */
.section {
  content-visibility: auto;
  contain-intrinsic-size: 500px;
  /* Browser skips rendering until needed */
}

/* SELECTOR PERFORMANCE */

/* ✓ FAST selectors */
.class { } /* Fast */
#id { }    /* Fast */

/* ✗ SLOW selectors */
* { } /* Slowest - universal */
[attribute="value"] { } /* Slower */
:nth-child(3n+1) { } /* Complex calculations */

/* Avoid deep nesting */
/* BAD */
.container .header .nav .menu .item .link { }

/* GOOD */
.nav-link { }

/* LAYOUT OPTIMIZATION */

/* Avoid layout thrashing (forced reflows) */
/* BAD */
const height = element.offsetHeight; /* Read */
element.style.height = height + 10 + 'px'; /* Write */
const width = element2.offsetWidth; /* Read - forces reflow! */
element2.style.width = width + 20 + 'px'; /* Write */

/* GOOD - Batch reads and writes */
const height = element.offsetHeight; /* Read */
const width = element2.offsetWidth;  /* Read */
element.style.height = height + 10 + 'px'; /* Write */
element2.style.width = width + 20 + 'px';  /* Write */

/* Use transform instead of layout properties */
/* BAD */
.element {
  top: 100px; /* Causes layout */
  left: 50px;
}

/* GOOD */
.element {
  transform: translate(50px, 100px); /* Composite only */
}

/* FONT LOADING OPTIMIZATION */

/* 1. font-display */
@font-face {
  font-family: 'CustomFont';
  src: url('font.woff2') format('woff2');
  font-display: swap; /* Show fallback immediately */
  /* optional - show fallback for 100ms, then custom font */
  /* block - hide text for 3s waiting for font */
  /* fallback - show fallback for 100ms, then custom for 3s */
}

/* 2. Preload critical fonts */
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>

/* 3. Subset fonts */
/* Only include characters you need */
@font-face {
  font-family: 'CustomFont';
  src: url('font-latin.woff2') format('woff2');
  unicode-range: U+0000-00FF; /* Latin only */
}

/* IMAGE OPTIMIZATION IN CSS */

/* 1. Use image-set for responsive images */
.hero {
  background-image: image-set(
    url('image-1x.jpg') 1x,
    url('image-2x.jpg') 2x,
    url('image-3x.jpg') 3x
  );
}

/* 2. Lazy load background images */
.lazy-bg {
  background-image: none;
}

.lazy-bg.loaded {
  background-image: url('large-image.jpg');
}

/* 3. Use modern formats */
.hero {
  background-image: url('image.webp');
  /* Fallback */
  background-image: image-set(
    url('image.webp') type('image/webp'),
    url('image.jpg') type('image/jpeg')
  );
}

/* CSS FILE ORGANIZATION FOR PERFORMANCE */

/* 1. Split by media queries */
<link rel="stylesheet" href="core.css">
<link rel="stylesheet" href="large.css" media="(min-width: 1024px)">
<link rel="stylesheet" href="print.css" media="print">

/* 2. Code splitting */
/* Load only what's needed per page */
/* home.css, about.css, contact.css */

/* 3. Use HTTP/2 for parallel loading */
/* Multiple CSS files load in parallel */

/* REDUCE CSS SPECIFICITY */

/* BAD - High specificity is slow to calculate */
#header .nav ul li a.active { }

/* GOOD - Low specificity */
.nav-link.is-active { }

/* USE CSS CUSTOM PROPERTIES EFFICIENTLY */

/* BAD - Redefining everywhere */
.button { background: #3498db; }
.link { color: #3498db; }
.border { border-color: #3498db; }

/* GOOD - Single source of truth */
:root {
  --primary: #3498db;
}

.button { background: var(--primary); }
.link { color: var(--primary); }
.border { border-color: var(--primary); }

/* MINIMIZE REPAINTS AND REFLOWS */

/* Properties that cause reflow (expensive) */
/* width, height, margin, padding, border, top, left, etc. */

/* Properties that cause repaint (less expensive) */
/* color, background, visibility, outline, box-shadow */

/* Properties that don't cause reflow or repaint (cheap) */
/* transform, opacity (composited) */

/* MEASURING CSS PERFORMANCE */

/* 1. Chrome DevTools Coverage */
/* Shows unused CSS */

/* 2. Lighthouse CSS audit */
/* Identifies opportunities */

/* 3. WebPageTest CSS analysis */
/* Detailed CSS metrics */

/* 4. Performance API */
performance.mark('css-start');
// Load CSS
performance.mark('css-end');
performance.measure('css-load', 'css-start', 'css-end');

/* CSS SIZE BUDGETS */

/* Recommended limits */
/* - Critical CSS: < 14KB (first TCP packet) */
/* - Total CSS: < 100KB minified */
/* - Selectors: < 5000 per stylesheet */

/* TOOLS FOR OPTIMIZATION */

/* 1. PurgeCSS - Remove unused CSS */
/* 2. cssnano - Minify CSS */
/* 3. Critical - Extract critical CSS */
/* 4. postcss - Optimize CSS */
/* 5. webpack css-loader - Bundle optimization */
```

---

## <a id="modern"></a>🆕 Modern CSS Features

### Q14: Explain CSS Custom Properties (CSS Variables). What are their advantages over Sass variables?

**Expected Answer:**
```css
/* CSS CUSTOM PROPERTIES */

/* Declaration */
:root {
  --primary-color: #3498db;
  --secondary-color: #2ecc71;
  --font-size-base: 16px;
  --spacing-unit: 8px;
}

/* Usage */
.button {
  background: var(--primary-color);
  font-size: var(--font-size-base);
  padding: calc(var(--spacing-unit) * 2);
}

/* Fallback values */
.element {
  color: var(--undefined-color, #000);
  /* If --undefined-color doesn't exist, use #000 */
}

/* Nested fallbacks */
.element {
  color: var(--color-1, var(--color-2, var(--color-3, #000)));
}

/* SCOPE */

/* Global scope */
:root {
  --global-var: value;
}

/* Component scope */
.component {
  --local-var: value;
}

.component .child {
  color: var(--local-var); /* Accessible */
}

/* Dynamic scoping */
.theme-dark {
  --bg-color: #1a1a1a;
  --text-color: #ffffff;
}

.theme-light {
  --bg-color: #ffffff;
  --text-color: #000000;
}

.content {
  background: var(--bg-color);
  color: var(--text-color);
}

/* JAVASCRIPT MANIPULATION */

/* Read variable */
const root = document.documentElement;
const primaryColor = getComputedStyle(root)
  .getPropertyValue('--primary-color');

/* Set variable */
root.style.setProperty('--primary-color', '#e74c3c');

/* Remove variable */
root.style.removeProperty('--primary-color');

/* ADVANTAGES OVER SASS */

/* 1. Runtime manipulation */
/* CSS Variables: Can change at runtime with JS */
document.documentElement.style.setProperty('--theme', 'dark');

/* SASS: Compiled, can't change after build */
$theme: dark; // Fixed at compile time

/* 2. Cascade and inheritance */
/* CSS Variables: Follow CSS cascade */
.parent {
  --color: blue;
}

.child {
  color: var(--color); /* Inherits blue */
}

/* SASS: No inheritance */

/* 3. Media query responsiveness */
/* CSS Variables: Can change with media queries */
:root {
  --columns: 1;
}

@media (min-width: 768px) {
  :root {
    --columns: 3;
  }
}

.grid {
  grid-template-columns: repeat(var(--columns), 1fr);
}

/* SASS: Requires duplicate code */

/* 4. DOM context awareness */
/* CSS Variables: Aware of DOM structure */
.card {
  --accent: blue;
}

.card.featured {
  --accent: red; /* Override for featured cards */
}

.card-title {
  color: var(--accent); /* Uses correct value based on parent */
}

/* PRACTICAL EXAMPLES */

/* 1. Theming system */
:root {
  --theme-bg: #ffffff;
  --theme-text: #000000;
  --theme-primary: #3498db;
}

[data-theme="dark"] {
  --theme-bg: #1a1a1a;
  --theme-text: #ffffff;
  --theme-primary: #2980b9;
}

body {
  background: var(--theme-bg);
  color: var(--theme-text);
}

.button {
  background: var(--theme-primary);
}

/* 2. Responsive spacing scale */
:root {
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 2rem;
  --space-xl: 4rem;
}

@media (min-width: 768px) {
  :root {
    --space-md: 1.5rem;
    --space-lg: 3rem;
    --space-xl: 6rem;
  }
}

.section {
  padding: var(--space-lg);
}

/* 3. Component variations */
.button {
  --button-bg: var(--primary);
  --button-color: white;
  --button-padding: 0.5rem 1rem;
  
  background: var(--button-bg);
  color: var(--button-color);
  padding: var(--button-padding);
}

.button--large {
  --button-padding: 1rem 2rem;
}

.button--danger {
  --button-bg: var(--danger);
}

/* 4. Animation with variables */
@keyframes slide {
  from {
    transform: translateX(var(--slide-from));
  }
  to {
    transform: translateX(var(--slide-to));
  }
}

.slide-left {
  --slide-from: -100%;
  --slide-to: 0%;
  animation: slide 0.3s ease;
}

.slide-right {
  --slide-from: 100%;
  --slide-to: 0%;
  animation: slide 0.3s ease;
}

/* 5. Calculating derived values *# 🎨 Hard CSS Developer Interview Questions

## 📚 Table of Contents
1. [CSS Fundamentals & Specificity](#fundamentals)
2. [Layout Systems (Flexbox & Grid)](#layouts)
3. [Responsive Design & Media Queries](#responsive)
4. [CSS Architecture & Methodology](#architecture)
5. [Animations & Transitions](#animations)
6. [Performance Optimization](#performance)
7. [Modern CSS Features](#modern)
8. [CSS-in-JS & Preprocessors](#preprocessors)
9. [Cross-Browser Compatibility](#compatibility)
10. [Real-World Scenarios](#scenarios)

---

## <a id="fundamentals"></a>⚡ CSS Fundamentals & Specificity

### Q1: Explain CSS specificity in detail. How is it calculated?

**Expected Answer:**
```css
/* Specificity is calculated as (inline, IDs, classes/attributes/pseudo-classes, elements/pseudo-elements) */

/* (0, 0, 0, 1) */
p { color: red; }

/* (0, 0, 1, 1) */
p.text { color: blue; }

/* (0, 1, 0, 1) */
#header p { color: green; }

/* (1, 0, 0, 0) */
<p style="color: purple;">Highest specificity</p>

/* Examples with calculations */
/* (0, 0, 1, 2) = 0012 */
div p.warning { color: orange; }

/* (0, 1, 2, 1) = 0121 */
#nav ul.menu li { color: black; }

/* (0, 2, 1, 0) = 0210 */
#header #nav .active { color: white; }

/* Special cases */
!important /* Overrides everything except another !important with higher specificity */
:where() /* Has 0 specificity */
:is(), :not(), :has() /* Takes specificity of most specific selector inside */

/* Specificity order (lowest to highest) */
* /* Universal selector: (0,0,0,0) */
elements /* (0,0,0,1) */
classes, attributes, pseudo-classes /* (0,0,1,0) */
IDs /* (0,1,0,0) */
inline styles /* (1,0,0,0) */
!important /* Overrides all */

/* Real-world example */
/* Which color wins? */
.container #header .nav li a { } /* (0,1,2,2) = 0122 */
#header ul li.active a { }      /* (0,1,1,3) = 0113 */
/* First one wins: 0122 > 0113 */

/* Best practices */
/* Avoid !important */
/* Keep specificity low */
/* Use classes over IDs for styling */
/* Avoid inline styles */
```

### Q2: What is the CSS Box Model? Explain `box-sizing` property.

**Expected Answer:**
```css
/* Standard Box Model (content-box) */
/* Width = content width only */
.box {
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  margin: 10px;
}
/* Total width = 200 + 40 + 10 = 250px (without margin) */
/* Total width with margin = 270px */

/* Border-box Model */
/* Width = content + padding + border */
.box-border {
  box-sizing: border-box;
  width: 200px; /* This includes padding and border */
  padding: 20px;
  border: 5px solid black;
  margin: 10px;
}
/* Content width = 200 - 40 - 10 = 150px */
/* Total width = 200px (without margin) */

/* Best practice: Apply to all elements */
*, *::before, *::after {
  box-sizing: border-box;
}

/* Visual representation */
/*
┌─────────────────────────────────────┐
│           Margin (transparent)       │
│  ┌───────────────────────────────┐  │
│  │        Border                 │  │
│  │  ┌─────────────────────────┐ │  │
│  │  │      Padding            │ │  │
│  │  │  ┌───────────────────┐ │ │  │
│  │  │  │    Content        │ │ │  │
│  │  │  │                   │ │ │  │
│  │  │  └───────────────────┘ │ │  │
│  │  └─────────────────────────┘ │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
*/

/* Margin collapse */
/* Vertical margins collapse to the larger of the two */
.box1 { margin-bottom: 20px; }
.box2 { margin-top: 30px; }
/* Gap between boxes = 30px (not 50px) */

/* Prevent margin collapse */
.parent {
  overflow: hidden; /* Creates BFC */
  /* or */
  display: flow-root; /* Modern solution */
  /* or */
  padding: 1px; /* Add padding/border */
}
```

### Q3: Explain the CSS Cascade and inheritance.

**Expected Answer:**
```css
/* CASCADE ORDER (lowest to highest priority) */
/* 1. User agent styles (browser defaults) */
/* 2. User styles (user's custom CSS) */
/* 3. Author styles (your CSS) */
/* 4. Author !important */
/* 5. User !important */
/* 6. User agent !important */

/* When specificity is equal, last rule wins */
p { color: red; }
p { color: blue; } /* This wins - declared last */

/* INHERITANCE */
/* Inherited properties */
body {
  color: #333;        /* Inherited by children */
  font-family: Arial; /* Inherited */
  font-size: 16px;    /* Inherited */
  line-height: 1.5;   /* Inherited */
}

/* Non-inherited properties */
div {
  border: 1px solid black; /* NOT inherited */
  padding: 10px;           /* NOT inherited */
  margin: 10px;            /* NOT inherited */
  background: white;       /* NOT inherited */
}

/* Force inheritance */
.child {
  border: inherit;  /* Force inherit from parent */
  padding: inherit;
}

/* Initial value */
.reset {
  color: initial; /* Reset to browser default */
}

/* Unset (inherit if inheritable, otherwise initial) */
.unset {
  color: unset;
  border: unset;
}

/* Revert (revert to user agent stylesheet) */
.revert {
  all: revert; /* Reset all properties */
}

/* Commonly inherited properties */
/* - color, font-*, text-*, line-height, letter-spacing, word-spacing */
/* - visibility, cursor, list-style-* */

/* Never inherited properties */
/* - margin, padding, border, width, height, display */
/* - position, top, right, bottom, left, z-index */
/* - background-*, overflow, float */
```

### Q4: What is Block Formatting Context (BFC)? How do you create one?

**Expected Answer:**
```css
/* BFC is an isolated rendering region where elements */
/* interact independently from outside elements */

/* Creating a BFC prevents: */
/* 1. Margin collapse */
/* 2. Float overlapping */
/* 3. Provides containment */

/* Ways to create BFC */

/* 1. Root element (html) */
/* Already a BFC */

/* 2. Float */
.float-container {
  float: left; /* Creates BFC */
}

/* 3. Absolute/Fixed positioning */
.positioned {
  position: absolute; /* Creates BFC */
}

/* 4. Display */
.inline-block { display: inline-block; } /* BFC */
.table-cell { display: table-cell; }     /* BFC */
.flex { display: flex; }                 /* BFC */
.grid { display: grid; }                 /* BFC */

/* 5. Overflow (not visible) */
.overflow {
  overflow: hidden;  /* BFC */
  overflow: auto;    /* BFC */
  overflow: scroll;  /* BFC */
}

/* 6. Modern solution */
.flow-root {
  display: flow-root; /* Cleanest BFC creation */
}

/* Practical examples */

/* Problem: Parent collapses with floated children */
.parent {
  border: 2px solid red;
}
.child {
  float: left;
  width: 100px;
  height: 100px;
}
/* Parent height = 0 */

/* Solution: Create BFC */
.parent {
  overflow: hidden; /* or display: flow-root; */
}
/* Parent now contains floated children */

/* Problem: Margin collapse */
.container {
  background: lightgray;
}
.box {
  margin-top: 20px;
}
/* Margin escapes container */

/* Solution: BFC */
.container {
  display: flow-root; /* Prevents margin collapse */
}

/* Problem: Float overlapping text */
.image {
  float: left;
}
.text {
  /* Text wraps around image */
}

/* Solution: BFC for text */
.text {
  display: flow-root; /* Creates separate context */
  /* or */
  overflow: hidden;
}
```

---

## <a id="layouts"></a>📐 Layout Systems (Flexbox & Grid)

### Q5: Explain Flexbox properties in detail. What's the difference between `flex-basis`, `flex-grow`, and `flex-shrink`?

**Expected Answer:**
```css
/* FLEX CONTAINER PROPERTIES */

.container {
  display: flex;
  
  /* Main axis direction */
  flex-direction: row | row-reverse | column | column-reverse;
  
  /* Wrapping */
  flex-wrap: nowrap | wrap | wrap-reverse;
  
  /* Shorthand */
  flex-flow: row wrap;
  
  /* Main axis alignment */
  justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
  
  /* Cross axis alignment */
  align-items: stretch | flex-start | flex-end | center | baseline;
  
  /* Multi-line cross axis alignment */
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
  
  /* Gap between items */
  gap: 10px; /* Modern way */
  row-gap: 10px;
  column-gap: 20px;
}

/* FLEX ITEM PROPERTIES */

/* flex-grow: How much item should grow relative to others */
.item {
  flex-grow: 0; /* Default - don't grow */
  flex-grow: 1; /* Grow to fill space */
  flex-grow: 2; /* Grow twice as much as items with flex-grow: 1 */
}

/* Example with flex-grow */
.container {
  width: 600px;
  display: flex;
}
.item-1 { flex-grow: 1; } /* Gets 200px */
.item-2 { flex-grow: 2; } /* Gets 400px */
/* Available space divided in 1:2 ratio */

/* flex-shrink: How much item should shrink when space is limited */
.item {
  flex-shrink: 1; /* Default - can shrink */
  flex-shrink: 0; /* Won't shrink below flex-basis */
  flex-shrink: 2; /* Shrinks twice as fast */
}

/* flex-basis: Initial size before growing/shrinking */
.item {
  flex-basis: auto; /* Default - based on content */
  flex-basis: 200px; /* Fixed starting size */
  flex-basis: 50%; /* Percentage of container */
  flex-basis: 0; /* Start from zero, grow/shrink from there */
}

/* FLEX SHORTHAND */
.item {
  /* flex: <grow> <shrink> <basis> */
  flex: 0 1 auto; /* Default */
  flex: 1;        /* Same as: 1 1 0% */
  flex: auto;     /* Same as: 1 1 auto */
  flex: none;     /* Same as: 0 0 auto */
}

/* Common patterns */
.equal-width {
  flex: 1; /* All items equal width */
}

.fixed-width {
  flex: 0 0 200px; /* Fixed 200px, no grow/shrink */
}

.grow-only {
  flex: 1 0 auto; /* Grow but don't shrink */
}

/* OTHER FLEX ITEM PROPERTIES */

/* Override align-items for specific item */
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}

/* Change visual order (doesn't affect DOM) */
.item {
  order: 0; /* Default */
  order: -1; /* Move to start */
  order: 1; /* Move to end */
}

/* PRACTICAL EXAMPLES */

/* Holy Grail Layout */
.container {
  display: flex;
  min-height: 100vh;
}
.sidebar { flex: 0 0 200px; }
.main { flex: 1; }
.aside { flex: 0 0 150px; }

/* Centering */
.center {
  display: flex;
  justify-content: center; /* Horizontal */
  align-items: center;     /* Vertical */
}

/* Sticky footer */
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
main { flex: 1; }
footer { flex: 0 0 auto; }

/* Equal height cards */
.card-container {
  display: flex;
  gap: 20px;
}
.card {
  flex: 1;
  display: flex;
  flex-direction: column;
}
.card-content { flex: 1; }
```

### Q6: Explain CSS Grid in detail. When would you use Grid vs Flexbox?

**Expected Answer:**
```css
/* CSS GRID - 2D Layout System */

/* GRID CONTAINER PROPERTIES */

.grid-container {
  display: grid;
  
  /* Define columns */
  grid-template-columns: 200px 1fr 2fr; /* Fixed, flexible, more flexible */
  grid-template-columns: repeat(3, 1fr); /* 3 equal columns */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* Responsive */
  
  /* Define rows */
  grid-template-rows: 100px auto 50px;
  grid-template-rows: repeat(3, 100px);
  
  /* Gap between cells */
  gap: 20px; /* Both row and column */
  row-gap: 10px;
  column-gap: 20px;
  
  /* Named grid areas */
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  
  /* Alignment */
  justify-items: start | end | center | stretch; /* Horizontal */
  align-items: start | end | center | stretch;   /* Vertical */
  justify-content: start | end | center | stretch | space-between | space-around;
  align-content: start | end | center | stretch | space-between | space-around;
  
  /* Implicit grid (auto-generated rows/columns) */
  grid-auto-rows: 100px;
  grid-auto-columns: 1fr;
  grid-auto-flow: row | column | dense;
}

/* GRID ITEM PROPERTIES */

.grid-item {
  /* Position by line numbers */
  grid-column-start: 1;
  grid-column-end: 3;
  grid-row-start: 1;
  grid-row-end: 2;
  
  /* Shorthand */
  grid-column: 1 / 3; /* Start / End */
  grid-row: 1 / 2;
  
  /* Span syntax */
  grid-column: span 2; /* Span 2 columns */
  grid-row: 2 / span 3; /* Start at 2, span 3 rows */
  
  /* Named areas */
  grid-area: header;
  
  /* Individual alignment */
  justify-self: start | end | center | stretch;
  align-self: start | end | center | stretch;
  
  /* Z-index works */
  z-index: 1;
}

/* ADVANCED GRID FEATURES */

/* fr unit - fraction of available space */
.grid {
  grid-template-columns: 1fr 2fr 1fr;
  /* Second column gets twice the space */
}

/* minmax() - responsive sizing */
.grid {
  grid-template-columns: minmax(200px, 1fr) 2fr;
  /* First column: min 200px, max 1fr */
}

/* auto-fit vs auto-fill */
.auto-fit {
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  /* Expands items to fill space */
}

.auto-fill {
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  /* Creates empty tracks */
}

/* Subgrid (limited support) */
.nested-grid {
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}

/* GRID vs FLEXBOX - When to use what? */

/* Use GRID when: */
/* ✓ 2D layout (rows AND columns) */
/* ✓ Complex layouts with overlapping */
/* ✓ You need to control both axes */
/* ✓ Layout-first approach */

.dashboard {
  display: grid;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
}

/* Use FLEXBOX when: */
/* ✓ 1D layout (row OR column) */
/* ✓ Content-first approach */
/* ✓ Distributing space among items */
/* ✓ Simple alignment needs */

.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* PRACTICAL GRID EXAMPLES */

/* Responsive card grid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

/* Dashboard layout */
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr 300px;
  grid-template-rows: 60px 1fr 40px;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  min-height: 100vh;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }

/* Magazine layout */
.magazine {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 20px;
}

.feature {
  grid-column: 1 / 9; /* Spans 8 columns */
  grid-row: 1 / 3;    /* Spans 2 rows */
}

.sidebar {
  grid-column: 9 / 13; /* Last 4 columns */
}

/* Masonry-like layout (imperfect, use grid-auto-flow: dense) */
.masonry {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  grid-auto-rows: 50px;
  grid-auto-flow: dense;
}

.item-tall { grid-row: span 4; }
.item-wide { grid-column: span 2; }
```

### Q7: How do you create a responsive layout without media queries?

**Expected Answer:**
```css
/* Modern CSS allows responsive layouts without media queries */

/* 1. CSS GRID with auto-fit/auto-fill */
.grid-responsive {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}
/* Items automatically wrap when container is too small */

/* 2. FLEXBOX with flex-wrap */
.flex-responsive {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.flex-responsive > * {
  flex: 1 1 300px; /* Grow, shrink, min 300px */
}
/* Items wrap when they can't maintain 300px */

/* 3. CLAMP() - Responsive sizing */
.responsive-text {
  font-size: clamp(1rem, 2.5vw, 2rem);
  /* Min 1rem, preferred 2.5vw, max 2rem */
}

.responsive-width {
  width: clamp(300px, 50%, 800px);
  /* Min 300px, preferred 50%, max 800px */
}

/* 4. CSS VARIABLES with calc() */
:root {
  --min-size: 300px;
  --max-size: 1200px;
  --preferred: 50vw;
}

.container {
  width: min(var(--max-size), max(var(--min-size), var(--preferred)));
}

/* 5. CONTAINER QUERIES (Modern) */
.container {
  container-type: inline-size;
  container-name: sidebar;
}

@container sidebar (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}

/* 6. ASPECT-RATIO */
.video {
  aspect-ratio: 16 / 9; /* Maintains ratio */
  width: 100%;
}

/* 7. MIN/MAX/CLAMP combinations */
.responsive-container {
  /* Fluid width between 300px and 1200px */
  width: min(100% - 2rem, 1200px);
  margin-inline: auto;
  padding-inline: clamp(1rem, 5vw, 3rem);
}

/* 8. LOGICAL PROPERTIES */
.responsive-spacing {
  padding-inline: clamp(1rem, 5%, 3rem);
  padding-block: clamp(0.5rem, 2%, 2rem);
  margin-inline: auto;
}

/* 9. INTRINSIC SIZING */
.intrinsic {
  width: fit-content; /* Shrinks to content */
  max-width: 100%;    /* But never exceeds parent */
}

.min-content { width: min-content; } /* Narrowest possible */
.max-content { width: max-content; } /* Widest without wrapping */

/* 10. COMPLETE RESPONSIVE LAYOUT EXAMPLE */
.page-layout {
  display: grid;
  gap: clamp(1rem, 3vw, 2rem);
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 300px), 1fr));
  padding: clamp(1rem, 5vw, 3rem);
  max-width: 1400px;
  margin-inline: auto;
}

.card {
  container-type: inline-size;
  padding: clamp(1rem, 3%, 2rem);
}

@container (min-width: 400px) {
  .card-content {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: 1rem;
  }
}

/* RESPONSIVE TYPOGRAPHY */
.responsive-type {
  font-size: clamp(1rem, 0.8rem + 0.5vw, 1.5rem);
  line-height: 1.5;
  /* Scales smoothly between viewports */
}

/* RESPONSIVE SPACING */
:root {
  --space-xs: clamp(0.5rem, 1vw, 1rem);
  --space-sm: clamp(1rem, 2vw, 1.5rem);
  --space-md: clamp(1.5rem, 3vw, 2.5rem);
  --space-lg: clamp(2rem, 5vw, 4rem);
  --space-xl: clamp(3rem, 8vw, 6rem);
}

.section {
  padding-block: var(--space-lg);
}
```

---

## <a id="responsive"></a>📱 Responsive Design & Media Queries

### Q8: Explain mobile-first vs desktop-first approach. What are the best practices for media queries?

**Expected Answer:**
```css
/* MOBILE-FIRST APPROACH (Recommended) */
/* Start with mobile styles, add complexity for larger screens */

/* Base styles (mobile) */
.container {
  padding: 1rem;
  font-size: 14px;
}

.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

/* Tablet and up */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
    font-size: 16px;
  }
  
  .grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 2rem;
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    padding: 3rem;
    max-width: 1200px;
    margin: 0 auto;
  }
  
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* DESKTOP-FIRST APPROACH */
/* Start with desktop, simplify for smaller screens */

/* Base styles (desktop) */
.container {
  padding: 3rem;
  font-size: 16px;
  max-width: 1200px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2rem;
}

/* Tablet and down */
@media (max-width: 1023px) {
  .container {
    padding: 2rem;
  }
  
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Mobile and down */
@media (max-width: 767px) {
  .container {
    padding: 1rem;
    font-size: 14px;
  }
  
  .grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
}

/* STANDARD BREAKPOINTS */
:root {
  /* Mobile first */
  --breakpoint-sm: 640px;   /* Small tablets */
  --breakpoint-md: 768px;   /* Tablets */
  --breakpoint-lg: 1024px;  /* Laptops */
  --breakpoint-xl: 1280px;  /* Desktops */
  --breakpoint-2xl: 1536px; /* Large desktops */
}

/* BEST PRACTICES */

/* 1. Use em/rem for breakpoints (respects user font size) */
@media (min-width: 48em) { /* 768px */ }

/* 2. Use logical properties */
@media (min-width: 768px) {
  .container {
    padding-inline: 2rem;  /* horizontal */
    padding-block: 1rem;   /* vertical */
    margin-inline: auto;   /* horizontal centering */
  }
}

/* 3. Range syntax (modern) */
@media (width >= 768px) {
  /* Cleaner than min-width */
}

@media (768px <= width < 1024px) {
  /* Between breakpoints */
}

/* 4. Feature queries with media queries */
@media (min-width: 768px) {
  @supports (display: grid) {
    .layout {
      display: grid;
    }
  }
}

/* 5. Combine multiple conditions */
@media (min-width: 768px) and (max-width: 1023px) {
  /* Tablet only */
}

@media (min-width: 1024px) and (orientation: landscape) {
  /* Desktop landscape */
}

/* 6. Media types and features */
@media screen and (min-width: 768px) { /* Screen only */ }
@media print { /* Print styles */ }
@media (prefers-color-scheme: dark) { /* Dark mode */ }
@media (prefers-reduced-motion: reduce) { /* Accessibility */ }
@media (hover: hover) { /* Devices with hover capability */ }
@media (pointer: fine) { /* Mouse/trackpad */ }
@media (pointer: coarse) { /* Touch */ }

/* ADVANCED RESPONSIVE PATTERNS */

/* Responsive navbar */
.navbar {
  display: flex;
  flex-direction: column;
}

@media (min-width: 768px) {
  .navbar {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }
}

/* Responsive images */
img {
  max-width: 100%;
  height: auto;
}

/* Art direction with picture */
<picture>
  <source media="(min-width: 1024px)" srcset="large.jpg">
  <source media="(min-width: 768px)" srcset="medium.jpg">
  <img src="small.jpg" alt="Description">
</picture>

/* Responsive typography scale */
:root {
  --font-size-sm: clamp(0.8rem, 0.7rem + 0.3vw, 1rem);
  --font-size-base: clamp(1rem, 0.9rem + 0.5vw, 1.25rem);
  --font-size-lg: clamp(1.25rem, 1.1rem + 0.7vw, 1.75rem);
  --font-size-xl: clamp(1.5rem, 1.3rem + 1vw, 2.5rem);
  --font-size-2xl: clamp(2rem, 1.5rem + 2vw, 4rem);
}

/* Container queries for component-based responsiveness */
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}

@container (min-width: 600px) {
  .card {
    grid-template-columns: 1fr 3fr;
  }
}

/* PRINT STYLES */
@media print {
  /* Hide non-essential elements */
  nav, aside, footer, .no-print {
    display: none;
  }
  
  /* Optimize for printing */
  body {
    font-size: 12pt;
    line-height: 1.5;
    color: black;
    background: white;
  }
  
  /* Page breaks */
  h1, h2, h3 {
    page-break-after: avoid;
  }
  
  /* Show link URLs */
  a[href]::after {
    content: " (" attr(href) ")";
  }
}
```

---

## <a id="architecture"></a>🏗️ CSS Architecture & Methodology

### Q9: Explain BEM, SMACSS, and OOCSS methodologies. When would you use each?

**Expected Answer:**
```css
/* BEM - Block Element Modifier */
/* Block: Standalone entity (component) */
/* Element: Part of block, no standalone meaning */
/* Modifier: Different state or version */

/* Syntax: block__element--modifier */

/* Block */
.card { }

/* Elements */
.card__header { }
.card__title { }
.card__body { }
.card__footer { }
.card__button { }

/* Modifiers */
.card--featured { } /* Featured card */
.card--large { }    /* Large card */
.card__button--primary { } /* Primary button in card */
.card__button--disabled { } /* Disabled button */

/* Benefits */
/* ✓ Clear component structure */
/* ✓ No nesting needed (flat specificity) */
/* ✓ Avoids naming conflicts */
/* ✓ Self-documenting */

/* Example */
<div class="card card--featured">
  <div class="card__header">
    <h2 class="card__title">Title</h2>
  </div>
  <div class="card__body">
    <p class="card__text">Content</p>
  </div>
  <div class="card__footer">
    <button class="card__button card__button--primary">Action</button>
  </div>
</div>

/* OOCSS - Object-Oriented CSS */
/* Separate structure from skin */
/* Separate container from content */

/* Structure (layout) */
.box {
  display: block;
  padding: 1rem;
  margin-bottom: 1rem;
}

/* Skin (appearance) */
.box-primary {
  background: blue;
  color: white;
  border: 2px solid darkblue;
}

.box-secondary {
  background: gray;
  color: black;
  border: 2px solid darkgray;
}

/* Combine */
<div class="box box-primary">Primary box</div>
<div class="box box-secondary">Secondary box</div>

/* Media object pattern (OOCSS) */
.media {
  display: flex;
  align-items: flex-start;
}

.media__image {
  margin-right: 1rem;
  flex-shrink: 0;
}

.media__body {
  flex: 1;
}

/* Benefits */
/* ✓ Reusable styles */
/* ✓ Smaller CSS files */
/* ✓ Flexible combinations */

/* SMACSS - Scalable and Modular Architecture */
/* Categorizes CSS into 5 types */

/* 1. BASE - Defaults, element selectors */
html {
  box-sizing: border-box;
}

body {
  font-family: Arial, sans-serif;
  line-height: 1.6;
}

a {
  color: blue;
  text-decoration: none;
}

/* 2. LAYOUT - Major sections, prefixed with l- */
.l-header {
  position: sticky;
  top: 0;
}

.l-sidebar {
  width: 250px;
}

.l-main {
  flex: 1;
}

.l-footer {
  padding: 2rem;
}

/* 3. MODULE - Reusable components */
.button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* 4. STATE - States of modules, prefixed with is- */
.is-active {
  font-weight: bold;
}

.is-hidden {
  display: none;
}

.is-loading {
  opacity: 0.5;
  pointer-events: none;
}

/* 5. THEME - Color schemes, typography */
.theme-dark {
  --bg-color: #1a1a1a;
  --text-color: #ffffff;
}

.theme-light {
  --bg-color: #ffffff;
  --text-color: #000000;
}

/* COMPARISON */

/* BEM: Best for */
/* ✓ Component-based projects */
/* ✓ Large teams */
/* ✓ React/Vue/Angular apps */

/* OOCSS: Best for */
/* ✓ Design systems */
/* ✓ Utility classes */
/* ✓ Maximizing reusability */

/* SMACSS: Best for */
/* ✓ Large projects */
/* ✓ Clear organization */
/* ✓ Multiple developers */

/* MODERN APPROACH - Combine methodologies */

/* Use BEM for components */
.user-card { }
.user-card__avatar { }
.user-card__name { }

/* Use OOCSS for utilities */
.mt-1 { margin-top: 0.25rem; }
.mb-2 { margin-bottom: 0.5rem; }
.flex { display: flex; }
.items-center { align-items: center; }

/* Use SMACSS categorization */
/* /base - Reset, typography */
/* /layout - Grid, containers */
/* /components - Buttons, cards */
/* /utilities - Helpers */
/* /themes - Color schemes */

/* CSS MODULES (Modern JS frameworks) */
/* Scoped styles, no naming conflicts */
.button {
  /* Compiled to .Button_button__abc123 */
  padding: 0.5rem 1rem;
}

/* ATOMIC CSS (Tailwind-style) */
<div class="flex items-center justify-between p-4 bg-white rounded-lg shadow">
  <h2 class="text-xl font-bold">Title</h2>
  <button class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
    Click
  </button>
</div>
```

### Q10: How do you manage CSS specificity issues in large projects?

**Expected Answer:**
```css
/* SPECIFICITY PROBLEMS IN LARGE PROJECTS */

/* Problem 1: Specificity wars */
/* BAD */
#header .nav ul li a { color: blue; }        /* (0,1,1,3) */
.navigation .menu .item .link { color: red; } /* (0,0,4,0) */
/* First one wins, causes confusion */

/* SOLUTION: Keep specificity low and flat */
/* GOOD */
.nav-link { color: blue; }

/* Problem 2: Overriding third-party styles */
/* BAD */
.some-library-class {
  color: red !important; /* Avoid !important */
}

/* GOOD: Increase specificity naturally */
.my-app .some-library-class {
  color: red;
}

/* Or use :where() to reset specificity */
:where(.some-library-class) {
  color: red; /* Specificity: (0,0,0,0) */
}

/* STRATEGIES FOR MANAGING SPECIFICITY */

/* 1. Use classes, avoid IDs for styling */
/* BAD */
#header { }

/* GOOD */
.header { }

/* 2. Avoid nesting */
/* BAD */
.header .nav .menu .item .link { }

/* GOOD (BEM) */
.nav__link { }

/* 3. Use CSS layers (modern) */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; }
}

@layer base {
  body { font-family: Arial; }
}

@layer components {
  .button { padding: 0.5rem 1rem; }
}

@layer utilities {
  .mt-4 { margin-top: 1rem; }
}

/* Layers have specificity: reset < base < components < utilities */
/* Even if utilities comes first in code, it has higher specificity */

/* 4. Scope styles with :where() */
:where(.card) {
  /* Specificity: (0,0,0,0) */
  /* Easy to override */
  padding: 1rem;
}

.card-special {
  /* Specificity: (0,0,1,0) */
  /* Overrides above */
  padding: 2rem;
}

/* 5. Use :is() for grouping (keeps specificity) */
/* BAD */
h1, h2, h3, h4, h5, h6 { }

/* GOOD */
:is(h1, h2, h3, h4, h5, h6) { }
/* Specificity = highest selector inside */

/* 6. Organize with ITCSS (Inverted Triangle CSS) */
/*
Settings    → Variables, config
Tools       → Mixins, functions
Generic     → Reset, normalize
Elements    → Element selectors
Objects     → Layout patterns
Components  → UI components
Utilities   → Helper classes

(Low specificity → High specificity)
*/

/* 7. CSS-in-JS solutions */
/* Automatically scoped, no global conflicts */
import styled from 'styled-components';

const Button = styled.button`
  /* Compiled to unique class name */
  padding: 0.5rem 1rem;
`;

/* 8. Shadow DOM for true encapsulation */
class MyComponent extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: 'open' });
    /* Styles in shadow DOM don't leak out */
  }
}

/* 9. Naming conventions */
/* Prefix components */
.c-button { } /* Component */
.l-grid { }   /* Layout */
.u-mt-1 { }   /* Utility */
.is-active { } /* State */
.t-dark { }   /* Theme */

/* 10. CSS Modules */
/* styles.module.css */
.button { }

/* Imported as */
import styles from './styles.module.css';
<button className={styles.button}>
/* Compiled to: .styles_button__unique123 */

/* SPECIFICITY DEBUGGING TOOLS */

/* 1. Browser DevTools */
/* Shows specificity and which rules are overridden */

/* 2. Specificity Calculator */
/* Tools like: https://specificity.keegan.st/ */

/* 3. Linters */
/* stylelint rules */
{
  "selector-max-id": 0,
  "selector-max-compound-selectors": 3,
  "selector-max-specificity": "0,3,0"
}

/* 4. CSS Stats tools */
/* Analyze specificity distribution */

/* PRACTICAL EXAMPLE: Refactoring high specificity */

/* BEFORE */
#main .content .post .post-header .post-title {
  font-size: 2rem;
  color: #333;
}

#main .content .post .post-header .post-meta {
  color: #666;
}

/* AFTER (BEM) */
.post__title {
  font-size: 2rem;
  color: #333;
}

.post__meta {
  color: #666;
}

/* Or with CSS Layers */
@layer components {
  .post__title { font-size: 2rem; color: #333; }
  .post__meta { color: #666; }
}

@layer utilities {
  .text-large { font-size: 2.5rem; } /* Can override components */
}
```

---

## <a id="animations"></a>🎬 Animations & Transitions

### Q11: What's the difference between CSS transitions and animations? When should you use each?

**Expected Answer:**
```css
/* TRANSITIONS */
/* Smooth change between two states (A → B) */
/* Triggered by state change (hover, focus, class toggle) */

.button {
  background: blue;
  color: white;
  transition: background 0.3s ease-in-out, transform 0.2s ease;
  /* property | duration | timing-function | delay */
}

.button:hover {
  background: darkblue;
  transform: scale(1.05);
}

/* Transition properties */
.element {
  transition-property: all; /* or specific: background, transform */
  transition-duration: 300ms;
  transition-timing-function: ease; /* ease, linear, ease-in, ease-out, ease-in-out, cubic-bezier */
  transition-delay: 0s;
  
  /* Shorthand */
  transition: all 0.3s ease 0s;
}

/* Multiple transitions */
.box {
  transition: 
    width 0.3s ease,
    height 0.3s ease 0.1s, /* 0.1s delay */
    background 0.5s linear;
}

/* ANIMATIONS */
/* Complex, multi-step animations */
/* Can run automatically, loop, reverse */

/* Define keyframes */
@keyframes slideIn {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

/* Apply animation */
.element {
  animation: slideIn 0.5s ease-out forwards;
  /* name | duration | timing-function | fill-mode */
}

/* Animation properties */
.animated {
  animation-name: slideIn;
  animation-duration: 1s;
  animation-timing-function: ease-in-out;
  animation-delay: 0.5s;
  animation-iteration-count: infinite; /* or number */
  animation-direction: normal; /* normal, reverse, alternate, alternate-reverse */
  animation-fill-mode: forwards; /* forwards, backwards, both, none */
  animation-play-state: running; /* running, paused */
  
  /* Shorthand */
  animation: slideIn 1s ease-in-out 0.5s infinite alternate forwards;
}

/* COMPLEX KEYFRAME ANIMATIONS */

/* Bounce effect */
@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-30px);
  }
  60% {
    transform: translateY(-15px);
  }
}

.bounce {
  animation: bounce 2s ease infinite;
}

/* Pulse effect */
@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

/* Loading spinner */
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.spinner {
  animation: spin 1s linear infinite;
}

/* Fade in and slide */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* WHEN TO USE WHAT? */

/* Use TRANSITIONS when: */
/* ✓ Simple state changes */
/* ✓ User-triggered (hover, focus, click) */
/* ✓ Two states (A → B) */
/* Examples: button hover, menu expand, tooltip */

.button {
  transition: all 0.2s ease;
}
.button:hover { transform: scale(1.05); }

/* Use ANIMATIONS when: */
/* ✓ Complex, multi-step animations */
/* ✓ Automatic/programmatic */
/* ✓ Need looping */
/* ✓ Need fine control */
/* Examples: loading spinners, attention-grabbing effects */

.loading {
  animation: spin 1s linear infinite;
}

/* PERFORMANCE CONSIDERATIONS */

/* GOOD: Animate transform and opacity (GPU-accelerated) */
.fast {
  animation: slideAndFade 0.3s ease;
}

@keyframes slideAndFade {
  from {
    transform: translateX(-20px);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

/* BAD: Animate layout properties (causes reflow) */
.slow {
  animation: moveSlow 0.3s ease;
}

@keyframes moveSlow {
  from {
    left: 0; /* BAD - causes reflow */
    width: 100px; /* BAD - causes reflow */
  }
  to {
    left: 100px;
    width: 200px;
  }
}

/* WILL-CHANGE for optimization */
.will-animate {
  will-change: transform, opacity;
  /* Tells browser to optimize */
}

.will-animate:hover {
  transform: scale(1.2);
  opacity: 0.8;
}

/* Remove will-change after animation */
.element {
  animation: slide 0.3s ease forwards;
}

.element.animation-complete {
  will-change: auto; /* Reset */
}

/* ACCESSIBILITY - Respect user preferences */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* Or provide alternative */
@media (prefers-reduced-motion: no-preference) {
  .animated {
    animation: slideIn 0.5s ease;
  }
}

@media (prefers-reduced-motion: reduce) {
  .animated {
    animation: none;
    /* Show final state immediately */
  }
}

/* CONTROLLING ANIMATIONS WITH JS */
/* Pause/Resume */
.paused {
  animation-play-state: paused;
}

/* Detect animation end */
element.addEventListener('animationend', () => {
  console.log('Animation completed');
});

/* STAGGERED ANIMATIONS */
.item {
  animation: fadeInUp 0.5s ease forwards;
  opacity: 0;
}

.item:nth-child(1) { animation-delay: 0.1s; }
.item:nth-child(2) { animation-delay: 0.2s; }
.item:nth-child(3) { animation-delay: 0.3s; }

/* Or with CSS custom properties */
.item {
  --delay: 0;
  animation: fadeInUp 0.5s ease forwards;
  animation-delay: calc(var(--delay) * 0.1s);
  opacity: 0;
}

/* Set in HTML */
<div class="item" style="--delay: 1"></div>
<div class="item" style="--delay: 2"></div>
<div class="item" style="--delay: 3"></div>
```

### Q12: How do you create performant animations? What properties should you animate?

**Expected Answer:**
```css
/* PERFORMANCE HIERARCHY */

/* BEST: GPU-accelerated properties (composite layer) */
/* transform, opacity */

/* BAD: Layout properties (causes reflow) */
/* width, height, top, left, right, bottom, margin, padding */

/* BAD: Paint properties (