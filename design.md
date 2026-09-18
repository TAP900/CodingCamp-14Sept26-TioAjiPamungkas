# Design Document: Expense & Budget Visualizer

## Overview

The Expense & Budget Visualizer is a standalone, client-side web application built with plain HTML, CSS, and Vanilla JavaScript. It lets users record expense transactions, see their running total, explore spending distribution through a pie chart, and manage custom categories — all without a backend server.

All data is persisted in the browser's Local Storage API. The chart is rendered by Chart.js loaded via CDN. The entire application is a single HTML file (`index.html`) supplemented by one CSS file (`css/styles.css`) and one JavaScript file (`js/app.js`). It opens directly in any modern browser without a build step or server.

### Goals

- Zero-dependency runtime (except Chart.js CDN)
- Instant feedback on every user action (within 300 ms per Requirement 11)
- Graceful degradation when Storage or Chart.js is unavailable
- Accessible, WCAG 2.1 AA-compliant contrast ratios in both themes

---

## Architecture

The application follows a **unidirectional data-flow** pattern implemented without a framework:

```
User Interaction
      │
      ▼
┌─────────────────────────────────────────────┐
│                  app.js                     │
│                                             │
│  ┌──────────┐    ┌────────────────────┐     │
│  │  State   │◄───│  Action Handlers   │     │
│  │ (in-mem) │    │ (add / delete /    │     │
│  └────┬─────┘    │  sort / theme /    │     │
│       │          │  custom category)  │     │
│       │          └────────────────────┘     │
│       │                    ▲                │
│       ▼                    │ DOM Events     │
│  ┌──────────┐    ┌──────────────────────┐   │
│  │ Renderer │───►│     DOM (index.html) │   │
│  └──────────┘    └──────────────────────┘   │
│       │                                     │
│       ▼                                     │
│  ┌──────────┐                               │
│  │ Storage  │  (localStorage read/write)    │
│  │  Layer   │                               │
│  └──────────┘                               │
└─────────────────────────────────────────────┘
```

### Key Architectural Decisions

| Decision | Rationale |
|---|---|
| Single-file JS (`js/app.js`) | Matches Requirement 10.1; keeps the project zero-build |
| In-memory state object as single source of truth | Avoids re-reading localStorage on every render; keeps UI updates fast |
| Full re-render on state change | Simple mental model; the transaction list is unlikely to reach thousands of entries |
| Chart.js for pie chart | Requirement 10.2 explicitly permits one chart library via CDN |
| localStorage for all persistence | Requirement 6; no backend permitted |

---

## Components and Interfaces

### HTML Structure (`index.html`)

```
<body>
  ├── <header>           — App title + Theme toggle button
  ├── #balance-section   — Balance_Display
  ├── #form-section      — Transaction_Form
  │     ├── #item-name   — Text input
  │     ├── #amount      — Number input
  │     ├── #category    — <select> + custom category sub-form
  │     └── #submit-btn  — Add Transaction button
  ├── #sort-section      — Sort_Control (3 buttons: amount↑, amount↓, category A–Z)
  ├── #list-section      — Transaction_List (<ul id="transaction-list">)
  └── #chart-section     — Pie_Chart (<canvas id="pie-chart">)
</body>
```

### JavaScript Modules (logical sections within `js/app.js`)

Because the project uses no build tools, the JS file is organized into clearly commented logical sections rather than ES modules:

#### 1. `State` — single source of truth

```js
const state = {
  transactions: [],   // Transaction[]
  categories: [],     // string[]  (default + custom)
  sortOption: null,   // 'amount-asc' | 'amount-desc' | 'category' | null
  theme: 'light',     // 'light' | 'dark'
};
```

#### 2. `StorageService` — localStorage wrapper

```
StorageService.load()          → { transactions, categories, theme } | null
StorageService.save(state)     → boolean (true = success)
StorageService.saveTheme(t)    → boolean
StorageService.loadTheme()     → 'light' | 'dark'
```

Wraps every localStorage call in try/catch and returns a success flag so callers can surface errors per Requirements 6.4, 6.5, 7.5, 9.5.

#### 3. `Validator` — pure validation functions

```
Validator.validateTransaction({ name, amount, category })
  → { valid: boolean, errors: { name?, amount?, category? } }

Validator.validateCategory(name, existingCategories)
  → { valid: boolean, error?: string }
```

#### 4. `Renderer` — DOM update functions

```
Renderer.renderBalance(total)
Renderer.renderTransactionList(transactions, sortOption)
Renderer.renderChart(transactions, chartInstance)
Renderer.renderCategoryOptions(categories)
Renderer.renderSortControls(sortOption)
Renderer.applyTheme(theme)
Renderer.showError(elementId, message)
Renderer.clearErrors(formId)
Renderer.showToast(message, type)   // 'error' | 'warning'
```

