# Prosperity - Data Flow & State Management Diagrams

**Version:** 1.1  
**Date:** October 15, 2025  
**Purpose:** Visual representation of data flow patterns and state management

---

## Important: Data Management Philosophy

**Flexible & Editable Approach:**

This system follows a flexible, user-controlled data management approach:

1. **Optional Transaction Fields**: All transaction amounts, fees, and costs are optional inputs. The system calculates with whatever data is provided.

2. **Fully Editable Values**: All holding values (quantity, purchase price, fees, total invested) and account cash balances remain fully editable at any time after creation.

3. **No Audit Trail**: Only current final values are stored. No historical changes or audit logs are maintained.

4. **Manual Adjustments Welcome**: Users can adjust any value at any time for:
   - Dividends received
   - Interest earned
   - Fees discovered later
   - Manual corrections
   - Any other reason

5. **Optional Automatic Updates**: Cash balance updates from transactions (deposits, withdrawals, transfers, purchases, sales) are optional and configurable based on user preference.

6. **No Enforcement**: No validation prevents negative balances or insufficient funds. All values are user-manageable.

---

## UI Loading Strategy - Hierarchical Panel System

**Progressive Disclosure with Smart Background Refresh:**

The interface uses a hierarchical panel system with stored calculated values and intelligent data freshness management:

1. **Panel Hierarchy**:
   - **Portfolio Panel** (top level) - Shows total portfolio summary
   - **Account Panels** (sub-level) - Shows account summaries when portfolio expanded
   - **Stock Panels** (detail level) - Shows individual stock details when account expanded

2. **Data Storage Strategy**:
   - **Stock Level**: All values stored with price, calculations, and `lastUpdated` timestamp
   - **Account Level**: Pre-calculated totals from constituent stocks with `lastUpdated` timestamp
   - **Portfolio Level**: Pre-calculated totals from all accounts with `lastUpdated` timestamp

3. **Initial Load Behavior**:
   - Single bulk database lookup retrieves all stored data
   - Interface populates immediately with stored values
   - Fast initial render (< 2 seconds) using pre-calculated data

4. **Data Freshness System**:
   - **Fresh Data** (< 15 minutes): Display normally with clean interface
   - **Stale Data** (> 15 minutes): Display with dark overlay + faint spinner
   - User sees data immediately but knows it's being refreshed

5. **Background Refresh Process**:
   - Triggered automatically when stale data detected
   - Also triggered on panel expansions if data is stale
   - Bulk API call fetches all live prices and exchange rates
   - All calculations updated in backend
   - Database bulk update with new values and timestamps
   - Interface receives fresh data and removes overlays/spinners

6. **User Experience Benefits**:
   - **Immediate Response**: Interface always loads instantly
   - **Data Availability**: Never blocked by API calls or network issues
   - **Visual Feedback**: Clear indication when data is being refreshed
   - **Progressive Enhancement**: Fresh data replaces stale data seamlessly
   - **Reduced API Costs**: Bulk operations instead of individual calls

### Panel Interaction & Background Refresh Flow

```mermaid
flowchart TD
    Start[User Interaction<br/>Load/Expand Panel]
    
    Start --> LoadStored[Load Stored Data<br/>from Database]
    
    LoadStored --> CheckTime{Data Age<br/>< 15 minutes?}
    
    CheckTime -->|Yes| ShowFresh[Display Clean Interface<br/>No Overlay/Spinner]
    
    CheckTime -->|No| ShowStale[Display with Dark Overlay<br/>+ Faint Spinner]
    
    ShowStale --> Background[Background Process:<br/>Bulk Data Refresh]
    
    Background --> BulkAPI[Bulk API Call:<br/>All Prices + FX Rates]
    
    BulkAPI --> Calculate[Calculate All Values:<br/>- Stock Levels<br/>- Account Totals<br/>- Portfolio Totals]
    
    Calculate --> BulkUpdate[Bulk Database Update<br/>All Levels + Timestamps]
    
    BulkUpdate --> Return[Return Fresh Data<br/>to Interface]
    
    Return --> UpdateUI[Update All Panels<br/>Remove Overlays/Spinners]
    
    ShowFresh --> UserReady[User Interacts<br/>with Fresh Data]
    UpdateUI --> UserReady
    
    UserReady --> End[Interface Ready<br/>All Data Fresh]

    style Start fill:#e1f5ff
    style ShowStale fill:#fff3cd
    style Background fill:#d4edda
    style BulkAPI fill:#d4edda
    style ShowFresh fill:#d4edda
    style UpdateUI fill:#d4edda
    style End fill:#d4edda
```

