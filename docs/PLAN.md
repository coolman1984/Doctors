# Plan — Eyada (working name), the Egyptian clinic system

Source of truth for scope: the owner's master prompt (2026-10-04). This file records how we execute it.
Studies: `docs/REFERENCE_STUDY.md` (what we inherit), `docs/MARKET_STUDY.md` (Arabic, the market).
Module architecture: `docs/MODULES.md`. Progress: `TASKS.md`. Playbook: `docs/EXECUTION_PLAN.md` (written after approval).

**Governing principle:** the system never silently misleads. A wrong patient file, a wrong drug on a printed
prescription or a lost visit is worse than a visible error. Every result is proven or plainly marked unproven.

## 1. Decided (do not re-open without the owner)

- Scope v1: solo clinic, multi-doctor clinic, small medical centre; all common specialties. Hospitals out.
- Engine = BAMS 2.7.0 fork (stdlib Python, SQLite WAL, offline-first, every PC a full copy, encrypted sync,
  office mode). Shell, design, gateway and method = Trip Orders 1.0.2. Discipline = G-MES PROJECT_EYE.
- Vanilla JS, no framework; one Windows installer installs and updates; portable run for the developer.
- The clinic PC is never reachable from the internet; patient phones talk only to the gateway mailbox.
- No final delete, edits are new entries, every action logged with user + PC, permissions checked on the server for
  every request, no secrets in clear, no clinic name/logo/legal text in code.
- **Core never locks; everything else is a module** with a manifest, lockable per customer by a signed licence.
  Locking hides and refuses, never deletes data.
- Signed visits are never edited; corrections are dated addenda with the author.
- `medical.view` is separate from booking and money; every open of a medical file is logged.
- Demo data fully synthetic; no real patient data in the repository, ever.

## 2. Assumptions (defaults used until the owner says otherwise — never block on them)

| # | Topic | Default |
|---|---|---|
| A1 | Product name | generic "عيادة / Eyada" in `config.json`; owner can rename anytime |
| A2 | Default language | Arabic (RTL) for new users; English per user |
| A3 | Ports / data folder | web 8100, sync 8463, `%ProgramData%\Eyada` |
| A4 | Patient identity | required: full name (≥ 3 parts suggested, not forced), mobile (Egyptian format normalised to `+20…`), age or birth date; sex. Optional: national id (14 digits, checked, never shown in lists), photo |
| A5 | Duplicate warning | same normalised mobile, or name similarity ≥ 0.85 (Arabic normalisation) with age ± 2 years → amber warning, never auto-merge |
| A6 | Revisit (إعادة) | free within **14 days** of a paid consultation by the same doctor; per-doctor setting (days, price 0 or reduced) |
| A7 | Queue number | per doctor per day, prefix by PC letter only when two PCs issue numbers offline (`A-12`); otherwise plain `12` |
| A8 | No-show | appointment not checked in by the end of its session → marked no-show automatically, counted per patient |
| A9 | Payments | methods = labels (cash, InstaPay, wallet, card, other); only **cash** counts in the drawer at day close |
| A10 | Day close | per user cash session; close shows expected vs counted; any difference is red and needs a reason |
| A11 | Prescription paper | A5 portrait default, A4 option; header/footer from config; drug lines free text + favourites (drug bank is a module) |
| A12 | Mailbox retention | 7 days; minimum data (first name, time, clinic name) |
| A13 | Licence | one signed file per customer: customer, clinic id, modules, PC limit, expiry (optional); developer key lives only on the owner's PC |
| A14 | Price model (advice, owner decides) | one-time licence + paid modules + yearly updates; never per patient |
| A15 | Doctor's share (module) | percentage per service per doctor, or fixed amount; settlement monthly |
| A16 | Consent (PDPL) | consent captured at registration (tick + who/when), printable consent form from config text |

## 3. Core (never locked — built and delivered first)

