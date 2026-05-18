# htmx Progressive Enhancement

## Infinite Scroll
```html
<div hx-get="/feed?page=2" hx-trigger="revealed" hx-swap="afterend">
  Loading...
</div>
```

## Inline Editing
```html
<span hx-get="/edit/123" hx-trigger="dblclick" hx-swap="outerHTML">
  Click to edit
</span>
```

## Search with Debounce
```html
<input type="search" hx-get="/search" hx-trigger="keyup changed delay:300ms"
       hx-target="#results" name="q">
```

## When to use htmx
- Server-rendered apps needing interactivity
- SEO-critical pages
- Reducing JS bundle (htmx = 14kb gzipped)
- Teams strong in backend, not React/Vue
