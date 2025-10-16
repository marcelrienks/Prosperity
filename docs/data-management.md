# Data Management Philosophy

**Document Version:** 1.1  
**Last Updated:** October 16, 2025  
**Status:** Approved

---

## Quick Reference: Field Editability

#### Investment Fields
| Field | Editable? | Calculation |
|-------|-----------|-------------|
| Ticker | ✏️ Yes | User entered |
| Quantity | ✏️ **YES** | User entered |
| Invested | ✏️ **YES** | User entered |
| Currency | 🔒 No | Set from account currency at creation |
| Price | 🔒 No | Fetched from API |
| Value | 🔒 No | `quantity × price` |
| Profit | 🔒 No | `value - invested` |

### Account Fields
| Field | Editable? | Calculation |
|-------|-----------|-------------|
| Deposited | 🔒 No | Sum of deposits + transfers in - transfers out |
| Invested | 🔒 No | Sum of all `investment.invested` |
| Value | 🔒 No | Sum of all `investment.value` |
| Cash | ✏️ **YES** | User entered (dividends, interest, leftover) |
| Balance | 🔒 No | `value + cash` |
| Profit | 🔒 No | `balance - deposited` |

### Portfolio Fields
| Field | Editable? | Calculation |
|-------|-----------|-------------|
| Total Deposited | 🔒 No | Sum of all `account.deposited` |
| Total Invested | 🔒 No | Sum of all `account.invested` |
| Total Value | 🔒 No | Sum of all `account.value` |
| Total Cash | 🔒 No | Sum of all `account.cash` (edit at account level) |
| Total Balance | 🔒 No | `totalValue + totalCash` |
| Total Profit | 🔒 No | `totalBalance - totalDeposited` |

### Transaction Records
| Type | Editable? | Mutability |
|------|-----------|------------|
| Investment | ✏️ Yes | Mutable - full edit access |
| Deposit | ✏️ Yes | **Editable** - add/edit/delete |
| Transfer | ✏️ Yes | **Editable** - add/edit/delete |

---

## Overview

This document outlines the core philosophy for data management in the Prosperity portfolio management application. Prosperity is a **personal portfolio tracker and overview tool** (not a professional financial management system), designed for personal use without strict financial regulatory requirements or comprehensive auditing needs.

### Application Purpose
- **Personal Use:** Not intended for professional or regulated financial use
- **Portfolio Tracking:** Overview and tracker, not a full portfolio manager
- **Simplified Auditing:** Only deposits and transfers require audit history (for total invested and annualized return calculations)
- **Manual Flexibility:** Dividends, interest, and profits tracked as manual adjustments to balances

---

## Core Principles

### 1. Data Mutability Rules


**Editability Rules:**
- ✅ **Deposit** - Can be added, edited, or deleted
- ✅ **Transfer** - Can be added, edited, or deleted
- ✅ **Investment** - Fully editable at any time
- ✅ **Account balances** - Directly editable at any time
- ✅ **Portfolio grand total** - Calculated from investments, manually adjustable

**No automatic cash balance updates:** Cash balances are only updated manually, not automatically, for any deposit, transfer, or investment. This is to account for fees, exchange rates, and other real-world adjustments.

**Full data refresh required:** After any addition or edit to investments, deposits, or transfers, a full data refresh is required. This recalculates all balances and values using live price data.

**No average purchase price:** The system does not track or calculate average purchase price for any investment.

**Standardized terminology:** The term "investment" is used throughout the documentation and UI (not "holding" or "stock").

**User roles:**
  - **Admin**: Full access to all features, configuration, and static lists
  - **Owner**: Full access to portfolio, but not to configuration or static lists
  - **Viewer**: Read-only access to portfolio

**Documentation status:** This documentation will be finalized after the API is complete.

### 2. Minimal Required Fields

- **Quantity:** ✅ Required
- **Invested Amount:** ✅ Required (total money used to buy the investment)
- **All Other Fields:** ⚠️ Optional (fees, etc.)

**Currency:** Not editable. On creation of an investment, the currency is automatically set from the account's configured currency.
- **Quantity:** ✅ Required
- **Invested Amount:** ✅ Required (total money used to buy the shares)
- **All Other Fields:** ⚠️ Optional (fees, purchase price per share, etc.)

**Deposit/Transfer Requirements:**
- **Amount:** ✅ Required
- **Date:** ✅ Required (for audit trail)
- **Account:** ✅ Required
- **Fee:** ⚠️ Optional (defaults to 0)

### 3. Insert or Update Logic

**Investment Behavior:**
- If investment doesn't exist → Insert new investment
- If investment already exists → Update existing investment with new values
- After any addition or edit (investment, deposit, or transfer) → Trigger a full data refresh to recalculate all balances and values using live price data

### 4. Balance Hierarchy

**Three-Tier Balance Structure:**
1. **Portfolio Grand Total** - Calculated from all accounts
2. **Account Balances** - Calculated from investments + cash in each account
3. **Investment Values** - Calculated as: Quantity × Current Price

**Current Value Calculation:**
- Investment current value = Quantity × Live Price (at any given time)
- Account current value = Sum of all investment values + Cash balance
- Portfolio total value = Sum of all account values

