# Coderfile: Build the Lounge Business Dashboard From Scratch

This document explains how to recreate the Lounge Business Dashboard from an empty folder. It includes the commands, file structure, implementation patterns, and commented code needed to understand the build.

## 1. Create the Project

Create an empty project directory and move into it:

```powershell
# Create the project folder.
New-Item -ItemType Directory lounge_dashboard
Set-Location lounge_dashboard

# Create a package manifest.
npm init -y

# Install the React runtime and Vite build tools.
npm install react react-dom vite @vitejs/plugin-react
```

Replace the generated `package.json` scripts with:

```json
{
  "name": "lounge-business-dashboard",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "@vitejs/plugin-react": "latest",
    "react": "latest",
    "react-dom": "latest",
    "vite": "latest"
  }
}
```

For a production project, pin dependency versions instead of using `latest`.

## 2. Create the File Structure

Create this structure:

```text
lounge_dashboard/
├── index.html
├── package.json
├── styles.css
└── src/
    ├── App.jsx
    └── main.jsx
```

Create the `src` folder:

```powershell
New-Item -ItemType Directory src
```

## 3. Create the HTML Entry Point

Create `index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="theme-color" content="#111b1d">
    <title>Business | Lounge Operations</title>

    <!-- The application uses these two typefaces for hierarchy and data labels. -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=Manrope:wght@400;500;600;700;800&display=swap" rel="stylesheet">
  </head>
  <body>
    <!-- React renders the complete dashboard into this element. -->
    <div id="root"></div>

    <!-- Vite resolves this module and its imports during development/build. -->
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

## 4. Mount React

Create `src/main.jsx`:

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import '../styles.css';

// Mount the React component tree into the root element from index.html.
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

`StrictMode` helps expose unsafe React patterns during development.

## 5. Define the Business Data

The current prototype stores demo data in memory. A production application should load these values from an authenticated API.

Add this data near the top of `src/App.jsx`:

```jsx
const COLORS = {
  playstation: '#62d4b6',
  billiard: '#f28b6c',
  pingpong: '#e9bd61',
  total: '#8fc4d6'
};

const HOURS = ['00', '03', '06', '09', '12', '15', '18', '21', '24'];
const DAYS = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];
const MONTHS = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];

// Active session counts for each lounge resource over the last 24 hours.
const active = {
  playstation: [2, 1, 3, 7, 12, 10, 17, 21, 16],
  billiard: [1, 0, 2, 4, 6, 8, 11, 10, 7],
  pingpong: [0, 1, 1, 3, 5, 7, 8, 9, 6]
};

// Daily cost values in USD for each resource.
const daily = {
  playstation: [184, 212, 198, 240, 276, 328, 302],
  billiard: [96, 110, 104, 121, 145, 174, 162],
  pingpong: [72, 88, 82, 94, 112, 138, 124]
};

// Monthly totals grouped by year.
const yearly = {
  2026: [6840, 7210, 7680, 7420, 8160, 8940, 9320, 8760, 0, 0, 0, 0],
  2025: [6120, 6580, 7040, 6880, 7350, 7920, 8240, 8010, 7680, 8460, 8790, 9240],
  2024: [5480, 5720, 6190, 6030, 6640, 7080, 7560, 7310, 7020, 7780, 8120, 8560]
};

// Combine the three resource costs for one day.
const totalAt = (group, index) =>
  group.playstation[index] + group.billiard[index] + group.pingpong[index];

// Add headroom so bars and labels never touch the top of the chart.
const maxFor = (values, step) => Math.max(
  step,
  Math.ceil((Math.max(...values, 0) * 1.15) / step) * step
);
```

## 6. Build Shared SVG Helpers

The dashboard uses SVG directly so every data mark can receive pointer, touch, and keyboard events.

```jsx
function ChartSvg({ label, children }) {
  return (
    <svg
      viewBox="0 0 900 270"
      role="img"
      aria-label={label}
      preserveAspectRatio="none"
      className="chart-draw"
    >
      {children}
    </svg>
  );
}

function Grid({ max, ticks = 3 }) {
  return (
    <>
      {Array.from({ length: ticks + 1 }, (_, index) => {
        const y = 12 + (218 * index) / ticks;
        const value = Math.round(max - (max * index) / ticks);

        return (
          <g key={index}>
            <line x1="48" y1={y} x2="890" y2={y} className="grid-line" />
            <text x="38" y={y + 3} textAnchor="end" className="axis-label">
              {value}
            </text>
          </g>
        );
      })}
    </>
  );
}

