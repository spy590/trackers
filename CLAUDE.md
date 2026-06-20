# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A collection of self-contained HTML trackers deployed via GitHub Pages at `https://spy590.github.io/trackers/`. No build system, no npm, no compilation step.

## Development

Open any `.html` file directly in a browser — there is no server required. To preview as GitHub Pages would serve it, any static file server works (e.g. `python3 -m http.server`).

Deploying is just `git push` — GitHub Pages redeploys automatically.

## Adding a tracker

1. Drop a self-contained `.html` file in the repo root.
2. Push. Each tracker is shared via its own direct URL (there is no landing page / `index.html`).

## Mirrored trackers

Some trackers are copies of files authored in another project — to publish updates, re-copy from source and push:

- `nam-training-101.html` ← `/Users/russellthatcher/ProjectsLocal/NamStudio/lessons/nam-training-101/index.html`

## Architecture

Each tracker is a **single `.html` file** with all CSS, JS, and data inlined — no external files, no imports, no framework. Dependencies (e.g. Chart.js) are loaded from CDN with SRI integrity hashes.

### World Cup tracker (`world-cup-tracker-2026.html`)

> **Deploy stamp:** the header card shows a `build YYYY.MM.DD·HHMMZ` label (the `.build` div under `.sub`). There is no build step, so **bump this string by hand (current UTC) on every deploy** — it's how a finished deploy is spotted on the live page.
>
> **Lifespan:** this tracker is only needed through the 2026 World Cup (final July 19, 2026); it can be retired afterward.
>
> **Install (PWA):** the header has an "Add to Home Screen" button. It depends on two sibling files at the repo root — `manifest.webmanifest` and `apple-touch-icon.png` — so this tracker is *not* fully self-contained. Android/Chromium fires the native install prompt; iOS Safari has no install API, so the button shows manual Share→"Add to Home Screen" steps.

Structure of the script section:

- **DATA block** — plain JS constants: `FLAG` (emoji map), `GROUPS` (group→teams), `MATCHES` (array of match objects via `M()` factory), `VENUES`, `KO_DATES`, `COORDS`/`VENUE_GEO` (lat/lon for maps), `SQUADS` (12 detailed team rosters via `p()` factory).
- **Tab system** — five panels (`schedule`, `groups`, `teams`, `bracket`, `stats`) switched by `.tab`/`.panel` classes; rendered by dedicated `render*()` functions called on tab click.
- **Overlay system** — a single modal (`#ovBack`) reused for team profiles, player profiles, and stadium details; `openTeam()` / `openPlayer()` push/pop state within the overlay using a back-button pattern.
- **Live data** — fetches from an external scores API on a timer; updates `hs`/`as` score fields on match objects and re-renders the live card in the schedule panel.
- **Charts** — Chart.js bar/doughnut charts in the Stats panel, initialised once and stored in variables to allow destroy-and-recreate on re-render.
- **Maps** — OpenStreetMap iframes embedded in venue overlays via `mapFrame()`.

CSS uses a single `:root` token set (`--bg`, `--accent`, `--live`, etc.) — all colours flow from there.
