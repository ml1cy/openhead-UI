# Handoff: Open Headunit v4 — head unit UI

## Overview

A full-screen head unit application for a **plain Android tablet** (not Android Automotive OS) acting as the device launcher. It wraps the existing Android Auto projection capability of `ml1cy/open-headunit` in a new shell and adds radio, Bluetooth media, telephony, navigation hand-off, vehicle data, camera, settings and driver profiles.

Designed at **1920×1080 landscape**, near-black surfaces with a single amber accent, a persistent left dock, and a pull-down quick-settings drawer.

## About the Design Files

The files in `design/` are **design references authored in HTML** — a prototype showing intended look and behaviour. They are **not production code to port**. The task is to recreate these screens natively in the Android app using its own patterns.

- `design/Open Headunit v4.dc.html` — the new design, all nine screens, clickable.
- `design/Open Headunit - Current UI.dc.html` — recreation of the app's UI as it exists today, for before/after comparison.
- `design/support.js` — runtime for the HTML prototypes only. Ignore it.

Open either HTML file directly in a browser. Click the dock, the tiles, the source tabs, the presets, the profile chip (top right) to reach every state.

**Recommended target:** Kotlin + Jetpack Compose. The current repo uses fragments with XML layouts; this UI is state-driven enough (one now-playing model feeding several screens, live tuning, source switching) that Compose will be markedly less work than XML. If Compose is off the table, the layout descriptions below still map onto ConstraintLayout, but expect more plumbing.

## Fidelity

**High fidelity.** Colours, type sizes, weights, spacing, radii and interaction states below are final and exact. Recreate them faithfully. Content strings are placeholder data — the shapes and lengths matter, the words do not.

---

## Canvas & shared frame

Every screen sits inside the same frame.

- Canvas: **1920×1080**, `overflow: hidden`, background `--bg`.
- **Left dock**: 132px wide, full height, background `--surface`, 1px right border `--line2`, padding 26px top / 22px bottom, items centred, 10px gap.
  - Brand mark: 46×46, radius 13px, background `--accent`, car glyph in `#12130F`, 14px bottom margin.
  - Nav buttons: 88×82, radius 18px, icon 27px above a 12px/700 label (letter-spacing .4px, font-stretch 88%). Idle `--muted`; hover background `--surface2`; **active** background `--soft`, foreground `--accent`, plus a 5×34px accent bar pinned to the left edge of the dock, radius 0 5px 5px 0.
  - Order: Home, Auto, Radio, Phone, Camera — then flex spacer — Settings. Settings also reads active while the Profiles screen is open.
- **Content column**: flex 1, padding 22px 32px 26px, 20px gap between rows.
- **Status bar**: 56px tall, one row.
  - Left: clock 36px/700 font-stretch 105% letter-spacing -.6px tabular-nums, then date 15px/500 `--muted`, 14px gap.
  - Then 1px × 24px `--line` dividers between: outside temperature (18px icon + 15px/500 text) and GPS status.
  - Right: Bluetooth, Wi-Fi and 4-bar signal glyphs at 20px in `--muted` (bars 4px wide, 6/10/14/18px tall, last bar `--faint`), 16px gap.
  - Far right: **profile chip** — 48px tall pill, radius 26px, background `--surface`, 1px `--line2`, containing a 34px accent avatar circle with the initial in `#12130F` 14px/700, the profile name 15px/600, and an 18px chevron in `--muted`. Tapping it opens the quick-settings drawer.

### Quick-settings drawer

Anchored top-right of the canvas, **720px wide**, background `--surface`, 1px `--line`, radius `0 0 28px 28px`, padding 28px 32px 34px, `--shadow`. Slides via `transform: translateY(-115% → 0)`, **260ms cubic-bezier(.2,.8,.3,1)**. A `rgba(0,0,0,.5)` scrim covers the canvas while open; tapping it or the ✕ closes.

