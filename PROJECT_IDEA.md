# Counter Room Sample Register — Project Idea

A simple internal web app for a garment/textile "counter room" (sample
room) to track physical fabric/garment samples: what's logged in, who has
taken a sample out, and when it's due back.

---

## 1. Who uses it

- **Admin** — full access: logs samples, edits/deletes, sees everything,
  manages the employee list.
- **Employee** — day-to-day use: log new samples, issue them out, mark them
  returned.

Two people, simple login, no public access.

---

## 2. Core idea

Every physical sample gets one digital record the moment it enters the
counter room. That record tracks its life:

```
Logged in → In Room → Issued to someone → Returned → In Room again
                              ↓
                       (if not returned in time) → Overdue
```

The app is really a live status board for "where is every sample right
now, and who's holding what."

---

## 3. Main screens / tabs

- **Dashboard** — top-line numbers at a glance: total samples, how many are
  in the room, how many are out, how many are overdue. A quick-scan home
  screen.
- **Register** — the full searchable/filterable list of every sample ever
  logged, with its current status, photo, and details. This is the "look
  anything up" screen.
- **Log Sample** — a form to add a brand-new sample: seal number, seal
  colour, sample type, buyer, PO number, merchant, item number, location,
  notes, and photos.
- **Action (Issue / Return)** — the working screen for handing a sample out
  or checking it back in:
  - *Issue*: pick a sample, who's taking it, which department, a return
    deadline, an optional photo of the person taking it, and a note.
  - *Return*: one tap to mark a sample back in the room.
- **Supervisor** — a focused view of everything currently checked out,
  sorted by how soon (or how overdue) it's due back — the "what needs
  chasing" screen.
- **Settings** — the current user's display name, and the employee list
  (admin only).

---

## 4. What a sample record needs to hold

- Seal number (the physical tag's number — used like an ID)
- Seal type/colour (e.g. Green / Red / White — visual at-a-glance marker)
- Sample type (e.g. PP Sample, Initial Sample, TOP Sample, GOLD Seal)
- Name/description of the sample
- Buyer, PO number, merchant, item number — the business context
- Rack/shelf location inside the counter room
- Photos of the sample itself
- Status: in-room or issued
- If issued: who took it, which department, when, expected return time,
  a photo of the person, and a note
- A history log of everything that's happened to it (logged, issued,
  returned) — so nothing is a mystery later

---

## 5. Key behaviors worth designing in from the start

- **Live updates** — if two people have the app open, one issuing a sample
  should show up for the other almost instantly, no manual refresh.
- **Overdue tracking** — anything issued past its deadline should visually
  stand out (colour, badge, sort-to-top) without anyone having to check.
- **Search & filter** — by buyer, merchant, sample type, status, or free
  text — the register needs to stay usable as it grows to hundreds of
  samples.
- **Photos as proof** — a photo of the sample when logged, and a photo of
  whoever takes it when issued, so there's a visual record either way.
- **Printable label** — something scannable/printable per sample (e.g. a QR
  code with the seal number) to physically tag it.
- **Simple login** — two named accounts, nothing more elaborate needed.
- **Mobile-first layout** — this gets used standing at a counter on a
  tablet/phone, not sitting at a desk — buttons and text need to be usable
  with a thumb.

---

## 6. Suggested file/section layout for a from-scratch build

However you choose to structure the code, these are the natural pieces:

- **Login / auth screen**
- **App shell** — header, live-status indicator, tab navigation
- **Dashboard view**
- **Register view** — list + search/filter + per-sample detail (photos,
  history, edit)
- **Log Sample view** — the intake form
- **Action view** — Issue sub-flow and Return sub-flow
- **Supervisor view** — overdue-focused list
- **Settings view** — profile name + employee management
- **Shared bits** — photo capture/upload, QR/label printing, toast/error
  notifications, live data sync

---

## 7. Nice-to-haves for a v2 (not essential to launch)

- Notifications/reminders as a deadline approaches
- Export the register to a spreadsheet
- Multiple counter rooms / locations
- Role-based permissions beyond the current two fixed accounts