function XLabels({ labels, left = 48, width = 842 }) {
  return (
    <>
      {labels.map((label, index) => (
        <text
          key={label}
          x={left + (width * index) / (labels.length - 1)}
          y="255"
          textAnchor="middle"
          className="axis-label"
        >
          {label}
        </text>
      ))}
    </>
  );
}
```

## 7. Make Chart Marks Interactive

`Mark` is shared by all four chart types. It provides:

- Desktop hover tooltips
- Keyboard focus
- Immediate one-tap interaction on phones
- Accessible labels for every data mark

```jsx
function Mark({
  children,
  period,
  source,
  value,
  onShow,
  onHide,
  selected,
  onToggle
}) {
  const label = `${period}, ${source}: ${value}`;

  return (
    <g
      tabIndex="0"
      role="img"
      aria-label={label}
      className={`chart-mark${selected ? ' is-selected' : ''}`}
      onPointerEnter={event => {
        // Touch devices should select on pointerdown instead of showing a hover preview.
        if (event.pointerType !== 'touch') onShow(event, period, source, value);
      }}
      onPointerMove={event => {
        if (event.pointerType !== 'touch') onShow(event, period, source, value);
      }}
      onPointerLeave={event => {
        if (event.pointerType !== 'touch') onHide();
      }}
      onPointerDown={event => {
        // pointerdown makes one-tap interaction immediate on phones.
        if (event.pointerType === 'touch') {
          event.preventDefault();
          onToggle(event, period, source, value);
        }
      }}
      onFocus={event => onShow(event, period, source, value)}
      onBlur={onHide}
    >
      {children}
    </g>
  );
}
```

## 8. Implement the Four Charts

### Area Chart: Active Sessions

```jsx
function ActiveChart({ tooltip }) {
  const max = 30;
  const series = [
    ['playstation', 'PlayStation', active.playstation, 0.14],
    ['billiard', 'Billiard', active.billiard, 0.12],
    ['pingpong', 'Ping pong', active.pingpong, 0.12]
  ];

  const point = (value, index) => ({
    x: 48 + (842 * index) / 8,
    y: 230 - (value / max) * 218
  });

  return (
    <ChartSvg label="Active sessions over the last 24 hours">
      <Grid max={max} />
      {series.map(([key, name, values, opacity]) => (
        <g key={key}>
          <polyline
            points={values.map((value, index) => {
              const item = point(value, index);
              return `${item.x},${item.y}`;
            }).join(' ')}
            fill="none"
            stroke={COLORS[key]}
            strokeWidth="3"
          />
          {values.map((value, index) => {
            const item = point(value, index);
            return (
              <Mark
                key={index}
                period={HOURS[index]}
                source={name}
                value={`${value} sessions`}
                selected={tooltip?.period === HOURS[index] && tooltip?.source === name}
                onShow={tooltip.show}
                onHide={tooltip.hide}
                onToggle={tooltip.toggle}
              >
                <circle cx={item.x} cy={item.y} r="7" fill={COLORS[key]} opacity={opacity + 0.25} />
              </Mark>
            );
          })}
        </g>
      ))}
      <XLabels labels={HOURS} />
    </ChartSvg>
  );
}
```

The other three charts follow the same pattern:

- Calculate a scale with `maxFor()`.
- Convert values into SVG coordinates.
- Render bars or points.
- Wrap interactive marks in `Mark`.
- Render axes with `Grid` and `XLabels`.

The current implementation of `CostChart`, `TrendChart`, and `MonthlyChart` is in [`src/App.jsx`](src/App.jsx).

## 9. Add the Main React State

The `App` component owns state shared by the navigation and charts:

```jsx
import { useEffect, useMemo, useState } from 'react';

function App() {
  const [query, setQuery] = useState('');
  const [feature, setFeature] = useState('all');
  const [year, setYear] = useState('2026');
  const [tooltip, setTooltip] = useState(null);

  // Store enough information to position the tooltip near the pointer.
  const showTooltip = (event, period, source, value) => {
    setTooltip({
      period,
      source,
      value,
      touch: event?.pointerType === 'touch',
      x: event?.clientX,
      y: event?.clientY
    });
  };

  const hideTooltip = () => {
    // Do not close a tooltip that was selected by touch.
    setTooltip(current => current?.touch ? current : null);
  };

  const toggleTooltip = (event, period, source, value) => {
    setTooltip(current => {
      const sameMark = current?.period === period && current?.source === source;
      return sameMark
        ? null
        : { period, source, value, touch: true, x: event?.clientX, y: event?.clientY };
    });
  };

  // Close a touch tooltip when the user taps outside a chart mark.
  useEffect(() => {
    const close = event => {
      if (event.pointerType === 'touch' && !event.target.closest('.chart-mark')) {
        setTooltip(null);
      }
    };

    document.addEventListener('pointerdown', close);
    return () => document.removeEventListener('pointerdown', close);
  }, []);

  const totalEarned = yearly[year].reduce((sum, value) => sum + value, 0);

  // Render the page structure and pass state into the chart components.
  return (
    <main className="page-shell">
      {/* Header, search, service navigation, and four chart panels go here. */}
    </main>
  );
}
```

## 10. Implement Feature Navigation

Each navigation item maps to one chart. The click handler prevents the browser's default anchor jump and scrolls to the chart title instead.

```jsx
const links = [
  { id: 'active-sessions-chart', name: 'Active sessions' },
  { id: 'daily-cost-chart', name: 'Daily total cost' },
  { id: 'daily-trend-chart', name: 'Daily cost trend' },
  { id: 'monthly-cost-chart', name: 'Monthly total cost' }
];

