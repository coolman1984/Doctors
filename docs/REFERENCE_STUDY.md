# Reference Study — what the clinic system inherits from the owner's three projects

Phase 0 deliverable. Studied on 2026-10-04 from shallow clones (read-only) of:

| Project | Commit | What it is | Role for us |
|---|---|---|---|
| `coolman1984/Yousef-Transportation` ("Trip Orders / أوامر التشغيل") | `9f5ef29` (1.0.2) | Office app + internet mailbox (gateway) + driver phone page | **Design, shell, structure, gateway, method** — the first reference |
| `coolman1984/Mr.Ayman-HR` (BAMS) | `a9f676e` (2.7.0) | Offline-first multi-PC engine, permissions, backups, logs | **The engine** (newer than the copy inside Trip Orders) |
| `coolman1984/opening-nerp-tcode` (G-MES automation) | `259969d` | Browser automation for factory systems | **Engineering discipline**: "never silently mislead", PROJECT_EYE, rules guarded by tests |

Read: Trip Orders `CLAUDE.md`, `docs/{EXECUTION_PLAN,DESIGN,PLAN,REFERENCE_STUDY,GATEWAY_SETUP,GUIDE_*}.md`, `TASKS.md`,
`DEVELOPMENT_HISTORY.md`, `.claude/skills/trip-orders/SKILL.md`, `css/tokens.css`, `config.json`, the `server/`, `gateway/`
and `js/` layout; BAMS `README.md`, `CLAUDE.md`, `DISTRIBUTED_SYNC_ARCHITECTURE.md`, `GUIDE_*.md`, `IDEAS.md`,
`DEVELOPMENT_HISTORY.md`; G-MES `PROJECT_EYE.md`, `.project-eye/*`, `AGENT_PLAYBOOK.md`, `PROJECT_EXPERIENCE.md` §1–2.

**Measured, not remembered** (one note that changes the plan): the owner's brief places "the gateway pattern" and
"the work loop in EXECUTION_PLAN.md" in the "أوامر التشغيل" project. They live in **Yousef-Transportation** (its
README title is "Trip Orders — أوامر التشغيل"), not in `opening-nerp-tcode`, which has no gateway and no
EXECUTION_PLAN. We take both from Yousef-Transportation, and the PROJECT_EYE discipline from `opening-nerp-tcode`.

Second measured note: Trip Orders forked BAMS **before** BAMS 2.6/2.7. BAMS now has two engine parts Trip Orders
lacks: `server/office.py` ("Use the office system": a PC that keeps no data and opens the main PC) and
`server/upgrade.py` (verified snapshot before an update, refuse data of a newer program). **We fork the engine
from BAMS 2.7.0, and the shell/design/gateway from Trip Orders 1.0.2.**

---

## 1. Inherit as-is (rename prefixes only)

| Part | From | Why it fits a clinic unchanged |
|---|---|---|
| `ed25519.py`, `tlscert.py`, `node.py`, `journal.py`, `replica.py`, `sync.py`, `backup.py`, `nodectl.py`, `system.py`, `office.py`, `upgrade.py`, `version.py` | BAMS 2.7 | Offline-first, every PC a full copy, encrypted pinned sync, signed hash-chained history, verified backups, restore as a new change, office mode for locked networks. A clinic with bad internet and 2–4 PCs is exactly this case. |
| `auth.py` core (PBKDF2 600k, lockout, idle logout, profiles, tick-box permissions, authority-signed admin changes, personal links without admin rights) | BAMS | Reception, doctors, accountant on one LAN; permissions checked per request on the server. |
| Logs: audit (every change), activity (pages, clicks, exports), security (logins), monthly JSONL never overwritten | BAMS | Becomes the legal **"who opened which patient file"** trail (PDPL). |
| Content-addressed uploads (`uploads/cas/<sha256>`), resumable verified copy between PCs | BAMS | Lab results, X-ray photos, scanned reports. |
| Soft delete + Recycle Bin; random ids; optimistic `ver`; schema number in every change | BAMS | "No final delete" rule. |
| Installer (Inno Setup, Nuitka exe, data in `%ProgramData%`), release workflow, `start.bat` portable run, `reset_admin.bat` | BAMS / Trip Orders | One installer installs and updates; developer runs portable. |
| Test harnesses: `harness.py`, `cluster.py`, `test_convergence.py`, `test_multinode.py`, browser tests with pre-installed Chromium | BAMS / Trip Orders | Two-PC and offline proof. |
| `gateway/` pattern: Cloudflare Worker + D1, office pulls/acks with HMAC (time + nonce + body hash), tokens stored only as hashes, GET never changes state, phone outbox in IndexedDB, ✓/✓✓ delivery | Trip Orders | Patient phone interactions without ever exposing the clinic PC. |
| Design tokens structure, 5 themes, 5 bundled OFL fonts, size S/M/L/XL, density, motion off, i18n EN/AR + RTL by logical properties, shell (sidebar, topbar, stacked side panels, `Ctrl K` palette, shortcuts, `?` sheet), welcome slides, guided tours, help centre, settings with live preview, empty/loading/error states, demo data removable in one step | Trip Orders | Owner requires the design "طبق الأصل في البنية". |
| Static guards: i18n key parity, no literal words in views, no `left/right` in CSS, no colours outside tokens, WCAG AA contrast test on every theme (`test_design.py`) | Trip Orders | Keeps Arabic/dark/contrast correct forever. |
| Method files: `CLAUDE.md`, `docs/EXECUTION_PLAN.md`, `docs/PLAN.md` (decided / defaults / phases), `TASKS.md`, `DEVELOPMENT_HISTORY.md` (newest first: what, why, mistakes, lessons), `IDEAS.md`, role guides, project skill | all three | Any agent continues the same way. |
| PROJECT_EYE: `PROJECT_EYE.md` + `.project-eye/{manifest.json,graph.yaml,rules.yaml}` + a test that proves each rule; "no capability removed before it is recorded as ported / covered / dropped, with a reason" | G-MES | Guards the core/module boundary mechanically. |
| Prime directive: **the tool never silently misleads** — prove the result or say plainly what is not proven | G-MES | The owner's governing principle for patient safety. |

