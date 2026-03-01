# 🎨 The Complete CSS Cheat Sheet
### From Zero to Expert

---

## 1. How CSS Works

CSS (Cascading Style Sheets) controls the visual presentation of HTML. There are three ways to include CSS:

```html
<!-- 1. External stylesheet (BEST - reusable across pages) -->
<link rel="stylesheet" href="styles.css">

<!-- 2. Internal (inside <head>) -->
<style>
  p { color: red; }
</style>

<!-- 3. Inline (AVOID - hard to maintain) -->
<p style="color: red;">Hello</p>
```

### CSS Rule Anatomy

```css
selector {
  property: value;  /* declaration */
  font-size: 16px;
  color: #333;
}
```

### The Cascade
Styles "cascade" — later rules override earlier ones (when specificity is equal):

```css
p { color: blue; }
p { color: red; }  /* ← This wins */
```

---

## 2. Selectors

### Basic Selectors

```css
*            { }   /* Universal — every element */
div          { }   /* Type/element selector */
.classname   { }   /* Class selector */
#idname      { }   /* ID selector (unique per page) */
```

### Combinators

```css
div p        { }   /* Descendant — any p inside a div */
div > p      { }   /* Child — direct p children of div */
h1 + p       { }   /* Adjacent sibling — p immediately after h1 */
h1 ~ p       { }   /* General sibling — all p after h1 */
```

### Attribute Selectors

```css
[type]            { }   /* Has this attribute */
[type="text"]     { }   /* Exact match */
[class~="btn"]    { }   /* Contains word "btn" in space-separated list */
[href^="https"]   { }   /* Starts with */
[href$=".pdf"]    { }   /* Ends with */
[href*="google"]  { }   /* Contains anywhere */
```

### Grouping

```css
h1, h2, h3, .title { font-weight: bold; }  /* Apply to all */
```

---

## 3. The Box Model

Every HTML element is a rectangular box made of:

```
┌─────────────────────────────────────┐
│             MARGIN                  │  ← Outside space (transparent)
│  ┌───────────────────────────────┐  │
│  │           BORDER              │  │  ← Visible border
│  │  ┌─────────────────────────┐  │  │
│  │  │         PADDING         │  │  │  ← Inside space (inherits bg)
│  │  │  ┌───────────────────┐  │  │  │
│  │  │  │      CONTENT      │  │  │  │  ← width × height
│  │  │  └───────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

```css
.box {
  width: 300px;
  height: 200px;
  padding: 20px;           /* All sides */
  padding: 10px 20px;      /* Top/bottom  Left/right */
  padding: 5px 10px 15px 20px; /* Top Right Bottom Left (clockwise) */
  margin: 10px auto;       /* auto = center horizontally */
  border: 2px solid #333;
  border-radius: 8px;      /* Rounded corners */
  border-radius: 50%;      /* Circle (on equal width/height) */
}
```

### Box Sizing (CRITICAL!)

```css
/* Default: width = content only (padding/border add to total size) */
* {
  box-sizing: border-box; /* ALWAYS add this — width includes padding + border */
}
```

### Display

```css
display: block;        /* Full width, stacks vertically (div, p, h1) */
display: inline;       /* Flows with text, ignores width/height (span, a) */
display: inline-block; /* Flows with text BUT respects width/height */
display: none;         /* Hidden, takes no space */
display: flex;         /* Flexbox container */
display: grid;         /* Grid container */
```

### Overflow

```css
overflow: visible;   /* Default — spills out */
overflow: hidden;    /* Clip content */
overflow: scroll;    /* Always show scrollbar */
overflow: auto;      /* Scrollbar only when needed */
overflow-x: hidden;  /* Horizontal only */
overflow-y: scroll;  /* Vertical only */
```

---

## 4. Typography

### Font Properties

```css
.text {
  font-family: 'Inter', Arial, sans-serif; /* Stack: fallbacks if first fails */
  font-size: 16px;      /* px, em, rem, %, vw */
  font-weight: 400;     /* 100–900, or: normal, bold */
  font-style: italic;   /* normal, italic, oblique */
  font-variant: small-caps;
  line-height: 1.5;     /* Unitless preferred — relative to font-size */
  letter-spacing: 0.05em;
  word-spacing: 2px;
}
```

### Text Properties

```css
.text {
  text-align: left | center | right | justify;
  text-decoration: none | underline | line-through | overline;
  text-transform: uppercase | lowercase | capitalize | none;
  text-indent: 2em;
  text-overflow: ellipsis; /* Shows "..." — needs overflow:hidden & white-space:nowrap */
  white-space: nowrap | pre | pre-wrap | normal;
  word-break: break-word;  /* Wrap long words */
  color: #333;
}
```

### Font Units

```css
/* px — absolute, predictable */
font-size: 16px;

