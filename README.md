# Lounge Business Dashboard

A dark, responsive business dashboard for a lounge management system. The application visualizes PlayStation console, billiard table, and ping pong table activity and earnings.

## How This Application Was Built

This application was built as a client-side React application using Vite.

- **React** manages the page structure, component rendering, and interactive state.
- **Vite** provides the development server and production build pipeline.
- **JSX** defines the dashboard UI and chart markup.
- **SVG** is used to render the four charts without a charting library. SVG keeps the visual output lightweight and allows each data point or bar to be interactive.
- **CSS** provides the dark visual system, responsive layout, typography, chart styling, focus states, and mobile behavior.
- **Google Fonts** provides the `Manrope` and `DM Mono` typefaces.

The main application is implemented in [`src/App.jsx`](src/App.jsx). React is mounted from [`src/main.jsx`](src/main.jsx), and the Vite HTML entry point is [`index.html`](index.html).

## Dashboard Scope

The Business page intentionally contains exactly four charts:

1. **Today / Last 24 Hours Active Sessions** - SVG area chart with PlayStation, billiard, and ping pong series.
2. **Daily Total Cost** - SVG stacked column chart with one segment per business resource.
3. **Daily Total Cost Trend** - SVG line chart using combined daily totals.
4. **Monthly Total Cost in the Current Year** - SVG column chart using the selected year's monthly totals.

No additional KPI cards, statistics, filters, or charts are included.

## Project Structure

```text
lounge_dashboard/
├── index.html          # Vite HTML entry point
├── package.json        # Scripts and React/Vite dependencies
├── package-lock.json   # Locked npm dependency versions
├── styles.css          # Global layout, dark theme, responsive styles
└── src/
    ├── App.jsx         # Dashboard components, data, state, and interactions
    └── main.jsx        # React application mount point
```

## React Components

`src/App.jsx` contains the following components:

- `App` - Owns search, feature selection, year selection, and tooltip state.
- `ActiveChart` - Renders the 24-hour active-session area chart.
- `CostChart` - Renders stacked daily cost columns.
- `TrendChart` - Renders the daily cost trend line.
- `MonthlyChart` - Renders monthly totals for the selected year.
- `ChartSvg` - Shared SVG chart wrapper.
- `Grid` - Shared horizontal chart grid and axis labels.
- `XLabels` - Shared horizontal axis labels.
- `Mark` - Shared interactive SVG mark with pointer, touch, and keyboard behavior.
- `Legend` - Shared chart legend.
- `PanelHeading` - Shared chart heading and unit label.

## Data and Calculations

The current dashboard uses in-memory demo data declared in `src/App.jsx`:

- `active` contains 24-hour session data for each resource.
- `daily` contains seven days of cost data for each resource.
- `yearly` contains monthly totals for 2024, 2025, and 2026.

Daily totals are calculated by adding the PlayStation, billiard, and ping pong values for each day. Chart maximums are calculated dynamically with `maxFor()`, which adds headroom above the largest value so bars, points, and labels remain inside the chart.

The yearly earnings total is calculated by summing the 12 monthly values for the selected year.

## User Interactions

- The feature search field filters the four service navigation links.
- The feature dropdown selects a chart and smoothly scrolls to its title.
- The service navigation links also scroll to their chart titles.
- Chart marks show contextual tooltips on desktop hover.
- Chart marks can be reached with the keyboard.
- On touch devices, a single tap opens a tooltip immediately.
- Tapping the same mark again closes its tooltip.
- Tapping another mark changes the selected tooltip.
- Selecting a year updates the monthly chart and total yearly earnings.

## Responsive Design

The layout is responsive through CSS media queries:

- Desktop uses a two-column chart grid with full-width area and yearly sections.
- Phones use a single-column chart layout.
- The four service navigation items become a two-by-two grid on narrow screens.
- Search and selection controls stack vertically on phones.
- SVG charts keep stable dimensions through a fixed viewBox and responsive container sizing.

The interface was checked at 320px and 390px phone widths, with no horizontal overflow detected.

## Current Application Boundary

This repository currently contains only a frontend. It does **not** include:

- A backend server
- REST or GraphQL API integration
- A database
- Authentication or authorization
- Persistent session or earnings storage
- Real-time updates
- Server-side validation

The values shown are demo values. A production version should replace the in-memory constants with an authenticated backend data source and should add loading, error, empty-state, last-updated, validation, and authorization handling.

## Requirements

- Node.js compatible with the installed Vite version.
- npm.

## Run Locally

Install dependencies:

```powershell
npm install
```

Start the development server:

```powershell
npm run dev
```

Open the local URL printed by Vite, typically:

```text
http://127.0.0.1:5173/
```

## Production Build

Create a production build:

```powershell
npm run build
```

The compiled output is written to `dist/`.

Preview the production build locally:

```powershell
npm run preview
```

## Validation

The current project has no automated test or lint script. The available baseline checks are:

```powershell
npm run build
npm audit
```

Manual browser checks should cover:

- Four charts render.
- Search filtering works.
- Feature navigation scrolls to chart titles.
- Year selection updates the chart and total earnings.
- Hover, keyboard focus, and one-tap chart interactions work.
- Phone widths around 320px and 390px do not overflow.
