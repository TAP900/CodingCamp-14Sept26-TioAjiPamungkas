# Requirements Document

## Introduction

The Expense & Budget Visualizer is a client-side web application that enables users to track personal expenses by entering transactions, viewing a categorized list, monitoring a running total balance, and exploring spending distribution through a dynamic pie chart. The application requires no backend server, persists data using the browser's Local Storage API, and is implemented with plain HTML, CSS, and Vanilla JavaScript. Users may also personalize the experience through custom categories, transaction sorting, and a dark/light mode toggle.

---

## Glossary

- **App**: The Expense & Budget Visualizer web application.
- **Transaction**: A single expense record consisting of an item name, a monetary amount, and a category.
- **Transaction_Form**: The UI form used to input and submit a new Transaction.
- **Transaction_List**: The scrollable UI component that displays all stored Transactions.
- **Category**: A label grouping Transactions by spending type (e.g., Food, Transport, Fun, or a user-defined label).
- **Custom_Category**: A user-defined Category name added beyond the default set.
- **Balance_Display**: The UI element at the top of the App showing the computed total of all Transaction amounts.
- **Pie_Chart**: The visual chart component (powered by Chart.js or equivalent) showing spending distribution by Category.
- **Storage**: The browser's Local Storage API used for client-side data persistence.
- **Theme**: The visual color scheme of the App, toggled between dark mode and light mode.
- **Sort_Control**: The UI control allowing users to reorder the Transaction_List by amount or category.

---

## Requirements

### Requirement 1: Transaction Input Form

**User Story:** As a user, I want to fill out a form with an item name, amount, and category so that I can record a new expense transaction.

#### Acceptance Criteria

1. THE Transaction_Form SHALL present an item name text field, a numeric amount field, and a category selector containing at minimum the options Food, Transport, and Fun.
2. WHEN the user submits the Transaction_Form with all fields populated and a valid positive numeric amount, THE App SHALL add the Transaction to the Transaction_List and persist it to Storage.
3. IF the user submits the Transaction_Form with one or more empty fields, THEN THE Transaction_Form SHALL display an inline validation error identifying each empty field and SHALL NOT add a Transaction.
4. IF the user enters a non-positive or non-numeric value in the amount field, THEN THE Transaction_Form SHALL display a validation error stating the amount must be a positive number and SHALL NOT add a Transaction.
5. WHEN a Transaction is successfully added, THE Transaction_Form SHALL reset all fields to their default empty or placeholder state.

---

### Requirement 2: Transaction List Display

**User Story:** As a user, I want to see a scrollable list of all my transactions so that I can review my recorded expenses at a glance.

#### Acceptance Criteria

1. THE Transaction_List SHALL display all stored Transactions, each showing the item name (up to 50 characters), amount formatted as a decimal number with exactly 2 decimal places preceded by the currency symbol, and category label.
2. WHILE the number of Transactions exceeds the visible height of the Transaction_List container, THE Transaction_List SHALL remain scrollable without obscuring other UI elements.
3. WHEN the App loads, THE Transaction_List SHALL render all Transactions previously persisted in Storage.
4. WHEN the user adds a new Transaction, THE Transaction_List SHALL insert the new entry at the top of the list and display it without requiring a page reload.
5. IF Storage contains no Transactions, THEN THE Transaction_List SHALL display a message indicating that no transactions have been recorded yet.

---

### Requirement 3: Transaction Deletion

**User Story:** As a user, I want to delete a transaction from the list so that I can correct mistakes or remove outdated records.

#### Acceptance Criteria

1. THE Transaction_List SHALL render a delete control for each Transaction entry.
2. WHEN the user activates the delete control for a Transaction, THE App SHALL display a confirmation prompt requiring explicit user approval before proceeding with deletion.
3. WHEN the user confirms deletion, THE App SHALL remove that Transaction from the Transaction_List, remove it from Storage, and update the Balance_Display and Pie_Chart to reflect the updated totals within 500 milliseconds.
4. IF Storage fails to remove the Transaction, THEN THE App SHALL display an error message indicating the deletion failed, retain the Transaction in the Transaction_List, and leave the Balance_Display and Pie_Chart unchanged.
5. WHEN a Transaction is deleted, THE Transaction_List SHALL reflect the removal without requiring a page reload.