### Hierarchical Panel Structure

```mermaid
flowchart TD
    Portfolio[Portfolio Panel<br/>Total Value: R1,250,000<br/>Total P/L: +15.2%<br/>Last Updated: 10:45 AM]
    
    Portfolio -->|Expand| Account1[Account Panel: TFSA<br/>Value: R350,000<br/>P/L: +12.8%<br/>Cash: R5,000<br/>Last Updated: 10:45 AM]
    
    Portfolio -->|Expand| Account2[Account Panel: US<br/>Value: R450,000<br/>P/L: +18.7%<br/>Cash: R15,000<br/>Last Updated: 10:45 AM]
    
    Portfolio -->|Expand| Account3[Account Panel: ZA<br/>Value: R350,000<br/>P/L: +14.1%<br/>Cash: R8,000<br/>Last Updated: 10:45 AM]
    
    Account1 -->|Expand| Stock1[Stock Panel: ANH<br/>Qty: 1000<br/>Price: R125.50<br/>Value: R125,500<br/>P/L: +8.2%<br/>Last Updated: 10:45 AM]
    
    Account2 -->|Expand| Stock2[Stock Panel: AAPL<br/>Qty: 15<br/>Price: $175.80<br/>Value: $2,637<br/>P/L: +15.8%<br/>Last Updated: 10:45 AM]
    
    Account3 -->|Expand| Stock3[Stock Panel: SHP<br/>Qty: 500<br/>Price: R156.25<br/>Value: R78,125<br/>P/L: +12.4%<br/>Last Updated: 10:45 AM]
    
    subgraph Fresh["Fresh Data (< 15min)"]
        Stock1
        Stock2
    end
    
    subgraph Stale["Stale Data (> 15min)"]
        Stock3
    end
    
    style Portfolio fill:#e1f5ff
    style Account1 fill:#fff3cd
    style Account2 fill:#fff3cd
    style Account3 fill:#fff3cd
    style Fresh fill:#d4edda
    style Stale fill:#f8d7da
```

---