Contents top to bottom:
1. Row: 56px accent avatar, name 24px/700 + "Active profile" 15px `--muted`, "Switch" button (48px pill, 1px `--line`) → Profiles screen, ✕ button (48px square pill).
2. **Volume** row, 30px top margin: 26px speaker icon, −/+ buttons (56px square, radius 16px, `--surface2`, 26px glyph), a 14px track radius 7px `--surface2` with an accent fill, and the value right-aligned in a 56px box at 24px/700 tabular-nums. Steps of 4, clamped 0–100.
3. **Brightness** row, same construction, 20px top margin. Steps of 6, clamped 10–100.
4. **Toggle tiles**, 4-up grid, 14px gap, 28px top margin: Night/Day, Wi-Fi, Bluetooth, Do not disturb. Each 96px tall, radius 20px, 26px icon over a 14px/600 label. Off = 1px `--line` border, transparent background, `--muted`. On = no border, `--soft` background, `--accent` foreground.

---

## Screens

### 1. Home

Rows: main grid (flex 1) then a 132px vehicle strip, 20px gap.
Main grid: `1fr / 716px`, 20px gap.

**Left column** — hero (flex 1) over a 172px recents row.

*Now-playing hero*: `--surface`, 1px `--line2`, radius 24px, padding 30px, 30px gap, `--shadow`.
- Art: 272×272, radius 20px, `linear-gradient(145deg,#FF7A18,#B3350C 62%,#3A1405)` with a `radial-gradient(circle at 74% 22%, rgba(255,255,255,.30), transparent 58%)` sheen. **Placeholder** — replace with real album/station art.
- Right: source label 12px/700 letter-spacing 1.6px uppercase `--accent` font-stretch 88%, a 5px `--faint` dot, "NOW PLAYING" in `--muted`. Title 46px/700 font-stretch 108% letter-spacing -1px, 14px top margin, single line ellipsised. Subtitle 22px/500 `--muted`, 10px top margin, ellipsised.
- Transport, bottom-aligned: prev 64px circle `--surface2`, play/pause 78px circle `--accent` with `#12130F` glyph, next 64px circle. Then a 1px × 44px divider, an "Open source" pill (56px tall, radius 28px, 1px `--line`) that routes to the active source's screen, spacer, then a speaker icon + volume value 20px/600 tabular-nums.

*Recents row*: two equal cards, 20px gap. Each `--surface`, 1px `--line2`, radius 22px, padding 22px 24px, space-between column. Eyebrow 12px/700 uppercase letter-spacing 1.5px `--muted`; title 28px/700 font-stretch 104%; sub 16px `--muted`.
- Left: "LAST STATION" / "Radio Nova" / "101.5 FM · Alternative" → Radio screen.
- Right: "RECENTLY PAIRED" with a green connected badge (7px dot + 13px/600 `--good`) on the eyebrow row / "Pixel 8 Pro" / "Bluetooth · Media, calls, contacts" → Settings ▸ Connectivity.

**Right column** — 3×2 tile grid, 20px gap. Each tile `--surface`, 1px `--line2`, radius 22px, padding 24px, space-between column, left-aligned. Icon container 60px, radius 17px, 30px glyph. Title 22px/700 font-stretch 102%; sub 14px `--muted`, 4px top margin.
- Android Auto (icon container `--soft` / `--accent`; sub reflects connection state), Radio, Media, Phone, Navigation, Camera (all `--surface2` / `--text`).

**Vehicle strip**: 132px, `--surface`, 1px `--line2`, radius 22px, padding 0 30px, grid `repeat(5,1fr) auto`, 30px gap, whole strip tappable → Vehicle screen. Each metric: 12px/700 uppercase label, then value 40px/700 font-stretch 110% letter-spacing -1px tabular-nums + unit 16px/600 `--muted`. Metrics: Speed 0 km/h · Battery 14.2 V · Coolant 89 °C · Fuel 62% · 430 km · Trip B 128 km · 6.4 L. Trailing "All vehicle data ›" in 15px/600 `--muted`.

### 2. Radio

Column: 54px source tab row, then a `1fr / 430px` grid, 18px gap.

**Source tabs**: FM, AM, DAB+, Internet, Bluetooth. Each 54px tall, radius 27px, padding 0 28px, 1px `--line`, `--surface`, 17px/600. Active: background `--accent`, text `#12130F`, border accent. Right side: "Scan"/"Scanning…" (or "Refresh" on non-tunable sources) and "Auto-store", both 54px outlined pills.

**Left column**, three stacked cards, 18px gap:

*Hero card* — `--surface`, radius 24px, padding 26px 30px, `--shadow`. 150×150 art (same gradient placeholder), then source label + metadata tag (RDS / Stream / A2DP), title 40px/700 font-stretch 106%, subtitle 19px `--muted`. Right-aligned readout: **76px/800 font-stretch 118% letter-spacing -2.6px tabular-nums**, with a 16px/600 uppercase unit under it.
- FM → `101.5` / MHz. AM → `720` / kHz. DAB+ → ensemble id / ENSEMBLE. Internet → stream index / STREAM. Bluetooth → `2/3` queue position / QUEUE.

*Tuning card* — `--surface`, radius 24px, padding 24px 30px 26px. Header: context label left (e.g. "Tuning · 87.5 – 108.0 MHz", "Now playing · A2DP from Pixel 8 Pro"), signal meter right.
- **Tunable sources (FM, AM)**: a 96px band, click-to-tune. Ticks rise from 26px above the baseline — minor 15px at .18 opacity, half 26px at .3, major 40px at .55, all 2px wide `--text`. Major ticks carry a 14px/600 `--muted` label at the baseline. Known stations show as 10px accent dots 70px up (full opacity when tuned, .38 otherwise). A 3px accent needle with a triangular cap is pinned at 50%; **the band scrolls under the needle** rather than the needle moving. FM window ±3.2 MHz, 0.1 steps; AM window ±108 kHz, 9 kHz steps.
- **Streaming sources (DAB+, Internet, Bluetooth)**: the ruler is replaced by an 8px progress track (radius 4px, `--surface2`, accent fill) with elapsed/duration 15px/600 tabular-nums below. Bluetooth shows real position (38% · 1:24 / 3:47); DAB and Internet show a full bar with "On air" / "Live".
- Controls row, 14px gap, 8px top margin: prev 82×66 radius 18px `--surface2`; then either **− / +** (66px squares, tunable sources) or **play/pause** (66px, accent); then next 82×66; spacer; then a "Add favourite" / "In favourites" outlined pill, 66px tall, with a star that fills accent when set.

*Presets card* — flex 1, `--surface`, radius 24px, padding 22px 26px. Header label changes per source: FM presets / AM presets / DAB services / Saved streams / On this phone. Grid: 4 columns, auto rows, 14px gap, each cell min 96px, radius 18px, 1px `--line`, `--surface2`, padding 16px 18px. Top row of the cell: "P1" in 11px/700 letter-spacing 1.4px `--faint` and an 8px favourite dot. Bottom: primary 24px/700 font-stretch 106% tabular-nums (frequency, or service name on non-tunable sources) and secondary 14px `--muted`. Active cell: accent border, `--soft` background.

**Right rail (430px)** — `--surface`, radius 24px. Header row (12px/700 uppercase label + count in 13px/600 `--faint`), then a scrolling list. Rows: 14px 12px padding, radius 16px, 16px gap — 52px gradient art square (radius 14px), name 18px/600 over sub 14px `--muted`, then a 4-bar signal meter. Active row background `--soft`.

### 3. Android Auto

Three states.
- **Idle**: centred 820px card, `--surface`, radius 28px, padding 56px, `--shadow`. 96px `--soft` icon tile, title 38px/700 font-stretch 106%, body 19px `--muted` capped at 560px, then two 70px pills — "Start wireless · Pixel 8 Pro" (accent) and "Use USB cable" (outlined).
- **Connecting**: 110px ring, 4px `--line` with an accent top segment, spinning 1s linear; title 32px/700; sub 18px `--muted`. Auto-advances after 1.8s in the prototype.
- **Projecting**: a status row — accent `--soft` pill with a pulsing 8px dot reading "Projecting from Pixel 8 Pro", link details in 15px `--muted`, and a "Disconnect" outlined pill — above a full-bleed frame (radius 24px, `--surface2`, 1px `--line2`) hatched with `repeating-linear-gradient(135deg, transparent 0 22px, var(--line2) 22px 23px)`.

**This frame is where the real projection surface goes.** Nothing of the head unit's own UI should be drawn inside it.

### 4. Phone