### 5. Manual Adjustments for Income

**Dividends, Interest, and Profit:**
- Tracked as manual adjustments to:
  - Account balance, OR
  - Portfolio grand total
- No separate transaction record required
- User updates balance directly when income received

---

## Implementation Details

### Investment (Stock Holdings) Data

```json
{
  "ticker": "AAPL",          // ✅ Required
  "quantity": 10,            // ✅ Required & Editable
  "invested": 1505.00,       // ✅ Required & Editable (total money spent)
  "currentPrice": 175.80,    // � NOT EDITABLE (fetched from API) - STORED
  "currentValue": 1758.00,   // � NOT EDITABLE (calculated: quantity × price) - STORED
  "profit": 253.00,          // � NOT EDITABLE (calculated: value - invested) - STORED
  "currency": "USD",         // ⚠️ Optional
  "lastUpdated": "2025-10-16T10:45:00Z"  // 🔒 Timestamp of last price fetch & calculation
}
```

**Field Editability:**
- **Quantity:** ✏️ Editable - User updates for additional purchases or sales
- **Invested:** ✏️ Editable - User updates to reflect actual money spent
- **Current Price:** 🔒 Not Editable - Fetched from financial API - **STORED in DB**
- **Current Value:** 🔒 Not Editable - Calculated (quantity × price) - **STORED in DB**
- **Profit:** 🔒 Not Editable - Calculated (value - invested) - **STORED in DB**
- **Last Updated:** 🔒 Not Editable - Timestamp of last refresh - **STORED in DB**

**Storage Strategy:**
- All calculated fields (price, value, profit) are **STORED** in the database
- Includes `lastUpdated` timestamp to track data freshness
- Allows immediate retrieval of potentially stale data
- Separate refresh operation updates all calculated fields + timestamp

**Use Cases:**
- **Additional Purchase:** User edits quantity (e.g., 10 → 15) and invested (e.g., $1,505 → $2,255)
- **Partial Sale:** User edits quantity (e.g., 10 → 5) and invested (e.g., $1,505 → $752.50)
- **Manual Correction:** User edits either field to correct data entry errors

**Insert or Update Logic:**
```
IF investment exists (same ticker, same account):
  IF user clicked "Add" button:
    → ADD to existing values:
       quantity = existing.quantity + new.quantity
       invested = existing.invested + new.invested
  ELSE IF user clicked "Edit" button:
    → REPLACE with new values:
       quantity = new.quantity
       invested = new.invested
ELSE:
  → INSERT new investment
END

THEN:
  → Mark data as stale (needs refresh)
  → Optionally trigger immediate refresh of this investment
  OR
  → Let user manually refresh or wait for next scheduled refresh
```

### Deposit & Transfer Data (IMMUTABLE)

```json
{
  "type": "deposit",         // ✅ Required (deposit/transfer)
  "amount": 10000.00,        // ✅ Required
  "date": "2025-10-15",      // ✅ Required
  "accountId": "acc_001",    // ✅ Required
  "transactionFee": 0,       // ⚠️ Optional (defaults to 0)
  "description": "..."       // ⚠️ Optional
}
```

**Mutability Rules:**
- ✅ Can ADD new deposit/transfer
- ✅ Can DELETE deposit/transfer
- ❌ CANNOT EDIT existing deposit/transfer
- **Reason:** Forms audit trail for total invested calculation

**Purpose:**
- Track total capital invested over time
- Calculate annualized returns
- Confirm investment history for tax purposes

### Account Balances

```json
{
  "accountId": "acc_001",
  "accountName": "TFSA",
  "deposited": 100000.00,           // 🔒 Calculated from deposits & transfers (not editable) - STORED
  "invested": 85000.00,             // 🔒 Calculated from sum of investment.invested (not editable) - STORED
  "value": 95000.00,                // 🔒 Calculated from sum of investment.value (not editable) - STORED
  "cash": 8000.00,                  // ✏️ EDITABLE (leftover + dividends + interest)
  "balance": 103000.00,             // 🔒 Calculated: value + cash (not editable) - STORED
  "profit": 3000.00,                // 🔒 Calculated: balance - deposited (not editable) - STORED
  "lastUpdated": "2025-10-16T10:45:00Z"  // 🔒 Timestamp of last calculation
}
```

**Field Calculations & Editability:**
- **Deposited:** 🔒 Not Editable - Calculated from sum of (deposits + transfers in - transfers out) - **STORED**
- **Invested:** 🔒 Not Editable - Calculated from sum of all investment.invested amounts in this account - **STORED**
- **Value:** 🔒 Not Editable - Calculated from sum of all investment.value (quantity × price) in this account - **STORED**
- **Cash:** ✏️ **EDITABLE** - User manually adjusts for dividends, interest, leftover from investments
- **Balance:** 🔒 Not Editable - Calculated as value + cash - **STORED**
- **Profit:** 🔒 Not Editable - Calculated as balance - deposited - **STORED**
- **Last Updated:** 🔒 Not Editable - Timestamp of last refresh - **STORED in DB**

