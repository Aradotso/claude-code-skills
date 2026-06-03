---
name: tufte-chart-design
description: Turn "make me a chart" into a Tufte-compliant visualization following principles from Edward Tufte's foundational books on data visualization.
triggers:
  - "make a chart"
  - "create a visualization"
  - "design a dashboard"
  - "improve this chart"
  - "apply Tufte principles"
  - "what chart should I use"
  - "visualize this data"
  - "build a KPI tile"
---

# Tufte Chart Design Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

A Claude Code skill that transforms basic chart requests into Tufte-compliant visualizations following the ten core principles distilled from Edward R. Tufte's three foundational books: *The Visual Display of Quantitative Information*, *Envisioning Information*, and *Visual Explanations*.

## What This Skill Does

Automatically applies Edward Tufte's principles of excellence in statistical graphics to any chart, dashboard, or data visualization request. It:

- Maximizes data-ink ratio by removing chartjunk
- Enforces graphical integrity (lie factor 0.95-1.05)
- Selects optimal chart types based on data shape and communication goal
- Produces clean, self-documenting visualizations with minimal overhead
- Prevents common mistakes (3D effects, dual axes, pie charts, rainbow scales)

## Installation

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/aref-vc/tufte-claude-skill.git ~/.claude/skills/tufte
```

Claude Code auto-loads on next launch. Verify by running `/tufte` or asking to "make a chart of anything."

Update:
```bash
cd ~/.claude/skills/tufte && git pull
```

Uninstall:
```bash
rm -rf ~/.claude/skills/tufte
```

## The Ten Principles

1. **Show the data** — Above all else. Every decision asks "does this help see the data?"
2. **Maximize data-ink ratio** — Ink that changes when data changes is data-ink; minimize everything else
3. **Erase non-data-ink** — Drop frame boxes, gridlines, minor ticks
4. **Erase redundant data-ink** — One encoding per quantity (not bar + number + gridline + tick)
5. **Graphical integrity** — Bars start at zero, no area for 1D quantities, lie factor ~1.0
6. **Small multiples** — N small charts on shared scale instead of N lines on one chart
7. **Layering and separation** — Data on top, labels next, scaffolding faintest
8. **Micro/macro readings** — Works from across the room and up close
9. **Smallest effective difference** — Gray for context, one accent for focus
10. **Word-data integration** — Sparklines in sentences, numbers next to visuals

## Chart Selection Decision Table

The skill uses this mapping from data shape + goal to chart type:

| Data Shape | Goal | Chart Type |
|---|---|---|
| Categories + values | Compare | Sorted dot plot with direct labels |
| Time series (single) | Trend | Sparkline + endpoint labels + current value |
| Time series (multiple) | Compare trends | Small multiples of sparklines |
| Two time points | Change | Slopegraph (slope = delta) |
| Part-to-whole | Composition | Sorted table with share % and bars |
| Distribution | Shape | Strip plot with quartile markers |
| Funnel stages | Conversion | Horizontal bars sized by stage 1 share |
| Cohort over time | Retention pattern | Sequential heatmap + sparklines |
| Geographic | Regional comparison | Single-hue choropleth + top/bottom annotations |
| Correlation | Relationship | Scatterplot with marginal distributions |

## The Kill List

These are **always** removed unless explicitly overridden:

- 3D effects, shadows, gradients
- Pie charts (use sorted table)
- Dual-axis charts (use small multiples)
- Rainbow color scales (use sequential single-hue)
- Frame boxes around the chart
- Gridlines (except faint reference lines when essential)
- Decorative icons or illustrations
- Redundant legends when direct labels fit
- Chart titles that repeat the axis labels
- Default axis titles ("Value", "Category")

## Output Formats

### HTML/SVG (Default)

Self-contained, zero dependencies. Style tokens:

```html
<style>
  :root {
    --tufte-text: #111;
    --tufte-gray: #999;
    --tufte-line: #ddd;
    --tufte-accent: #e45;
    --tufte-font: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  }
  svg { font-family: var(--tufte-font); font-size: 12px; }
  .data-label { fill: var(--tufte-text); }
  .context-label { fill: var(--tufte-gray); font-size: 11px; }
  .gridline { stroke: var(--tufte-line); stroke-width: 1; }
  .data-point { fill: var(--tufte-text); }
  .accent { fill: var(--tufte-accent); }
