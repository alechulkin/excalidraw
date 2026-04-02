# Canvas search results scroll — design

## Context

Canvas search lives in `@excalidraw/excalidraw` as `SearchMenu`, embedded in the default sidebar’s search tab (`DefaultSidebar`). Results are rendered in `MatchList` inside one or two `.layer-ui__search-result-container` blocks (frames, then text), wrapped by a single outer scroller `.layer-ui__search-results-scroll`. Previously, `overflow-y: auto` on the section containers combined with a flex parent that did not establish a definite height led to overflow or nested scrollbars. The fix is the flexbox pattern in **Decisions** §1. Focus changes via prev/next must scroll the active row’s DOM node into view so the highlight stays visible inside the scroll region.

## Goals / Non-Goals

**Goals:**

- The match list area SHALL consume remaining sidebar height below the search field and counter row, with vertical scrolling when content exceeds that area.
- Changing the focused match (UI arrows, clicking a row, or any existing focus path) SHALL keep the focused row visible inside the scrollable region without confusing jumps.
- The scroll region that contains `MatchList` (e.g. the results scroller under `.layer-ui__search-results-scroll` and any scrollable ancestor that clips the list) SHOULD expose appropriate ARIA for screen readers, such as `role="region"` and a concise `aria-label` (aligned with existing i18n) naming the results area.
- Scroll animations invoked by `scrollIntoView` (see **Decisions** §2) SHALL respect the user’s `prefers-reduced-motion` preference: use non-animated or instant scrolling when that media query matches, and only use smooth or animated scroll behavior when reduced motion is not requested.
- Keyboard behavior SHALL keep focus predictable relative to the scrollable results area: **Tab** / **Shift+Tab** follow the normal focus order for focusable controls in the search UI (field, navigation buttons, etc.); **Arrow** keys and prev/next controls change the *focused match* index (including wrap from last→first and first→last) without moving DOM focus away from the element that currently has focus, so updating the highlighted row does not steal keyboard focus. Rows that are not in the tab order (e.g. `tabIndex={-1}`) remain reachable by pointer; if a row is explicitly focused (e.g. after a click), Tab order from that point should remain consistent with the rest of the panel rather than trapping focus inside the list unless the product intentionally adds a roving-tabindex pattern later.
- Preserve current grouping (frames vs text), i18n, and canvas zoom/pan behavior tied to the focused match.

**Non-Goals:**

- Virtualizing thousands of DOM nodes or replacing the list with “current match only” UI (issue discussion suggests this as a possible follow-up; not required for the initial fix).
- Changing search matching semantics or keyboard shortcuts beyond what is needed for scroll-into-view.

## Decisions

1. **Flex height and scroll** — Use these concrete DOM targets and rules (implemented in `SearchMenu.scss` / `SearchMenu.tsx`):

   - **`.layer-ui__search`** (column flex root wrapping header, count row, and results): `display: flex; flex-direction: column`; `min-height: 0` so the flex item can shrink below its content height and pass a bounded height to descendants; `flex: 1 1 auto` (or equivalent) so the search panel fills the sidebar tab area.
   - **`.layer-ui__search-results-scroll`** (element wrapping the entire `MatchList`): `flex: 1 1 0` (equivalent intent to `flex: 1` with a zero flex-basis for “fill remaining space”); `min-height: 0`; `overflow-y: auto` (and `overflow-x: hidden` if horizontal clipping is desired) so the list has a definite max height and one vertical scrollbar for all matches.
   - **`.layer-ui__search-result-container`** (frames vs text sections): **do not** set `overflow-y: auto` here; scrolling stays on `.layer-ui__search-results-scroll` only, avoiding double scrollbars between outer wrapper and inner sections.
   - **If two independent scroll regions are ever required** (e.g. separate scroll for frames and texts): apply the same pattern to **each** region—each gets its own flex child with `flex: 1` (or `flex: 1 1 0`), `min-height: 0`, and `overflow-y: auto`—rather than stacking `overflow-y: auto` on nested wrappers.

   *Rationale:* matches browser flex behavior and fixes overflow without new dependencies. *Alternative:* fixed `max-height` in px — rejected as fragile across themes and sidebar widths.

