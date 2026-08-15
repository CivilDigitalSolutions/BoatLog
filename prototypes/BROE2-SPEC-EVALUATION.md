# BROE2 Competition Module — evaluation against BoatLog

Assessment of *BoatLog — BROE2 Competition Module, build specification v0.1*
(Civil Digital, August 2026) as a fit for the existing BoatLog product.

**Summary.** The spec is strong and the philosophical fit is unusually clean —
it is the same product posture BoatLog already sells on incident reporting. The
open questions are not about whether it belongs, but about *who it is for* and
*when each part lands*. Two things are worth changing before Phase 1 starts.

---

## 1. Where it fits well

**The posture is already BoatLog's posture.** §1 puts automated submission out of
scope: every upload is performed by a human in the BROE2 UI. That is precisely the
line the product already takes in public on incident reporting —

> "BoatLog is not British Rowing, and it does not submit reports to them. […]
> BoatLog's job is to help your club capture the details accurately at the time,
> so filing that report is quick and complete rather than a reconstruction from
> memory a week later." — `incident-reporting/index.html`

So there is no new positioning to invent and no new objection to handle: the site
FAQ already answers "is BoatLog an official British Rowing product?" with a plain
no. Same relationship to British Rowing's systems, same value proposition —
capture it properly, make the official submission fast.

**It extends existing primitives rather than running alongside them.** The spec
reaches for things BoatLog already has: `membershipNumber` as the join key to
`people` (§6.5), `boatName` as a candidate match against `assets` (§6.1), the
existing six-week retention and anonymisation policy (§9), and the incident-report
bookmarklet's clipboard hand-off pattern for the entry worksheet (§11). This is a
module, not a second product.

**The engineering judgement is sound.** Header-sniffing over fixed column order
(D5), diff-over-truncate (D6), the fenced `boatlog` namespace (§4.1), separate
mappers and non-interchangeable enums for the two results formats (D7, §7.3), and
the Raw Time / Time correctness rule (§7) are all the right calls. The traps it
documents — Crew Baselines carrying *Competitors'* columns despite its filename
(§6.2), the status value being `Withdraw` not `Withdrawn` (§6.1), `Substitute`
being `Yes`/`No` while everything else is `Y`/`N` (§6.5), an en-dash inside a
status enum (§6.13) — are each worth a lost day to whoever writes the mapper.

## 2. What needs deciding

### 2.1 It serves a different buyer from everything shipped so far

Every published feature — boat sign-out, RAG flags, damage reporting, incident
logging — is **boathouse operations for the whole membership**. BROE2 is
**competition administration for one or two officers**, and Phases 1–3 serve
*organising* clubs specifically. Most clubs enter competitions; comparatively few
host them.

That is a different buyer, a different sales conversation and a different
navigation slot on a site that currently has four product pages of one kind.
Worth an explicit decision on whether this is a fifth headline product or a
priced add-on module.

### 2.2 Phasing puts the broadest-appeal, cheapest deliverable last but one

§12 places the club-side crew builder and entry worksheet at **Phase 4**. But §11
is the only part of the spec that serves BoatLog's *existing* customer base, and
it is comfortably the cheapest thing in the document: a form that mirrors the
BROE2 entry order, a per-crew clipboard payload reusing the bookmarklet pattern
that already exists, and a CSV. No ZIP parsing, no import diff, no new Firestore
model, no third-party personal data, and therefore none of §9.

**Recommendation:** pull §11 forward ahead of, or alongside, Phase 1. It gets the
module in front of every customer instead of a handful, it is shippable while the
Phase 1 blockers below are still outstanding, and its advisory-only framing
(§10.3) is honest and easy to defend.

### 2.3 Offline is BoatLog's signature claim, and race day is the hardest case

The site leads on working without WiFi at the boathouse. A head course on race day
is the most offline-hostile day in the rowing calendar — and it now has a
five-day results deadline attached (§7.4).

D1 parses the ZIP client-side, which helps. But the spec is silent on offline
behaviour for the things that actually happen on the bank: results capture (§7),
and the crew-number, boating-time and marshalling work held in the `boatlog`
namespace. These need the same offline-first treatment the kiosk already has.

This is both the biggest technical risk in the module and its strongest
differentiator — offline race-day capture is a claim BoatLog can credibly make
and most competitors cannot.

### 2.4 Data protection is a step change, not an extension

§9 is candid, but understates one point. Today BoatLog holds data about **its own
club's members**. This module means holding names, calculated ages, membership
numbers, and up to eight telephone numbers and four email addresses per crew for
**thousands of athletes at other clubs** who have no relationship with BoatLog at
all.

The published privacy policy is written for the member-and-club relationship and
would need a new section. The DPA change §9 asks for is real work. The controls
proposed are the right ones — in particular the per-competition toggle defaulting
to *discard* contact fields — and that default should be treated as
non-negotiable rather than a preference.

### 2.5 One gap in D3

Using BROE2 `CrewID` as the Firestore document ID (D3) is a good call and buys
free idempotency. But it couples document identity to a third party's ID scheme,
and the club-side crew builder (§11, Phase 4) creates crews that do **not** yet
have a `CrewID` — they get one only after a human keys the entry into BROE2. That
path needs its own ID strategy and a reconciliation step on the next import. The
spec does not currently address it.

## 3. Blocking before Phase 1

The spec identifies these itself and is right to; they should be treated as a hard
gate, not a parallel workstream:

1. **Format currency (§13.2).** V8 is dated June 2020. It is now 2026. No mapper
   should be written speculatively against a six-year-old document.
2. **Fixtures (§13.5, §14).** Every mapper is fixture-driven and the results
   exporters need golden-file tests over exact column order, header spelling and
   enum strings. Obtain real exports first.
3. **Encoding and delimiters (§13.4).** Determine empirically. UK club names carry
   apostrophes and accents.

## 4. Proof of concept

`prototypes/broe2-import-review/` implements §5 step 8, the import diff
confirmation screen — the point where D6's promise either becomes visible to an
organiser or stays a private implementation detail. See that directory's
`NOTES.md` for what it demonstrates and what is mocked.
