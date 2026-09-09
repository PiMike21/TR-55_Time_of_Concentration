# TR-55 Time of Concentration Lab

A standalone browser-based calculator for estimating **time of concentration (Tc)** using the travel-time procedures in **USDA-SCS Technical Release 55 (TR-55), Chapter 3**.

The application models a hydraulic flow path as:

1. one fixed **sheet-flow** reach at the upstream end; followed by
2. any number of **shallow concentrated-flow** and **open-channel-flow** segments.

It calculates travel time for each segment, cumulative travel time along the path, the resulting Tc, TR-55 method warnings, and printable calculation reports.

> **Reference basis:** USDA Soil Conservation Service, *Technical Release 55 — Urban Hydrology for Small Watersheds*, Second Edition, June 1986, Chapter 3 and Appendix F.

---

## Overview

TR-55 defines time of concentration as the time required for runoff to travel from the hydraulically most distant point of a watershed to the point of interest. For a path containing multiple flow regimes, Tc is determined by summing the travel times of the consecutive flow segments.

This app turns the TR-55 Worksheet 3 workflow into an interactive calculation tool that is easier to use for long or mixed hydraulic paths.

### Primary capabilities

- Fixed upstream sheet-flow segment
- Unlimited downstream shallow concentrated-flow segments
- Unlimited downstream open-channel-flow segments
- Live travel-time and Tc calculations
- Segment reordering, duplication, and deletion
- TR-55 sheet-flow Manning's `n` presets
- Custom sheet-flow roughness
- Paved and unpaved shallow concentrated flow
- Manning-equation channel calculations
- Cumulative hydraulic travel-time table
- Hydraulic flow-path visualization
- Calculation substitutions
- Built-in QA/QC and regression checks
- TR-55 method-limit warnings
- Automatic browser persistence
- JSON input save/load
- CSV segment export
- Compact and detailed printable reports
- No external libraries or server required

---

## Quick Start

The application is a single HTML file:

```text
tr55_time_of_concentration_lab.html
```

### Option 1 — Open directly

Open the HTML file in a modern desktop browser.

No installation, build process, package manager, or internet connection is required.

### Option 2 — Run from a local web server

For development or more predictable browser storage behavior, serve the file from a local HTTP server.

For example, if Python is installed:

```bash
python -m http.server 8000
```

Then browse to:

```text
http://localhost:8000/tr55_time_of_concentration_lab.html
```

---

## Calculation Workflow

The interface follows the hydraulic path from upstream to downstream.

```text
Hydraulically most distant point
          |
          v
   +----------------+
   |   Sheet Flow   |
   +----------------+
          |
          v
 +----------------------+
 | Shallow Concentrated |
 +----------------------+
          |
          v
   +----------------+
   |  Channel Flow  |
   +----------------+
          |
          v
       Outlet
```

Only the sheet-flow segment is fixed. Add as many shallow concentrated or channel reaches as necessary to represent the downstream flow path.

The calculated time of concentration is:

```text
Tc = sum of all segment travel times
```

---

# Inputs

## Project / Drainage Path

The **Project / drainage path** field identifies the calculation in the interface, saved JSON, and printed report.

Examples:

```text
North Basin - Path A
DA-3 to Inlet 27
Existing Conditions - West Watershed
```

---

## P2 — 2-Year, 24-Hour Rainfall

The sheet-flow equation requires the **2-year, 24-hour rainfall depth**, `P2`, in inches.

```text
P2 = 2-year, 24-hour rainfall depth, in
```

`P2` affects the sheet-flow travel time only. Shallow concentrated and channel segments are velocity-based and do not directly use `P2`.

---

# Flow Types

## 1. Sheet Flow

Sheet flow represents shallow runoff over a plane surface near the upstream end of the hydraulic path.

The application uses the TR-55 kinematic sheet-flow equation:

```text
             0.007 (nL)^0.8
Tt = ------------------------------
        P2^0.5 * s^0.4
```

where:

| Variable | Description | Units |
|---|---|---|
| `Tt` | travel time | hr |
| `n` | sheet-flow Manning's roughness coefficient | dimensionless |
| `L` | sheet-flow length | ft |
| `P2` | 2-year, 24-hour rainfall | in |
| `s` | land / hydraulic-grade-line slope | ft/ft |

### Sheet-flow roughness presets

The app includes the TR-55 Table 3-1 sheet-flow roughness values:

| Surface | Manning's n |
|---|---:|
| Smooth surfaces — concrete, asphalt, gravel, or bare soil | 0.011 |
| Fallow — no residue | 0.05 |
| Cultivated soil — residue cover <= 20% | 0.06 |
| Cultivated soil — residue cover > 20% | 0.17 |
| Short grass prairie | 0.15 |
| Dense grasses | 0.24 |
| Bermudagrass | 0.41 |
| Range — natural | 0.13 |
| Woods — light underbrush | 0.40 |
| Woods — dense underbrush | 0.80 |

A **Custom Manning's n** option is also available.

### 300-foot sheet-flow limit

TR-55 states that the kinematic sheet-flow solution should not be used for sheet-flow lengths greater than **300 ft**.

The calculator does not prevent a longer value from being entered, but it displays a prominent method warning and includes that warning in the printed report.

For a path longer than the allowable sheet-flow reach, transition the downstream portion to the appropriate shallow concentrated or channel flow regime.

---

## 2. Shallow Concentrated Flow

After sheet flow transitions into a defined path, runoff is commonly represented as shallow concentrated flow.

The app supports:

- **Unpaved**
- **Paved**

Instead of requiring graphical interpolation from TR-55 Figure 3-1, the app uses the equations provided in Appendix F that generate the paved and unpaved relationships.

### Unpaved

```text
V = 16.1345 * sqrt(s)
```

### Paved

```text
V = 20.3282 * sqrt(s)
```

where:

| Variable | Description | Units |
|---|---|---|
| `V` | average velocity | ft/s |
| `s` | watercourse / hydraulic-grade-line slope | ft/ft |

Travel time is then calculated from:

```text
Tt = L / (3600 V)
```

where `L` is the flow length in feet.

---

## 3. Open Channel Flow

Open-channel segments use the US customary Manning equation.

The required inputs are:

- flow length, `L`
- channel slope, `s`
- Manning's roughness coefficient, `n`
- cross-sectional flow area, `A`
- wetted perimeter, `Pw`

### Hydraulic radius

```text
R = A / Pw
```

### Manning velocity

```text
V = (1.49 / n) * R^(2/3) * s^(1/2)
```

### Travel time

```text
Tt = L / (3600 V)
```

where:

| Variable | Description | Units |
|---|---|---|
| `A` | cross-sectional flow area | ft² |
| `Pw` | wetted perimeter | ft |
| `R` | hydraulic radius | ft |
| `n` | Manning's roughness coefficient | dimensionless |
| `s` | hydraulic-grade-line / channel slope | ft/ft |
| `V` | average velocity | ft/s |
| `L` | segment length | ft |
| `Tt` | segment travel time | hr |

Channel Manning's `n` is entered by the user. The TR-55 Chapter 3 procedure refers the engineer to standard hydraulic references for appropriate open-channel roughness values.

---

# Time of Concentration

For consecutive reaches along the hydraulic path:

```text
Tc = Tt1 + Tt2 + Tt3 + ... + Ttm
```

The app reports both:

### Calculated Tc

The direct arithmetic sum of all entered segment travel times.

### TR-55 Tc Used

The value after applying the TR-55 minimum:

```text
TR-55 Tc used = max(Calculated Tc, 0.10 hr)
```

The calculated value is never hidden. If the calculated Tc is below `0.10 hr`, both values are shown so the reviewer can see that the minimum was applied.

---

# Segment Management

Downstream segments are stored as an ordered hydraulic path.

Each shallow concentrated or channel segment can be:

- moved upstream
- moved downstream
- duplicated
- deleted

The sheet-flow segment is always Segment 1 and cannot be deleted or moved.

Segment numbering updates automatically when the path is reordered.

---

# Results

The primary results area reports:

- Calculated Tc
- TR-55 Tc used
- Tc in minutes
- Total flow-path length
- Total number of segments
- Sheet-flow travel-time subtotal
- Shallow-flow travel-time subtotal
- Channel-flow travel-time subtotal

---

## Hydraulic Flow Path

The **Hydraulic flow path** panel provides a visual upstream-to-downstream representation of the calculation.

Each reach displays the primary hydraulic inputs and computed travel time.

This view is intended to help identify:

- missing reaches
- incorrect ordering
- unexpected velocities
- disproportionate travel-time contributions
- transitions between flow regimes

---

## Segment Results

