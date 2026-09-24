# Implementation Plan: Expense & Budget Visualizer

## Overview

This plan builds the client-side Expense & Budget Visualizer incrementally in three files: `index.html`, `css/styles.css`, and `js/app.js`, using Chart.js (via CDN) for the pie chart. Work starts with the static scaffold (markup in the required top-to-bottom order, stylesheet, and an empty JS skeleton), then layers in the JS logical sections from the design: constants/config, storage layer, pure domain logic, formatting, rendering, and finally event wiring that ties everything together on `DOMContentLoaded`. Each step builds on the previous one so there is no orphaned code.

Testing is documented as optional. The user stated no test setup is required for delivery, so all test-writing sub-tasks are marked with `*` and MUST NOT be implemented as part of delivery. They map each of the 12 correctness properties to a property test for traceability.

## Tasks

- [x] 1. Scaffold the static app shell (HTML + CSS + JS skeleton)
  - [x] 1.1 Create `index.html` structure with components in top-to-bottom order
    - Add `<header>` Balance_Display region with `#balance` and `#balance-warning` nodes
    - Add `<section>` Input_Form as `#tx-form` with `#name`, `#amount`, `#category` (options: Food, Transport, Fun; blank default), and per-field error nodes
    - Add `<section>` Transaction_List with scrollable `#tx-list`, `#empty-state`, and `#list-error`
    - Add `<section>` Pie_Chart with `<canvas id="pie">` and `#chart-empty` message node
    - Link `css/styles.css` in `<head>`, load Chart.js from CDN, and load `js/app.js` with `defer`
    - _Requirements: 1.1, 2.1, 2.4, 3.1, 4.4, 5.5, 7.1_

  - [x] 1.2 Create `css/styles.css` (the single stylesheet)
    - Style the four regions to render Balance_Display, Input_Form, Transaction_List, Pie_Chart top-to-bottom
    - Make `#tx-list` vertically scrollable with a max height and overflow-y auto
    - Apply minimum 14px text size and ensure text/background contrast of at least 4.5:1
    - Use a fluid/responsive layout with no horizontal scroll or overlap from 320px to 1920px
    - Add styles for validation error text, save-failure banner, and range-exceeded indication
    - _Requirements: 2.2, 7.1, 7.3, 7.4_

  - [x] 1.3 Create `js/app.js` skeleton (the single behavior file)
    - Lay out the logical section comment blocks: Constants & config, State, Storage layer, Domain logic, Formatting, Rendering, Event wiring
    - Declare empty function stubs matching the design's conceptual signatures so later steps fill them in
    - _Requirements: 7.3_

- [x] 2. Define constants, config, and state
  - [x] 2.1 Add constants/config and in-memory state
    - Define category list `["Food", "Transport", "Fun"]`, `MAX_NAME_LEN = 50`, `MAX_AMOUNT = 999999999.99`, `MAX_AMOUNT_CENTS`, `BALANCE_MIN`/`BALANCE_MAX`, `MAX_TRANSACTIONS = 10000`, and storage key `"ebv.transactions.v1"`
    - Declare the `transactions` array as the single source of truth and a `lastValidBalanceCents` state variable
    - _Requirements: 1.1, 4.5, 6.6_

- [ ] 3. Implement the storage layer
  - [x] 3.1 Implement `loadTransactions()` with error handling
    - Read the storage key; wrap `JSON.parse` in try/catch; return `[]` when the key is absent, value is not valid JSON, parsed value is not an array, or any element fails shape/type validation (id, name, positive integer amountCents, valid category, createdAt)
    - Never throw; leave the stored value in place on failure
    - _Requirements: 6.3, 6.4_

  - [-] 3.2 Implement `saveTransactions(transactions)` with error handling
    - Serialize to JSON and wrap `localStorage.setItem` in try/catch to handle quota-exceeded, disabled storage, and private-mode errors
    - Return `true` on success and `false` on failure; never throw
    - _Requirements: 6.1, 6.2, 6.5_


- [ ] 4. Implement domain logic (pure functions)
  - [~] 4.1 Implement `validateTransactionInput(raw)`
    - Validate name: trimmed length ≥ 1 and raw length ≤ 50
    - Validate amount on its string form: parses as a number, `> 0`, `≤ 999,999,999.99`, at most 2 decimal places; then convert to integer cents
    - Validate category: one of Food, Transport, Fun (a value is selected)
    - Return `{ ok: true, value: { name, amountCents, category } }` or `{ ok: false, errors: { name?, amount?, category? } }`
    - _Requirements: 1.4, 1.5, 1.6_

  - [~] 4.2 Implement `createTransaction(fields)`
    - Build a Transaction from validated fields, assigning `id` (e.g. `crypto.randomUUID()`) and `createdAt` (epoch ms)
    - _Requirements: 1.2_


  - [~] 4.6 Implement `computeTotal(transactions)`
    - Sum all `amountCents`; return `{ cents, exceededRange }` where `exceededRange` is true when the total falls outside `BALANCE_MIN`..`BALANCE_MAX`
    - _Requirements: 4.1, 4.5_

  - [~] 4.7 Implement `aggregateByCategory(transactions)`
    - Group summed `amountCents` by category, dropping any category summing to zero
    - _Requirements: 5.1, 5.2_


