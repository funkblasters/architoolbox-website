# ArchiToolbox — Strategy Notes

## The idea

A collection of small desktop tools that solve painful, repetitive, Italian-specific problems
for architects, geometri, and engineers. Each tool is standalone and works without the others.
A lightweight launcher app (ArchiToolbox) optionally ties them together for update management
and cross-promotion — think JetBrains Toolbox, much simpler.

---

## Value proposition

**Not selling software. Selling time and certainty.**

The target user is not a power user who enjoys configuring tools. They are a professional who
hits the same bureaucratic wall repeatedly, loses an afternoon, and just wants it solved.
The value is crystallized expertise: the correct workflow, in the correct order, with the
known edge cases already handled. A script that "mostly works" is not a substitute.

The secondary value: **peace of mind at submission time.** A professional who has submitted
50 clean files with the same tool stops questioning it. That trust is hard for a competitor
to displace.

---

## Market

Italian technical professionals submitting to:
- **Agenzia delle Entrate** — DOCFA (cadastral updates), Pregeo (topographic surveys)
- **Comuni** — CILA, SCIA, SUE (building permits and notifications)
- **Agenzia delle Entrate / Entratel** — via Desktop Telematico

The non-regular submitter is the best customer. They do DOCFA once a year, remember the
pain vividly, have no established workflow to defend, and €10 is nothing compared to the
afternoon they'd otherwise waste.

---

## Moat

Not technical — the algorithms are simple. The moat is:

1. **Domain knowledge encoded as working software.** The pipeline order, the edge cases,
   the Italian-specific rules. Getting it all right takes months of real-world testing.
2. **Trust built on correctness.** Once someone processes 50 files and every submission goes
   through first time, they don't switch.
3. **Breadth over time.** One tool is easily copied. 6–8 tools under a shared brand,
   all solving Italian-specific problems, is much harder to replicate.
4. **Word of mouth in a small professional community.** Geometri and architects talk to each
   other. One evangelist in a studio reaches the whole network.

---

## Distribution strategy

- **Word of mouth** as primary vector — the audience is small and connected
- **CFP/CPD courses** (crediti formativi professionali) as a secondary channel:
  - Architects, engineers, geometri need to log training hours every year
  - A course on "the correct DOCFA/digital workflow" has built-in demand
  - The tool is the backbone of the course; the course builds trust in the tool
  - Everyone who enrolls is a warm lead; the enrollment list is the most valuable asset

---

## Business model

**Per-tool, one-time purchase, low price (€5–10).**

- Simple enough for a solo developer with no infrastructure overhead
- Matches how these professionals buy tools
- Payment via Gumroad or Lemon Squeezy — handles EU VAT automatically, no fixed cost,
  percentage only on sales

**When there are 3+ tools:** flat annual ArchiToolbox subscription (€8–12/month) gives
access to everything and funds ongoing maintenance. The "spec update" angle justifies
recurring pricing: DOCFA rules can change, subscribers get the fix automatically.

**The course** is the higher-revenue line. €150–200/seat × 20 architects = €3,000–4,000
per run. The tool is the reason they find you; the course is where the money is.

Credits, freemium tiers, and per-conversion pricing are wrong models here:
the "cost" per conversion is local compute on the user's machine. There is nothing to
meter and no server to protect.

---

## Infrastructure (intentionally minimal)

The desktop-first architecture is a deliberate business decision, not just a technical one:

- **Zero per-user cost** — local compute, no server, no scaling problem
- **No privacy concern** — floor plans and cadastral data never leave the machine
- **Works offline** — no dependency on a third-party service going down

What's actually needed:
- **Version check**: one static JSON at `https://architoolbox.io/releases/<tool>/latest.json`,
  fetched on startup, shows a banner if outdated. No server, no database.
- **Distribution**: GitHub Releases for binaries. Gumroad/LemonSqueezy for payment + delivery.
- **Subscription (future)**: Supabase free tier handles auth and entitlement check until
  there are hundreds of paying subscribers.

### Download URL redirect pattern

Each tool's `latest.json` contains a `download_url` pointing to `https://architoolbox.io/download/<tool>`.
That URL is a simple redirect (301) to the actual Gumroad/LemonSqueezy page.

```
App update banner
  → https://architoolbox.io/download/dxfcleaner  (you control this forever)
    → https://yourname.gumroad.com/l/dxfcleaner   (update this redirect if you switch platforms)
```

