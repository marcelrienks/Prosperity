# Data Management Philosophy

**Document Version:** 1.0  
**Last Updated:** October 15, 2025  
**Status:** Approved

---

## Overview

This document outlines the core philosophy for data management in the Prosperity portfolio management application, specifically regarding transactions, cash balances, and value editability.

---

## Core Principles

### 1. Flexibility First

**All transaction fields are optional**
- Amounts, fees, costs, and prices can be omitted during entry
- System calculates with whatever data is available
- Missing values can be added later through editing

### 2. Complete Editability

**Everything remains editable after creation**
- Stock holding values (quantity, purchase price, fees, total invested)
- Account cash balances
- Transaction amounts and descriptions
- No fields become "locked" or "read-only"

### 3. No Historical Tracking

**Only current state matters**
- No audit trail or change history maintained
- No log of who changed what or when
- Database stores only the current final values
- Performance and simplicity over historical compliance

### 4. User-Controlled Automation

**Optional automatic updates**
- Transactions can optionally update cash balances automatically
- User configurable per preference
- Manual adjustments always available as alternative
- No enforced validation or balance checks

### 5. Manual Adjustments Welcome

**Direct value editing for any reason**
- Dividends received → adjust cash balance
- Interest earned → adjust cash balance
- Fees discovered later → adjust holding costs
- Corrections → edit any value directly
- No justification or description required

---

## Implementation Details

### Holdings Data

```json
{
  "quantity": 10,           // ✅ Editable anytime
  "purchasePrice": 150.00,  // ✅ Editable anytime
  "transactionFee": 5.00,   // ✅ Editable anytime
  "totalInvested": 1505.00, // ✅ Editable anytime
  "currency": "USD"          // ✅ Editable anytime
}
```

**All fields remain editable** to allow:
- Correcting data entry errors
- Adding fees discovered after purchase
- Adjusting average cost basis for multiple purchases
- Manual recalculations

### Cash Balances

```json
{
  "accountId": "acc_001",
  "cashBalance": 5000.00    // ✅ Directly editable anytime
}
```

**Cash balance can be:**
- Automatically updated by transactions (if enabled)
- Manually adjusted at any time
- Set to any value (positive or negative)
- Edited without requiring a description

### Transactions

```json
{
  "type": "deposit",
  "amount": 10000.00,      // ⚠️ Optional field
  "description": "...",     // ⚠️ Optional field
  "transactionFee": 0       // ⚠️ Optional field
}
```

**Transaction fields are optional:**
- Amounts can be omitted (system uses available data)
- Fees are optional (default to 0)
- Descriptions are optional
- Transactions serve as records, not strict calculations

---

## Transaction Processing Flow

### Adding a Stock Purchase

**Step 1: User Entry**
```
User provides:
- Ticker: AAPL ✅ Required
- Quantity: 10 ✅ Required
- Purchase Price: $150 ⚠️ Optional
- Transaction Fee: $5 ⚠️ Optional (defaults to 0)
```

**Step 2: System Calculation**
```
IF purchase price provided:
  Total Invested = (Quantity × Purchase Price) + Fee
ELSE:
  Total Invested = use current market price or leave blank
```

**Step 3: Cash Balance Update**
```
IF automatic balance updates enabled:
  Cash Balance -= Total Invested
ELSE:
  Cash Balance unchanged (user adjusts manually later)
```

**Step 4: Post-Creation**
```
✅ All values remain editable
✅ User can adjust purchase price later
✅ User can add fees discovered later
✅ User can recalculate total invested manually
```

### Adding a Deposit

**Step 1: User Entry**
```
User provides:
- Account: TFSA ✅ Required
- Amount: R10,000 ⚠️ Optional
- Description: "Monthly deposit" ⚠️ Optional
```

**Step 2: Cash Balance Update**
```
IF automatic balance updates enabled AND amount provided:
  Cash Balance += Amount
ELSE:
  Cash Balance unchanged (user adjusts manually)
```

**Step 3: Post-Creation**
```
✅ Deposit recorded in transaction history
✅ Amount can be edited later
✅ Cash balance can be adjusted independently
```

### Manual Cash Adjustment (Dividends, Interest, etc.)

**User Action:**
```
User clicks "Edit Cash Balance" on account
- Enters new balance value: R15,234.56
- OR enters adjustment: +R234.56
- No description required
- No justification needed
```