**Storage Strategy:**
- All calculated totals (deposited, invested, value, balance, profit) are **STORED** in the database
- Cash field is editable and stored as-is
- Includes `lastUpdated` timestamp to track data freshness
- Allows immediate retrieval of potentially stale account totals
- Separate refresh operation recalculates from investments and updates all fields + timestamp

**Balance Calculation:**
```
Deposited = Sum(Deposits) + Sum(Transfers In) - Sum(Transfers Out)  → STORE
Invested = Sum(Investment.Invested for all investments in account)   → STORE
Value = Sum(Investment.Value for all investments in account)         → STORE
Cash = User Editable (manually maintained)                           → STORE as-is
Balance = Value + Cash                                               → STORE
Profit = Balance - Deposited                                         → STORE
LastUpdated = Current timestamp                                      → STORE
```

**Cash Balance Management:**
- User directly edits cash balance field
- Use cases: 
  - Dividend received → Increase cash
  - Interest earned → Increase cash
  - Leftover from trades → Adjust cash
  - Manual corrections → Adjust cash
- No transaction records needed for cash adjustments
- Cash edits trigger recalculation of balance and profit (but not value or invested)

### Portfolio Grand Total

```json
{
  "totalDeposited": 500000.00,      // 🔒 Sum of all account.deposited (not editable) - STORED
  "totalInvested": 425000.00,       // 🔒 Sum of all account.invested (not editable) - STORED
  "totalValue": 485000.00,          // 🔒 Sum of all account.value (not editable) - STORED
  "totalCash": 30000.00,            // 🔒 Sum of all account.cash (not editable at portfolio level) - STORED
  "totalBalance": 515000.00,        // 🔒 Sum of all account.balance OR totalValue + totalCash (not editable) - STORED
  "totalProfit": 15000.00,          // 🔒 totalBalance - totalDeposited (not editable) - STORED
  "totalProfitPercent": 3.00,       // 🔒 (totalProfit / totalDeposited) × 100 (not editable) - STORED
  "lastUpdated": "2025-10-16T10:45:00Z"  // 🔒 Timestamp of last portfolio calculation
}
```

**Field Calculations:**
- **Total Deposited:** 🔒 Not Editable - Sum of all account.deposited - **STORED**
- **Total Invested:** 🔒 Not Editable - Sum of all account.invested - **STORED**
- **Total Value:** 🔒 Not Editable - Sum of all account.value (all investments) - **STORED**
- **Total Cash:** 🔒 Not Editable at portfolio level - Sum of all account.cash (edit individual accounts) - **STORED**
- **Total Balance:** 🔒 Not Editable - totalValue + totalCash - **STORED**
- **Total Profit:** 🔒 Not Editable - totalBalance - totalDeposited - **STORED**
- **Total Profit %:** 🔒 Not Editable - (totalProfit / totalDeposited) × 100 - **STORED**
- **Last Updated:** 🔒 Not Editable - Timestamp of last portfolio refresh - **STORED in DB**

**Storage Strategy:**
- All portfolio totals are **STORED** in the database as aggregated calculations
- Includes `lastUpdated` timestamp to track data freshness
- Allows immediate retrieval of potentially stale portfolio summary
- Separate refresh operation recalculates from all accounts and updates all fields + timestamp
- Portfolio level is read-only; no direct editing (modify accounts and investments instead)

**Calculation Flow:**
```
1. Calculate each investment current value (quantity × price)           → STORE at investment level
2. Sum investments per account → Account value                          → STORE at account level
3. Add account cash balance → Account balance                           → STORE at account level
4. Sum all accounts → Portfolio totals                                  → STORE at portfolio level

Hierarchy:
Investment.value → Account.value → Portfolio.totalValue
Investment.profit → Account.profit → Portfolio.totalProfit

Cascade Refresh:
1. Fetch live prices for all investments
2. Recalculate and store all investment values/profits + timestamp
3. Recalculate and store all account values/balances/profits + timestamp
4. Recalculate and store portfolio totals + timestamp
```

**Important Notes:**
- Portfolio-level totals are all calculated (no direct editing)
- To adjust cash, edit individual account cash balances
- To adjust deposited, add/delete deposit or transfer records
- All investment values update automatically when prices refresh

---

## API Design: Read vs Refresh Strategy

### Overview

The application implements a dual-API strategy to balance performance with data freshness:

1. **Read Endpoints (GET)**: Return stored calculated values immediately (potentially stale)
2. **Refresh Endpoint (POST)**: Fetch live prices, recalculate all levels, store results, return fresh data

This allows the UI to render instantly with cached data, then refresh in the background for live updates.

### Read Endpoints (Immediate - Stale Data)

**GET /api/portfolio**
```json
// Returns stored portfolio totals immediately
{
  "totalDeposited": 500000.00,
  "totalInvested": 425000.00,
  "totalValue": 485000.00,
  "totalCash": 30000.00,
  "totalBalance": 515000.00,
  "totalProfit": 15000.00,
  "totalProfitPercent": 3.00,
  "lastUpdated": "2025-10-16T10:30:00Z"  // Shows age of data
}
```

