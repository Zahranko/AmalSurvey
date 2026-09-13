# Al Amal — Patient Questionnaires (public page)

The anonymous page patients use to fill in a questionnaire, served as a single
self-contained `index.html` — CSS, JavaScript and all. No build step, no
dependencies. Styled after the public feedback form (`AmalFeedback`).

**It is a phone app shell, like the feedback form.** `<body>` is
`position:fixed` and never scrolls; the header is pinned at the top, the submit
button in a bar pinned at the bottom, and only the content between them
(`#scroller`) scrolls. Zoom is off — `user-scalable=no` for Android, plus a
script guard against pinch, double-tap, `Ctrl`+wheel and `Ctrl`+`+`/`-`/`0` for
Safari and desktop. The keyboard doesn't move the layout: `--app-h` ignores it,
and `--kb-h` pads the scroller so the phone field can be lifted out from behind
it (the same `syncShell` guard the feedback page uses — see its README). Text
inputs are 16px so Safari never auto-zooms on focus. On screens under 700px
tall the intro line is dropped to leave room for the questions.

To check the phone layout on a desktop, load it in an iframe
(`<iframe src="/radiology" width="390" height="780">`) rather than trusting a
resized window.

**One page serves every questionnaire.** The first path segment is the
questionnaire's link, as set in the console (*Questionnaires → New
questionnaire → Page link*):

```
https://survey.alamalhospitaljo.com/radiology
```

`.htaccess` rewrites every path that isn't a real file to `index.html`; the
script reads the slug from `location.pathname` and loads the questions from
the API. An unknown or deactivated questionnaire shows "هذا الاستبيان غير متاح".

Each question is answered on the same five-point scale — جيد جداً, جيد, متوسط,
سيئ, سيئ جداً (`VeryGood … VeryBad` on the wire). Every question must be
answered before the page submits; the API enforces the same rule.

## The two things that must be set

**1. `API_BASE`** near the top of the `<script>` block — the AlAmalBusiness API
origin. Hardcoded on purpose (static page, no runtime to read config from, and
the value is public anyway).

**2. CORS** — this page's origin must be in the API's `Cors:AllowedOrigins`
(`AlAmalBusiness.Api/appsettings.json`). `https://survey.alamalhospitaljo.com`
is already listed; change both if the subdomain differs.

The console builds the "copy link" URLs from its own
`QUESTIONNAIRE_PUBLIC_URL` env var (default
`https://survey.alamalhospitaljo.com`) — keep it pointing here.

## Endpoints it calls

Both on `PublicQuestionnaireController`, rate limited 10 requests/minute per IP
(`PublicFormLimit`, shared with the feedback form's policy).

| | |
|---|---|
| `GET /api/public/questionnaire/{slug}` | `{ title, slug, description, departmentName, questions: [{ id, text }] }`, 404 if unknown/inactive |
| `POST /api/public/questionnaire/{slug}` | body `{ answers: [{ questionId, rating }], name?, phoneNumber? }` → `{ success, error }` |

Name and phone are optional. A phone, when typed, must hold 7–15 digits — the
page checks this before submitting, and the API enforces the same rule.

## Deploying

Same as `AmalFeedback`: point Hostinger's Git deployment (hPanel → Websites →
the subdomain → Advanced → GIT) at this repository's `main` branch, installed
into the subdomain's document root. Needs `mod_rewrite` (on by default).

## Local preview

`fetch` needs an origin, and the rewrite needs a server. On `localhost` /
`127.0.0.1` two query params help:

```
http://127.0.0.1:5500/radiology?api=http://localhost:5106
http://127.0.0.1:5500/?q=radiology&api=http://localhost:5106   (no rewrite available)
```

`?api=` is ignored on any other host. The local API must allow the preview's
origin, e.g. start it with `Cors__AllowedOrigins__3=http://127.0.0.1:5500`.
