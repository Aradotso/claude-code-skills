---
name: tufte-chart-design
description: Apply Edward Tufte's visualization principles to produce data-honest, high data-ink ratio charts with minimal chartjunk
triggers:
  - make a chart
  - create a visualization
  - design a dashboard
  - improve this chart
  - what chart should I use
  - apply Tufte principles
  - visualize this data
  - build a graph
---

# Tufte Chart Design Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill teaches you to create charts following Edward Tufte's principles from his three foundational books: *The Visual Display of Quantitative Information*, *Envisioning Information*, and *Visual Explanations*. It turns "make me a chart" into a Tufte-compliant visualization with maximum data-ink ratio and zero chartjunk.

## What This Skill Does

- **Activates automatically** when you're asked to create, improve, or critique any chart or visualization
- **Applies ten core principles** distilled from Tufte's work
- **Chooses the right chart type** based on data shape and analytical goal
- **Removes chartjunk** — 3D effects, unnecessary gridlines, redundant legends, pie charts, dual axes
- **Produces clean output** as inline SVG, React (Recharts), or D3-in-React
- **Enforces graphical integrity** — lie factor between 0.95 and 1.05, bars always start at zero

## The Ten Principles

1. **Show the data** — Above all else. Every decision asks "does this help see the data?"
2. **Maximize data-ink ratio** — Ink that changes when data changes is data-ink. Everything else is overhead.
3. **Erase non-data-ink** — Remove frame boxes, gridlines, tick marks at minor units.
4. **Erase redundant data-ink** — One encoding per quantity. Don't show "37" as a bar + number + gridline + tick.
5. **Graphical integrity** — Bars start at zero. Never use area for 1D quantities. Lie factor = (size of effect shown) / (size of effect in data).
6. **Small multiples** — N small charts on shared scales beats one chart with N overlapping lines.
7. **Layering and separation** — Data on top, labels next, scaffolding faintest. Avoid 1+1=3 visual noise.
8. **Micro/macro readings** — One shape from across the room, every point legible up close.
9. **Smallest effective difference** — Gray for context, one accent for focus. Subtle but clear.
10. **Word-data integration** — Sparklines in sentences. Numbers next to visuals. Tables are charts.

## When This Skill Activates

Auto-triggers on:
- "Make a chart of...", "visualize...", "graph...", "plot..."
- "Build a dashboard", "design a KPI tile"
- "Improve this chart", "critique this visual"
- "What chart should I use for...?"
- `/tufte` command or mentioning Tufte by name
- Any code editing session involving chart/visualization code

Does NOT trigger for:
- Logos, illustrations, decorative graphics
- Flowcharts, architecture diagrams, wireframes
- Pure typography or layout without quantitative data

## Chart Selection Guide

| Data Shape | Goal | Use | NOT |
|---|---|---|---|
| One time series | Show trend | Sparkline or line chart | Area chart, bar chart |
| Multiple time series | Compare trends | Small multiples (separate charts) | Multi-line spaghetti |
| Categories (≤10) | Compare values | Sorted dot plot or bar chart | Pie, donut, 3D bars |
| Categories (>10) | Show distribution | Strip plot or histogram | Rainbow color scales |
| Two time points | Show change | Slopegraph | Clustered bars |
| Part-to-whole | Show composition | Sorted table with percentages | Pie, donut, stacked bars |
| Process stages | Show flow | Horizontal bars + percentages | 3D funnel |
| Correlation | Show relationship | Scatterplot | Dual-axis line chart |
| Cohorts over time | Show retention | Sequential heatmap + sparklines | Rainbow heatmap |
| Geographic | Show regional pattern | Choropleth (single-hue sequential) | Rainbow choropleth |

## Kill List — Never Use

❌ **3D effects** — distort area perception, add no information  
❌ **Pie charts** — humans can't compare angles; use sorted table instead  
❌ **Donut charts** — same problem as pie, worse  
❌ **Dual-axis charts** — viewer can't tell if correlation is real or axis-manipulation  
❌ **Rainbow color scales** — perceptually non-uniform; use sequential single-hue  
❌ **Gradient fills** — decorative ink, not data-ink  
❌ **Drop shadows, glows** — chartjunk  
❌ **Chart frames/boxes** — erase, data should float  
❌ **Heavy gridlines** — if you need them, print values instead  
❌ **Redundant legends** — direct-label instead  

## HTML/SVG Output Pattern