</style>
```

**Sparkline template:**

```html
<svg width="120" height="30" viewBox="0 0 120 30">
  <polyline 
    points="0,15 20,18 40,12 60,20 80,8 100,14 120,10"
    fill="none" 
    stroke="#111" 
    stroke-width="1.5"
  />
  <text x="0" y="28" class="context-label">Jan</text>
  <text x="120" y="28" text-anchor="end" class="context-label">Dec</text>
</svg>
<span style="font-weight:600; margin-left:8px;">8.2M</span>
```

**Dot plot template:**

```html
<svg width="400" height="200" viewBox="0 0 400 200">
  <!-- Sorted by value descending -->
  <text x="0" y="20" class="data-label">Product A</text>
  <circle cx="120" cy="17" r="3" class="data-point"/>
  <text x="130" y="20" class="data-label">$420K</text>
  
  <text x="0" y="45" class="data-label">Product B</text>
  <circle cx="120" cy="42" r="3" class="data-point"/>
  <text x="130" y="45" class="data-label">$380K</text>
  
  <!-- Faint reference line at mean -->
  <line x1="110" y1="0" x2="110" y2="200" class="gridline" opacity="0.3"/>
</svg>
```

### React + Recharts

Theme override for Recharts:

```jsx
import { LineChart, Line, ResponsiveContainer } from 'recharts';

const tufteTheme = {
  cartesianGrid: { strokeDasharray: '0', stroke: '#ddd', strokeWidth: 0.5 },
  xAxis: { 
    axisLine: false, 
    tickLine: false,
    tick: { fill: '#999', fontSize: 11 }
  },
  yAxis: { 
    axisLine: false, 
    tickLine: false,
    tick: { fill: '#999', fontSize: 11 }
  },
};

