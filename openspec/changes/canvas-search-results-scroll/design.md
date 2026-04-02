## Context

Canvas search lives in `@excalidraw/excalidraw` as `SearchMenu`, embedded in the default sidebar’s search tab (`DefaultSidebar`). Results are rendered in `MatchList` inside one or two `.layer-ui__search-result-container` blocks (frames, then text). Styles already set `overflow-y: auto` on those containers, but the parent `.layer-ui__search` uses `flex: 1 0 auto` (no shrink) and the overall column may not propagate a bounded height into children. In nested flex layouts, missing `min-height: 0` (or equivalent) often prevents scrollable regions from obtaining a max height, so content overflows the sidebar instead of scrolling. Focus changes via prev/next do not scroll the DOM node for the active row into view, so the highlight can move off-screen.

## Goals / Non-Goals

**Goals:**

- The match list area SHALL consume remaining sidebar height below the search field and counter row, with vertical scrolling when content exceeds that area.
- Changing the focused match (UI arrows, clicking a row, or any existing focus path) SHALL keep the focused row visible inside the scrollable region without confusing jumps.
- Preserve current grouping (frames vs text), i18n, and canvas zoom/pan behavior tied to the focused match.

**Non-Goals:**

- Virtualizing thousands of DOM nodes or replacing the list with “current match only” UI (issue discussion suggests this as a possible follow-up; not required for the initial fix).
- Changing search matching semantics or keyboard shortcuts beyond what is needed for scroll-into-view.

## Decisions

1. **Flex height and scroll** — Apply the standard flexbox pattern: ensure the search root and the results wrapper participate in a column flex chain with `flex: 1`, `min-height: 0` (or `overflow: hidden` on the intermediate flex parent where appropriate), so the scrollable element has a definite max height. Prefer one outer scroll container wrapping the entire `MatchList` if that simplifies layout versus two independent scroll areas (frames vs text); if two sections must stay separate, each scroll region needs the same height constraint. *Rationale:* matches browser behavior and fixes overflow without new dependencies. *Alternative:* fixed `max-height` in px — rejected as fragile across themes and sidebar widths.

2. **Scroll focused item into view** — When `focusIndex` changes, call `scrollIntoView` on the active row’s element (via `ref` on `ListItem` or a container query), with `block: "nearest"` (or `"center"` if `"nearest"` still clips in edge cases) to avoid shifting the list more than necessary. *Rationale:* minimal code, aligns with keyboard navigation expectations. *Alternative:* manual `scrollTop` math — more control but more code and edge cases.

3. **Implementation location** — Adjust `SearchMenu.scss` and `SearchMenu.tsx` first; only touch sidebar shell components if inspection shows the tab panel does not pass height down. *Rationale:* keeps the change localized.

## Risks / Trade-offs

- **[Risk] Double scrollbars** if both an outer wrapper and inner containers set `overflow-y: auto` → **Mitigation:** use a single scrollable ancestor or remove redundant overflow from inner boxes.
- **[Risk] `scrollIntoView` scrolls the whole page** in some browsers → **Mitigation:** scope scroll to the list container if needed (`scrollIntoView` options / scrollable ancestor), or use a ref on the scroll parent.
- **[Trade-off]** Rendering hundreds of rows remains DOM-heavy; acceptable for the reported bug; revisit virtualization if performance issues appear.

## Migration Plan

Not applicable — client-only UI change; no data migration. Rollback is a revert of the layout and scroll-into-view logic.

## Open Questions

- Whether the sidebar tab content already supplies `min-height: 0`; confirm during implementation and adjust the smallest parent necessary.
- Final choice between `"nearest"` vs `"center"` for `scrollIntoView` after manual QA with long lists and narrow sidebars.