**GET /api/accounts/{accountId}**
```json
// Returns stored account balances immediately
{
  "accountId": "acc_001",
  "accountName": "TFSA",
  "deposited": 100000.00,
  "invested": 85000.00,
  "value": 95000.00,
  "cash": 8000.00,
  "balance": 103000.00,
  "profit": 3000.00,
  "lastUpdated": "2025-10-16T10:30:00Z"
}
```

**GET /api/investments/{investmentId}**
```json
// Returns stored investment data immediately
{
  "investmentId": "inv_001",
  "accountId": "acc_001",
  "symbol": "AAPL",
  "quantity": 100,
  "invested": 15000.00,
  "price": 175.50,
  "value": 17550.00,
  "profit": 2550.00,
  "profitPercent": 17.00,
  "lastUpdated": "2025-10-16T10:30:00Z"
}
```

### Refresh Endpoint (Live Data - Recalculates All)

**POST /api/portfolio/refresh**
```json
// Request (optional filters)
{
  "accountIds": ["acc_001", "acc_002"],  // Optional: specific accounts
  "symbols": ["AAPL", "MSFT"]            // Optional: specific symbols
}

// Response (fresh calculated data)
{
  "portfolio": {
    "totalDeposited": 500000.00,
    "totalInvested": 425000.00,
    "totalValue": 487200.00,            // ← Updated with live prices
    "totalCash": 30000.00,
    "totalBalance": 517200.00,          // ← Recalculated
    "totalProfit": 17200.00,            // ← Recalculated
    "totalProfitPercent": 3.44,         // ← Recalculated
    "lastUpdated": "2025-10-16T10:45:00Z"  // ← Fresh timestamp
  },
  "accounts": [
    {
      "accountId": "acc_001",
      "deposited": 100000.00,
      "invested": 85000.00,
      "value": 96200.00,                // ← Updated from investments
      "cash": 8000.00,
      "balance": 104200.00,             // ← Recalculated
      "profit": 4200.00,                // ← Recalculated
      "lastUpdated": "2025-10-16T10:45:00Z"
    }
  ],
  "investments": [
    {
      "investmentId": "inv_001",
      "symbol": "AAPL",
      "quantity": 100,
      "invested": 15000.00,
      "price": 177.20,                  // ← Live price from Google Finance
      "value": 17720.00,                // ← Recalculated
      "profit": 2720.00,                // ← Recalculated
      "profitPercent": 18.13,           // ← Recalculated
      "lastUpdated": "2025-10-16T10:45:00Z"
    }
  ],
  "priceUpdates": {
    "AAPL": { "oldPrice": 175.50, "newPrice": 177.20, "change": 1.70 },
    "MSFT": { "oldPrice": 420.00, "newPrice": 422.50, "change": 2.50 }
  }
}
```

### Refresh Operation Flow

**Backend Process:**
```
1. FETCH live prices for all investments (Google Finance API)
   └─ Parallel requests for all unique symbols
   
2. UPDATE Investment Level (for each investment):
   price = live_price_from_google_finance()
   value = quantity × price                    → STORE in DB
   profit = value - invested                   → STORE in DB
   profitPercent = (profit / invested) × 100   → STORE in DB
   lastUpdated = current_timestamp()           → STORE in DB
   
3. UPDATE Account Level (for each account):
   deposited = sum(deposits + transfers)       → STORE in DB
   invested = sum(investment.invested)         → STORE in DB
   value = sum(investment.value)               → STORE in DB
   cash = (unchanged - user editable)          → Already stored
   balance = value + cash                      → STORE in DB
   profit = balance - deposited                → STORE in DB
   lastUpdated = current_timestamp()           → STORE in DB
   
4. UPDATE Portfolio Level:
   totalDeposited = sum(account.deposited)     → STORE in DB
   totalInvested = sum(account.invested)       → STORE in DB
   totalValue = sum(account.value)             → STORE in DB
   totalCash = sum(account.cash)               → STORE in DB
   totalBalance = totalValue + totalCash       → STORE in DB
   totalProfit = totalBalance - totalDeposited → STORE in DB
   totalProfitPercent = (totalProfit / totalDeposited) × 100 → STORE in DB
   lastUpdated = current_timestamp()           → STORE in DB
   
5. RETURN complete refreshed data to client
```

### UI/UX Pattern

**Initial Page Load:**
```javascript
// 1. Immediately fetch and render stale data (instant display)
const portfolioData = await GET('/api/portfolio');
renderDashboard(portfolioData);  // Shows with lastUpdated timestamp

// 2. Then trigger background refresh for live data
const freshData = await POST('/api/portfolio/refresh');
updateDashboard(freshData);  // Updates with fresh prices and new timestamp
showRefreshSuccess("Portfolio updated with live prices");
```

**Manual Refresh Button:**
```javascript
async function handleRefreshClick() {
  showLoadingSpinner();
  const freshData = await POST('/api/portfolio/refresh');
  updateDashboard(freshData);
  hideLoadingSpinner();
  showToast("Refreshed with live prices at " + freshData.portfolio.lastUpdated);
}
```

