```markdown
---
name: tufte-chart-design
description: Apply Edward Tufte's principles to create clear, honest, high data-ink ratio charts
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

# tufte-chart-design

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

A Claude Code skill that transforms "make me a chart" into a Tufte-compliant visualization. Distilled from Edward R. Tufte's three foundational books: *The Visual Display of Quantitative Information*, *Envisioning Information*, and *Visual Explanations*.

## What This Skill Does

Automatically applies ten core principles of information design to any chart request:

1. **Show the data** — prioritize the actual data over decoration
2. **Maximize data-ink ratio** — only ink that changes with data is essential
3. **Erase non-data-ink** — remove frames, gridlines, unnecessary ticks
4. **Erase redundant data-ink** — one encoding per quantity
5. **Graphical integrity** — lie factor 0.95–1.05, bars start at zero
6. **Small multiples** — N small charts on shared scale vs one chart with N lines
7. **Layering and separation** — data on top, labels next, scaffolding faintest
8. **Micro/macro readings** — readable from afar and up close
9. **Smallest effective difference** — gray for context, one accent for focus
10. **Word-data integration** — sparklines in sentences, numbers next to visuals

## Installation

This skill is already installed in your Claude Code environment at `~/.claude/skills/tufte/`.

To verify installation:
```bash
ls ~/.claude/skills/tufte/
```

You should see: `SKILL.md`, `principles.md`, `chart-selection.md`, `kill-list.md`, `checklist.md`, and `presets/`.

## Core Files Reference

| File | Purpose |
|------|---------|
| `principles.md` | The ten rules with Tufte quotes |
| `chart-selection.md` | Data shape + goal → chart type decision table |
| `kill-list.md` | Elements to remove (3D, pie, dual-axis, rainbow, etc.) |
| `checklist.md` | 12-item pre-publish validation |
| `presets/html-svg.md` | Self-contained HTML/SVG templates |
| `presets/react.md` | Recharts theme + D3-in-React patterns |

## Activation

The skill auto-activates when you:
- Ask to make, create, design, or visualize a chart
- Request a dashboard or KPI tile
- Ask to critique or improve an existing chart
- Ask "what chart should I use for..."
- Invoke `/tufte` directly

## Chart Selection Logic

The skill uses this decision table from `chart-selection.md`:

| Data shape | Goal | Chart type |
|------------|------|------------|
| 5–20 categories + 1 value | Compare | Sorted dot plot with direct labels |
| Time series, 1 metric | Show trend | Sparkline + current value + endpoints |
| Time series, 2–4 metrics | Compare trends | Small multiples (4 sparklines stacked) |
| Time series, 5+ metrics | Show pattern | Indexed line chart (all start at 100) |
| 2 time points, 5–12 items | Show change | Slopegraph |
| Stages with drop-off | Show conversion | Horizontal bars (% of stage 1) + retention labels |
| Part-to-whole, <7 parts | Show composition | Sorted table with % bar |
| Distribution, <100 points | Show spread | Strip plot + quartile markers |
| Distribution, 100+ points | Show shape | Histogram (not rainbow) + mean/median lines |
| Geographic, sequential | Show magnitude | Single-hue choropleth + annotations |
| Cohorts × time | Show retention | Heatmap (sequential hue) + sparklines |

## Output Formats

### HTML/SVG (default)

Self-contained, no dependencies. Uses templates from `presets/html-svg.md`.

**Example: Sparkline**
```html
<div style="font-family: 'Gill Sans', sans-serif; font-size: 14px; color: #333;">
  Revenue: <b style="font-size: 18px;">$847k</b>
  <svg width="80" height="20" style="vertical-align: middle; margin: 0 8px;">
    <polyline 
      points="0,15 10,12 20,13 30,10 40,8 50,9 60,5 70,3 80,2" 
      fill="none" 
      stroke="#333" 
      stroke-width="1.5"/>
  </svg>
  <span style="color: #999; font-size: 12px;">Jan</span>
  <span style="color: #999; font-size: 12px; margin-left: 4px;">Dec</span>
