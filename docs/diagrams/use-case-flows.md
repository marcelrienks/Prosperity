# Prosperity - Use Case Flow Diagrams

**Version:** 1.0  
**Date:** October 15, 2025  
**Purpose:** Visual representation of key user workflows and use cases

---

## Table of Contents
1. [User Authentication Flow](#user-authentication-flow)
2. [Dashboard Load Flow](#dashboard-load-flow)
3. [Add New Holding Flow](#add-new-holding-flow)
4. [Price Refresh Flow](#price-refresh-flow)
5. [Record Deposit Flow](#record-deposit-flow)
6. [Record Transfer Flow](#record-transfer-flow)
7. [Edit Holding (Multiple Purchases)](#edit-holding-multiple-purchases)
8. [Data Import/Export Flow](#data-importexport-flow)

---

## User Authentication Flow

```mermaid
sequenceDiagram
    actor User
    participant Browser as Blazor WASM
    participant API as Azure Functions
    participant DB as Cosmos DB

    User->>Browser: Enter email & password
    Browser->>API: POST /api/auth/login<br/>{email, password}
    
    API->>DB: Query user by email
    DB-->>API: User record
    
    alt Valid Credentials
        API->>API: Verify password hash
        API->>API: Generate JWT token
        API-->>Browser: 200 OK<br/>{accessToken, refreshToken}
        Browser->>Browser: Store tokens
        Browser->>Browser: Redirect to Dashboard
        Browser-->>User: Show Dashboard
    else Invalid Credentials
        API-->>Browser: 401 Unauthorized<br/>{error: "Invalid credentials"}
        Browser-->>User: Show error message
    end
```

---

## Dashboard Load Flow

```mermaid
sequenceDiagram
    actor User
    participant Browser as Blazor WASM
    participant API as Azure Functions
    participant DB as Cosmos DB
    participant Cache as Price Cache

    User->>Browser: Navigate to Dashboard
    Browser->>Browser: Check JWT token
    
    Browser->>API: GET /api/portfolio/summary<br/>Authorization: Bearer {token}
    
    API->>API: Validate JWT
    API->>API: Extract userId
    
    par Fetch Accounts
        API->>DB: Query accounts<br/>(userId)
        DB-->>API: Accounts list
    and Fetch Holdings
        API->>DB: Query holdings<br/>(userId)
        DB-->>API: Holdings list
    and Fetch Transactions
        API->>DB: Query transactions<br/>(userId)
        DB-->>API: Transactions list
    end
    
    API->>Cache: Check price cache<br/>(15 min TTL)
    
    alt Prices Cached
        Cache-->>API: Cached prices
    else Prices Stale
        Note over API: Prices fetched<br/>on next refresh
    end
    
    API->>API: Calculate totals<br/>P/L, conversions
    API-->>Browser: 200 OK<br/>{portfolioSummary}
    
    Browser->>Browser: Update state (Fluxor)
    Browser->>Browser: Render dashboard
    Browser-->>User: Display portfolio overview

    Note over User,DB: Page load complete<br/>(< 3 seconds)
```

---

## Add New Holding Flow

```mermaid
sequenceDiagram
    actor User
    participant Browser as Blazor WASM
    participant API as Azure Functions
    participant DB as Cosmos DB
    participant ExtAPI as Google Finance API

    User->>Browser: Click "Add Holding"
    Browser->>Browser: Open add holding form
    
    User->>Browser: Fill form<br/>(ticker, account, quantity, etc.)
    User->>Browser: Click "Submit"
    
    Browser->>Browser: Validate input
    
    alt Validation Fails
        Browser-->>User: Show validation errors
    else Validation Passes
        Browser->>API: POST /api/holdings<br/>{holdingData}
        
        API->>API: Validate JWT & data
        
        API->>ExtAPI: GET current price<br/>for {ticker}
        
        alt API Success
            ExtAPI-->>API: Current price
            API->>API: Calculate values<br/>(invested, P/L)
            
            alt Holding Already Exists
                API->>DB: Query existing holding<br/>(ticker, accountId)
                DB-->>API: Existing holding
                API->>API: Calculate average cost<br/>Avg = ((Qty1×Price1) + (Qty2×Price2)) / (Qty1+Qty2)
                API->>DB: Update holding<br/>(averaged values)
            else New Holding
                API->>DB: Insert holding
            end
            
            DB-->>API: Success
            API->>DB: Update account.cashBalance<br/>(optional, if configured)
            API-->>Browser: 201 Created<br/>{holdingData}
            
            Browser->>Browser: Update state
            Browser->>Browser: Refresh investments page
            Browser-->>User: Show success message<br/>"Holding added successfully"
        else API Failure
            ExtAPI-->>API: Error (price unavailable)
            API-->>Browser: 400 Bad Request<br/>{error: "Cannot fetch price"}
            Browser-->>User: Show error<br/>"Unable to fetch current price"
        end
    end
```

---

## Price Refresh Flow

```mermaid
sequenceDiagram
    actor User
    participant Browser as Blazor WASM
    participant API as Azure Functions
    participant DB as Cosmos DB
    participant Cache as Price Cache
    participant ExtAPI as Google Finance API

    User->>Browser: Click "Refresh Prices"
    Browser->>Browser: Show loading spinner
    
    Browser->>API: GET /api/prices/refresh<br/>Authorization: Bearer {token}
    
    API->>API: Validate JWT & extract userId
    API->>DB: Query all holdings<br/>(userId)
    DB-->>API: Holdings list
    
    API->>API: Extract unique<br/>ticker+exchange pairs
    
    loop For each ticker
        API->>Cache: Check cache<br/>(ticker, 15 min TTL)
        
        alt Price Cached
            Cache-->>API: Cached price
        else Price Stale
            API->>ExtAPI: GET /quote/{ticker}
            
            alt Success
                ExtAPI-->>API: Current price
                API->>Cache: Update cache
            else Failure
                ExtAPI-->>API: Error
                Note over API: Log error<br/>Keep old price
            end
        end
    end
    
    API->>API: Calculate new values<br/>for all holdings<br/>(currentValue, P/L)
    
    API->>DB: Batch update holdings<br/>(current prices & values)
    DB-->>API: Success
    
    API->>API: Aggregate portfolio<br/>summary
    
    API-->>Browser: 200 OK<br/>{updatedSummary, errors: []}
    
    Browser->>Browser: Update state
    Browser->>Browser: Hide loading spinner
    Browser->>Browser: Update timestamp
    Browser-->>User: Show success<br/>"Prices updated: 54 stocks"
    
    alt Partial Failures
        Browser-->>User: Show warning<br/>"3 stocks failed to update"
    end

    Note over User,ExtAPI: Refresh complete<br/>(< 10 seconds)
```

---

## Record Deposit Flow

```mermaid
sequenceDiagram
    actor User
    participant Browser as Blazor WASM
    participant API as Azure Functions
    participant DB as Cosmos DB

    User->>Browser: Navigate to "Deposits & Transfers"
    User->>Browser: Click "Add Deposit"
    Browser->>Browser: Open deposit form
    
    User->>Browser: Fill form<br/>(date, account, amount, currency)
    User->>Browser: Click "Submit"
    
    Browser->>Browser: Validate input<br/>(amount > 0, required fields)
    
    alt Validation Fails
        Browser-->>User: Show validation errors
    else Validation Passes
        Browser->>API: POST /api/transactions<br/>{type: "deposit", ...}
        
        API->>API: Validate JWT & data
        
        API->>DB: Insert transaction
        DB-->>API: Transaction created
        
        API->>DB: Update account<br/>account.deposited += amount<br/>account.cashBalance += amount
        DB-->>API: Account updated
        
        API-->>Browser: 201 Created<br/>{transactionData}
        
        Browser->>Browser: Update state
        Browser->>Browser: Refresh transactions list
        Browser->>Browser: Update account totals
        Browser-->>User: Show success<br/>"Deposit recorded successfully"
    end

    Note over User,DB: Cash balance increased<br/>Deposited total increased
```

---

## Record Transfer Flow

```mermaid
sequenceDiagram
    actor User
    participant Browser as Blazor WASM
    participant API as Azure Functions
    participant DB as Cosmos DB

    User->>Browser: Click "Add Transfer"
    Browser->>Browser: Open transfer form
    
    User->>Browser: Fill form<br/>(date, from account, to account, amount)
    User->>Browser: Click "Submit"
    
    Browser->>Browser: Validate input<br/>(from ≠ to, amount > 0)
    
    alt Validation Fails
        Browser-->>User: Show validation errors
    else Validation Passes
        Browser->>API: POST /api/transactions<br/>{type: "transfer", fromAccountId, toAccountId, amount}
        
        API->>API: Validate JWT & data
        API->>DB: Get fromAccount
        DB-->>API: From account data
        
        alt Insufficient Funds
            API->>API: Check cashBalance >= amount
            API-->>Browser: 400 Bad Request<br/>{error: "Insufficient funds"}
            Browser-->>User: Show error<br/>"Insufficient cash in source account"
        else Sufficient Funds
            API->>DB: Begin transaction
            
            API->>DB: Insert transfer record
            DB-->>API: Transfer created
            
            API->>DB: Update fromAccount<br/>deposited -= amount<br/>cashBalance -= amount
            DB-->>API: From account updated
            
            API->>DB: Update toAccount<br/>deposited += amount<br/>cashBalance += amount
            DB-->>API: To account updated
            
            API->>DB: Commit transaction
            
            API-->>Browser: 201 Created<br/>{transferData}
            
            Browser->>Browser: Update state
            Browser->>Browser: Refresh transactions
            Browser->>Browser: Update both account totals
            Browser-->>User: Show success<br/>"Transfer completed successfully"
        end
    end

    Note over User,DB: Cash moved between accounts<br/>Net deposited unchanged
```

---

## Edit Holding (Multiple Purchases)

```mermaid
sequenceDiagram
    actor User
    participant Browser as Blazor WASM
    participant API as Azure Functions
    participant DB as Cosmos DB

    User->>Browser: Click "Edit" on holding
    Browser->>Browser: Open edit form<br/>(pre-filled with current data)
    
    User->>Browser: Update fields<br/>(e.g., add more shares)
    Note over User: Example:<br/>Existing: 10 shares @ $100<br/>Adding: 5 shares @ $120
    
    User->>Browser: Click "Save"
    
    Browser->>Browser: Validate input
    
    Browser->>API: PUT /api/holdings/{id}<br/>{updatedData}
    
    API->>API: Validate JWT & data
    API->>DB: Get existing holding
    DB-->>API: Current holding data
    
    API->>API: Detect quantity increase
    
    alt Multiple Purchases (Quantity Increased)
        API->>API: Calculate average cost<br/>oldQty = 10, oldPrice = $100<br/>newQty = 5, newPrice = $120<br/><br/>avgPrice = ((10×100) + (5×120)) / (10+5)<br/>avgPrice = (1000 + 600) / 15<br/>avgPrice = $106.67
        
        API->>API: Calculate new totals<br/>quantity = 15<br/>purchasePrice = $106.67<br/>totalInvested = 15 × $106.67 = $1,600
        
        API->>DB: Update holding<br/>(averaged values)
        DB-->>API: Holding updated
        
        API-->>Browser: 200 OK<br/>{updatedHolding}
        
        Browser->>Browser: Update state
        Browser->>Browser: Refresh investments page
        Browser-->>User: Show success<br/>"Holding updated (averaged)"
    else Other Edits
        API->>DB: Update holding<br/>(direct update)
        DB-->>API: Success
        API-->>Browser: 200 OK
        Browser-->>User: Show success
    end

    Note over User,DB: Average cost basis maintained<br/>Single holding row preserved
```

---

## Data Import/Export Flow

```mermaid
flowchart TD
    Start([User Action]) --> Choice{Import or Export?}
    
    Choice -->|Export| Export1[Navigate to Settings]
    Export1 --> Export2[Click 'Export Data']
    Export2 --> Export3[Select export type:<br/>Holdings/Deposits/All]
    Export3 --> Export4[API generates CSV]
    Export4 --> Export5[Download CSV file]
    Export5 --> ExportEnd([Export Complete])
    
    Choice -->|Import| Import1[Navigate to Settings]
    Import1 --> Import2[Click 'Import Data']
    Import2 --> Import3[Select import type:<br/>Holdings/Deposits/Transfers]
    Import3 --> Import4[Upload CSV file]
    Import4 --> Import5[API validates data]
    
    Import5 --> Valid{All rows valid?}
    Valid -->|Yes| Import6[Preview data]
    Valid -->|No| Import7[Show errors in preview<br/>highlight invalid rows]
    
    Import6 --> Import8[User confirms import]
    Import7 --> Import8
    
    Import8 --> Import9[API processes valid rows]
    Import9 --> Import10[Insert into database]
    Import10 --> Import11[Update account totals]
    Import11 --> Import12[Generate summary:<br/>X imported, Y errors]
    Import12 --> Import13{Any errors?}
    
    Import13 -->|Yes| Import14[Offer error report download]
    Import13 -->|No| Import15[Show success message]
    
    Import14 --> ImportEnd([Import Complete])
    Import15 --> ImportEnd

    style Start fill:#e1f5ff
    style ExportEnd fill:#d4edda
    style ImportEnd fill:#d4edda
    style Valid fill:#fff3cd
    style Import13 fill:#fff3cd
```

---

## User Journey - First Time Setup

```mermaid
flowchart TD
    Start([New User]) --> Register[Register Account<br/>Email/Password]
    Register --> Login[Login to Application]
    Login --> Dashboard[View Empty Dashboard]
    
    Dashboard --> Setup1{Configure Settings?}
    Setup1 -->|Yes| Settings[Go to Settings]
    Settings --> Theme[Select Light/Dark Theme]
    Theme --> Dropdowns[Customize Dropdowns<br/>Exchanges, Industries, etc.]
    Dropdowns --> Setup2[Save Settings]
    Setup2 --> CreateAccounts
    Setup1 -->|No| CreateAccounts
    
    CreateAccounts[Create Accounts<br/>TFSA, ZA, US, EUR, Prop] --> ImportData{Import from Excel?}
    
    ImportData -->|Yes| Import[Import Holdings CSV]
    Import --> ImportDeposits[Import Deposits CSV]
    ImportDeposits --> Review
    
    ImportData -->|No| Manual[Add Holdings Manually]
    Manual --> AddDeposit[Record Initial Deposits]
    AddDeposit --> Review
    
    Review[Review Dashboard] --> RefreshPrices[Refresh Stock Prices]
    RefreshPrices --> Verify[Verify Calculations]
    Verify --> Complete([Setup Complete<br/>Ready to Use!])

    style Start fill:#e1f5ff
    style Complete fill:#d4edda
    style Setup1 fill:#fff3cd
    style ImportData fill:#fff3cd
```

---

## Error Handling Flow

```mermaid
flowchart TD
    UserAction[User Action] --> APICall[API Request]
    
    APICall --> CheckAuth{JWT Valid?}
    CheckAuth -->|No| Unauthorized[401 Unauthorized<br/>Redirect to Login]
    CheckAuth -->|Yes| ProcessRequest
    
    ProcessRequest[Process Request] --> ExternalAPI{External API Call?}
    
    ExternalAPI -->|Yes| CallExternal[Call Google Finance /<br/>ExchangeRate API]
    CallExternal --> ExtSuccess{Success?}
    
    ExtSuccess -->|No| CacheCheck{Cached Data?}
    CacheCheck -->|Yes| UseCached[Use Cached Data<br/>Show Warning]
    CacheCheck -->|No| ShowError[Show Error<br/>Keep Old Values]
    
    ExtSuccess -->|Yes| UpdateCache[Update Cache]
    UpdateCache --> ProcessDB
    
    ExternalAPI -->|No| ProcessDB
    
    ProcessDB[Database Operation] --> DBSuccess{Success?}
    
    DBSuccess -->|No| DBError[500 Internal Error<br/>Show User-Friendly Message]
    DBSuccess -->|Yes| ValidateData{Data Valid?}
    
    ValidateData -->|No| ValidationError[400 Bad Request<br/>Show Validation Errors]
    ValidateData -->|Yes| Success[200 OK<br/>Return Data]
    
    UseCached --> Success
    Success --> UpdateUI[Update UI<br/>Show Success Message]
    
    Unauthorized --> End([End])
    ShowError --> End
    DBError --> End
    ValidationError --> End
    UpdateUI --> End

    style UserAction fill:#e1f5ff
    style Success fill:#d4edda
    style Unauthorized fill:#f8d7da
    style ShowError fill:#f8d7da
    style DBError fill:#f8d7da
    style ValidationError fill:#f8d7da
    style End fill:#f8f9fa
```

---

*Last Updated: October 15, 2025*