**Auto-Refresh Strategy:**
```javascript
// Option 1: Periodic background refresh
setInterval(() => {
  POST('/api/portfolio/refresh').then(updateDashboard);
}, 5 * 60 * 1000);  // Every 5 minutes

// Option 2: Refresh on stale data detection
if (isDataStale(portfolioData.lastUpdated, thresholdMinutes: 10)) {
  POST('/api/portfolio/refresh').then(updateDashboard);
}
```

### Data Freshness Indicators

**UI Display:**
```
Portfolio Balance: $517,200.00
Last Updated: 5 minutes ago [🔄 Refresh Now]

[Green indicator] < 5 minutes old
[Yellow indicator] 5-15 minutes old  
[Red indicator] > 15 minutes old
```

### Benefits of This Approach

1. **Instant Rendering**: Users see data immediately (no waiting for price lookups)
2. **Accurate When Needed**: Fresh data available on-demand via refresh
3. **Reduced API Calls**: Google Finance API only called during explicit refresh
4. **Offline Capability**: Can view portfolio even without internet (stale data)
5. **User Control**: Manual refresh button for when users want latest prices
6. **Transparent Staleness**: lastUpdated timestamp shows data age

### Error Handling

**If Refresh Fails:**
```json
// Return error but keep showing stale data
{
  "error": "Failed to fetch live prices",
  "reason": "Google Finance API timeout",
  "fallbackData": {
    "portfolio": { /* stale data from DB */ },
    "lastUpdated": "2025-10-16T10:30:00Z",
    "note": "Showing cached data from 15 minutes ago"
  }
}
```

---

## Transaction Processing Flow

### Adding an Investment (Stock Purchase)

**Scenario 1: Adding via "Add" Button (New or Existing Investment)**

**Step 1: User Entry**
```
User provides:
- Ticker: AAPL ✅ Required
- Quantity: 10 ✅ Required
- Invested: $1,505 ✅ Required (total spent)
```

**Step 2: Check if Investment Exists**
```
QUERY existing investment (ticker = AAPL, accountId = account)

IF investment exists:
  → ADD to existing values (automatic accumulation):
     quantity = existing.quantity + new.quantity (e.g., 20 + 10 = 30)
     invested = existing.invested + new.invested (e.g., $3,000 + $1,505 = $4,505)
ELSE:
  → INSERT new investment with provided values
END
```

**Step 3: Recalculation**
```
1. Fetch current price from API → e.g., $175.80
2. Calculate value = quantity × price → e.g., 30 × $175.80 = $5,274
3. Calculate profit = value - invested → e.g., $5,274 - $4,505 = $769
4. Recalculate account.value (sum all investments)
5. Recalculate account.balance (value + cash)
6. Recalculate account.profit (balance - deposited)
7. Recalculate portfolio totals
```

**Scenario 2: Editing Existing Investment (Manual Adjustment)**

**Step 1: User Opens Edit Form**
```
Pre-populated with current values:
- Ticker: AAPL
- Quantity: 30 ✏️ Editable
- Invested: $4,505 ✏️ Editable
```

**Step 2: User Updates Values**
```
Example - Selling 10 shares:
- Quantity: 30 → 20
- Invested: $4,505 → $3,000 (user decides how to adjust)
```

**Step 3: Recalculation**
```
1. Fetch current price from API → $175.80
2. Calculate value = 20 × $175.80 = $3,516
3. Calculate profit = $3,516 - $3,000 = $516
4. Recalculate account and portfolio totals
```

**Step 4: Post-Operation**
```
✅ Investment values updated
✅ All calculations refreshed
✅ User can edit again anytime
```

### Adding a Deposit (IMMUTABLE)

**Step 1: User Entry**
```
User provides:
- Account: TFSA ✅ Required
- Amount: R10,000 ✅ Required
- Date: 2025-10-15 ✅ Required
- Transaction Fee: R0 ⚠️ Optional (defaults to 0)
```

**Step 2: Record Creation**
```
✅ Create immutable deposit record
✅ Add to deposit history for audit trail
```

**Step 3: Balance Update (Optional)**
```
User may manually adjust account cash balance
OR
User may leave cash balance unchanged
```

**Step 4: Post-Creation**
```
✅ Deposit appears in history
❌ CANNOT edit deposit amount or date
✅ CAN delete deposit if incorrect (and re-add)
```

### Adding a Transfer (IMMUTABLE)

**Step 1: User Entry**
```
User provides:
- From Account: TFSA ✅ Required
- To Account: US ✅ Required
- Amount: R5,000 ✅ Required
- Date: 2025-10-15 ✅ Required
- Fee: R50 ⚠️ Optional
```

**Step 2: Record Creation**
```
✅ Create immutable transfer record
✅ Add to transfer history for audit trail
```

**Step 3: Balance Update (Optional)**
```
User may manually adjust both account cash balances
OR
User may leave balances unchanged
```

**Step 4: Post-Creation**
```
✅ Transfer appears in history
❌ CANNOT edit transfer details
✅ CAN delete transfer if incorrect (and re-add)
```

### Manual Cash Balance Adjustment (Dividends, Interest, Profit)

**User Action:**
```
User clicks "Edit Cash Balance" on account
- Current Cash: $5,000
- Enters adjustment: +$234.56 (dividend received)
- New Cash: $5,234.56
- No transaction record created
- No description required
```

