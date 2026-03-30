# oh-my-codex v0.11.10

**Patch release for approved handoff alias parsing hardening and clean release metadata sync**

`0.11.10` follows `0.11.9` with a deliberately narrow release: it locks approved execution handoff parsing against quoting regressions for both Ralph and Team alias forms, then cuts a clean metadata-aligned patch release.

## Highlights

- Approved `$ralph` launch hints are now protected by single-quoted regression coverage.
- Approved `$team` launch hints are now protected by single-quoted regression coverage.
- Node and Cargo release metadata are synchronized to `0.11.10` for a clean release cut.

## What’s Changed

### Fixes
- add regression coverage for single-quoted approved `$ralph` launch hints in planning artifact parsing
- add regression coverage for single-quoted approved `$team` launch hints in planning artifact parsing

### Changed
- bump release metadata from `0.11.9` to `0.11.10` across the Node and Cargo packages
- refresh `CHANGELOG.md`, `docs/release-notes-0.11.10.md`, and `RELEASE_BODY.md` for the release cut

## Verification

- `npx biome lint src/planning/__tests__/artifacts.test.ts`
- `npm run build && node --test dist/planning/__tests__/artifacts.test.js`
- `npm run test:sparkshell`
- `npm run test:team:cross-rebase-smoke`
- `npm run smoke:packed-install`
- `npm test`

## Remaining risk

- This release is intentionally narrow and centered on regression coverage plus metadata synchronization.
- Future approved handoff grammar changes should keep alias-form coverage aligned across both Ralph and Team paths.

## Contributors

- [@Yeachan-Heo](https://github.com/Yeachan-Heo) (Bellman)

**Full Changelog**: [`v0.11.9...v0.11.10`](https://github.com/Yeachan-Heo/oh-my-codex/compare/v0.11.9...v0.11.10)