#### 5. `ChartManager` — Chart.js wrapper

```
ChartManager.init(canvasId)         → Chart instance | null
ChartManager.update(chart, data)    → void
ChartManager.showPlaceholder()      → void
```

Handles the case where Chart.js CDN fails to load (Requirement 10.5, 11.4).

#### 6. Action Handlers (event listeners)

| Handler | Triggered by |
|---|---|
| `handleAddTransaction` | form submit |
| `handleDeleteTransaction` | delete button click (with confirm dialog) |
| `handleSortChange` | sort button click |
| `handleAddCategory` | custom category form submit |
| `handleThemeToggle` | theme toggle button click |

#### 7. `App.init()` — bootstrap function

Called on `DOMContentLoaded`. Loads state from storage, renders all UI sections, attaches event listeners.

---

## Data Models

### Transaction

```js
/**
 * @typedef {Object} Transaction
 * @property {string} id          - UUID (crypto.randomUUID() or Date.now().toString())
 * @property {string} name        - Item name, max 50 characters
 * @property {number} amount      - Positive decimal number
 * @property {string} category    - Category label string
 * @property {number} timestamp   - Unix ms timestamp (Date.now()) for secondary sort
 */
```

### AppState (persisted to localStorage)

```js
/**
 * @typedef {Object} PersistedState
 * @property {Transaction[]} transactions
 * @property {string[]}      customCategories  — user-defined category names only
 * @property {string}        theme             — 'light' | 'dark'
 */
```

The default categories (`['Food', 'Transport', 'Fun']`) are defined as a constant in `app.js` and merged with `customCategories` at runtime. Only custom categories are stored, keeping the stored payload minimal.

### localStorage Keys

| Key | Value |
|---|---|
| `ebv_transactions` | JSON array of `Transaction` objects |
| `ebv_custom_categories` | JSON array of category name strings |
| `ebv_theme` | `'light'` or `'dark'` |

Using prefixed keys (`ebv_`) avoids collisions with other apps sharing the same origin.

### Sort State

Sort is **not** persisted (Sort_Control starts with no active option on load per Requirement 8.1). It lives only in `state.sortOption`.

### Balance Computation

Balance is always derived — never stored:

```js
const total = state.transactions.reduce((sum, t) => sum + t.amount, 0);
```

### Pie Chart Data Derivation

```js
// Group by category, sum amounts
const byCategory = state.transactions.reduce((acc, t) => {
  acc[t.category] = (acc[t.category] ?? 0) + t.amount;
  return acc;
}, {});
// Categories with zero transactions are excluded (Requirement 7.6)
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Transaction round-trip through storage

*For any* valid transaction (non-empty name ≤ 50 chars, positive amount, non-empty category), serializing the transaction collection to JSON and deserializing it back should produce a collection where every transaction field is bit-for-bit identical to the original.

**Validates: Requirements 6.1, 6.3**

---

### Property 2: Balance equals sum of all transaction amounts

*For any* non-empty collection of transactions, the computed balance displayed by `renderBalance` should equal the arithmetic sum of all transaction amounts, formatted to exactly 2 decimal places.

**Validates: Requirements 4.1, 4.2, 4.3**

---

### Property 3: Adding a valid transaction grows the list by exactly one

*For any* transaction list state and any valid transaction input, after a successful add the transaction list length should be exactly one greater than before.

**Validates: Requirements 1.2, 2.4**

---

### Property 4: Whitespace and invalid inputs are always rejected

*For any* combination of inputs where the name is empty or all-whitespace, the amount is non-positive or non-numeric, or the category is empty, the validator should return `valid: false` and the transaction list should remain unchanged.

**Validates: Requirements 1.3, 1.4**

---

### Property 5: Deleting a transaction removes exactly that transaction

*For any* transaction list containing a transaction with a given id, after deletion the resulting list should contain every other transaction exactly once and no entry with the deleted id.

**Validates: Requirements 3.3, 3.5**

---

### Property 6: Pie chart segments cover all and only represented categories

*For any* non-empty transaction list, the set of category labels in the pie chart data should be exactly the set of category labels that appear in at least one transaction — no more, no fewer.

**Validates: Requirements 5.1, 7.6**

---

### Property 7: Custom category validation rejects duplicates case-insensitively

*For any* existing category list and any new category name that is a case-insensitive duplicate of an existing entry, or is empty, or exceeds 50 characters, `validateCategory` should return `valid: false`.

**Validates: Requirements 7.2, 7.3**

---

### Property 8: Sort by amount produces a correctly ordered list

*For any* non-empty transaction list and sort direction (ascending or descending), the sorted list should satisfy the invariant that every adjacent pair of transactions is in the correct relative order by amount, with ties broken by descending timestamp.

**Validates: Requirements 8.2, 8.5**

---

### Property 9: Sort by category produces alphabetically ordered groups with date tie-break

*For any* non-empty transaction list sorted by category, transactions should be grouped so that every transaction's category comes at or after all transactions with lexicographically earlier category names, and within the same category transactions are ordered by descending timestamp.

**Validates: Requirements 8.2, 8.6**

---

### Property 10: Theme toggle is an involution (self-inverse)

*For any* current theme value, toggling twice should return to the original theme. In other words, `toggle(toggle(theme)) === theme`.

**Validates: Requirements 9.2**

---

## Error Handling

### Storage Unavailable on Read (Initialization)

- Catch the `localStorage` exception in `StorageService.load()`.
- Return `null` to `App.init()`.
- Display a dismissible warning banner (Requirement 6.5).
- Initialize with empty transactions, default categories, light theme.
- All UI controls remain interactive.

### Storage Failure on Write (Add / Delete / Custom Category)

- `StorageService.save()` returns `false`.
- The action handler aborts the in-memory state change.
- Display an inline toast error (Requirement 6.4, 7.5).
- The UI reflects the pre-action state (no phantom entry, no phantom deletion).

### Chart.js CDN Failure

- `ChartManager.init()` checks `typeof Chart === 'undefined'`.
- Shows a static error message inside `#chart-section` (Requirements 10.5, 11.4).
- All non-chart features (form, list, balance, theme, sorting) remain fully functional.