Grid `1fr / 560px`, 20px gap.
- **Recents** (left): `--surface`, radius 24px. Header "RECENT CALLS" + "Synced from Pixel 8 Pro" 13px/600 `--muted`. Rows: 18px 16px padding, radius 18px, 20px gap — 54px circular avatar (`--surface2`, initial 20px/700 `--muted`), name 21px/600, meta 15px (missed calls in `#FF6B5A`), timestamp 15px `--faint`.
- **Dialpad** (right): `--surface`, radius 24px, padding 28px. 76px readout, 46px/700 font-stretch 106% letter-spacing 1px tabular-nums, centred; shows "Enter a number" when empty. Then a 3-column grid, 14px gap, keys radius 20px `--surface2` min-height 78px — digit 30px/600 over letters 11px/700 letter-spacing 1.6px `--faint` (13px reserved height so rows align). Bottom row: 96px backspace (`--surface2`) and a flex-1 call button, 78px, radius 20px, background `--good`, text `#06240F`, 20px/700.

### 5. Camera

Flex row, 20px gap.
- **Viewport** (flex 1): radius 24px, 1px `--line2`. Feed placeholder is a horizon gradient with scanlines. Overlays: top-left REC badge (44px pill, `rgba(8,9,11,.72)`, blur 8px, pulsing red dot, "REC · Dashcam loop 3 min"); top-right "R · Reversing".
- **Guidelines**: an SVG overlay in a 1000×620 viewBox — a green swept corridor (4px), two faint 3px extension lines, and three 7px distance bars: green at y550, amber `#FFC53D` at y440, red `#FF4D3D` at y340. Toggleable.
- **Right rail (340px)**: distance card (label, 52px/700 font-stretch 112% value in `#FFC53D` + unit, five 12px segment bars — red, amber, then three `--surface3`); a guidelines row with a 64×36 switch (radius 18px, 4px padding, 28px white knob, accent when on, `--surface3` when off); a 2×2 source grid (Rear active, Front, Cabin, Split — 60px cells, radius 16px, 1px `--line`); spacer; and a 76px "Save last 30 s" button.

### 6. Navigation

Grid `1fr / 470px`, 20px gap.
- **Map** (left): radius 24px, `--surface2`. 96px grid lines from two `--line2` gradients, roads drawn as thick `--line` strokes, the route as a 12px accent polyline with a 15px accent destination dot and a 17px `--good` origin dot ringed in `--surface2`. **Placeholder** — a real map view replaces the whole element.
- Floating manoeuvre card, top-left, 20px padding, radius 20px, `--surface`, `--shadow`: 46px accent turn arrow, "400 m" 34px/700, street name 17px `--muted`.
- **Right**: a 74px search field (radius 22px, magnifier + "Search a destination" 19px `--muted`) over a recent-destinations list — rows 18px 14px, 46px icon tile radius 14px, name 19px/600, sub 14px `--muted`, ETA 16px/600 `--accent`.

### 7. Vehicle

Four gauge cards across the top (flex 1, 20px gap), then a 300px two-column row.
- **Gauge card**: `--surface`, radius 24px, padding 28px. 12px/700 uppercase label; centred value **82px/800 font-stretch 114% letter-spacing -3px tabular-nums** with a 22px/600 `--muted` unit; a 10px track (radius 5px, `--surface2`) with a coloured fill; a 14px `--muted` note. Speed 0 km/h (0%, "Parked · handbrake on"), Coolant 89 °C (62%), Battery 14.2 V (84%, `--good` fill, "Alternator charging"), Fuel 62% (62%, "430 km to empty").
- **Tyre pressure**: 2×2 grid of 16px 20px rows, radius 16px, `--surface2` — position 16px/600 `--muted`, value 20px/700 tabular-nums. Out-of-range values in `#FFC53D` (rear right 1.9 bar).
- **Trip B**: three figures at 46px/700 font-stretch 110% with 15px `--muted` captions, plus an info row (radius 16px, `--surface2`, 15px `--muted`) noting the OBD source.

### 8. Settings

