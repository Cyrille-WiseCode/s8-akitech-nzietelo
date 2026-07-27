# CLAUDE.md — Notes pour la coordination pédagogique (Akieni Academy)

Ce fichier est destiné à la coordination et aux instances de Claude/Cursor
qui aideraient à faire évoluer le scaffold. Il ne doit pas être distribué
aux apprenants tel quel (il révèle les correspondances avec les corrigés).

## What this repo is

A teaching scaffold for a student project ("Akieni Academy Cohorte 2"): a
7-page carpooling ("covoiturage") mini-site for Brazzaville, split into a
Python/Flask backend and a vanilla HTML/CSS/JS frontend. This repo is
distributed **only to the Data Science and Full Stack tracks** — it is a
pure coding scaffold, not a process document. Product Manager and Business
Analyst work (product framing, specs) happens upstream, outside this repo,
via separate deliverables; none of that process is referenced here.

This is the second scaffold in Cohorte 2, after the COMAKI cooperative
project. Functional scope is comparable in depth (7 pages vs COMAKI's 6,
15 backend functions vs COMAKI's 16, 6 API routes).

## Commands

Backend (from `backend/`):

```bash
pip install -r requirements.txt
python -m pytest -v          # 28 tests
python app.py                # API on http://localhost:5000
```

Frontend: no build step. Open `frontend/functions.test.html` in a browser to
run the 23 JS unit tests. Open any page in `frontend/*/` in a browser to use
the site (requires the Flask backend running).

## Architecture

**Backend** (`backend/`):
- `app.py` — Flask entrypoint, 6 API routes (`/api/trajets`, `/api/trajets/<id>`,
  `/api/reservations`, `/api/reservations/<tel>`, `/api/dashboard`,
  `/api/quartiers`), plus a `/` route returning a helpful JSON message
  (mirrors COMAKI's pattern — prevents a raw 404 when someone opens the
  API root directly). NOT edited by students.
- `controllers.py` — orchestrates `logic.py` functions and loads
  `data/trajets.json`. NOT edited by students.
- `logic.py` — the only backend file DS students edit. 15 functions split
  into three zones (A: search & availability, B: reservations & tracking,
  C: statistics & dashboard). The first function
  (`filtrer_trajets_disponibles`) ships fully implemented with inline
  comments, as a worked example of expected code style — mirrors the
  pattern used in the COMAKI scaffold's `logic.py`. All other functions
  have a docstring (Paramètre/Retourne/Exemple, with a concrete worked
  example) followed by `# TODO : à compléter` + `pass`.
- `tests/test_logic.py` — 28 pytest tests that are the actual spec for
  `logic.py`. With only the worked example filled in, 6 tests pass
  out of the box (the 3 targeting that function, plus a few edge cases
  that happen to pass against `pass`-stub functions returning `None`
  compared to falsy expectations — this is expected and not a bug).
- `data/trajets.json` — static fixtures (8 quartiers, 8 conducteurs,
  15 trajets, 20 reservations spanning April to July 2026).

**Frontend** (`frontend/`): 7 static HTML pages, each in its own subdirectory
with a dedicated CSS file, sharing `main.js` and `functions.js` at the
`frontend/` root.
- `main.js` — pre-wired: detects which page is loaded via a unique
  `#page-*` body id, calls the Flask API, reads the URL, writes results
  into the DOM. Calls `functions.js` for all data
  transformation/formatting/validation. NOT edited by students.
- `functions.js` — the only JS file students edit. 12 pure functions, two
  per Full Stack person.
- `functions.test.html` — browser test runner (23 tests).
- Subdirectories: `accueil/`, `recherche/`, `trajet/`, `proposer/`,
  `mes-trajets/`, `dashboard/`, `confirmation/` — each contains one HTML
  skeleton with detailed `<!-- TODO -->` comments (expected layout
  technique, sub-elements, CSS class names already used by `main.js` for
  dynamically injected content) and one empty CSS file. HTML files contain
  only the DOM elements that `main.js` expects (IDs, form elements, script
  tags). Elements marked "NE PAS MODIFIER" are the backend connection
  wiring and must not be renamed or removed.

## Which files are editable

- `backend/logic.py` — Data Science track only (3 zones, 15 functions,
  minus the 1 worked example already provided).
- `frontend/functions.js` — Full Stack track, 2 functions per dev.
- `frontend/*/` — each subdirectory's HTML and CSS files are the FS dev's
  canvas. Elements with IDs referenced by `main.js` must not be removed
  or renamed.

Explicitly marked "NE PAS MODIFIER": `backend/app.py`, `backend/controllers.py`,
`frontend/main.js`, `frontend/functions.test.html`, the `module.exports`
block at the bottom of `functions.js`, and all ID-bearing elements in HTML
skeletons.

## Known issue fixed

Earlier versions of this scaffold had no `/` route on the Flask API,
causing a raw Flask 404 page when a student opened `http://localhost:5000/`
directly (a natural first thing to try). Fixed by adding the `/` route
above. Also, the README previously didn't specify HOW to actually view
the frontend pages (opening an HTML file directly via `file://` breaks
the relative script paths and can behave inconsistently with `fetch`
across browsers) — the README now explicitly instructs serving the
frontend via `python -m http.server 5500` from `frontend/`, confirmed
working end-to-end (CORS headers verified present on cross-origin
requests from `localhost:5500` to `localhost:5000`).

## `_solution_CP/`

Instructor-only answer key (`logic_solution.py`, `functions_solution.js`)
used for auto-grading. Delete before distributing the repo to students.
Don't copy from it into student-facing files. Keep in sync: any function
added to student-facing files needs a matching solution here.

Verification: with `_solution_CP/logic_solution.py` copied over
`backend/logic.py`, all 28 pytest tests pass. With `functions_solution.js`
loaded, all 23 JS tests pass. The Flask API has been end-to-end tested
with the corrigé — the 6 routes respond correctly and the business rule
"a reservation on a full trip is rejected" is verified live (curl POST
returns 400 with "Trajet complet").

## What is intentionally NOT in this repo

No cahier des charges, no BA backlog/FRD/BPMN, no PM brief. Those are
separate deliverables produced upstream by other tracks and never
distributed as part of this coding scaffold — students on DS and FS work
directly from the code contract (docstrings, JSON structure, HTML IDs)
and the business context paragraph in README.md, not from a requirements
document.
