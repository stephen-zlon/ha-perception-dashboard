---
name: eink-home-dashboard
description: |
  Help someone design and build their own always-on e-ink home dashboard from scratch: a low-power display on the wall that shows weather, calendar, home-sensor data, chores, photos, or anything else, refreshed on a schedule by a small always-on computer. Use this whenever a person wants to "build an e-ink dashboard," "make a smart home display," "put my calendar/weather on an e-ink frame," "render a dashboard image and push it to a display," set up a "family wall display," or asks how to drive a color e-ink display — whether a finished network photo frame (e.g. BLOOMIN8 Spectra 6) or a Raspberry-Pi-driven e-paper panel (e.g. Pimoroni Inky Impression, Waveshare) — from Home Assistant or other data sources. Also use for the sub-problems: designing for a 6-color e-ink palette, dithering photos for e-ink, building an HTML-to-image render pipeline, or scheduling and hardening the whole thing so it survives unattended. Works regardless of the exact frame, computer, or data hub — it teaches the portable pattern and adapts to what the person has.
---

# Build Your Own E-Ink Home Dashboard

Guide a person through building an always-on e-ink dashboard: a wall display that quietly shows the day's weather, what's on the calendar, the state of the house, and whatever else matters to them — repainting itself every hour or so with no screen glare, no blue light, and almost no power.

This skill teaches a **portable pattern**, not one product. The person's exact frame, computer, and data sources will differ; help them map the pattern onto what they have (or help them choose). The end goal is a small pipeline they own and can extend for years: **data → HTML → image → the frame**, on a timer.

Work *with* the person. Ask what they already have before recommending purchases, build the smallest thing that renders first, and get one real image on the glass before adding features. A dashboard that shows three things reliably beats one that shows twelve and breaks weekly.

> **First, a gut check — is this the right project for this person?** Be honest with them: this is a hands-on, tinkerer's build. It means a little Python, a bit of Linux, standing up a data hub, and an afternoon or two of fiddling and troubleshooting. For the right person that *is* the fun. For plenty of others it's a chore they'll abandon. If what they actually want is the **outcome** — a calm display of their day on the wall — and not the project, point them straight at a **finished dashboard device like TRMNL** (a ~$139 7.5" e-ink dashboard with a big plugin library and weeks of battery). Yes, it's monochrome and you live inside its app rather than owning the stack — but it's a genuine, honorable off-ramp that delivers most of the payoff with almost none of the work. Recommend it without a hint of "you gave up." The goal is a display the person is happy to live with, not a merit badge for building it the hard way.

---

## What You'll Need

Walk the person through these four pieces. Only the first two are strictly required to start.

**1. An e-ink display that accepts a pushed image.** This is the wall piece. The important properties to establish:
- **Color vs. grayscale.** Modern color e-ink (the "Spectra 6" generation) shows six pigments; older/cheaper panels are black-and-white or black-white-red. Color changes how you design (see *Designing for E-Ink*).
- **How you get an image onto it — this splits displays into two camps, and the whole build bends around which one you have.** *Finished network frames* (e.g. BLOOMIN8 Spectra 6) expose a local endpoint you push a finished image to over HTTP — the simplest path, and the one this guide assumes by default. *DIY e-paper panels* (Pimoroni Inky Impression, Waveshare e-paper HATs) have no network of their own: you wire them to a Raspberry Pi, and that Pi is *both* the always-on computer (item 2) *and* the display driver, drawing to the panel with the vendor's Python library. Establish your camp early — it decides the "push" step (an HTTP POST vs. a local library call) and whether items 1 and 2 are two devices or one.
- **Resolution and orientation.** You render to the panel's exact pixel dimensions (e.g. 1200×1600 portrait). Get the number before designing.

**2. An always-on computer to render and push.** Something cheap that never sleeps: a Raspberry Pi, a mini PC, a NAS that runs containers, or a spare Mac Mini. Its whole job is to run a script on a schedule that builds the image and sends it to the frame. It needs Python and enough muscle to run a headless browser (a Pi 4 / Pi 5 is plenty).

