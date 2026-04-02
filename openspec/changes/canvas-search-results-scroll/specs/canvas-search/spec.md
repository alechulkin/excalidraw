## ADDED Requirements

### Requirement: Search results list scrolls within the sidebar

The canvas search UI SHALL display match results in a vertically scrollable region when the list height would exceed the space available in the sidebar below the search field and result counter controls.

#### Scenario: Many matches fit in the panel

- **GIVEN** the editor is open with the canvas visible and the sidebar Search UI is active
- **GIVEN** a search for a query term has been executed and the results list shows N matches (N ≥ 1)
- **GIVEN** the scrollable results area has viewport height H, each result row has height R in the uniform-height case so the combined row height is C = N×R, and when row heights differ C is the sum of those heights; and C ≤ H
- **GIVEN** the results list is scrolled to the top (initial vertical scroll offset zero)
- **WHEN** the user views the results area after the search has completed
- **THEN** no scrollbar is required for the results area (or the scrollbar is present but inactive) and all rows remain visible without overflow clipping

#### Scenario: Many matches exceed the panel

- **GIVEN** the editor is open with the canvas visible and the sidebar Search UI is active
- **GIVEN** a search for a query term has been executed and the results list shows N matches (N ≥ 1)
- **GIVEN** the scrollable results area has viewport height H, each result row has height R in the uniform-height case so the combined row height is C = N×R, and when row heights differ C is the sum of those heights; and C > H
- **GIVEN** the results list is scrolled to the top (initial vertical scroll offset zero)
- **WHEN** the user views the results area after the search has completed
- **THEN** the results area shows a vertical scrollbar (or equivalent scrolling affordance) and the user SHALL be able to scroll to see every result row

#### Scenario: No matches found

- **GIVEN** the editor is open with the canvas visible and the sidebar Search UI is active
- **WHEN** a search has completed for a query and the result set contains zero matches
- **THEN** the result navigation controls (previous/next) SHALL NOT be shown
- **AND** the results list SHALL show no result rows (empty list inside the scrollable results region is acceptable)
- **AND** any empty-state or “no match” messaging the product shows for this case remains acceptable

#### Scenario: Single match

- **GIVEN** the sidebar Search UI is active and a search yields exactly one match
- **GIVEN** the scrollable results viewport height H and the single row’s height satisfy C ≤ H (same condition as in “Many matches fit in the panel”)
- **WHEN** the user views the results area after the search has completed
- **THEN** no scrollbar is required for the results area (or the scrollbar is present but inactive), consistent with “Many matches fit in the panel”
- **AND** if previous/next controls are shown, they SHALL leave the sole match focused (including when navigation uses wrap-around indexing so the focused index stays on that single row)

### Requirement: Focused match stays visible while navigating

The canvas search UI SHALL keep the currently focused result row visible inside the scrollable results area when the focused match changes through the built-in previous/next controls or when focus moves to another row in a way that the product already supports.

#### Scenario: Navigate down with next control

- **WHEN** the user moves focus from one match to the next using the search result navigation control
- **THEN** the newly focused row SHALL be scrolled into view inside the results area if it was previously outside the visible viewport of that area

#### Scenario: Navigate up with previous control

- **WHEN** the user moves focus to the previous match using the search result navigation control
- **THEN** the newly focused row SHALL be scrolled into view inside the results area if it was previously outside the visible viewport of that area

#### Scenario: Focus on first/last item when navigating

Relates to “Navigate down with next control” and “Navigate up with previous control” for scroll-into-view when the focused index changes at list ends.

**Next from last**

- **GIVEN** there are at least two matches and the focused row is the last in the ordered results list
- **WHEN** the user activates the next-match navigation control
- **THEN** focus SHALL wrap to the first match (index 0)
- **AND** that row SHALL be scrolled into view inside the results area if it was previously outside the visible viewport of that area

**Previous from first**

- **GIVEN** there are at least two matches and the focused row is the first in the ordered results list
- **WHEN** the user activates the previous-match navigation control
- **THEN** focus SHALL wrap to the last match
- **AND** that row SHALL be scrolled into view inside the results area if it was previously outside the visible viewport of that area
