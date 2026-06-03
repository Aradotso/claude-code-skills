---
name: tufte-data-visualization
description: Generate Tufte-compliant charts and visualizations following principles from Edward Tufte's foundational books on data visualization
triggers:
  - make a chart
  - create a visualization
  - design a dashboard
  - apply Tufte principles
  - improve this chart
  - what chart should I use
  - visualize this data
  - create a data visualization
---

# Tufte Data Visualization Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

A Claude Code skill that transforms chart requests into Tufte-compliant visualizations distilled from Edward R. Tufte's three foundational books: *The Visual Display of Quantitative Information*, *Envisioning Information*, and *Visual Explanations*.

## Installation

```bash
git clone https://github.com/aref-vc/tufte-claude-skill.git ~/.claude/skills/tufte
```

The skill auto-loads on Claude Code's next launch. Verify with `/tufte` or by asking Claude to "make a chart."

Update:
```bash
cd ~/.claude/skills/tufte && git pull
```

Uninstall:
```bash
rm -rf ~/.claude/skills/tufte
```

## Core Principles

The skill applies ten principles from Tufte's work:

1. **Show the data** — every decision asks "does this help see the data?"
2. **Maximize data-ink ratio** — ink that changes when data changes is data-ink
3. **Erase non-data-ink** — remove frame boxes, gridlines, minor ticks
4. **Erase redundant data-ink** — one encoding per quantity
5. **Graphical integrity** — lie factor 0.95-1.05, bars start at zero
6. **Small multiples** — N charts on shared scale vs. one chart with N lines
7. **Layering and separation** — data on top, labels next, scaffolding faintest
8. **Micro/macro readings** — readable from afar and up close
9. **Smallest effective difference** — subtle distinctions, gray + one accent
10. **Word-data integration** — sparklines in sentences, numbers near visuals

## Chart Selection

The skill picks chart types based on your data structure and goal:

| Data shape | Goal | Chart type |
|------------|------|------------|
| 1 value over time | Show trend | Sparkline |
| 5-20 categories | Compare | Dot plot (sorted) |
| 20+ categories | Compare | Table with inline sparklines |
| Two time periods | Show change | Slopegraph |
| Part-to-whole | Show composition | Sorted table with % bars |
| Time series, multiple | Compare trends | Small multiples |
| Distribution | Show spread | Strip plot or quantile dot plot |
| Funnel/steps | Show drop-off | Horizontal bars (% of start) |
| Geographic | Show regional patterns | Choropleth (sequential single-hue) |
| Cohort/grid | Show patterns | Table with values + sequential color |

## Usage Patterns

### Basic Chart Generation

```javascript
// Request: "Make a chart of monthly revenue for the last 12 months"
// Output: Self-contained HTML with inline SVG sparkline

<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: "Gill Sans", sans-serif; margin: 40px; }
    .sparkline { display: inline-block; vertical-align: middle; margin: 0 8px; }
  </style>
</head>
<body>
  <div>
    <span style="font-size: 32px; font-weight: 600;">$847K</span>
    <svg class="sparkline" width="120" height="30" viewBox="0 0 120 30">
      <polyline 
        points="0,20 10,18 20,15 30,16 40,14 50,12 60,10 70,8 80,9 90,6 100,4 110,2"
        fill="none" 
        stroke="#555" 
        stroke-width="1.5"
      />
      <circle cx="0" cy="20" r="2" fill="#555" />
      <circle cx="110" cy="2" r="2" fill="#555" />
    </svg>
    <span style="color: #999; font-size: 14px;">Jan $620K → Dec $847K</span>
  </div>
</body>
</html>
```

### Dot Plot for Category Comparison

```html
<!-- Request: "Compare sales by product category" -->
<svg width="600" height="300" viewBox="0 0 600 300">
  <!-- Sorted by value, highest first -->
  <text x="10" y="30" font-size="14" fill="#333">Electronics</text>
  <circle cx="450" cy="26" r="4" fill="#333" />
  <text x="460" y="30" font-size="12" fill="#666">$2.4M</text>
  
  <text x="10" y="60" font-size="14" fill="#333">Furniture</text>
  <circle cx="380" cy="56" r="4" fill="#333" />
  <text x="390" y="60" font-size="12" fill="#666">$1.8M</text>
  
  <text x="10" y="90" font-size="14" fill="#333">Office Supplies</text>
  <circle cx="320" cy="86" r="4" fill="#333" />
  <text x="330" y="90" font-size="12" fill="#666">$1.3M</text>
  
  <!-- Light reference line at max -->
  <line x1="150" y1="0" x2="150" y2="300" stroke="#e0e0e0" stroke-width="1" />
</svg>
```

### Small Multiples for Time Series