The segment table provides one row per hydraulic reach.

Typical columns include:

| Column | Description |
|---|---|
| Segment | ordered reach number and name |
| Type | Sheet, Shallow, or Channel |
| `L` | flow length |
| `s` | slope |
| `n` | Manning's roughness, when applicable |
| `R` | hydraulic radius for channel flow |
| `V` | calculated velocity for velocity-based reaches |
| `Tt` | individual segment travel time |
| Cum. T | cumulative travel time through the segment |

The final cumulative time equals the calculated Tc.

---

# Calculation Steps

The **Calculation steps** tab expands the active calculation into equation substitutions.

It shows:

- sheet-flow equation and substituted values
- shallow concentrated-flow velocity and travel time
- channel hydraulic radius
- channel Manning velocity
- segment travel time
- final Tc summation
- TR-55 minimum-Tc application

This tab is useful when an individual reach needs detailed review without expanding the default compact report.

---

# Validation / QA-QC

The **Validation / QA-QC** tab contains both locked regression checks and current-run consistency checks.

## TR-55 Example 3-1 Regression Case

The application includes a locked regression case based on TR-55 Example 3-1.

The reference flow path is:

```text
AB: Sheet flow
    Dense grass
    n = 0.24
    L = 100 ft
    s = 0.01 ft/ft
    P2 = 3.6 in

BC: Shallow concentrated flow
    Unpaved
    L = 1,400 ft
    s = 0.01 ft/ft

CD: Channel flow
    n = 0.05
    A = 27 ft²
    Pw = 28.2 ft
    L = 7,300 ft
    s = 0.005 ft/ft
```

TR-55 reports approximately:

```text
Sheet Tt   = 0.30 hr
Shallow Tt = 0.24 hr
Channel Tt = 0.99 hr

Tc          = 1.53 hr
```

The application's locked tests verify that the implementation reproduces these rounded worksheet results within defined tolerances.

## Current-run checks

The QA/QC panel also checks the active project for items such as:

- positive segment travel times
- correct Tc summation
- sheet-flow length relative to the 300-ft method limit

A passing QA/QC panel demonstrates internal implementation consistency; it does not establish calibration, regulatory acceptance, or correctness of the user's hydraulic assumptions.

---

# Method Warnings

The calculator identifies selected TR-55 method conditions and surfaces them in the UI and reports.

Current warnings include:

## Sheet flow longer than 300 ft

A warning is issued when:

```text
Lsheet > 300 ft
```

## Calculated Tc below 0.10 hr

When:

```text
Calculated Tc < 0.10 hr
```

the app preserves the calculated value and separately reports:

```text
TR-55 Tc used = 0.10 hr
```

Warnings do not block calculation. This allows the engineer to review the entered condition while clearly identifying departures from the stated method limits.

---

# Print Reports

Select **Print report** from the Segment Results tab.

Two report formats are available.

## Compact Calculation Report

The default report is optimized for engineering review and minimum page count.

It uses **letter-size landscape orientation** and includes:

- project / drainage-path identifier
- `P2`
- generated date and time
- calculated Tc
- TR-55 Tc used
- Tc in minutes
- total flow length
- segment count
- active method warnings
- one compact row per hydraulic segment
- cumulative travel time
- equation basis
- TR-55 reference note

The table header repeats automatically when long flow paths continue onto additional printed pages.

This is the recommended report for routine calculation packages.

## Full Calculation Appendix

The full report begins with the same compact calculation sheet and then adds:

1. expanded equation substitutions for every segment
2. Validation / QA-QC results

Use this option when a reviewer needs detailed computational traceability.

---

# CSV Export

**Export segment CSV** creates a comma-separated file containing the hydraulic segment results.

The export includes:

```text
segment
type
length_ft
slope_ft_per_ft
manning_n
hydraulic_radius_ft
velocity_ft_per_s
travel_time_hr
cumulative_time_hr
```

This is useful for:

- spreadsheet review
- independent QA/QC
- calculation archives
- comparison with other hydraulic tools
- downstream data processing

---

# JSON Save / Load

The complete calculation input state can be saved to JSON and loaded later.

Saved state includes:

- project / path name
- `P2`
- sheet-flow surface
- sheet-flow Manning's `n`
- sheet-flow length
- sheet-flow slope
- all downstream segments
- segment types
- segment names
- segment hydraulic properties