**System Response:**
```
✅ Update cash balance immediately
✅ No transaction record created (optional)
✅ No audit log entry
✅ Display updated balance across all views
```

---

## Benefits

### For Users
1. **Flexibility** - Enter data in any order, fill gaps later
2. **Error Correction** - Fix mistakes easily without admin intervention
3. **Simplicity** - No complex workflows or approval processes
4. **Reality** - Matches how people actually manage portfolios

### For System
1. **Simplicity** - No complex audit trail infrastructure
2. **Performance** - Minimal database writes and storage
3. **Maintainability** - Fewer edge cases to handle
4. **Scalability** - Less data to store and query

---

## Trade-offs

### What We Gain
✅ Simple, flexible data management  
✅ Easy error correction  
✅ User-friendly interface  
✅ Fast development and maintenance  
✅ Lower storage costs  

### What We Lose
❌ No audit trail for compliance  
❌ No change history for disputes  
❌ No automatic error detection  
❌ No data integrity enforcement  
❌ Not suitable for regulated financial institutions  

---

## User Experience Examples

### Example 1: Forgot to Include Fee

**Initial Entry:**
```
Purchase: 10 shares of AAPL @ $150
Total Invested: $1,500
```

**Later Discovery:**
```
User realizes: "Oh, there was a $25 brokerage fee!"
```

**Solution:**
```
1. Edit holding
2. Update "Transaction Fee" field: $25
3. Update "Total Invested" field: $1,525
4. Save ✅
```

### Example 2: Dividend Received

**Situation:**
```
Received $50 dividend on MSFT holdings
```

**Solution:**
```
1. Click "Edit Cash Balance" on US account
2. Current: $2,000 → New: $2,050
3. Save ✅
```

**Alternative (if user prefers tracking):**
```
1. Add "Deposit" transaction
2. Amount: $50
3. Description: "MSFT dividend"
4. Manually adjust cash balance if auto-update disabled
```

### Example 3: Average Cost Basis Update

**Initial Purchase:**
```
10 shares @ $100 = $1,000
```

**Second Purchase:**
```
5 shares @ $120 = $600
```

**User Calculation:**
```
Average: ($1,000 + $600) / 15 shares = $106.67/share
```

**Solution:**
```
1. Edit holding
2. Update Quantity: 15
3. Update Purchase Price: $106.67
4. Update Total Invested: $1,600
5. Save ✅
```

---

## Settings Configuration

### User Preference: Automatic Balance Updates

**Setting: "Automatically update cash balance from transactions"**

**Options:**
- ✅ **Enabled** (Default)
  - Deposits increase cash balance
  - Withdrawals decrease cash balance
  - Transfers adjust both accounts
  - Purchases decrease cash balance
  - Sales increase cash balance
  
- ❌ **Disabled**
  - Transactions recorded only
  - Cash balance managed manually
  - User adjusts balance when needed

**Recommendation:** Enable for convenience, disable for manual control

---

## Technical Implementation Notes

### Database Schema

**No audit fields required:**
```json
{
  "id": "hold_001",
  "createdAt": "2024-05-15T10:00:00Z",   // Initial creation only
  "updatedAt": "2025-10-15T10:00:00Z",   // Last update timestamp
  // No createdBy, updatedBy, changeLog, etc.
}
```

### API Design

**PUT endpoint allows updating any field:**
```http
PUT /api/holdings/{id}
{
  "quantity": 15,
  "purchasePrice": 106.67,
  "transactionFee": 25,
  "totalInvested": 1625
}
```

**No validation beyond basic types:**
- No "field is immutable" errors
- No "insufficient balance" errors
- No "unauthorized change" errors

### Frontend UX

**Edit mode:**
- All fields show as editable inputs
- No disabled/readonly states (except calculated fields like current P/L)
- Inline editing where possible
- No "Are you sure?" confirmation dialogs

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

The Prosperity application prioritizes **flexibility and user control** over strict data integrity and audit compliance. This approach:

- Matches the personal portfolio management use case
- Simplifies development and maintenance
- Provides superior user experience
- Reduces system complexity

This philosophy should guide all future development decisions around data management, validation, and editability.

---

*Last Updated: October 15, 2025*
