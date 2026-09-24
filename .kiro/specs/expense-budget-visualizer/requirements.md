# Requirements Document

## Introduction

The Expense & Budget Visualizer is a client-side web application that allows a single user to record expense transactions, view them in a list, track a running total, and see spending distribution across categories through a pie chart. All data is stored in the browser using the Local Storage API, so no backend server is required. The application is built with HTML, CSS, and vanilla JavaScript, and is intended to run as a standalone web app or browser extension in modern browsers.

## Glossary

- **Visualizer**: The complete client-side Expense & Budget Visualizer web application.
- **Input_Form**: The UI component that accepts a new transaction's details (item name, amount, category).
- **Transaction**: A single recorded expense entry consisting of an item name, an amount, and a category.
- **Transaction_List**: The UI component that displays all recorded transactions.
- **Category**: The classification of a transaction, limited to one of "Food", "Transport", or "Fun".
- **Balance_Display**: The UI component that shows the total of all recorded transaction amounts.
- **Total_Balance**: The sum of the amounts of all recorded transactions, rounded to 2 decimal places, constrained to the range -999,999,999.99 to 999,999,999.99.
- **Pie_Chart**: The visual component that shows spending distribution grouped by category.
- **Storage**: The browser Local Storage API used to persist transaction data client-side.
- **Amount**: A numeric monetary value associated with a transaction, greater than 0, at most 999,999,999.99, with at most 2 decimal places.
- **Item_Name**: The descriptive label of a transaction, a string of at most 50 characters containing at least one non-whitespace character.

## Requirements

### Requirement 1: Add a Transaction

**User Story:** As a user, I want to add an expense with a name, amount, and category, so that I can record my spending.

#### Acceptance Criteria

1. THE Input_Form SHALL provide an item name field limited to a maximum of 50 characters, an amount field that accepts a numeric value, and a category field limited to the values "Food", "Transport", and "Fun".
2. WHEN a user submits the Input_Form with an item name containing at least one non-whitespace character and at most 50 characters, an amount that is a number greater than 0, at most 999,999,999.99, and with at most 2 decimal places, and a selected category, THE Visualizer SHALL create a Transaction from the submitted values and add the Transaction to the Transaction_List.
3. WHEN a Transaction is added, THE Visualizer SHALL clear the Input_Form fields.
4. IF the item name field contains no non-whitespace character or exceeds 50 characters when the Input_Form is submitted, THEN THE Visualizer SHALL reject the submission and display a validation message identifying the item name field.
5. IF the amount field is empty, contains a value that is not a number greater than 0, exceeds 999,999,999.99, or has more than 2 decimal places when the Input_Form is submitted, THEN THE Visualizer SHALL reject the submission and display a validation message identifying the amount field.
6. IF the category field has no selected value when the Input_Form is submitted, THEN THE Visualizer SHALL reject the submission and display a validation message identifying the category field.

### Requirement 2: Display the Transaction List

**User Story:** As a user, I want to see all my recorded expenses in a list, so that I can review what I have spent.

#### Acceptance Criteria

1. THE Transaction_List SHALL display each recorded Transaction with its item name, amount, and category.
2. WHERE the number of recorded transactions exceeds the visible area, THE Transaction_List SHALL provide vertical scrolling to access all transactions.
3. WHEN a Transaction is added, THE Transaction_List SHALL display the new Transaction within 1 second of the add operation completing.
4. WHILE no transactions are recorded, THE Transaction_List SHALL display an empty-state message indicating that no transactions exist.
5. WHEN a Transaction is displayed, THE Transaction_List SHALL show the item name truncated to a maximum of 50 characters and the amount formatted as a currency value with two decimal places.
6. WHEN multiple transactions are displayed, THE Transaction_List SHALL order them from most recently added to least recently added.

### Requirement 3: Delete a Transaction

**User Story:** As a user, I want to delete a recorded expense, so that I can remove entries I no longer want to track.

#### Acceptance Criteria

