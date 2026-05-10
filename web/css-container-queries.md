# CSS Container Queries: Responsive Components, Not Just Viewports

Container queries let components respond to their *parent's* size instead of the viewport. This is the biggest CSS feature since Flexbox. Supported in all modern browsers since Feb 2023.

## The Problem with Media Queries

Media queries only know about the viewport. A card component at 300px width in a sidebar looks identical to one at 300px in a full-width layout — but they shouldn't.

## Container Queries: The Fix

### Step 1: Define a containment context

```css
/* The PARENT declares itself as a container */
.sidebar {
  container-type: inline-size;
  container-name: sidebar;
}

/* Shorthand */
.card-grid {
  container: card-grid / inline-size;
}
```

### Step 2: Query the container

```css
/* Card adapts based on its container's width, not the viewport */
.card {
  display: grid;
  grid-template-columns: 200px 1fr;
  gap: 1rem;
}

@container (max-width: 400px) {
  .card {
    grid-template-columns: 1fr;
  }

  .card img {
    aspect-ratio: 16 / 9;
    width: 100%;
  }
}

/* Named container queries — target specific ancestors */
@container sidebar (max-width: 300px) {
  .card {
    font-size: 0.875rem;
  }
}
```

## Container Query Units

New units relative to the container dimensions:

| Unit | Meaning |
|------|---------|
| `cqw` | 1% of container width |
| `cqh` | 1% of container height |
| `cqi` | 1% of container inline size |
| `cqb` | 1% of container block size |
| `cqmin` | Smaller of cqi/cqb |
| `cqmax` | Larger of cqi/cqb |

```css
/* Fluid typography relative to container, not viewport */
.card-title {
  font-size: clamp(1rem, 3cqi, 1.75rem);
}

/* Spacing that scales with container */
.card-body {
  padding: 2cqi;
}
```

## Real-World Example: Dashboard Widgets

```html
<div class="dashboard">
  <div class="widget large">
    <div class="chart-card">...</div>
  </div>
  <div class="widget small">
    <div class="chart-card">...</div>
  </div>
</div>
```

```css
.widget {
  container: widget / inline-size;
}

.chart-card {
  display: grid;
  grid-template-columns: 1fr 1fr;
}

@container widget (max-width: 350px) {
  .chart-card {
    grid-template-columns: 1fr;
  }

  .chart-card .legend {
    display: none;
  }

  .chart-card .value {
    font-size: 2rem;
    text-align: center;
  }
}

@container widget (min-width: 600px) {
  .chart-card {
    grid-template-columns: 2fr 1fr 1fr;
  }
}
```

## Style Container Queries (Newer)

Query computed styles of ancestors (Chrome 111+):

```css
.theme-wrapper {
  container-name: theme;
}

@container style(--theme: dark) {
  .card {
    background: #1a1a2e;
    color: #eee;
  }
}
```

## Key Takeaway

Start using `container-type: inline-size` on layout wrappers (sidebars, grids, widget containers) today. Write component CSS with `@container` instead of `@media`. Your components become truly reusable across different layout contexts.

## Browser Support

Chrome 105+ | Firefox 110+ | Safari 16+ | Edge 105+
Style queries: Chrome 111+ (other browsers implementing)