# CLAUDE.md — Notes pour la coordination pédagogique (Akieni Academy)

Ce fichier est destiné à la coordination et aux instances de Claude/Cursor
qui aideraient à faire évoluer le scaffold. Il ne doit pas être distribué
aux apprenants tel quel (il révèle les correspondances avec les corrigés).

## What this repo is

A teaching scaffold for a student project ("Akieni Academy Cohorte 2"): a
9-page carpooling ("covoiturage") mini-site for Brazzaville, split into a
Python/Flask backend and a vanilla HTML/CSS/JS frontend. This repo is
distributed **only to the Data Science and Full Stack tracks** — it is a
pure coding scaffold, not a process document. Product Manager and Business
Analyst work (product framing, specs) happens upstream, outside this repo,
via separate deliverables; none of that process is referenced here.

This is the second scaffold in Cohorte 2, after the COMAKI cooperative
project. Functional scope is comparable in depth (9 pages vs COMAKI's 6,
17 backend functions vs COMAKI's 16, 8 API routes).

Signup/login was added after the initial scaffold as a 4th backend zone
(Zone D — Comptes & authentification) and a 7th Full Stack pair (FS7),
following the exact same "infra already wired, students write pure
functions only" contract as the original 6 features. Passwords are
hashed with `werkzeug.security` (no new dependency — already pulled in
transitively by Flask); there is no session/token, login just returns
success or failure.

## Commands

Backend (from `backend/`):

```bash
pip install -r requirements.txt
python -m pytest -v          # 33 tests
python app.py                # API on http://localhost:5000
```

Frontend: no build step. Open `frontend/functions.test.html` in a browser to
run the 27 JS unit tests. Open any page in `frontend/*/` in a browser to use
the site (requires the Flask backend running).

## Architecture

**Backend** (`backend/`):
- `app.py` — Flask entrypoint, 8 API routes (`/api/trajets`, `/api/trajets/<id>`,
  `/api/reservations`, `/api/reservations/<tel>`, `/api/dashboard`,
  `/api/quartiers`, `/api/inscription`, `/api/login`), plus a `/` route
  returning a helpful JSON message (mirrors COMAKI's pattern — prevents a
  raw 404 when someone opens the API root directly). NOT edited by
  students.
- `controllers.py` — orchestrates `logic.py` functions and loads
  `data/trajets.json` and `data/comptes.json`. Password hashing
  (`werkzeug.security.generate_password_hash`/`check_password_hash`)
  lives here, not in `logic.py` — kept out of the pure-function file
  deliberately. NOT edited by students.
- `logic.py` — the only backend file DS students edit. 17 functions split
  into four zones (A: search & availability, B: reservations & tracking,
  C: statistics & dashboard, D: accounts & authentication). The first
  function (`filtrer_trajets_disponibles`) ships fully implemented with
  inline comments, as a worked example of expected code style — mirrors
  the pattern used in the COMAKI scaffold's `logic.py`. All other
  functions have a docstring (Paramètre/Retourne/Exemple, with a
  concrete worked example) followed by `# TODO : à compléter` + `pass`.
- `tests/test_logic.py` — 33 pytest tests that are the actual spec for
  `logic.py`. With only the worked example filled in, 6 tests pass
  out of the box (the 3 targeting that function, plus a few edge cases
  that happen to pass against `pass`-stub functions returning `None`
  compared to falsy expectations — this is expected and not a bug).
- `data/trajets.json` — static fixtures (8 quartiers, 8 conducteurs,
  15 trajets, 20 reservations spanning April to July 2026).
- `data/comptes.json` — accounts store, starts as `{"comptes": []}`.
  Each account: `{id, nom, telephone, mot_de_passe_hash}`. `telephone`
  is the unique login identifier, same role as the existing
  `/api/reservations/<tel>` lookup key.

**Frontend** (`frontend/`): 9 static HTML pages, each in its own subdirectory
with a dedicated CSS file, sharing `main.js` and `functions.js` at the
`frontend/` root.
- `main.js` — pre-wired: detects which page is loaded via a unique
  `#page-*` body id, calls the Flask API, reads the URL, writes results
  into the DOM. Calls `functions.js` for all data
  transformation/formatting/validation. NOT edited by students.
- `functions.js` — the only JS file students edit. 14 pure functions, two
  per Full Stack person.
- `functions.test.html` — browser test runner (27 tests).
- Subdirectories: `accueil/`, `recherche/`, `trajet/`, `proposer/`,
  `mes-trajets/`, `dashboard/`, `confirmation/`, `inscription/`, `login/`
  — each contains one HTML skeleton with detailed `<!-- TODO -->`
  comments (expected layout technique, sub-elements, CSS class names
  already used by `main.js` for dynamically injected content) and one
  empty CSS file. HTML files contain only the DOM elements that
  `main.js` expects (IDs, form elements, script tags). Elements marked
  "NE PAS MODIFIER" are the backend connection wiring and must not be
  renamed or removed.

## Which files are editable

- `backend/logic.py` — Data Science track only (4 zones, 17 functions,
  minus the 1 worked example already provided).
- `frontend/functions.js` — Full Stack track, 2 functions per dev.
- `frontend/*/` — each subdirectory's HTML and CSS files are the FS dev's
  canvas. Elements with IDs referenced by `main.js` must not be removed
  or renamed.

Explicitly marked "NE PAS MODIFIER": `backend/app.py`, `backend/controllers.py`,
`frontend/main.js`, `frontend/functions.test.html`, the `module.exports`
block at the bottom of `functions.js`, and all ID-bearing elements in HTML
skeletons.

## Session client (localStorage)

Login is stateless server-side — `POST /api/login` just returns
`{ok, compte}` or `{ok: False, erreur}`, no server session/token. The
frontend persists the returned `{id, nom, telephone}` in
`localStorage["bracovoit_session"]` (`frontend/main.js`: `getSession`/
`setSession`/`clearSession`). `initNavSession()` renders a login-state
indicator into a `#nav-session` element present on every page (infra,
NE PAS MODIFIER), with a déconnexion button that just clears the key and
reloads. This session drives two conveniences, both infra-only (no new
pure functions, no backend change): the trajet page's reservation form
pre-fills nom/téléphone when logged in, and "Mes trajets" auto-loads for
the logged-in user's phone number. Don't build a server-side session/JWT
for this scaffold — it's deliberately out of scope for the pedagogical
depth intended here.

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
`backend/logic.py`, all 33 pytest tests pass. With `functions_solution.js`
loaded, all 27 JS tests pass. The Flask API has been end-to-end tested
with the corrigé — the 8 routes respond correctly and the business rules
"a reservation on a full trip is rejected" (curl POST returns 400 with
"Trajet complet") and "signup with a duplicate phone number is rejected"
(curl POST `/api/inscription` returns 400) are verified live.

## What is intentionally NOT in this repo

No cahier des charges, no BA backlog/FRD/BPMN, no PM brief. Those are
separate deliverables produced upstream by other tracks and never
distributed as part of this coding scaffold — students on DS and FS work
directly from the code contract (docstrings, JSON structure, HTML IDs)
and the business context paragraph in README.md, not from a requirements
document.