- [ ] 5. Implement formatting helpers
  - [~] 5.1 Implement `formatCurrency(cents)` and `truncateName(name)`
    - `formatCurrency`: convert integer cents to a string with exactly two decimal places (e.g. 1234 → "12.34")
    - `truncateName`: return a name truncated to at most 50 characters
    - _Requirements: 2.5_

- [ ] 6. Checkpoint - core logic complete
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 7. Implement rendering
  - [ ] 7.1 Implement `renderList(transactions)`
    - Render each transaction row with truncated name, currency-formatted amount, and category, plus a delete control carrying the transaction id
    - Order rows strictly newest-first by `createdAt`; show `#empty-state` when there are no transactions
    - _Requirements: 2.1, 2.4, 2.5, 2.6, 3.1_

  - [ ]* 7.2 Write property test for rendered row formatting
    - **Property 4: Rendered transactions are truncated and currency-formatted**
    - **Validates: Requirements 2.5**

  - [ ]* 7.3 Write property test for newest-first ordering
    - **Property 5: Rendered list is ordered newest-first**
    - **Validates: Requirements 2.6**

  - [ ] 7.4 Implement `renderBalance(transactions)`
    - Call `computeTotal`; when in range, display the formatted balance and update `lastValidBalanceCents`; when empty, show `0.00`
    - When `exceededRange` is true, retain the last valid balance and show a range-exceeded indication
    - _Requirements: 4.1, 4.4, 4.5_

  - [ ]* 7.5 Write property test for out-of-range balance handling
    - **Property 12: Out-of-range totals retain the last valid balance**
    - **Validates: Requirements 4.5**

  - [ ] 7.6 Implement `renderChart(transactions)`
    - Build datasets from `aggregateByCategory`; create/update a Chart.js pie chart with segments proportional to each category's share
    - When there is no spending data, hide segments and show the `#chart-empty` message
    - _Requirements: 5.1, 5.2, 5.5_

  - [ ] 7.7 Implement `renderAll(transactions)`
    - Call `renderList`, `renderBalance`, and `renderChart` from the current array so all three views stay consistent (single write path to the DOM)
    - _Requirements: 2.3, 4.2, 4.3, 5.3, 5.4, 7.2_

- [ ] 8. Wire up events and initialization
  - [ ] 8.1 Implement `handleSubmit(event)` for adding a transaction
    - Prevent default, read the form fields, run `validateTransactionInput`
    - On failure, show per-field messages and do not mutate state
    - On success, prepend the new transaction (newest-first), call `saveTransactions`; if save returns false keep in-memory data and show a "could not save" banner; then `renderAll` and clear the form
    - _Requirements: 1.2, 1.3, 1.4, 1.5, 1.6, 4.2, 5.3, 6.1, 6.5, 7.2_

  - [ ] 8.2 Implement `handleDelete(transactionId)` with confirmation
    - Prompt with native `confirm()` before mutating; on cancel, leave state unchanged
    - On confirm, remove from a working copy and only commit if `saveTransactions` succeeds; on save failure restore the transaction and show a list-level error indication; then `renderAll`
    - Attach via event delegation on `#tx-list`
    - _Requirements: 3.2, 3.3, 3.4, 3.5, 3.6, 4.3, 5.4, 6.2, 6.5, 7.2_

  - [ ]* 8.3 Write property test for delete-and-recompute
    - **Property 7: Deleting a transaction removes it and recomputes derived values**
    - **Validates: Requirements 3.2, 3.5**

  - [ ]* 8.4 Write property test for cancelled deletion
    - **Property 8: Cancelling deletion leaves the set unchanged**
    - **Validates: Requirements 3.4**

  - [ ] 8.5 Implement `init()` and bind on `DOMContentLoaded`
    - On load, call `loadTransactions()` into the `transactions` array, bind the form submit and list delete delegation handlers, and call `renderAll` to populate all three views (including empty states)
    - _Requirements: 6.3, 6.4, 2.4, 4.4, 5.5_

- [ ] 9. Final checkpoint - integration complete
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional test-writing tasks; the user stated no test setup is required for delivery, so they MUST NOT be implemented as part of delivery. They exist for traceability, mapping each of the 12 correctness properties to a property test.
- All application code stays in exactly three files: `index.html`, `css/styles.css`, `js/app.js`. Chart.js is a third-party CDN dependency and does not count against the one-JS-file rule.
- Each task references specific granular requirements for traceability.
- Checkpoints ensure incremental validation at core-logic and integration boundaries.
- `renderAll` is the only path that writes to the DOM views, keeping list, balance, and chart consistent.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.2", "1.3"] },
    { "id": 1, "tasks": ["2.1"] },
    { "id": 2, "tasks": ["3.1", "3.2", "4.1", "4.2", "4.6", "4.7", "5.1"] },
    { "id": 3, "tasks": ["3.3", "3.4", "4.3", "4.4", "4.5", "4.8", "4.9"] },
    { "id": 4, "tasks": ["7.1", "7.4", "7.6"] },
    { "id": 5, "tasks": ["7.2", "7.3", "7.5", "7.7"] },
    { "id": 6, "tasks": ["8.1", "8.2", "8.5"] },
    { "id": 7, "tasks": ["8.3", "8.4"] }
  ]
}
```
