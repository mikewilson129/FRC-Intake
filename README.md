# FRC Family Intake

A single-page, tablet-friendly intake form for a Family Resource Center (FRC).
Families complete their own intake in the language of their choice, either
on their own phone or computer (the plain link — "remote mode") or on the
FRC's shared tablet (the `?office` link), where staff then print a CRM-ready
data-entry copy. There is no backend
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
- **`release-done.html`** — the small "Thank you" page the signed release
  lands on (see "Release of information" below). It's the only other page.

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
- The language menu marks Haitian Creole "(beta)". Portuguese came out of
  beta in Version 16 after a native-speaker review (`LANG_OPTIONS`).
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
| `?office` | **In-office (FRC tablet) version** for the whole session: "All set!" with the hand-off to staff, the staff panel (Submit, Print, Edit, Start over) and the staff "CRA mode" switch. Without it the form is in **remote mode** (since Version 16.2): for families on their own phone or computer — see "Remote mode" below. | `index.html?office` |
| `?cra` | **CRA mode, locked on** for the whole session. This is the link staff send to CRA families. Welcome shows a "CRA mode" label that can't be tapped off. | `index.html?cra` |
| `?lang=es` / `?lang=pt` / `?lang=ht` | Starts in Spanish / Portuguese / Haitian Creole (the dropdown shows it selected and still works). Anything else starts in English. | `index.html?lang=es` |
| `?releasetest` | **Staff only — never send to families.** Shows the release buttons when they're switched off for everyone (`RELEASE_FOR_EVERYONE = false`), so staff can test them on the live site. While they're on for everyone it changes nothing. See "Release of information" below. | `index.html?releasetest` |
| combined | e.g. the tablet in CRA mode in Spanish | `index.html?office&cra&lang=es` |