**System Response:**
```
✅ Update account.cash = $5,234.56
✅ Recalculate account.balance = account.value + $5,234.56
✅ Recalculate account.profit = account.balance - account.deposited
✅ Recalculate portfolio.totalCash (sum all account.cash)
✅ Recalculate portfolio.totalBalance
✅ Display updated balance across all views
```

**Use Cases:**
- Dividend received → Increase account cash balance
- Interest earned → Increase account cash balance
- Profit taken (without selling investment) → Adjust cash
- Manual corrections → Adjust cash balance
- Leftover cash from trades → Adjust cash

---

## Benefits

### For Personal Portfolio Tracking
1. **Simplicity** - Minimal required fields for quick entry
2. **Flexibility** - Full editability for investments and balances
3. **Audit Trail Where Needed** - Immutable deposits/transfers for total invested tracking
4. **Manual Control** - User manages dividends, interest, and adjustments directly
5. **No Over-Engineering** - Appropriate for personal use, not professional/regulated systems

### For Users
1. **Quick Entry** - Only quantity and invested amount required
2. **Easy Corrections** - Fix investment details anytime
3. **Direct Adjustments** - Add dividends/interest without complex workflows
4. **Clear History** - Deposit/transfer audit trail for annualized returns

### For System
1. **Simple Logic** - Insert or update investments automatically
2. **Performance** - Minimal validation and constraint checking
3. **Clear Calculations** - Three-tier balance hierarchy (portfolio → account → investment)
4. **Appropriate Complexity** - Matches personal use case, not over-regulated

---

## Trade-offs

### What We Gain
✅ Appropriate for personal portfolio tracking  
✅ Simple insert-or-update logic for investments  
✅ Audit trail where it matters (deposits/transfers)  
✅ Full flexibility for investment adjustments  
✅ Direct balance management for dividends/interest  
✅ Quick entry with minimal required fields  
✅ Clear three-tier balance hierarchy  

### What We Accept
⚠️ Investments are editable (not immutable)  
⚠️ No comprehensive audit trail for investment changes  
⚠️ Manual tracking of dividends and interest  
⚠️ User responsibility for correct data entry  
⚠️ Not suitable for regulated financial institutions  
⚠️ Not designed for professional financial management  

### Design Philosophy
This is a **personal portfolio tracker**, not a professional financial system. The data management approach prioritizes:
- Simplicity and speed for personal use
- Flexibility for corrections and adjustments
- Audit trail only where legally necessary (deposits/transfers for total invested)
- User control over all balances and values  

---

## User Experience Examples

### Example 1: Quick Investment Entry (New Investment)

**Initial Entry via "Add" Button:**
```
Ticker: MSFT
Quantity: 10
Invested: $1,500
→ Submit
```

**System Creates:**
```
Investment:
  Ticker: MSFT
  Quantity: 10
  Invested: $1,500
  Price: $320.50 (fetched from API)
  Value: $3,205 (calculated: 10 × $320.50)
  Profit: $1,705 (calculated: $3,205 - $1,500)
```

### Example 2: Additional Purchase (Via "Add" Button - Automatic Accumulation)

**User Adds Same Investment:**
```
Ticker: MSFT (already exists)
Quantity: 5
Invested: $800
→ Submit via "Add" button
```

**System Automatically Accumulates:**
```
Investment Updated:
  Ticker: MSFT
  Quantity: 15 (10 + 5)
  Invested: $2,300 ($1,500 + $800)
  Price: $320.50 (fetched from API)
  Value: $4,807.50 (calculated: 15 × $320.50)
  Profit: $2,507.50 (calculated: $4,807.50 - $2,300)
```

### Example 3: Manual Adjustment (Via "Edit" Button)

**User Wants to Sell 5 Shares:**
```
1. Click "Edit" on MSFT investment
2. Current values shown:
   - Quantity: 15
   - Invested: $2,300
3. User updates:
   - Quantity: 10 (15 - 5)
   - Invested: $1,533 (user decides to proportionally reduce)
4. Submit via "Edit" form
```

**System Replaces Values:**
```
Investment Updated:
  Ticker: MSFT
  Quantity: 10 (replaced with 10)
  Invested: $1,533 (replaced with $1,533)
  Price: $320.50 (fetched from API)
  Value: $3,205 (calculated: 10 × $320.50)
  Profit: $1,672 (calculated: $3,205 - $1,533)
```

### Example 4: Dividend Received

**Situation:**
```
Received $50 dividend on MSFT holdings
```

**Solution:**
```
1. Click "Edit Cash Balance" on US account
2. Current: $2,000 → New: $2,050
3. Save ✅

Account Updates:
  Cash: $2,050 (edited)
  Balance: $3,205 + $2,050 = $5,255 (recalculated)
  Profit: Recalculated based on new balance
```

### Example 5: Adding Capital (Deposit)

**Situation:**
```
Deposited R10,000 into TFSA account
```