Grid `330px / 1fr`, 20px gap.
- **Category rail**: `--surface`, radius 24px, padding 18px, 6px gap. Rows 70px, radius 16px, padding 0 22px, 18px/600 — an 8px dot (accent when selected, `--faint` otherwise) then the label. Selected row: `--soft` background, `--accent` text. Version block at the bottom, 13px `--faint`, line-height 1.6.
- **Detail pane**: `--surface`, radius 24px, padding 30px 34px. Title 30px/700 font-stretch 106%. Rows: 24px vertical padding, 1px `--line2` bottom border, 24px gap — label 20px/600 over sub 15px `--muted`, and on the right either a 64×36 switch or a 18px/600 `--muted` value.
- Categories and their rows are enumerated in `CONTENT.md`. Display ▸ Night theme and Connectivity ▸ Wi-Fi/Bluetooth are wired live to the same state the quick-settings drawer uses.

### 9. Driver profiles

Header (32px/700 title + 17px `--muted` sub), then three equal cards, 20px gap. Each `--surface`, radius 24px, padding 30px, 1px border — `--accent` when active, `--line2` otherwise. 72px avatar circle (accent with `#12130F` initial when active, `--surface2` with `--muted` otherwise), name 28px/700 font-stretch 104%, last-drive line 15px `--muted`. Below: wrapped chips, 9px 16px, radius 16px, `--surface2`, 14px/600 `--muted`. Footer button 64px, radius 18px, 17px/700 — accent "Switch to <name>" when inactive, `--surface2`/`--muted` "Active profile" when active.

---

## Interactions & behaviour

| Trigger | Result |
|---|---|
| Dock item | Switches screen. Settings stays lit on the Profiles screen. |
| Home tile / recents card | Routes to the matching screen; Media opens Radio with the Bluetooth source selected; the paired-phone card opens Settings ▸ Connectivity. |
| Vehicle strip | Opens the Vehicle screen. |
| Profile chip | Opens the quick-settings drawer. Scrim tap or ✕ closes. |
| Radio source tab | Switches source and resets the list selection; changes the readout, tuning card, preset grid and rail together. |
| Ruler click | Tunes to the frequency under the pointer, quantised (0.1 MHz / 9 kHz). |
| − / + | One step. Seek ◀ ▶ jumps to the next known station, wrapping at the band edge. |
| Prev / next on streaming sources | Steps the list index, wrapping. |
| Preset / rail row | Tunes or selects. |
| Scan | Shows "Scanning…" for 1.6s then lands on a station. |
| Favourite | Toggles, keyed per source (`fm:101.5`, `dab:Nova DAB`) so a DAB favourite never writes an FM frequency. |
| Android Auto connect | Idle → connecting (1.8s) → projecting. Disconnect returns to idle. |
| Dialpad | Appends up to 14 characters; backspace removes one. |
| Guidelines switch | Shows/hides the camera overlay. |
| Day/Night, Wi-Fi, BT, DND | Toggle immediately; the theme swaps the whole token set. |
| Profile switch | Sets the active profile, reflected in the status chip and drawer. |

Transitions: dock and tiles 180ms background/colour; drawer 260ms `cubic-bezier(.2,.8,.3,1)`; spinner 1s linear; pulses 1.6–1.8s ease-in-out. Everything else is instant. Respect a "reduce motion" preference by dropping all of it.

Hover states are documented because the prototype runs in a browser — on a touch panel, implement them as **pressed** states instead.

## State

```
screen        home | radio | auto | phone | cam | nav | vehicle | settings | profiles
theme         night | day                     (also derivable from light sensor / sunset)
qs            drawer open
src           fm | am | dab | net | bt
freq          Float, 87.5–108.0, step 0.1     (FM)
amFreq        Int, 522–1620, step 9           (AM)
pick          Int index into the current list (DAB / Internet / Bluetooth)
playing       Boolean
scanning      Boolean
favs          Set<String>, keys "<src>:<id>"
volume        Int 0–100, step 4
brightness    Int 10–100, step 6
profile       String
dialed        String, max 14
guides        Boolean
setTab        display | sound | conn | radio | vehicle | profiles | system
wifi bt dnd   Boolean
```

The "current item" is derived, never stored: on FM/AM it is the station nearest the tuned frequency within tolerance (0.25 MHz / 8 kHz) or null; on the other sources it is `list[pick]`. The home hero and the radio hero read from the same derived value — build it once.

## Design tokens

