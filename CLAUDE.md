# cs-tools — Claude Code Reference

## Project Overview
Connected Stars care management system — a suite of single-page HTML applications for a residential care provider. No build process; all code is self-contained in each HTML file (embedded CSS + JS).

**Firebase project**: `connected-stars-handover`  
**Auth domain**: `connected-stars-handover.firebaseapp.com`  
**Firebase SDK version**: `10.12.2` (ESM from gstatic CDN)

## Tech Stack
- Vanilla HTML / CSS / JavaScript (no framework)
- Firebase Authentication (email/password)
- Firestore (NoSQL) — real-time reads via `getDocs`/`getDoc`, writes via `addDoc`/`updateDoc`/`setDoc`/`deleteDoc`
- Google Fonts: DM Sans + DM Mono
- All apps are fully self-contained `.html` files in the repo root

## File Structure
```
index.html         — Dashboard / login portal (20 KB)
cs-handover.html   — CS care handover tool (153 KB)
o-handover.html    — Overnight/operational handover (114 KB)
super-admin.html   — Admin management panel (58 KB)
task-board.html    — Kanban task manager (104 KB)
```

## Tool Documentation
Read these instead of the full HTML files:
- [cs-handover.md](cs-handover.md) — CS Handover tool
- [o-handover.md](o-handover.md) — O Handover tool
- [super-admin.md](super-admin.md) — Super Admin panel
- [task-board.md](task-board.md) — Task Board

## Shared Firestore Collections
These collections are read/written by multiple tools:

| Collection | Owner | Consumers |
|---|---|---|
| `staffProfiles` | super-admin | all tools (auth profile lookup) |
| `units` | super-admin | cs-handover, o-handover, task-board |
| `serviceUsers` | super-admin | cs-handover, o-handover, task-board |
| `regulatoryBodies` | super-admin | super-admin (dropdown source) |
| `adhocTasks` | cs-handover | task-board (reads/writes same collection) |

### `staffProfiles` schema
```js
{
  uid: string,       // = Firestore doc ID = Firebase Auth UID
  name: string,
  initials: string,  // 2-3 chars, used as avatar/attribution
  role: 'admin' | 'staff',
  email: string,
  status: 'active' | 'archived'
}
```

### `units` schema
```js
{ name: string, status: 'active' | 'archived', createdAt: Timestamp }
```

### `serviceUsers` schema
```js
{
  name: string,
  dob: string,           // YYYY-MM-DD
  currentUnitId: string,
  localAuthority: string,
  regulatoryBody: string,
  status: 'active' | 'moved-on',
  moveOnDate: string | null,
  unitHistory: [{ unitId, fromDate, toDate }],
  createdAt: Timestamp
}
```

## Design System (CSS Variables)
```css
--brand: #1446a0    --brand-dark: #0d3278    --brand-mid: #2d5bbf
--brand-light: #e8eef8
--bg: #f4f6fb       --surface: #fff
--text: #1a1a2e     --text2: #4a5568         --text3: #6b7280
--border: #dde3ef   --border2: #b8c5e0
--green: #27ae60    --amber: #e67e22          --red: #c0392b
--purple: #7b2d8b   --teal: #0e7490
--r: 10px           --rs: 8px
--sh / --sh-md / --sh-lg  (box-shadow tokens)
```

## Cross-Tool Conventions
- Attribution always stored as `initials` (2-3 char) from `staffProfiles`
- Dates always stored as `YYYY-MM-DD` string (never Date objects)
- Times stored as `HH:MM` 24h string
- Soft-delete via `status: 'inactive'` (not hard delete) for boards/lists/cards
- `serverTimestamp()` used for all `createdAt` / `updatedAt` fields
- Observer mode (cs-handover): read-only view, all write buttons hidden
- Admin role check: `profile.role === 'admin'` — admins see more UI and have no restrictions

## Maintenance Rule
**After modifying any HTML file, update the corresponding `.md` file to reflect the change.**  
This keeps the documentation accurate and avoids Claude needing to re-read the full HTML.

Changes that require MD updates:
- New Firestore collection or field added
- New tab, view, or section added
- New modal added
- State variable added or renamed
- Key function added, renamed, or significantly changed
- Auth/access rules changed
