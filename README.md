# Rota

Rota turns place recommendations found in travel articles and pasted post captions into a map list that the traveller reviews, and into an editable plan for 1 to 7 days with estimated times. English and Turkish interface.

> **Status: in development.** It is a working product that has not been publicly launched. The source code is private. The problem is a hypothesis and has not been validated with users, and no users, reviews or adoption are claimed. This repository documents the product thinking.
>
> Part of [Ahsen Varlı İçgüdel's portfolio](https://ahsenvarlicgudel.vercel.app) · [LinkedIn](https://www.linkedin.com/in/ahsen-varl%C4%B1-i%C3%A7g%C3%BCdel-2b785018)

![Check the locations](images/confirm.webp)

## Key product decisions

1. **Rules-based extraction before any language model.** Rota uses no AI model today. Extraction is rules-based, so there is no API key, no paid model and no account. A language model is considered only if measurement shows a gap it would close.
2. **No invented locations, and the user decides when it is ambiguous.** A single reliable match is placed on the map automatically. When several branches come up, the traveller chooses and the place is not added until they do; a place with no match is listed as not found. The product never invents an address or coordinates. (Until 25 November 2026 every place needed a manual confirmation; that step was removed with my approval.)
3. **Defer accounts and sync.** One active trip lives in the browser with a JSON backup. Accounts and cross-device sync wait for a later phase that needs a server, a privacy notice and legal review.

Details and limits are below.

## Problem and target user

Recommendations pile up in blog posts, saved posts and notes. Before a trip the traveller has to pull place names out of long text, find the right branch on a map, remember why a place was saved and split the places into days.

**Hypotheses (all untested):** people who save travel posts often can't find them again; day plans are built by hand from scattered sources; poor internet access abroad makes a plan harder to use during a trip.

**Assumed user:** a traveller who collects recommendations weeks before a short city trip of 1 to 7 days, wants to edit their own selection instead of receiving a generated route, and would rather not create an account.

**Discovery:** short past-behaviour interviews are planned (asking how people saved and used recommendations, not showing the product). None have been recorded yet.

## What works today

- **Import:** from an open article URL, or from pasted text or a social-post caption. Video and Reels links are not read; a field next to the link takes the pasted caption instead.
- **Location step:** single reliable matches are placed automatically, ambiguous branches show options, and unmatched places are shown as not found, not hidden.
- **Discover mode** for travellers with no plan yet: place suggestions from Wikivoyage guides with attribution (CC BY-SA 4.0).
- **Editable day plan** for 1 to 7 days with estimated arrival and leave times from a departure time. Remove a stop (with undo), delete a place, add one to a day, or drag stops between days; keyboard buttons do the same.
- **Today view**, a map, Google Maps day links, KML for Google My Maps and a JSON backup.
- **Installable and offline:** add it to the home screen; after one online visit the last trip and plan open without a connection (map tiles and import still need a network). On Android the installed app takes a shared link or text from the share menu.
- **First run:** a skippable three-step introduction and a backup reminder, because the trip lives only in the browser.

| | |
|---|---|
| ![Import](images/import.webp) | ![Location step](images/confirm.webp) |
| Paste text or a caption. | Automatic matches, and a branch choice where several came up. |
| ![Places and map](images/places-map.webp) | ![Day plan](images/day-plan.webp) |
| Places with their source sentences, on a map. | Editable day plan with times, drag-and-drop and remove or delete. |
| ![Today](images/today.webp) | ![Day plan on a phone](images/mobile-day-plan.webp) |
| The "Today" view. | The day plan at phone width. |

All screenshots use an invented sample text about Copenhagen and were captured on 25 November 2026. The map results come from the live public geocoder and differ between runs.

## Selected product decisions and trade-offs

- **Reading, extracting and matching are separate stages**, and success and gaps are reported for each. A single "places found" number hides where a pipeline fails.
- **A location is verified only after the user confirms it.** The product never invents an address or coordinates.
- **Rules-based extraction first**, with no API key, paid model or account. A language model or a new map provider is considered only if measurement shows a gap it would close.
- **No scraping of protected pages** and no video understanding. The user pastes the caption.
- **One active trip in the browser**, with a JSON backup to move it. Accounts and sync are deferred until there is a server, a privacy notice and legal review.
- **Honest planning:** estimated times use a straight-line walking estimate and are editable guesses. Stops are grouped by straight-line proximity and the map says so. There are no road-routing, opening-hours or live-transit claims.
- **Restrained use of the public geocoder:** one request per second, a 24-hour cache, back-off after refusals and a 30-second search budget.
- **Measure before building:** the first sprint measures the existing pipeline on real articles, and stronger extraction is considered only against the gaps found.
- **A written list of claims the product will not make:** no video understanding, no automatic Instagram or Google Maps list import, no "AI" or "smart assistant" wording, no "best route", no accuracy percentage, no "works in every city", no sync, no invented users or reviews.

## Architecture overview

1. **Import:** a server route reads an open article page after URL safety checks, or accepts pasted text. A rules-based extractor finds candidate places (proper names, travel cues and a small example gazetteer for two cities), capped per import. Every place keeps its source sentence.
2. **Location matching:** a public OpenStreetMap-based geocoder restricted to the destination city, then a check that the type of the found place fits what the name implies. The user confirms every place.
3. **Discovery:** Wikivoyage guides through its public API, with a stored copy for two example cities.
4. **State and output:** the trip lives in the browser. The user edits the plan and exports it.

**Data flow, stated plainly:** pasted text and links go to the app's server for import, place names go to the public geocoder, and map tiles come from OpenStreetMap. Only the saved trip stays in the browser, so "nothing leaves your device" would be wrong.

## Technology (verified from source)

TypeScript, Next.js (vinext), React 19, Leaflet, Mozilla Readability, Zod and drizzle-orm. Tests use `node:test` with fixtures, and CI runs the tests, the type check and the build.

## How it is built

The owner decides scope, cost and release. Claude Code agents do the planning, implementation, code review and a browser walkthrough of the running app through GitHub issues, task branches and draft pull requests. They are started by GitHub events and a daily schedule, not by a continuous loop. The walkthrough and product-manager roles were specified on 25 November 2026 and have not run yet. Details and the flow diagram: [agent-based development](https://github.com/ahsenvi/product-decision-assistant-showcase/blob/main/docs/agent-based-development.md).

## Limits and next steps

- Extraction can miss names or suggest non-places, so review is required.
- A slow map service can leave an import partial.
- Installation, offline use and the Android share menu have been tested in a headless browser, not on a real phone. iOS has no share-menu import.
- One trip per browser, with no sync.
- **Not built:** photo upload, memory journal, account connection, publishing to social media, multiple trips. They are on the roadmap and not shown as working.

The next phase is an open owner decision.