/* em — relative to parent font-size (compounds!) */
/* If parent is 16px, 1.5em = 24px */
font-size: 1.5em;

/* rem — relative to root html font-size (preferred) */
/* If html is 16px, 1.5rem = 24px, everywhere */
font-size: 1.5rem;

/* vw/vh — relative to viewport */
font-size: 4vw;  /* 4% of viewport width */
```

### Google Fonts

```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
```
```css
body { font-family: 'Inter', sans-serif; }
```

### Truncate Text with Ellipsis

```css
.truncate {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Multi-line truncation */
.clamp {
  display: -webkit-box;
  -webkit-line-clamp: 3;       /* Show 3 lines max */
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

---

## 5. Colors & Backgrounds

### Color Formats

```css
color: red;                  /* Named color */
color: #ff0000;              /* Hex */
color: #f00;                 /* Shorthand hex */
color: rgb(255, 0, 0);       /* RGB */
color: rgba(255, 0, 0, 0.5); /* RGB + alpha (opacity 0–1) */
color: hsl(0, 100%, 50%);    /* Hue Saturation Lightness */
color: hsla(0, 100%, 50%, 0.5);
color: oklch(55% 0.2 30);    /* Modern — perceptually uniform */
```

### Background

```css
.element {
  background-color: #f5f5f5;
  background-image: url('image.jpg');
  background-size: cover;     /* Fill container, may crop */
  background-size: contain;   /* Fit inside, may letterbox */
  background-size: 100% 100%;
  background-position: center center; /* x y */
  background-repeat: no-repeat | repeat | repeat-x | repeat-y;
  background-attachment: scroll | fixed | local;
  background-clip: border-box | padding-box | content-box | text;
  
  /* Shorthand: color image repeat position / size */
  background: #fff url('bg.png') no-repeat center / cover;
}
```

### Gradients

```css
/* Linear */
background: linear-gradient(to right, #ff6b6b, #ffd93d);
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
background: linear-gradient(to bottom, transparent, rgba(0,0,0,0.8));

/* Radial */
background: radial-gradient(circle, #ff6b6b, #764ba2);
background: radial-gradient(circle at top left, #fff 0%, #000 100%);

/* Conic */
background: conic-gradient(from 0deg, red, yellow, green, blue, red);

/* Multiple layers (first = top) */
background:
  linear-gradient(rgba(0,0,0,0.4), rgba(0,0,0,0.4)),
  url('photo.jpg') center / cover;
```

### Opacity vs Alpha

```css
/* opacity affects the element AND all its children */
.element { opacity: 0.5; }

/* rgba/hsla only affects that one property */
.element { background-color: rgba(0, 0, 0, 0.5); color: black; } /* text stays opaque */
```

---

## 6. Layout: Display & Positioning

### Position

```css
position: static;    /* Default — normal document flow */
position: relative;  /* Offset from normal position; creates positioning context */
position: absolute;  /* Removed from flow; positioned relative to nearest non-static ancestor */
position: fixed;     /* Removed from flow; fixed to viewport */
position: sticky;    /* Normal flow until threshold, then fixed */

/* Used with: */
top: 10px;
right: 0;
bottom: 0;
left: 50%;
z-index: 10;  /* Stack order (higher = in front); only works on positioned elements */
```

### Positioning Recipes

```css
/* Center absolutely positioned element */
.centered {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* Sticky header */
header {
  position: sticky;
  top: 0;
  z-index: 100;
}

/* Fixed overlay */
.overlay {
  position: fixed;
  inset: 0;  /* shorthand for top:0 right:0 bottom:0 left:0 */
  background: rgba(0,0,0,0.5);
}
```

### Float (Legacy — prefer Flexbox/Grid)

```css
img { float: left; margin-right: 10px; }
.clearfix::after { content: ''; display: block; clear: both; }
```

---

## 7. Flexbox

Flexbox handles **one-dimensional** layout (row OR column).

### Container Properties

```css
.container {
  display: flex;

  /* Direction */
  flex-direction: row | row-reverse | column | column-reverse;

  /* Wrapping */
  flex-wrap: nowrap | wrap | wrap-reverse;

  /* Shorthand for flex-direction + flex-wrap */
  flex-flow: row wrap;

  /* Alignment on MAIN axis (row = horizontal) */
  justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;

  /* Alignment on CROSS axis (row = vertical) */
  align-items: stretch | flex-start | flex-end | center | baseline;

  /* Alignment of wrapped lines */
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;

  gap: 16px;           /* Space between items */
  gap: 16px 24px;      /* Row gap  Column gap */
}
```

### Item Properties

```css
.item {
  /* Growth and shrink */
  flex-grow: 1;    /* How much to grow (0 = don't grow) */
  flex-shrink: 1;  /* How much to shrink (0 = don't shrink) */
  flex-basis: auto; /* Initial size before growing/shrinking */

  /* Shorthand */
  flex: 1;           /* flex: 1 1 0 — grow and shrink equally */
  flex: 0 0 200px;   /* Fixed 200px, no grow/shrink */

  /* Individual cross-axis alignment */
  align-self: auto | stretch | flex-start | flex-end | center | baseline;

  /* Order (default 0; lower = earlier) */
  order: -1;
}
```

### Flexbox Cheat Patterns

```css
/* Horizontal center */
.parent { display: flex; justify-content: center; }

/* Vertical center */
.parent { display: flex; align-items: center; }

/* Perfect center */
.parent { display: flex; justify-content: center; align-items: center; }

/* Space items out */
.nav { display: flex; justify-content: space-between; align-items: center; }

/* Equal columns */
.columns { display: flex; }
.columns > * { flex: 1; }

/* Push last item to end */
.flex-parent { display: flex; }
.push-right { margin-left: auto; }
```

---

## 8. CSS Grid

Grid handles **two-dimensional** layout (rows AND columns).

### Container Properties

```css
.container {
  display: grid;

  /* Define columns */
  grid-template-columns: 200px 1fr 1fr;     /* Fixed + flexible */
  grid-template-columns: repeat(3, 1fr);     /* 3 equal columns */
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); /* Responsive */

  /* Define rows */
  grid-template-rows: auto 1fr auto;         /* Header, content, footer */
  grid-template-rows: repeat(3, 100px);

  /* Gaps */
  gap: 24px;
  column-gap: 24px;
  row-gap: 16px;

  /* Named areas */
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";

  /* Alignment */
  justify-items: start | end | center | stretch;   /* Horizontal within cells */
  align-items: start | end | center | stretch;     /* Vertical within cells */
  justify-content: start | end | center | stretch | space-between | space-around;
  align-content: start | end | center | stretch | space-between | space-around;
}
```

### Item Properties

```css
.item {
  /* Span columns */
  grid-column: 1 / 3;       /* From line 1 to line 3 */
  grid-column: span 2;       /* Span 2 columns */
  grid-column: 1 / -1;       /* Full width */

  /* Span rows */
  grid-row: 1 / 3;
  grid-row: span 2;

  /* Assign to named area */
  grid-area: header;
  grid-area: sidebar;
  grid-area: main;
  grid-area: footer;

  /* Individual alignment */
  justify-self: start | end | center | stretch;
  align-self: start | end | center | stretch;
}
```

### Grid Recipes

```css
/* Holy Grail Layout */
.page {
  display: grid;
  grid-template-areas:
    "header header header"
    "nav    main   aside"
    "footer footer footer";
  grid-template-rows: auto 1fr auto;
  grid-template-columns: 200px 1fr 200px;
  min-height: 100vh;
}

/* Responsive card grid */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

/* Centering with grid */
.center {
  display: grid;
  place-items: center; /* Shorthand for justify-items + align-items */
}
```

---

## 9. Responsive Design & Media Queries

### Viewport Meta Tag (Required!)

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

### Media Queries

```css
/* Mobile-first (RECOMMENDED — start small, add complexity) */
.element { font-size: 14px; }       /* Mobile default */

@media (min-width: 768px) {         /* Tablet and up */
  .element { font-size: 16px; }
}

@media (min-width: 1024px) {        /* Desktop and up */
  .element { font-size: 18px; }
}

/* Desktop-first (start large, scale down) */
@media (max-width: 767px) { }
@media (max-width: 1023px) { }
```

### Common Breakpoints

```css
/* Bootstrap-style breakpoints */
@media (min-width: 576px)  { }  /* sm — Small devices */
@media (min-width: 768px)  { }  /* md — Medium devices */
@media (min-width: 992px)  { }  /* lg — Large devices */
@media (min-width: 1200px) { }  /* xl — Extra large */
@media (min-width: 1400px) { }  /* xxl */
```

### Other Media Features

```css
/* Dark mode */
@media (prefers-color-scheme: dark) {
  body { background: #1a1a1a; color: #fff; }
}

/* Reduced motion (accessibility!) */
@media (prefers-reduced-motion: reduce) {
  * { animation: none !important; transition: none !important; }
}

/* Print */
@media print {
  nav, footer { display: none; }
  body { font-size: 12pt; color: #000; }
}

/* Orientation */
@media (orientation: landscape) { }
@media (orientation: portrait) { }

/* Hover capability (touch vs mouse) */
@media (hover: hover) {
  .btn:hover { background: blue; }
}
```

### Fluid Typography

```css
/* clamp(min, preferred, max) */
h1 {
  font-size: clamp(1.5rem, 4vw, 3rem);  /* Never below 1.5rem, never above 3rem */
}
```

---

## 10. Transitions & Animations

### Transitions

Transitions smoothly change from one state to another (e.g., on hover).

```css
.button {
  background: blue;
  /* property  duration  timing-function  delay */
  transition: background 0.3s ease 0s;

  /* Multiple properties */
  transition: background 0.3s ease, transform 0.2s ease;

  /* All properties */
  transition: all 0.3s ease;
}

.button:hover {
  background: darkblue;
  transform: translateY(-2px);
}
```

### Timing Functions

```css
transition-timing-function: linear;           /* Constant speed */
transition-timing-function: ease;             /* Slow → fast → slow (default) */
transition-timing-function: ease-in;          /* Slow start */
transition-timing-function: ease-out;         /* Slow end */
transition-timing-function: ease-in-out;      /* Slow start and end */
transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); /* Custom curve */
transition-timing-function: steps(5, end);    /* Stepped/choppy */
```

### Keyframe Animations

```css
/* Define the animation */
@keyframes fadeIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes slideUp {
  0%   { transform: translateY(20px); opacity: 0; }
  100% { transform: translateY(0);    opacity: 1; }
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50%       { transform: translateY(-30px); }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50%       { transform: scale(1.05); }
}

/* Apply the animation */
.element {
  animation-name: fadeIn;
  animation-duration: 0.5s;
  animation-timing-function: ease;
  animation-delay: 0.1s;
  animation-iteration-count: 1;          /* or: infinite */
  animation-direction: normal;           /* or: reverse, alternate, alternate-reverse */
  animation-fill-mode: both;             /* forwards: stays at end; backwards: starts at 0%; both: both */
  animation-play-state: running;         /* or: paused */

  /* Shorthand: name duration timing delay count direction fill-mode */
  animation: fadeIn 0.5s ease 0.1s 1 normal both;

  /* Multiple animations */
  animation: fadeIn 0.5s ease, slideUp 0.5s ease;
}
```

### Useful Animation Snippets

```css
/* Skeleton loading pulse */
@keyframes shimmer {
  0%   { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}
.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

/* Spinner */
@keyframes spin {
  to { transform: rotate(360deg); }
}
.spinner {
  width: 32px; height: 32px;
  border: 3px solid #eee;
  border-top-color: #333;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

/* Fade in on load */
.hero { animation: fadeIn 1s ease both; }

/* Stagger children */
.item:nth-child(1) { animation-delay: 0.1s; }
.item:nth-child(2) { animation-delay: 0.2s; }
.item:nth-child(3) { animation-delay: 0.3s; }
```

---

## 11. Transforms

Transforms move, rotate, scale, or skew elements **without affecting layout**.

```css
/* 2D Transforms */
transform: translateX(50px);            /* Move right 50px */
transform: translateY(-20px);           /* Move up 20px */
transform: translate(50px, -20px);      /* Move X and Y */

transform: scaleX(1.5);                 /* Stretch horizontally */
transform: scaleY(0.8);                 /* Compress vertically */
transform: scale(1.2);                  /* Scale both axes equally */
transform: scale(1.2, 0.8);             /* Scale X and Y separately */

transform: rotate(45deg);              /* Rotate clockwise */
transform: rotate(-0.25turn);           /* Rotate using turns */

transform: skewX(15deg);
transform: skew(15deg, 5deg);

/* 3D Transforms */
transform: translateZ(50px);
transform: translate3d(10px, 20px, 30px);
transform: rotateX(45deg);              /* Tilt top back */
transform: rotateY(30deg);              /* Spin around Y axis */
transform: rotateZ(45deg);              /* Same as rotate() */
transform: perspective(500px) rotateY(20deg); /* Perspective for 3D depth */

/* Combine (applied right to left) */
transform: translateX(100px) rotate(45deg) scale(1.2);

/* Transform origin (default: center center) */
transform-origin: top left;
transform-origin: 0 0;
transform-origin: 50% 50%;

/* 3D context for children */
.parent {
  perspective: 1000px;         /* Distance of viewer */
  transform-style: preserve-3d;
}

/* Flip card */
.card { transform-style: preserve-3d; transition: transform 0.6s; }
.card:hover { transform: rotateY(180deg); }
.card-front, .card-back { backface-visibility: hidden; }
.card-back { transform: rotateY(180deg); }
```

---

## 12. Custom Properties (CSS Variables)

```css
/* Define on :root to make global */
:root {
  --color-primary: #6366f1;
  --color-text: #1a1a1a;
  --color-bg: #ffffff;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 32px;
  --radius: 8px;
  --shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  --font-sans: 'Inter', system-ui, sans-serif;
  --transition: 0.2s ease;
}

/* Use with var() */
.button {
  background: var(--color-primary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  transition: var(--transition);
}

/* Fallback value */
color: var(--color-text, #333);

/* Override in component scope */
.dark-card {
  --color-bg: #1a1a1a;
  --color-text: #ffffff;
  background: var(--color-bg);
  color: var(--color-text);
}

/* Dark mode via variables (elegant!) */
@media (prefers-color-scheme: dark) {
  :root {
    --color-text: #f5f5f5;
    --color-bg: #1a1a1a;
  }
}

/* Manipulate in JavaScript */
/* document.documentElement.style.setProperty('--color-primary', '#ff0000'); */
```

---

## 13. Pseudo-classes & Pseudo-elements

### Pseudo-classes (State/Position)

```css
/* User interaction */
a:hover       { }   /* Mouse over */
a:active      { }   /* Being clicked */
a:focus       { }   /* Keyboard focused */
a:visited     { }   /* Already visited */
input:focus-visible { }  /* Focused via keyboard (not mouse) */

/* Form state */
input:checked      { }
input:disabled     { }
input:enabled      { }
input:required     { }
input:valid        { }
input:invalid      { }
input:placeholder-shown { }

/* Structural */
:first-child       { }   /* First sibling */
:last-child        { }   /* Last sibling */
:nth-child(2)      { }   /* Second child */
:nth-child(odd)    { }   /* 1, 3, 5... */
:nth-child(even)   { }   /* 2, 4, 6... */
:nth-child(3n)     { }   /* Every 3rd: 3, 6, 9... */
:nth-child(3n+1)   { }   /* 1, 4, 7... */
:first-of-type     { }   /* First of this element type */
:last-of-type      { }   /* Last of this element type */
:nth-of-type(2)    { }
:only-child        { }   /* Only child */
:not(.excluded)    { }   /* Everything except .excluded */
:is(h1, h2, h3)    { }   /* Matches any in list */
:where(h1, h2, h3) { }   /* Same but zero specificity */
:has(.child)       { }   /* Parent has this child — CSS4! */
```

### Pseudo-elements (Virtual Elements)

```css
/* Pseudo-elements use :: (double colon) */
p::before {
  content: '→ ';   /* Required (can be empty string '') */
  color: blue;
}

p::after {
  content: '';
  display: block;
}

p::first-line    { font-weight: bold; }
p::first-letter  { font-size: 2em; float: left; }
::placeholder    { color: #aaa; font-style: italic; }
::selection      { background: #ffe08a; color: #000; }
::marker         { color: red; }       /* List bullet/number */

/* Useful patterns */

/* Clearfix */
.clearfix::after { content: ''; display: block; clear: both; }

/* Decorative line after heading */
h2::after {
  content: '';
  display: block;
  width: 50px; height: 3px;
  background: var(--color-primary);
  margin-top: 8px;
}

/* Tooltip with pseudo-element */
[data-tooltip]::after {
  content: attr(data-tooltip); /* Uses attribute value! */
  position: absolute;
  background: #333;
  color: #fff;
  padding: 4px 8px;
  border-radius: 4px;
  opacity: 0;
  transition: opacity 0.2s;
}
[data-tooltip]:hover::after { opacity: 1; }
```

---

## 14. Shadows, Filters & Effects

### Box Shadow

```css
/* offset-x  offset-y  blur  spread  color */
box-shadow: 2px 4px 8px 0px rgba(0, 0, 0, 0.15);

/* Multiple shadows */
box-shadow:
  0 1px 2px rgba(0,0,0,0.1),
  0 4px 8px rgba(0,0,0,0.1);

/* Inset shadow (inner) */
box-shadow: inset 0 2px 4px rgba(0,0,0,0.1);

/* Colored glow */
box-shadow: 0 0 20px rgba(99, 102, 241, 0.5);

/* No shadow */
box-shadow: none;
```

### Text Shadow

```css
/* offset-x  offset-y  blur  color */
text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
text-shadow: 0 0 10px rgba(255, 255, 255, 0.8);  /* Glow */
```

### Filters

```css
filter: blur(4px);
filter: brightness(1.2);      /* > 1 = brighter */
filter: contrast(1.5);
filter: grayscale(1);          /* 0–1 */
filter: hue-rotate(90deg);
filter: invert(1);             /* 0–1 */
filter: opacity(0.5);
filter: saturate(2);
filter: sepia(0.8);
filter: drop-shadow(2px 4px 8px rgba(0,0,0,0.3)); /* Like box-shadow but follows shape */

/* Combine */
filter: brightness(1.1) contrast(1.1) saturate(1.2);

/* Backdrop filter (blur behind element) */
.frosted-glass {
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.3);
}
```

### Blend Modes

```css
/* Mix element with background */
mix-blend-mode: multiply | screen | overlay | darken | lighten |
                color-dodge | color-burn | hard-light | soft-light |
                difference | exclusion | hue | saturation | color | luminosity;

/* Mix background images */
background-blend-mode: multiply;
```

---

## 15. Advanced Selectors & Specificity

### Specificity Rules

Specificity determines which rule wins when multiple rules target the same element.

```
 0 – 0 – 0
 │   │   │
 │   │   └── Type selectors (div, p), pseudo-elements (::before) = 0,0,1
 │   └────── Class (.foo), attributes ([type]), pseudo-classes (:hover) = 0,1,0
 └────────── ID (#foo) = 1,0,0
```

```css
/* Specificity: */
p              { }  /* 0,0,1 */
.box           { }  /* 0,1,0 */
#header        { }  /* 1,0,0 */
div.box        { }  /* 0,1,1 */
#header .nav a { }  /* 1,1,1 */

/* !important overrides everything (use sparingly!) */
color: red !important;
```

### Selector Tips

```css
/* :is() reduces repetition */
:is(h1, h2, h3) .subtitle { color: gray; }
/* is equivalent to: */
h1 .subtitle, h2 .subtitle, h3 .subtitle { color: gray; }

/* :where() — same but zero specificity */
:where(header, footer) a { color: inherit; }

/* :has() — the "parent selector" (modern browsers) */
/* Style a card when it has an image */
.card:has(img) { padding: 0; }
/* Style label when following invalid input */
input:invalid + label { color: red; }
/* Style form when it has a focused input */
form:has(:focus-visible) { border-color: blue; }

/* :not() with complex selectors (modern) */
a:not([href^="http"]):not([href^="#"]) { color: inherit; }
```

---

## 16. CSS Functions

```css
/* Math */
width: calc(100% - 2rem);          /* Mix units */
width: calc(100% / 3 - 1rem);
font-size: calc(1rem + 0.5vw);

min-width: min(80%, 600px);         /* Smaller of two values */
max-width: max(50%, 400px);         /* Larger of two values */
font-size: clamp(1rem, 2.5vw, 2rem);/* clamp(min, preferred, max) */

/* Colours */
color: rgb(255, 0, 0);
color: hsl(220, 90%, 50%);
color: color-mix(in srgb, blue 30%, white); /* Mix two colours */

/* Gradients (see section 5) */

/* Shapes */
clip-path: circle(50%);
clip-path: polygon(0 0, 100% 0, 50% 100%);

/* Counter */
counter-increment: section;
content: counter(section) ". ";

/* attr() — use HTML attributes in CSS */
content: attr(data-label);

/* env() — safe area insets (mobile notches) */
padding-bottom: env(safe-area-inset-bottom);

/* url() */
background-image: url('image.png');
cursor: url('cursor.png'), auto;
```

---

## 17. Clipping, Masking & Shapes

### Clip Path

```css
/* Hide parts of an element */
clip-path: circle(50% at 50% 50%);
clip-path: ellipse(60% 40% at 50% 50%);
clip-path: polygon(50% 0%, 100% 100%, 0% 100%); /* Triangle */
clip-path: inset(10% 20% 10% 20% round 8px);   /* Rounded rectangle cutout */

/* Diagonal section */
.section {
  clip-path: polygon(0 0, 100% 0, 100% 85%, 0 100%);
}
```

### CSS Shapes (Text wrapping)

```css
img {
  float: left;
  shape-outside: circle(50%);           /* Text wraps around a circle */
  shape-outside: polygon(0 0, 100% 0, 50% 100%);
  shape-outside: url('shape.png');      /* Wraps around image alpha */
  shape-margin: 10px;
}
```

### Object Fit (Images/Video)

```css
img, video {
  object-fit: fill;       /* Stretch (default) */
  object-fit: contain;    /* Letterbox */
  object-fit: cover;      /* Crop to fill */
  object-fit: none;       /* No resize */
  object-position: center center;  /* Like background-position */
  object-position: top;
}

/* Common pattern: fixed-size image container */
.avatar {
  width: 64px; height: 64px;
  border-radius: 50%;
  overflow: hidden;
}
.avatar img {
  width: 100%; height: 100%;
  object-fit: cover;
}
```

---

## 18. Design Patterns & Best Practices

### CSS Reset / Normalize

```css
/* Modern minimal reset */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  font-size: 16px;
  -webkit-text-size-adjust: 100%;
}

body {
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}

img, video, canvas, svg {
  display: block;
  max-width: 100%;
}

input, button, textarea, select {
  font: inherit;
}

p, h1, h2, h3, h4, h5, h6 {
  overflow-wrap: break-word;
}
```

### BEM Naming Convention

```css
/* Block__Element--Modifier */
.card { }                        /* Block */
.card__title { }                 /* Element */
.card__image { }                 /* Element */
.card--featured { }              /* Modifier */
.card__title--large { }          /* Element modifier */
```

### Design Tokens / Theme

```css
:root {
  /* Colors */
  --color-brand-50:  #eef2ff;
  --color-brand-500: #6366f1;
  --color-brand-900: #312e81;

  /* Semantic */
  --color-text-primary: #111;
  --color-text-secondary: #666;
  --color-surface: #fff;
  --color-surface-raised: #f9fafb;
  --color-border: #e5e7eb;

  /* Spacing scale */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;
  --space-16: 64px;

  /* Typography */
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-4xl: 2.25rem;

  /* Border radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1), 0 4px 6px rgba(0,0,0,0.05);

  /* Transitions */
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --duration-fast: 150ms;
  --duration-normal: 250ms;
}
```

### Utility Classes (Tailwind-style)

```css
.flex  { display: flex; }
.grid  { display: grid; }
.hidden { display: none; }

.items-center { align-items: center; }
.justify-center { justify-content: center; }
.justify-between { justify-content: space-between; }

.text-center { text-align: center; }
.font-bold { font-weight: 700; }
.uppercase { text-transform: uppercase; }

.rounded { border-radius: var(--radius-md); }
.rounded-full { border-radius: 9999px; }

.shadow { box-shadow: var(--shadow-md); }
.transition { transition: all var(--duration-normal) var(--ease-out); }

.sr-only { /* Screen-reader only — accessible hiding */
  position: absolute;
  width: 1px; height: 1px;
  padding: 0; margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### Common UI Components

```css
/* Button */
.btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  font-size: 0.875rem;
  font-weight: 600;
  border: none;
  border-radius: var(--radius-md);
  cursor: pointer;
  transition: all 0.15s ease;
  text-decoration: none;
  user-select: none;
}
.btn:hover { filter: brightness(1.1); transform: translateY(-1px); }
.btn:active { transform: translateY(0); filter: brightness(0.95); }
.btn:focus-visible { outline: 2px solid var(--color-brand-500); outline-offset: 2px; }

/* Card */
.card {
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
  overflow: hidden;
  transition: box-shadow 0.2s, transform 0.2s;
}
.card:hover {
  box-shadow: var(--shadow-lg);
  transform: translateY(-2px);
}

/* Input */
.input {
  width: 100%;
  padding: 10px 14px;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  font-size: 1rem;
  outline: none;
  transition: border-color 0.15s, box-shadow 0.15s;
}
.input:focus {
  border-color: var(--color-brand-500);
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.2);
}

