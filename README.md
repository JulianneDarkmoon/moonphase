# Moon Phase Dashboard

A GitHub Action that renders a **1200 × 1600** moon-phase dashboard for a
**6-colour E Ink Spectra 6** panel and publishes it with GitHub Pages, where a
display (e.g. Seeed E1004 + SenseCraft HMI) can fetch it by URL.

The image shows the current moon (about two thirds of the screen), the phase
name, illumination %, moon age, distance, the next new and full moon (with the
traditional full-moon name, e.g. *Hunter's Moon*, and a *Supermoon* flag), a
7-day strip of upcoming phases, and optionally moonrise/moonset.

Everything is computed locally (PyEphem) and the moon surface is generated
procedurally, so there are no API keys and no downloads.

## How the image is made for the e-paper panel

* Only the six panel colours are used: black, white, blue, green, red, yellow.
* The moon is shaded in grey and **Floyd–Steinberg dithered with black and
  white only** (mixing in colours makes coloured speckle). The faint blue dots
  on the dark side are earthshine.
* Text, rings and stars are flat colours, so they stay crisp.
* The PNG is written with pure primaries (`#000`, `#FFF`, `#00F`, `#0F0`,
  `#F00`, `#FF0`), which is what most e-paper converters expect. Set
  `OUTPUT_PALETTE=device` to write the panel's measured colours instead.

## Setup

1. **Create a public GitHub repo** and push this project to it.
2. **Turn on Pages**: Settings → Pages → *Build and deployment* → Source:
   **GitHub Actions**.
3. **(Optional) add repo variables** under Settings → Secrets and variables →
   Actions → *Variables*:

   | Variable     | Example               | Meaning                                              |
   |--------------|-----------------------|------------------------------------------------------|
   | `TZ_NAME`    | `America/Los_Angeles` | Time zone for dates/times (default shown)            |
   | `LAT`, `LON` | `34.05`, `-118.24`    | Your location; adds moonrise/moonset to the display  |
   | `HEMISPHERE` | `south`               | Flip the moon as seen from the southern hemisphere   |
   | `ROTATE`     | `90`                  | Rotate the output 0/90/180/270° clockwise            |

4. Run **Actions → Update Moon Dashboard → Run workflow** once. After that it
   runs automatically every day at 08:15 UTC (just after midnight Pacific).
   Change the `cron` line in the workflow if you're in another time zone.

No secrets or tokens are needed. Your image is then at:

```
https://<your-username>.github.io/<repo-name>/moon_dashboard.png
```

Point your display at that URL. `moon_data.json` is published next to it with
the numbers behind the image, and the site root shows the image in a browser.
Pages may serve a cached copy for a few minutes after a deploy.

**Heads up:** GitHub disables scheduled workflows in public repos after 60 days
without repository activity. If the image stops updating, re-enable the
workflow under the Actions tab (or push any commit).

## Run locally

```bash
pip install -r requirements.txt
python scripts/render_moon.py                       # -> output/moon_dashboard.png
MOON_NOW=2026-10-26T12:00:00+00:00 LAT=34.05 LON=-118.24 python scripts/render_moon.py
```

Other switches: `DITHER=0` (no dithering), `OUTPUT_DIR=...`. See the docstring
at the top of `scripts/render_moon.py`.

## Notes

* Phase names use ±~18 h windows around new, quarter and full moon.
* Fonts: [Inter](https://rsms.me/inter/) (SIL OFL), bundled in `assets/fonts`
  so the runner doesn't depend on system fonts.
* To tune colours for your panel, edit the `DEVICE` list in
  `scripts/render_moon.py`.
