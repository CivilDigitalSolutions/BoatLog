# BROE2 Import Review — prototype notes

Prototype of one screen from the **BoatLog BROE2 Competition Module, build
specification v0.1**. Mock data throughout; no network, no persistence, no build
step. Open `index.html` in a browser.

---

## Why this screen

The spec gives it one line — section 5, step 8, *"Present diff for confirmation"*
— but it is where the module either earns an organiser's trust or loses it.

Decision **D6** says every import is a diff, never a truncate-and-replace, so that
banding and race-day work done in BoatLog survives a re-import. **§4.1** makes that
safe by fencing off a `boatlog` namespace the import writer never touches. Both are
correct, and both are invisible. An organiser who has spent an evening assigning
bands, crew numbers and boating times will not paste in a fresh export unless the
screen in front of them makes that promise legible *before* they commit.

**§5.2** then names the two changes that genuinely invalidate that work —
substitutions and CRI drift — and says to flag them prominently. So they are the
thesis of the screen, not two more rows in a table: the count leads the page, and
one control filters to exactly those crews.

## What it demonstrates

| Behaviour | Spec |
|---|---|
| Diff-first framing — only differences are written, 118 unchanged crews stay untouched | D6, §5.2 |
| The six diff categories as the primary filter, with the two banding-critical ones marked | §5.2 |
| Seat-level substitution detail: who left, who arrived, and each PRI | §5.2, §6.5 |
| Live CRI against the frozen baseline, with the movement and the band it invalidates | §6.1 banding trigger, §6.2 |
| "Your work, kept" — crew number, boating time and marshalling shown as preserved | §4.1, D6 |
| Contact minimisation, defaulting to discard, with the effect visible per crew | §9 |
| Competitor names rendered as initial + surname | §9 export control |
| File manifest: 11 of 13 known files present, absent ones tolerated, not fatal | §3, §5.1 |
| Upload-capable files marked (Event Bands, Crew Bands) | §6.10, §6.12 |
| Non-blocking checks, including unknown columns logged rather than failed | §10.1, D5 |
| Masters actual category diverging from the entered sub-category | §6.7 |
| A crew referencing a band that does not exist yet in Event Bands | §6.12 precondition |
| Status written as BROE2 sends it — `Withdraw`, not `Withdrawn` | §6.1 |
| Overseas entry: four-letter code beginning `Z`, no British Rowing index | §6.1, §6.14 |
| Composite crew showing both club codes | §6.1 |

## Design

Uses BoatLog's existing design system (`assets/css/styles.css`) rather than a new
visual language: navy `#0B1F3A`, teal `#2EC4B6`, the system font stack, no
webfonts, token-driven. Extended in two ways an app screen needs and the marketing
site does not:

- **Semantic state colours**, deliberately separate from the teal accent, so
  "needs attention" never competes with "this is the action". Teal is reserved
  for the commit button.
- **App density** — a tighter type scale and tabular figures for crew IDs, CRI
  values and times.

Light and dark are both defined at token level, including the un-stamped
`prefers-color-scheme` state.

## What is mocked

- 20 changed crews and 4 of the 118 unchanged; the rest are summarised, not listed.
- Club names, competitor names, contact numbers and email addresses are invented.
  Club codes follow the real 3-letter / 4-letter-beginning-`Z` convention.
- No ZIP is parsed. Filenames, byte size, SHA-256 and format version are fixtures.
- Commit does not write anything; it swaps the bar for a confirmation state.

## Not built

Deliberately out of frame for a single-screen proof of concept: the upload step
itself, banding assignment, results capture, the two results export formats, and
the club-side crew builder.
