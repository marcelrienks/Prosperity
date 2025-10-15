# Prosperity - System Architecture Diagrams

**Version:** 1.0  
**Date:** October 15, 2025  
**Purpose:** Visual representation of Prosperity's system architecture

---

## Table of Contents
1. [High-Level System Architecture](#high-level-system-architecture)
2. [Component Architecture](#component-architecture)
3. [Database Schema Relationships](#database-schema-relationships)
4. [Deployment Architecture](#deployment-architecture)

---

## High-Level System Architecture

This diagram shows the overall system architecture with the hybrid AWS/Azure cloud approach.

```mermaid
flowchart TB
    subgraph User["User Layer"]
        Browser["Web Browser<br/>(Desktop/Mobile)"]
    end

    subgraph AWS["AWS Cloud Infrastructure"]
        CloudFront["CloudFront CDN<br/>(prosperity.yourdomain.com)"]
        S3["S3 Bucket<br/>(Static Hosting)"]
        Route53["Route 53<br/>(DNS Management)"]
        
        Route53 --> CloudFront
        CloudFront --> S3
    end

    subgraph Azure["Azure Cloud Infrastructure"]
        Functions["Azure Functions (Go)<br/>(API Endpoints)"]
        CosmosDB[("Cosmos DB<br/>(NoSQL Database)")]
        KeyVault["Key Vault<br/>(Secrets Management)"]
        AppInsights["Application Insights<br/>(Monitoring)"]
        
        Functions --> CosmosDB
        Functions -.->|Secrets| KeyVault
        Functions -.->|Telemetry| AppInsights
    end

    subgraph External["External Services"]
        GoogleFinance["Google Finance API<br/>(Stock Prices)"]
        ExchangeRate["ExchangeRate-API<br/>(Currency Conversion)"]
    end

    Browser -->|HTTPS| CloudFront
    Browser -->|API Calls<br/>HTTPS + JWT| Functions
    Functions -->|Price Data| GoogleFinance
    Functions -->|FX Rates| ExchangeRate

    style Browser fill:#e1f5ff
    style CloudFront fill:#ff9900
    style S3 fill:#ff9900
    style Route53 fill:#ff9900
    style Functions fill:#0078d4
    style CosmosDB fill:#0078d4
    style KeyVault fill:#0078d4
    style AppInsights fill:#0078d4
    style GoogleFinance fill:#d4edda
    style ExchangeRate fill:#d4edda
```

---

## Component Architecture

This diagram shows the detailed component structure of the frontend and backend layers.

```mermaid
flowchart TB
    subgraph Frontend["Frontend Layer - Blazor WebAssembly"]
        direction TB
        
        subgraph Pages["Pages"]
            Dashboard["Dashboard.razor<br/>(Portfolio Overview)"]
            Investments["Investments.razor<br/>(Holdings Management)"]
            Deposits["DepositsAndTransfers.razor<br/>(Cash Flow Tracking)"]
            Settings["Settings.razor<br/>(Configuration)"]
            Login["Login.razor<br/>(Authentication)"]
        end
        
        subgraph Components["Shared Components"]
            AccountCard["AccountSummaryCard.razor"]
            HoldingRow["HoldingRow.razor"]
            LoadingSpinner["LoadingSpinner.razor"]
        end
        
        subgraph Services["Frontend Services"]
            ApiService["ApiService.cs<br/>(HTTP Client)"]
            AuthService["AuthService.cs<br/>(Authentication)"]
            StateService["StateService.cs<br/>(Fluxor State)"]
        end
        
        Pages --> Components
        Pages --> Services
    end

    subgraph Backend["Backend Layer - Azure Functions (Go)"]
        direction TB
        
        subgraph Handlers["HTTP Handlers"]
            AccountHandler["accounts.go"]
            HoldingHandler["holdings.go"]
            TransactionHandler["transactions.go"]
            PriceHandler["prices.go"]
            AuthHandler["auth.go"]
        end
        
        subgraph Middleware["Middleware"]
            JWTMiddleware["auth.go<br/>(JWT Validation)"]
            CORSMiddleware["cors.go<br/>(CORS Config)"]
            LogMiddleware["logging.go"]
        end
        
        subgraph BusinessLogic["Service Layer"]
            PortfolioService["portfolio_service.go<br/>(Calculations)"]
            PriceService["price_service.go<br/>(API Integration)"]
            AuthServiceGo["auth_service.go<br/>(Token Management)"]
        end
        
        subgraph DataAccess["Data Access Layer"]
            AccountRepo["account_repository.go"]
            HoldingRepo["holding_repository.go"]
            TransactionRepo["transaction_repository.go"]
            UserRepo["user_repository.go"]
        end
        
        Handlers --> Middleware
        Handlers --> BusinessLogic
        BusinessLogic --> DataAccess
    end

    subgraph Database["Database Layer"]
        direction LR
        Users[("users<br/>collection")]
        Accounts[("accounts<br/>collection")]
        Holdings[("holdings<br/>collection")]
        Transactions[("transactions<br/>collection")]
    end

    Frontend -->|HTTPS/JSON| Backend
    Backend --> Database

    style Frontend fill:#e1f5ff
    style Backend fill:#fff3cd
    style Database fill:#d4edda
```

---

## Database Schema Relationships

This diagram shows the relationships between data entities in Cosmos DB.

```mermaid
erDiagram
    USER ||--o{ ACCOUNT : "owns"
    USER ||--o{ HOLDING : "owns"
    USER ||--o{ TRANSACTION : "owns"
    ACCOUNT ||--o{ HOLDING : "contains"
    ACCOUNT ||--o{ TRANSACTION : "records"
    HOLDING ||--o{ TRANSACTION : "references"

    USER {
        string id PK
        string email UK
        string passwordHash
        string firstName
        string lastName
        datetime createdAt
        datetime lastLogin
        json preferences
    }

    ACCOUNT {
        string id PK
        string userId FK
        string name
        string type
        string description
        string currency
        decimal cashBalance
        decimal deposited
        datetime createdAt
        datetime updatedAt
    }

    HOLDING {
        string id PK
        string userId FK
        string accountId FK
        string ticker
        string exchange
        string industry
        string type
        string action
        decimal quantity
        decimal purchasePrice
        decimal totalInvested
        decimal currentPrice
        decimal currentValue
        decimal profitLoss
        decimal profitLossPercent
        string currency
        datetime lastPriceUpdate
        datetime createdAt
        datetime updatedAt
    }

    TRANSACTION {
        string id PK
        string userId FK
        string type
        datetime date
        string accountId FK
        string fromAccountId FK
        string toAccountId FK
        decimal amount
        string currency
        string description
        datetime createdAt
    }
```

---

## Deployment Architecture

This diagram shows the CI/CD pipeline and deployment flow.

```mermaid
flowchart TB
    subgraph Developer["Development"]
        Dev["Developer<br/>(Local Machine)"]
        Git["Git Repository<br/>(GitHub)"]
        
        Dev -->|git push| Git
    end

    subgraph CICD["CI/CD Pipeline - GitHub Actions"]
        direction TB
        
        subgraph FrontendPipeline["Frontend Pipeline"]
            FE_Build["Build Blazor WASM<br/>(dotnet publish)"]
            FE_Test["Run Tests"]
            FE_Deploy["Deploy to S3"]
            FE_Invalidate["Invalidate CloudFront"]
            
            FE_Build --> FE_Test --> FE_Deploy --> FE_Invalidate
        end
        
        subgraph BackendPipeline["Backend Pipeline"]
            BE_Build["Build Go Functions<br/>(go build)"]
            BE_Test["Run Tests"]
            BE_Deploy["Deploy to Azure Functions"]
            
            BE_Build --> BE_Test --> BE_Deploy
        end
    end

    subgraph Production["Production Environment"]
        direction TB
        
        subgraph AWS_Prod["AWS Production"]
            S3_Prod["S3 Bucket"]
            CF_Prod["CloudFront"]
        end
        
        subgraph Azure_Prod["Azure Production"]
            Func_Prod["Azure Functions"]
            DB_Prod[("Cosmos DB")]
        end
    end

    Git -->|Trigger on Push| CICD
    FrontendPipeline -->|Deploy| AWS_Prod
    BackendPipeline -->|Deploy| Azure_Prod
    Func_Prod --> DB_Prod

    style Developer fill:#f8f9fa
    style CICD fill:#fff3cd
    style Production fill:#d4edda
    style FrontendPipeline fill:#e1f5ff
    style BackendPipeline fill:#fce8e6
```

---

## Technology Stack Overview

```mermaid
flowchart LR
    subgraph Frontend["Frontend Technologies"]
        Blazor[".NET 8 Blazor WASM"]
        MudBlazor["MudBlazor UI Components"]
        Fluxor["Fluxor State Management"]
    end

    subgraph Backend["Backend Technologies"]
        Go["Go 1.21+"]
        AzFunc["Azure Functions"]
        JWT["JWT Authentication"]
    end

    subgraph Data["Data Technologies"]
        Cosmos["Cosmos DB (NoSQL)"]
        JSON["JSON Documents"]
    end

    subgraph External["External APIs"]
        GFinance["Google Finance API"]
        ExRate["ExchangeRate-API"]
    end

    Frontend -.->|API Calls| Backend
    Backend -.->|Data Persistence| Data
    Backend -.->|Price/FX Data| External

    style Frontend fill:#e1f5ff
    style Backend fill:#fff3cd
    style Data fill:#d4edda
    style External fill:#fce8e6
```

---

*Last Updated: October 15, 2025*