### Night (default)
| Token | Value |
|---|---|
| bg | `#08090B` |
| surface | `#131519` |
| surface2 | `#1C1F25` |
| surface3 | `#262A31` |
| line | `rgba(255,255,255,.10)` |
| line2 | `rgba(255,255,255,.055)` |
| text | `#F3F4F6` |
| muted | `#8C919B` |
| faint | `#5A5F69` |
| accent | `#FF7A18` |
| accent soft | `rgba(255,122,24,.15)` |
| shadow | `0 18px 50px rgba(0,0,0,.55)` |

### Day
| Token | Value |
|---|---|
| bg | `#E8E9EC` |
| surface | `#FFFFFF` |
| surface2 | `#F1F2F5` |
| surface3 | `#E2E4E9` |
| line | `rgba(15,18,25,.12)` |
| line2 | `rgba(15,18,25,.07)` |
| text | `#14171C` |
| muted | `#666C77` |
| faint | `#989EA9` |
| accent soft | `rgba(255,122,24,.14)` |
| shadow | `0 14px 40px rgba(20,24,32,.14)` |

Accent is constant across themes. Foreground on accent is always `#12130F`.

### Semantic
success `#3FD07A` (on-accent text `#06240F`) · warning `#FFC53D` · danger `#FF4D3D` · missed call `#FF6B5A`

### Typography

**Archivo**, variable, width axis 75–125, weight 400–800. Ships under the SIL Open Font License; bundle the variable TTF rather than relying on Google Fonts at runtime.

| Role | Size / weight / width | Notes |
|---|---|---|
| Frequency readout | 76 / 800 / 118 | letter-spacing -2.6px, tabular |
| Gauge value | 82 / 800 / 114 | letter-spacing -3px, tabular |
| Hero title | 46 / 700 / 108 | letter-spacing -1px |
| Dialpad readout | 46 / 700 / 106 | letter-spacing 1px, tabular |
| Trip figure | 46 / 700 / 110 | tabular |
| Radio hero title | 40 / 700 / 106 | |
| Vehicle strip value | 40 / 700 / 110 | tabular |
| Clock | 36 / 700 / 105 | tabular |
| Screen title | 30–32 / 700 / 106 | |
| Card title | 28 / 700 / 104 | |
| Preset primary | 24 / 700 / 106 | tabular |
| Tile title | 22 / 700 / 102 | |
| List primary | 18–21 / 600 / 100 | |
| Body / value | 15–17 / 500–600 / 100 | |
| Caption | 13–14 / 500 / 100 | `--muted` |
| **Eyebrow label** | 12 / 700 / 88 | letter-spacing 1.5px, uppercase, `--muted` |
| Preset index | 11 / 700 / 100 | letter-spacing 1.4px, `--faint` |

All numeric readouts use tabular figures so they do not jitter while tuning.

### Geometry
Radii 13 / 14 / 16 / 17 / 18 / 20 / 22 / 24 / 26–28 / pill / circle. Spacing steps 4 / 6 / 8 / 10 / 12 / 14 / 16 / 18 / 20 / 22 / 24 / 26 / 28 / 30 / 32. Grid gap is 20px almost everywhere; 14px inside dense grids.

**Touch targets**: nothing interactive is under 44px. Dock buttons are 88×82, transport 64–78px, source tabs 54px, presets 96px, keypad keys 78px. Preserve this — it is the difference between usable and dangerous at speed.

## Assets

Nothing is licensed or copied in. Everything is drawn.

- **Icons**: 24px-grid line icons, 1.7px stroke, round caps and joins, `currentColor` — home, car, radio, phone, camera, gear, note, navigation arrow, Bluetooth, Wi-Fi, volume, star, backspace, search, pin, turn arrow, info, thermometer. Swap for your icon set as long as the weight and 24px grid match.
- **Station and album art**: amber gradient placeholders. Replace with real art; keep the radius and size.
- **Camera feed and map**: CSS/SVG placeholders standing in for a real `SurfaceView` and map view.

## Files

```
design/Open Headunit v4.dc.html          the design — all nine screens, clickable
design/Open Headunit - Current UI.dc.html  today's UI, for comparison
design/support.js                        prototype runtime, not for porting
ANDROID_IMPLEMENTATION.md                platform notes: APIs, hardware, privilege requirements
CONTENT.md                               all placeholder data and copy
```