</div>
```

**Example: Dot plot**
```html
<svg width="400" height="200" xmlns="http://www.w3.org/2000/svg">
  <style>
    text { font-family: 'Gill Sans', sans-serif; font-size: 13px; fill: #333; }
    .label { text-anchor: end; }
    .value { text-anchor: start; fill: #666; }
  </style>
  
  <!-- Product C: 847 -->
  <text x="140" y="25" class="label">Product C</text>
  <circle cx="150" cy="22" r="3" fill="#333"/>
  <line x1="150" y1="22" x2="297" y2="22" stroke="#ddd" stroke-width="1"/>
  <text x="302" y="25" class="value">847</text>
  
  <!-- Product A: 623 -->
  <text x="140" y="55" class="label">Product A</text>
  <circle cx="150" cy="52" r="3" fill="#333"/>
  <line x1="150" y1="52" x2="258" y2="52" stroke="#ddd" stroke-width="1"/>
  <text x="263" y="55" class="value">623</text>
  
  <!-- Product B: 412 -->
  <text x="140" y="85" class="label">Product B</text>
  <circle cx="150" cy="82" r="3" fill="#333"/>
  <line x1="150" y1="82" x2="221" y2="82" stroke="#ddd" stroke-width="1"/>
  <text x="226" y="85" class="value">412</text>
</svg>
```

### React with Recharts

Minimal theme, overrides defaults to match Tufte principles.

**Example: Sparkline component**
```jsx
import { LineChart, Line, ResponsiveContainer } from 'recharts';

const data = [
  {month: 'Jan', value: 734},
  {month: 'Feb', value: 782},
  {month: 'Mar', value: 761},
  {month: 'Apr', value: 805},
  {month: 'May', value: 823},
  {month: 'Jun', value: 847}
];

export default function RevenueSparkline() {
  return (
    <div style={{fontFamily: 'Gill Sans, sans-serif', fontSize: 14}}>
      Revenue: <strong style={{fontSize: 18}}>$847k</strong>
      <ResponsiveContainer width={80} height={20} style={{display: 'inline-block', margin: '0 8px', verticalAlign: 'middle'}}>
        <LineChart data={data}>
          <Line 
            type="monotone" 
            dataKey="value" 
            stroke="#333" 
            strokeWidth={1.5} 
            dot={false}
            isAnimationActive={false}
          />
        </LineChart>
      </ResponsiveContainer>
      <span style={{color: '#999', fontSize: 12}}>Jan</span>
      <span style={{color: '#999', fontSize: 12, marginLeft: 4}}>Dec</span>
    </div>
  );
}
```

**Example: Tufte theme for Recharts**
```jsx
// Apply to any Recharts chart
const tufteTheme = {
  // Remove all chrome
  cartesianGrid: false,
  
  // Minimal axes
  xAxis: {
    axisLine: false,
    tickLine: false,
    tick: { fill: '#666', fontSize: 12, fontFamily: 'Gill Sans, sans-serif' }
  },
  
  yAxis: {
    axisLine: false,
    tickLine: false,
    tick: { fill: '#666', fontSize: 12, fontFamily: 'Gill Sans, sans-serif' }
  },
  
  // No legend by default (use direct labels)
  legend: false,
  
  // Single accent color
  colors: ['#333', '#999', '#ccc']
};

// Usage
<BarChart data={data}>
  <XAxis dataKey="name" {...tufteTheme.xAxis} />
  <YAxis {...tufteTheme.yAxis} />
  <Bar dataKey="value" fill="#333" />
</BarChart>
```

### D3 in React (for custom charts)

Use when Recharts can't produce the pattern (slopegraph, small multiples grid, custom annotations).

**Example: Slopegraph**
```jsx
import { useRef, useEffect } from 'react';
import * as d3 from 'd3';

const data = [
  {team: 'Engineering', q2: 847, q3: 923},
  {team: 'Sales', q2: 734, q3: 782},
  {team: 'Marketing', q2: 623, q3: 601}, // Only decliner
  {team: 'Support', q2: 512, q3: 534}
];

export default function TeamSlopegraph() {
  const svgRef = useRef();
  
  useEffect(() => {
    const svg = d3.select(svgRef.current);
    const width = 300;
    const height = 200;
    const margin = {top: 20, right: 80, bottom: 20, left: 80};
    
    const y = d3.scaleLinear()
      .domain([0, d3.max(data, d => Math.max(d.q2, d.q3))])
      .range([height - margin.bottom, margin.top]);
    
    svg.selectAll('*').remove();
    
    // Lines
    data.forEach(d => {
      const color = d.q3 < d.q2 ? '#c00' : '#333'; // Red for decline
      
      svg.append('line')
        .attr('x1', margin.left)
        .attr('y1', y(d.q2))
        .attr('x2', width - margin.right)
        .attr('y2', y(d.q3))
        .attr('stroke', color)
        .attr('stroke-width', 1.5);
      
      // Q2 label
      svg.append('text')
        .attr('x', margin.left - 5)
        .attr('y', y(d.q2) + 4)
        .attr('text-anchor', 'end')
        .attr('font-size', 12)
        .attr('fill', '#666')
        .text(d.q2);
      
      // Q3 label
      svg.append('text')
        .attr('x', width - margin.right + 5)
        .attr('y', y(d.q3) + 4)
        .attr('text-anchor', 'start')
        .attr('font-size', 12)
        .attr('fill', color)
        .text(d.q3);
      
      // Team name (at Q3 end)
      svg.append('text')
        .attr('x', width - margin.right + 35)
        .attr('y', y(d.q3) + 4)
        .attr('text-anchor', 'start')
        .attr('font-size', 12)
        .attr('fill', '#333')
        .text(d.team);
    });
    
    // Period labels
    svg.append('text')
      .attr('x', margin.left)
      .attr('y', 12)
      .attr('text-anchor', 'middle')
      .attr('font-size', 11)
      .attr('fill', '#999')
      .text('Q2');
    
    svg.append('text')
      .attr('x', width - margin.right)
      .attr('y', 12)
      .attr('text-anchor', 'middle')
      .attr('font-size', 11)
      .attr('fill', '#999')
      .text('Q3');
      
  }, []);
  
  return <svg ref={svgRef} width={300} height={200} style={{fontFamily: 'Gill Sans, sans-serif'}} />;
}
```

## Kill List (Always Remove)

From `kill-list.md`:

- ❌ 3D effects, shadows, gradients
- ❌ Pie charts (use sorted table instead)
- ❌ Dual-axis charts (use small multiples or indexing)
- ❌ Rainbow color scales (use sequential single hue)
- ❌ Frame boxes around charts
- ❌ Heavy gridlines (use faint lines or none)
- ❌ Legends when direct labels fit
- ❌ Redundant axis titles ("Revenue" label + "Revenue ($)" axis title)
- ❌ Chart titles that repeat the surrounding text
- ❌ Decorative icons in dashboards
- ❌ Animated transitions longer than 200ms
- ❌ More than two fonts or four colors

## Pre-Publish Checklist

From `checklist.md` — run before declaring a chart done:

- [ ] Bars start at zero (or use dot plot instead)
- [ ] Lie factor between 0.95 and 1.05
- [ ] Data-ink ratio > 0.5 (more than half the ink is data)
- [ ] No frame box
- [ ] No or faint gridlines
- [ ] Labels are direct (no legend lookup required)
- [ ] Sorted by value (not alphabetically) when order isn't time
- [ ] One accent color max, rest in gray
- [ ] Readable at arm's length and up close
- [ ] Units in context ("$847k" not "847" with a "$1000s" note)
- [ ] Annotations for the insight, not decoration
- [ ] Would pass the "print it in black and white" test

## Common Patterns

### Pattern: Dashboard with sparklines in table

```html
<table style="font-family: 'Gill Sans', sans-serif; border-collapse: collapse; font-size: 14px;">
  <thead>
    <tr style="border-bottom: 1px solid #333;">
      <th style="text-align: left; padding: 8px;">Metric</th>
      <th style="text-align: right; padding: 8px;">Current</th>
      <th style="padding: 8px;">Trend (12mo)</th>
      <th style="text-align: right; padding: 8px; color: #666;">vs Target</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="padding: 8px;">Revenue</td>
      <td style="text-align: right; padding: 8px;">$847k</td>
      <td style="padding: 8px;">
        <svg width="60" height="20">
          <polyline points="0,15 10,12 20,13 30,10 40,8 50,9 60,5" fill="none" stroke="#333" stroke-width="1.5"/>
        </svg>
      </td>
      <td style="text-align: right; padding: 8px; color: #060;">+6%</td>
    </tr>
    <tr>
      <td style="padding: 8px;">Churn</td>
      <td style="text-align: right; padding: 8px;">4.2%</td>
      <td style="padding: 8px;">
        <svg width="60" height="20">
          <polyline points="0,5 10,7 20,6 30,9 40,11 50,10 60,14" fill="none" stroke="#c00" stroke-width="1.5"/>
        </svg>
      </td>
      <td style="text-align: right; padding: 8px; color: #c00;">+1.7pp</td>
    </tr>
  </tbody>
</table>
```

### Pattern: Small multiples (four regions, shared scale)

```html
<div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 16px; font-family: 'Gill Sans', sans-serif;">
  <div>
    <div style="font-size: 12px; color: #666; margin-bottom: 4px;">West: $1.2M</div>
    <svg width="180" height="40">
      <polyline points="0,30 20,25 40,26 60,22 80,18 100,19 120,14 140,10 160,8 180,7" fill="none" stroke="#333" stroke-width="1.5"/>
    </svg>
  </div>
  <div>
    <div style="font-size: 12px; color: #666; margin-bottom: 4px;">East: $980k</div>
    <svg width="180" height="40">
      <polyline points="0,32 20,30 40,28 60,26 80,24 100,23 120,20 140,18 160,17 180,15" fill="none" stroke="#333" stroke-width="1.5"/>
    </svg>
  </div>
  <div>
    <div style="font-size: 12px; color: #666; margin-bottom: 4px;">Central: $760k</div>
    <svg width="180" height="40">
      <polyline points="0,35 20,34 40,36 60,33 80,32 100,30 120,28 140,27 160,25 180,23" fill="none" stroke="#333" stroke-width="1.5"/>
    </svg>
  </div>
  <div>
    <div style="font-size: 12px; color: #666; margin-bottom: 4px;">South: $540k</div>
    <svg width="180" height="40">
      <polyline points="0,37 20,38 40,36 60,35 80,34 100,36 120,33 140,32 160,30 180,29" fill="none" stroke="#333" stroke-width="1.5"/>
    </svg>
  </div>
</div>
```

### Pattern: Conversion funnel (horizontal bars)

```html
<svg width="400" height="120" xmlns="http://www.w3.org/2000/svg">
  <style>
    text { font-family: 'Gill Sans', sans-serif; font-size: 13px; fill: #333; }
  </style>
  
  <!-- Visited: 100% -->
  <text x="0" y="18">Visited</text>
  <rect x="80" y="5" width="300" height="16" fill="#ddd"/>
  <text x="385" y="18" text-anchor="end">10,000</text>
  
  <!-- Signed up: 42% of visited -->
  <text x="0" y="48">Signed up</text>
  <rect x="80" y="35" width="126" height="16" fill="#999"/>
  <text x="211" y="48" text-anchor="start" fill="#666" font-size="11"> 42%</text>
  <text x="385" y="48" text-anchor="end">4,200</text>
  
  <!-- Activated: 68% of signup -->
  <text x="0" y="78">Activated</text>
  <rect x="80" y="65" width="86" height="16" fill="#999"/>
  <text x="171" y="78" text-anchor="start" fill="#666" font-size="11"> 68%</text>
  <text x="385" y="78" text-anchor="end">2,856</text>
  
  <!-- Paid: 31% of activated ⚠️ -->
  <text x="0" y="108">Paid</text>
  <rect x="80" y="95" width="27" height="16" fill="#c00"/>
  <text x="112" y="108" text-anchor="start" fill="#c00" font-size="11"> 31% ⚠️</text>
  <text x="385" y="108" text-anchor="end">885</text>
</svg>
```

## Troubleshooting

### "The chart looks plain"

That's the goal. Tufte: "Clutter and confusion are failures of design, not attributes of information." If the data is interesting, plainness lets it speak.

### "Can I use color for categories?"

Only if the categories are few (≤4) and color adds meaning. Otherwise, use direct labels and one color. From `principles.md`: "Color should be reserved for emphasis and differentiation, not decoration."

### "What if I need to show two metrics on one chart?"

- If they share a scale: one chart, two colors (data + gray context)
- If scales differ: small multiples (two charts stacked, shared x-axis)
- Never dual-axis — from `kill-list.md`: "The human eye cannot accurately compare two different y-scales in the same viewport."

### "The client wants 3D bars"

Override allowed. Explain the alternative in a comment:

```html
<!-- 
  Client request: 3D bars
  Tufte alternative: sorted dot plot (ranks in 2 seconds vs decode time for 3D depth cues)
-->
```

### "How do I show uncertainty?"

- Confidence interval: faint gray band behind the line
- Error bars: only if they don't overlap (otherwise use gradient from min to max)
- Annotations: "±3% margin" in one note, not repeated per data point

### "Can I use a pie chart for percentages?"

No. From `kill-list.md`: "The human eye is poor at comparing angles and areas. A sorted table with a % bar is faster and more accurate."

```html
<!-- Instead of pie: -->
<table>
  <tr><td>Product C</td><td style="text-align: right;">42%</td><td><div style="width: 84px; height: 8px; background: #333;"></div></td></tr>
  <tr><td>Product A</td><td style="text-align: right;">31%</td><td><div style="width: 62px; height: 8px; background: #333;"></div></td></tr>
  <tr><td>Product B</td><td style="text-align: right;">27%</td><td><div style="width: 54px; height: 8px; background: #333;"></div></td></tr>
</table>
```

## Configuration

No configuration file. The skill applies principles by default. To override per-chart:

- "I need a pie because..." → produces pie + comment with alternative
- "Ignore Tufte for this one" → produces what you ask for, no enforcement
- "Use brand colors: #ff0000, #00ff00" → uses your palette but keeps other principles

## Advanced Usage

### Custom annotations

Annotations should explain, not label. Good: "Largest drop in 5 years." Bad: "Q3."

```html
<svg width="400" height="200">
  <!-- Line chart here -->
  
  <!-- Annotation for the insight -->
  <circle cx="240" cy="120" r="3" fill="#c00"/>
  <text x="245" y="115" font-size="11" fill="#c00">Largest drop in 5 years</text>
  <line x1="240" y1="120" x2="240" y2="100" stroke="#c00" stroke-width="0.5" stroke-dasharray="2,2"/>
</svg>
```

### Layering (micro/macro)

Data on top (black, sharp), scaffolding behind (gray, faint).

```html
<svg>
  <!-- Gridlines: faint, behind -->
  <line x1="0" y1="50" x2="400" y2="50" stroke="#eee" stroke-width="0.5"/>
  
  <!-- Data: dark, in front -->
  <polyline points="..." fill="none" stroke="#333" stroke-width="2"/>
  
  <!-- Labels: medium weight, between -->
  <text fill="#666" font-size="12">...</text>
</svg>
```

### Sparkline in prose

From `principles.md` principle 10: "The best graphics are integrated with text."

```html
<p style="font-family: 'Gill Sans', sans-serif; font-size: 14px; line-height: 1.6;">
  Revenue grew 18% in Q4 
  <svg width="40" height="14" style="vertical-align: middle;">
    <polyline points="0,10 10,8 20,9 30,5 40,3" fill="none" stroke="#333" stroke-width="1"/>
  </svg>
  driven by enterprise accounts.
</p>
```

## References

- Edward R. Tufte, *The Visual Display of Quantitative Information* (1983, 2nd ed. 2001)
- Edward R. Tufte, *Envisioning Information* (1990)
- Edward R. Tufte, *Visual Explanations* (1997)

All quotations in `principles.md` are attributed and used under fair-use for educational reference.

---

**Remember:** Above all else, show the data. Every other decision asks "does this help the reader see the data?"
```