```javascript
// Request: "Show revenue trend by region"
// Output: React component with Recharts

import { LineChart, Line, ResponsiveContainer } from 'recharts';

const data = {
  North: [12, 15, 14, 18, 20, 22, 24, 26, 25, 28, 30, 32],
  South: [8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19],
  East: [15, 16, 14, 15, 17, 18, 20, 22, 21, 23, 25, 27],
  West: [10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21]
};

export default function RegionalTrends() {
  return (
    <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '20px' }}>
      {Object.entries(data).map(([region, values]) => (
        <div key={region}>
          <div style={{ fontSize: '18px', fontWeight: 600, marginBottom: '8px' }}>
            {region}: ${values[values.length - 1]}K
          </div>
          <ResponsiveContainer width="100%" height={80}>
            <LineChart data={values.map((v, i) => ({ value: v }))}>
              <Line 
                type="monotone" 
                dataKey="value" 
                stroke="#555" 
                strokeWidth={1.5}
                dot={false}
              />
            </LineChart>
          </ResponsiveContainer>
        </div>
      ))}
    </div>
  );
}
```

### Slopegraph for Period Comparison

```html
<!-- Request: "Compare Q2 vs Q3 revenue by team" -->
<svg width="400" height="300" viewBox="0 0 400 300">
  <!-- Left labels (Q2) -->
  <text x="10" y="50" font-size="13" fill="#666">Q2</text>
  <text x="10" y="30" font-size="14" fill="#333">Sales: $120K</text>
  <text x="10" y="80" font-size="14" fill="#333">Marketing: $95K</text>
  <text x="10" y="130" font-size="14" fill="#333">Product: $85K</text>
  <text x="10" y="180" font-size="14" fill="#333">Support: $60K</text>
  
  <!-- Right labels (Q3) -->
  <text x="390" y="50" font-size="13" fill="#666" text-anchor="end">Q3</text>
  <text x="390" y="20" font-size="14" fill="#d73027" text-anchor="end">Sales: $145K</text>
  <text x="390" y="70" font-size="14" fill="#333" text-anchor="end">Marketing: $110K</text>
  <text x="390" y="120" font-size="14" fill="#333" text-anchor="end">Product: $98K</text>
  <text x="390" y="170" font-size="14" fill="#333" text-anchor="end">Support: $68K</text>
  
  <!-- Lines connecting periods (steepest in accent color) -->
  <line x1="140" y1="30" x2="250" y2="20" stroke="#d73027" stroke-width="2" />
  <line x1="140" y1="80" x2="250" y2="70" stroke="#999" stroke-width="1" />
  <line x1="140" y1="130" x2="250" y2="120" stroke="#999" stroke-width="1" />
  <line x1="140" y1="180" x2="250" y2="170" stroke="#999" stroke-width="1" />
</svg>
```

### Dashboard with KPI Table

```html
<!-- Request: "Build a dashboard for four key metrics" -->
<!DOCTYPE html>
<html>
<head>
  <style>
    table { border-collapse: collapse; font-family: "Gill Sans", sans-serif; }
    th { text-align: left; font-weight: 600; font-size: 13px; 
         color: #666; padding: 8px 12px; border-bottom: 1px solid #ddd; }
    td { padding: 8px 12px; font-size: 14px; }
    .metric { font-weight: 600; font-size: 18px; }
    .delta { font-size: 13px; color: #666; }
    .alert { color: #d73027; }
  </style>
</head>
<body>
  <table>
    <thead>
      <tr>
        <th>Metric</th>
        <th>Current</th>
        <th>10-month trend</th>
        <th>vs Target</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Revenue</td>
        <td class="metric">$2.4M</td>
        <td>
          <svg width="80" height="20">
            <polyline points="0,15 8,14 16,12 24,13 32,11 40,9 48,8 56,6 64,7 72,4" 
                      fill="none" stroke="#555" stroke-width="1.5" />
          </svg>
        </td>
        <td class="delta">+8%</td>
      </tr>
      <tr>
        <td>Active Users</td>
        <td class="metric">12.5K</td>
        <td>
          <svg width="80" height="20">
            <polyline points="0,12 8,11 16,10 24,11 32,12 40,13 48,14 56,15 64,16 72,17" 
                      fill="none" stroke="#d73027" stroke-width="1.5" />
          </svg>
        </td>
        <td class="delta alert">-12%</td>
      </tr>
      <tr>
        <td>Conversion</td>
        <td class="metric">3.2%</td>
        <td>
          <svg width="80" height="20">
            <polyline points="0,14 8,14 16,13 24,13 32,12 40,12 48,11 56,11 64,10 72,10" 
                      fill="none" stroke="#555" stroke-width="1.5" />
          </svg>
        </td>
        <td class="delta">+5%</td>
      </tr>
      <tr>
        <td>NPS</td>
        <td class="metric">67</td>
        <td>
          <svg width="80" height="20">
            <polyline points="0,10 8,10 16,11 24,10 32,9 40,9 48,8 56,8 64,7 72,7" 
                      fill="none" stroke="#555" stroke-width="1.5" />
          </svg>
        </td>
        <td class="delta">+2%</td>
      </tr>
    </tbody>
  </table>
</body>
</html>
```

### React Component with Tufte Theme