**3. A data hub (strongly recommended: Home Assistant).** The dashboard is only as good as the data behind it. **Home Assistant** is the recommended spine because it already speaks to weather services, calendars, thermostats, and Zigbee/Matter sensors, and exposes all of it through one local API — so your render script makes one kind of call instead of ten. It's free and self-hosted. If the person doesn't want a hub, you can hit public APIs directly (a weather API, a calendar's iCal feed), but you'll wire each one by hand and maintain each one separately.

**4. (Optional) Smart-home sensors.** Anything they want on the display: indoor temperature, air quality, door/leak sensors, a thermostat. These flow in through the hub. Start with zero and add later — the dashboard is useful with just weather and a calendar.

### Picking a display

Buyable displays fall into four categories. This guide's pattern targets the first two; know all four so you can steer the person to the right one — **the deciding question is always "can I push my own image to it?"**

- **Finished network frame with an open push (best fit).** Accepts an arbitrary image over local HTTP or an API, so you push whatever you render. This is what the guide assumes. Examples: BLOOMIN8 (Spectra 6, HTTP upload); maker frames built on an ESP32-S3 controller (e.g. Good Display's Wi-Fi Spectra 6 frames, Seeed Studio's Spectra 6 kits) that you can point at your own image.
- **DIY e-paper panel + Raspberry Pi.** Pimoroni Inky Impression, Waveshare e-paper HATs. The Pi is computer and driver both, and "push" becomes a local library call. Great if they already own a Pi and don't mind a little wiring.
- **Consumer color e-ink photo frames (careful — often a dead end).** Aura Ink, Reflection Frame and similar are lovely Spectra 6 frames, but they're built to show *photos* through the maker's own app/cloud, and most won't let you push a custom dashboard image. Only pick one if you confirm it exposes a local upload or API; otherwise it's the wrong tool for a dashboard, however nice it looks.
- **Finished dashboard devices (the no-build option).** TRMNL is a battery-powered e-ink dashboard (7.5", currently monochrome) with a big plugin library and the option to point it at your own server — so someone who wants the *outcome* without building a pipeline can get most of the way there out of the box. Trade-offs: black-and-white today, and you live inside its framework rather than owning the whole stack. **This is the fallback to recommend whenever the rest of this guide sounds like more tinkering than the person wants** — it's the "I just want the result" answer, and it's a good one.

Rule of thumb: for a dashboard you control, buy for the **open push**, not the prettiest marketing shot. A frame that only speaks its own app will fight you forever.

### The cheapest way in

Two things drive the price: **color and size.** A small black-and-white panel can be a tenth the cost of a big six-color one — and the software (Home Assistant, the render tools) is free, while the always-on computer is often something you already own. The budget ladder:

- **Cheapest, "just prove the loop" — ~$55–120.** A small-to-mid **mono or tri-color** e-paper panel (e.g. a 7.5" Waveshare, ~$55) driven by a Raspberry Pi — ideally one you already have. Black-and-white and modest size, but it teaches the whole pipeline for pocket change.
- **No-build — ~$140–220.** A finished dashboard gadget: **TRMNL** ($139, 7.5" mono) or the larger **TRMNL X** ($219, 10.3"). You skip building the pipeline; the trade is monochrome and living inside their app.
- **Color DIY sweet spot — ~$155–200.** A **7.3" six-color** panel (Pimoroni **Inky Impression 7.3"**, ~$90) on a capable Pi (a Pi 5 2GB is ~$65). Full Spectra-6 color, you own the whole stack, moderate size.
- **The wall centerpiece — $275+ for the display alone.** A big **13.3" six-color** display. The DIY **Inky Impression 13.3"** is **$275** and still needs a Pi; a finished push-to frame (like the one in the reference build below) is a comparable outlay. This is the splurge tier.

On the computer: **$0 if you reuse** an old laptop, mini-PC, NAS, or a Pi from a drawer. If buying, a Raspberry Pi runs ~$15 (Zero 2 W) to ~$65 (Pi 5 2GB) — but the browser-render approach in this guide wants ~2GB of RAM to be comfortable, so favor a 2GB+ Pi or any spare computer over the tiny 512MB boards (those are fine for *driving* an Inky panel if you render elsewhere).

*(Prices are mid-2026 US ballparks and move — Raspberry Pi especially has been climbing with the memory shortage — so treat them as orders of magnitude, not quotes, and check before buying.)*

> Ask the person which of these they already have. The most common starting point is "a frame I just bought and a Raspberry Pi in a drawer," in which case skip straight to standing up Home Assistant and the render pipeline.

---

## How It Works (the pipeline)

Every build follows the same five-stage loop, run on a schedule:

```
  data hub / APIs        Python              headless browser         image            the frame
 ┌──────────────┐   ┌──────────────┐   ┌────────────────────┐   ┌────────────┐   ┌────────────┐
 │ weather,     │──▶│ fetch each   │──▶│ render HTML+CSS to │──▶│ crop/      │──▶│ HTTP push  │
 │ calendar,    │   │ source, fill │   │ an exact-size PNG/ │   │ dither if  │   │ or direct  │
 │ sensors      │   │ an HTML tmpl │   │ JPEG screenshot    │   │ needed     │   │ panel draw │
 └──────────────┘   └──────────────┘   └────────────────────┘   └────────────┘   └────────────┘
        ▲                                                                                │
        └──────────────────────────  cron every ~hour  ◀────────────────────────────────┘
```

The key architectural choice — and the one that makes this pleasant to build — is **render HTML to an image with a headless browser** rather than drawing pixels by hand. You get the full power of HTML/CSS/SVG for layout, then screenshot it at the panel's exact resolution. The browser is a means to an end; the frame only ever sees a flat image.

Why this beats the alternatives: hand-drawing with an imaging library is tedious and ugly; native e-ink SDKs are low-level and panel-specific. HTML gives you real typography, flexbox/grid layout, and crisp vector icons, and it's easy to preview in any browser while you design.

---

## Build Process

Build in this order. Each step produces something you can see, so problems surface early.

### Step 1 — Learn your panel's constraints first

Before any code, pin down: exact pixel dimensions, orientation, color capability, and how images get onto it. Design decisions cascade from these. For a color panel, read *Designing for E-Ink* below and internalize the palette **before** you style anything — retrofitting colors later is painful.

### Step 2 — Stand up the data hub

Get Home Assistant running on the always-on computer (or a separate box) and connect the first data source — usually weather and a calendar, because they're instantly useful and need no hardware. Confirm you can read a value from its API. In Home Assistant, every piece of data is an "entity" (`sensor.outdoor_temperature`, `weather.home`, `calendar.family`) with a `state` and `attributes`. That uniform shape is the whole point of using a hub.

### Step 3 — Build the render pipeline

Three pieces, kept separate:

- **An HTML template** with placeholder tokens (`{{WEATHER}}`, `{{CALENDAR}}`, `{{DATE}}`) and all the CSS. Design it at the panel's exact dimensions.
- **Fetcher functions** — one per data source, each returning clean data or `None` on failure (see *Integration Patterns*).
- **A generator** that fills the template's placeholders with rendered HTML, then screenshots it.

Use a headless browser via Playwright (Python). The one non-obvious trick that avoids a whole class of bugs:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page(viewport={"width": 1200, "height": 1600})  # your panel size
    page.set_content(page_html, wait_until="networkidle")   # NOT goto(file://...)
    page.wait_for_timeout(1500)                             # let fonts/render settle
    page.screenshot(path="dashboard.png")
    browser.close()
```

Use `set_content()` (feed the HTML directly), **not** `goto("file://…")`. When you point a browser at a file, it needs filesystem permission for every asset — fonts, images — and that permission often isn't there when the script later runs unattended from cron. Feeding the HTML in-memory sidesteps it. For the same reason, **inline your assets**: convert fonts and images to base64 data URIs embedded in the HTML so the browser never touches disk. (Fonts: read the file, base64-encode it, and replace `url('/path/font.ttf')` with `url('data:font/ttf;base64,…')` before rendering.)

### Step 4 — Get one image onto the frame

Push the rendered image to the panel. For network frames this is typically a single HTTP upload:

```python
import requests
with open("dashboard.png", "rb") as f:
    requests.post(f"http://{FRAME_IP}/upload?filename=dashboard.png&show_now=1",
                  files={"file": f}, timeout=15)
```

(The exact URL/params depend on the frame; `show_now=1` is a common "display immediately" flag.) For a directly-wired panel, this step is a call into the panel's Python library instead. **Stop here and celebrate the first real image on the glass** — everything after is refinement.

### Step 5 — Put it on a schedule

Run the script every hour (or whatever cadence suits — e-ink refreshes are visible, so hourly is calmer than every minute). On Linux/macOS, cron is the simplest reliable choice:

```
0 6-23 * * *  /path/to/venv/bin/python3 /path/to/dashboard.py >> /tmp/dashboard.log 2>&1
```

Consider skipping overnight hours when nobody's looking (saves frame wear and lets it sleep). Log to a file so you can diagnose failures you didn't witness.

### Step 6 — Make it survive being ignored

This is what separates a toy from something that runs for a year:
- **Every fetcher fails soft** — one dead data source must never blank the whole dashboard (see below).
- **A watchdog** — have the script write a "last success" timestamp somewhere the hub can see, and alert you (a phone push via Home Assistant) if it goes stale for a few hours. A dark dashboard should page you, not wait to be noticed.
- **Freshness on the display itself** — render a small "updated 3:00pm" stamp so a frozen image is obvious at a glance.

---

## Designing for E-Ink

E-ink is not a screen; it's a digital print. Design like a letterpress poster: bold, high-contrast, graphic. This section assumes a 6-color "Spectra" panel; for black-and-white panels the same philosophy applies with fewer colors.

### Work from a fixed palette

A Spectra-6 panel has exactly six physical pigments. Pick your hex values to match them and use **only** those:

| Role | Example hex | Use for |
|------|-------------|---------|
| Black | `#000000` | Text, borders, primary content |
| White | `#FFFFFF` | Backgrounds |
| Red | `#A02020` | Emphasis, warnings, overdue states |
| Yellow | `#F0E050` | Alerts, warmth, accents |
| Green | `#608050` | "OK"/on-track status, nature |
| Blue | `#5080B8` | Secondary data, times, labels, links |

(Exact values vary by panel; get your manufacturer's target colors and tune to them.)

### What NOT to do

- **No grays.** `#808080`, `#CCCCCC` and friends dither into grainy black/white speckle. If you want "lighter," use one of your six colors, not a tint.
- **No cream/beige backgrounds** — they dither into yellow/white noise.
- **No gradients, no `box-shadow`/`text-shadow`, no `rgba()` alpha on colored elements, no opacity.** Transparency composites into colors the panel can't show, so it dithers unpredictably.
- **No anti-aliased curves from CSS.** For any icon or illustration, use **SVG** with solid fills from the palette — it stays crisp at the panel's ~110–150 DPI where fine CSS detail turns to mud.

### When dithering IS the right call

Two cases where you *want* Floyd-Steinberg dithering:
- **Photographs** (a family photo strip, a weather radar image). A continuous-tone photo can't be posterized to six flat colors; dithering is how it's meant to render.
- **Illustrations deliberately drawn for the dithered look.**

Important nuance: **let the frame do the dithering if it can.** Many color panels dither on-device. If so, push a *clean, full-color* image and let the hardware convert it — pre-dithering on your computer looks rough on an RGB monitor and can double-dither on the panel into mud. Only dither yourself if the frame expects an already-reduced image.

### Respect content caps

The single most common visual bug: content overflowing a card and getting silently clipped (with `overflow: hidden` there's no scrollbar to warn you). Give every region a **line budget** and enforce it in code — e.g. calendar = 10 lines max, weather = 3 forecast days, a list = 5 items with a "+N more" indicator. The only authoritative test is to render the actual image and look at it; there is no reflow safety net on a fixed-size canvas.

---

## Integration Patterns

### The fail-soft fetcher (use this shape for every source)

```python
def fetch_weather():
    """Return a dict, or None on any failure. Never raise."""
    try:
        # ... call the hub / API ...
        return data
    except Exception as e:
        print(f"  warning: weather unavailable: {e}")
        return None
```

Then every HTML builder handles `None` by rendering a small "unavailable" note, not by crashing. **One flaky API must never take down the whole board.** This is the rule that makes the dashboard trustworthy.

### Reading data from Home Assistant

HA exposes a REST API and a WebSocket API. From a render script, the REST API with a long-lived token is simplest: `GET /api/states/sensor.outdoor_temperature` returns the state and attributes as JSON. Store the token outside your code (an env var or a sibling file), never hard-code it. Refresh short-lived tokens before a batch of calls; a `401` comes back as the literal text `401: Unauthorized`, so read the response as text before assuming JSON.

### Weather, robustly

Weather integrations disagree on field names for the same thing (`precipitation_probability` vs `precip_probability` vs `pop`). Check several names, and use explicit `is not None` tests — `0` is a valid probability but falsy, so `value or default` silently drops real zeros. Consider a two-source merge (one provider for conditions/forecast, another for precipitation amounts) with a clean fallback chain if the primary returns empty.

### Images (radar, photos)

Fetch the raw image, crop/scale it to your card's aspect ratio with an imaging library (Pillow), then embed it as a base64 data URI so the render step needs no disk access. Decide per *Designing for E-Ink* whether you or the frame handles the dithering.

### Local, personal data

Some of the best dashboard content isn't in any API — a shared shopping list, a chore rota, little notes. On a Mac you can pull these from Apple Notes via AppleScript; elsewhere a plain text file in a synced folder, or a simple hub helper, works. Parse defensively: people type dates eight different ways, so try multiple formats and degrade gracefully.

### Smart-home automations feeding the board

If you trigger automations off dashboard devices (a button that forces a refresh, a sensor that changes a card), prefer **entity-based triggers** over device-ID triggers. Device IDs get reassigned when a hub re-discovers hardware, silently breaking the automation; entity IDs are stable.

---

## Gotchas & Failure Modes

Hard-won, all generalizable:

- **Keep the Python environment standalone and never delete it.** The most catastrophic outage pattern: a virtualenv that the scheduler depends on gets deleted (or is a symlink into a folder that gets cleaned up), and every scheduled run fails instantly and silently — dark for hours before anyone notices. Make the venv a real, independent directory, put it *outside* any folder you might tidy, and before deleting anything that could be a dependency, resolve symlinks (`ls -la`, `readlink`) first.
- **Cron has fewer permissions than your terminal.** A script that renders perfectly when you run it by hand can fail from cron because the scheduled context lacks file-access permissions your login shell has (notably macOS TCC around synced/cloud folders). Inline your assets and use in-memory rendering so the browser needs no file access, and always test the *scheduled* path, not just the interactive one.
- **You can't fully test the render everywhere.** A headless browser needs system libraries that some sandboxes lack. Do real render tests on the actual always-on machine.
- **The frame sleeps.** Network e-ink frames drop into low-power sleep and won't answer HTTP. Treat a connection timeout on push as normal — generate and save the image anyway, and let the next cycle deliver it when the panel wakes.
- **Fixed canvas, no reflow.** Content that fits in a desktop browser preview can still clip on the panel. Budget lines and inspect the real output image.
- **Emoji and exotic glyphs** often don't render on e-ink — strip them from any text you pull from user-authored sources.

---

## Customize It for Your Home

The pattern is deliberately generic so it renders on the first try; it gets good as the person tailors it:

- **Swap in their data.** Their weather station, their calendars, the three sensors they actually care about. Start minimal.
- **Match their panel's palette.** If their frame isn't Spectra-6 (black/white, or black/white/red), redefine the palette table and design within it.
- **Choose their cards.** Weather + calendar is a strong default. From there: home climate, a chore tracker, a photo strip, transit times, a "now playing" strip — whatever earns its space on the glass.
- **Tune the cadence.** Hourly is calm; some people want a morning refresh and little else.

The fastest way to a dashboard someone loves: get the smallest version onto the frame, live with it for a day, and change the one thing that annoyed them. Repeat. A build like this usually takes several small passes to settle — that's the process working, not a sign anything's wrong.

---

## A Reference Build (one configuration that works)

If it helps to point at something concrete, here is one proven setup — treat it as *an* answer, not *the* answer:

- **Frame:** a 13.3" Spectra-6 color e-ink frame that accepts image uploads over local HTTP and does its own on-device 6-color dithering.
- **Always-on computer:** a Mac Mini that runs the render script from cron, hourly during waking hours.
- **Data hub:** Home Assistant, aggregating Environment Canada weather + radar, shared calendars, a personal weather station, Zigbee sensors (temperature, air quality, leak/vibration), and an ecobee thermostat.
- **Pipeline:** Python fills an HTML template, Playwright/Chromium screenshots it at 1200×1600, Pillow crops the radar, everything is base64-inlined, and the script POSTs the clean JPEG to the frame's upload URL. A watchdog writes a "last success" timestamp back to the hub and pushes a phone alert if it goes stale.
- **Content:** weather (3-day + extras), calendar with waste-collection days, indoor climate/air quality, live radar with a wind compass, and a "domesticitude" strip — chores, a shopping list, and little notes to the household.

Every one of those choices is swappable. The bones — data → HTML → headless-browser image → push, on a timer, failing soft — are what carry over to any home.
