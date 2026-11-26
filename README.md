# Luka

**Pressure losses in air ductwork — in one HTML file.**

### ▶ [Open Luka in your browser](https://kata030.github.io/Luka/)

Try an example right away: [office network with fan sizing](https://kata030.github.io/Luka/?example=office-controlled) · [run the self-tests](https://kata030.github.io/Luka/?selftest)

Draw a duct network, calculate volume flows and pressures, size fans and export the results to Excel. No server, no build step, no installation: open `luka.html` in a current browser and start drawing.

---

## Features

- **Graphical network editor** — drag components from the palette, connect ports, rotate, align, distribute and duplicate symbols, with full undo/redo.
- **Fast, robust solver** — Newton method on a Schur-complement system (RCM ordering + envelope Cholesky). The example networks (~135 components) solve in about 15 ms; a 6000-component tree in about 0.5 s.
- **Fan sizing** — finds the required fan pressure so that the most critical flow controller is fully open, and shows the throttling pressure of all others.
- **Re-dependent fittings** — fixed ζ, power-law or tabulated ζ(Re) (Akima interpolation), and a bend correlation (Krüger / Miller) for round and rectangular sections.
- **Live results** — auto-calculation in a Web Worker 250 ms after each edit; colour the drawing by power loss, pressure loss, gauge pressure or controller throttling.
- **Excel export** — inputs and results of every component written to `.xlsx`.
- **Projects** — saved as `*.luka.json`, autosaved in the browser; legacy `.pip` projects can be converted.
- **Light and dark mode** — follows the system or set manually under *View › Appearance*.
- **Built-in self-tests** — hand calculations, scipy and legacy reference values.

## Getting started

The quickest way is the hosted version at **https://kata030.github.io/Luka/**. To run it locally instead:

1. Download [`luka.html`](luka.html).
2. Open it in a current Chrome, Edge, Firefox or Safari (double-click, or serve it from any static host such as GitHub Pages).

An internet connection is needed on first load, because React, ReactDOM, htm and React Flow are loaded from jsDelivr at pinned versions. Everything else is inside the file.

### URL options

| Parameter       | Effect                                       |
| --------------- | -------------------------------------------- |
| `?example`      | Loads the first embedded example network     |
| `?example=<id>` | Loads a specific example (see list below)    |
| `?selftest`     | Opens the self-test report on start          |

### Offline use

Download the vendor files below and paste each file's content into its `<script>` tag in place of the `src` attribute (likewise for the React Flow stylesheet):

```
https://cdn.jsdelivr.net/npm/react@18.3.1/umd/react.production.min.js
https://cdn.jsdelivr.net/npm/react-dom@18.3.1/umd/react-dom.production.min.js
https://cdn.jsdelivr.net/npm/htm@3.1.1/dist/htm.umd.js
https://cdn.jsdelivr.net/npm/@xyflow/react@12.12.0/dist/umd/index.js
https://cdn.jsdelivr.net/npm/@xyflow/react@12.12.0/dist/style.css
```

## Components

| Type         | Kinds                                                                        | Ports              | Main inputs                                     |
| ------------ | ---------------------------------------------------------------------------- | ------------------ | ----------------------------------------------- |
| `duct`       | duct                                                                         | in, out            | length, roughness k, section                    |
| `zeta`       | bend, filter, fire damper, smoke damper, sound absorber, throttle, custom    | in, out            | ζ, section                                      |
| `zetaRe`     | same as `zeta`                                                               | in, out            | power law (ζref, n) or ζ(Re) table, section     |
| `bendRe`     | bend                                                                         | in, out            | radius, ζref, relative roughness, section       |
| `tee`        | tee                                                                          | in, out, branch    | ζ per leg, section per leg                      |
| `fan`        | generic, axial, radial                                                       | in, out            | volume flow                                     |
| `fanSizing`  | sizing                                                                       | in, out            | — (pressure rise is the result)                 |
| `controller` | flow controller                                                              | in, out            | set flow, ζ when open, section                  |
| `fixedDp`    | fixed Δp                                                                     | in, out            | Δp (negative = loss, positive = rise), section  |

Sections are round (`d`) or rectangular (`w × h`). Units are SI internally; the UI shows flows in m³/h and dimensions in mm.

## Physics in brief

- Connections are ideal, loss-free junctions; an unconnected port is open to the atmosphere (gauge pressure 0 Pa).
- Duct friction: Δp = λ · L/D_h · ρ/2 · v², with λ from **Churchill (1977)** covering laminar, transitional and turbulent flow, and Re = v · D_h / ν using the hydraulic diameter.
- Fittings: Δp = ζ · ρ/2 · v², with ζ constant, a power law in Re, a tabulated curve, or a bend correlation.
- Air properties: ρ = p / (287.05 · T), viscosity after Sutherland; custom ρ and ν are possible.
- Convergence: mass residual < 1e-10 m³/s and pressure residual < 1e-7 Pa.

Several intentional corrections to the legacy Python model are documented in the header comment of `luka.html` (section 4). With those neutralised, results agree with the legacy solver within 0.1–0.3 %.

## Example networks

Open them via *File › Open example network* or `?example=<id>`:

| ID                  | Description                                                                        |
| ------------------- | ---------------------------------------------------------------------------------- |
| `office-controlled` | HRI office, 14 flow controllers at 200 m³/h; required fan pressure 370.5 Pa        |
| `office-fixed`      | HRI office with a fixed fan flow                                                   |
| `production-hall`   | Supply air for a production building                                              |
| `toilet-exhaust`    | Toilet exhaust                                                                     |
| `variable-zeta`     | Two parallel branches: fixed vs. Re-dependent bend ζ                               |
| `museum-supply`     | BOLKA report 2004, museum supply air, 36,300 m³/h; fan pressure 1383.1 Pa          |
| `museum-exhaust`    | Same report, exhaust air; fan pressure 674.9 Pa                                    |
| `ahu-typical`       | Typical air handling unit after Azem (2018), 10,000 m³/h with heat recovery        |
| `office-supply-72`  | TU Berlin HLK exercise 7.2, open-plan office supply; fan pressure 828.61 Pa        |

## Testing

In the app: *Help › Run self-tests*, or open `luka.html?selftest`.

Headless with Node 18+:

```bash
node -e "const h=require('fs').readFileSync('luka.html','utf8');
  for (const id of ['luka-core','luka-io','luka-tests'])
    eval(h.split('<script id=\"'+id+'\">').pop().split('</script>')[0]);
  for (const t of LukaTests.runTests()) console.log(t.ok?'PASS':'FAIL', t.name);"
```

Visual check: open `?example=office-controlled`. The status bar should show *Converged* and a required fan pressure of about 370.5 Pa.

## Architecture

Everything lives in `luka.html`, split into clearly marked script blocks:

| Block            | Contents                                                                                      |
| ---------------- | --------------------------------------------------------------------------------------------- |
| `luka-core`      | Fluid properties, friction, correlations, component registry, network assembly, solver. DOM-free; also runs as a Web Worker and in Node. |
| `luka-io`        | Project format and migration, XLSX writer, file helpers                                       |
| `luka-tests`     | Self-tests                                                                                    |
| `luka-pip2json`  | Python converter for legacy `.pip` projects                                                   |
| `luka-ui`        | React UI (htm templates, React Flow canvas, panels, dialogs)                                  |
| Example blocks   | Embedded example projects as JSON                                                             |

The layering is strict: core knows nothing about IO or UI, IO uses core, and UI uses both. The header comment at the top of the file is the full developer documentation — please read it before contributing, including the *How to…* section on adding component types, result values or format changes.

## Known limitations

- Tee ζ values are constant inputs (no automatic ζ from flow ratios).
- No fan curves (fans are fixed-flow or sized) and no duct leakage.
- No part-load sweeps, system curves, ζ database, automatic duct sizing or snap-to-grid yet.
- Connections are straight lines without length or loss; use a duct component for that.

Contributions toward any of these are welcome.

## Credits

**HRI-Luka** — idea and supervision: Karsten Tawackolian and Tongyao Huang; software: Hendrik Berg, Joanna Hurnowicz, Xue Tian, Tongyao Huang (Hermann-Rietschel-Institut, TU Berlin).

Funded by the German Federal Ministry for Economic Affairs and Climate Action, research programme Luft-KonVerTeR (BMWi 03ET1606A).

Companion tool: [AHU-Life](https://ahulife.eonerc.rwth-aachen.de/).

## License

The original HRI-Luka is licensed under GPL-3.0. Luka is a derivative work and is distributed under **GPL-3.0-or-later**. Vendor libraries: React, ReactDOM and React Flow (MIT), htm (Apache-2.0).
