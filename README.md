# FRC Family Intake

A single-page, tablet-friendly intake form for a Family Resource Center (FRC).
Clients complete their own intake on a shared tablet in the language of their
choice; staff then print a CRM-ready data-entry copy. There is no backend
required for the core flow — everything lives in one file.

## How it's built

- **`index.html`** — the entire app: markup shell, CSS, and JS, in one file.
  No build step, no dependencies, no bundler. Open it in a browser and it runs.
- The JS renders screens into `#app` on the fly (no framework). Screens are
  defined as functions on the `SCREENS` object (`SCREENS.welcome`,
  `SCREENS.famBasics`, `SCREENS.intake`, `SCREENS.review`, `SCREENS.finish`,
  etc.) — see the `/* ============ SCREENS ============ */` section.
- Answers are kept in memory only. Nothing persists across a page refresh —
  this is intentional (see Privacy below).
- `#printRoot` holds a separate, staff-facing print layout (`@media print`
  CSS) that's built from the same in-memory answers when staff hit Print.

## Language / translation

- `LANG` controls the active display language; `tr(s)` looks up `s` in the
  `I18N` dictionary for the current language and falls back to English if a
  string is missing.
- **Stored answers are always kept in English** — only the on-screen display
  is translated. This matters because the CRM print output is staff-facing
  and must match the CRM's English fields regardless of what language the
  client used to answer.
- Supported languages are listed in `LANGS`. Full dictionaries currently
  exist for a subset (e.g. Spanish, Portuguese, Haitian Creole); others fall
  back to English for untranslated strings.
- Translation dictionaries are drafts (see the comment above the Spanish
  dictionary) — new or changed client-facing strings should be reviewed with
  bilingual staff before going live, and any string missing from a
  dictionary will silently display in English, so watch for that during
  testing after adding new questions.

## Remote submission (proof of concept)

- `WORKER_URL` / `SUBMIT_TOKEN` and `submitIntake()` post the completed
  intake as JSON to a Cloudflare Worker endpoint, in addition to the
  in-browser print flow.
- This is explicitly a **proof of concept, not a BAA-covered channel** — see
  the comment directly above `submitIntake()`. Print-and-shred remains the
  fallback path if remote submission fails or is unavailable, and staff
  should treat remote submission as best-effort only.
- `SUBMIT_TOKEN` is a plain static token embedded in client-side code. It
  should be rotated whenever it may have leaked (e.g. after sharing the file,
  a public commit, or a compromised tablet), and should not be treated as a
  real secret — it's a light deterrent, not access control.

## Privacy / data handling

- Nothing is saved on the device. Closing or refreshing the page erases all
  answers — this is by design, since tablets are shared/public devices.
  Staff must print (or successfully remote-submit) before closing the page.
- Keep this in mind when testing: a refresh mid-test loses your progress,
  same as it would for a client.

## Making changes

Since everything is one file, a few conventions keep it maintainable:

1. **New questions/screens** go in the `SCREENS` object, following the
   pattern of existing screens (look at `SCREENS.intake` or `SCREENS.basics`
   for the general shape: render inputs, validate, advance).
2. **New client-facing strings** should be added as plain English string
   literals so `tr()` can pick them up, then added to each language's
   dictionary in `I18N` (or explicitly left out to fall back to English,
   with a note for translators).
3. **Any new field that should appear on the printed/CRM copy** needs a
   corresponding line in the print-building logic (the `p-*` classes /
   `#printRoot` construction) — adding a field to the on-screen flow alone
   will not make it show up on the printout.
4. **Don't add persistence** (localStorage, sessionStorage, cookies) without
   discussing it first — the no-persistence behavior is a deliberate privacy
   choice for a shared-device intake form, not an oversight.
5. Since there's no build step, test changes by opening `index.html` directly
   in a browser and clicking through the flow (including Print preview) in
   at least English and one other language before committing.

## Repo layout

```
index.html   the entire application
README.md    this file
```
