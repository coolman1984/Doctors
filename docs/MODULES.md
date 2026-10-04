# Modules — how features are added, opened and locked without touching the core

Status: **design (Phase 0)**. Becomes the developer manual when Phase 1 lands (`tools/new_module.py` + tests).

## 1. Shape of a module

```
modules/<id>/
  manifest.json        # identity, deps, entities, permissions, settings, pages, hooks, i18n, help, tests
  server.py            # optional: register(api) -> routes, event handlers, guards
  migrations/          # optional: 001_<name>.py … (add tables/fields only; never drop)
  js/<page>.js         # pages and extension-point renderers
  i18n/ar.json  i18n/en.json
  help/ar.md  help/en.md
  tests/test_<id>.py
```

`manifest.json` (all keys validated on start; an invalid manifest disables only that module and says why):

```json
{
  "id": "queue_display", "version": "1.0.0", "core": ">=1.0",
  "name": {"ar": "شاشة الدور", "en": "Queue display"},
  "requires": [],
  "entities": {"queueScreens": {"table": "mod_queue_screens", "fields": {"name": "T", "doctorIds": "J"}}},
  "permissions": [["Queue display", [["queue_display.manage", "Manage hall screens"]]]],
  "settings": {"queue_display.voice": false},
  "pages": [{"id": "queue-screens", "js": "js/screens.js", "perm": "queue_display.manage", "nav": "operations"}],
  "hooks": {"dashboard.cards": "js/card.js", "events": ["queue.called"]},
  "help": ["help/ar.md", "help/en.md"],
  "tests": ["tests/test_queue_display.py"]
}
```

## 2. The core knows nothing about modules

The core exposes **extension points** and emits **events**; a module subscribes. The core never imports a module.

| Extension point | Where it shows |
|---|---|
| `nav.<group>` | sidebar entries |
| `patient.tabs` / `patient.actions` | tabs and buttons in the patient side panel |
| `visit.sections` | extra sections in the visit (e.g. tooth chart) |
| `print.blocks` | extra blocks on printed papers |
| `dashboard.cards` | cards on the home page |
| `reports` | entries in Reports |
| `settings.cards` | cards in Settings |
| `palette` | command palette entries |

Server events (after the change is committed): `patient.created`, `appointment.booked|cancelled|checked_in|no_show`,
`queue.called`, `visit.signed`, `payment.recorded`, `day.closed`. Handlers run after commit; a failing handler is
logged and shown to the administrator, it never undoes the core change.

## 3. Open and lock

- **Licence:** `license.json` next to `config.json`: `{customer, clinicId, modules: [...], maxPCs, expires?, issued}`,
  signed with the developer's Ed25519 key (engine's `ed25519.py`). The public key is in the program; the private key
  never leaves the owner's PC (`tools/license.py sign` reads it from a path outside the repository).
- A module is **open** when: it is in the licence, its manifest is valid, every `requires` module is open, and the
  administrator has not switched it off on the Developer panel (owner-only, needs the developer key on that PC).
- **Locked** = its pages, nav entries, hooks and palette entries are not sent to browsers; its routes answer
  `403 {error: "module_locked", message_key}`; its events are not delivered. **Its tables and rows stay**, keep
  syncing between PCs, are included in backups, and come back unchanged when reopened.
- Dependency: a module whose dependency is locked is locked too, and the screen says which one and why.

## 4. Mandatory tests (`tests/test_modules.py`)

1. Core only (no licence) → every core test passes, no module route answers.
2. All modules open → every test passes.
3. For every module: create data → lock → data still in DB, routes refused, pages hidden → unlock → same data.
4. A module whose dependency is locked refuses to open.
5. A tampered licence (one byte changed) opens nothing and is reported.
6. Static: no file under `server/` or `js/` (core) imports or names a module id (PROJECT_EYE rule).

## 5. Add a module in minutes

```
python tools/new_module.py my_module --ar "اسم الوحدة" --en "Module name"
```
Copies `modules/_template`, fills the manifest, i18n, help and a passing test. Then: fill `manifest.json`, write the
page, run `python -m unittest tests.test_modules`, add the id to a customer's licence with `tools/license.py`.