export default function TrendChart({ data }) {
  return (
    <ResponsiveContainer width="100%" height={200}>
      <LineChart data={data} margin={{ top: 5, right: 20, left: 0, bottom: 5 }}>
        <XAxis dataKey="month" {...tufteTheme.xAxis} />
        <YAxis {...tufteTheme.yAxis} />
        <Line 
          type="monotone" 
          dataKey="value" 
          stroke="#111" 
          strokeWidth={1.5}
          dot={false}
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

**Slopegraph with D3 in React:**

```jsx
import { scaleLinear, scalePoint } from 'd3-scale';

export default function Slopegraph({ data }) {
  // data: [{ team: "Alpha", q2: 45, q3: 52 }, ...]
  const height = 300;
  const margin = { top: 20, right: 80, left: 80, bottom: 20 };
  
  const yScale = scaleLinear()
    .domain([0, Math.max(...data.flatMap(d => [d.q2, d.q3]))])
    .range([height - margin.bottom, margin.top]);
  
  const xScale = scalePoint()
    .domain(['Q2', 'Q3'])
    .range([margin.left, 400 - margin.right]);

  return (
    <svg width="400" height={height}>
      {data.map((d, i) => {
        const change = d.q3 - d.q2;
        const isNegative = change < 0;
        return (
          <g key={i}>
            <line
              x1={xScale('Q2')}
              y1={yScale(d.q2)}
              x2={xScale('Q3')}
              y2={yScale(d.q3)}
              stroke={isNegative ? '#e45' : '#ccc'}
              strokeWidth={isNegative ? 1.5 : 1}
            />
            <text x={xScale('Q2') - 5} y={yScale(d.q2) + 4} textAnchor="end" fontSize="11" fill="#999">
              {d.team}
            </text>
            <text x={xScale('Q2') + 5} y={yScale(d.q2) + 4} fontSize="12">{d.q2}</text>
            <text x={xScale('Q3') - 5} y={yScale(d.q3) + 4} textAnchor="end" fontSize="12">{d.q3}</text>
          </g>
        );
      })}
      <text x={xScale('Q2')} y={15} textAnchor="middle" fontSize="11" fill="#999">Q2</text>
      <text x={xScale('Q3')} y={15} textAnchor="middle" fontSize="11" fill="#999">Q3</text>
    </svg>
  );
}
```

## Common Patterns

### Dashboard KPI tiles

Convert big-number tiles to **table with sparklines:**

```html
<table style="border-collapse:collapse; font-family: -apple-system, sans-serif;">
  <thead>
    <tr style="border-bottom: 1px solid #ddd;">
      <th style="text-align:left; padding:8px; font-weight:500; color:#999;">Metric</th>
      <th style="text-align:right; padding:8px; font-weight:500; color:#999;">Current</th>
      <th style="padding:8px; font-weight:500; color:#999;">Trend (10 mo)</th>
      <th style="text-align:right; padding:8px; font-weight:500; color:#999;">vs Target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding:8px;">Revenue</td>
      <td style="padding:8px; text-align:right; font-weight:600;">$2.4M</td>
      <td style="padding:8px;">
        <svg width="80" height="24" viewBox="0 0 80 24">
          <polyline points="0,12 10,14 20,10 30,16 40,8 50,11 60,9 70,7 80,6" fill="none" stroke="#111" stroke-width="1.5"/>
        </svg>
      </td>
      <td style="padding:8px; text-align:right; color:#080;">+8%</td>
    </tr>
    <tr style="background:#fff5f5;">
      <td style="padding:8px;">Churn</td>
      <td style="padding:8px; text-align:right; font-weight:600;">4.2%</td>
      <td style="padding:8px;">
        <svg width="80" height="24" viewBox="0 0 80 24">
          <polyline points="0,8 10,9 20,10 30,11 40,12 50,13 60,15 70,16 80,18" fill="none" stroke="#e45" stroke-width="1.5"/>
        </svg>
      </td>
      <td style="padding:8px; text-align:right; color:#e45;">+1.2pp</td>
    </tr>
  </tbody>
</table>
```

### Small multiples for cohorts

```html
<div style="display:grid; grid-template-columns:repeat(4, 1fr); gap:16px; font-family:sans-serif;">
  <div>
    <div style="font-size:11px; color:#999; margin-bottom:4px;">Jan Cohort</div>
    <svg width="120" height="30" viewBox="0 0 120 30">
      <polyline points="0,5 20,8 40,12 60,16 80,18 100,20 120,22" fill="none" stroke="#111" stroke-width="1.5"/>
    </svg>
    <div style="font-weight:600; margin-top:4px;">68% at 6mo</div>
  </div>
  <div>
    <div style="font-size:11px; color:#999; margin-bottom:4px;">Feb Cohort</div>
    <svg width="120" height="30" viewBox="0 0 120 30">
      <polyline points="0,5 20,7 40,10 60,14 80,17 100,19 120,21" fill="none" stroke="#111" stroke-width="1.5"/>
    </svg>
    <div style="font-weight:600; margin-top:4px;">71% at 6mo</div>
  </div>
  <!-- ... more cohorts on same scale -->
</div>
```

### Before/After Check

When improving an existing chart, apply this checklist:

1. ✅ Bars start at zero (or use dot plot)
2. ✅ No 3D, shadows, gradients
3. ✅ No dual axis (split into two charts)
4. ✅ Sorted by value (unless time series)
5. ✅ Direct labels instead of legend when possible
6. ✅ Gray for context, one color for accent
7. ✅ Faint gridlines or none
8. ✅ No frame box
9. ✅ Lie factor = (graphic effect size / data effect size) ≈ 1.0
10. ✅ Readable at both overview and detail scale
11. ✅ Numbers printed for precise lookup
12. ✅ Title integrated (not floating above)

## Troubleshooting

**Problem:** Chart looks empty after applying Tufte principles  
**Solution:** You removed too much. Keep faint reference gridlines for scale, and ensure data points are visible (min 2px radius).

**Problem:** Client insists on pie chart  
**Solution:** Produce the pie, but include a comment noting the Tufte alternative:
```html
<!-- Tufte alternative: sorted table with bar segments showing share % -->
```

**Problem:** Too many categories for direct labels  
**Solution:** Use small multiples (one chart per category) or filter to top N + "Other."

**Problem:** Time series has huge variance, sparkline looks flat  
**Solution:** Use log scale or break into two panels (recent detail + full history context).

**Problem:** Dashboard needs to fit executive's "single glance" requirement  
**Solution:** Table with sparklines achieves this better than tiles. Show all KPIs in one table with trend + current value + delta.

## Usage Examples

### Request a chart

```
User: "Make a chart of monthly sales for the last 12 months"

Assistant produces:
- Sparkline (12 data points)
- Endpoint labels (Jan, Dec)
- Current value printed beside sparkline
- No frame, no gridlines, no legend
```

### Improve existing chart

```
User: "Apply Tufte to this bar chart" [pastes code]
