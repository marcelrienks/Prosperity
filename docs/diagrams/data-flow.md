# Prosperity - Data Flow & State Management Diagrams

**Version:** 1.0  
**Date:** October 15, 2025  
**Purpose:** Visual representation of data flow patterns and state management

---

## Table of Contents
1. [Data Flow Overview](#data-flow-overview)
2. [State Management (Fluxor)](#state-management-fluxor)
3. [Portfolio Calculation Flow](#portfolio-calculation-flow)
4. [Cash Balance Management Flow](#cash-balance-management-flow)
5. [Multi-Currency Conversion Flow](#multi-currency-conversion-flow)

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
        Cache[(Price Cache<br/>15 min TTL)]
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
    
    Business -->|External Data?| ExtAPI
    ExtAPI -->|Response| Business
    Business -->|Cache| Cache
    
    Business -->|Query/Update| Repo
    Repo -->|CRUD| DB
    
    DB -->|Result| Repo
    Repo -->|Data| Business
    Business -->|Response| Handler
    Handler -->|JSON| Service
    
    Service -->|Success| Effect
    Effect -->|Update| State
    State -->|Re-render| UI
    UI -->|Display| User

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

## Portfolio Calculation Flow

This diagram shows how portfolio metrics are calculated from raw data.

```mermaid
flowchart TD
    Start[Raw Portfolio Data]
    
    Start --> Holdings[Holdings Data]
    Start --> Accounts[Accounts Data]
    Start --> Trans[Transactions Data]
    
    Holdings --> H1{For Each Holding}
    H1 --> H2[Current Price × Quantity<br/>= Current Value]
    H2 --> H3[Current Value - Total Invested<br/>= Profit/Loss Amount]
    H3 --> H4["(P/L Amount / Total Invested) × 100<br/>= P/L Percentage"]
    H4 --> H5[Currency Conversion<br/>if needed]
    
    H5 --> AccSum[Account Summary]
    
    Accounts --> A1{For Each Account}
    A1 --> A2[Sum Holdings Current Values<br/>= Account Current Value]
    A2 --> A3[Sum Holdings Total Invested<br/>= Account Invested]
    A3 --> A4[Account Current Value - Account Invested<br/>= Account P/L]
    A4 --> A5[Account Cash Balance<br/>from Deposits/Transfers]
    A5 --> A6[Account Current Value + Cash<br/>= Account Total Balance]
    
    A6 --> AccSum
    
    Trans --> T1[Sum Deposits per Account<br/>= Account Deposited]
    T1 --> T2[Apply Transfers<br/>+/- between accounts]
    T2 --> AccSum
    
    AccSum --> Port[Portfolio Totals]
    Port --> P1[Sum All Account Current Values<br/>= Total Current Value]
    Port --> P2[Sum All Account Invested<br/>= Total Invested]
    Port --> P3[Sum All Account Cash<br/>= Total Cash]
    Port --> P4[Total Current Value - Total Invested<br/>= Total P/L]
    Port --> P5[Sum All Deposited<br/>= Total Deposited]
    
    P1 --> Final
    P2 --> Final
    P3 --> Final
    P4 --> Final
    P5 --> Final[Portfolio Summary]
    
    Final --> Display[Display on Dashboard]

    style Start fill:#e1f5ff
    style Final fill:#d4edda
    style Display fill:#d4edda
```

---

## Cash Balance Management Flow

This diagram shows how cash balances are updated by various operations.

```mermaid
flowchart TD
    Account[(Account<br/>Cash Balance)]
    
    Deposit[Deposit Operation] -->|+ Amount| Account
    TransferIn[Transfer In] -->|+ Amount| Account
    TransferOut[Transfer Out] -->|-Amount| Account
    Purchase[Stock Purchase<br/>Optional] -->|-Amount| Account
    Manual[Manual Adjustment] -->|+/- Amount| Account
    
    Account --> Check{Balance >= 0?}
    Check -->|Yes| Valid[Valid State]
    Check -->|No| Error[Show Error<br/>Cannot Transfer/Purchase]
    
    Valid --> Display[Display in UI]
    
    style Account fill:#d4edda
    style Deposit fill:#e1f5ff
    style TransferIn fill:#e1f5ff
    style Manual fill:#e1f5ff
    style TransferOut fill:#f8d7da
    style Purchase fill:#f8d7da
    style Error fill:#f8d7da
    style Valid fill:#d4edda
```

### Cash Balance Formula

```mermaid
flowchart LR
    subgraph Formula["Cash Balance Calculation"]
        Start[Initial Balance: 0]
        
        Start -->|+| Deposits[All Deposits]
        Deposits -->|+| TransIn[Transfers In]
        TransIn -->|-| TransOut[Transfers Out]
        TransOut -->|-| Purch[Purchases<br/>if enabled]
        Purch -->|+/-| Adj[Manual Adjustments]
        
        Adj --> Final[Current Cash Balance]
    end
    
    style Start fill:#e1f5ff
    style Final fill:#d4edda
    style Deposits fill:#d4edda
    style TransIn fill:#d4edda
    style Divs fill:#d4edda
    style TransOut fill:#f8d7da
    style Purch fill:#f8d7da
```

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
    
    API1 --> Cache1{Check Cache<br/>24h TTL}
    API2 --> Cache2{Check Cache<br/>24h TTL}
    API3 --> Cache3{Check Cache<br/>24h TTL}
    
    Cache1 -->|Cached| Rate1[Use Cached Rate]
    Cache1 -->|Stale| Fetch1[Fetch Fresh Rate]
    Fetch1 --> Rate1
    
    Cache2 -->|Cached| Rate2[Use Cached Rate]
    Cache2 -->|Stale| Fetch2[Fetch Fresh Rate]
    Fetch2 --> Rate2
    
    Cache3 -->|Cached| Rate3[Use Cached Rate]
    Cache3 -->|Stale| Fetch3[Fetch Fresh Rate]
    Fetch3 --> Rate3
    
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