2. **Scroll focused item into view** — Implemented in `MatchList` (`SearchMenu.tsx`), not in a `useEffect` on `focusIndex`:

   - **DOM target:** Each `ListItem` registers its root `div` in a `Map<number, HTMLDivElement>` via a ref callback keyed by global result index (frames first, then texts). When `focusIndex` or the match list identity (`matches.nonce`) changes, resolve `itemRefs.current.get(focusIndex)` and call `scrollIntoView` on that element only (no container `querySelector` needed).
   - **Timing:** Run the scroll from a `useLayoutEffect` keyed on `focusIndex` and `matches.nonce` so the scroll runs after DOM updates and **before** the browser paints, avoiding a one-frame flash or visible jump that a plain `useEffect` can cause.
   - **Options:** Pass `block: "nearest"` and `inline: "nearest"` by default so the list moves the minimum amount. If manual QA shows clipping at the first/last row or with sticky chrome, retry or switch to `block: "center"` for those edge cases only.
   - **Motion:** Read `window.matchMedia("(prefers-reduced-motion: reduce)")` when performing the scroll; use `behavior: "auto"` when it matches, and `behavior: "smooth"` otherwise (only for this path—do not force smooth scroll for users who prefer reduced motion).
   - **Rapid navigation:** Coalesce many `focusIndex` updates in quick succession (e.g. held arrow keys) by scheduling `scrollIntoView` with `requestAnimationFrame`: cancel any pending frame id in the effect cleanup and schedule a single callback so at most one scroll runs per frame toward the latest index, reducing redundant layout work. A short debounce is an acceptable alternative if rAF is problematic in a given environment.
   - *Rationale:* minimal code, aligns with keyboard navigation expectations. *Alternative:* manual `scrollTop` math — more control but more code and edge cases.

3. **Implementation location** — Adjust `SearchMenu.scss` and `SearchMenu.tsx` first; only touch sidebar shell components if inspection shows the tab panel does not pass height down. *Rationale:* keeps the change localized.

## Risks / Trade-offs

- **[Risk] Double scrollbars** if both `.layer-ui__search-results-scroll` and `.layer-ui__search-result-container` (or similar inner boxes) set `overflow-y: auto` → **Mitigation:** keep a single scrollable ancestor (the results wrapper); inner section containers stay `overflow` visible (aside from row-level clipping like `.preview-text` ellipsis).
- **[Risk] `scrollIntoView` scrolls the whole page** in some browsers → **Mitigation:** scope scroll to the list container if needed (`scrollIntoView` options / scrollable ancestor), or use a ref on the scroll parent.
- **[Trade-off]** Rendering hundreds of rows remains DOM-heavy; acceptable for the reported bug; revisit virtualization if performance issues appear. For production acceptance and mitigations:
  - **Initial render budget:** treat the current approach as acceptable when opening or refreshing the search panel stays roughly under **300 ms** to first paint/interaction with on the order of **500** visible result rows on a mid-range laptop (measure in DevTools Performance; budget is a guideline, not a hard SLA). Exceeding it consistently should trigger investigation before shipping further list features.
  - **Memory and retention:** each full list multiplies DOM nodes and React fiber work per tab; with **many browser tabs** open, total resident memory grows roughly with (tabs × result count × row complexity), which increases pressure on the tab discarder and can shorten effective **retention** (tabs evicted sooner, or jank when returning to a cold tab). Prefer not to hold unbounded match arrays in memory longer than needed if future work adds persistence across navigations.
  - **Mitigation ladder:** before committing to **full list virtualization**, consider **lazy rendering** for off-screen rows (e.g. **IntersectionObserver**–gated mounts or lightweight placeholders) as a smaller fallback—fewer live nodes while preserving scroll height and `scrollIntoView` behavior—then escalate to virtualization if budgets still fail.

## Migration Plan

Not applicable — client-only UI change; no data migration. Rollback is a revert of the layout and scroll-into-view logic.

## Open Questions

- Whether the sidebar tab content also needs `min-height: 0` in addition to `.layer-ui__search`; confirm if any ancestor flex item still blocks height propagation.
- Final choice between `"nearest"` vs `"center"` for `scrollIntoView` after manual QA with long lists and narrow sidebars.
