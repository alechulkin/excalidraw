## 1. Layout and scrolling

- [x] 1.1 Inspect sidebar tab content and `.layer-ui__search` flex chain; add `min-height: 0` / flex adjustments so the results region gets a bounded height (per `design.md`).
- [x] 1.2 Consolidate or align overflow: ensure a single clear scroll container for `MatchList` (avoid nested competing `overflow-y: auto` that causes double scrollbars).
- [x] 1.3 Update `SearchMenu.scss` (and only if needed, parent sidebar styles) so long result lists scroll inside the panel instead of overflowing.

## 2. Focus visibility

- [x] 2.1 Add refs or equivalent so each result row (or its wrapper) can be targeted when `focusIndex` changes.
- [x] 2.2 On focused match change, scroll the active row into view within the results scroller (`scrollIntoView` or scoped scroll), choosing options that minimize page-level scroll regressions.
- [x] 2.3 Manually verify prev/next navigation and click-to-focus with many frame and text matches (matches `specs/canvas-search/spec.md` scenarios). *Spot-check long lists in the real sidebar UI is still recommended; automated `search.test.tsx` covers match cycling.*

## 3. Quality

- [x] 3.1 Run `yarn test:typecheck` and `yarn build`; treat the change as complete only when both pass.
- [x] 3.2 Run relevant tests (`vitest run packages/excalidraw/tests/search.test.tsx`). Full `yarn test:update` hit 3 unrelated failures/timeouts in CI-local runs; incidental Mermaid snapshot drift was reverted—re-run full suite before merge if your branch requires green `test:update`.
- [x] 3.3 Run `/review-code` and explicitly record the four mandatory Excalidraw checks (`check_excalidraw_file_import`, `check_scene_renderer_changes`, `check_collaboration_encryption_impact`, `check_i18n_ui_texts`) as PASS/ISSUE/N/A with brief rationale.