const focusFeature = id => {
  setFeature(id);

  // Scroll to the heading, not the middle of the SVG chart.
  document
    .getElementById(id)
    ?.querySelector('.panel-heading')
    ?.scrollIntoView({ behavior: 'smooth', block: 'start' });
};

<nav className="service-nav" aria-label="Business features">
  {links
    .filter(link => !query || link.name.toLowerCase().includes(query.toLowerCase()))
    .map(link => (
      <a
        key={link.id}
        href={`#${link.id}`}
        className={`service-link${feature === link.id ? ' is-active' : ''}`}
        onClick={event => {
          // Avoid the native jump competing with smooth scrolling.
          event.preventDefault();
          focusFeature(link.id);
        }}
      >
        {link.name}
      </a>
    ))}
</nav>
```

## 11. Implement Year Selection

The yearly selector changes both the monthly chart data and the total earned value:

```jsx
<div className="year-controls">
  <label className="year-select">
    <span className="sr-only">Select year</span>
    <select value={year} onChange={event => {
      setYear(event.target.value);
      setTooltip(null);
    }}>
      {Object.keys(yearly).sort((a, b) => b - a).map(option => (
        <option key={option} value={option}>{option}</option>
      ))}
    </select>
  </label>

  <div className="year-earning">
    <span>Total earned</span>
    <strong>${totalEarned.toLocaleString()}</strong>
  </div>
</div>
```

The chart receives the selected year's values:

```jsx
<MonthlyChart
  values={yearly[year]}
  year={year}
  tooltip={tooltipApi}
/>
```

## 12. Add the Responsive Dark Theme

The dashboard's CSS uses variables so the visual system is easy to maintain:

```css
:root {
  --ink: #edf4f2;
  --muted: #96aaa9;
  --line: #304342;
  --paper: #111b1d;
  --panel: #192527;
  --mint: #62d4b6;
  --coral: #f28b6c;
  --gold: #e9bd61;
  --navy: #8fc4d6;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  color: var(--ink);
  background: var(--paper);
  font-family: 'Manrope', sans-serif;
}

.chart-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 22px;
}

.chart-panel {
  min-width: 0;
  padding: 28px;
  background: var(--panel);
  border: 1px solid #2b3c3e;
}

.chart-panel-wide {
  grid-column: span 2;
}

.chart-wrap {
  width: 100%;
  height: 270px;
}

.chart-wrap svg {
  display: block;
  width: 100%;
  height: 100%;
}

.chart-mark {
  cursor: crosshair;
  outline: none;
  touch-action: manipulation;
}

@media (max-width: 760px) {
  .page-shell {
    padding: 34px 18px 48px;
  }

  .business-tools {
    flex-direction: column;
  }

  .service-nav {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .chart-grid {
    grid-template-columns: 1fr;
  }

  .chart-panel-wide {
    grid-column: auto;
  }
}

@media (prefers-reduced-motion: reduce) {
  .chart-draw {
    animation: none;
  }
}
```

The complete styling is in [`styles.css`](styles.css). The mobile rules stack controls and charts so the layout remains usable at narrow phone widths.

## 13. Add Production Commands

Start the development server:

```powershell
npm run dev
```

Build the production files:

```powershell
npm run build
```

Preview the production build:

```powershell
npm run preview
```

Use `npm ci` in CI or deployment environments after committing `package-lock.json`:

```powershell
npm ci
npm run build
```

## 14. Validate the Application

Run the baseline checks:

```powershell
npm run build
npm audit
```

Manually verify:

- Exactly four charts render.
- The search field filters navigation items.
- The feature dropdown selects the correct chart.
- Navigation lands on chart titles.
- Year selection updates the monthly chart and total earnings.
- Hover tooltips work on desktop.
- Keyboard focus exposes every chart mark.
- One tap opens a tooltip on touch devices.
- A second tap closes the selected tooltip.
- The daily `640` total remains inside the chart with its label above the bar.
- 320px and 390px phone widths have no horizontal overflow.
- Reduced-motion users do not receive chart reveal animation.

## 15. Current Production Boundary

The current implementation is a frontend prototype. It has no backend, API, database, authentication, authorization, persistence, or real-time updates. All data is currently hard-coded in `src/App.jsx`.

For production, replace the constants with an authenticated data service and add:

- Loading states
- API error states
- Empty states
- Last-updated timestamps
- Server-side aggregation
- Input and response validation
- Authorization by lounge or tenant
- Audit logging for financial data
- Database transactions and consistency rules
- Secure environment and secret management
- Automated unit, component, and browser tests
