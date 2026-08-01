# Perception Theme

A slick, modern **glassmorphic** dashboard for Home Assistant in a deliberately
tiny, near‑black‑and‑white palette: frosted‑glass cards, hairline borders, big
rounded corners, and translucent whites over a dark background.

> **This is for a backlit screen** (wall tablet, monitor, phone) — glassmorphism
> relies on blur and transparency that e‑ink panels can't render.

## What's in this repo

| Path | What it is |
|------|------------|
| `themes/perception.yaml` | The theme. Sets the palette + background and applies the frosted‑glass effect to every card via card‑mod. |
| `dashboards/perception.yaml` | A sample dashboard (clock, weather, lights, climate, media) using core cards. |
| `www/perception/InterVariable.woff2` | The bundled, self‑hosted **Inter** variable font (all weights in one file), served at `/local/`. |
| `hacs.json` | Lets HACS install the **theme** straight from this repo. |

## Requirements

- **Home Assistant** 2024.8 or newer (uses the modern *sections* dashboard view).
- **[card-mod](https://github.com/thomasloven/lovelace-card-mod)** — the frontend
  plugin that injects the blur/transparency CSS. Glassmorphism is **not** possible
  with core cards alone, so this is mandatory. Install it from HACS (default store).
- **Inter font (bundled, self‑hosted).** No internet needed at runtime — you copy
  one `.woff2` into `config/www/perception/` during install (steps below).

---

## Installation

There are three ways to install, from most to least automated. Pick one.

### Method A — HACS (recommended, least copying)

Because you're publishing to a **private** repo, HACS can still install from it as
a *custom repository* (the GitHub account HACS is linked to must have read access
to the repo).

1. **Install card-mod:** HACS → search **card-mod** → **Download** → restart HA if prompted.
2. **Add this repo to HACS:** HACS → ⋮ (top‑right) → **Custom repositories** →
   paste your repo URL → **Category: Theme** → **Add**.
3. Find **Perception Theme** in HACS → **Download**. This copies
   `themes/perception.yaml` into your `config/themes/` folder for you.
4. **Install the font** (HACS copies only the theme file): create
   `config/www/perception/` and put **`InterVariable.woff2`** in it — grab it from
   this repo's `www/perception/` folder. HA serves it at
   `/local/perception/InterVariable.woff2`, which the theme references.
5. Continue to [Enable the theme](#enable-the-theme) and
   [Add the dashboard](#add-the-dashboard).

> HACS installs **themes**, but it cannot install a Lovelace *dashboard config*.
> The dashboard YAML is always added by you (paste or `!include`, below). That's a
> Home Assistant limitation, not a gap in this repo.

### Method B — Manual copy

1. Install **card-mod** from HACS (step 1 above).
2. Copy `themes/perception.yaml` into your Home Assistant `config/themes/` folder.
3. Copy `www/perception/InterVariable.woff2` into `config/www/perception/`
   (create the folder if needed). HA serves it at `/local/perception/…`.
4. Make sure `configuration.yaml` loads themes:
   ```yaml
   frontend:
     themes: !include_dir_merge_named themes
   ```
5. Restart Home Assistant.

### Method C — Git clone into `/config` (best for development)

Clone the repo straight onto the HA box and point HA at the files in place, so
there's **nothing to copy** and updates are a `git pull`. See
[Development workflow](#development-workflow).

---

## Enable the theme

- **Per user:** click your name (bottom‑left) → **Theme → Perception**, or
- **Per dashboard view:** the sample already sets `theme: Perception` on the view.

If the theme doesn't appear, go to **Developer Tools → YAML → Reload Themes**
(no restart needed), then hard‑refresh the browser.

## Add the dashboard

**Quick way (paste):** open any dashboard → ⋮ → **Edit dashboard** → ⋮ →
**Raw configuration editor** → paste the `views:` block from
`dashboards/perception.yaml`.

**Permanent YAML dashboard:** add to `configuration.yaml`:

```yaml
lovelace:
  dashboards:
    perception:
      mode: yaml
      filename: dashboards/perception.yaml
      title: Perception
      icon: mdi:cube-outline
      show_in_sidebar: true
```

## Swap in your entities

The sample uses common placeholder ids (`light.living_room`,
`sensor.living_room_temperature`, `weather.home`, `media_player.living_room`,
`person.me`). Replace them with your real entity ids — most won't exist as‑is.

---

## Development workflow (edit and see changes in real time)

The goal is a tight loop with **no Home Assistant restarts**. Set it up once:

**1. Get an editor on the HA box.** Install one of these add‑ons:
   - **Studio Code Server** — full VS Code in the browser, editing `/config` directly, or
   - **Samba share** / **SSH & Web Terminal** — mount `/config` on your Mac and edit locally.

**2. Put the repo in `/config` and point HA at it (no copying).** From the
   SSH/Terminal add‑on, clone your private repo (use a deploy key or a PAT):
   ```bash
   cd /config
   git clone git@github.com:<you>/<repo>.git glass-dashboard
   ```
   Then wire HA to the cloned files:
   - **Theme** — symlink it into the themes folder so `!include_dir_merge_named themes` picks it up:
     ```bash
     ln -s /config/glass-dashboard/themes/perception.yaml /config/themes/perception.yaml
     ```
   - **Font** — symlink the self‑hosted font into `www/` so `/local/` can serve it:
     ```bash
     mkdir -p /config/www && ln -s /config/glass-dashboard/www/perception /config/www/perception
     ```
   - **Dashboard** — point a YAML‑mode dashboard straight at the repo file:
     ```yaml
     lovelace:
       dashboards:
         perception:
           mode: yaml
           filename: glass-dashboard/dashboards/perception.yaml
           title: Perception
           show_in_sidebar: true
     ```

**3. The edit loop:**
   | You changed... | Apply it without a restart |
   |----------------|----------------------------|
   | The **theme** / card‑mod CSS | **Developer Tools → YAML → Reload Themes**, then hard‑refresh the browser (`Cmd/Ctrl+Shift+R`). |
   | The **dashboard** YAML (YAML mode) | Just hard‑refresh the browser. |
   | The **dashboard** via **Raw config editor** | Save in the editor — it applies instantly. |

**4. Tweak CSS live first (optional but fast).** Open the browser dev tools,
   adjust `backdrop-filter`, `background` alpha, `border-radius` on `ha-card` until
   it looks right, then paste the final values into `themes/perception.yaml`.

**5. Deploy / update:** commit and push from your machine, then on the HA box
   `cd /config/glass-dashboard && git pull` and reload themes. HACS users just hit
   **Update** in HACS when you tag a new version.

> **About "a zip for easier install":** Home Assistant has no native "install a
> zip" flow for themes or dashboards, so a zip wouldn't save you steps. The
> automated paths are **HACS** (custom repository, above) or **git clone**. You can
> still attach a zip to a GitHub Release for archival, but HACS/git are the real
> one‑step options.

---

## Releasing (versioning for HACS)

HACS versions this repo from **Git tags / GitHub Releases**, not from Home
Assistant (HA has no concept of theme versions). Once at least one release
exists, HACS shows the tag in its download dialog and only lights up the
**Update** button when a newer tag exists — much calmer than tracking every
commit on `main`.

Publish a new version like this:

1. Commit and push your changes to `main`:
   ```bash
   git add -A
   git commit -m "Describe the change"
   git push origin main
   ```
2. Tag and create the GitHub Release (the tag *is* the version):
   ```bash
   gh release create v1.1.0 --target main --title "v1.1.0" --notes "What changed"
   ```
3. In HACS, users get an **Update** prompt → redownload. If the theme name or
   file changed, they reselect **Perception** under Profile → Theme, then run
   **Developer Tools → YAML → Reload Themes** and hard‑refresh.

Notes:
- Use consistent semver tags (`v1.2.3`); HACS sorts them to find the latest and
  offers the last 5 releases plus the default branch.
- `hacs.json`'s `homeassistant` key is a *minimum HA version* gate, not the
  theme's own version.
- A single‑file theme needs no `zip_release`/`filename` in `hacs.json` — HACS
  grabs the one YAML in `themes/`.

---

## Customization

All styling lives in `themes/perception.yaml`:

- **Background image instead of the gradient** — drop an image in `config/www/` and set:
  ```yaml
  lovelace-background: "url('/local/wallpaper.jpg') center/cover fixed"
  ```
- **More / less frost** — change `blur(22px)` and the `rgba(255,255,255,0.06)` alpha
  in the `card-mod-card` block.
- **Rounding** — `ha-card-border-radius` (and the `border-radius` in `card-mod-card`).
- **Palette** — it's intentionally minimal (black, white, translucent whites). Keep
  accents near‑white to preserve the look.

## Troubleshooting

- **No blur / cards look flat** — card‑mod isn't installed or loaded. Confirm it's in
  HACS and hard‑refresh. On very old browsers `backdrop-filter` is unsupported.
- **Theme not in the list** — Developer Tools → YAML → **Reload Themes**; check the
  `frontend: themes:` include and that the file is in `config/themes/`.
- **Glass looks black** — the blur has nothing bright behind it. That's expected with
  the dark gradient; use a brighter/wallpaper background if you want more glassiness.
- **Cards clip content** — a fixed layout has no reflow; give lists a sensible item
  cap and check on the actual tablet resolution.
- **Private repo won't add in HACS** — the GitHub account HACS is authenticated with
  must have read access to the repo.
