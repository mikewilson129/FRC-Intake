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

## Links (URL tags)

Add these to the end of the page address. They're read once when the page
loads and are never saved; they can be combined.

| Tag | What it does | Example |
|---|---|---|
| `?cra` | **CRA mode, locked on** for the whole session. This is the link staff send to CRA families. Welcome shows a "CRA mode" label that can't be tapped off. | `index.html?cra` |
| `?lang=es` / `?lang=pt` / `?lang=ht` | Starts in Spanish / Portuguese / Haitian Creole (the dropdown shows it selected and still works). Anything else starts in English. | `index.html?lang=es` |
| both | CRA mode in Spanish | `index.html?cra&lang=es` |

- Without `?cra`, staff can turn CRA mode on or off with the small, quiet
  "CRA mode" button on the Welcome screen (off by default; English only,
  since it's staff-facing).
- **CRA mode** is for families coming in for a CRA (open, or at risk of
  one). Staff decide which families are CRA; families aren't asked. In CRA
  mode, "No one else — finish up" needs a completed adult intake **and** a
  completed child intake (one child is enough). Until then the household
  screen explains that both are needed, and the printout shows a "CRA mode"
  line near the top. Nothing else about the form changes.
- "Start over" reloads the same address, so a `?cra` / `?lang=` link keeps
  its mode and language for the next family.

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
- Submit gives up after 30 seconds (`SUBMIT_TIMEOUT_MS`) and shows "Submission
  timed out — please submit again…", so a bad connection can't leave the
  button stuck on "Submitting…". A timed-out request may still have reached
  the server, so submitting again can occasionally store a duplicate copy.
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
- Once anything has been entered, the page asks the browser to warn before
  a refresh or close, and pull-down-to-refresh is disabled. Browsers show
  their own generic wording for the warning, and **iPhone/iPad Safari often
  skips it**, so this reduces accidental data loss but can't fully prevent
  it on Apple devices. The "Start over" button deliberately skips the warning.

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
6. **A greyed-out button should say why.** Screens with required answers
   pass `why` to `shell()` (shown under the button only while it's greyed
   out). Text boxes that affect the button call `setNext(ok, why)` as the
   person types instead of re-rendering — don't look the button up with
   `document.querySelector(".btn-primary")`.
7. **Finish requirements live in `finishRule()`** — change the rule there,
   not in the household screen or Submit button separately.
8. **Age-dependent questions:** if a question only applies at certain ages
   and is auto-answered otherwise, also clear the auto-answer when it stops
   applying (see the `craUnder12`, `childPresentAuto` and `hasJobAuto` flags),
   so a corrected date of birth asks the question for real.

## Future ideas (not built yet)

- **Save progress (internal vs. external).** Clients may fill this out on
  their own iPhones at home, where Safari often skips the leave-page
  warning, so an accidental refresh can still erase everything. Saving
  progress on the device would fix that, but conflicts with the no-saving
  privacy choice on the shared Android tablets at the FRC. A likely shape:
  an "external" (client's own phone) mode that saves progress, and an
  "internal" (shared tablet) mode that keeps today's no-saving behavior.
- **Child questions that can't be skipped (to review).** Three child-intake
  questions must be answered before Continue turns on, and have no Skip
  link: "Is there an active CRA for {name}?" (ages 12+), "Is {name} present
  and able to answer a few questions directly?" (ages 11+), and "Is {name}
  currently living with their family?" (all ages). Decide whether each
  should stay required or get a Skip / "Not sure" option. (CRA mode, added
  in Version 15, doesn't change them.)
- **Households with no adult.** "No one else — finish up" needs a completed
  adult intake (plus a child's in CRA mode), so a household with no adults
  entered (e.g. a youth on their own) can't finish.
- **CRA mode — possible next steps (left out of Version 15 on purpose):** a
  CRA screening question, a "which child is this visit about" step, an "at
  risk of a CRA" answer, and a staff override for the finish rule.
- **More personalized wording on the printout.** The printed/CRM copy
  intentionally keeps the CRM's own wording (e.g. "This child/youth feels
  safe in his/her home") so staff can match it to CRM fields. Making it more
  personal (names, they/them, etc.) needs a decision about how far it can
  drift from the CRM field labels.

## Changelog

### Version 15

- **CRA mode.** For families coming in for a CRA (open, or at risk of one),
  who need both an adult and a child intake. Turn it on with the `?cra` link
  (locked on for the session) or the quiet "CRA mode" button on Welcome (off
  by default; English only). In CRA mode:
  - "No one else — finish up" also needs at least one completed child
    intake (under 18), on top of the completed adult intake. One child is
    enough even if the household has several. Unfinished extra intakes
    still get the Version 14 "finish without completing theirs?" pop-up.
  - Until that's met, the household screen shows "We need to have a full
    intake for you and for your child that was referred to us. Please
    complete both before finishing. We only need name, DOB, and health
    insurance indicator for other members of the family.", plus "If your
    child isn't listed yet, tap '+ Add a household member'." when no child
    is listed. The line under the
    greyed-out button reads "This button turns on once an adult's intake
    and a child's intake are both complete."
  - Hidden (they contradict the rule): the "Complete an intake for a child
    when they need direct referrals…" hint, and the "…covered by your
    intake — you don't need one for your child…" note under the adult.
  - The printout shows "CRA mode — this visit required an adult intake and
    a child intake." near the top.
  - No change to any question, its wording, or the under-12 CRA auto-skip.
- **Finish rule in one place (`finishRule()`).** The household screen's
  "No one else — finish up" and the staff **Submit** button both check it,
  so an intake that doesn't meet the rule can't be submitted by any path.
  (Version 13 let a household finish with only a child's intake — even with
  no adult listed at all; Version 14 started requiring a finished adult.)
  When no one 18+ is on the household list, it now also says "If you're not
  on this list yet, tap '+ Add a household member' to add yourself."
- **Housing question is one list again** ("Your family is:", as in
  Version 11): Living in your own apartment or home (owned or rented);
  Homeless but Sheltered; Homeless and Not Sheltered; Decline to answer.
  Stored values are unchanged from Version 12 (the first stores the CRM
  wording "…their own…"; Decline stores "Not Answered"); info blurbs stay
  on the two homeless answers; still required. Removed the group step and
  its strings ("Housed/sheltered", "Unhoused/unsheltered", "Which fits
  best?"). Note: the submitted JSON no longer includes
  `family.livingGroup`.
- **Language link.** `?lang=es`, `?lang=pt` or `?lang=ht` starts the form
  in that language; anything else starts in English. Combines with CRA mode
  (`?cra&lang=es`). See "Links (URL tags)" above.
- **Translations:** the new household-screen strings in Spanish,
  Portuguese and Haitian Creole (drafts — review with bilingual staff).
  Spanish V14 wording updated after staff review: "respuesta sobre el
  seguro médico".

### Version 14

Fixes for the "intake freezes / Continue won't work" reports, plus other
bugs found while investigating.

- **Fixed: Continue stuck greyed out on "Your family."** The button only
  re-checked itself when the phone number was typed, so entering the phone
  before the last name left it greyed out (and erasing the last name
  afterward left it on). All three fields now update it as you type.
- **A greyed-out button now says why.** A short line under the button lists
  what's still needed (e.g. "Still needed: family last name, 10-digit phone
  number" or "To continue, please answer the question above."). The phone
  message ("needs 10 digits") and email message now appear while typing;
  a number entered with a leading 1 gets "Leave off the 1 at the start…".
- **Finishing with an unfinished intake.** "No one else — finish up" now
  turns on as soon as one adult's intake is complete (before, any started
  intake — even one opened by accident — kept it greyed out until finished,
  with no way to cancel). If anyone's intake is unfinished, a pop-up asks
  "John's intake isn't complete. Finish without completing theirs?" (Finish
  anyway / Go back). The family list now shows "Intake in progress" for
  anyone started. Unfinished intakes still print, marked "INTAKE NOT
  COMPLETED" with "Services needed: Yes (intake not completed)".
- **Submit can't get stuck.** It gives up after 30 seconds with "Submission
  timed out — please submit again, or print a copy to send to the FRC," and
  any error now releases the button. After a successful submit the button
  shows "Submitted ✓" (prevents accidental duplicates); tapping Edit
  re-opens Submit so changed answers can be sent.
- **Date-of-birth corrections ask the right questions.** If a DOB change
  moves someone between adult and child questions, or across the child age
  cut-offs (11 and 12), their intake re-opens with a note explaining why,
  answers that still apply are kept, and newly relevant questions (e.g.
  CRA at 12+) are asked instead of keeping an automatic "No." The other
  set's answers are parked in the background and restored if the DOB is
  changed back; they're never printed or submitted.
- **Printed dates use the current date** (the day it's printed/submitted),
  not the day the page was loaded.
- **Fixed: Back from "First, about you" erased what was typed** for the
  first person (same for adding someone else).
- **Fixed: Review → Edit opened the wrong question** for sections answered
  automatically (CRA for under-12s; an adult's Basic Needs when food and
  clothing were picked as reasons). Those sections no longer show Edit.
- **Wording:** "If a question doesn't feel comfortable to answer, you're
  welcome to skip it" is now "Most questions can be skipped if you'd rather
  not answer" (Welcome screen and screen intros), since a few are required.
- **Translations:** added Spanish/Portuguese/Haitian Creole for all new
  text, plus strings that were showing in English ("There are no wrong
  answers." on the child safety screen, the Spanish Welcome-screen language
  tip, and some Review values like "Own words (blank)"). Removed duplicate
  Spanish entries ("Preferred name" had two different translations). New
  translations are drafts — review with bilingual staff.
- **Minor:** typing certain words (e.g. "constructor") in the role search
  no longer crashes the screen; the "papa" role shortcut keeps Grandfather.
- **Unchanged on purpose:** the Notes box — closing it with ✕ uncovers
  Continue.

### Version 13

- **Fixed: blank page after editing basic info from Review.** On a
  person's Review screen, tapping Edit on the basic-info section (name,
  date of birth, etc.) and then Save left the page blank under the header.
  Save now returns to that person's Review.
- **Safety net for screen errors.** If a screen ever fails to load, the
  page no longer goes blank. It shows "Something went wrong on this
  screen" with a note that answers are still saved, a **Go back** button
  (returns to the previous screen), and a staff-only **Staff: print what's
  been entered** button. The error is also logged to the browser console
  ("Screen failed: …") to help with troubleshooting. The message and
  "Go back" are translated (Spanish/Portuguese/Haitian Creole); the staff
  print button stays English only.

### Version 12

- **Reordered the family screens:** (1) Your family — last name, phone,
  email; (2) Your home — housing question, then address; (3) Household &
  income — household type, then income; then "How did you hear about us."
- **Two-level housing question** ("Your family is:"), like the referral
  question: Housed/sheltered, Unhoused/unsheltered, or Decline to answer,
  then the specific answer. "Homeless but Sheltered" is offered under both
  groups on purpose (it's the gray-area answer). The screen shows "Living in
  *your* own apartment…", but the printout keeps the CRM wording ("…*their*
  own…"); "Decline to answer" records the CRM value "Not Answered."
  After a group is picked it shrinks to a small tag (with "change"), and
  "Which fits best?" is shown as a bold question, so it's clear a second
  answer is still needed.
- **Household question reworded** to "Which best describes the caregivers
  living in your household?"
- **4-step progress bar** at the top of every screen after Welcome:
  Step 1 Family information (through adding household members), Step 2 Your
  information (the parent's own intake), Step 3 Other family members, Step 4
  Review & submit. The percentage restarts within each step and is
  approximate by design. The family list in Steps 2/3 and the Step 4 screen
  show the step with no percentage; another person's intake shows its own %.
- **Notes box enlarged** (about 480px wide, 260px-tall text area).
- **Bolded the instructions** on "First, about you" and the other
  household-member screens, and the **category titles** on "How did you
  hear about us."
- **Leave-page warning + pull-to-refresh block** (see Privacy above for the
  iPhone limitation).
- **Wording consistency pass (on-screen only; printout untouched):** the CRA
  question now uses the child's name; the child's race question reads
  "…{name} identifies as"; the "other community agencies" question is a
  full question for both adults and children; the DTA/MassHealth hints no
  longer say "you/your" on children's screens; the preferred-name hint says
  "you" on "First, about you."
- **Translations** (Spanish/Portuguese/Haitian Creole) added for every new
  or changed string. Also fixed a translation bug: the word "Single" was
  shared by the household-type and marital-status questions, so Spanish and
  Portuguese marital status showed "single parent" and Haitian Creole
  household type showed "unmarried." Each now translates correctly; the
  English and stored CRM values are unchanged.

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
  sections straight to the family list, because Back relies on the
  navigation history stack, which only contained the one step actually
  visited. Entry points that jump into the middle of an intake ("Continue
  intake" resuming a saved step, "Edit info" resuming, and Review's
  per-section "Edit") now seed the history stack with an entry for every
  earlier non-auto-skipped step via a new `enterIntakeAt()` helper, the
  same as if the user had actually clicked through them — so the ordinary
  Back button (unchanged) naturally lands on the right previous step no
  matter how the screen was reached.
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