### Form Validation Errors

- `Validator.validateTransaction()` returns an errors map.
- Each field's error is rendered as an `aria-live` inline message immediately below the field.
- The form is not submitted; no transaction is added.

### Confirmation Before Delete

- A native `window.confirm()` dialog is shown before deletion (Requirement 3.2).
- If the user cancels, no state change occurs.
- Using the native dialog avoids custom modal complexity while satisfying the requirement.

### Transaction Deletion — Storage Failure

- If `StorageService.save()` returns `false` after a delete:
  - Roll back the in-memory deletion.
  - Re-render the list to restore the entry.
  - Show an error toast (Requirement 3.4).

---

## Testing Strategy

### Dual Testing Approach

The testing strategy combines **example-based unit tests** for specific scenarios and edge cases with **property-based tests** for universal behavioral invariants. The pure functions (`Validator`, `StorageService` serialization, sort comparators, balance computation, chart data derivation) are well-suited for property-based testing.

### Recommended Libraries

| Role | Library |
|---|---|
| Test runner | [Vitest](https://vitest.dev/) (zero-config, browser-compatible output) |
| Property-based testing | [fast-check](https://fast-check.dev/) |
| DOM testing | `jsdom` (via Vitest's `environment: 'jsdom'`) |

### Unit Tests (example-based)

Focus on:
- Specific valid and invalid form submissions
- Correct currency formatting (`$0.00`, `$1,234.56`)
- Empty-state messages in Transaction_List and Pie_Chart
- Theme persistence across simulated page reloads
- Sort_Control active-state marker rendering
- Confirmation dialog cancel path (no state change)
- Chart.js CDN failure fallback message rendering

### Property-Based Tests

Each property from the Correctness Properties section maps to one property test. Minimum **100 iterations** per test.

| Test | Property | fast-check Arbitraries |
|---|---|---|
| Storage round-trip | Property 1 | `fc.array(transactionArb)` |
| Balance equals sum | Property 2 | `fc.array(transactionArb, { minLength: 1 })` |
| Add grows list by 1 | Property 3 | `fc.array(transactionArb)` + `validInputArb` |
| Invalid input rejected | Property 4 | `invalidInputArb` (whitespace names, non-positive amounts) |
| Delete removes exactly one | Property 5 | `fc.array(transactionArb, { minLength: 1 })` |
| Chart covers exactly used categories | Property 6 | `fc.array(transactionArb, { minLength: 1 })` |
| Category validation rejects duplicates | Property 7 | `fc.array(fc.string())` + duplicates / boundary names |
| Amount sort order | Property 8 | `fc.array(transactionArb, { minLength: 2 })` |
| Category sort order | Property 9 | `fc.array(transactionArb, { minLength: 2 })` |
| Theme toggle involution | Property 10 | `fc.constantFrom('light', 'dark')` |

**Tag format for each property test:**
```js
// Feature: expense-budget-visualizer, Property N: <property_text>
```

### Integration / Smoke Tests

- App loads and renders from a pre-seeded `localStorage` fixture (Requirement 6.3)
- Chart.js CDN unavailable: error banner shown, list/form/balance fully interactive (Requirements 10.5, 11.4)
- `localStorage` quota exceeded simulation: error toast shown, in-memory state preserved

### Accessibility

- Contrast ratio checks using `axe-core` or manual Lighthouse audit for both themes (Requirement 12.2)
- Keyboard navigation through form, sort controls, delete buttons
- `aria-live` regions for dynamic error messages and balance updates