When asked for inline SVG or given no preference, use this pattern:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      font-size: 11px;
      color: #1a1a1a;
      background: white;
      padding: 40px;
    }
    .label { font-size: 10px; fill: #666; }
    .value { font-size: 13px; fill: #1a1a1a; font-weight: 500; }
    .data-point { fill: #1a1a1a; }
    .data-line { stroke: #1a1a1a; stroke-width: 1.5; fill: none; }
    .reference-line { stroke: #e0e0e0; stroke-width: 1; }
    .accent { fill: #d73027; }
  </style>
</head>
<body>
  <svg width="600" height="200" viewBox="0 0 600 200">
    <!-- Data visualization here -->
  </svg>
</body>
</html>
```

### Sparkline Example

```html
<div style="font-family: sans-serif; padding: 20px;">
  <div style="font-size: 11px; color: #666; margin-bottom: 4px;">Monthly Active Users</div>
  <div style="display: flex; align-items: baseline; gap: 12px;">
    <span style="font-size: 24px; font-weight: 500; color: #1a1a1a;">247K</span>
    <svg width="120" height="24" style="display: block;">
      <polyline 
        points="0,18 10,15 20,16 30,14 40,12 50,13 60,10 70,9 80,11 90,8 100,6 110,4 120,3"
        stroke="#1a1a1a" 
        stroke-width="1.5" 
        fill="none"
      />
      <text x="0" y="24" font-size="9" fill="#999">Jan</text>
      <text x="110" y="24" font-size="9" fill="#999" text-anchor="end">Dec</text>
    </svg>
  </div>
</div>
```

### Dot Plot Example

```html
<svg width="400" height="180" viewBox="0 0 400 180">
  <!-- Sorted by value, descending -->
  <circle cx="320" cy="20" r="4" fill="#1a1a1a"/>
  <text x="330" y="24" font-size="11" fill="#1a1a1a">Product A</text>
  <text x="310" y="24" font-size="11" fill="#666" text-anchor="end">$247K</text>
  
  <circle cx="280" cy="45" r="4" fill="#1a1a1a"/>
  <text x="290" y="49" font-size="11" fill="#1a1a1a">Product B</text>
  <text x="270" y="49" font-size="11" fill="#666" text-anchor="end">$213K</text>
  
  <circle cx="190" cy="70" r="4" fill="#1a1a1a"/>
  <text x="200" y="74" font-size="11" fill="#1a1a1a">Product C</text>
  <text x="180" y="74" font-size="11" fill="#666" text-anchor="end">$145K</text>
  
  <!-- Scale reference at bottom -->
  <line x1="0" y1="160" x2="320" y2="160" stroke="#e0e0e0" stroke-width="1"/>
  <text x="0" y="175" font-size="9" fill="#999">$0</text>
  <text x="320" y="175" font-size="9" fill="#999" text-anchor="end">$250K</text>
</svg>
```

## React/Recharts Output Pattern

When asked for React or Recharts:

```jsx
import { LineChart, Line, ResponsiveContainer, XAxis, YAxis } from 'recharts';

const data = [
  { month: 'Jan', value: 180 },
  { month: 'Feb', value: 195 },
  { month: 'Mar', value: 205 },
  { month: 'Apr', value: 198 },
  { month: 'May', value: 215 },
  { month: 'Jun', value: 247 },
];

export default function RevenueChart() {
  return (
    <div style={{ fontFamily: 'system-ui, sans-serif' }}>
      <div style={{ fontSize: 11, color: '#666', marginBottom: 4 }}>
        Monthly Revenue (thousands)
      </div>
      <ResponsiveContainer width="100%" height={120}>
        <LineChart data={data} margin={{ top: 5, right: 5, bottom: 5, left: 5 }}>
          <XAxis 
            dataKey="month" 
            axisLine={false}
            tickLine={false}
            tick={{ fontSize: 10, fill: '#999' }}
            interval="preserveStartEnd"
          />
          <YAxis 
            hide={true}
          />
          <Line 
            type="monotone" 
            dataKey="value" 
            stroke="#1a1a1a" 
            strokeWidth={1.5}
            dot={false}
          />
        </LineChart>
      </ResponsiveContainer>
      <div style={{ fontSize: 20, fontWeight: 500, color: '#1a1a1a' }}>
        $247K
      </div>
    </div>
  );
}
```

### Recharts Theme Config

Apply these props to strip chartjunk:

```jsx
// Universal settings
const tufteTheme = {
  xAxis: {
    axisLine: false,
    tickLine: false,
    tick: { fontSize: 10, fill: '#999' },
  },
  yAxis: {
    axisLine: false,
    tickLine: false,
    tick: { fontSize: 10, fill: '#999' },
    // Often better to hide Y-axis and direct-label instead
  },
  grid: {
    strokeDasharray: '3 3',
    stroke: '#f0f0f0', // very faint if you must show
  },
  // NO tooltip by default — print values on chart
  // NO legend — direct label instead
};
```

## D3-in-React Pattern (for Custom Charts)

For slopegraphs, custom dot plots, small multiples:

```jsx
import { useEffect, useRef } from 'react';
import * as d3 from 'd3';

export default function Slopegraph({ data }) {
  const svgRef = useRef();
  
  useEffect(() => {
    const svg = d3.select(svgRef.current);
    const width = 400;
    const height = 300;
    const margin = { top: 20, right: 80, bottom: 20, left: 80 };
    
    // Clear previous render
    svg.selectAll('*').remove();
    
    const y = d3.scaleLinear()
      .domain([0, d3.max(data, d => Math.max(d.q2, d.q3))])
      .range([height - margin.bottom, margin.top]);
    
    // Draw lines
    svg.selectAll('line.slope')
      .data(data)
      .join('line')
      .attr('class', 'slope')
      .attr('x1', margin.left)
      .attr('x2', width - margin.right)
      .attr('y1', d => y(d.q2))
      .attr('y2', d => y(d.q3))
      .attr('stroke', d => d.q3 < d.q2 ? '#d73027' : '#ccc')
      .attr('stroke-width', 1.5);
    
    // Left labels + values
    svg.selectAll('text.left-label')
      .data(data)
      .join('text')
      .attr('class', 'left-label')
      .attr('x', margin.left - 8)
      .attr('y', d => y(d.q2))
      .attr('text-anchor', 'end')
      .attr('alignment-baseline', 'middle')
      .attr('font-size', 11)
      .attr('fill', '#1a1a1a')
      .text(d => `${d.team} ${d.q2}`);
    
    // Right values
    svg.selectAll('text.right-label')
      .data(data)
      .join('text')
      .attr('class', 'right-label')
      .attr('x', width - margin.right + 8)
      .attr('y', d => y(d.q3))
      .attr('text-anchor', 'start')
      .attr('alignment-baseline', 'middle')
      .attr('font-size', 11)
      .attr('fill', '#1a1a1a')
      .text(d => d.q3);
    
    // Column headers
    svg.append('text')
      .attr('x', margin.left)
      .attr('y', 10)
      .attr('text-anchor', 'middle')
      .attr('font-size', 10)
      .attr('fill', '#666')
      .text('Q2');
    
    svg.append('text')
      .attr('x', width - margin.right)
      .attr('y', 10)
      .attr('text-anchor', 'middle')
      .attr('font-size', 10)
      .attr('fill', '#666')
      .text('Q3');
      
  }, [data]);
  
  return <svg ref={svgRef} width={400} height={300} />;
}
```

## Small Multiples Pattern

When comparing multiple series, create separate charts on a shared scale instead of overlaying:

```jsx
const regions = ['North', 'South', 'East', 'West'];
const sharedYDomain = [0, 300]; // consistent scale across all

return (
  <div style={{ display: 'grid', gridTemplateColumns: 'repeat(2, 1fr)', gap: 20 }}>
    {regions.map(region => (
      <div key={region}>
        <div style={{ fontSize: 11, color: '#666', marginBottom: 4 }}>
          {region}
        </div>
        <ResponsiveContainer width="100%" height={100}>
          <LineChart data={data.filter(d => d.region === region)}>
            <YAxis domain={sharedYDomain} hide />
            <XAxis dataKey="month" tick={{ fontSize: 9 }} />
            <Line type="monotone" dataKey="value" stroke="#1a1a1a" strokeWidth={1.5} dot={false} />
          </LineChart>
        </ResponsiveContainer>
      </div>
    ))}
  </div>
);
```

## Dashboard KPI Tile Pattern

Replace giant number tiles with table + sparkline + target delta:

```jsx
const kpis = [
  { metric: 'Revenue', value: 247, trend: [180,195,205,198,215,247], target: 250, unit: 'K' },
  { metric: 'Users', value: 12.4, trend: [10.2,10.8,11.1,11.5,12.0,12.4], target: 12, unit: 'K' },
  { metric: 'Churn', value: 3.2, trend: [2.1,2.4,2.7,2.9,3.1,3.2], target: 2.5, unit: '%' },
];

return (
  <table style={{ fontFamily: 'sans-serif', fontSize: 11, borderCollapse: 'collapse' }}>
    <thead>
      <tr style={{ borderBottom: '1px solid #e0e0e0', textAlign: 'left' }}>
        <th style={{ padding: '8px 12px', fontWeight: 500 }}>Metric</th>
        <th style={{ padding: '8px 12px', fontWeight: 500 }}>Current</th>
        <th style={{ padding: '8px 12px', fontWeight: 500 }}>Trend</th>
        <th style={{ padding: '8px 12px', fontWeight: 500 }}>vs Target</th>
      </tr>
    </thead>
    <tbody>
      {kpis.map(kpi => {
        const delta = ((kpi.value - kpi.target) / kpi.target * 100).toFixed(0);
        const isBad = (kpi.metric === 'Churn' ? delta > 0 : delta < 0);
        return (
          <tr key={kpi.metric}>
            <td style={{ padding: '8px 12px' }}>{kpi.metric}</td>
            <td style={{ padding: '8px 12px', fontWeight: 500 }}>
              {kpi.value}{kpi.unit}
            </td>
            <td style={{ padding: '8px 12px' }}>
              <Sparkline data={kpi.trend} width={60} height={20} />
            </td>
            <td style={{ 
              padding: '8px 12px', 
              color: isBad ? '#d73027' : '#666',
              fontWeight: isBad ? 500 : 400
            }}>
              {delta > 0 ? '+' : ''}{delta}%
            </td>
          </tr>
        );
      })}
    </tbody>
  </table>
);
```

## Pre-Publish Checklist

Before declaring a chart done, verify:

- [ ] **Data-ink ratio > 0.7** — most ink changes when data changes
- [ ] **No chartjunk** — no 3D, gradients, shadows, decorative elements
- [ ] **Lie factor 0.95–1.05** — visual effect matches data effect
- [ ] **Bars start at zero** — if showing magnitude
- [ ] **Direct labels** — no legend unless unavoidable
- [ ] **Sorted by value** — categories ordered by size, not alphabet
- [ ] **One accent max** — gray for context, one color for focus
- [ ] **Readable at two scales** — shape visible from distance, values legible up close
- [ ] **No dual axis** — if tempted, make small multiples instead
- [ ] **Numbers printed** — if precision matters, show the value
- [ ] **Appropriate chart type** — matches data shape + analytical goal
- [ ] **All ink necessary** — tried removing every element; only kept what's essential

## Common Overrides

If a user explicitly requests something on the kill list:

> "Make a pie chart — the CFO wants a pie"

**Comply**, but add a comment noting the Tufte alternative:

```html
<!-- Pie chart as requested. Tufte alternative: sorted table with percentages. -->
<svg>...</svg>
```

User stays in control; you provide the alternative for reference.

## Color Rules

- **Default**: Grayscale. Black for data, mid-gray (#666) for labels, light-gray (#e0e0e0) for reference lines.
- **One accent**: Use red (#d73027) or blue (#4575b4) to highlight one series or flag an outlier.
- **Sequential scales**: Single-hue (light to dark). Never rainbow.
- **Diverging scales**: Two hues meeting at neutral (red-white-blue). Equal steps.
- **Categorical**: Only if categories have no natural order. Max 5 colors. Consider direct-label + grayscale instead.

## Troubleshooting

| Issue | Fix |
|---|---|
| "Chart feels cramped" | Increase padding, reduce font size, or use small multiples |
| "Can't read labels" | Direct-label at data points instead of axis; rotate if needed |
| "Too much white space" | Tufte prefers it — resist urge to fill; let data breathe |
| "Legend overlaps data" | Remove legend; direct-label each line/bar at its endpoint |
| "Gridlines requested" | Print values on chart instead; use faint reference line only at key thresholds |
| "Dual-axis requested" | Make two separate charts with shared X-axis; stack vertically |

## Configuration

No installation needed — this skill is already active in your environment.

**Override strictness per request**:
- Default: Apply Tufte principles unless user explicitly asks otherwise
- User can say "ignore Tufte for this one" or "CFO wants X" and you comply (with a note of the alternative)

**Stack preference** (auto-detected from user request):
- "inline SVG" or no preference → self-contained HTML/SVG
- "in React" or "with Recharts" → Recharts component
- "with D3" or "custom layout" → D3-in-React

## Reference

This skill distills:
- *The Visual Display of Quantitative Information* (1983, 2nd ed. 2001) — data honesty, data-ink ratio, lie factor, small multiples
- *Envisioning Information* (1990) — layering, micro/macro, escaping flatland
- *Visual Explanations* (1997) — smallest effective difference, narrative, visual confections

Core Tufte quote (VDQI p.105):
> Above all else show the data. Maximize the data-ink ratio. Erase non-data-ink. Erase redundant data-ink. Revise and edit.

---

**When in doubt**: Show the data. Remove everything else. Let the numbers speak.