**Solution:**
```
1. Click "Add Deposit"
2. Account: TFSA
3. Amount: R10,000
4. Date: 2025-10-16
5. Submit ✅

Results:
→ Creates immutable deposit record
→ Account.deposited increases by R10,000 (recalculated)
→ Account.profit recalculated (balance - deposited)
→ User manually adjusts TFSA cash balance if desired
```

### Example 6: Correcting a Deposit

**Situation:**
```
Accidentally entered R10,000 instead of R1,000
```

**Solution:**
```
1. Delete incorrect deposit
2. Add new deposit with correct amount: R1,000
✅ (Cannot edit deposits - must delete and re-add)
```

### Example 7: Inter-Account Transfer

**Situation:**
```
Transfer R5,000 from TFSA to US account
```

**Solution:**
```
1. Click "Add Transfer"
2. From: TFSA
3. To: US
4. Amount: R5,000
5. Date: 2025-10-15
6. Submit ✅
→ Creates immutable transfer record
→ User manually adjusts both account cash balances
```

---

## Settings Configuration

### Data Entry Preferences

**Setting: "Require Optional Fields"**

**Options:**
- ❌ **Minimal Entry** (Default)
  - Only quantity and invested required
  - Fastest entry for quick tracking
  
- ✅ **Detailed Entry**
  - Require purchase price and fees
  - More detailed tracking

**Recommendation:** Start with minimal, add details as needed

---

## Technical Implementation Notes

### Database Schema

**Investments Collection (Mutable - Editable Fields):**
```json
{
  "id": "inv_001",
  "userId": "user_123",
  "accountId": "acc_tfsa",
  "ticker": "AAPL",
  "quantity": 10,              // ✏️ EDITABLE by user
  "invested": 1505.00,         // ✏️ EDITABLE by user
  "currentPrice": 175.80,      // 🔒 NOT EDITABLE (fetched from API)
  "currentValue": 1758.00,     // 🔒 NOT EDITABLE (calculated: quantity × price)
  "profit": 253.00,            // 🔒 NOT EDITABLE (calculated: value - invested)
  "currency": "USD",           // Optional
  "createdAt": "2024-05-15T10:00:00Z",
  "updatedAt": "2025-10-16T10:00:00Z"
}
```

**Accounts Collection (Calculated + Editable Cash):**
```json
{
  "id": "acc_tfsa",
  "userId": "user_123",
  "name": "TFSA",
  "deposited": 100000.00,      // 🔒 NOT EDITABLE (calculated from deposits/transfers)
  "invested": 85000.00,        // 🔒 NOT EDITABLE (calculated: sum of investment.invested)
  "value": 95000.00,           // 🔒 NOT EDITABLE (calculated: sum of investment.value)
  "cash": 8000.00,             // ✏️ EDITABLE by user
  "balance": 103000.00,        // 🔒 NOT EDITABLE (calculated: value + cash)
  "profit": 3000.00,           // 🔒 NOT EDITABLE (calculated: balance - deposited)
  "updatedAt": "2025-10-16T10:00:00Z"
}
```

**Deposits Collection (Immutable):**
```json
{
  "id": "dep_001",
  "userId": "user_123",
  "accountId": "acc_tfsa",
  "type": "deposit",
  "amount": 10000.00,
  "date": "2025-10-16",
  "transactionFee": 0.00,      // Optional, defaults to 0
  "createdAt": "2025-10-16T10:00:00Z"
  // No updatedAt - immutable
}
```

**Transfers Collection (Immutable):**
```json
{
  "id": "xfr_001",
  "userId": "user_123",
  "fromAccountId": "acc_tfsa",
  "toAccountId": "acc_us",
  "type": "transfer",
  "amount": 5000.00,
  "date": "2025-10-16",
  "transactionFee": 50.00,     // Optional
  "createdAt": "2025-10-16T10:00:00Z"
  // No updatedAt - immutable
}
```

### API Design

**Investment Endpoints:**
```http
POST   /api/investments       # Insert or update
PUT    /api/investments/{id}  # Update existing
DELETE /api/investments/{id}  # Delete
```

**Deposit Endpoints:**
```http
POST   /api/deposits          # Add new (immutable)
DELETE /api/deposits/{id}     # Delete
# No PUT endpoint - cannot edit
```

**Transfer Endpoints:**
```http
POST   /api/transfers         # Add new (immutable)
DELETE /api/transfers/{id}    # Delete
# No PUT endpoint - cannot edit
```

**Account Balance Endpoints:**
```http
PUT    /api/accounts/{id}/cash    # Edit cash balance only
GET    /api/accounts/{id}         # Get account with all calculated fields
```

### Business Logic

**Investment Insert or Update (via "Add" Button):**
```go
func AddInvestment(userId, accountId, ticker string, data InvestmentData, source string) {
    existing := FindInvestment(userId, accountId, ticker)
    
    if existing != nil && source == "add_button" {
        // Accumulate - ADD to existing values
        existing.Quantity += data.Quantity
        existing.Invested += data.Invested
        existing.UpdatedAt = time.Now()
        Save(existing)
    } else if source == "edit_button" {
        // Replace - overwrite with new values
        existing.Quantity = data.Quantity
        existing.Invested = data.Invested
        existing.UpdatedAt = time.Now()
        Save(existing)
    } else {
        // Insert new investment
        newInvestment := CreateInvestment(data)
        Save(newInvestment)
    }
    
    // Fetch current price from API
    currentPrice := FetchPrice(ticker)
    
    // Calculate derived fields (NOT stored, calculated on-the-fly or cached)
    currentValue := existing.Quantity * currentPrice
    profit := currentValue - existing.Invested
    
    // Recalculate account balances
    RecalculateAccountBalance(accountId)
    RecalculatePortfolioTotal(userId)
}
```

