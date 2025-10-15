# Prosperity - System Architecture Design

**Version:** 1.0  
**Date:** October 15, 2025  
**Status:** Approved  
**Author:** Marcel Rienks  

---

## Executive Summary

This document outlines the technical architecture for **Prosperity**, a personal stock portfolio management application designed to replace an Excel-based investment tracking system. The architecture adopts a hybrid cloud approach combining AWS static hosting with Azure serverless backend, prioritizing cost-efficiency, learning opportunities, and modern technology exposure while maintaining simplicity suitable for personal use.

### Architecture Overview
- **Frontend:** Blazor WebAssembly (WASM) static site hosted on AWS S3 + CloudFront
- **Backend:** Azure Functions written in Go for serverless API endpoints
- **Database:** Azure Cosmos DB (NoSQL) or Azure SQL Database
- **Authentication:** Azure AD B2C or custom JWT-based authentication
- **External APIs:** Stock price data providers (Alpha Vantage, Yahoo Finance API, or similar)

---

## Table of Contents
1. [Architecture Philosophy](#architecture-philosophy)
2. [Technology Stack](#technology-stack)
3. [System Architecture](#system-architecture)
4. [Component Design](#component-design)
5. [Database Design](#database-design)
6. [API Design](#api-design)
7. [Security Architecture](#security-architecture)
8. [Data Flow](#data-flow)
9. [Infrastructure & Deployment](#infrastructure--deployment)
10. [Performance & Scalability](#performance--scalability)
11. [Cost Analysis](#cost-analysis)
12. [Risk Mitigation](#risk-mitigation)
13. [Future Considerations](#future-considerations)

---

## Architecture Philosophy

### Core Principles

**1. Hybrid Cloud Strategy**
- Leverage existing AWS infrastructure for static hosting and domain management
- Utilize Azure for backend services to gain exposure to Azure ecosystem
- Accept cross-cloud complexity as a learning investment
- Support multi-user access with role-based permissions

**2. Serverless-First Approach**
- Eliminate always-on infrastructure costs
- Pay only for actual usage (suitable for personal application)
- Auto-scaling without manual intervention
- Minimal operational overhead

**3. Technology Learning Goals**
- **Blazor WASM:** Modern .NET frontend framework with C# throughout
- **Go:** Efficient, compiled language for Azure Functions
- **Azure Services:** Hands-on experience with Azure cloud platform
- **Hybrid Architecture:** Real-world experience managing cross-cloud systems

**4. Cost Optimization**
- Static hosting on AWS (pennies per month with existing infrastructure)
- Serverless functions (pay per execution, not per hour)
- Minimal database costs (small dataset, infrequent writes)
- Free tiers where possible (Azure Functions consumption plan)

**5. Simplicity Over Complexity**
- Multi-user application with role-based access control
- Manual data entry focus (no complex broker integrations)
- Straightforward CRUD operations
- Real-time data queries (no caching complexity)
- Minimal state management

---

## Technology Stack

### Frontend Layer

**Blazor WebAssembly (WASM)**
- **Version:** .NET 8.0+
- **Justification:**
  - Full C# stack (frontend + potential shared models with backend)
  - Runs entirely in browser (truly static hosting)
  - Rich component model and dependency injection
  - Native .NET debugging experience
  - Strong typing and compile-time checks
  - Active ecosystem and Microsoft support

**UI Framework:**
- **MudBlazor** or **Radzen Blazor Components**
  - Material Design components
  - Responsive out-of-the-box
  - Rich data grid for holdings display
  - Chart components for visualizations

**State Management:**
- **Fluxor** (Redux pattern for Blazor)
  - Predictable state management
  - Time-travel debugging
  - Clear separation of concerns

**HTTP Client:**
- Built-in `HttpClient` with custom API service layer
- CORS-enabled calls to Azure Functions

---

### Backend Layer

**Azure Functions (Go)**
- **Runtime:** Go 1.21+
- **Hosting Plan:** Consumption Plan (serverless)
- **Justification:**
  - Lightweight, fast startup times
  - Excellent concurrency model
  - Low memory footprint (cost-effective)
  - Strong standard library for HTTP and JSON
  - Learning opportunity for Go ecosystem
  - Native cloud support

**API Framework:**
- Azure Functions HTTP triggers
- Custom routing and middleware
- JSON request/response handling

**Authentication:**
- JWT token validation middleware
- Azure AD B2C integration (optional)
- Custom authentication service

---

### Database Layer

**Option A: Azure Cosmos DB (NoSQL) - Recommended**
- **API:** SQL API (document database)
- **Justification:**
  - Flexible schema for evolving requirements
  - Serverless tier (pay per request)
  - Built-in partitioning and scaling
  - 1000 RU/s free tier (sufficient for personal use)
  - Global distribution capability (if needed)
  - JSON-native (aligns with Go and API responses)

**Schema Design:**
- Partition Key: `userId` (single user, but future-proof)
- Collections: `accounts`, `holdings`, `transactions`, `dividends`, `users`

**Option B: Azure SQL Database**
- Serverless tier available
- Familiar relational model
- Stronger consistency guarantees
- More expensive for infrequent access patterns

**Decision: Use Azure Cosmos DB for cost-effectiveness and learning exposure**

---

### External Services

**Stock Price API**
- **Primary:** Alpha Vantage (free tier: 25 requests/day)
- **Backup:** Yahoo Finance API (via third-party library)
- **Fallback:** Manual price entry

**Currency Exchange Rates**
- **Primary:** ExchangeRate-API.com (free tier: 1500 requests/month)
- **Backup:** European Central Bank API (free, reliable)

**Hosting & CDN**
- **AWS S3:** Static site hosting
- **AWS CloudFront:** CDN for global performance
- **AWS Route 53:** DNS management (existing domain)

**CI/CD**
- **GitHub Actions:** Build and deployment pipelines
- Deploy frontend to AWS S3
- Deploy backend to Azure Functions

---

## System Architecture

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                          User Browser                       │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Blazor WebAssembly Application                 │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────────────┐    │ │
│  │  │Dashboard │ │Investment│ │Deposits & Transfers  │    │ │
│  │  │   Page   │ │   Page   │ │        Page          │    │ │
│  │  └──────────┘ └──────────┘ └──────────────────────┘    │ │
│  │                                                        │ │
│  │  ┌─────────────────────────────────────────────────┐   │ │
│  │  │         State Management (Fluxor)               │   │ │
│  │  └─────────────────────────────────────────────────┘   │ │
│  │                                                        │ │
│  │  ┌─────────────────────────────────────────────────┐   │ │
│  │  │         API Service Layer (HttpClient)          │   │ │
│  │  └─────────────────────────────────────────────────┘   │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                           │
                           │ HTTPS (CORS)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                         AWS Cloud                           │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              CloudFront CDN                            │ │
│  │      (Custom Domain: prosperity.yourdomain.com)        │ │
│  └─────────────────────┬──────────────────────────────────┘ │
│                        │                                    │
│  ┌─────────────────────▼──────────────────────────────────┐ │
│  │         S3 Bucket (Static Hosting)                     │ │
│  │    - index.html, Blazor WASM files                     │ │
│  │    - CSS, JavaScript, WASM binaries                    │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

                           │
                           │ HTTPS API Calls
                           │ (Authorization: Bearer <JWT>)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                        Azure Cloud                          │
│  ┌────────────────────────────────────────────────────────┐ │
│  │            Azure Functions (Go Runtime)                │ │
│  │  ┌───────────────────────────────────────────────────┐ │ │
│  │  │  HTTP Triggers (API Endpoints)                    │ │ │
│  │  │  - GET  /api/accounts                             │ │ │
│  │  │  - GET  /api/holdings                             │ │ │
│  │  │  - POST /api/holdings                             │ │ │
│  │  │  - PUT  /api/holdings/{id}                        │ │ │
│  │  │  - DELETE /api/holdings/{id}                      │ │ │
│  │  │  - GET  /api/dividends                            │ │ │
│  │  │  - POST /api/auth/login                           │ │ │
│  │  │  - GET  /api/prices/refresh                       │ │ │
│  │  └───────────────────────────────────────────────────┘ │ │
│  │                                                        │ │
│  │  ┌───────────────────────────────────────────────────┐ │ │
│  │  │  Middleware Layer                                 │ │ │
│  │  │  - JWT Validation                                 │ │ │
│  │  │  - CORS Configuration                             │ │ │
│  │  │  - Error Handling                                 │ │ │
│  │  │  - Logging                                        │ │ │
│  │  └───────────────────────────────────────────────────┘ │ │
│  │                                                        │ │
│  │  ┌───────────────────────────────────────────────────┐ │ │
│  │  │  Service Layer (Business Logic)                   │ │ │
│  │  │  - Portfolio calculations                         │ │ │
│  │  │  - Currency conversions                           │ │ │
│  │  │  - P/L computations                               │ │ │
│  │  └───────────────────────────────────────────────────┘ │ │
│  │                                                        │ │
│  │  ┌───────────────────────────────────────────────────┐ │ │
│  │  │  Data Access Layer                                │ │ │
│  │  │  - Cosmos DB SDK                                  │ │ │
│  │  │  - Query builders                                 │ │ │
│  │  └───────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Azure Cosmos DB (NoSQL)                        │ │
│  │  ┌────────────┐ ┌────────────┐ ┌──────────────┐        │ │
│  │  │  accounts  │ │  holdings  │ │ transactions │        │ │
│  │  │ collection │ │ collection │ │  collection  │        │ │
│  │  └────────────┘ └────────────┘ └──────────────┘        │ │
│  │  ┌────────────┐ ┌────────────┐                         │ │
│  │  │ dividends  │ │   users    │                         │ │
│  │  │ collection │ │ collection │                         │ │
│  │  └────────────┘ └────────────┘                         │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         Azure AD B2C (Optional)                        │ │
│  │         - User authentication                          │ │
│  │         - Token issuing                                │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

                           │
                           │ External API Calls (Scheduled)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    External Services                        │
│  ┌────────────────┐  ┌──────────────────┐                   │
│  │  Alpha Vantage │  │ ExchangeRate-API │                   │
│  │  (Stock Prices)│  │  (FX Rates)      │                   │
│  └────────────────┘  └──────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Component Design

### Frontend Components (Blazor WASM)

#### 1. Page Components

**Dashboard.razor**
- Summary cards for each account
- Total portfolio metrics
- Quick stats (P/L, current value, deposited)
- Navigation to detail pages

**Investments.razor**
- Expandable account sections
- Holdings data grid (MudBlazor DataGrid)
- Add/Edit/Delete modals
- Filtering and sorting
- Real-time price display

**DepositsAndTransfers.razor**
- Deposit and transfer entry form (relates to accounts from Investments page)
- List of all deposits and transfers (with account references)
- Ability to input new deposits (assign to account)
- Ability to input transfers (from/to accounts)
- Summary of total deposits and transfers
- Date filtering and search
- Data used for annualised performance stats (along with total portfolio balance)

**Auth/Login.razor**
- Login form
- Password reset link
- Registration (if needed)

#### 2. Shared Components

**AccountSummaryCard.razor**
- Reusable account display component
- Props: account data, expand/collapse state
- Color-coded P/L indicators

**HoldingRow.razor**
- Single holding display in data grid
- Inline editing capability
- Action buttons (edit, delete)

**LoadingSpinner.razor**
- Loading state indicator
- Used during API calls

**ErrorBoundary.razor**
- Global error handling
- User-friendly error messages

#### 3. Services (Frontend)

**ApiService.cs**
- Centralized HTTP client wrapper
- Handles authentication headers
- Request/response serialization
- Error handling and retry logic

**StateService.cs**
- Fluxor store configuration
- Actions, reducers, effects
- State persistence (local storage)

**AuthService.cs**
- Login/logout functionality
- Token management
- Session handling

---

### Backend Components (Azure Functions - Go)

#### 1. HTTP Trigger Functions

**handlers/accounts.go**
```go
// GET /api/accounts
func GetAccountsHandler(w http.ResponseWriter, r *http.Request)

// GET /api/accounts/{id}
func GetAccountByIdHandler(w http.ResponseWriter, r *http.Request)

// POST /api/accounts
func CreateAccountHandler(w http.ResponseWriter, r *http.Request)

// PUT /api/accounts/{id}
func UpdateAccountHandler(w http.ResponseWriter, r *http.Request)
```

**handlers/holdings.go**
```go
// GET /api/holdings
func GetHoldingsHandler(w http.ResponseWriter, r *http.Request)

// GET /api/holdings/{id}
func GetHoldingByIdHandler(w http.ResponseWriter, r *http.Request)

// POST /api/holdings
func CreateHoldingHandler(w http.ResponseWriter, r *http.Request)

// PUT /api/holdings/{id}
func UpdateHoldingHandler(w http.ResponseWriter, r *http.Request)

// DELETE /api/holdings/{id}
func DeleteHoldingHandler(w http.ResponseWriter, r *http.Request)
```

**handlers/dividends.go**
```go
// GET /api/dividends
func GetDividendsHandler(w http.ResponseWriter, r *http.Request)

// POST /api/dividends
func CreateDividendHandler(w http.ResponseWriter, r *http.Request)

// PUT /api/dividends/{id}
func UpdateDividendHandler(w http.ResponseWriter, r *http.Request)

// DELETE /api/dividends/{id}
func DeleteDividendHandler(w http.ResponseWriter, r *http.Request)
```

**handlers/prices.go**
```go
// GET /api/prices/refresh
func RefreshPricesHandler(w http.ResponseWriter, r *http.Request)

// GET /api/prices/{ticker}
func GetPriceByTickerHandler(w http.ResponseWriter, r *http.Request)
```

**handlers/auth.go**
```go
// POST /api/auth/login
func LoginHandler(w http.ResponseWriter, r *http.Request)

// POST /api/auth/register
func RegisterHandler(w http.ResponseWriter, r *http.Request)

// POST /api/auth/refresh
func RefreshTokenHandler(w http.ResponseWriter, r *http.Request)
```

#### 2. Middleware (Go)

**middleware/auth.go**
```go
func JWTAuthMiddleware(next http.Handler) http.Handler
func ExtractUserContext(next http.Handler) http.Handler
```

**middleware/cors.go**
```go
func CORSMiddleware(next http.Handler) http.Handler
```

**middleware/logging.go**
```go
func LoggingMiddleware(next http.Handler) http.Handler
```

**middleware/error.go**
```go
func ErrorHandlingMiddleware(next http.Handler) http.Handler
```

#### 3. Service Layer (Go)

**services/portfolio_service.go**
```go
type PortfolioService struct {
    db *cosmosdb.Client
}

func (s *PortfolioService) CalculateTotalValue(userId string) (float64, error)
func (s *PortfolioService) CalculateProfitLoss(holdingId string) (float64, float64, error)
func (s *PortfolioService) GetAccountSummary(accountId string) (*AccountSummary, error)
```

**services/price_service.go**
```go
type PriceService struct {
    apiKey string
    cache  *PriceCache
}

func (s *PriceService) FetchStockPrice(ticker, exchange string) (float64, error)
func (s *PriceService) FetchExchangeRate(from, to string) (float64, error)
func (s *PriceService) RefreshAllPrices(userId string) error
```

**services/auth_service.go**
```go
type AuthService struct {
    db        *cosmosdb.Client
    jwtSecret string
}

func (s *AuthService) Login(email, password string) (string, error)
func (s *AuthService) ValidateToken(token string) (*UserClaims, error)
func (s *AuthService) HashPassword(password string) (string, error)
```

#### 4. Data Access Layer (Go)

**repositories/account_repository.go**
```go
type AccountRepository struct {
    client *cosmosdb.Client
}

func (r *AccountRepository) GetAll(userId string) ([]Account, error)
func (r *AccountRepository) GetById(id string) (*Account, error)
func (r *AccountRepository) Create(account *Account) error
func (r *AccountRepository) Update(account *Account) error
```

**repositories/holding_repository.go**
**repositories/dividend_repository.go**
**repositories/user_repository.go**

---

## Database Design

### Azure Cosmos DB Schema

#### Collection: `users`
```json
{
  "id": "user_123",
  "userId": "user_123",
  "email": "user@example.com",
  "passwordHash": "bcrypt_hash_here",
  "firstName": "Marcel",
  "lastName": "Rienks",
  "role": "Admin",
  "status": "Active",
  "createdAt": "2025-10-15T10:00:00Z",
  "lastLogin": "2025-10-15T10:30:00Z"
}
```

#### Collection: `accounts`
```json
{
  "id": "acc_001",
  "userId": "user_123",
  "accountId": "acc_001",
  "name": "TFSA",
  "type": "TFSA",
  "description": "Tax-Free Savings Account",
  "currency": "ZAR",
  "cashBalance": 5000.00,
  "createdAt": "2025-01-01T00:00:00Z",
  "updatedAt": "2025-10-15T10:00:00Z"
}
```

#### Collection: `holdings`
```json
{
  "id": "hold_001",
  "userId": "user_123",
  "holdingId": "hold_001",
  "accountId": "acc_001",
  "ticker": "AAPL",
  "exchange": "NASDAQ",
  "industry": "Technology",
  "type": "Investment",
  "action": "Hold",
  "quantity": 10,
  "purchasePrice": 150.00,
  "purchaseDate": "2024-05-15T00:00:00Z",
  "totalInvested": 1500.00,
  "currentPrice": 175.50,
  "currentValue": 1755.00,
  "profitLoss": 255.00,
  "profitLossPercent": 17.00,
  "currency": "USD",
  "lastPriceUpdate": "2025-10-15T10:00:00Z",
  "createdAt": "2024-05-15T10:00:00Z",
  "updatedAt": "2025-10-15T10:00:00Z"
}
```

#### Collection: `transactions`
```json
{
  "id": "txn_001",
  "userId": "user_123",
  "transactionId": "txn_001",
  "accountId": "acc_001",
  "type": "deposit",
  "amount": 10000.00,
  "currency": "ZAR",
  "date": "2025-01-15T00:00:00Z",
  "description": "Monthly deposit",
  "createdAt": "2025-01-15T10:00:00Z"
}
```

#### Collection: `dividends`
```json
{
  "id": "div_001",
  "userId": "user_123",
  "dividendId": "div_001",
  "accountId": "acc_001",
  "holdingId": "hold_001",
  "ticker": "AAPL",
  "amount": 5.20,
  "currency": "USD",
  "paymentDate": "2025-08-15T00:00:00Z",
  "exDividendDate": "2025-07-30T00:00:00Z",
  "type": "dividend",
  "createdAt": "2025-08-15T10:00:00Z"
}
```

#### Collection: `auditlogs`
```json
{
  "id": "log_001",
  "userId": "user_123",
  "userEmail": "user@example.com",
  "action": "USER_LOGIN",
  "resourceType": "User",
  "resourceId": "user_123",
  "ipAddress": "192.168.1.1",
  "result": "Success",
  "details": {
    "userAgent": "Mozilla/5.0...",
    "location": "Cape Town, ZA"
  },
  "timestamp": "2025-10-15T10:00:00Z"
}
```

### Partition Strategy

**Partition Key:** `userId`

**Rationale:**
- Scalable for multi-user application
- All queries scoped by userId
- Efficient query performance within user's data
- Admin users can query across partitions when needed

### Indexing Strategy

**Cosmos DB Automatic Indexing:**
- Index all properties by default
- Custom indexes for frequently queried fields:
  - `accountId`
  - `ticker`
  - `date` (for transactions and dividends)
  - `timestamp` (for audit logs)
  - `role` and `status` (for user queries)

---

## API Design

### REST API Endpoints

#### Authentication

| Method | Endpoint             | Description                  | Auth Required |
|--------|----------------------|------------------------------|---------------|
| POST   | `/api/auth/login`    | User login                   | No            |
| POST   | `/api/auth/register` | User registration            | No            |
| POST   | `/api/auth/refresh`  | Refresh access token         | Yes           |
| POST   | `/api/auth/logout`   | Logout (invalidate session)  | Yes           |

#### Accounts

| Method | Endpoint             | Description                  | Auth Required |
|--------|----------------------|------------------------------|---------------|
| GET    | `/api/accounts`      | List all accounts            | Yes           |
| GET    | `/api/accounts/{id}` | Get account by ID            | Yes           |
| POST   | `/api/accounts`      | Create new account           | Yes           |
| PUT    | `/api/accounts/{id}` | Update account               | Yes           |
| DELETE | `/api/accounts/{id}` | Delete account               | Yes           |

#### Holdings

| Method | Endpoint             | Description                  | Auth Required |
|--------|----------------------|------------------------------|---------------|
| GET    | `/api/holdings`      | List all holdings            | Yes           |
| GET    | `/api/holdings/{id}` | Get holding by ID            | Yes           |
| POST   | `/api/holdings`      | Create new holding           | Yes           |
| PUT    | `/api/holdings/{id}` | Update holding               | Yes           |
| DELETE | `/api/holdings/{id}` | Delete holding               | Yes           |

#### Dividends

| Method | Endpoint              | Description                  | Auth Required |
|--------|-----------------------|------------------------------|---------------|
| GET    | `/api/dividends`      | List all dividends           | Yes           |
| GET    | `/api/dividends/{id}` | Get dividend by ID           | Yes           |
| POST   | `/api/dividends`      | Create new dividend entry    | Yes           |
| PUT    | `/api/dividends/{id}` | Update dividend              | Yes           |
| DELETE | `/api/dividends/{id}` | Delete dividend              | Yes           |

#### Admin (Role: Admin only)

| Method | Endpoint                    | Description                       | Auth Required | Role Required |
|--------|----------------------------|-----------------------------------|---------------|---------------|
| GET    | `/api/admin/users`         | List all users                    | Yes           | Admin         |
| GET    | `/api/admin/users/{id}`    | Get user by ID                    | Yes           | Admin         |
| POST   | `/api/admin/users`         | Create new user                   | Yes           | Admin         |
| PUT    | `/api/admin/users/{id}`    | Update user (name, email, role)   | Yes           | Admin         |
| DELETE | `/api/admin/users/{id}`    | Delete user                       | Yes           | Admin         |
| PUT    | `/api/admin/users/{id}/status` | Activate/deactivate user      | Yes           | Admin         |
| POST   | `/api/admin/users/{id}/reset-password` | Generate password reset | Yes           | Admin         |
| GET    | `/api/admin/audit-logs`    | Get audit logs                    | Yes           | Admin         |
| GET    | `/api/admin/audit-logs/user/{id}` | Get audit logs for specific user | Yes        | Admin         |

#### Prices

| Method | Endpoint                  | Description                       | Auth Required |
|--------|---------------------------|-----------------------------------|---------------|
| GET    | `/api/prices/refresh`     | Refresh all portfolio prices      | Yes           |
| GET    | `/api/prices/{ticker}`    | Get current price for ticker      | Yes           |
| GET    | `/api/prices/exchange`    | Get exchange rates                | Yes           |

#### Portfolio

| Method | Endpoint                  | Description                       | Auth Required |
|--------|---------------------------|-----------------------------------|---------------|
| GET    | `/api/portfolio/summary`  | Get portfolio summary             | Yes           |
| GET    | `/api/portfolio/performance` | Get performance metrics        | Yes           |

### Request/Response Examples

#### POST `/api/holdings`

**Request:**
```json
{
  "accountId": "acc_001",
  "ticker": "GOOGL",
  "exchange": "NASDAQ",
  "industry": "Technology",
  "type": "Investment",
  "action": "Buy",
  "quantity": 5,
  "purchasePrice": 140.50,
  "purchaseDate": "2025-10-01T00:00:00Z",
  "currency": "USD"
}
```

**Response (201 Created):**
```json
{
  "id": "hold_123",
  "accountId": "acc_001",
  "ticker": "GOOGL",
  "exchange": "NASDAQ",
  "industry": "Technology",
  "type": "Investment",
  "action": "Buy",
  "quantity": 5,
  "purchasePrice": 140.50,
  "purchaseDate": "2025-10-01T00:00:00Z",
  "totalInvested": 702.50,
  "currentPrice": 145.75,
  "currentValue": 728.75,
  "profitLoss": 26.25,
  "profitLossPercent": 3.74,
  "currency": "USD",
  "lastPriceUpdate": "2025-10-15T10:00:00Z",
  "createdAt": "2025-10-15T10:00:00Z",
  "updatedAt": "2025-10-15T10:00:00Z"
}
```

#### GET `/api/portfolio/summary`

**Response (200 OK):**
```json
{
  "userId": "user_123",
  "totalDeposited": 150000.00,
  "totalInvested": 145000.00,
  "totalCurrentValue": 165000.00,
  "totalCashBalance": 5000.00,
  "totalBalance": 170000.00,
  "totalProfitLoss": 20000.00,
  "totalProfitLossPercent": 13.79,
  "currency": "ZAR",
  "accounts": [
    {
      "accountId": "acc_001",
      "name": "TFSA",
      "deposited": 50000.00,
      "invested": 48000.00,
      "currentValue": 52000.00,
      "cashBalance": 2000.00,
      "profitLoss": 4000.00,
      "profitLossPercent": 8.33,
      "currency": "ZAR"
    }
  ],
  "lastUpdated": "2025-10-15T10:00:00Z"
}
```

### Error Response Format

**Standard Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid ticker symbol provided",
    "details": [
      {
        "field": "ticker",
        "message": "Ticker must be 1-5 uppercase characters"
      }
    ],
    "timestamp": "2025-10-15T10:00:00Z",
    "requestId": "req_abc123"
  }
}
```

**HTTP Status Codes:**
- `200 OK` - Successful GET, PUT
- `201 Created` - Successful POST
- `204 No Content` - Successful DELETE
- `400 Bad Request` - Validation error
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

---

## Security Architecture

### Authentication Flow

**JWT-Based Authentication:**

1. **User Login:**
   - User submits email + password to `/api/auth/login`
   - Backend validates credentials against hashed password in database
   - Generate JWT access token (short-lived, 15 minutes)
   - Generate refresh token (long-lived, 7 days)
   - Return both tokens to client

2. **Token Storage:**
   - Blazor stores tokens in browser's SessionStorage (access token) and LocalStorage (refresh token)
   - Access token sent in `Authorization: Bearer <token>` header

3. **Token Validation:**
   - Azure Functions middleware validates JWT signature
   - Check expiration timestamp
   - Extract user ID from claims
   - Attach user context to request

4. **Token Refresh:**
   - When access token expires, use refresh token to get new access token
   - Call `/api/auth/refresh` with refresh token
   - Receive new access token

### Authorization

**Single-User Model:**
- All data scoped by `userId` from JWT claims
- Database queries automatically filter by `userId`
- No complex role-based access control needed

### Data Security

**Encryption:**
- **In Transit:** TLS 1.3 for all HTTPS connections
- **At Rest:** Azure Cosmos DB automatic encryption
- **Passwords:** bcrypt hashing with salt (cost factor 12)

**CORS Configuration:**
- Allow origin: `https://prosperity.yourdomain.com`
- Allowed methods: GET, POST, PUT, DELETE, OPTIONS
- Allowed headers: Authorization, Content-Type
- Credentials: true (for cookies if needed)

**Secrets Management:**
- Azure Key Vault for storing sensitive configuration
- Database connection strings
- JWT signing key
- External API keys (Alpha Vantage, etc.)
- Environment variables for Azure Functions

### API Rate Limiting

**Considerations:**
- Single-user application (low risk of abuse)
- External API rate limits (Alpha Vantage: 25 requests/day)
- Implement caching to minimize external API calls
- Optional: Rate limit per IP if needed

---

## Data Flow

### User Login Flow

```
User → Blazor WASM → Azure Functions → Cosmos DB
                       (auth/login)
1. User enters credentials
2. Frontend sends POST to /api/auth/login
3. Backend validates password hash
4. Generate JWT tokens
5. Return tokens to frontend
6. Frontend stores tokens
7. Redirect to dashboard
```

### Dashboard Load Flow

```
User → Blazor WASM → Azure Functions → Cosmos DB
                       (portfolio/summary)
1. User navigates to dashboard
2. Frontend sends GET to /api/portfolio/summary (with JWT)
3. Backend validates token, extracts userId
4. Query accounts, holdings from Cosmos DB
5. Calculate totals, P/L, currency conversions
6. Return aggregated summary
7. Frontend renders dashboard with data
```

### Add Holding Flow

```
User → Blazor WASM → Azure Functions → Cosmos DB → External API
                       (holdings)
1. User fills "Add Holding" form
2. Frontend sends POST to /api/holdings
3. Backend validates input
4. Fetch current stock price from Alpha Vantage
5. Calculate initial P/L (likely 0)
6. Insert holding document in Cosmos DB
7. Return created holding to frontend
8. Frontend updates state and refreshes view
```

### Price Refresh Flow

```
User → Blazor WASM → Azure Functions → Cosmos DB → External API
                       (prices/refresh)
1. User clicks "Refresh Prices" button
2. Frontend sends GET to /api/prices/refresh
3. Backend queries all holdings for userId
4. For each unique ticker+exchange:
   a. Check cache (if < 15 min old, use cached)
   b. Otherwise, call Alpha Vantage API
   c. Update cache
5. Update currentPrice, currentValue, P/L for each holding
6. Batch update holdings in Cosmos DB
7. Return updated portfolio summary
8. Frontend re-renders with new values
```

---

## Infrastructure & Deployment

### AWS Infrastructure (Frontend)

**S3 Bucket Configuration:**
- Bucket name: `prosperity-frontend`
- Static website hosting enabled
- Public read access (via bucket policy)
- Versioning enabled for rollback capability

**CloudFront Distribution:**
- Origin: S3 bucket
- Custom domain: `prosperity.yourdomain.com`
- SSL certificate: AWS Certificate Manager (ACM)
- Caching: Default TTL 86400 (1 day)
- Compress objects automatically
- HTTP to HTTPS redirect

**Route 53 DNS:**
- A record alias pointing to CloudFront distribution
- Existing domain integration

### Azure Infrastructure (Backend)

**Resource Group:**
- Name: `rg-prosperity-prod`
- Region: East US (or closest to user)

**Azure Functions App:**
- Name: `func-prosperity-api`
- Runtime: Custom (Go)
- Hosting Plan: Consumption (serverless)
- Storage Account: `stprosperityfunc` (for function metadata)
- Application Insights enabled (monitoring)

**Azure Cosmos DB:**
- Account name: `cosmos-prosperity`
- API: Core (SQL)
- Consistency level: Session (balance of consistency and performance)
- Serverless tier
- Automatic failover: Optional (adds cost)

**Azure Key Vault:**
- Name: `kv-prosperity`
- Store secrets:
  - `CosmosDBConnectionString`
  - `JWTSigningKey`
  - `AlphaVantageAPIKey`
  - `ExchangeRateAPIKey`

**Application Insights:**
- Name: `appi-prosperity`
- Logging and monitoring for Azure Functions
- Custom metrics for API performance

### CI/CD Pipeline (GitHub Actions)

#### Frontend Pipeline (`.github/workflows/deploy-frontend.yml`)

```yaml
name: Deploy Frontend to AWS

on:
  push:
    branches: [main]
    paths:
      - 'src/frontend/**'
      - '.github/workflows/deploy-frontend.yml'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'
      
      - name: Restore dependencies
        run: dotnet restore src/frontend/Prosperity.Web.csproj
      
      - name: Build Blazor WASM
        run: dotnet publish src/frontend/Prosperity.Web.csproj -c Release -o publish
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Deploy to S3
        run: aws s3 sync publish/wwwroot s3://prosperity-frontend --delete
      
      - name: Invalidate CloudFront cache
        run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```

#### Backend Pipeline (`.github/workflows/deploy-backend.yml`)

```yaml
name: Deploy Backend to Azure

on:
  push:
    branches: [main]
    paths:
      - 'src/backend/**'
      - '.github/workflows/deploy-backend.yml'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      
      - name: Build Go functions
        run: |
          cd src/backend
          GOOS=linux GOARCH=amd64 go build -o handler
      
      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Deploy to Azure Functions
        uses: Azure/functions-action@v1
        with:
          app-name: func-prosperity-api
          package: src/backend
```

### Environment Configuration

**Frontend Environment Variables:**
- `BLAZOR_API_BASE_URL`: `https://func-prosperity-api.azurewebsites.net/api`

**Backend Environment Variables (Azure Functions App Settings):**
- `COSMOS_DB_ENDPOINT`: From Key Vault
- `COSMOS_DB_KEY`: From Key Vault
- `JWT_SECRET_KEY`: From Key Vault
- `ALPHA_VANTAGE_API_KEY`: From Key Vault
- `EXCHANGE_RATE_API_KEY`: From Key Vault
- `ALLOWED_ORIGINS`: `https://prosperity.yourdomain.com`
- `TOKEN_EXPIRY_MINUTES`: `15`
- `REFRESH_TOKEN_EXPIRY_DAYS`: `7`

---

## Performance & Scalability

### Performance Targets

- **Page Load Time:** < 2 seconds (first load)
- **API Response Time:** < 500ms (95th percentile)
- **Price Refresh Time:** < 5 seconds (for 50 holdings)
- **Dashboard Render:** < 1 second

### Optimization Strategies

**Frontend:**
- Blazor WASM lazy loading for components
- Virtual scrolling for large holdings lists
- Local state caching (avoid redundant API calls)
- Optimistic UI updates for better perceived performance

**Backend:**
- Price caching (15-minute TTL)
- Batch operations for bulk updates
- Connection pooling for Cosmos DB
- Asynchronous processing for non-critical operations

**Database:**
- Appropriate partition key (`userId`)
- Query optimization (indexed fields)
- Minimize cross-partition queries
- Use Cosmos DB point reads when possible

**CDN & Caching:**
- CloudFront caching for static assets
- Browser caching headers
- Service Worker for offline capability (post-MVP)

### Scalability Considerations

**Current Scale:**
- Single user
- ~50 holdings
- ~500 transactions/year
- Minimal concurrent requests

**Future Scale (if multi-user):**
- Azure Functions scale automatically (Consumption Plan)
- Cosmos DB auto-scales with RU/s demand
- CloudFront handles global distribution
- No changes needed to architecture

---

## Cost Analysis

### AWS Costs (Monthly Estimate)

| Service           | Usage                     | Cost        |
|-------------------|---------------------------|-------------|
| S3 Storage        | 100 MB                    | $0.01       |
| S3 Requests       | 1,000 GET requests        | $0.01       |
| CloudFront        | 1 GB data transfer        | $0.10       |
| Route 53          | 1 hosted zone             | $0.50       |
| **AWS Total**     |                           | **$0.62**   |

### Azure Costs (Monthly Estimate)

| Service                | Usage                          | Cost        |
|------------------------|--------------------------------|-------------|
| Azure Functions        | 100,000 executions, 1 GB-s     | $0.00 (Free tier) |
| Cosmos DB Serverless   | 1M RU/s, 1 GB storage          | $0.25 (Free tier covers most) |
| Key Vault              | 1,000 operations               | $0.03       |
| Application Insights   | 1 GB ingestion                 | $0.00 (Free tier) |
| **Azure Total**        |                                | **$0.28**   |

### External Services

| Service                | Usage                     | Cost        |
|------------------------|---------------------------|-------------|
| Alpha Vantage          | 25 requests/day (free)    | $0.00       |
| ExchangeRate-API       | 1,500 requests/month      | $0.00       |
| **External Total**     |                           | **$0.00**   |

### **Total Monthly Cost: ~$0.90 USD**

**Annual Cost: ~$11 USD**

**Comparison to Traditional Hosting:**
- VPS (DigitalOcean): $6/month = $72/year
- **Savings: $61/year (85% reduction)**

---

## Risk Mitigation

### Technical Risks

| Risk                              | Impact  | Mitigation Strategy                                    |
|-----------------------------------|---------|--------------------------------------------------------|
| Cross-cloud latency               | Medium  | Cache aggressively, optimize API calls, async loading |
| External API rate limits          | High    | Implement caching, fallback to manual entry            |
| Go + Azure Functions complexity   | Medium  | Thorough testing, monitoring, fallback to .NET if needed |
| Cosmos DB cost overruns           | Low     | Serverless tier limits, monitoring alerts              |
| JWT token security                | High    | Short expiry, HTTPS only, secure storage               |
| Blazor WASM browser compatibility | Low     | Target modern browsers, graceful degradation           |

### Operational Risks

| Risk                              | Impact  | Mitigation Strategy                                    |
|-----------------------------------|---------|--------------------------------------------------------|
| Data loss                         | High    | Daily automated backups, export functionality          |
| Service outages (AWS/Azure)       | Medium  | Status monitoring, local data caching                  |
| Vendor lock-in                    | Medium  | Abstract data layer, exportable data formats           |
| Learning curve (Go, Azure)        | Low     | Phased implementation, thorough documentation          |

### Security Risks

| Risk                              | Impact  | Mitigation Strategy                                    |
|-----------------------------------|---------|--------------------------------------------------------|
| Credential compromise             | High    | Strong password requirements, rate limiting            |
| XSS/injection attacks             | Medium  | Input validation, parameterized queries, CSP headers   |
| Man-in-the-middle attacks         | High    | HTTPS everywhere, HSTS, certificate pinning            |
| Unauthorized API access           | High    | JWT validation, userId scoping, audit logging          |

---

## Future Considerations

### Post-MVP Enhancements

**Frontend:**
- Progressive Web App (PWA) with offline support
- Advanced charting library (Chart.js, D3.js)
- Dark mode theme
- Export to PDF/CSV functionality

**Backend:**
- WebSocket support for real-time price updates
- Scheduled Azure Functions for automatic price refresh (Timer trigger)
- Historical price data storage and trending
- Email notifications (dividends, price alerts)

**Database:**
- Time-series data for historical tracking
- Aggregated views for performance optimization

### Multi-User Expansion

If expanding beyond personal use:
- Implement proper role-based access control
- Add user management admin panel


---

## Appendix

### Technology Selection Rationale

#### Why Blazor WASM over React/Angular/Vue?

**Pros:**
- Full C# stack (if backend were .NET)
- Strong typing throughout
- Excellent tooling (Visual Studio, Rider)
- Component reusability
- Learning opportunity for modern .NET

**Cons:**
- Larger initial payload (~2-3 MB)
- Smaller ecosystem than React
- Fewer third-party components

**Decision:** Blazor for C# consistency and .NET ecosystem, despite larger bundle size.

#### Why Go over .NET for Azure Functions?

**Pros:**
- Fast startup times (cold start < 500ms)
- Low memory footprint (cost savings)
- Excellent concurrency for async operations
- Learning opportunity for Go
- Cross-platform binary

**Cons:**
- Less mature Azure Functions support
- Custom runtime handler required
- Smaller community for Azure-specific issues

**Decision:** Go for learning goals, performance, and cost-efficiency. Fallback to .NET if issues arise.

#### Why Azure Functions over AWS Lambda?

**Pros:**
- Learning opportunity for Azure
- Experience with different cloud provider
- Comparable pricing to AWS Lambda
- Good Go support (custom runtime)

**Cons:**
- Cross-cloud complexity
- Potential latency (AWS → Azure)
- Separate credential management

**Decision:** Azure for learning exposure, accepting cross-cloud trade-offs.

#### Why Cosmos DB over SQL Database?

**Pros:**
- Serverless tier (pay-per-request)
- Free tier (1000 RU/s)
- Schema flexibility
- Auto-scaling
- Learning opportunity for NoSQL

**Cons:**
- Less familiar than SQL
- Query limitations vs relational DB
- Potential cost overruns if not monitored

**Decision:** Cosmos DB for cost-effectiveness and NoSQL learning experience.

---

### Glossary

- **WASM:** WebAssembly - binary instruction format for web browsers
- **JWT:** JSON Web Token - token-based authentication standard
- **CORS:** Cross-Origin Resource Sharing - browser security mechanism
- **CDN:** Content Delivery Network - distributed server network for static content
- **RU/s:** Request Units per second - Cosmos DB throughput measure
- **P/L:** Profit/Loss
- **TFSA:** Tax-Free Savings Account (South African investment account)
- **JSE:** Johannesburg Stock Exchange

---

### References

- [Blazor WebAssembly Documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor/)
- [Azure Functions Go Custom Handler](https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-other?tabs=go)
- [Azure Cosmos DB Documentation](https://learn.microsoft.com/en-us/azure/cosmos-db/)
- [AWS S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [AWS CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)
- [Alpha Vantage API](https://www.alphavantage.co/documentation/)

---

**Document Status:** ✅ Approved  
**Last Updated:** October 15, 2025