## Table of Contents
1. [UI Loading Strategy - Hierarchical Panel System](#ui-loading-strategy---hierarchical-panel-system)
2. [Hierarchical Panel Structure](#hierarchical-panel-structure)
3. [Data Flow Overview](#data-flow-overview)
4. [State Management (Fluxor)](#state-management-fluxor)
5. [Portfolio Calculation Flow](#portfolio-calculation-flow)
6. [Cash Balance Management Flow](#cash-balance-management-flow)
7. [Multi-Currency Conversion Flow](#multi-currency-conversion-flow)

---

## Data Flow Overview

This diagram shows how data flows through the entire system from user action to database persistence.

```mermaid
flowchart TB
    User([User Action])
    
    subgraph Frontend["Frontend - Blazor WASM"]
        UI[UI Component<br/>Dashboard/Investments/etc.]
        Action[Dispatch Action<br/>Fluxor]
        Effect[Effect Handler]
        State[State Store<br/>Fluxor]
        Service[API Service]
    end
    
    subgraph Backend["Backend - Azure Functions"]
        Handler[HTTP Handler]
        Middleware[Middleware<br/>JWT/CORS/Logging]
        Business[Business Logic<br/>Service Layer]
        Repo[Repository<br/>Data Access]
    end
    
    subgraph Data["Data Layer"]
        DB[(Cosmos DB)]
    end
    
    subgraph External["External APIs"]
        ExtAPI[Google Finance /<br/>ExchangeRate API]
    end
    
    User -->|Interact| UI
    UI -->|Dispatch| Action
    Action -->|Trigger| Effect
    Effect -->|Call| Service
    
    Service -->|HTTP Request| Handler
    Handler -->|Validate| Middleware
    Middleware -->|Process| Business
    
    Business -->|Bulk Refresh?| ExtAPI
    ExtAPI -->|All Prices/Rates| Business
    
    Business -->|Bulk Query/Update| Repo
    Repo -->|Stored Data + Bulk Updates| DB
    
    DB -->|Pre-calculated Data| Repo
    Repo -->|Hierarchical Data| Business
    Business -->|Panel Data + Freshness| Handler
    Handler -->|JSON + Timestamps| Service
    
    Service -->|Success| Effect
    Effect -->|Update Panel State| State
    State -->|Re-render Panels| UI
    UI -->|Display with Freshness Indicators| User

    style User fill:#e1f5ff
    style Frontend fill:#fff3cd
    style Backend fill:#fce8e6
    style Data fill:#d4edda
    style External fill:#e7e8ea
```

---

## State Management (Fluxor)

This diagram shows the Redux-pattern state management using Fluxor in the Blazor frontend.

```mermaid
flowchart LR
    subgraph Component["UI Component"]
        View[Component View<br/>@inject IState]
        Dispatch[Dispatch Action]
    end
    
    subgraph Store["Fluxor Store"]
        State[(Application State)]
        Reducer[Reducer]
    end
    
    subgraph Effects["Side Effects"]
        Effect[Effect Handler]
        API[API Service]
    end
    
    View -->|User Action| Dispatch
    Dispatch -->|Action| Reducer
    Reducer -->|Update| State
    
    Dispatch -->|Async Action| Effect
    Effect -->|HTTP Call| API
    API -->|Response| Effect
    Effect -->|Success/Failure Action| Reducer
    
    State -->|Subscribe| View
    State -->|Re-render| View

    style Component fill:#e1f5ff
    style Store fill:#d4edda
    style Effects fill:#fff3cd
```

### Example State Structure

```mermaid
flowchart TB
    Root[Application State]
    
    Root --> Auth[Authentication State]
    Root --> Portfolio[Portfolio State]
    Root --> UI[UI State]
    
    Auth --> Token[JWT Token]
    Auth --> User[Current User]
    Auth --> IsAuth[Is Authenticated]
    
    Portfolio --> Accounts[Accounts List]
    Portfolio --> Holdings[Holdings List]
    Portfolio --> Trans[Transactions List]
    Portfolio --> Summary[Portfolio Summary]
    
    UI --> Loading[Loading States]
    UI --> Errors[Error Messages]
    UI --> Theme[Theme Settings]

    style Root fill:#e1f5ff
    style Auth fill:#fff3cd
    style Portfolio fill:#d4edda
    style UI fill:#fce8e6
```

---

## Portfolio Calculation & Storage Flow

This diagram shows how portfolio metrics are calculated, stored, and retrieved with timestamp tracking.

```mermaid
flowchart TD
    Trigger[Background Refresh Triggered<br/>Data > 15 minutes old]
    
    Trigger --> Fetch[Bulk API Fetch<br/>All Current Prices + FX Rates]
    
    Fetch --> Holdings[Calculate Stock Level]
    Holdings --> H1{For Each Holding}
    H1 --> H2[Current Price × Quantity<br/>= Current Value]
    H2 --> H3[Current Value - Total Invested<br/>= Profit/Loss Amount]
    H3 --> H4["(P/L Amount / Total Invested) × 100<br/>= P/L Percentage"]
    H4 --> H5[Currency Conversion<br/>with fresh FX rates]
    
    H5 --> Store1[Store Stock Data:<br/>- All calculated values<br/>- Current price<br/>- lastUpdated timestamp]
    
    Store1 --> AccLevel[Calculate Account Level]
    AccLevel --> A1{For Each Account}
    A1 --> A2[Sum Holdings Current Values<br/>= Account Current Value]
    A2 --> A3[Sum Holdings Total Invested<br/>= Account Invested]
    A3 --> A4[Account Current Value - Account Invested<br/>= Account P/L]
    A4 --> A5[Account Cash Balance<br/>+ Account Current Value<br/>= Account Total Balance]
    
    A5 --> Store2[Store Account Data:<br/>- All calculated totals<br/>- lastUpdated timestamp]
    
    Store2 --> PortLevel[Calculate Portfolio Level]
    PortLevel --> P1[Sum All Account Values<br/>= Total Current Value]
    P1 --> P2[Sum All Account Invested<br/>= Total Invested]
    P2 --> P3[Sum All Account Cash<br/>= Total Cash]
    P3 --> P4[Total Current Value - Total Invested<br/>= Total P/L]
    
    P4 --> Store3[Store Portfolio Data:<br/>- All calculated totals<br/>- lastUpdated timestamp]
    
    Store3 --> BulkDB[Bulk Database Update<br/>All Levels + Timestamps]
    
    BulkDB --> Return[Return Fresh Data<br/>to Interface]
    
    Return --> Update[Update All Panels<br/>Remove Stale Indicators]

    style Trigger fill:#e1f5ff
    style Fetch fill:#d4edda
    style Store1 fill:#fff3cd
    style Store2 fill:#fff3cd
    style Store3 fill:#fff3cd
    style BulkDB fill:#d4edda
    style Update fill:#d4edda
```

---

## Cash Balance Management Flow

This diagram shows how cash balances can be updated by various operations. **Important:** All balance updates are optional and configurable. The cash balance field remains fully editable at all times for manual adjustments.

```mermaid
flowchart TD
    Account[(Account<br/>Cash Balance<br/>Fully Editable)]
    
    Deposit[Deposit Operation<br/>Optional Auto-Update] -->|+ Amount| Account
    TransferIn[Transfer In<br/>Optional Auto-Update] -->|+ Amount| Account
    TransferOut[Transfer Out<br/>Optional Auto-Update] -->|-Amount| Account
    Purchase[Stock Purchase<br/>Optional Auto-Update] -->|-Amount| Account
    Sale[Stock Sale<br/>Optional Auto-Update] -->|+ Amount| Account
    Manual[Manual Adjustment<br/>Dividends/Interest/Fees] -->|+/- Amount| Account
    
    Account --> Display[Display in UI<br/>Direct Edit Enabled]
    
    style Account fill:#d4edda
    style Deposit fill:#e1f5ff
    style TransferIn fill:#e1f5ff
    style Sale fill:#e1f5ff
    style Manual fill:#e1f5ff
    style TransferOut fill:#f8d7da
    style Purchase fill:#f8d7da
    style Display fill:#d4edda
```

**Note:** The error checking for balance validation has been removed. Users can adjust balances freely, and all transaction updates to cash balance are optional based on user settings.

### Cash Balance Formula

```mermaid
flowchart LR
    subgraph Formula["Cash Balance - Flexible Calculation"]
        Start[Current Cash Balance]
        
        Note1[All Updates Optional<br/>Based on User Settings]
        
        Start -->|+| Deposits[Deposits<br/>optional auto-update]
        Deposits -->|+| TransIn[Transfers In<br/>optional auto-update]
        TransIn -->|-| TransOut[Transfers Out<br/>optional auto-update]
        TransOut -->|-| Purch[Purchases<br/>if enabled]
        Purch -->|+| Sales[Sales<br/>if enabled]
        Sales -->|+/-| Adj[Manual Adjustments<br/>Always Available]
        
        Adj --> Final[Current Cash Balance<br/>Fully Editable Anytime]
        
        Note1 -.-> Start
    end
    
    style Start fill:#e1f5ff
    style Final fill:#d4edda
    style Deposits fill:#d4edda
    style TransIn fill:#d4edda
    style Sales fill:#d4edda
    style TransOut fill:#f8d7da
    style Purch fill:#f8d7da
    style Note1 fill:#fff3cd
```

**Key Points:**
- Cash balance starts at user-defined value (default $0)
- All transaction-based updates are optional (user configurable)
- Manual adjustments always available for dividends, interest, fees, or corrections
- No audit trail maintained - only current value stored
- Balance can be edited directly at any time without restrictions

---

## Multi-Currency Conversion Flow

This diagram shows how multi-currency holdings are converted to the base currency (ZAR).

```mermaid
flowchart TB
    Holdings[Holdings in<br/>Multiple Currencies]
    
    Holdings --> USD[USD Holdings]
    Holdings --> EUR[EUR Holdings]
    Holdings --> GBP[GBP Holdings]
    Holdings --> ZAR[ZAR Holdings]
    
    USD --> API1[Get USD/ZAR Rate]
    EUR --> API2[Get EUR/ZAR Rate]
    GBP --> API3[Get GBP/ZAR Rate]
    ZAR --> Direct[No Conversion<br/>1:1]
    
    subgraph ExternalAPI["ExchangeRate-API"]
        API1
        API2
        API3
    end
    
    API1 --> Rate1[Fetch Fresh Rate]
    API2 --> Rate2[Fetch Fresh Rate]
    API3 --> Rate3[Fetch Fresh Rate]
    
    Rate1 --> Convert1[Value USD × Rate<br/>= Value ZAR]
    Rate2 --> Convert2[Value EUR × Rate<br/>= Value ZAR]
    Rate3 --> Convert3[Value GBP × Rate<br/>= Value ZAR]
    Direct --> Convert4[Value ZAR]
    
    Convert1 --> Sum[Sum All Values<br/>in ZAR]
    Convert2 --> Sum
    Convert3 --> Sum
    Convert4 --> Sum
    
    Sum --> Total[Total Portfolio Value<br/>in ZAR]
    
    Total --> Dashboard[Display on Dashboard]

    style Holdings fill:#e1f5ff
    style ExternalAPI fill:#fff3cd
    style Total fill:#d4edda
    style Dashboard fill:#d4edda
```

### Currency Conversion Example

```mermaid
flowchart LR
    subgraph Example["Example Calculation"]
        H1["Holding 1:<br/>10 shares AAPL<br/>@ $175 = $1,750"]
        H2["Holding 2:<br/>5 shares BMW<br/>@ €95 = €475"]
        H3["Holding 3:<br/>50 shares ANH<br/>@ R450 = R22,500"]
        
        H1 --> R1["Rate: $1 = R18.50<br/>$1,750 × 18.50<br/>= R32,375"]
        H2 --> R2["Rate: €1 = R20.00<br/>€475 × 20.00<br/>= R9,500"]
        H3 --> R3["Already ZAR<br/>= R22,500"]
        
        R1 --> Total["Total Portfolio Value<br/>R32,375 + R9,500 + R22,500<br/>= R64,375"]
    end
    
    style H1 fill:#e1f5ff
    style H2 fill:#e1f5ff
    style H3 fill:#e1f5ff
    style Total fill:#d4edda
```

---

## Average Cost Basis Calculation

This diagram shows how average cost basis is calculated for multiple purchases of the same stock.

```mermaid
flowchart TD
    Start[User Buys Same Stock Again]
    
    Start --> Existing[Existing Holding:<br/>Qty1 = 10 shares<br/>Price1 = $100<br/>Total1 = $1,000]
    
    Start --> New[New Purchase:<br/>Qty2 = 5 shares<br/>Price2 = $120<br/>Total2 = $600]
    
    Existing --> Calc[Calculate Average]
    New --> Calc
    
    Calc --> Formula["Average Price Formula:<br/>(Qty1 × Price1) + (Qty2 × Price2)<br/>÷ (Qty1 + Qty2)"]
    
    Formula --> Step1["(10 × $100) + (5 × $120)<br/>÷ (10 + 5)"]
    Step1 --> Step2["($1,000 + $600) ÷ 15"]
    Step2 --> Step3["$1,600 ÷ 15"]
    Step3 --> Result["Average Price = $106.67"]
    
    Result --> Update[Update Holding:<br/>Quantity = 15<br/>Purchase Price = $106.67<br/>Total Invested = $1,600]
    
    Update --> Display[Display in Holdings Table]

    style Start fill:#e1f5ff
    style Formula fill:#fff3cd
    style Result fill:#d4edda
    style Update fill:#d4edda
    style Display fill:#d4edda
```

---

## API Request/Response Cycle

This diagram shows the complete lifecycle of an API request.

```mermaid
sequenceDiagram
    participant UI as UI Component
    participant Store as Fluxor Store
    participant Effect as Effect Handler
    participant API as API Service
    participant Backend as Azure Functions
    participant DB as Cosmos DB

    UI->>Store: Dispatch LoadHoldingsAction
    Store->>Effect: Trigger LoadHoldingsEffect
    Effect->>Store: Dispatch LoadHoldingsStartAction
    Store->>UI: Update state (loading: true)
    
    Effect->>API: getHoldings()
    API->>API: Add Authorization header
    API->>Backend: GET /api/holdings
    
    Backend->>Backend: Validate JWT
    Backend->>Backend: Extract userId
    Backend->>DB: Query holdings (userId)
    DB-->>Backend: Holdings data
    
    alt Success
        Backend-->>API: 200 OK {holdings}
        API-->>Effect: Success response
        Effect->>Store: Dispatch LoadHoldingsSuccessAction(data)
        Store->>UI: Update state (holdings, loading: false)
        UI->>UI: Render holdings list
    else Error
        Backend-->>API: 401/500 Error
        API-->>Effect: Error response
        Effect->>Store: Dispatch LoadHoldingsFailureAction(error)
        Store->>UI: Update state (error, loading: false)
        UI->>UI: Display error message
    end
```

---

*Last Updated: October 15, 2025*
