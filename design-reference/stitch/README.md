# SAMUDERA Stitch Reference Pack

These files are **design references for Cline**, not production source code.

## Route mapping

- `dashboard.html` -> `/dashboard`
- `incidents.html` -> `/incidents`
- `incident-detail.html` -> `/incidents/[id]`
- `vessels.html` -> `/vessels`
- `cables.html` -> `/cables`
- `policies.html` -> `/policies`
- `observer.html` -> `/observer`

## Precedence

When a Stitch prototype conflicts with project contracts, follow this order:

1. `PRD.md`
2. `ARCHITECTURE.md`
3. `.clinerules/00-samudera.md`
4. `docs/IMPLEMENTATION_PLAN.md`
5. `DESIGN.md`
6. These Stitch reference files

`observer.html` is the **latest corrected Observer export** and supersedes the
obsolete Observer document that was embedded in the earlier combined export.

## How Cline should use these files

Cline is responsible for translating these prototypes into production React /
Next.js components.

It should preserve:
- layout hierarchy and proportions
- visual density
- dark SAMUDERA styling
- information grouping
- component placement
- status styling
- route-specific interactions described by DESIGN.md

It should NOT copy literally:
- per-screen Tailwind CDN configuration
- duplicate font imports
- duplicate Material Symbols imports
- `href="#"` links
- handcrafted SVG/mock map geometry
- remote placeholder images
- duplicated shell markup
- unsupported routes or buttons
- browser-side policy/physics calculations
- obsolete or corrected prototype text/data

## Mandatory production corrections

Use `DESIGN.md` as authority for corrections, including:
- Dashboard `TRACKED = 4`, not 142
- Dashboard `REPLAY-LINKED INCIDENTS = 4`, not `CABLES @ RISK = 4`
- Dashboard visible `DEMO TELCO • NOC OPERATOR`
- `/cables`: `Historical AIS Replay`, never `Live AIS`
- `/vessels`: `configured SEAX-1 cable buffer`, not `protected corridor`
- remove non-MVP `System Status` and `Settings`
- normalize internal topbar to 56px
- latest Observer amber `warning` / `warning-dark` tokens