/* Badge */
.badge {
  display: inline-flex;
  align-items: center;
  padding: 2px 10px;
  font-size: 0.75rem;
  font-weight: 600;
  border-radius: 9999px;
  background: var(--color-brand-50);
  color: var(--color-brand-500);
}

/* Divider */
.divider {
  border: none;
  border-top: 1px solid var(--color-border);
  margin: var(--space-6) 0;
}
```

---

## 19. Expert Tips & Tricks

### Logical Properties (RTL-friendly)

```css
/* Instead of left/right, use start/end */
margin-inline-start: 16px;    /* margin-left in LTR */
margin-inline-end: 16px;      /* margin-right in LTR */
padding-block-start: 24px;    /* padding-top */
padding-block-end: 24px;      /* padding-bottom */
border-inline-start: 2px solid blue;
inset-inline-start: 0;        /* left in LTR */
```

### Scroll Behavior & Snap

```css
/* Smooth scroll */
html { scroll-behavior: smooth; }

/* Scroll padding for fixed headers */
html { scroll-padding-top: 80px; }

/* Scroll snap */
.carousel {
  display: flex;
  overflow-x: scroll;
  scroll-snap-type: x mandatory;
}
.slide {
  flex: 0 0 100%;
  scroll-snap-align: start;
}
```

### CSS Grid Subgrid

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}
.card {
  grid-column: span 1;
  display: grid;
  grid-template-rows: subgrid; /* Aligns rows across cards! */
  grid-row: span 3;
}
```