Use **Save input JSON** to create a project input file.

Use **Load input JSON** to restore it.

---

# Automatic Persistence

The application attempts to save the current input state in browser `localStorage`.

When the page is reopened in the same browser context, the previous calculation can be restored automatically.

If local storage is unavailable or blocked, the calculator continues to operate; use JSON files for explicit project persistence.

> Browser handling of `localStorage` for files opened directly with `file://` can vary. Serving the app over local HTTP generally provides more predictable persistence.

---

# TR-55 Example

Use **Load TR-55 example** to populate the app with the Example 3-1 hydraulic path.

This is useful for:

- checking that the application is operating correctly
- learning the interface
- reviewing expected segment behavior
- comparing the app with the published TR-55 worksheet

---

# Reset

**Reset** clears the saved browser state and returns the calculator to a simple starting condition.

This operation does not delete JSON or CSV files previously saved to disk.

---

# Units

The current application uses **US customary units**, consistent with the equations implemented from TR-55.

| Quantity | Unit |
|---|---|
| Rainfall depth | in |
| Length | ft |
| Slope | ft/ft |
| Area | ft² |
| Wetted perimeter | ft |
| Hydraulic radius | ft |
| Velocity | ft/s |
| Travel time | hr |
| Tc | hr and min |

Metric conversion is not currently implemented.

---

# Engineering Assumptions and Limitations

This application is a calculation aid, not a substitute for hydrologic or hydraulic engineering judgment.

## Hydraulic path selection

The app does **not** determine the hydraulically most distant point or the correct hydraulic path automatically.

The user is responsible for identifying the path from field information, mapping, grading, drainage design, or other appropriate sources.

## Flow-regime transitions

The app does not automatically determine where:

- sheet flow becomes shallow concentrated flow
- shallow concentrated flow becomes channel flow
- a storm drain or culvert controls the travel path

The user must select appropriate segment boundaries.

## Sheet-flow applicability

The TR-55 sheet-flow equation is limited to sheet-flow lengths of 300 ft or less.

## Channel hydraulics

The application computes Manning velocity from user-entered geometric and roughness data. It does not:

- solve normal depth
- solve critical depth
- compute a water-surface profile
- determine bankfull geometry
- size channels
- perform gradually varied flow calculations

The user is responsible for supplying representative channel inputs.

## Storm sewers and pipes

TR-55 notes that storm-sewer systems require careful identification of the actual hydraulic path and appropriate pipe velocity methods.

This calculator currently provides sheet, shallow concentrated, and open-channel reaches only. It does not provide a dedicated pipe-flow solver.

## Culverts and storage effects

A culvert or bridge with significant upstream storage can behave as a reservoir outlet. Such conditions may require storage routing beyond this travel-time calculation.

## Reservoirs and lakes

TR-55 discusses reservoir/lake travel time separately. This app does not currently include a dedicated reservoir/lake segment.

## Regulatory requirements

The tool does not determine whether TR-55 is the governing method for a particular:

- municipality
- county
- state
- federal agency
- drainage manual
- permit
- project type

Always use the methodology and design criteria required by the reviewing authority.

---

# Calculation Reference

The implemented relationships are based on:

**USDA Soil Conservation Service**  
*Urban Hydrology for Small Watersheds*  
Technical Release 55  
Second Edition, June 1986

Primary application sections:

- Chapter 3 — Time of Concentration and Travel Time
- Table 3-1 — Roughness coefficients for sheet flow
- Figure 3-1 — Average velocities for shallow concentrated flow
- Equation 3-1 — Travel time
- Equation 3-3 — Sheet-flow travel time
- Equation 3-4 — Manning velocity
- Worksheet 3 — Time of Concentration / Travel Time
- Appendix F — Equations for figures and exhibits
- Example 3-1 — Tc regression / validation case

---

# Application Architecture

The application is intentionally implemented as a single self-contained HTML file.

```text
tr55_time_of_concentration_lab.html
```

It contains:

```text
HTML
  User interface
  Input controls
  Result panels
  Report markup

CSS
  Application layout
  Responsive behavior
  Segment styling
  QA/QC styling
  Print-report styling

JavaScript
  State management
  Hydraulic calculations
  Segment management
  Validation
  Rendering
  CSV export
  JSON save/load
  localStorage persistence
  Report generation
```

There are no external runtime dependencies.

---

# Core Calculation Functions

