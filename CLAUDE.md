# Eyada (clinic system) – rules for every change

Owner: Mohamed. Users are doctors, receptionists, assistants, accountants and centre owners who are **not technical**.
Sibling of BAMS (`coolman1984/Mr.Ayman-HR`, the engine) and Trip Orders (`coolman1984/Yousef-Transportation`, the design,
shell and gateway). Start with `docs/PLAN.md`, then `TASKS.md` (where to continue), `docs/REFERENCE_STUDY.md`,
`docs/MARKET_STUDY.md`, `docs/MODULES.md`.

**Prime directive: the system never silently misleads.** A patient file opened under another name, a wrong drug on a
printed prescription or a lost visit is worse than a visible error. Prove every result; say plainly what is not proven.

## Always

1. Update in the same change: `DEVELOPMENT_HISTORY.md` (newest first: what, why, mistakes, lessons), `IDEAS.md`,
   guides, `TASKS.md` test map, `docs/RELEASE_NOTES.md` for user-visible changes, `PROJECT_EYE.md` for structure changes.
2. Measure facts with a script; never copy numbers from memory. Test before every push; a regression test for every bug.
3. UI in **Arabic and English** (i18n keys only, RTL by logical CSS properties); colours only from tokens;
   green/amber/red mean trust only. Check: Arabic? dark theme? offline? two PCs? empty database? core only?
4. The core never imports a module; modules use extension points and events (`docs/MODULES.md`).
5. Replies to the owner: simple Egyptian Arabic — الخلاصة ← اللي اتعمل ← محتاج منك (الافتراض بين قوسين) ← الخطوة الجاية.
   No pasted code, no long file lists.

## Never

- Never delete for good: soft delete + Recycle Bin; an edit is a new entry; a signed visit is never edited (addenda only).
- Never skip the server permission check; never show medical data to a user without `medical.view`; never skip the
  file-open log.
- Never store or log passwords, tokens or keys in clear. Never make the clinic PC reachable from the internet.
- Never put a clinic name, logo or legal text in code – only in `config.json`.
- Never commit real patient data, real names, photos or the developer's licence key. Demo data is synthetic.
- Never remove a capability before `docs/CAPABILITY_MAP.md` says ported / covered / dropped deliberately, and why.
- Never push to `main` or force-push. Never push with red tests.
