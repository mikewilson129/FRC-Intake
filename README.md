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
- As of Version 10, **Submit is the primary button** on the finish screen
  (Print is now the secondary, plain-text action next to it). This is a
  UI change only — Print is still required for CRM data entry, and the
  underlying submission is still non-BAA/proof-of-concept as noted above.
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

## Changelog

### Version 11

- **Persistent floating Notes box** — a small "Notes" button, fixed to the
  bottom-right corner and visible on every screen, expands into a plain-text
  textarea (no formatting, no character limit, placeholder "Optional — use
  this space for any additional information"). Collapsed by default on
  every fresh load; once expanded it stays open across screen navigation
  until manually closed. Content is a single running note for the whole
  intake (not per-person, not reset between screens) and is built once
  outside the normal per-screen render cycle so it isn't recreated on
  navigation. Included in the printed output as a labeled "NOTES" section
  near the end when non-empty. Same behavior for everyone (staff or
  client) — not gated by role. Label/placeholder are translated
  (Spanish/Portuguese/Haitian Creole).
- **Fixed: "Continue intake" broke Back navigation.** Resuming an
  in-progress intake from the family list jumped straight to the saved
  step, but Back would then skip past all of that person's earlier
  sections straight to the family list, because Back relied on the
  browser-style history stack, which only contained the one step actually
  visited. Back on an intake screen now computes the actual previous step
  directly (skipping any auto-skipped steps) instead of depending on how
  the screen was reached, so it works the same whether you arrived via
  normal sequential navigation, "Continue intake", "Edit info", or Review's
  per-section "Edit."
- **Fixed: "Edit info" always exited to the family list.** Editing a
  household member's basic info (name/DOB/etc.) while their intake was
  still in progress and hitting Save always returned to the family list,
  even though the intake itself wasn't finished. Save now resumes the
  person's intake at whatever step they'd reached, matching "Continue
  intake." (Editing basics from the completed-intake Review screen still
  correctly returns to Review, unchanged.)
- **Fixed: Review's per-section Edit screen had three redundant buttons.**
  Editing a section from the Review & Edit screen showed "Back," "Back to
  review," and "Done editing" — all three returned to Review. "Back to
  review" is now removed entirely; Back navigates to the actual previous
  section (using the same fix as above), and "Done editing" is the only
  button that returns to Review.

### Version 10

- **S3 print/CRM summary reorder** — the printed safety section now lists
  Violence, Court, Detained, Run away, Exploited, Gang in that order. This
  is a print-output change only; the on-screen question flow and gating
  (e.g. Violence/Exploited/Gang only shown if the child is present to
  self-report) are unchanged.
- **CRA flag indent fix** — the "IDENTIFIED AS CRA — PRESENT TO CLINICIAN"
  line in the printed Needs & Follow-ups box now aligns with the other
  checkbox items instead of sitting flush left.
- **Food/clothing needs now pull through from the parent's direct answer
  too** — previously a child's intake only auto-filled "Yes" for food/
  clothing help if the parent picked "Food resources"/"Clothing resources"
  in their reason-for-visit. Now it also picks up the parent's answer to
  the separate direct yes/no question later in their intake.
- **Under-11 child intake shortcuts** — for children age 10 and younger,
  the "is the child present to answer directly" question is skipped and
  automatically set to No, and the "concerns about alcohol or drug use"
  question is skipped entirely. Ages 11+ are unaffected.
- **Court question info blurb (children only)** — the child's "involved in
  court?" question now includes the hint "For Care and Protection, CRA,
  criminal charges, etc." Not added to the adult version.
- **Finish screen changes**:
  - Removed "(backup copy)" wording from the Submit button and status
    messages.
  - Success message simplified to "Submitted." (no longer includes the
    backup submission id).
  - Failure message changed to "Submission failed — please try again or
    download/print a copy to send to the FRC."
  - Submit and Print swapped positions: Submit is now the primary
    (bold/colored) button in the main action position; Print is now a
    plain text-link style button, matching "Start over (erase all)".

### Cloudflare Worker (`frc-intake-poc`, separate deploy — not in this repo)

- Removed the unused email-notification code path (Resend integration).
  Submissions were never actually being emailed in practice — only stored
  in D1 — so this just removes dead code and the now-unneeded
  `RESEND_API_KEY` / `STAFF_EMAIL_TO` / `STAFF_EMAIL_FROM` secrets.

## Repo layout

```
index.html   the entire application
README.md    this file
```
