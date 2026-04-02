## ADDED Requirements

### Requirement: Search results list scrolls within the sidebar

The canvas search UI SHALL display match results in a vertically scrollable region when the list height would exceed the space available in the sidebar below the search field and result counter controls.

#### Scenario: Many matches fit in the panel

- **WHEN** the user runs a search and the combined height of all result rows is less than or equal to the available results area
- **THEN** no scrollbar is required for the results area (or the scrollbar is present but inactive) and all rows remain visible without overflow clipping

#### Scenario: Many matches exceed the panel

- **WHEN** the user runs a search and the combined height of all result rows exceeds the available results area
- **THEN** the results area shows a vertical scrollbar (or equivalent scrolling affordance) and the user SHALL be able to scroll to see every result row

### Requirement: Focused match stays visible while navigating

The canvas search UI SHALL keep the currently focused result row visible inside the scrollable results area when the focused match changes through the built-in previous/next controls or when focus moves to another row in a way that the product already supports.

#### Scenario: Navigate down with next control

- **WHEN** the user moves focus from one match to the next using the search result navigation control
- **THEN** the newly focused row SHALL be scrolled into view inside the results area if it was previously outside the visible viewport of that area

#### Scenario: Navigate up with previous control

- **WHEN** the user moves focus to the previous match using the search result navigation control
- **THEN** the newly focused row SHALL be scrolled into view inside the results area if it was previously outside the visible viewport of that area