Why the indirection:
- The `openExternal` IPC handler in each Electron app is whitelisted to `architoolbox.io` only,
  so even a compromised `latest.json` cannot redirect users to a third-party URL directly.
- If you ever switch payment platforms, you update one redirect — no app update needed.
- The canonical download URL is yours permanently, independent of Gumroad/LemonSqueezy.

Total fixed monthly cost: **~€0** until subscription revenue justifies Supabase (~€25/month).

---

## ArchiToolbox launcher

A small system-tray app, entirely optional. If the user adopts it:
- Sees all installed tools with current vs. latest version
- One-click update per tool
- "New tools available" section — organic cross-promotion at zero cost

How it works locally:
- Each tool writes `~/.architoolbox/tools/<toolname>.json` on install: `{"version": "x.y.z"}`
- Launcher reads all those files, compares against a hosted `toolbox.json`
- Updates = download new installer, run it, update local JSON

No shared database, no IPC between tools, no registry hacks. Buildable in a week
once two tools exist.

---

## Tool #1: DXF Cleaner per DOCFA (exists)

Prepares a DXF from AutoCAD/BricsCAD for DOCFA submission:
- Sets DXF version to AC1015
- Flattens to Z=0
- Consolidates layers (everything → layer 0, preserves DOCFA_POLIGONI and RIQUADRO)
- Removes XREFs, explodes blocks, converts MTEXT→TEXT
- Closes open polylines within tolerance
- Removes annotations (HATCH, DIMENSION, ATTRIB, LEADER, etc.)
- PURGE + OVERKILL
- Centers on origin
- Validates DOCFA_POLIGONI (closed LWPOLYLINEs, color→surface type A–G mapping)

**Known pain point documented on:** GeoLIVE, TopGeometri, professionearchitetto.it,
geometra-roma.com. Professionals actively search for guides on this exact workflow.

---

## Potential future tools

### Ranking by effort vs. signal

| Tool | Effort | Signal | Obstacle |
|---|---|---|---|
| PDF/A Converter | Low | High | None |
| CXF → DXF/PDF overlay | Medium | High | None |
| DOCFA Area Validator | Medium | High | None |
| CILA/SCIA Assembler | Medium | High | Per-comune variation |
| Bulk Catasto Fetcher | Medium | High | ToS risk |
| PRG Zone Checker | High | Very high | Data fragmentation |

---

### Tool #2 candidate: PDF/A Converter & Validator ⭐

**The problem:** Every SUE/SCIA/CILA submission requires PDF/A format. Many practice
rejections happen because the professional produced a normal PDF and the portal rejected
it silently or with a cryptic error. The Salva Casa decree (fully in force May 2025)
reshuffled the entire modulistica, making this a fresh and recurring pain.

**The tool:** Drag in PDFs, get PDF/A-compliant output with a compliance badge. Validates
existing PDFs too. Under the hood: LibreOffice or Ghostscript, both free and local.

**Why build this next:** Smallest scope in the catalog. Zero infrastructure. Zero
Italian-specific logic needed in the engine — the value is the context (bureaucratic
submissions) and the simplicity. Fits ArchiToolbox naturally. Buildable in days.

---

### Tool #3 candidate: CXF → Usable Output (no QGIS required)

**The problem:** QGIS plugins for CXF conversion exist (Cxf_in, CXF to Shape Vestito,
CatastoIT_GML_Merger_Pro) but require installing and learning QGIS. The typical geometra
wants to download their CXF from the Agenzia delle Entrate portal, drop it in a tool, and
get out either a clean DXF overlay or a printable PDF with parcels on a satellite basemap.
That last step — from "I have the CXF" to "I have something I can use in a report" — is
currently a QGIS tax that non-GIS professionals cannot or will not pay.

**The tool:** Drop in a CXF file → choose output (DXF overlay, georeferenced PDF with OSM
basemap, or shapefile) → done. No GIS knowledge required.

**Why build this:** The QGIS plugins prove the demand; the gap is the UX. Same pattern
as DXF Cleaner: the algorithm exists and is open, the value is making it accessible.

*Sources: QGIS plugin repository, pigreco/workshop-estate-gis-2021 on GitHub.*

---

### Tool #4 candidate: PRG Zone Checker (harder, higher ceiling)

