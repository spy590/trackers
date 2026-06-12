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

## Architecture

Each tracker is a **single `.html` file** with all CSS, JS, and data inlined — no external files, no imports, no framework. Dependencies (e.g. Chart.js) are loaded from CDN with SRI integrity hashes.

### World Cup tracker (`world-cup-tracker-2026.html`)

Structure of the script section:

- **DATA block** — plain JS constants: `FLAG` (emoji map), `GROUPS` (group→teams), `MATCHES` (array of match objects via `M()` factory), `VENUES`, `KO_DATES`, `COORDS`/`VENUE_GEO` (lat/lon for maps), `SQUADS` (12 detailed team rosters via `p()` factory).
- **Tab system** — five panels (`schedule`, `groups`, `teams`, `bracket`, `stats`) switched by `.tab`/`.panel` classes; rendered by dedicated `render*()` functions called on tab click.
- **Overlay system** — a single modal (`#ovBack`) reused for team profiles, player profiles, and stadium details; `openTeam()` / `openPlayer()` push/pop state within the overlay using a back-button pattern.
- **Live data** — fetches from an external scores API on a timer; updates `hs`/`as` score fields on match objects and re-renders the live card in the schedule panel.
- **Charts** — Chart.js bar/doughnut charts in the Stats panel, initialised once and stored in variables to allow destroy-and-recreate on re-render.
- **Maps** — OpenStreetMap iframes embedded in venue overlays via `mapFrame()`.

CSS uses a single `:root` token set (`--bg`, `--accent`, `--live`, etc.) — all colours flow from there.