**Account Balance Calculation:**
```go
func RecalculateAccountBalance(accountId string) {
    account := FindAccount(accountId)
    investments := FindInvestmentsByAccount(accountId)
    deposits := FindDepositsByAccount(accountId)
    transfers := FindTransfersByAccount(accountId)
    
    // Calculate deposited (from immutable records)
    account.Deposited = SumDeposits(deposits) + 
                        SumTransfersIn(transfers) - 
                        SumTransfersOut(transfers)
    
    // Calculate invested (sum of all investment.invested)
    account.Invested = 0
    for _, inv := range investments {
        account.Invested += inv.Invested
    }
    
    // Calculate value (sum of all investment.value)
    account.Value = 0
    for _, inv := range investments {
        currentPrice := FetchPrice(inv.Ticker)
        account.Value += inv.Quantity * currentPrice
    }
    
    // Cash is user-editable, don't recalculate
    // account.Cash remains as set by user
    
    // Calculate balance
    account.Balance = account.Value + account.Cash
    
    // Calculate profit
    account.Profit = account.Balance - account.Deposited
    
    Save(account)
}
```
    RecalculateAccountBalance(accountId)
    RecalculatePortfolioTotal(userId)
}
```

**Deposit/Transfer Validation:**
```go
func CreateDeposit(data DepositData) error {
    // Validate required fields
    if data.Amount == 0 || data.Date == "" || data.AccountId == "" {
        return errors.New("amount, date, and account are required")
    }
    
    // Create immutable record
    deposit := Deposit{
        ID:             GenerateID(),
        Amount:         data.Amount,
        Date:           data.Date,
        AccountId:      data.AccountId,
        TransactionFee: data.TransactionFee, // Defaults to 0 if not provided
        CreatedAt:      time.Now(),
    }
    
    Save(deposit)
    return nil
}

// No UpdateDeposit function - immutable
```

### Frontend UX

**Investment Form:**
```
Required Fields:
- Ticker        [Text Input] *Required
- Quantity      [Number]     *Required
- Invested      [Currency]   *Required

Optional Fields:
- Purchase Price [Currency]  Optional
- Fee            [Currency]  Optional (defaults to 0)
- Notes          [Text]      Optional
```

**Deposit Form:**
```
Required Fields:
- Account       [Dropdown]   *Required
- Amount        [Currency]   *Required
- Date          [Date]       *Required

Optional Fields:
- Fee           [Currency]   Optional (defaults to 0)
- Description   [Text]       Optional

Actions:
[Submit]  [Cancel]
(No edit once created - delete and re-add to correct)
```

---

## Future Considerations

### If Audit Trail Becomes Required

**Potential triggers:**
- Regulatory requirements
- Multi-user disputes
- Tax authority requests
- Business expansion

**Implementation options:**
1. **Opt-in audit logging** - Enable per account/user
2. **Separate audit database** - Don't impact main DB performance
3. **Event sourcing** - Store all events, rebuild state
4. **Blockchain** - Immutable transaction ledger (overkill?)

### If Data Integrity Becomes Critical

**Potential needs:**
- Prevent negative balances
- Enforce transaction sequencing
- Validate calculations automatically
- Lock historical data

**Implementation:**
- Add validation layer
- Implement workflow approvals
- Create "locked" periods (e.g., tax year closed)
- Separate "draft" vs "committed" states

---

## Conclusion

The Prosperity application is designed as a **personal portfolio tracker and overview tool** with data management principles that balance simplicity with necessary audit requirements:

### Key Design Decisions

1. **Minimal Required Fields**
   - Investments: Only quantity and invested amount required
   - Deposits/Transfers: Only amount, date, and account required
   - Fast entry for personal tracking needs

2. **Smart Mutability**
   - Investments: Fully editable (insert or update logic)
   - Deposits/Transfers: Immutable (add/delete only) for audit trail
   - Balances: Directly editable for dividends/interest

3. **Three-Tier Balance Hierarchy**
   - Portfolio grand total (calculated from accounts)
   - Account balances (calculated from investments + cash)
   - Investment values (quantity × current price)

4. **Manual Income Tracking**
   - Dividends, interest, and profits tracked as direct balance adjustments
   - No complex transaction records needed
   - User control and simplicity prioritized

5. **Appropriate for Personal Use**
   - Not designed for professional or regulated financial management
   - Portfolio overview and tracker, not full portfolio manager
   - Audit trail only where necessary (deposits/transfers)
   - Flexibility and speed for personal tracking

This philosophy should guide all future development decisions around data management, validation, mutability, and calculation logic.

---

*Last Updated: October 16, 2025*
