# Counter Room Sample Register — Build Notes

Live artifact: https://claude.ai/code/artifact/915ef6f5-31c7-4a3a-b4d2-d7b311ebe449
Demo logins: `priyanshu` / `1234` (admin), `ankush` / `1234` (employee)

## What it is

A single-file mobile-first web app for a garment sample room to track physical
samples through logged-in → in-room → issued → returned/overdue. Built from
`PROJECT_IDEA.md` plus functional requirements pulled from the user's Firebase
reference build. Branded with Rugs Creation's real identity and seeded with
their real sample data (see below).

## Navigation (current architecture)

5 views: **Dashboard** (summary tiles + composition bar + recent activity,
no work forms), **Master Data** (searchable register, tap-to-expand cards
with photo/history/actions), **Log Sample** (intake form), **Issue**
(in-room cards → inline hand-off form), **Return** (issued cards,
soonest-due-first, one-tap). Top tabs ≥760px, bottom nav bar below that.

## Branding

Colors and logo pulled directly from the user's `index.html` reference app —
their real "Rugs Creation" identity, not invented. Logo is the exact base64
PNG wordmark from their file, on a fixed dark chip (header + login) so it
reads on both themes. Accent violet `#7C5CFC`/`#8F73FF`, seal-tag colors
Green `#3F6B52` / Red `#9B3B3B` / White (their exact `SEAL_TYPES` values).
Status colors re-derived from their palette and re-validated for CVD-safety
and contrast (their originals didn't pass at full brightness). The app's
existing light/dark tokens map onto their paper-card/ink-shell look 1:1, so
no architecture change was needed to carry the brand.

## Real data import (done)

The user's `index.html` has no embedded records (`SAMPLES = []`, populated
only from a live Firebase Realtime Database) — nothing static to copy. They
had me export it via the browser (Claude in Chrome, already signed into
their Firebase console) — Firebase console → Realtime Database → ⋮ → Export
JSON — then pulled into this session via the device-file bridge (Downloads
folder access, granted by the user) since this sandbox can't reach their
live Firebase directly.

Result: 33 real samples, 1 employee record. The export was **163 MB** —
almost entirely full-resolution photos (2 photos/sample average, ~2.3 MB
each as base64). Compressed each sample's first photo (PIL, resize to 480px
max dimension, JPEG q72, EXIF-orientation corrected) before import, dropping
total photo payload to **~1.2 MB** — necessary, since localStorage has only
a ~5–10 MB per-origin quota and the raw export would have blown through it.

Field mapping from their schema to this app's: `sealNumber`→`seal`,
`sealType`→`color` (lowercased), `sampleType`→`type` (matched exactly),
`itemNumber`→`item`, `poNumber`→`po`, `description`→`notes`, plus a new
`sampleDate` field (their business date, not in the original schema)
surfaced as a detail row in Master Data. All 33 records are currently
`status:"in-room"` (nothing issued right now) — Issue/Return show empty
until real hand-offs happen; that's correct, not a bug.

Deliberately **not** imported: the `employees` node (held Ankush's personal
phone number) — irrelevant to sample records, not something to bake into a
page that could end up shared.

The old fictional demo records were fully replaced, not merged. Storage keys
bumped `_v2`→`_v3` so a browser with old demo data in localStorage picks up
the real seed fresh.

## Storage architecture — status: decided, not yet built

Current build still uses `localStorage` (demo-grade, per-browser only) — the
real import proved this doesn't scale to production photo volume (163 MB
raw for just 33 samples).

Path explored and ruled out:
- **Firebase Storage** — natural fit (reference app already uses Firebase
  Auth + Realtime DB) but requires upgrading to the Blaze (pay-as-you-go)
  plan, which means adding a billing method even though usage would very
  likely stay in the free tier. User explicitly doesn't want to attach a
  card for this, so this path is **out**.
- **Google Drive** — technically possible via the Drive API, but is
  consent/OAuth-based: either every tablet user has to individually sign in
  and approve access, or everything routes through one shared service
  account. Poor fit for a shared kiosk-style counter tablet. User was told
  this plainly and did not pick it.
- **Reusing the existing Supabase project (`rugscreation-manifest`)** —
  confirmed free ($0/month) and already connected to the user's org, but
  the user wants Counter Room kept fully separate from whatever else lives
  in that project.

**Current plan:** the user is setting up a **new, dedicated Supabase
project** themselves and will hand over the **Project URL + anon/public
key** once it exists, along with either their own SQL schema or asking for
one to be drafted. Meanwhile they're taking the current HTML file to
publish it themselves on GitHub (Pages) as an interim step — that copy is a
**snapshot**, not connected to anything, and will drift out of sync with
further changes made here unless explicitly re-synced.

**Next step once Supabase details arrive:** wire the dashboard to real
Supabase Storage (photo uploads, replacing inline base64) and a `samples`
table (replacing `localStorage` reads/writes) using the anon key client-side
— never the `service_role` key, which must stay server-side only.

## Bug fixed: "stuck Logging…" modal

Earlier version used fixed-position modals for Log/Edit/Issue; one was
missing its open/close CSS pairing, so the save worked but the modal never
visually closed. Fixed by replacing all modal overlays with inline in-card
expand sections, which have no separate display-toggle CSS to fall out of
sync — removes the bug class structurally.

## iOS 12 / Android tablet compatibility

No flex `gap` (unsupported pre-Safari 14.1) — `.hstack` margin-sibling
pattern instead; grid `gap` kept (fine since Safari 10.1). 16px minimum
input font-size (prevents iOS auto-zoom). No `capture` attribute on file
inputs (unreliable on iPad per the reference app's own comments). ~38–48px
touch targets. Based on documented WebKit/iOS 12 limits, not device-tested —
worth checking photo capture on a real iOS 12 iPad.

## Validated

`node --check` on extracted script passes; every `getElementById` has a
matching id (zero missing); no duplicate ids; JS-toggled display states all
have matching CSS. Screenshot-tested phone (390px) and desktop (1100px)
widths, light and dark themes, both roles, and with the real imported data —
login, branding, navigation, card expand, search, Log submit, Issue,
Return all confirmed working.

## Package contents

- `counter-room-dashboard.html` — the app itself (self-contained, drop
  straight into GitHub Pages or any static host)
- `PROJECT_IDEA.md` — the user's original spec this was built from
- `reference-app-rugs-creation.html` — the user's Firebase-backed reference
  build (source of the data model, branding, and domain constraints —
  visual design was not copied, only pulled in explicitly later on request)
- `design-notes.md` — this document
