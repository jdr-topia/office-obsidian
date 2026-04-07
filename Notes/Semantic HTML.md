---
created: 2026-04-07
tags: [accessibility, html, a11y]
related: [ [[Accessibility (a11y) MOC]], [[ARIA attributes]] ]
status: active
---

# Semantic HTML

Semantic HTML means using HTML elements that carry inherent meaning about the content they contain. The browser and assistive technologies (screen readers) use this meaning to build the accessibility tree — a parallel representation of the DOM used by screen readers.

## Why Semantics Matter

A `<div>` has no meaning. A `<button>` tells the browser:
- This is interactive.
- It can receive keyboard focus.
- It fires a `click` event on `Space`/`Enter`.
- It exposes a `role="button"` to the accessibility tree.

Using a `<div onClick={...}>` instead of a `<button>` loses all of that automatically — and you have to recreate it manually.

---

## Common Semantic Elements

### Document Structure
```html
<header>   <!-- Site or section header -->
<nav>      <!-- Navigation links -->
<main>     <!-- Primary content (one per page) -->
<aside>    <!-- Secondary/supplemental content -->
<footer>   <!-- Site or section footer -->
<section>  <!-- Thematic grouping with a heading -->
<article>  <!-- Self-contained content (blog post, card) -->
```

### Content
```html
<h1> - <h6>  <!-- Heading hierarchy — NEVER skip levels -->
<p>          <!-- A paragraph -->
<ul> / <ol>  <!-- Unordered / Ordered lists -->
<figure> + <figcaption>  <!-- Image with caption -->
<time datetime="2024-01-15">January 15</time>  <!-- Machine-readable dates -->
<address>    <!-- Contact information -->
```

### Interactive
```html
<button>   <!-- Any clickable action -->
<a href=""><!-- Navigation to a URL -->
<input>    <!-- Form data entry -->
<label>    <!-- Associates text with an input -->
<select>   <!-- Dropdown selection -->
```

---

## The Golden Rule

> **Use the most specific semantic element available before reaching for `<div>` or `<span>`.**

### Anti-Patterns to Avoid

```html
<!-- ❌ Bad: div acting as a button -->
<div onClick={handleSubmit}>Submit</div>

<!-- ✅ Good: Actual button -->
<button type="submit" onClick={handleSubmit}>Submit</button>

<!-- ❌ Bad: Skipping heading levels -->
<h1>Page Title</h1>
<h3>Section</h3>  <!-- Skipped h2 -->

<!-- ✅ Good: Logical hierarchy -->
<h1>Page Title</h1>
<h2>Section</h2>
<h3>Sub-section</h3>

<!-- ❌ Bad: Link that acts like a button -->
<a href="#" onClick={openModal}>Open Modal</a>

<!-- ✅ Good -->
<button onClick={openModal}>Open Modal</button>
```

---

## Images & Alt Text

```html
<!-- Informative image: describe the content -->
<img src="chart.png" alt="Bar chart showing 40% growth in Q1 2024" />

<!-- Decorative image: empty alt so screen readers skip it -->
<img src="divider.png" alt="" role="presentation" />

<!-- Complex image: use figcaption for long descriptions -->
<figure>
  <img src="architecture.png" alt="Frontend architecture diagram" />
  <figcaption>
    The diagram shows the flow from client to CDN to API gateway.
  </figcaption>
</figure>
```

---

## Related Topics
- [[Accessibility (a11y) MOC]]
- [[ARIA attributes]]
- [[Keyboard navigation and focus management]]
