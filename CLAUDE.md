@AGENTS.md

## Corrections to AGENTS.md

- AGENTS.md's tech-stack versions are stale. `package.json` (`packageManager`, `engines`, dependencies) is authoritative; CI runs Node 24 while `.nvmrc` pins 20.
- `scripts/generateDevicesIndex.js` writes an obsolete `src/devices/index.ts` format (default imports). Do not run it; edit `src/devices/index.ts` by hand.

## Gotchas

- A new vendor file in `src/devices/` must also be imported in `src/devices/index.ts` as `import {definitions as <name>} from "./<file>";`. `test/index.test.ts` fails otherwise.
- `src/index.ts` imports `./models-index.json`, which is generated (and gitignored). `pnpm run build` writes it to `dist/` via `dist/indexer.js`; the vitest global setup (`test/vitest-setup.ts`) rebuilds it from `src/` before every test run.
- The indexer throws on duplicate `model`, duplicate `zigbeeModel` (case-insensitive) or identical `fingerprint` across all vendor files, so a definition can break the build even when it type-checks.
- `test/checks.test.ts` validates definitions:
  - expose `access` SET/GET bits must match whether a toZigbee converter for that key has `convertSet`/`convertGet`;
  - numeric exposes with SET access need `value_min` and `value_max`;
  - lights exposing `color_temp` need an explicit range unless the model is listed in `test/colortemp_range_missing_allowed.ts`;
  - expose and option names must be snake_case;
  - vendor name casing must match existing definitions of that vendor.

## Commits and releases

- release-please builds `CHANGELOG.md` and the version from Conventional Commit subjects, so the subject is user-facing. Scopes in use: `feat(add): <models>` for new devices, `fix(detect): Detect <id> as <vendor> <model>` for detection-only fixes, `fix(ignore):` for dependency bumps (dependabot prefix), plain `feat:`/`fix:` with `<Vendor> <model>: <change>` otherwise.
- Pushing a tag publishes to npm and then dispatches `update_dep` to Koenkk/zigbee2mqtt (`.github/workflows/ci.yml`).

## Downstream coupling

- zigbee2mqtt consumes this package and user external converters import from it. `package.json` `exports` exposes `./converters/*`, `./devices/*` and `./lib/*`, so renaming or removing an exported symbol in those modules is a breaking change for them.
- Changing an existing expose's `property`/name or a `model` string changes the MQTT payload and Home Assistant entities for users of that device.