```jsx
// Request: "Create a time series chart in React"
import { LineChart, Line, ResponsiveContainer, XAxis, YAxis } from 'recharts';

const data = [
  { month: 'Jan', value: 620 },
  { month: 'Feb', value: 680 },
  { month: 'Mar', value: 720 },
  { month: 'Apr', value: 710 },
  { month: 'May', value: 750 },
  { month: 'Jun', value: 780 },
  { month: 'Jul', value: 800 },
  { month: 'Aug', value: 820 },
  { month: 'Sep', value: 810 },
  { month: 'Oct', value: 830 },
  { month: 'Nov', value: 840 },
  { month: 'Dec', value: 847 }
];

export default function RevenueChart() {
  return (
    <div style={{ fontFamily: '"Gill Sans", sans-serif' }}>
      <div style={{ fontSize: '24px', fontWeight: 600, marginBottom: '8px' }}>
        Monthly Revenue
      </div>
      <ResponsiveContainer width="100%" height={200}>
        <LineChart data={data} margin={{ top: 5, right: 30, left: 0, bottom: 5 }}>
          <XAxis 
            dataKey="month" 
            axisLine={false}
            tickLine={false}
            tick={{ fill: '#999', fontSize: 12 }}
          />
          <YAxis 
            axisLine={false}
            tickLine={false}
            tick={{ fill: '#999', fontSize: 12 }}
            tickFormatter={(v) => `$${v}K`}
          />
          <Line 
            type="monotone" 
            dataKey="value" 
            stroke="#333" 
            strokeWidth={1.5}
            dot={{ r: 2, fill: '#333' }}
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}
```

## Kill List

The skill automatically removes these elements:

- ❌ 3D effects, shadows, gradients
- ❌ Pie charts (use sorted tables instead)
- ❌ Dual-axis charts (make two charts)
- ❌ Rainbow color scales (use sequential single-hue)
- ❌ Frame boxes around charts
- ❌ Heavy gridlines (use faint reference lines sparingly)
- ❌ Redundant legends (direct label instead)
- ❌ Chart titles that repeat axis labels
- ❌ Decorative icons in dashboards
- ❌ Area encoding for 1D quantities

## Stack Options

Specify your preferred output format:

```bash
# Self-contained HTML/SVG (default)
"Make a chart as inline SVG"

# React with Recharts
"Create a chart in React"

# D3 in React (for custom layouts)
"Use D3 for a slopegraph"
```

## Override Defaults

Request non-Tufte elements when needed:

```bash
"Make a pie chart of revenue mix — the CFO wants a pie"
```

Claude will produce the requested chart but note the Tufte alternative in a comment.

## Critique Existing Charts

```bash
"Apply Tufte to this chart"
```

Paste chart code or describe the visualization. Claude will identify violations and produce a rewritten version.

## Configuration

No configuration needed. The skill activates automatically when you:

- Ask to create, make, design, or visualize a chart
- Request dashboard or KPI tiles
- Critique or improve existing visualizations
- Invoke `/tufte` directly

## Checklist

Before finalizing any chart, the skill verifies:

1. ✓ Data visible and readable
2. ✓ No frame box, minimal gridlines
3. ✓ Direct labels (no legend hunting)
4. ✓ Sorted by value (categories) or time
5. ✓ One encoding per quantity
6. ✓ Lie factor 0.95-1.05
7. ✓ Gray for context, one accent for focus
8. ✓ Values printed where needed
9. ✓ Works at distance and up close
10. ✓ Title adds information (not decoration)
11. ✓ Nothing on kill list
12. ✓ Would Tufte approve?

## Troubleshooting

**Q: Chart still has gridlines**  
A: Specify "remove all gridlines" or "apply strict Tufte rules"

**Q: Need a dual-axis chart**  
A: Request explicitly: "I need dual-axis despite Tufte's rule" — Claude will comply but suggest the alternative (small multiples on shared scale)

**Q: Color scale not working for heatmap**  
A: Request "sequential single-hue scale" — avoid rainbow (red-yellow-green)

**Q: Sparkline too small**  
A: Specify dimensions: "sparkline 150px wide, 40px tall"

**Q: Want to see the before/after examples**  
A: Open `~/.claude/skills/tufte/before-after.html` in your browser

## File Reference

- `principles.md` — Ten rules with Tufte quotes
- `chart-selection.md` — Data shape + goal → chart type
- `kill-list.md` — Elements to remove
- `checklist.md` — 12-item pre-publish check
- `presets/html-svg.md` — SVG templates and style tokens
- `presets/react.md` — Recharts theme + D3 patterns
- `cheatsheet.html` / `cheatsheet.pdf` — One-page reference

## Resources

Read the skill files for complete guidance:

```bash
~/.claude/skills/tufte/principles.md
~/.claude/skills/tufte/chart-selection.md
~/.claude/skills/tufte/kill-list.md
```

---

**License:** MIT  
**Source:** https://github.com/aref-vc/tufte-claude-skill
