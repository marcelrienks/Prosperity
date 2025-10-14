# Personal Requirements Document

**Date:** October 14, 2025  
**Author:** Marcel Rienks  
**Based on:** Analysis of current Excel-based portfolio management system (Investments.xlsx)

---

## Executive Summary

This document defines the requirements for Prosperity, a personal portfolio management application designed to replace the current Excel-based system. The application must replicate and enhance the current manual workflow while providing real-time price updates, automated calculations, and improved visualization capabilities.

---

## Current System Analysis

### Existing Excel Structure
The current portfolio management system consists of four worksheets:

1. **Summary Sheet** - High-level portfolio overview across all accounts
2. **Investments Sheet** - Detailed holdings with 54 stocks across 5 accounts
3. **Deposits Sheet** - Transaction and deposit history
4. **Annualised Sheet** - Dividend and deposit tracking for annualized calculations

### Key Accounts Currently Managed
- **ZA** - South African stocks (JSE-listed)
- **TFSA** - Tax-Free Savings Account
- **US** - US-based stocks and ETFs
- **EUR** - European stocks
- **Prop** - Property shares and REITs

### Current Data Points Tracked
- Stock ticker code and exchange
- Industry and investment type
- Purchase quantity, price, and total invested
- Current value with live price integration
- Profit/loss (absolute and percentage)
- Historical profit/loss changes
- Dividend payments and dates

---

## Core Application Requirements

### 1. Summary Dashboard Page

**Purpose:** Provide a high-level overview of the entire portfolio at a glance.

**Required Data Display:**
- **Deposited Amount** - Total capital deposited across all accounts
- **Invested Amount** - Total currently invested in holdings
- **Current Value** - Real-time total portfolio value
- **Cash/Slush Balance** - Uninvested cash per account
- **Total Balance** - Current value + cash
- **Profit/Loss** - Both absolute (ZAR) and percentage
- **Historical Comparison** - Change in profit/loss over time

**Account Summary Cards:**
- Display each account (ZA, TFSA, US, EUR, Prop) as a card or row
- Show key metrics per account:
  - Deposited amount
  - Invested amount
  - Current value
  - Cash balance
  - Profit/loss (amount and %)
- Color-code positive/negative performance
- Support currency conversion (USD, EUR to ZAR)

**Totals Row:**
- Aggregate all accounts
- Display overall portfolio performance
- Show combined profit/loss metrics

**Key Features:**
- Real-time price updates reflected in calculations
- Automatic currency conversion using current exchange rates
- Visual indicators for performance (green/red, up/down arrows)
- Quick navigation to detailed account views

---

### 2. Investments Page (Portfolio Management)

**Purpose:** Manage and view all investment holdings across accounts with expandable account sections.

**Page Structure:**
```
Investments Overview
├── Account: ZA (Expandable)
│   ├── Summary: Total Invested, Value, P/L
│   └── Holdings List (when expanded)
│       ├── ETFGLD - ETF - JSE
│       ├── ANH - Beverages - JSE
│       └── ... (all ZA holdings)
├── Account: TFSA (Expandable)
│   └── Holdings List
├── Account: US (Expandable)
│   └── Holdings List
├── Account: EUR (Expandable)
│   └── Holdings List
└── Account: Prop (Expandable)
    └── Holdings List
```

**Account-Level Display (Collapsed):**
- Account name
- Total deposited
- Total invested
- Current value
- Profit/loss (amount and %)
- Number of holdings
- Expand/collapse toggle

**Holdings Detail (Expanded):**

For each stock/holding, display:
- **Code** - Ticker symbol (e.g., ETFGLD, ANH, AAPL)
- **Exchange** - Trading exchange (JSE, NYSE, NASDAQ, etc.)
- **Industry** - Sector/industry classification
- **Type** - Investment, Speculation, Undervalued, etc.
- **Action** - Current action (Hold, Buy, Sell)
- **Quantity** - Number of shares owned
- **Current Price** - Real-time market price
- **Current Value** - Quantity × Current Price
- **Amount Invested** - Total capital invested in this holding
- **Profit/Loss** - Value - Invested (amount and %)
- **Historical P/L** - Change in profit/loss over time

**Functionality:**
- **Add New Holding** - Form to add stock with:
  - Ticker code
  - Account assignment
  - Exchange
  - Industry/sector
  - Type classification
  - Purchase quantity
  - Purchase price
  - Purchase date