**The problem:** Daily workflow — a client asks "posso fare X on my parcel?" You need to
look up the PRG, find the zona omogenea, extract indici urbanistici (IF, RC, H max,
destinazione d'uso), cross-reference with catasto to confirm the parcel. Currently this
means navigating a comune's geoportal (if one exists), downloading PDFs or shapefiles,
opening them in something, hunting for the parcel. The Salva Casa decree made PRG lookups
even more frequent because tolleranze and sanatorie depend on zone classification.

**The tool:** Input comune + foglio/particella (or click a point on a map) → see the PRG
zone highlighted → read the relevant indices and regulations. Think Google Maps but for
urban planning rules.

**Technical path:** Static web app (no server costs) that hits public WMS endpoints
from comuni geoportals directly from the browser, overlaid on Leaflet + OSM. Start with
20 major comuni that expose machine-readable WMS (Torino, Milano, Roma, Bologna, etc.).

**The obstacle:** No national standard for PRG data. Most small comuni publish PDFs or
nothing at all. Coverage will always be partial. Needs a curated list of supported comuni
maintained over time — that's ongoing content work, not just engineering.

**Why it matters anyway:** Even 20 major comuni covers a large fraction of Italian
professional activity. The tool could ship as a web app (unlike the others) since it
doesn't process local files — just reads public endpoints.

---

### Tool #5 candidate: DOCFA Area Validator

**The problem:** On 64-bit Windows, DOCFA imports a clean DXF but silently fails to
calculate `area dichiarata`, blocking the entire 1N form. The user has no idea why.
This is a documented bug on GeoLIVE forum with many "anche io" replies.

**The tool:** Pre-validate polyline geometry, compute DOCFA surface areas by color
(A–G categories), export a summary the professional can cross-check before submission.
Catches the issue before it wastes a trip to the cadastral office.

*Source: GeoLIVE forum, documented Windows 64-bit DOCFA bug.*

---

### Tool #6 candidate: CILA/SCIA Document Assembler

**The problem:** Every practice submission = a specific set of documents in a specific
order, consistently named, all in PDF/A, many needing a digital signature. Professionals
assemble this manually from scattered files every single time.

**The tool:** Define document slots for a practice type (relazione tecnica, planimetria
stato attuale, planimetria progetto, attestazione conformità…), drag files in, auto-rename
to the comune's naming convention, validate PDF/A compliance, export a ready-to-upload ZIP.

**The obstacle:** Naming conventions vary significantly by comune. Needs to ship with
templates for major comuni and let users define custom templates — ongoing content burden.
Build PDF/A Converter first; this tool can absorb it as a dependency.

---

### Tool #7 candidate: Bulk Catasto Data Fetcher

**The problem:** Handling a condominium with 20 units means 20 manual downloads of
visure and planimetrie from the Sister/Agenzia delle Entrate portal, one by one.
The "download massivo" service exists but is designed for geographic bulk downloads,
not for fetching specific subalterni by list.

**The tool:** Input a CSV of foglio/particella/subalterno → batch-download all visure
and planimetrie → organized output folder.

**The obstacle:** Likely conflicts with Agenzia delle Entrate ToS for automated access.
Investigate before building.

---

### Tool #8 candidate: Pregeo / Digital Signature Debugger

**The problem:** Submitting libretti misure via Pregeo fails with cryptic errors
("Rejected by policy", signature validation failures). Diagnosing requires knowing
which certificate store, which Desktop Telematico version, which Entratel configuration.

**The tool:** Diagnose the local environment and surface a plain-language fix.

**Note:** Defensive tool, not a workflow tool. Lower monetization potential. Better
suited as free content (guide/checklist) that builds trust than as a paid product.

*Source: TopGeometri forum, recurring threads in 2025–2026.*

---

## What NOT to build

- **APE (Attestato Prestazione Energetica):** mela.work, regional SIPEE platforms,
  multiple commercial tools already cover this well. Crowded.
- **Computo metrico:** PriMus, ACCA, dozens of others. Complex domain, crowded.
- **Full GIS for catasto:** TopGeometri sells Geocat and CorrMap for this. Not worth
  competing head-on.

---

## What to build next (not yet decided)

Primary signal to look for: a forum thread where 30+ people reply "anche io" to a
complaint about wasting time on a specific repetitive task. That's the next tool.

Communities to monitor:
- [forum.topgeometri.it](https://forum.topgeometri.it)
- [geolive.org/forum](https://www.geolive.org/forum)
- [professionearchitetto.it/bacheca](https://www.professionearchitetto.it/bacheca)
- Facebook: "TECNICI FORUM - architetti geometri ingegneri periti"

---

*Notes from product/strategy conversation, April 2026.*