- On the tablet (`?office`) without `?cra`, staff can turn CRA mode on or
  off with the small, quiet "CRA mode" button on the Welcome screen (off by
  default; English only, since it's staff-facing). Family (remote) links
  have no such button; use a `?cra` link.
- **CRA mode** is for families coming in for a CRA (open, or at risk of
  one). Staff decide which families are CRA; families aren't asked. In CRA
  mode, "No one else — finish up" needs a completed adult intake **and** a
  completed child intake (one child is enough). Until then the household
  screen explains that both are needed, and the printout shows a "CRA mode"
  line near the top. Nothing else about the form changes.
- "Start over" (tablet only) reloads the same address, so an `?office`
  link keeps the tablet version, and `?cra` / `?lang=` keep their mode and
  language, for the next family.

### Staff links

The live form is at `https://mikewilson129.github.io/FRC-Intake/`. Since Version 16.2 the plain link is the
**family (remote) version**, for families filling it out on their own phone
or computer. The **FRC tablet** uses the `?office` links.

**Family links** (send these to families):

| Use | Link |
|---|---|
| Standard (English) | `https://mikewilson129.github.io/FRC-Intake/` |
| Spanish | `https://mikewilson129.github.io/FRC-Intake/?lang=es` |
| Portuguese | `https://mikewilson129.github.io/FRC-Intake/?lang=pt` |
| Haitian Creole | `https://mikewilson129.github.io/FRC-Intake/?lang=ht` |
| CRA (English) | `https://mikewilson129.github.io/FRC-Intake/?cra` |
| CRA — Spanish | `https://mikewilson129.github.io/FRC-Intake/?cra&lang=es` |
| CRA — Portuguese | `https://mikewilson129.github.io/FRC-Intake/?cra&lang=pt` |
| CRA — Haitian Creole | `https://mikewilson129.github.io/FRC-Intake/?cra&lang=ht` |

**FRC tablet links** (in the office):

| Use | Link |
|---|---|
| Tablet (English) | `https://mikewilson129.github.io/FRC-Intake/?office` |
| Tablet — Spanish | `https://mikewilson129.github.io/FRC-Intake/?office&lang=es` |
| Tablet — Portuguese | `https://mikewilson129.github.io/FRC-Intake/?office&lang=pt` |
| Tablet — Haitian Creole | `https://mikewilson129.github.io/FRC-Intake/?office&lang=ht` |
| Tablet CRA (English) | `https://mikewilson129.github.io/FRC-Intake/?office&cra` |
| Tablet CRA — Spanish | `https://mikewilson129.github.io/FRC-Intake/?office&cra&lang=es` |
| Tablet CRA — Portuguese | `https://mikewilson129.github.io/FRC-Intake/?office&cra&lang=pt` |
| Tablet CRA — Haitian Creole | `https://mikewilson129.github.io/FRC-Intake/?office&cra&lang=ht` |

The tablet's bookmark / home-screen shortcut should be the `?office` link
(usually Tablet, English); "Start over" keeps it.

On the language links the family can still switch languages from the
dropdown. On the CRA links, CRA mode stays on and can't be turned off on
screen.

**Staff-only test links** (never send these to families). The release
buttons are on for everyone (since Version 16.1), so these aren't needed
now. If the buttons are ever switched off again, these still show them so
staff can test on the live site:

| Use | Link |
|---|---|
| Tablet, with release buttons | `https://mikewilson129.github.io/FRC-Intake/?office&releasetest` |
| Family (remote), with release buttons | `https://mikewilson129.github.io/FRC-Intake/?releasetest` |

`releasetest` combines with the other tags too (e.g. `?lang=es&releasetest`).

### Hosting-move checklist

When the intake form moves to a new address, check each of these:

- [ ] Jotform release forms → Settings → Thank You page must redirect to
  release-done.html at the form's current address. Update it whenever the
  intake form moves (e.g. to intake.pw4c.org), or the panel won't
  auto-close. Both forms, each with its own `?form=` tag:
  - CFFS/JRI release (`262673921694064`): `…/release-done.html?form=262673921694064`
  - Other agencies/people release (`262774481459066`): `…/release-done.html?form=262774481459066`
- [ ] Update the FRC tablet's bookmark / home-screen shortcut to the new
  address **with `?office`** (a plain link opens the family version).
- [ ] Update the family and tablet links in "Staff links" above.

## Remote mode (the default link)

Added in Version 16, for families filling out the form on their own phone
or computer. Since Version 16.2 it's what the plain link opens; the FRC
tablet uses `?office` instead. There's no on-screen switch, and it
combines with `?cra` and `?lang=`. The questions are
the same (a few are worded differently — see the table below); what
changes:

- **The ending.** "No one else — finish up" goes to **"Almost done"**:
  a "Sign release for {first name}" button for each person with a
  completed intake (when the release buttons are on — see "Release of
  information"), then a **Submit** button the family presses. Releases are
  encouraged but not required. "‹ Back" works until they submit.
  - **Unsigned releases:** while anyone listed hasn't signed their CFFS/JRI
    release, Submit looks greyed out (it turns green once everyone has
    signed) but still works: tapping it first shows "Please complete the releases for each
    household member listed here. We need these releases completed for us
    to appropriately serve your family." with **Go back** (the prominent
    button) and **Submit anyway** (so staff doing an intake over the phone
    can click past). It's asked once: a retry after a failed send isn't
    asked again. Not shown when the release buttons are switched off.
  - Success → **"Intake sent"**: "Thank you! Your intake was sent to the
    FRC. Someone from the FRC will follow up with you." / "You can close
    this page now." No Back, Print, staff panel or Start over; the Notes
    button is hidden (a note typed now wouldn't be sent); closing the page
    no longer asks "Leave site?".
  - Failure (or a 30-second timeout) → "Please try again. If it still
    doesn't work, call the FRC at 978-296-8080." Submit stays available.
- **No printing.** The staff data-entry copy is only built as the text
  that gets submitted; it never goes into the page's print area. If the
  family uses the browser's own Print (or Share > Print), all that prints
  is "Printing isn't available." — after Submit, "Printing isn't available.
  Your intake was sent to the FRC." — never the staff copy (with its Needs &
  Follow-ups box) or the screen. Tablet printing is unchanged.
- **No print fallback.** So the remote submission (proof of concept — see
  below) is the only way a remote intake reaches the FRC. If it keeps
  failing, the family is asked to call.
- **Lines written for the FRC tablet** show a remote version instead. The
  phone number appears only on Welcome, the household list and the
  submit-failed message; elsewhere families are pointed to the Notes box
  or told to try their best:

  | Where | Tablet (unchanged) | Remote |
  |---|---|---|
  | Welcome | A staff member is nearby if anything is confusing — just ask. Most questions can be skipped… | If anything is confusing, call us at 978-296-8080. Most questions can be skipped… |
  | Welcome | Nothing is saved on this tablet. When this page is closed, everything on it is erased. | Nothing is saved on this device. If this page is closed before you submit, everything on it is erased. |
  | Your home | State: Massachusetts (tell a staff member if that's not right) | State: Massachusetts — if you live in a different state, please let us know in the Notes section. |
  | How did you hear about us (1) | …Pick the closest fit — a staff member can help if you're not sure. | …Pick the closest fit — if you're not sure, try your best and tell us in the Notes section in your own words. |
  | How did you hear about us (2) | …This question can't be skipped — ask a staff member if none of these look right. | …This question can't be skipped — if none of these look right, try your best and tell us in the Notes section in your own words. |
  | "One suggestion first" | If you're staff or have a reason to go in a different order, you can continue anyway. | If you have a reason to go in a different order, you can continue anyway. |
  | Your household | If you're unsure, just ask a staff member — that's what we're here for. | If you're unsure, call us at 978-296-8080 — that's what we're here for. |
  | What brings you in — screen title (adult) | What brings you in | How we can help |
  | What brings you in — screen title (child) | What brings {name} in | How we can help |
  | What brings you in — question (adult) | What brings you in today? | What can we help you with? |
  | What brings you in — question (child) | What brings you in for {name} today? | What can we help you with for {name}? |
  | What brings you in — hint (adult and child) | A few words is plenty — a staff member can help you sort it out. | A few words is plenty — we can help you sort it out later. |
  | Child present (11+) | Is {name} present and able to answer a few questions directly? | Is {name} with you and able to answer a few questions directly? |
  | Court status (child 12+) | If you're not sure, a staff member can help. | If you're not sure, just try your best. |

  Also hidden in remote mode: the staff "CRA mode" switch on Welcome (a
  `?cra` link still shows its "CRA mode" label) and the "Staff: print
  what's been entered" button on the "Something went wrong" screen. The
  tablet-only "All set!" screen and staff panel never appear.
- **Printout / submission** (staff-facing, English): a "Completed remotely —
  the family filled out and submitted this intake on their own device."
  line near the top (above the CRA mode line); the header says "Submitted
  from the family's own device" instead of "Printed from the intake
  tablet"; "Category chosen by the family:" instead of "…on tablet:"; and a
  signed release shows "signed remotely" instead of "signed on tablet".
- **Phone number:** `FRC_PHONE` in `index.html` (one place). It's kept on
  one line on screen.
- The review/printout labels (CRM wording, e.g. "Child present to answer
  directly?") are unchanged.

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

## Release of information (signed in the intake)

Added in Version 16. On the client "All set!" screen (before staff tap
"Next steps"), or remote mode's "Almost done" screen, each person with a
completed intake gets a **"Sign release for {first name}"** button. It
opens the CFFS/JRI release (`https://form.jotform.com/262673921694064`) in
a full-screen panel inside the intake page, already filled in:

| Jotform field | Filled with |
|---|---|
| `clientname[first]`, `clientname[last]` | the person's legal first/last name (not preferred name) |
| `dob[month]`, `dob[day]`, `dob[year]` | their date of birth (`03`, `07`, `1985`) |
| `input26[shorttext-1]` | the adult giving consent: an adult signs for themselves; for a child, the primary contact (the first adult whose intake was started; the printout marks them "Primary Contact: Yes"), or else the first adult with a completed intake. Jotform copies it to the second spot itself. |
| `language` | the intake's language: `en`, `es`, `pt`, or `ht` (Haitian Creole / Kreyòl Ayisyen) |

Everything prefilled can still be changed in Jotform.

- **On for everyone** (tablet and remote) since Version 16.1. To hide the
  buttons again, set `RELEASE_FOR_EVERYONE = false` in `index.html`; staff
  can still test them with the staff-only `?releasetest` links (see
  "Staff-only test links"). With them hidden, the tablet links work as in
  Version 15 apart from the wording changes in the Version 16 changelog.

- **The intake page never leaves.** The release loads in a sandboxed iframe
  (scripts, forms, same-origin and popups allowed; no top navigation of any
  kind), so nothing in the release can take the intake page away. The panel
  always shows a **Close** button. The leave-page warning is unchanged.
- **Jotform setting (one time):** set the form's Thank You page to redirect
  to `https://mikewilson129.github.io/FRC-Intake/release-done.html?form=262673921694064`. It has
  to be on the same site as the intake, or the intake ignores it. The
  `?form=` tag gives people who use the form directly (not in the intake)
  a "Fill out another release" button on that page; it's optional. Adding
  `?lang=es` / `pt` / `ht` shows that page in another language (the old `bzj` still works),
  but it's optional: the intake covers it with its own translated "Thank you".
  Until the redirect is set, people can still sign, but the panel shows
  Jotform's own thank-you page, doesn't close by itself, and nobody is
  marked as signed; tap Close. If the form moves, see the hosting-move
  checklist above.
- **After signing:** the panel shows "Thank you" for 2.5 seconds
  (`RELEASE_THANKS_MS`) and closes;
  that person's button changes to **"Release signed ✓"** and no longer
  opens the release, so a tap can't create a duplicate signed release in
  Jotform; the printout's Releases box
  shows a checked box: `☑ Name - CFFS/JRI — signed on tablet` (remote
  mode: "signed remotely"); unsigned releases keep the empty box `☐`; and
  the submitted JSON has `releaseSigned: true` on that person (the field
  is left out for anyone who didn't sign).
- **Closed early, or the release doesn't load:** nothing else changes.
  Submit and Print work as before, and the button still says "Sign
  release for…", so it can be tried again.
- **Unsigned releases are flagged (skippable):** on the tablet, tapping
  "Next steps" on "All set!" first asks the family to complete them
  ("Go back" / "Continue anyway"), and the staff panel's Submit names who
  hasn't signed ("Go back" / "Submit anyway"); in remote mode, Submit asks
  the family (see "Remote mode"). Each is asked once per intake.
- **Staff panel (tablet):** "Next steps" shows, for each person with a
  completed intake, **"Open release for {first name}"** if they haven't
  signed (opens it straight away), or **"Redo release for {first name}"**
  if they have (warns first that signing again creates a second signed
  release in Jotform). Staff-facing, English only. Remote mode has no staff
  panel.

### Other releases (tablet staff panel, after Submit)

Added in Version 16.3, for releases to outside agencies and people, using a
second Jotform form (`https://form.jotform.com/262774481459066`). Tablet
(`?office`) only; remote mode doesn't have it.

- **Only after Submit.** Until staff press Submit, the staff panel says
  "Releases appear here after you press Submit." — a reminder not to skip
  Submit. After a successful Submit a **Releases** section appears. If staff
  tap Edit it hides until they Submit again (anything signed stays ticked).
- **Suggested from the intake:** one button per item on the printout's
  Releases list besides CFFS/JRI, e.g. "Testa — DCF", "Testa — Lahey
  Health", "Kiddo — School release". That's each of DCF, DDS, DMH and DYS
  checked on the agencies question, each community agency typed (as
  typed), and a school release for each child — only for people with a
  completed intake.
- **"+ Complete another release"** for anything else; it can be used any
  number of times.
- **Prefill:** only the consent name, in the first blank of `input22`
  (`input22[firstname-1]`): an adult's own suggestion → that adult; a
  child's suggestion or "Complete another release" → the primary contact
  (as for CFFS/JRI). Everything else is filled in on the form.
- **After signing** (same panel, Close button, 2.5-second "Thank you"): a
  suggestion turns into e.g. "Testa — DCF ✓" and can't be opened again (no
  duplicates; use "Complete another release" for a redo). "Complete another
  release" shows a count ("Other releases signed: 2").
- **Ticks follow the button, not the form.** The intake can't see what's
  typed in Jotform, so it ticks the suggestion whose button opened the
  panel. Jotform's submissions are the real record.
- **Printout:** signed suggestions show `☑ Name - DCF — signed on tablet`,
  plus `☑ 2 other releases signed on tablet` for "Complete another
  release". These only appear on printouts made after signing; the copy
  sent to the FRC at Submit doesn't have them.
- **Jotform setting:** this form's Thank You page must redirect to
  `https://mikewilson129.github.io/FRC-Intake/release-done.html?form=262774481459066`,
  or the panel won't close by itself and nothing is ticked (tap Close).
  The `?form=` tag gives people who use the form directly a "Fill out
  another release" button back to it.

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
- Remote mode runs on the family's own phone or computer and saves
  nothing there either. The leave-page warning works the same until they
  submit (iPhone Safari often skips it, as above), so a refresh before
  Submit erases everything; after a successful Submit closing the page
  doesn't warn.
- The release panel sends the person's name and date of birth to Jotform
  in the release address (that's how prefilling works), so Jotform receives
  them as soon as the panel opens, even if it's closed without signing. The
  intake never logs, saves or submits that address. If Jotform ever tries to
  navigate the intake page, the browser blocks it and may print the address
  in its own developer-console warning; that stays on the device.

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
8. **Lines written for the FRC tablet** use `byMode(tabletLine,
   remoteLine)` so remote mode shows its own version. It returns the
   untranslated line; if the line has `{n}`, wrap it in `trn(…, FRC_PHONE)`
   or `trn(…, name)`. Add both lines to each dictionary.
9. **Age-dependent questions:** if a question only applies at certain ages
   and is auto-answered otherwise, also clear the auto-answer when it stops
   applying (see the `craUnder12`, `childPresentAuto` and `hasJobAuto` flags),
   so a corrected date of birth asks the question for real.

## Future ideas (not built yet)

- **Save progress (internal vs. external).** Clients may fill this out on
  their own iPhones at home (remote mode, Version 16), where Safari often
  skips the leave-page warning, so an accidental refresh can still erase everything. Saving
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
- **Separate address for a second adult** (e.g. divorced parents at
  different addresses) — discuss for a future version; needs a place in
  the CRM. For now, families use the Notes box.
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

### Version 16.6

- **Fix: Haitian Creole releases open in Kreyòl again.** The release form's
  Kreyòl translation in Jotform was moved to the correct language, Kreyòl
  Ayisyen, whose code is `ht` (it had been under `bzj`, Belize Kriol). The
  intake now opens the release with `language=ht`, and the release "Thank
  you" page's "Fill out another release" link uses `ht` too. Without this,
  Haitian Creole families saw the release in English. The "Thank you" page
  still accepts the old `?lang=bzj`. Name, date of birth and consent
  prefill were never affected.

### Version 16.5

- **Remote "Almost done": Submit looks greyed out until every CFFS/JRI
  release is signed**, then turns green. It still works while grey —
  tapping it shows the "Please complete the releases…" warning, as before.
  The tablet is unchanged.

### Version 16.4

- **Fix: the consent name now fills in on the agency/people release.** The
  first blank of its consent line is a "first name"-type blank, so its
  prefill address is `input22[firstname-1]` (Version 16.3 used
  `input22[shorttext-1]`, which Jotform ignored).

### Version 16.3

- **Other releases on the tablet staff panel, after Submit.** A Releases
  section with buttons suggested from the intake (state agencies,
  community agencies, school release for each child) and "+ Complete
  another release" (any number of times), using the outside agencies/people
  release form. Only the consent name is prefilled. Signed suggestions get
  a ✓ on screen and ☑ on later printouts. Before Submit the panel says
  "Releases appear here after you press Submit." Tablet only. See "Other
  releases" under "Release of information".
- **Warnings about unsigned CFFS/JRI releases** (skippable, each asked once
  per intake; "Go back" is the prominent button):
  - **Remote Submit** and the **tablet's "Next steps"** (the family's
    hand-off): "Please complete the releases for each household member
    listed here. We need these releases completed for us to appropriately
    serve your family." — **Go back** / **Submit anyway** (remote) or
    **Continue anyway** (tablet). Spanish, Portuguese and Haitian Creole are
    drafts.
  - **Tablet staff Submit** (staff-facing, English): "CFFS/JRI release not
    signed yet for: Kiddo. Use "Open release for…" above, or submit
    anyway." — **Go back** / **Submit anyway**.
- **`release-done.html`:** opened directly (not inside the intake) with
  `?form=` set to one of the two release forms, it shows a "Fill out another
  release" button back to that form (English, Spanish, Portuguese, Haitian
  Creole; translations are drafts).
- **Jotform:** set the Thank You redirect of the new form to
  `release-done.html?form=262774481459066`, and of the CFFS/JRI form to
  `release-done.html?form=262673921694064` (see the hosting-move checklist).

### Version 16.2

- **The plain link is now the family (remote) version.** The FRC tablet
  (in-office) version needs `?office` (it combines with `?cra`, `?lang=`
  and `?releasetest`). **Update the tablet's bookmark / home-screen
  shortcut to the `?office` link**, or it will open the family version (no
  staff panel or Print). The `?remote` tag is gone (remote is the
  default). Staff links, URL tags and the hosting-move checklist updated.
- **Printout: signed releases get a checked box.** In the Releases box, a
  release signed in the intake now shows `☑ Name - CFFS/JRI — signed on
  tablet` (remote: "signed remotely") instead of `☐ … — signed on tablet ✓`.
  Unsigned releases keep the empty box.

### Version 16.1

- **Release buttons switched on for everyone** (tablet and remote):
  `RELEASE_FOR_EVERYONE = true`. The Jotform Thank You redirect to
  `release-done.html` is set. `?releasetest` now only matters if the
  buttons are switched off again.
- **Shorter "Thank you" after signing:** the panel now closes after 2.5
  seconds instead of 5.

### Version 16

- **Sign the CFFS/JRI release on the tablet.** "All set!" now shows a
  "Sign release for {first name}" button for each person with a completed
  intake (usually 1–2). It opens the prefilled Jotform release in a
  full-screen panel inside the intake, with a Close button; the intake page
  itself never navigates, so nothing is lost. After signing, the panel says
  "Thank you", closes after 5 seconds, and marks the person: their button
  changes to "Release signed ✓" and stops opening the release (no duplicate
  signed releases in Jotform), "signed on tablet ✓" next to their CFFS/JRI
  line in the printout's Releases box, and `releaseSigned: true` in the
  submitted JSON.
  Unsigned people print exactly as before. Staff can redo a signed release
  from the staff panel ("Redo release for {first name}", after a warning).
  See "Release of information" above, including the one-time Jotform Thank
  You redirect and the new hosting-move checklist.
- **Release buttons hidden until tested.** They only appear with the
  staff-only `?releasetest` tag until `RELEASE_FOR_EVERYONE` is set to
  `true`; without the tag, the tablet links' screens, printout and
  submitted JSON are the same as Version 15 except for the household
  wording and Portuguese changes below. (This replaces the earlier
  `RELEASE_ENABLED` switch.)
- **Staff panel "Open release for {first name}"** for anyone with a
  completed intake who hasn't signed (no warning), next to the existing
  "Redo release for {first name}" for people who have.
- **Remote mode (`?remote`)** for links families complete on their own
  phone or computer. Locked on for the session; combines with `?cra` and
  `?lang=`. New ending: "Almost done" (release buttons + Submit; releases
  encouraged, not required) → "Intake sent" (no Print, staff panel or
  Start over). A failed submit says "Please try again. If it still doesn't
  work, call the FRC at 978-296-8080." Lines written for the tablet get
  remote versions (the phone number only on Welcome, the household list and
  the submit-failed message; the state line and referral hints point to the
  Notes box; "How we can help" / "What can we help you with?"; "Is {name}
  with you…?"), the CRA
  switch and crash-screen staff print button are hidden, and the printout
  gets a "Completed remotely" line (plus "Submitted from the family's own
  device", "Category chosen by the family", "signed remotely ✓"). Remote
  mode never prints the staff copy: the browser's Print shows only
  "Printing isn't available." (after Submit: "…Your intake was sent to the
  FRC."). See "Remote mode" above for the full list. New staff links for
  remote, remote + language and remote + CRA.
- **Household wording (tablet and remote).** "First, about you": "We'll add
  everyone that lives in your home — starting with you. We do need a name,
  date of birth, and the insurance answer." The add-someone screen now says,
  in bold, "Please add everyone that lives in your home — adults and
  children." (was "We'd like to include everyone who lives with you —
  adults and children.").
- **Portuguese, after a native-speaker review.** "Co-partner" now shows as
  "Co-responsável Legal (Guarda Compartilhada / Coparentalidade)" (the
  stored/CRM value is still "Co-partner"). Typing "responsável" (or
  "responsavel"), "guarda" or "coparentalidade" in the role search finds
  it; "responsável" still finds Legal Guardian Parent and Temporary
  Guardian too. "Tribunal de Família" and "Ze/Zir/Zirs" stay as they are.
  Português is no longer marked "(beta)" in the language menu; Haitian
  Creole stays beta.
- **New page: `release-done.html`.** Shows "Thank you" (English, Spanish,
  Portuguese or Haitian Creole via `?lang=`) and tells the intake the release
  was signed. The intake only accepts that message from its own site, from
  the open panel.
- **Translations:** button labels ("Sign release for…", "Release signed ✓"),
  panel title, Close and Thank you, plus every new or changed remote-mode
  and household line, in Spanish, Portuguese and Haitian Creole.
  - Spanish: reviewed by staff, except the lines changed just before merge
    (state line, both referral hints, court hint, "How we can help", "What
    can we help you with…", "…with you…", the two household lines and the
    two printing notices), which are drafts.
  - Portuguese and Haitian Creole: drafts — review with bilingual staff.
    (The Portuguese pre-merge lines use the wording from the Portuguese
    review doc.)

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
    intake for you and for your child who was referred to us. Please
    complete both before finishing. For other members of the family, we
    only need their name, date of birth, and whether they have health
    insurance.", plus "If your
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
index.html          the entire application
release-done.html   "Thank you" page the signed release redirects to (Version 16)
README.md           this file
```
