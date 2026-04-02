## Why

Canvas search in the sidebar shows a flat list of matches (frames and text). When there are many results, the list overflows the panel: users cannot reach items below the fold, and using the prev/next controls changes focus without keeping the highlighted row in view, so the list appears to “jump” and feels broken ([GitHub issue #11092](https://github.com/excalidraw/excalidraw/issues/11092)). Fixing scroll containment and focus visibility restores predictable navigation and aligns with expectations for long result sets.

## What Changes

- Constrain the search panel layout so the match list sits in a region with a bounded height and vertical scrolling.
- When the focused match changes (arrow buttons, keyboard, or programmatic focus), scroll the list so the active row stays visible (e.g. `scrollIntoView` with appropriate options).
- Preserve existing search behavior: query input, match counts, frame/text grouping, and canvas highlighting for the focused match.
- Optionally document or defer “virtualization / show only current match” for very large match counts as a follow-up if the first fix is insufficient for performance.

## Capabilities

### New Capabilities

- `canvas-search`: Sidebar canvas search (Search menu) shall present match results in a scrollable area with a stable layout, and shall keep the currently focused match visible when the user moves focus through the list.

### Modified Capabilities

- (none — no existing OpenSpec capabilities in this repo)

## Impact

- **Primary**: `packages/excalidraw/components/SearchMenu.tsx`, `packages/excalidraw/components/SearchMenu.scss` (flex/overflow and scroll-into-view behavior).
- **Secondary**: Any parent layout that wraps `SearchMenu` in the default sidebar (`packages/excalidraw/components/DefaultSidebar.tsx` / sidebar tab content) if height constraints must be applied there.
- **Tests**: Add or extend component/tests if the repo covers `SearchMenu` or sidebar layout; otherwise manual verification in the app.
- **APIs**: No public API or package export changes expected.
