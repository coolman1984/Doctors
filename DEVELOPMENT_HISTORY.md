# Development History and Lessons Learned

Newest first. Every change adds an entry: what changed, why, mistakes, lessons.

---

## Phase 0 – study and plan (2026-10-04)

**What:** read the three reference projects (Trip Orders `9f5ef29`, BAMS `a9f676e`, G-MES `259969d`), researched the
Egyptian clinic market, law (PDPL 151/2020 + regulations of Nov 2025, e-receipt phase 7, universal health insurance) and
competitors; wrote `docs/REFERENCE_STUDY.md`, `docs/MARKET_STUDY.md`, `docs/PLAN.md`, `docs/MODULES.md`, `CLAUDE.md`.

**Findings that changed the plan**
- The gateway and EXECUTION_PLAN the brief attributes to the "أوامر التشغيل" project are in Yousef-Transportation
  (Trip Orders), not in `opening-nerp-tcode`. Measured, not assumed.
- Trip Orders forked BAMS before 2.6/2.7, so it lacks office mode and the safe-upgrade check. The engine comes from BAMS 2.7.
- The PDPL compliance period ends **1 Nov 2026**; health data needs explicit consent and a licence. Consent capture and
  the file-open log go into the core, not a module.

**Mistakes / limits:** competitor websites were blocked by the sandbox network; prices come from search-result extracts.
Workflow pains marked "ميداني" are field knowledge to confirm with two real clinics.