### Container Queries (Modern!)

```css
/* Size based on parent container, not viewport */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card { flex-direction: row; }
}
```

### Aspect Ratio

```css
.video-wrapper {
  aspect-ratio: 16 / 9;   /* Maintains ratio */
  width: 100%;
}

.square { aspect-ratio: 1; }
.portrait { aspect-ratio: 3 / 4; }
```

### Pointer Events & User Select

```css
.overlay { pointer-events: none; }       /* Click-through */
.btn { pointer-events: all; }            /* Re-enable */
.no-select { user-select: none; }        /* Can't select text */
.select-all { user-select: all; }        /* Click = select all */
```

### Custom Scrollbar

```css
::-webkit-scrollbar { width: 8px; }
::-webkit-scrollbar-track { background: #f1f1f1; }
::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}
::-webkit-scrollbar-thumb:hover { background: #555; }

/* Firefox */
* { scrollbar-width: thin; scrollbar-color: #888 #f1f1f1; }
```

### Accent Color (Forms)

```css
:root { accent-color: #6366f1; }
/* Applies to: checkbox, radio, range, progress */
```

### CSS Nesting (Modern CSS)

```css
/* Without preprocessor! */
.card {
  background: white;
  padding: 16px;

  & .title {
    font-size: 1.25rem;
  }

  &:hover {
    box-shadow: var(--shadow-lg);
  }

  @media (min-width: 768px) {
    padding: 32px;
  }
}
```