---

### Requirement 4: Total Balance Display

**User Story:** As a user, I want to see my total expenditure at the top of the page so that I always know how much I have spent in total.

#### Acceptance Criteria

1. THE Balance_Display SHALL be positioned at the top of the App and SHALL show the sum of the amounts of all Transactions as a decimal number with exactly 2 decimal places preceded by the currency symbol.
2. WHEN a Transaction is added, THE Balance_Display SHALL update to reflect the new total within 1 second of the user interaction completing, without a page reload.
3. WHEN a Transaction is deleted, THE Balance_Display SHALL update to reflect the reduced total within 1 second of the user interaction completing, without a page reload.
4. WHEN no Transactions exist, THE Balance_Display SHALL show a total of zero formatted as a decimal number with exactly 2 decimal places preceded by the currency symbol (e.g., $0.00).

---

### Requirement 5: Spending Distribution Pie Chart

**User Story:** As a user, I want to see a pie chart of my spending by category so that I can understand where my money is going visually.

#### Acceptance Criteria

1. THE Pie_Chart SHALL display one segment per Category that has at least one expense Transaction, sized proportionally to that Category's total expense amount relative to the sum of all expense Transaction amounts.
2. WHEN a Transaction is added or deleted, THE Pie_Chart SHALL re-render to reflect the updated Category totals within 1 second without a page reload.
3. WHEN no expense Transactions exist, THE Pie_Chart SHALL display a placeholder state containing a message indicating that no spending data is available.
4. THE Pie_Chart SHALL display a legend that includes, for each segment, the Category name and the percentage of total spending that segment represents, rounded to one decimal place.

---

### Requirement 6: Client-Side Data Persistence

**User Story:** As a user, I want my transactions to be saved automatically so that my data is available when I return to the app.

#### Acceptance Criteria

1. WHEN a Transaction is added, THE App SHALL write the updated Transaction collection to Storage and SHALL NOT consider the add operation complete until the write succeeds.
2. WHEN a Transaction is deleted, THE App SHALL write the updated Transaction collection to Storage and SHALL NOT consider the delete operation complete until the write succeeds.
3. WHEN the App initializes, THE App SHALL read all Transactions from Storage and render them in the Transaction_List, Balance_Display, and Pie_Chart.
4. IF a write to Storage fails during an add or delete operation, THEN THE App SHALL display an error message indicating the save failed, preserve the in-memory Transaction state, and keep all UI controls fully interactive.
5. IF Storage is unavailable or returns a read error on initialization, THEN THE App SHALL display a dismissible warning message and initialize with an empty Transaction collection, with all UI controls fully interactive.

---

### Requirement 7: Custom Categories

**User Story:** As a user, I want to add my own spending categories so that I can organize expenses in a way that fits my personal habits.

#### Acceptance Criteria

1. WHEN the user opens the category selector in the Transaction_Form, THE Transaction_Form SHALL provide an input field and a submit control that allow the user to enter and submit a new Custom_Category name.
2. WHEN a Custom_Category is submitted with a name that is non-empty, unique (case-insensitive), and between 1 and 50 characters in length, THE App SHALL add the Custom_Category to the category selector in the Transaction_Form and persist the updated category list to Storage.
3. IF the user submits a Custom_Category name that is empty, already exists (case-insensitive), or exceeds 50 characters, THEN THE Transaction_Form SHALL display a validation error message indicating the specific violation and SHALL NOT add the category to the category selector or to Storage.
4. WHEN the App initializes, THE App SHALL load all previously persisted Custom_Categories and include them in the Transaction_Form category selector.
5. IF Storage is unavailable when THE App attempts to persist a new Custom_Category, THEN THE App SHALL display an error message indicating the category could not be saved and SHALL NOT add the Custom_Category to the category selector.
6. IF a Custom_Category has no associated transactions, THEN THE Pie_Chart SHALL NOT render a segment for that Custom_Category.

---

### Requirement 8: Transaction Sorting

**User Story:** As a user, I want to sort my transaction list by amount or category so that I can find and review expenses more efficiently.

#### Acceptance Criteria