1. THE Transaction_List SHALL display a delete control adjacent to each displayed Transaction.
2. WHEN a user activates the delete control for a Transaction, THE Visualizer SHALL remove that Transaction from the Transaction_List within 1 second and update the displayed Transaction_List to no longer show that Transaction.
3. WHEN a user activates the delete control for a Transaction, THE Visualizer SHALL prompt the user to confirm the deletion before removing the Transaction.
4. IF the user cancels the deletion confirmation prompt, THEN THE Visualizer SHALL retain the Transaction in the Transaction_List unchanged.
5. WHEN a Transaction is removed from the Transaction_List, THE Visualizer SHALL recalculate and update all affected budget totals and visualizations to exclude the removed Transaction within 1 second.
6. IF removal of a Transaction fails, THEN THE Visualizer SHALL retain the Transaction in the Transaction_List and display an error indication informing the user that the deletion did not complete.

### Requirement 4: Display and Update Total Balance

**User Story:** As a user, I want to see the total of my expenses at the top of the page, so that I can understand my overall spending at a glance.

#### Acceptance Criteria

1. THE Balance_Display SHALL show the Total_Balance calculated as the sum of the amounts of all recorded transactions, rounded to 2 decimal places.
2. WHEN a Transaction is added, THE Visualizer SHALL recalculate the Total_Balance and update the Balance_Display within 500 milliseconds.
3. WHEN a Transaction is deleted, THE Visualizer SHALL recalculate the Total_Balance and update the Balance_Display within 500 milliseconds.
4. WHILE no transactions are recorded, THE Balance_Display SHALL show a Total_Balance of 0.00.
5. IF the recalculated Total_Balance would fall outside the range -999,999,999.99 to 999,999,999.99, THEN THE Visualizer SHALL retain the last valid Total_Balance and display an indication that the range was exceeded.

### Requirement 5: Visualize Spending by Category

**User Story:** As a user, I want to see a pie chart of my spending by category, so that I can understand where my money goes.

#### Acceptance Criteria

1. THE Pie_Chart SHALL display spending distribution grouped by Category, where each segment represents the sum of transaction amounts for one Category and each segment's angular size is proportional to that Category's share of total spending.
2. THE Pie_Chart SHALL exclude any Category whose summed transaction amount equals zero from the displayed segments.
3. WHEN a Transaction is added, THE Visualizer SHALL update the Pie_Chart to reflect the current spending distribution within 1 second.
4. WHEN a Transaction is deleted, THE Visualizer SHALL update the Pie_Chart to reflect the current spending distribution within 1 second.
5. WHILE no transactions are recorded, THE Pie_Chart SHALL hide all segments and display a text message indicating that no spending data exists.

### Requirement 6: Persist Data Client-Side

**User Story:** As a user, I want my expenses to remain saved between visits, so that I do not lose my data when I close the browser.

#### Acceptance Criteria

1. WHEN a Transaction is added, THE Visualizer SHALL write the current set of transactions to Storage within 500 milliseconds.
2. WHEN a Transaction is deleted, THE Visualizer SHALL write the current set of transactions to Storage within 500 milliseconds.
3. WHEN the Visualizer loads, THE Visualizer SHALL read the stored transactions from Storage and populate the Transaction_List, Balance_Display, and Pie_Chart from them.
4. IF the data read from Storage is absent or cannot be interpreted as a set of transactions when the Visualizer loads, THEN THE Visualizer SHALL initialize with an empty set of transactions and SHALL display the Transaction_List, Balance_Display, and Pie_Chart in their empty state.
5. IF writing the current set of transactions to Storage fails when a Transaction is added or deleted, THEN THE Visualizer SHALL retain the in-memory set of transactions and SHALL present a visible indication that the data could not be saved.
6. WHILE the number of stored transactions is at or below a maximum of 10,000 transactions, THE Visualizer SHALL persist and reload every transaction in the set without loss or truncation.

### Requirement 7: Presentation and Responsiveness

**User Story:** As a user, I want a clean and readable interface that responds quickly, so that the application is easy and pleasant to use.

#### Acceptance Criteria

1. THE Visualizer SHALL present the Balance_Display, Input_Form, Transaction_List, and Pie_Chart in top-to-bottom order with the Balance_Display first, using a minimum text size of 14px and a minimum text contrast ratio of 4.5:1.
2. WHEN a user adds or deletes a Transaction, THE Visualizer SHALL update the Transaction_List, Balance_Display, and Pie_Chart within 200 milliseconds.
3. THE Visualizer SHALL render its layout using a single CSS file and its behavior using a single JavaScript file.
4. WHERE the viewport width is between 320px and 1920px, THE Visualizer SHALL present its layout without horizontal scrolling or overlapping content.