- **Edit Holding** - Modify existing position details
- **Delete/Sell Holding** - Remove or mark as sold
- **Bulk Actions** - Select multiple holdings for batch operations
- **Search & Filter**:
  - Filter by account
  - Filter by industry
  - Filter by type
  - Search by ticker code
  - Sort by profit/loss, value, etc.

**Live Price Integration:**
- Automatic price updates from market data API
- Display last update timestamp
- Manual refresh option
- Handle different exchanges (JSE, NYSE, etc.)
- Currency conversion for foreign stocks

---

### 3. Annualised Dividends Report

**Purpose:** Track dividend income and deposit history for annualized return calculations.

**Data Structure:**
- **Date** - Transaction/dividend date
- **Amount** - Dividend received (positive) or deposit made (negative)
- **Type** - Dividend vs Deposit (automatically determined by sign)
- **Account** - Which account received/deposited (future enhancement)
- **Stock** - Which holding paid dividend (future enhancement)

**Report Views:**

**A. Chronological Transaction List**
- Date-ordered list of all dividends and deposits
- Running total calculations
- Year-to-date summaries
- Month-by-month breakdowns

**B. Dividend Summary**
- Total dividends received (all-time)
- Dividends by year
- Dividends by account
- Average monthly dividend income
- Dividend growth rate

**C. Annualised Returns**
- IRR (Internal Rate of Return) calculation
- XIRR for irregular cash flows
- Annualized return percentage
- Time-weighted returns
- Money-weighted returns

**D. Deposit History**
- Total deposits by year
- Total deposits by account
- Deposit frequency analysis
- Average deposit amount

**Visualizations:**
- Line chart: Dividend income over time
- Bar chart: Yearly deposit vs dividend comparison
- Pie chart: Dividend sources by account
- Growth chart: Cumulative returns

**Export Capabilities:**
- Export to CSV for tax reporting
- PDF report generation
- Date range filtering

---

## Additional Feature Requirements

### 4. Asset Types to Support
- **Stocks** - JSE-listed, NYSE, NASDAQ, international exchanges
- **ETFs** - Local and global exchange-traded funds
- **Property Shares** - REITs, fractional property platforms
- **Unit Trusts** - Mutual funds
- **Cash Holdings** - Uninvested cash balances

### 5. Currency Support and Conversion
- **Primary Currency:** ZAR (South African Rand)
- **Foreign Currencies:** USD, EUR, GBP
- **Exchange Rates:**
  - Automatic real-time FX rate updates
  - Display foreign amounts in original currency
  - Convert to ZAR for portfolio totals
  - Show FX gain/loss separately (future)
- **Display Options:**
  - Toggle between ZAR and original currency
  - Show dual currency display

### 6. Stock Exchanges Supported
- **JSE** - Johannesburg Stock Exchange (primary)
- **NYSE** - New York Stock Exchange
- **NASDAQ** - NASDAQ Stock Market
- **LSE** - London Stock Exchange
- **XETRA** - Deutsche Börse
- **Other** - Support for additional exchanges as needed

### 7. Classification Fields
- **Industry/Sector** - Beverages, Banks, Mining, ETF, etc.
- **Investment Type** - Investment, Speculation, Undervalued, Growth
- **Action Status** - Hold, Buy, Sell, Watch

### 8. Authentication & Security
- Email/password login
- Optional 2FA (TOTP)
- Data encryption at rest and in transit
- Session management
- Password reset functionality

### 9. Data Refresh & Historical Data
- **Price Updates:**
  - Real-time during market hours (15-min delay acceptable)
  - Daily end-of-day updates
  - Manual refresh button
- **Historical Data:**
  - Maintain 5+ years of price history
  - Track historical profit/loss changes
  - Preserve dividend payment history

### 10. Mobile/Responsive Design
- Fully responsive web application
- Mobile-optimized views for:
  - Summary dashboard
  - Account overview
  - Individual holdings
- Touch-friendly controls
- Collapsible sections for mobile efficiency

---

## Feature Prioritization Matrix

| Feature                                | MVP | Phase 2 | Future |
|----------------------------------------|-----|---------|--------|
| Summary Dashboard                      | X   |         |        |
| Investments Page (expandable accounts) | X   |         |        |
| Annualised Dividends Report            | X   |         |        |
| Manual stock entry                     | X   |         |        |
| Real-time price updates                | X   |         |        |
| Multi-currency support (ZAR, USD, EUR) | X   |         |        |
| Profit/loss calculations               | X   |         |        |
| Account management (5 accounts)        | X   |         |        |
| Stock classification (industry, type)  | X   |         |        |
| Basic authentication                   | X   |         |        |
| Responsive mobile design               | X   |         |        |
| Add/Edit/Delete holdings               | X   |         |        |
| Search and filter holdings             |     | X       |        |
| Dividend entry and tracking            |     | X       |        |
| CSV import/export                      |     | X       |        |
| Advanced reporting (charts/graphs)     |     | X       |        |
| Price alerts                           |     |         | X      |
| Tax reporting                          |     |         | X      |
| Historical performance charts          |     |         | X      |
| Automatic dividend capture             |     |         | X      |
| Portfolio rebalancing suggestions      |     |         | X      |
| 2FA implementation                     |     |         | X      |