The application's calculation logic is organized around a few small functions.

Conceptually:

```javascript
calcSheet(n, L, p2, s)
shallowVelocity(surface, s)
travelTime(L, V)
channelCalc(segment)
compute()
```

`compute()` evaluates the complete ordered hydraulic path and returns the segment-level and overall Tc results used by the UI, reports, and exports.

A key design objective is that the application, QA/QC panel, CSV export, and print report all derive from the same calculation result rather than maintaining separate computational implementations.

---

# Data Model

A typical project state is conceptually similar to:

```javascript
{
  projectName: "North Basin - Path A",
  p2: 3.6,

  sheetSurface: "0.24",
  sheetN: 0.24,
  sheetLength: 100,
  sheetSlope: 0.01,

  segments: [
    {
      type: "shallow",
      name: "BC",
      surface: "unpaved",
      length: 1400,
      slope: 0.01
    },
    {
      type: "channel",
      name: "CD",
      length: 7300,
      slope: 0.005,
      n: 0.05,
      area: 27,
      wettedPerimeter: 28.2
    }
  ]
}
```

The order of `segments` is the downstream hydraulic order.

---

# Browser Requirements

The calculator is intended for current desktop versions of modern browsers, including:

- Microsoft Edge
- Google Chrome
- Mozilla Firefox
- Safari

The app relies on standard browser functionality including:

- ES6 JavaScript
- `localStorage`
- `Blob`
- `FileReader`
- object URLs
- CSS Grid / Flexbox
- CSS print media
- `window.print()`

No browser extension is required.

---

# Development

Because the application has no build system, development can be done directly in the HTML file.

A simple workflow is:

```text
1. Edit tr55_time_of_concentration_lab.html
2. Refresh the browser
3. Load the TR-55 example
4. Review QA/QC
5. Test compact print
6. Test full appendix print
7. Export CSV / JSON as needed
```

For changes to hydraulic equations, update the regression tests at the same time and independently verify the expected values before changing locked benchmarks.

---

# Verification Checklist for Changes

Before releasing a modified version of the calculator, verify at minimum:

- the TR-55 Example 3-1 locked checks pass
- the sheet-flow equation still reproduces the benchmark value
- paved and unpaved shallow-flow equations are unchanged unless intentionally revised
- channel hydraulic radius is `A / Pw`
- Manning velocity uses the correct US customary coefficient
- travel time uses `L / (3600 V)`
- Tc equals the sum of all segment travel times
- the 0.10-hour minimum affects `Tc used`, not the displayed raw calculation
- sheet-flow lengths above 300 ft generate a warning
- adding, deleting, duplicating, and reordering segments preserves calculation order
- saved JSON reloads correctly
- exported CSV matches displayed results
- compact print remains readable with a long segment list
- full appendix includes calculations and QA/QC
- no print-only content appears in the normal application view

---

# Recommended Future Enhancements

Potential extensions that fit the current architecture include:

- dedicated pipe / storm-sewer travel-time segments
- reservoir / lake segment support
- channel cross-section helpers
- built-in open-channel Manning's `n` reference table
- map or schematic attachment to the report
- prepared-by / checked-by project metadata
- calculation revision identifiers
- SI / metric unit support
- import/export of multiple hydraulic paths
- comparison of existing and proposed Tc
- optional jurisdiction-specific warnings
- direct handoff of Tc to a broader TR-55 runoff / peak-discharge workflow

Future additions should preserve the current principle that the hydraulic path remains explicit, ordered, reviewable, and reproducible.

---

# Engineering Disclaimer

This software is provided as an engineering calculation aid.

It does not replace:

- the TR-55 source document
- field verification
- professional engineering judgment
- hydraulic analysis where required
- calibration
- agency-approved software where mandated
- jurisdiction-specific criteria
- independent QA/QC

The engineer or designer remains responsible for the hydraulic path, input data, applicability of the method, interpretation of results, and final design decisions.

---

# Licensing

No open-source license is declared by this README.

If the application will be publicly distributed or maintained as an open-source project, add an appropriate `LICENSE` file and update this section accordingly.

---

## Short Description

> **TR-55 Time of Concentration Lab is an NRCS TR-55 Chapter 3 calculator for sheet, shallow concentrated, and channel flow. Build multi-segment flow paths, calculate travel time and Tc live, review QA/QC checks, and generate compact or detailed calculation reports.**