| ID | Core part | Done when |
|---|---|---|
| C1 | Engine fork, rename, module loader + licence check, empty-core tests green | engine tests + module system tests green |
| C2 | Shell: themes, fonts, i18n AR/EN, sidebar, topbar, side panels, palette, shortcuts, settings with preview, help, tours, slides | smoke test in ar+Clinic Day and en+Night Shift |
| C3 | Patients: identity, duplicate warning, identity bar, allergy/chronic strip, consent, file-open log | domain tests + browser flow |
| C4 | Doctors, schedules, services and prices | |
| C5 | Appointments and booking + today's queue (walk-in, urgent, no-show, check-in, call next) | queue tests incl. two PCs offline |
| C6 | Visit (كشف): basic medical record (complaint, exam, diagnosis, plan), sign → addenda only | sign/addendum tests |
| C7 | Prescription and printed reports (A5/A4, doctor header, identity on paper, allergy check) | print tests |
| C8 | Cash: receipts, revisit rule, discounts with reason, refunds as new rows, cash session, day close | money tests |
| C9 | Users and profiles (Doctor, Reception, Accountant, Manager, Viewer, Administrator) | permission matrix test |
| C10 | Backups, logs, Recycle Bin, devices & sync, office mode (engine), basic reports (day, month, per doctor) | |
| C11 | Demo data + one-step removal; guides (doctor, reception, manager, developer) | |

## 4. Modules (order chosen by the market study: pain × reach ÷ cost)

| # | Module id | Arabic name | Depends on | Why this place |
|---|---|---|---|---|
| M1 | `queue_display` | شاشة الدور في الصالة | core | Small, visible "wow", answers pain 1 for busy doctors |
| M2 | `rx_bank` | قوالب الروشتات وبنك الأدوية | core | Doctor speed (pain 5) |
| M3 | `patient_link` | الحجز برابط والتذكير والتأكيد واستمارة قبل الزيارة (البوابة) | core | No-shows (pain 1), competitors' strongest card |
| M4 | `center` | المركز متعدد الأطباء: النسب والتسوية | core | Centres (pain 7) |
| M5 | `insurance` | التأمين والتعاقدات | core | Pain 8; centres |
| M6 | `labs` | طلبات ونتائج المعامل والأشعة + الصور | core | Medical completeness |
| M7 | `dental` | الأسنان: خريطة الأسنان وخطة العلاج | core | Largest private specialty |
| M8 | `obgyn` | النسا والتوليد: متابعة الحمل | core | |
| M9 | `peds` | الأطفال: منحنيات النمو والتطعيمات | core | |
| M10 | `physio` | العلاج الطبيعي: الجلسات والباقات | core | |
| M11 | `eye` | العيون: القياس والنظارة | core | |
| M12 | `derm` | الجلدية: صور قبل/بعد وجلسات | core | |
| M13 | `eta_receipt` | الإيصال الإلكتروني | core (+ `center` optional) | Legally needed now mainly by centres in Cairo/Alex |
| M14 | `inventory` | المخزون والمستلزمات | core | |
| M15 | `mgmt_reports` | تقارير الإدارة والعرض من البيانات الحية | core | |
| M16 | `patient_comms` | التواصل مع المرضى | `patient_link` | |
| M17 | `privacy_kit` | ملف الامتثال لحماية البيانات (سجلات للمركز، طلبات المرضى) | core | Deadline 1 Nov 2026 — core already logs; this packages it |

## 5. Phases

| Phase | Deliverable | Gate |
|---|---|---|
| 0 Study & plan | REFERENCE_STUDY, MARKET_STUDY, PLAN, MODULES draft, CLAUDE.md, TASKS, history | **owner approves once** |
| 1 Engine + module system | C1 | core-only / all-modules / lock-unlock / dependency tests green |
| 2 Shell | C2 | static guards + contrast test + browser smoke |
| 3 Core domain | C3–C9 | domain + browser tests |
| 4 Core delivery | C10–C11, installer, guides | full suite, release build |
| 5+ Modules | M1 → M17, one PR-sized slice each | module tests + core-only test still green |

## 6. Risks

| Risk | Mitigation |
|---|---|
| A module changes data the core relies on | modules only add tables/fields of their own; they talk to the core through extension points and events; PROJECT_EYE rule + test |
| Locking a module on one PC but not another diverges data | the licence is per installation but **schemas are always registered** and data always syncs; locking is presentation + API refusal only |
| ETA e-receipt cannot be tested without credentials | build against the documented schema with a local fake; mark "not verified against ETA" until the owner's pre-production test |
| PDPL details (licence fees per clinic size) still being clarified | the system provides the records; legal advice stays the clinic's |
| Old reception PC | budget: first paint < 1.5 s on a 4 GB / Core i3 PC; no framework; tables virtualised over 500 rows |