1. THE Sort_Control SHALL offer sorting options for ascending amount, descending amount, and category name (alphabetical), with no sort option active by default on initial load.
2. WHEN the user selects a sort option, THE Transaction_List SHALL re-render the Transactions in the selected order without a page reload.
3. WHILE a sort option is active, THE Transaction_List SHALL maintain that sort order when new Transactions are added or existing Transactions are deleted.
4. THE Sort_Control SHALL display a distinct active state marker on the currently selected sort option, and display no active state marker when no sort option is selected.
5. IF two or more Transactions share the same amount when sorting by amount, THEN THE Transaction_List SHALL order those Transactions by their date in descending order as a secondary sort.
6. WHEN the user selects the category sort option, THE Transaction_List SHALL order Transactions with the same category name by date in descending order as a secondary sort.

---

### Requirement 9: Dark/Light Mode Toggle

**User Story:** As a user, I want to switch between dark and light visual themes so that I can use the app comfortably in different lighting environments.

#### Acceptance Criteria

1. THE App SHALL provide a Theme toggle control that is persistently visible and reachable within one interaction from the main interface.
2. WHEN the user activates the Theme toggle, THE App SHALL immediately switch between dark mode and light mode color schemes and apply the change to all rendered UI components without requiring a page reload.
3. WHEN the App initializes, THE App SHALL apply the Theme last selected by the user as persisted in Storage, defaulting to light mode if no preference is stored.
4. THE Theme toggle control SHALL update its icon or label to reflect the currently active Theme immediately after each toggle activation.
5. IF Storage is unavailable when THE App attempts to read the saved Theme preference on initialization, THEN THE App SHALL default to light mode and display no error message.

---

### Requirement 10: Technology and Compatibility Constraints

**User Story:** As a developer, I want the app to be built with plain HTML, CSS, and Vanilla JavaScript so that it runs in any modern browser without build tools or server setup.

#### Acceptance Criteria

1. THE App SHALL be implemented using HTML for structure, a single CSS file located in the `css/` directory for styling, and a single JavaScript file located in the `js/` directory for behavior.
2. THE App SHALL NOT depend on JavaScript frameworks or libraries except for a single chart rendering library (Chart.js) loaded via a CDN `<script>` tag; no other external scripts, modules, or build-tool-generated bundles are permitted.
3. THE App SHALL render all UI elements, accept all user inputs, and persist data correctly in the two most recent stable releases of Chrome, Firefox, Edge, and Safari available at the time of testing, without requiring additional plugins or configuration.
4. THE App SHALL operate as a standalone web application loadable by opening the HTML file directly in a browser, with no backend server required.
5. IF the Chart.js CDN resource fails to load, THEN THE App SHALL display an error message indicating that chart rendering is unavailable while all non-chart features remain functional.

---

### Requirement 11: Performance and Responsiveness

**User Story:** As a user, I want the app to load quickly and respond instantly to my actions so that managing my expenses feels effortless.

#### Acceptance Criteria

1. THE App SHALL complete initial render and display all persisted Transactions within 2 seconds on the latest stable release of Chrome, Firefox, Edge, or Safari with no network dependency beyond the initial CDN script load.
2. WHEN the user adds or deletes a Transaction, THE App SHALL update the Transaction_List, Balance_Display, and Pie_Chart within 300 milliseconds of the user interaction completing.
3. THE App SHALL accept and visually acknowledge user input within 300 milliseconds during any data update operation, with no full-page reloads triggered by user actions.
4. IF the Chart.js CDN script fails to load during initialization, THEN THE App SHALL display an error message indicating chart rendering is unavailable and halt chart initialization while keeping all other features fully functional.

---

### Requirement 12: Visual Design and Usability

**User Story:** As a user, I want a clean, readable interface with a clear visual hierarchy so that I can use the app without confusion or instructions.

#### Acceptance Criteria

1. THE App SHALL present the Balance_Display, Transaction_Form, Transaction_List, and Pie_Chart as visually distinct sections, each with a visible boundary and a section label that identifies its purpose.
2. THE App SHALL use typography with a contrast ratio of at least 4.5:1 for normal text (under 18pt or 14pt bold) and at least 3:1 for large text (18pt or larger, or 14pt bold) in both dark and light Theme modes, as defined by WCAG 2.1 AA.
3. THE App SHALL apply a consistent base font size, uniform spacing between sections, and aligned form controls so that the visual hierarchy is perceivable without instructions.