## 2. Adapt for clinics (and why)

| Part | Change | Why |
|---|---|---|
| Central registries (`store.ENTITIES`, `auth.PERMISSIONS`, `app.py` if-chain of routes, `shell.js` `PAGES`) | Become **registries filled by the core and by modules** (`register_entity`, `register_permissions`, `register_routes`, `TO.registerPage`). The core registers its own parts the same way. | Trip Orders hard-codes every page/route in one place; a lockable module must plug in without editing the core. |
| Entities | Core: patients, appointments, visits, visit notes (addenda), prescriptions, rx items, payments, cash sessions, services/prices, doctors/schedules, consents, file-open log. | New domain. |
| Merge rules | `visits.status` rank `open < signed`, signed never reverts; a signed visit is never edited — corrections are **addenda** (insert-only rows). Payments and receipts insert-only; refunds are new negative rows. Queue number = per-doctor per-day with the **PC letter** (Trip Orders numbering lesson). | Medical record integrity; two reception PCs offline at once. |
| Permissions | Groups: Patients, Medical record (separate from Booking & money), Visits, Prescriptions, Cash, Reports, Lists, Admin. Profiles: Doctor, Reception, Accountant, Manager, Viewer (+ Administrator). `medical.view` is separate from `booking.view`/`cash.view`; every open of a medical file is logged. | Owner's safety rules + PDPL. |
| Trust colours | Same three colours, clinic meaning: **green** = proven (identity confirmed, receipt balanced, backup verified), **amber** = needs attention (possible duplicate patient, unpaid, unsigned visit), **red** = danger (allergy match on a prescribed drug, cash difference at closing, failed backup). Never decorative. | Owner: "لون الثقة محجوز لمعناه فقط". |
| Theme values and names | New calm medical palette (petrol blue brand, violet single accent — distinct from green/amber/red and from info blue), themes: **صباح العيادة / Clinic Day** (light, layered, not flat white), **النوبتجية / Night Shift** (dark), **كتّان / Linen** (warm light), **جرافيت / Graphite** (neutral dark), **تباين عالٍ / High Contrast**. Same token names, same contrast test. | Owner: change only colour values and theme names. |
| Brand and legal text | `config.json` → `brand {name, short, logo, colors}`, `clinic {name, address, phones, doctor titles}`, `print {rx_header, rx_footer, legal}`; Settings → Organisation edits them. Default product name generic: "عيادة / Eyada". | Rule: no clinic name/logo/legal text in code. |
| Ports and folders | Web **8100**, sync **8463**, data `%ProgramData%\Eyada`, hash domains `EY-CS1`, `EY-SESSION1`, `EY-LINK1`, headers `X-EY-*`. | Must coexist with BAMS (8080/8443) and Trip Orders (8090/8453) on one PC. |
| Gateway | Same Worker skeleton; new card types: booking slots (no patient data), appointment confirmation, pre-visit form. **Minimum data in the mailbox**: first name + time + clinic, never diagnoses; retention 7 days (not 30). | Health data is sensitive under PDPL; the mailbox is outside the clinic. |
| Demo data | Fully synthetic Egyptian names, phones `010…`-shaped but from a reserved fake range, invented drugs list marked "demo"; removable in one step. | Owner: no real patient data in the repository ever. |
| Excel/Word | Keep reader/writer for exports (patient list, cash day, reports) and for **import of an old patients list**; drop the trip workbook layout. | Clinics often have an Excel or an old program's export. |

## 3. Not for us (dropped deliberately)

| Part | Reason |
|---|---|
| Break-area domain (BAMS), trips/odometer/vendor reconciliation (Trip Orders) | Other domains. |
| Trip Orders `com_office.py` (DRM files via Office COM) | Solved a company-DRM problem clinics do not have; recoverable from Trip Orders if ever needed. |
| `studio/` film pipeline | Marketing tool; later maybe for a product film, not part of the product. |
| G-MES CDP/browser automation code | Different problem; we keep only its discipline. |
| "English only" rule (BAMS) | Owner wants Arabic and English. |
| Open join without approval (BAMS 2.4 default) | Kept available, but **default = approval code** for clinics: a patient file must not reach an unknown PC on a shared building network. |

## 4. What none of the three has (new work)

1. **Module system** with manifests, dependency checks, signed per-customer licence, lock = hide + refuse, never delete.
2. Medical safety layer: identity bar on every patient screen and print, allergy/chronic strip, signed visits + addenda,
   prescription safety checks, file-open log.
3. Money of a clinic: services and prices, revisit window (free or reduced within N days), discounts with reason,
   payment methods (cash, InstaPay, wallets, card) as labels, cash session per user + day close with difference.
4. Queue (الدور) per doctor per day with walk-in, urgent, no-show, and a later hall display (module).
5. Printing: prescription and report templates on A5/A4 with the doctor's header from config, QR of the visit number.

## 5. Capability rule (from PROJECT_EYE)

No inherited capability is removed from the fork until a line in `docs/CAPABILITY_MAP.md` says **ported**,
**covered by** (what), or **dropped deliberately** (why). The first entries are §3 above.
