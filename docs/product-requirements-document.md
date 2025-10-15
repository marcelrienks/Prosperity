# Prosperity - Product Requirements Document (PRD)

**Version:** 1.0  
**Date:** October 15, 2025  
**Status:** Final  
**Author:** Marcel Rienks  
**Document Type:** Product Requirements Document

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Product Vision & Goals](#product-vision--goals)
3. [Success Criteria](#success-criteria)
4. [User Profile](#user-profile)
5. [Product Scope](#product-scope)
6. [Functional Requirements](#functional-requirements)
7. [Non-Functional Requirements](#non-functional-requirements)
8. [User Interface Requirements](#user-interface-requirements)
9. [Data Requirements](#data-requirements)
10. [Integration Requirements](#integration-requirements)
11. [Settings & Configuration](#settings--configuration)
12. [Edge Cases & Error Handling](#edge-cases--error-handling)
13. [Data Migration & Import/Export](#data-migration--importexport)
14. [Out of Scope](#out-of-scope)
15. [Assumptions & Constraints](#assumptions--constraints)
16. [Appendix](#appendix)

---

## Executive Summary

**Prosperity** is a personal stock portfolio management web application designed to replace an Excel-based investment tracking system. The primary purpose is to provide a modern, automated, and user-friendly interface for tracking stock investments across multiple accounts with real-time price updates, multi-currency support, and comprehensive performance analytics.

**Key Objectives:**
- Eliminate manual Excel data entry and formula maintenance
- Provide real-time stock price updates via Google Finance API
- Support multiple investment accounts (TFSA, US, ZA, EUR, Property)
- Calculate profit/loss, returns, and portfolio performance automatically
- Enable efficient portfolio management on both desktop and mobile devices
- Serve as a learning exercise for modern web technologies (Blazor WASM, Go, Azure, AWS)

**Technology Stack:**
- **Frontend:** Blazor WebAssembly hosted on AWS S3 + CloudFront
- **Backend:** Azure Functions (Go runtime)
- **Database:** Azure Cosmos DB (NoSQL)
- **Authentication:** JWT-based authentication
- **External APIs:** Google Finance API for stock prices, ExchangeRate-API for currency conversion

---

## Product Vision & Goals

### Product Vision
Prosperity is a **personal learning project** designed to modernize and simplify portfolio management by replacing manual Excel tracking with an automated, real-time web application. The focus is on functionality, ease of use, and exposure to modern cloud technologies.

### Primary Goals
1. **Functional Parity:** Replicate all capabilities of the current Excel-based system
2. **Efficiency:** Reduce time spent on portfolio tracking and manual data entry
3. **Automation:** Eliminate manual price lookups and calculation errors
4. **Learning:** Gain hands-on experience with Blazor WASM, Go, Azure, and hybrid cloud architecture
5. **Accessibility:** Enable portfolio management from any device (desktop primary, mobile secondary)

### Non-Goals
- Commercial product development or multi-user SaaS platform
- Automated broker integration or trading capabilities
- Social features or portfolio sharing
- Advanced financial modeling or investment advice
- Mobile native applications (web-only responsive design)

---

## Success Criteria

The MVP will be considered successful if it meets the following criteria:

### Functional Success Metrics
- ✅ Replicates 100% of current Excel functionality
- ✅ Supports all 5 existing accounts (ZA, TFSA, US, EUR, Prop)
- ✅ Provides real-time price updates for all 54+ stocks
- ✅ Calculates profit/loss automatically and accurately
- ✅ Handles multi-currency conversions (USD, EUR, GBP to ZAR)
- ✅ Displays summary dashboard with all key metrics
- ✅ Enables full CRUD operations for holdings, deposits, and transfers

### Performance Success Metrics
- ⚡ Faster than current Excel workflow (less time from open to insight)
- ⚡ Easier to use than Excel (fewer clicks, less manual entry)
- ⚡ Page loads in < 3 seconds (desktop)
- ⚡ Price refresh completes in < 10 seconds for all holdings

### Quality Success Metrics
- 🔒 Data security maintained (authentication, encryption)
- 📱 Fully responsive on desktop and mobile
- 💾 Reliable data persistence (no data loss)
- 📤 Bulk import/export capability for data portability

---

## User Profile

### Primary User
**Name:** Marcel Rienks  
**Role:** Individual Investor & Application Owner  
**Technical Proficiency:** High (software developer)  
**Financial Knowledge:** Intermediate investor with multi-account portfolio

### User Characteristics
- Manages personal investment portfolio across 5 accounts
- Currently tracks ~54 holdings manually in Excel
- Invests in South African (JSE), US (NYSE/NASDAQ), and European stocks
- Requires daily access to portfolio performance data
- Values efficiency, accuracy, and data control
- Comfortable with web applications and modern UX patterns

### Usage Patterns
- **Frequency:** Daily portfolio checks
- **Primary Device:** Desktop computer
- **Secondary Device:** Mobile phone (for quick checks)
- **Peak Usage:** After market close, during trading hours for price checks
- **Data Entry:** Ad-hoc when making new purchases or recording deposits/transfers

### User Needs
1. Quick access to total portfolio value and performance
2. Easy addition of new stock purchases
3. Real-time price updates without manual lookups
4. Clear visibility of profit/loss per holding and per account
5. Tracking of deposits, transfers, and available cash balances
6. Historical data preservation for long-term analysis
7. Bulk data import/export for backup and migration

---

## Product Scope

### In Scope (MVP)

#### Core Pages
1. **Summary Dashboard** - Portfolio overview across all accounts
2. **Investments Page** - Detailed holdings management with expandable accounts
3. **Deposits & Transfers Page** - Transaction history and cash flow tracking
4. **Settings Page** - Application configuration and customization

#### Core Features
- Manual stock entry (ticker, quantity, price, date)
- Real-time price updates via Google Finance API
- Multi-account organization (5 accounts: ZA, TFSA, US, EUR, Prop)
- Multi-currency support (USD, EUR, GBP, ZAR)
- Automatic profit/loss calculations
- Cash balance tracking per account
- Deposit recording (capital additions to accounts)
- Transfer tracking (money movement between accounts)
- Average cost basis calculation for multiple purchases of same stock
- Responsive design (desktop and mobile)
- Light/Dark theme support
- User authentication (email/password)
- Bulk data import/export (CSV format)
- Customizable dropdown options (exchange, industry, type, action, currency)

### Out of Scope (MVP)

#### Deferred to Post-MVP or Future
- Advanced charting and visualizations (beyond basic P/L display)
- Dividend tracking beyond account cash balance updates
- Price alerts and notifications
- Historical performance charts and trend analysis
- Tax reporting and SARS integration
- Automated dividend capture from external sources
- Portfolio optimization and rebalancing suggestions
- Broker API integration for automatic trade imports
- Two-factor authentication (2FA)
- Advanced filtering and search beyond basic account/ticker lookup
- Mobile native applications
- Offline mode / PWA capabilities
- Social sharing or multi-user features

---

## Functional Requirements

### FR-1: Summary Dashboard Page

**Purpose:** Provide a high-level overview of the entire portfolio at a glance.

#### FR-1.1: Portfolio Totals Display
**Description:** Display aggregated portfolio metrics across all accounts.

**Required Data Points:**
- **Deposited Amount:** Total capital deposited across all accounts (sum of all deposits minus transfers out, plus transfers in)
- **Invested Amount:** Total currently invested in holdings (sum of all holdings' total invested amounts)
- **Current Value:** Real-time total portfolio value (sum of all holdings' current values)
- **Total Cash Balance:** Uninvested cash across all accounts (sum of all account cash balances)
- **Total Balance:** Current Value + Total Cash Balance
- **Profit/Loss (Amount):** (Current Value - Invested Amount) in ZAR
- **Profit/Loss (%):** ((Current Value - Invested Amount) / Invested Amount) × 100

**Display Requirements:**
- All amounts displayed in ZAR (primary currency)
- Currency conversions applied automatically using current exchange rates
- Profit/loss displayed with color coding (green for positive, red for negative)
- Visual indicators (↑ / ↓ arrows) for performance direction
- Large, prominent display for total portfolio value
- Refresh button to manually update prices and recalculate

**Acceptance Criteria:**
- [ ] Dashboard loads in < 3 seconds
- [ ] All calculated values are accurate to 2 decimal places
- [ ] Profit/loss percentages match manual Excel calculations
- [ ] Color coding applies correctly based on positive/negative values
- [ ] Refresh button updates all prices and recalculates totals

#### FR-1.2: Account Summary Cards
**Description:** Display summary information for each investment account in card or table format.

**Required Data Per Account:**
- Account name (ZA, TFSA, US, EUR, Prop)
- Account type/description
- Deposited amount (total capital added to this account)
- Invested amount (sum of holdings' total invested)
- Current value (sum of holdings' current values)
- Cash balance (uninvested funds available)
- Total balance (current value + cash balance)
- Profit/loss amount and percentage
- Number of holdings in account

**Interaction:**
- Click on account card to navigate to Investments page filtered to that account
- Expand/collapse account details inline (optional)
- Visual distinction between accounts (color coding or icons)

**Acceptance Criteria:**
- [ ] All 5 accounts displayed on dashboard
- [ ] Each account shows accurate deposited, invested, and current value
- [ ] Cash balance reflects latest deposits/transfers/withdrawals
- [ ] Clicking account navigates to filtered Investments page
- [ ] Account metrics sum correctly to portfolio totals

#### FR-1.3: Last Updated Timestamp
**Description:** Display when prices were last refreshed.

**Requirements:**
- Show timestamp of last price update (e.g., "Last updated: 15:30, Oct 15, 2025")
- Update timestamp after manual or automatic refresh
- Display in user's local timezone

**Acceptance Criteria:**
- [ ] Timestamp visible on dashboard
- [ ] Timestamp updates after price refresh
- [ ] Timestamp format is clear and readable

---

### FR-2: Investments Page (Portfolio Management)

**Purpose:** Manage and view all investment holdings across accounts with detailed information and CRUD operations.

#### FR-2.1: Account-Based Organization
**Description:** Organize holdings by account with expandable/collapsible sections.

**Page Structure:**
```
Investments Page
├── Account Section: ZA (Expandable)
│   ├── Account Summary Bar (Deposited, Invested, Current Value, P/L)
│   └── Holdings Table (when expanded)
│       ├── Stock 1
│       ├── Stock 2
│       └── ...
├── Account Section: TFSA (Expandable)
├── Account Section: US (Expandable)
├── Account Section: EUR (Expandable)
└── Account Section: Prop (Expandable)
```

**Account Section (Collapsed State):**
- Account name/identifier
- Total deposited
- Total invested
- Current value
- Profit/loss (amount and %)
- Number of holdings
- Expand/collapse icon

**Account Section (Expanded State):**
- Same summary as collapsed
- Full holdings table visible below

**Acceptance Criteria:**
- [ ] All 5 accounts displayed with summary data
- [ ] Click to expand/collapse account sections
- [ ] Collapsed by default on page load
- [ ] Account summaries match dashboard data
- [ ] Smooth expand/collapse animation

#### FR-2.2: Holdings Table
**Description:** Display detailed information for each stock holding within an account.

**Required Columns:**
1. **Ticker/Code** - Stock ticker symbol (e.g., AAPL, ETFGLD, ANH)
2. **Exchange** - Trading exchange (JSE, NYSE, NASDAQ, LSE, XETRA, etc.) - dropdown configurable in Settings
3. **Industry** - Sector/industry classification (e.g., Technology, Banks, Beverages, ETF) - dropdown configurable in Settings
4. **Type** - Investment classification (Investment, Speculation, Undervalued, Growth, etc.) - dropdown configurable in Settings
5. **Action** - Current recommendation (Hold, Buy, Sell, Watch) - dropdown configurable in Settings
6. **Quantity** - Number of shares owned
7. **Purchase Price** - Average cost per share (calculated for multiple purchases)
8. **Total Invested** - Total capital invested (Quantity × Purchase Price)
9. **Current Price** - Real-time market price (from Google Finance API)
10. **Current Value** - Quantity × Current Price
11. **Profit/Loss (Amount)** - Current Value - Total Invested
12. **Profit/Loss (%)** - ((Current Value - Total Invested) / Total Invested) × 100
13. **Currency** - Original currency of stock (USD, EUR, ZAR, GBP) - dropdown configurable in Settings
14. **Actions** - Edit and Delete buttons

**Display Requirements:**
- Values displayed in original currency (not converted to ZAR)
- Profit/loss color-coded (green positive, red negative)
- Sortable columns (by ticker, value, P/L, etc.)
- Responsive table layout (stacks on mobile)
- Current price updates on page load and manual refresh

**Acceptance Criteria:**
- [ ] All holdings displayed with complete data
- [ ] Current prices fetch successfully from Google Finance API
- [ ] Profit/loss calculations are accurate
- [ ] Color coding applied correctly
- [ ] Table is sortable by all numeric columns
- [ ] Responsive design works on mobile screens

#### FR-2.3: Add New Holding
**Description:** Form to add a new stock purchase to an account.

**Required Form Fields:**
1. **Account** - Dropdown selection (ZA, TFSA, US, EUR, Prop)
2. **Ticker/Code** - Text input (e.g., AAPL, GOOGL)
3. **Exchange** - Dropdown (values from Settings - customizable)
4. **Industry** - Dropdown (values from Settings - customizable)
5. **Type** - Dropdown (values from Settings - customizable)
6. **Action** - Dropdown (values from Settings - customizable)
7. **Quantity** - Numeric input (decimal allowed for fractional shares)
8. **Purchase Price** - Numeric input (decimal, 2+ places)
9. **Purchase Date** - Date picker
10. **Currency** - Dropdown (values from Settings - customizable)

**Behavior:**
- Form accessible via "Add Holding" button on Investments page
- All fields required except Action (optional)
- Validate ticker symbol format (1-5 uppercase characters)
- Validate numeric inputs (positive numbers only)
- Fetch current price on form submission to populate initial current value
- Calculate Total Invested automatically (Quantity × Purchase Price)

**On Submit:**
1. Validate all inputs
2. Fetch current price from Google Finance API
3. Calculate initial P/L (likely $0 if purchased today)
4. Save holding to database
5. Update account totals
6. Refresh Investments page to show new holding
7. Display success message

**Acceptance Criteria:**
- [ ] Form accessible from Investments page
- [ ] All required fields validated before submission
- [ ] Ticker symbol validated for proper format
- [ ] Current price fetched successfully on submission
- [ ] New holding appears immediately in holdings table
- [ ] Account totals update correctly
- [ ] Error messages display for invalid inputs

#### FR-2.4: Edit Existing Holding
**Description:** Modify details of an existing stock holding.

**Behavior:**
- Edit button in holdings table opens edit form
- Pre-populate form with current holding data
- Allow updates to all fields except original purchase date (optional restriction)
- **Multiple Purchases Handling:** If user purchases more of the same stock:
  - **Average Cost Basis:** Recalculate average purchase price
  - Formula: `New Avg Price = ((Existing Quantity × Existing Price) + (New Quantity × New Price)) / (Existing Quantity + New Quantity)`
  - Update Total Invested accordingly
  - Maintain single holding row (do NOT create duplicate entries)

**On Submit:**
1. Validate inputs
2. If quantity or price changed, recalculate averages
3. Fetch latest current price
4. Recalculate P/L
5. Update database
6. Refresh holdings table
7. Display success message

**Acceptance Criteria:**
- [ ] Edit form pre-populated with existing data
- [ ] Average cost basis calculated correctly for additional purchases
- [ ] Updated holding displays immediately
- [ ] Account totals recalculated
- [ ] Error handling for invalid inputs

#### FR-2.5: Delete Holding
**Description:** Remove a stock holding from the portfolio.

**Behavior:**
- Delete button in holdings table row
- Confirmation dialog: "Are you sure you want to delete [TICKER]?"
- On confirmation, remove holding from database
- Update account totals
- Refresh holdings table
- Display success message

**Acceptance Criteria:**
- [ ] Confirmation dialog prevents accidental deletion
- [ ] Holding removed from database on confirmation
- [ ] Holdings table refreshes immediately
- [ ] Account totals update correctly
- [ ] Success message displayed

#### FR-2.6: Price Refresh
**Description:** Manually refresh stock prices for all holdings.

**Behavior:**
- "Refresh Prices" button visible on Investments page
- On click:
  1. Display loading indicator
  2. Fetch current prices for all unique ticker/exchange combinations from Google Finance API
  3. Update current price for each holding
  4. Recalculate current value and P/L for each holding
  5. Update last updated timestamp
  6. Refresh page display
  7. Hide loading indicator
  8. Display success message with number of prices updated

**Error Handling:**
- If Google Finance API fails for a specific stock:
  - Display error icon next to that stock's price
  - Show tooltip: "Unable to fetch price for [TICKER]"
  - Do NOT update that stock's price (keep previous value)
  - Continue updating other stocks
- If API completely unavailable:
  - Display global error message: "Unable to refresh prices. Please try again later."
  - Keep all previous prices

**Acceptance Criteria:**
- [ ] Refresh button accessible and functional
- [ ] Loading indicator displays during fetch
- [ ] All stock prices update successfully
- [ ] Current values and P/L recalculated
- [ ] Timestamp updates after refresh
- [ ] Error handling graceful for API failures
- [ ] Partial failures don't block other price updates

---

### FR-3: Deposits & Transfers Page

**Purpose:** Track capital deposits into accounts and transfers between accounts to maintain accurate portfolio funding history and available cash balances.

#### FR-3.1: Deposits & Transfers Table
**Description:** Display all deposit and transfer transactions in chronological order.

**Required Columns:**
1. **Date** - Transaction date
2. **Type** - "Deposit" or "Transfer"
3. **Account** - Source account (for Deposits and Transfers Out) or Destination account (for Transfers In)
4. **From Account** - Source account (for Transfers only)
5. **To Account** - Destination account (for Transfers only)
6. **Amount** - Transaction amount (always positive value)
7. **Currency** - Transaction currency
8. **Description/Notes** - Optional text description
9. **Actions** - Edit and Delete buttons

**Display Requirements:**
- Sorted by date (most recent first by default)
- Deposits shown with single account
- Transfers shown with From → To account flow
- Color coding: Deposits (blue), Transfers (orange)
- Currency displayed with amount
- Responsive table (stacks on mobile)

**Calculated Totals (displayed above or below table):**
- Total Deposits (all accounts)
- Total Transfers (net should be $0 across all accounts)
- Net Cash Flow by Account

**Acceptance Criteria:**
- [ ] All deposits and transfers displayed
- [ ] Sorted chronologically (newest first)
- [ ] Type clearly distinguished (Deposit vs Transfer)
- [ ] Transfer flow (From → To) clearly visible
- [ ] Totals calculated correctly
- [ ] Responsive design on mobile

#### FR-3.2: Add Deposit
**Description:** Record a capital deposit into a specific account.

**Required Form Fields:**
1. **Date** - Date picker (default: today)
2. **Account** - Dropdown (ZA, TFSA, US, EUR, Prop)
3. **Amount** - Numeric input (positive only)
4. **Currency** - Dropdown (values from Settings)
5. **Description** - Text input (optional)

**Behavior:**
- "Add Deposit" button opens form
- Validate amount > 0
- On submit:
  1. Save deposit to database
  2. **Increase account's Deposited Total by amount**
  3. **Increase account's Cash Balance by amount**
  4. Refresh Deposits & Transfers table
  5. Update Summary Dashboard totals
  6. Display success message

**Impact on Account:**
- `Account.Deposited += Deposit.Amount`
- `Account.CashBalance += Deposit.Amount`

**Acceptance Criteria:**
- [ ] Form accessible from Deposits & Transfers page
- [ ] All fields validated
- [ ] Deposit saved to database
- [ ] Account deposited total increases correctly
- [ ] Account cash balance increases correctly
- [ ] Dashboard reflects updated totals
- [ ] Success message displayed

#### FR-3.3: Add Transfer
**Description:** Record a transfer of funds from one account to another.

**Required Form Fields:**
1. **Date** - Date picker (default: today)
2. **From Account** - Dropdown (ZA, TFSA, US, EUR, Prop)
3. **To Account** - Dropdown (ZA, TFSA, US, EUR, Prop) - must be different from "From Account"
4. **Amount** - Numeric input (positive only)
5. **Currency** - Dropdown (values from Settings)
6. **Description** - Text input (optional)

**Behavior:**
- "Add Transfer" button opens form
- Validate amount > 0
- Validate From Account ≠ To Account
- On submit:
  1. Save transfer to database
  2. **Decrease From Account's Deposited Total by amount**
  3. **Decrease From Account's Cash Balance by amount**
  4. **Increase To Account's Deposited Total by amount**
  5. **Increase To Account's Cash Balance by amount**
  6. Refresh Deposits & Transfers table
  7. Update Summary Dashboard
  8. Display success message

**Impact on Accounts:**
- From Account:
  - `FromAccount.Deposited -= Transfer.Amount`
  - `FromAccount.CashBalance -= Transfer.Amount`
- To Account:
  - `ToAccount.Deposited += Transfer.Amount`
  - `ToAccount.CashBalance += Transfer.Amount`

**Validation:**
- From Account must have sufficient cash balance (CashBalance >= Transfer.Amount)
- Display error if insufficient funds: "Insufficient cash balance in [From Account]. Available: [CashBalance]"

**Acceptance Criteria:**
- [ ] Form accessible from Deposits & Transfers page
- [ ] From/To accounts must be different
- [ ] Amount validated against From Account cash balance
- [ ] Transfer saved to database
- [ ] From Account deposited and cash decrease correctly
- [ ] To Account deposited and cash increase correctly
- [ ] Dashboard reflects updated totals
- [ ] Error message for insufficient funds
- [ ] Success message displayed

#### FR-3.4: Edit Deposit/Transfer
**Description:** Modify an existing deposit or transfer transaction.

**Behavior:**
- Edit button opens pre-populated form
- Allow updates to date, amount, description
- Account changes allowed with caution (recalculate impact)
- On submit:
  1. Reverse original transaction impact on accounts
  2. Apply updated transaction impact
  3. Update database
  4. Refresh table and dashboard
  5. Display success message

**Acceptance Criteria:**
- [ ] Edit form pre-populated with existing data
- [ ] Account totals recalculated correctly after edit
- [ ] Cash balances updated accurately
- [ ] Success message displayed

#### FR-3.5: Delete Deposit/Transfer
**Description:** Remove a deposit or transfer transaction.

**Behavior:**
- Delete button with confirmation dialog
- On confirmation:
  1. Reverse transaction impact on account(s)
  2. Remove from database
  3. Refresh table and dashboard
  4. Display success message

**Impact on Accounts:**
- Deposit deletion: Decrease Deposited Total and Cash Balance
- Transfer deletion: Reverse both From and To account impacts

**Acceptance Criteria:**
- [ ] Confirmation dialog prevents accidental deletion
- [ ] Transaction removed from database
- [ ] Account totals and cash balances reversed correctly
- [ ] Dashboard updated
- [ ] Success message displayed

#### FR-3.6: Transfers History View
**Description:** Separate view to filter and display only transfers between accounts.

**Behavior:**
- Toggle or tab to switch between "All Transactions" and "Transfers Only"
- Transfers view shows only transfers (excludes deposits)
- Displays From → To flow clearly
- Shows net impact per account

**Acceptance Criteria:**
- [ ] Toggle accessible on Deposits & Transfers page
- [ ] Transfers view filters correctly
- [ ] Transfer flow clearly visible
- [ ] Net transfer impact calculated per account

---

### FR-4: Cash Balance Management

**Purpose:** Maintain accurate available cash balance per account, updated by deposits, transfers, and stock purchases.

#### FR-4.1: Cash Balance Tracking
**Description:** Each account maintains a cash balance representing uninvested funds.

**Cash Balance Rules:**
- **Initial State:** Cash Balance = $0 (unless migrated from existing data)
- **Deposits:** Cash Balance += Deposit Amount
- **Transfers In:** Cash Balance += Transfer Amount
- **Transfers Out:** Cash Balance -= Transfer Amount
- **Stock Purchases:** When adding or editing a holding with increased investment, optionally decrease Cash Balance by the additional investment (user preference in Settings)
- **Dividends:** Cash Balance += Dividend Amount (manual entry as deposit or automatic if configured)

**Display Requirements:**
- Cash balance displayed on Summary Dashboard per account
- Cash balance displayed in Investments page account summary
- Cash balance editable directly (manual adjustment with description)

**Acceptance Criteria:**
- [ ] Cash balance starts at $0 for new accounts
- [ ] Deposits increase cash balance correctly
- [ ] Transfers update both accounts correctly
- [ ] Stock purchases optionally decrease cash balance
- [ ] Cash balance visible on Dashboard and Investments page
- [ ] Manual adjustments allowed with audit trail

#### FR-4.2: Manual Cash Adjustment
**Description:** Allow direct adjustment of cash balance (for dividends, interest, corrections).

**Behavior:**
- "Adjust Cash" button on account summary
- Form fields:
  - Account
  - Adjustment Amount (positive or negative)
  - Description/Reason
  - Date
- On submit:
  - Update Cash Balance
  - Log adjustment in transaction history
  - Display success message

**Acceptance Criteria:**
- [ ] Manual adjustment form accessible
- [ ] Cash balance updates correctly
- [ ] Adjustment logged with description
- [ ] Visible in transaction history

---

### FR-5: Settings Page

**Purpose:** Configure application preferences, customize dropdown options, and manage user account settings.

#### FR-5.1: Theme Settings
**Description:** Allow user to select Light or Dark theme.

**Options:**
- Light Mode (default)
- Dark Mode
- Auto (follow system preference) - optional future enhancement

**Behavior:**
- Radio buttons or toggle switch
- Apply theme immediately on selection
- Persist theme preference to user profile or local storage
- Theme applies across all pages

**Acceptance Criteria:**
- [ ] Light/Dark theme toggle accessible
- [ ] Theme changes apply immediately
- [ ] Theme preference persists across sessions
- [ ] All pages styled consistently in selected theme

#### FR-5.2: Customizable Dropdown Options
**Description:** Manage dropdown values for Exchange, Industry, Type, Action, and Currency fields used in holdings management.

**Editable Dropdown Categories:**
1. **Exchanges** (e.g., JSE, NYSE, NASDAQ, LSE, XETRA)
2. **Industries** (e.g., Technology, Banks, Beverages, Mining, ETF, Property)
3. **Types** (e.g., Investment, Speculation, Undervalued, Growth)
4. **Actions** (e.g., Hold, Buy, Sell, Watch)
5. **Currencies** (e.g., ZAR, USD, EUR, GBP)

**Management Interface:**
- Display existing values in a list
- "Add New" button to add custom values
- "Edit" button to rename existing values
- "Delete" button to remove values (with confirmation and check for usage)
- Drag-and-drop to reorder values (optional)

**Default Values (Pre-populated):**

**Exchanges:**
- JSE (Johannesburg Stock Exchange)
- NYSE (New York Stock Exchange)
- NASDAQ
- LSE (London Stock Exchange)
- XETRA (Deutsche Börse)

**Industries:**
- Technology
- Banks
- Beverages
- Mining
- ETF
- Property
- Healthcare
- Consumer Goods
- Energy
- Telecommunications

**Types:**
- Investment
- Speculation
- Undervalued
- Growth

**Actions:**
- Hold
- Buy
- Sell
- Watch

**Currencies:**
- ZAR (South African Rand)
- USD (US Dollar)
- EUR (Euro)
- GBP (British Pound)

**Behavior:**
- Changes save immediately or on "Save Settings" button
- Deleted values: Check if used by any holdings; if yes, display warning and prevent deletion or reassign
- Added values appear in dropdowns immediately

**Acceptance Criteria:**
- [ ] All dropdown categories editable
- [ ] Default values pre-populated
- [ ] Add, edit, delete operations work correctly
- [ ] Deletion prevented if value in use (with warning message)
- [ ] New values appear in forms immediately
- [ ] Changes persist across sessions

#### FR-5.3: User Profile Settings
**Description:** Manage user account information.

**Editable Fields:**
- Name
- Email address
- Password (change password functionality)

**Behavior:**
- Display current name and email
- "Change Password" button opens password change form
  - Current Password (validation)
  - New Password (min 8 characters, complexity requirements)
  - Confirm New Password
- Save changes with confirmation

**Acceptance Criteria:**
- [ ] User profile editable
- [ ] Password change requires current password
- [ ] New password meets complexity requirements
- [ ] Success message on save

#### FR-5.4: Data Management Settings
**Description:** Configure data-related preferences.

**Options:**
- **Auto-refresh prices on page load:** On/Off toggle
- **Default sort order:** Dropdown (e.g., by ticker, by P/L, by value)
- **Date format preference:** Dropdown (e.g., MM/DD/YYYY, DD/MM/YYYY, YYYY-MM-DD)
- **Number format:** Decimal places (0, 2, 4)

**Acceptance Criteria:**
- [ ] Preferences configurable
- [ ] Settings apply globally
- [ ] Preferences persist across sessions

#### FR-5.5: About & Help
**Description:** Application information and version.

**Display:**
- Application name: Prosperity
- Version number
- Last updated date
- Link to documentation (optional)
- Contact/support information (optional)

---

### FR-6: User Authentication

**Purpose:** Secure access to the application with email/password authentication.

#### FR-6.1: User Registration
**Description:** Allow new users to create an account.

**Form Fields:**
- First Name
- Last Name
- Email address (unique)
- Password (min 8 characters, complexity requirements)
- Confirm Password

**Behavior:**
- Validate email uniqueness
- Hash password (bcrypt)
- Create user record in database
- Automatically log in user after registration
- Redirect to Dashboard

**Acceptance Criteria:**
- [ ] Registration form accessible
- [ ] Email uniqueness validated
- [ ] Password complexity enforced
- [ ] User created in database
- [ ] Automatic login after registration

#### FR-6.2: User Login
**Description:** Authenticate existing users.

**Form Fields:**
- Email address
- Password
- "Remember Me" checkbox (optional)

**Behavior:**
- Validate credentials against database
- Generate JWT token on success
- Store token in session/local storage
- Redirect to Dashboard
- Display error for invalid credentials

**Acceptance Criteria:**
- [ ] Login form accessible
- [ ] Valid credentials authenticate successfully
- [ ] JWT token generated and stored
- [ ] User redirected to Dashboard
- [ ] Error message for invalid credentials

#### FR-6.3: User Logout
**Description:** End user session and clear authentication.

**Behavior:**
- "Logout" button in navigation or user menu
- Clear JWT token from storage
- Redirect to login page
- Display logout confirmation message

**Acceptance Criteria:**
- [ ] Logout button accessible
- [ ] Token cleared from storage
- [ ] User redirected to login page
- [ ] Cannot access protected pages after logout

#### FR-6.4: Password Reset
**Description:** Allow users to reset forgotten passwords.

**Behavior:**
- "Forgot Password" link on login page
- Enter email address
- Send password reset email (or display reset link if email not configured)
- Reset link expires after 1 hour
- User enters new password
- Password updated in database
- Redirect to login page

**Acceptance Criteria:**
- [ ] Password reset link accessible
- [ ] Reset email sent (or link displayed)
- [ ] Reset link expires after 1 hour
- [ ] New password saved successfully
- [ ] User can log in with new password

---

## Non-Functional Requirements

### NFR-1: Performance

**Load Time:**
- Summary Dashboard: < 3 seconds on desktop
- Investments Page: < 3 seconds (collapsed accounts)
- Expanded account: < 1 second
- Deposits & Transfers Page: < 2 seconds

**Price Refresh:**
- All stocks (54+): < 10 seconds total
- Single stock: < 1 second
- Cache prices for 15 minutes to minimize API calls

**Responsiveness:**
- UI interactions: < 100ms response time
- Form submissions: < 500ms feedback

**Acceptance Criteria:**
- [ ] Page load times meet targets on standard broadband
- [ ] Price refresh completes within 10 seconds
- [ ] UI feels responsive and snappy

---

### NFR-2: Scalability

**Current Scale:**
- Single user
- 5 accounts
- ~50-100 holdings
- ~500 transactions/year

**Future Considerations:**
- Architecture supports multiple users (partitioned by userId)
- Database can scale to thousands of holdings
- API rate limits managed with caching

**Acceptance Criteria:**
- [ ] Application handles 100+ holdings without performance degradation
- [ ] Database queries optimized for 1000+ records

---

### NFR-3: Reliability

**Uptime:**
- No specific SLA (personal use)
- Acceptable downtime: Several hours per month for maintenance

**Data Integrity:**
- No data loss during normal operations
- Automatic backups (daily)
- Export functionality for manual backups

**Error Recovery:**
- Graceful degradation if external APIs fail
- User-friendly error messages
- Retry logic for transient failures

**Acceptance Criteria:**
- [ ] No data loss during testing
- [ ] Export/import maintains data integrity
- [ ] Error messages clear and actionable

---

### NFR-4: Security

**Authentication:**
- JWT-based authentication
- Password hashing (bcrypt, cost factor 12)
- Session timeout after 7 days inactivity

**Data Encryption:**
- HTTPS for all communications (TLS 1.3)
- Cosmos DB encryption at rest (automatic)

**Authorization:**
- All data scoped by userId
- No cross-user data access

**Secrets Management:**
- API keys stored in Azure Key Vault
- Database connection strings in environment variables
- No secrets in source code

**Acceptance Criteria:**
- [ ] Passwords hashed securely
- [ ] HTTPS enforced on all endpoints
- [ ] User data isolated by userId
- [ ] Secrets not exposed in code or logs

---

### NFR-5: Usability

**Ease of Use:**
- Intuitive navigation
- Minimal clicks to common tasks
- Clear labels and instructions
- Consistent UI patterns

**Accessibility:**
- Keyboard navigation support
- Semantic HTML
- Adequate color contrast (WCAG AA)
- Screen reader friendly (future enhancement)

**Learning Curve:**
- New user can add first holding in < 2 minutes
- Dashboard understandable without training

**Acceptance Criteria:**
- [ ] Navigation is intuitive
- [ ] Common tasks require minimal clicks
- [ ] Color contrast meets WCAG AA standards
- [ ] Keyboard navigation functional

---

### NFR-6: Compatibility

**Browser Support:**
- Chrome (latest)
- Edge (latest)
- Firefox (latest)
- Safari (latest)

**Device Support:**
- Desktop: Windows, macOS, Linux
- Mobile: iOS (Safari), Android (Chrome)

**Screen Sizes:**
- Desktop: 1920×1080 and above (primary)
- Laptop: 1366×768 and above
- Tablet: 768×1024 (iPad)
- Mobile: 375×667 and above (iPhone SE)

**Acceptance Criteria:**
- [ ] Application functional on all listed browsers
- [ ] Responsive design works on all screen sizes
- [ ] Touch interactions work on mobile devices

---

### NFR-7: Maintainability

**Code Quality:**
- Clean, readable code
- Consistent naming conventions
- Commented where necessary
- Modular architecture

**Documentation:**
- Architecture documentation (completed)
- API documentation
- Setup and deployment guides
- Inline code comments

**Testing:**
- Unit tests for critical business logic
- Integration tests for API endpoints
- Manual testing for UI flows

**Acceptance Criteria:**
- [ ] Code follows established conventions
- [ ] Documentation is complete and up-to-date
- [ ] Tests cover critical functionality

---

## User Interface Requirements

### UI-1: Navigation

**Primary Navigation:**
- Top navigation bar (horizontal)
- Logo/App name (links to Dashboard)
- Menu items:
  - Dashboard
  - Investments
  - Deposits & Transfers
  - Settings
- User menu (right-aligned):
  - User name/email
  - Logout button

**Mobile Navigation:**
- Hamburger menu icon
- Slide-out or dropdown menu
- Same menu items as desktop

**Acceptance Criteria:**
- [ ] Navigation accessible on all pages
- [ ] Active page highlighted
- [ ] Mobile menu functional and intuitive

---

### UI-2: Layout & Spacing

**General:**
- Consistent padding and margins
- White space for readability
- Responsive grid system
- Card-based layout for summary sections

**Typography:**
- Clear hierarchy (headings, body text)
- Readable font sizes (min 14px body text)
- Sans-serif font (e.g., Inter, Roboto, Open Sans)

**Acceptance Criteria:**
- [ ] Layout is clean and uncluttered
- [ ] Typography is readable
- [ ] Responsive on all devices

---

### UI-3: Color Scheme

**Light Theme (Default):**
- Primary: TBD (branded color)
- Background: White or light gray
- Text: Dark gray or black
- Positive (profit): Green (#28a745)
- Negative (loss): Red (#dc3545)
- Neutral: Gray

**Dark Theme:**
- Primary: TBD (branded color)
- Background: Dark gray or black
- Text: White or light gray
- Positive: Green (adjusted for contrast)
- Negative: Red (adjusted for contrast)
- Neutral: Gray

**Acceptance Criteria:**
- [ ] Color scheme consistent across pages
- [ ] Sufficient contrast in both themes
- [ ] Profit/loss colors distinct and intuitive

---

### UI-4: Interactive Elements

**Buttons:**
- Clear labels (e.g., "Add Holding," "Refresh Prices")
- Consistent styling (primary, secondary, danger)
- Hover and active states
- Loading states (spinners during async operations)

**Forms:**
- Clear labels and placeholders
- Validation feedback (inline errors)
- Required field indicators (asterisks)
- Submit and cancel buttons

**Tables:**
- Sortable columns (click header to sort)
- Alternating row colors for readability
- Hover state on rows
- Responsive (horizontal scroll on mobile)

**Modals/Dialogs:**
- Centered on screen
- Overlay background (dim)
- Close button (X icon)
- Confirmation dialogs for destructive actions

**Acceptance Criteria:**
- [ ] Buttons have clear hover states
- [ ] Forms provide validation feedback
- [ ] Tables are sortable and readable
- [ ] Modals are functional and accessible

---

### UI-5: Responsive Design

**Breakpoints:**
- Mobile: < 768px
- Tablet: 768px - 1024px
- Desktop: > 1024px

**Mobile Adaptations:**
- Stack layouts vertically
- Collapse navigation to hamburger menu
- Expand/collapse sections for space saving
- Simplified tables (cards or stacked rows)
- Touch-friendly buttons (min 44×44px)

**Acceptance Criteria:**
- [ ] Layout adapts at defined breakpoints
- [ ] Mobile UI is touch-friendly
- [ ] No horizontal scrolling on mobile (except tables)
- [ ] All functionality accessible on mobile

---

## Data Requirements

### DR-1: Data Model

#### Entities

**User**
- `id` (UUID)
- `email` (unique, indexed)
- `passwordHash` (bcrypt)
- `firstName`
- `lastName`
- `createdAt` (timestamp)
- `lastLogin` (timestamp)
- `preferences` (JSON: theme, dateFormat, etc.)

**Account**
- `id` (UUID)
- `userId` (foreign key, indexed)
- `name` (e.g., "ZA", "TFSA", "US", "EUR", "Prop")
- `type` (e.g., "TFSA", "Taxable", "RRSP")
- `description` (optional)
- `currency` (default currency for account, e.g., "ZAR")
- `cashBalance` (decimal, current uninvested cash)
- `deposited` (decimal, total capital deposited)
- `createdAt` (timestamp)
- `updatedAt` (timestamp)

**Holding**
- `id` (UUID)
- `userId` (foreign key, indexed)
- `accountId` (foreign key, indexed)
- `ticker` (stock symbol, e.g., "AAPL")
- `exchange` (e.g., "NYSE", "JSE")
- `industry` (e.g., "Technology", "Banks")
- `type` (e.g., "Investment", "Speculation")
- `action` (e.g., "Hold", "Buy", "Sell")
- `quantity` (decimal, shares owned)
- `purchasePrice` (decimal, average cost per share)
- `totalInvested` (decimal, quantity × purchasePrice)
- `currentPrice` (decimal, latest market price)
- `currentValue` (decimal, quantity × currentPrice)
- `profitLoss` (decimal, currentValue - totalInvested)
- `profitLossPercent` (decimal, calculated)
- `currency` (e.g., "USD", "ZAR")
- `lastPriceUpdate` (timestamp)
- `createdAt` (timestamp)
- `updatedAt` (timestamp)

**Transaction** (Deposits & Transfers)
- `id` (UUID)
- `userId` (foreign key, indexed)
- `type` ("Deposit" or "Transfer")
- `date` (transaction date)
- `accountId` (for deposits, or "To Account" for transfers)
- `fromAccountId` (for transfers only, nullable)
- `toAccountId` (for transfers only, nullable)
- `amount` (decimal, always positive)
- `currency` (e.g., "ZAR", "USD")
- `description` (optional text)
- `createdAt` (timestamp)

**Settings** (User Preferences)
- Stored in `User.preferences` JSON field or separate Settings table
- Theme (light/dark)
- Auto-refresh prices (on/off)
- Default sort order
- Date format
- Number format (decimal places)

**Dropdown Options** (Customizable)
- Stored as JSON in User preferences or separate tables
- Exchanges (array of strings)
- Industries (array of strings)
- Types (array of strings)
- Actions (array of strings)
- Currencies (array of strings)

---

### DR-2: Data Validation Rules

**Ticker Symbol:**
- 1-5 uppercase characters
- Alphanumeric only

**Quantity:**
- Positive number
- Decimal allowed (fractional shares)

**Purchase Price:**
- Positive number
- Decimal (2+ places)

**Amount (Deposits/Transfers):**
- Positive number
- Decimal (2+ places)

**Email:**
- Valid email format
- Unique in database

**Password:**
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character (optional)

---

### DR-3: Data Retention

**User Data:**
- Retained indefinitely (personal application)
- User can export and delete account

**Holdings:**
- Retained indefinitely
- Historical holdings maintained (soft delete option)

**Transactions:**
- Retained indefinitely for historical analysis

**Price History:**
- Optionally store historical prices (future enhancement)
- Current implementation: store only latest price

---

## Integration Requirements

### INT-1: Google Finance API

**Purpose:** Fetch real-time stock prices for all holdings.

**API Details:**
- **Endpoint:** Google Finance API (or equivalent - e.g., Yahoo Finance, Alpha Vantage)
- **Rate Limits:** Check API documentation (Google Finance typically allows reasonable usage)
- **Authentication:** API key (if required)

**Usage:**
- Fetch price by ticker symbol and exchange
- Example: `AAPL` on `NASDAQ`, `ANH` on `JSE`
- Parse response for current price
- Handle errors gracefully (stock not found, API unavailable)

**Caching Strategy:**
- Cache prices for 15 minutes
- Reduce redundant API calls
- Update cache on manual refresh

**Error Handling:**
- If API fails for a specific stock:
  - Log error
  - Display error icon next to stock
  - Keep previous price
  - Continue fetching other stocks
- If API completely unavailable:
  - Display global error message
  - Keep all previous prices
  - Suggest manual entry or retry

**Acceptance Criteria:**
- [ ] Prices fetched successfully for all supported exchanges
- [ ] Errors handled gracefully
- [ ] Caching reduces API calls
- [ ] Manual refresh bypasses cache

---

### INT-2: ExchangeRate-API (Currency Conversion)

**Purpose:** Fetch current exchange rates for multi-currency portfolio calculations.

**API Details:**
- **Endpoint:** ExchangeRate-API.com or European Central Bank API
- **Rate Limits:** 1,500 requests/month (free tier)
- **Authentication:** API key (if required)

**Usage:**
- Fetch exchange rates relative to ZAR (base currency)
- Example: USD/ZAR, EUR/ZAR, GBP/ZAR
- Cache rates for 24 hours (exchange rates change daily)
- Update on manual refresh or daily

**Error Handling:**
- If API fails:
  - Use cached rates (with warning)
  - Display message: "Using exchange rates from [date]"
  - Allow manual entry of rates (Settings option - future enhancement)

**Acceptance Criteria:**
- [ ] Exchange rates fetched successfully
- [ ] Rates cached for 24 hours
- [ ] Errors handled with fallback to cached rates
- [ ] Currency conversions accurate

---

## Settings & Configuration

### SETT-1: Application Configuration

**Environment Variables (Azure Functions App Settings):**
- `COSMOS_DB_ENDPOINT` - Cosmos DB connection endpoint
- `COSMOS_DB_KEY` - Cosmos DB access key
- `JWT_SECRET_KEY` - Secret key for JWT signing
- `GOOGLE_FINANCE_API_KEY` - Google Finance API key (if required)
- `EXCHANGE_RATE_API_KEY` - ExchangeRate-API key
- `ALLOWED_ORIGINS` - CORS allowed origins (frontend URL)
- `TOKEN_EXPIRY_DAYS` - JWT expiration (default: 7 days)

**Frontend Configuration (Blazor):**
- `API_BASE_URL` - Azure Functions API base URL (e.g., `https://func-prosperity-api.azurewebsites.net/api`)

---

### SETT-2: User Preferences

**Stored in User Profile:**
- Theme (light/dark)
- Auto-refresh prices on page load (on/off)
- Default sort order (ticker, P/L, value)
- Date format (MM/DD/YYYY, DD/MM/YYYY, YYYY-MM-DD)
- Number format (decimal places: 0, 2, 4)

**Stored in User Settings (Customizable Dropdowns):**
- Exchanges list
- Industries list
- Types list
- Actions list
- Currencies list

---

## Edge Cases & Error Handling

### EC-1: Stock Delisted

**Scenario:** A stock is delisted from the exchange and no longer has a price.

**Handling:**
- Manual removal by user (Delete Holding)
- Display warning if price fetch fails repeatedly
- Allow user to mark holding as "Inactive" (future enhancement)
- Keep historical data for record-keeping

**Acceptance Criteria:**
- [ ] User can delete delisted stock manually
- [ ] Error message if price unavailable
- [ ] Historical data preserved

---

### EC-2: Stock Splits / Reverse Splits

**Scenario:** A company performs a stock split (e.g., 2-for-1) or reverse split.

**Handling:**
- **Manual adjustment required:**
  - User edits holding to update quantity and purchase price
  - Example: 2-for-1 split: Double quantity, halve purchase price
- Future enhancement: Automatic split detection and adjustment

**Acceptance Criteria:**
- [ ] User can manually adjust quantity and price
- [ ] Average cost basis recalculates correctly

---

### EC-3: External API Downtime

**Scenario:** Google Finance API or ExchangeRate-API is unavailable.

**Handling:**

**Price API Down:**
- Display error message: "Unable to refresh prices. Please try again later."
- Keep previous prices visible
- Display last updated timestamp
- Allow user to manually enter prices (future enhancement)

**Exchange Rate API Down:**
- Use cached exchange rates
- Display warning: "Using exchange rates from [date]. Conversions may not reflect current rates."
- Allow manual refresh retry

**Acceptance Criteria:**
- [ ] Error messages displayed clearly
- [ ] Previous data remains visible
- [ ] User can retry refresh
- [ ] Cached data used as fallback

---

### EC-4: Data Entry Errors

**Scenario:** User enters incorrect data (e.g., wrong quantity, price, or ticker).

**Handling:**
- **Edit functionality:** User can edit any holding at any time
- **Delete functionality:** User can delete and re-add if needed
- **No undo/history:** Simplicity prioritized; manual correction required
- **Validation on input:** Prevent obviously invalid data (negative numbers, invalid ticker format)

**Acceptance Criteria:**
- [ ] User can edit holdings to correct errors
- [ ] User can delete and re-add holdings
- [ ] Validation prevents invalid data entry

---

### EC-5: Insufficient Cash Balance for Transfer

**Scenario:** User attempts to transfer more cash than available in source account.

**Handling:**
- Validation on form submission
- Error message: "Insufficient cash balance in [From Account]. Available: [CashBalance], Requested: [TransferAmount]"
- Prevent transfer until sufficient funds available

**Acceptance Criteria:**
- [ ] Validation prevents insufficient fund transfers
- [ ] Clear error message displayed
- [ ] User can adjust transfer amount

---

### EC-6: Multiple Purchases of Same Stock (Average Cost Basis)

**Scenario:** User purchases the same stock multiple times at different prices.

**Handling:**
- **Single holding entry:** Maintain one row per ticker per account
- **Average cost basis calculation:**
  - Formula: `New Avg Price = ((Existing Quantity × Existing Price) + (New Quantity × New Price)) / (Existing Quantity + New Quantity)`
  - Update Total Invested: `New Total Invested = New Quantity Total × New Avg Price`
- **Example:**
  - Existing: 10 shares @ $100 = $1,000 invested
  - New purchase: 5 shares @ $120 = $600
  - Result: 15 shares @ $106.67 avg = $1,600 total invested

**Acceptance Criteria:**
- [ ] Average cost basis calculated correctly
- [ ] Single holding row maintained
- [ ] Total invested updated accurately

---

### EC-7: Account Deletion with Holdings

**Scenario:** User attempts to delete an account that still contains holdings or has cash balance.

**Handling:**
- Validation on delete: Check if account has holdings or cash > 0
- Error message: "Cannot delete account [Name]. Please remove all holdings and transfer cash balance first."
- Prevent deletion until account is empty

**Acceptance Criteria:**
- [ ] Validation prevents deletion of non-empty accounts
- [ ] Clear error message displayed
- [ ] User can empty account before deletion

---

## Data Migration & Import/Export

### MIG-1: Bulk Data Import

**Purpose:** Import existing portfolio data from Excel or CSV file into the application.

**Supported File Format:**
- CSV (Comma-Separated Values)

**Import Templates:**

**Holdings Import:**
- CSV Columns: `Account, Ticker, Exchange, Industry, Type, Action, Quantity, PurchasePrice, PurchaseDate, Currency`
- Example:
  ```
  Account,Ticker,Exchange,Industry,Type,Action,Quantity,PurchasePrice,PurchaseDate,Currency
  TFSA,AAPL,NASDAQ,Technology,Investment,Hold,10,150.00,2024-05-15,USD
  ZA,ANH,JSE,Beverages,Investment,Buy,50,450.00,2023-08-20,ZAR
  ```

**Deposits Import:**
- CSV Columns: `Date, Account, Amount, Currency, Description`
- Example:
  ```
  Date,Account,Amount,Currency,Description
  2024-01-15,TFSA,10000.00,ZAR,Monthly deposit
  2024-02-15,US,5000.00,USD,Transfer from savings
  ```

**Transfers Import:**
- CSV Columns: `Date, FromAccount, ToAccount, Amount, Currency, Description`
- Example:
  ```
  Date,FromAccount,ToAccount,Amount,Currency,Description
  2024-03-10,ZA,TFSA,2000.00,ZAR,Rebalance
  ```

**Import Process:**
1. User navigates to Settings > Data Management
2. Click "Import Data" button
3. Select file type (Holdings, Deposits, Transfers)
4. Upload CSV file
5. Preview data with validation errors highlighted
6. Confirm import
7. Data imported into database
8. Success message with summary (X holdings imported, Y errors)

**Validation:**
- Check for required fields
- Validate data types (numeric values, dates)
- Validate ticker symbols (format)
- Check for duplicate entries (optional: skip or merge)
- Display errors in preview before final import

**Error Handling:**
- Invalid rows skipped with error report
- User can download error report (CSV with error descriptions)
- Partial import allowed (valid rows imported, invalid skipped)

**Acceptance Criteria:**
- [ ] Import form accessible from Settings
- [ ] CSV files parsed correctly
- [ ] Validation catches invalid data
- [ ] Preview shows data before import
- [ ] Data imported successfully
- [ ] Error report downloadable
- [ ] Success summary displayed

---

### MIG-2: Bulk Data Export

**Purpose:** Export portfolio data for backup, analysis, or migration to another system.

**Export Options:**

**Full Portfolio Export:**
- All accounts, holdings, deposits, and transfers in a single CSV or multiple CSV files (zipped)

**Individual Exports:**
- **Holdings Export:** All holdings with current prices and P/L
- **Deposits Export:** All deposits
- **Transfers Export:** All transfers
- **Account Summary Export:** Summary metrics per account

**Export Format:**
- CSV (default)
- JSON (optional)

**Export Process:**
1. User navigates to Settings > Data Management
2. Click "Export Data" button
3. Select export type (Full, Holdings, Deposits, Transfers)
4. Click "Download"
5. File downloads to user's device

**Acceptance Criteria:**
- [ ] Export form accessible from Settings
- [ ] CSV files generated correctly
- [ ] All data included in export
- [ ] File downloads successfully
- [ ] Exported data can be re-imported

---

### MIG-3: One-Time Excel Migration

**Purpose:** Migrate existing data from the current Excel-based system to Prosperity.

**Process:**
1. Export Excel data to CSV format (manual or automated)
2. Use Bulk Import functionality to import CSV files
3. Verify data accuracy in Prosperity
4. Make manual corrections if needed
5. Discontinue Excel usage once migration complete

**Timeline:**
- Performed once after MVP is complete and tested
- Estimated time: 1-2 hours

**Acceptance Criteria:**
- [ ] All Excel holdings migrated to Prosperity
- [ ] All deposits and transfers migrated
- [ ] Data accuracy verified
- [ ] Excel file archived as backup

---

## Out of Scope

The following features are explicitly **not included in the MVP** and may be considered for future phases:

### Advanced Features
- Advanced charting and visualizations (line charts, pie charts, performance graphs)
- Historical performance tracking and trending
- Tax reporting and SARS integration
- Price alerts and notifications (email, push, SMS)
- Automated dividend capture from external sources
- Portfolio optimization and rebalancing suggestions
- Benchmark comparison (vs. S&P 500, JSE All Share Index)
- Sector allocation analysis
- Risk assessment and metrics (beta, volatility, Sharpe ratio)

### Integration Features
- Broker API integration (automatic trade imports)
- Bank account linking
- Credit card integration
- Third-party financial software integration (QuickBooks, etc.)

### Social Features
- Portfolio sharing with others
- Community features or forums
- Social media integration
- Public portfolio leaderboards

### Advanced Security
- Two-factor authentication (2FA)
- Biometric authentication (fingerprint, face ID)
- Single Sign-On (SSO)

### Mobile Applications
- Native iOS app
- Native Android app
- Offline mode with sync
- Push notifications

### Multi-User Features
- User roles and permissions (admin, viewer, editor)
- Family account management
- Advisor access
- Collaborative portfolio management

---

## Assumptions & Constraints

### Assumptions
1. **Single User:** Application designed for one primary user (Marcel Rienks)
2. **Manual Data Entry:** User manually enters stock purchases, deposits, and transfers
3. **Internet Connectivity:** Application requires internet access (no offline mode)
4. **Modern Browsers:** User has access to latest versions of Chrome, Edge, Firefox, or Safari
5. **Desktop Primary:** Primary usage on desktop; mobile is secondary
6. **Google Finance API Availability:** Google Finance API (or equivalent) remains accessible and free/low-cost
7. **Learning Curve Acceptable:** User willing to spend time learning and configuring the application
8. **Excel Expertise:** User comfortable with Excel-style workflows and financial concepts

### Constraints
1. **Budget:** Minimal cost (<$5/month for AWS + Azure free tiers)
2. **Time:** Part-time development (evenings/weekends)
3. **Technology Stack:** Locked to Blazor WASM (frontend) and Go (backend) for learning goals
4. **API Rate Limits:** Google Finance and ExchangeRate-API free tier limitations
5. **No Commercial Use:** Personal use only; not intended for sale or distribution
6. **No Support Team:** Single developer; no dedicated support or QA team
7. **Browser Compatibility:** Focus on modern browsers; no IE11 support

---

## Appendix

### A. Glossary

- **TFSA:** Tax-Free Savings Account (South African investment account)
- **JSE:** Johannesburg Stock Exchange (South African stock exchange)
- **P/L:** Profit/Loss
- **FX:** Foreign Exchange (currency conversion)
- **CRUD:** Create, Read, Update, Delete (basic data operations)
- **JWT:** JSON Web Token (authentication mechanism)
- **API:** Application Programming Interface
- **CSV:** Comma-Separated Values (file format)
- **WASM:** WebAssembly (technology for running compiled code in browsers)
- **CDN:** Content Delivery Network
- **NoSQL:** Non-relational database (Cosmos DB)

---

### B. Acronyms

- **MVP:** Minimum Viable Product
- **PRD:** Product Requirements Document
- **UI:** User Interface
- **UX:** User Experience
- **TBD:** To Be Determined
- **API:** Application Programming Interface
- **CORS:** Cross-Origin Resource Sharing
- **HTTPS:** Hypertext Transfer Protocol Secure
- **TLS:** Transport Layer Security

---

### C. References

- [Stakeholder Requirements Document](stakeholder-requirements.md)
- [Architecture Design Document](architecture-design.md)
- [Competitive Analysis Document](competitive-analysis.md)
- [Project Roadmap](project-roadmap.md)

---

### D. User Stories Summary

**As a user, I want to...**

1. See my total portfolio value at a glance on the Dashboard.
2. View each account's performance separately with key metrics.
3. Expand and collapse accounts on the Investments page to focus on specific holdings.
4. Add new stock purchases with all relevant details (ticker, quantity, price, etc.).
5. Edit existing holdings to correct errors or record additional purchases.
6. Delete holdings when I sell a stock.
7. See real-time stock prices automatically updated on page load.
8. Manually refresh prices with a button click.
9. Record deposits into accounts to track capital contributions.
10. Record transfers between accounts to manage cash flow.
11. View my deposit and transfer history on the Deposits & Transfers page.
12. Track available cash balance per account.
13. Customize dropdown options (exchanges, industries, types, actions, currencies) in Settings.
14. Switch between Light and Dark themes.
15. Securely log in with email and password.
16. Log out to end my session.
17. Export all my portfolio data to CSV for backup.
18. Import data from CSV to migrate from my Excel system.
19. See clear error messages if stock prices fail to update.
20. Use the application on both my desktop and mobile phone.

---

### E. Revision History

| Version | Date       | Author        | Description              |
|---------|------------|---------------|--------------------------|
| 1.0     | 2025-10-15 | Marcel Rienks | Initial PRD creation     |

---

**Document Status:** ✅ Final  
**Last Updated:** October 15, 2025  
**Next Steps:** Begin Core Development (MVP Implementation)

---

*End of Product Requirements Document*