---

## MVP Scope Definition

### Core MVP Features:
1. **Summary Dashboard** displaying all accounts with key metrics
2. **Investments Page** with expandable account sections and full holdings management
3. **Annualised Dividends Report** with transaction history and basic calculations
4. **Manual data entry** for stocks, dividends, and deposits
5. **Real-time price integration** using Google Finance or equivalent API
6. **Multi-currency support** for ZAR, USD, and EUR
7. **Automatic calculations** for profit/loss, totals, and percentages
8. **Basic authentication** with email/password
9. **Responsive design** for desktop and mobile
10. **Support for 5 account types** (ZA, TFSA, US, EUR, Prop)

### Out of Scope for MVP:
- Advanced charting and visualizations
- Automated dividend capture
- CSV import/export
- Price alerts and notifications
- Tax reporting tools
- Historical performance tracking
- Portfolio optimization suggestions

---

## User Stories

### Summary Dashboard
1. As a user, I want to see my total portfolio value at a glance so I can quickly understand my financial position.
2. As a user, I want to view each account's performance separately so I can compare different investment strategies.
3. As a user, I want to see my profit/loss in both absolute and percentage terms so I can measure performance.
4. As a user, I want foreign investments automatically converted to ZAR so I can see my true total value.

### Investments Page
5. As a user, I want to expand/collapse accounts so I can focus on specific holdings without clutter.
6. As a user, I want to add new stock purchases with all relevant details so I can track my investments.
7. As a user, I want to see real-time prices for my holdings so I know my current portfolio value.
8. As a user, I want to edit holding details when I make additional purchases or corrections.
9. As a user, I want to classify stocks by industry and type so I can analyze my portfolio allocation.
10. As a user, I want to see each stock's profit/loss so I can identify winners and losers.

### Annualised Dividends Report
11. As a user, I want to track all dividend payments by date so I can monitor income generation.
12. As a user, I want to see total dividends received so I can measure passive income.
13. As a user, I want to view my deposit history so I can track capital contributions.
14. As a user, I want to calculate annualized returns including dividends so I can assess true performance.

### General
15. As a user, I want to access the application securely so my financial data is protected.
16. As a user, I want to use the app on my phone and desktop so I can check investments anywhere.
17. As a user, I want the app to automatically calculate all values so I don't need manual Excel formulas.

---

## Technical Preferences

### Data Entry Approach
- **MVP:** Manual entry with web forms
- **Phase 2:** CSV import for bulk data
- **Future:** API integration with brokers for automatic syncing

### Price Data Source
- **Primary:** Google Finance API or similar free/low-cost service
- **Backup:** Manual price entry option
- **Requirement:** Support for JSE, NYSE, NASDAQ exchanges

### Calculation Requirements
- All profit/loss calculations automatic
- Currency conversions using live exchange rates
- Real-time recalculation when prices update
- Maintain calculation audit trail (future)

---

## Privacy & Security Requirements

### Data Protection
- All financial data encrypted at rest and in transit
- No third-party data sharing without explicit consent
- User data export capability (GDPR-style compliance)
- User data deletion on request

### Authentication
- Secure password hashing (bcrypt or similar)
- Email verification for account creation
- Password reset functionality
- Session timeout and management

### Backup
- Regular automated backups
- User-initiated backup capability
- Data export in standard formats (CSV, JSON)

---

## Success Criteria

The MVP will be considered successful if it:
1. ✅ Replicates all current Excel functionality
2. ✅ Provides real-time price updates (removing manual Google Finance checks)
3. ✅ Displays summary, investments, and dividends pages correctly
4. ✅ Calculates all profit/loss and totals automatically
5. ✅ Supports all 5 current accounts (ZA, TFSA, US, EUR, Prop)
6. ✅ Handles multi-currency conversions accurately
7. ✅ Works on both desktop and mobile devices
8. ✅ Maintains data security and user privacy
9. ✅ Loads and updates quickly (< 2 seconds for main pages)
10. ✅ Requires less manual effort than current Excel system

---

*End of document*