### Print-friendly Utilities

```css
@media print {
  .no-print { display: none !important; }
  a[href]::after { content: " (" attr(href) ")"; }
  
  /* Avoid page breaks inside elements */
  .card { break-inside: avoid; }
  h2, h3 { break-after: avoid; }
}
```

### Performance Best Practices

```css
/* Tell browser what will animate */
.animated {
  will-change: transform, opacity; /* Use sparingly! */
}

/* GPU acceleration */
.smooth {
  transform: translateZ(0);  /* Forces GPU layer */
}

/* Prefer transform/opacity over top/left for animations */
/* ❌ Slow: */
.bad { transition: left 0.3s; }
/* ✅ Fast: */
.good { transition: transform 0.3s; }
```

### Centering Cheat Sheet

```css
/* Flexbox (most versatile) */
.parent { display: flex; justify-content: center; align-items: center; }

/* Grid */
.parent { display: grid; place-items: center; }

/* Absolute positioning */
.child { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }

/* Margin auto (horizontal only, must have width) */
.child { width: 500px; margin: 0 auto; }

/* Text centering */
.parent { text-align: center; line-height: [container-height]; }
```

### Debugging CSS

```css
/* Outline everything to see layout */
* { outline: 1px solid red; }

/* Debug specific element */
.suspect {
  outline: 2px solid lime;
  background: rgba(0, 255, 0, 0.1) !important;
}

/* Show invisible elements */
* { background: rgba(255, 0, 0, 0.1) !important; }
```

---

## Quick Reference Card

| Need | Property |
|------|----------|
| Center horizontally | `margin: 0 auto` or `display:flex; justify-content:center` |
| Center vertically | `display:flex; align-items:center` |
| Perfect center | `display:grid; place-items:center` |
| Full viewport height | `min-height: 100vh` |
| Remove default list styles | `list-style: none; padding: 0; margin: 0` |
| Hide element (no space) | `display: none` |
| Hide element (keeps space) | `visibility: hidden` or `opacity: 0` |
| Responsive image | `max-width: 100%; height: auto` |
| Circular element | `border-radius: 50%` (equal width/height) |
| Text doesn't wrap | `white-space: nowrap` |
| Force text to wrap | `word-break: break-all` |
| Blur background | `backdrop-filter: blur(10px)` |
| Custom cursor | `cursor: pointer/crosshair/none/url(...)` |
| Disable pointer events | `pointer-events: none` |
| Smooth scroll | `scroll-behavior: smooth` (on html) |
| Fix image to container | `object-fit: cover` |
| GPU animation | `will-change: transform` |
| Accessible hide | Use `.sr-only` class |

---

*CSS is a living language. Always check [MDN Web Docs](https://developer.mozilla.org) and [Can I Use](https://caniuse.com) for browser support. Experiment at [CodePen](https://codepen.io) or browser DevTools.*