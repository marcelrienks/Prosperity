# Prosperity

**Personal Stock Portfolio Management Application**

[![Status](https://img.shields.io/badge/status-planning-blue.svg)](https://github.com/marcelrienks/Prosperity)
[![Phase](https://img.shields.io/badge/phase-design%20complete-green.svg)](docs/project-roadmap.md)
[![Documentation](https://img.shields.io/badge/docs-complete-brightgreen.svg)](docs/README.md)
[![License](https://img.shields.io/badge/license-personal-lightgrey.svg)](#)

> A modern web application to replace Excel-based investment tracking with real-time price updates, automated calculations, and comprehensive portfolio analytics.

---

## 🎯 Project Overview

**Prosperity** is a personal portfolio management application designed to modernize and simplify stock investment tracking. Built as a learning project, it demonstrates a hybrid cloud architecture (AWS + Azure) using modern web technologies including Blazor WebAssembly and Go serverless functions.

### Key Features
- 📊 **Real-time Portfolio Dashboard** - Instant view of total value, P/L, and performance across all accounts
- 💼 **Multi-Account Management** - Organize investments across TFSA, US, ZA, EUR, and Property accounts
- 📈 **Live Price Updates** - Automatic stock price fetching via Google Finance API
- 💱 **Multi-Currency Support** - USD, EUR, GBP with automatic conversion to ZAR
- 💰 **Cash Flow Tracking** - Deposits, transfers, and available balances per account
- 📥 **Import/Export** - Bulk CSV import/export for data portability
- 🌓 **Light/Dark Themes** - Customizable appearance
- ⚙️ **Configurable Settings** - Customize exchanges, industries, types, and actions

---

## 🏗️ Architecture Overview

Prosperity uses a **hybrid cloud architecture** combining AWS for static hosting and Azure for serverless backend:

```
┌─────────────┐
│   Browser   │ (Blazor WebAssembly)
└──────┬──────┘
       │
       ├─────────→ AWS S3 + CloudFront (Static Assets)
       │
       └─────────→ Azure Functions (Go) ─→ Cosmos DB
                   ├─→ Google Finance API
                   └─→ ExchangeRate-API
```

### Technology Stack

**Frontend**
- Blazor WebAssembly (.NET 8)
- MudBlazor UI Components
- Fluxor State Management
- Hosted on AWS S3 + CloudFront

**Backend**
- Azure Functions (Go 1.21+)
- Serverless / Consumption Plan
- JWT Authentication
- RESTful API

**Data**
- Azure Cosmos DB (NoSQL)
- JSON Document Storage
- Partitioned by userId

**External APIs**
- Google Finance API (Real-time Stock Prices & Currency Conversion)

**Infrastructure**
- AWS: S3, CloudFront, Route 53
- Azure: Functions, Cosmos DB, Key Vault, Application Insights
- CI/CD: GitHub Actions

[📐 View Detailed Architecture](docs/diagrams/system-architecture.md)

---

## 📚 Documentation

Comprehensive documentation is available in the [`docs/`](docs/) directory:

### Core Documents

| Document | Description | Status |
|----------|-------------|--------|
| [**Product Requirements Document**](docs/product-requirements-document.md) | Complete feature specifications, user stories, acceptance criteria | ✅ Complete |
| [**Architecture Design**](docs/architecture-design.md) | Technical architecture, technology decisions, infrastructure setup | ✅ Approved |
| [**Stakeholder Requirements**](docs/stakeholder-requirements.md) | Business requirements based on current Excel workflow | ✅ Complete |
| [**Competitive Analysis**](docs/competitive-analysis.md) | Market research and feature comparison | ✅ Complete |
| [**Project Roadmap**](docs/project-roadmap.md) | Phases, milestones, and timeline | ✅ Complete |

### Visual Documentation

| Category | Diagrams | Description |
|----------|----------|-------------|
| [**System Architecture**](docs/diagrams/system-architecture.md) | 5 diagrams | High-level architecture, components, database schema, deployment |
| [**Use Case Flows**](docs/diagrams/use-case-flows.md) | 10 diagrams | User workflows, authentication, CRUD operations, data flows |
| [**Data Flow**](docs/diagrams/data-flow.md) | 10 diagrams | State management, calculations, currency conversion, API cycles |

📖 **[Complete Documentation Index](docs/README.md)**

---

## 🚀 Getting Started

### Prerequisites

- .NET 8 SDK or later
- Go 1.21+ (for backend development)
- Node.js 18+ (for tooling)
- Azure account (free tier)
- AWS account (free tier)
- Google Finance API access
- ExchangeRate-API key (free tier)

### Quick Start

**Clone the repository:**
```bash
git clone https://github.com/marcelrienks/Prosperity.git
cd Prosperity
```

**Review documentation:**
```bash
# Open documentation in your browser
start docs/README.md
```

**Set up development environment:**
```bash
# Frontend (Blazor WASM)
cd src/frontend
dotnet restore
dotnet build

# Backend (Azure Functions - Go)
cd src/backend
go mod download
go build
```

> **Note:** Detailed setup instructions will be added during core development.

---

## 📖 Key Concepts

### Accounts
Organize investments across multiple account types:
- **TFSA** - Tax-Free Savings Account
- **ZA** - South African stocks (JSE)
- **US** - US stocks (NYSE, NASDAQ)
- **EUR** - European stocks
- **Prop** - Property shares and REITs

### Holdings Management
- Track stock purchases with quantity, price, and date
- **Average cost basis** calculated automatically for multiple purchases
- Real-time price updates
- Profit/Loss calculations (amount and percentage)
- Multi-currency support

### Cash Flow Tracking
- **Deposits:** Capital added to accounts
- **Transfers:** Money moved between accounts
- **Cash Balance:** Uninvested funds per account
- Complete transaction history

### Multi-Currency Portfolio
- Holdings in USD, EUR, GBP, ZAR
- Automatic currency conversion using live exchange rates
- Display in original currency or converted to ZAR
- FX gain/loss tracking (future)

---

## 💡 Use Cases

### Primary Workflows

1. **View Portfolio Summary**
   - See total portfolio value, P/L, and performance at a glance
   - View breakdown by account
   - Check recent price changes

2. **Add New Investment**
   - Record stock purchase (ticker, quantity, price, date)
   - Automatic price fetching and P/L calculation
   - Assign to specific account

3. **Record Deposits/Transfers**
   - Log capital deposits into accounts
   - Transfer funds between accounts
   - Maintain accurate cash balances

4. **Refresh Prices**
   - Manual refresh button for latest prices
   - Batch update all holdings
   - Recalculate portfolio values and P/L

5. **Import/Export Data**
   - Bulk import holdings from CSV
   - Export portfolio data for backup
   - Migrate from existing Excel system

[📋 View Detailed Use Case Flows](docs/diagrams/use-case-flows.md)

---

## 🎨 Screenshots & Mockups

> Screenshots and mockups will be added during core development.

**Planned Pages:**
- Dashboard (Portfolio Overview)
- Investments (Holdings Management)
- Deposits & Transfers (Cash Flow)
- Settings (Configuration)

---

## � Project Documentation

### Completed Planning
- ✅ Competitive analysis
- ✅ Stakeholder requirements gathering
- ✅ MVP feature definition
- ✅ User stories and acceptance criteria
- ✅ System architecture design
- ✅ Technology selection and justification
- ✅ Database schema design
- ✅ API design and specification
- ✅ Security and compliance planning
- ✅ Visual documentation (25 diagrams)

[� View Project Status](docs/project-roadmap.md)

---

## 📊 Architecture Summary

### Documentation Statistics
- **Total Pages:** ~145 pages
- **Diagrams:** 25 Mermaid diagrams
- **User Stories:** 20+ stories
- **API Endpoints:** 15+ endpoints

### Architecture Components
- **Frontend Components:** 10+ pages and shared components
- **Backend Handlers:** 5 main handler groups
- **Database Collections:** 4 main collections
- **External Integrations:** 2 APIs

### Costs
- **Estimated Monthly Cost:** ~$0.90 USD
  - AWS: $0.62/month
  - Azure: $0.28/month
  - External APIs: Free tier
- **Comparison:** 85% savings vs. traditional VPS hosting

---

## 🛠️ Development

### Project Structure

```
Prosperity/
├── docs/                          # Documentation
│   ├── product-requirements-document.md
│   ├── architecture-design.md
│   ├── stakeholder-requirements.md
│   ├── competitive-analysis.md
│   ├── project-roadmap.md
│   ├── README.md                  # Documentation index
│   └── diagrams/                  # Mermaid diagrams
│       ├── system-architecture.md
│       ├── use-case-flows.md
│       ├── data-flow.md
│       └── README.md
├── src/                           # Source code (to be created)
│   ├── frontend/                  # Blazor WASM
│   └── backend/                   # Azure Functions (Go)
├── tests/                         # Tests (to be created)
└── README.md                      # This file
```

### Development Workflow

1. **Branch from main** for new features
2. **Write tests** (TDD approach)
3. **Implement feature** following architecture
4. **Update documentation** if needed
5. **Create pull request** with description
6. **Review and merge** to main
7. **Deploy** via GitHub Actions

### Coding Standards

- **Frontend (C#):** Follow .NET conventions, use async/await
- **Backend (Go):** Follow Go best practices, effective error handling
- **Commits:** Conventional commit messages (feat:, fix:, docs:)
- **Documentation:** Keep diagrams and docs in sync with code

---

## 🔐 Security

### Authentication
- JWT-based authentication
- Short-lived access tokens (15 minutes)
- Long-lived refresh tokens (7 days)
- Secure password hashing (bcrypt)

### Data Protection
- **In Transit:** TLS 1.3 encryption (HTTPS everywhere)
- **At Rest:** Azure Cosmos DB automatic encryption
- **Secrets:** Azure Key Vault for API keys and connection strings

### Access Control
- All data scoped by `userId`
- No cross-user data access
- JWT validation on all API requests

[🔒 View Security Architecture](docs/architecture-design.md#security-architecture)

---

## 💰 Cost Analysis

### Monthly Operating Costs

| Service | Usage | Cost |
|---------|-------|------|
| **AWS S3** | 100 MB storage, 1K requests | $0.02 |
| **AWS CloudFront** | 1 GB transfer | $0.10 |
| **AWS Route 53** | 1 hosted zone | $0.50 |
| **Azure Functions** | 100K executions | $0.00 (free tier) |
| **Azure Cosmos DB** | 1M RU/s, 1 GB | $0.25 (free tier) |
| **Azure Key Vault** | 1K operations | $0.03 |
| **Google Finance API** | 25 requests/day | $0.00 (free) |
| **ExchangeRate-API** | 1.5K requests/month | $0.00 (free) |
| **Total** | | **~$0.90/month** |

**Annual:** ~$11 USD (vs. $72/year for traditional hosting = 85% savings)

[💵 View Detailed Cost Analysis](docs/architecture-design.md#cost-analysis)

---

## 🤝 Contributing

This is currently a **personal learning project** not open for external contributions. However, feedback and suggestions are welcome!

### Feedback
- Create an issue for bugs or feature requests
- Share ideas for improvements
- Report documentation errors

---

## 📝 License

This is a personal project developed for individual use and learning purposes.

**Copyright © 2025 Marcel Rienks. All rights reserved.**

---

## 📧 Contact

**Project Owner:** Marcel Rienks

**Repository:** [github.com/marcelrienks/Prosperity](https://github.com/marcelrienks/Prosperity)

---

## 🙏 Acknowledgments

### Technologies
- [Blazor WebAssembly](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor) - Microsoft
- [MudBlazor](https://mudblazor.com/) - Open source Blazor component library
- [Azure Functions](https://azure.microsoft.com/services/functions/) - Microsoft
- [Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) - Microsoft
- [Go](https://go.dev/) - Google
- [Google Finance API](https://www.google.com/finance) - Google
- [ExchangeRate-API](https://www.exchangerate-api.com/) - ExchangeRate-API

### Learning Resources
- [Microsoft Learn](https://learn.microsoft.com/)
- [Go by Example](https://gobyexample.com/)
- [Azure Architecture Center](https://learn.microsoft.com/azure/architecture/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

## 📚 Further Reading

### Documentation Deep Dives
- **Feature Specifications:** [Product Requirements Document](docs/product-requirements-document.md)
- **Technical Architecture:** [Architecture Design Document](docs/architecture-design.md)
- **Visual Guides:** [System Diagrams](docs/diagrams/README.md)
- **Project Planning:** [Roadmap & Milestones](docs/project-roadmap.md)

### Key Concepts
- **Average Cost Basis:** [Data Flow Diagram](docs/diagrams/data-flow.md#average-cost-basis-calculation)
- **Multi-Currency:** [Currency Conversion Flow](docs/diagrams/data-flow.md#multi-currency-conversion-flow)
- **Cash Balances:** [Cash Management Flow](docs/diagrams/data-flow.md#cash-balance-management-flow)
- **State Management:** [Fluxor Pattern](docs/diagrams/data-flow.md#state-management-fluxor)

---

## 🎓 Learning Goals

This project serves as a hands-on learning experience for:

✅ **Modern Web Development**
- Blazor WebAssembly and .NET 8
- Component-based architecture
- State management patterns (Redux/Fluxor)

✅ **Backend Development**
- Go programming language
- Serverless architecture
- RESTful API design

✅ **Cloud Computing**
- Hybrid cloud architecture (AWS + Azure)
- Serverless functions
- NoSQL databases (Cosmos DB)
- Cloud storage and CDN

✅ **DevOps**
- CI/CD pipelines (GitHub Actions)
- Infrastructure as Code
- Monitoring and observability

✅ **Software Engineering**
- Requirements gathering
- Architecture design
- API design
- Security best practices
- Cost optimization

---

## 📈 Success Metrics

The project will be considered successful if it:

✅ **Functional**
- Replicates 100% of Excel functionality
- Provides real-time price updates
- Calculates all metrics accurately
- Handles multi-currency conversions correctly

✅ **Performance**
- Faster than current Excel workflow
- Page loads in < 3 seconds
- Price refresh completes in < 10 seconds

✅ **Quality**
- Data security maintained
- Fully responsive (desktop and mobile)
- No data loss
- Bulk import/export capability

✅ **Learning**
- Practical experience with Blazor, Go, and cloud platforms
- Understanding of hybrid cloud architectures
- Exposure to modern web development practices

---

## 🔮 Future Vision

### Post-MVP Features
- 📊 Advanced charting (Chart.js, D3.js)
---

## 📝 Outstanding Planning Items

The following items require further definition before or during implementation:

### 4. UI/UX Specifications
**Current Status:** Partially complete (TBD items found)

**What's Missing:**
- Brand colors (Primary colors marked as "TBD")
- Detailed wireframes/mockups
- Component library decisions (MudBlazor vs Radzen final choice)
- Responsive breakpoint specifications
- Icon library selection
- Loading states specifications
- Animation/transition guidelines

**Recommendation:**
- Finalize color scheme decisions
- Create wireframes or mockups for key pages
- Document component styling standards

---

### 9. Error Handling Standards
**Current Status:** General error handling mentioned

**What's Missing:**
- Specific error codes/categories
- Error message standards
- Retry logic specifications
- Circuit breaker patterns
- User-facing error message guidelines

**Recommendation:**
Create `docs/error-handling-standards.md` with:
- Error code catalog
- Error response format specifications
- Retry and timeout policies

---

### 10. API Documentation
**Current Status:** API endpoints listed but not fully documented

**What's Missing:**
- Complete OpenAPI/Swagger specification
- Request/response examples for all endpoints
- Error response documentation
- Rate limiting details
- API versioning strategy

**Recommendation:**
- Create comprehensive API specification
- Document all request/response formats
- Define error response standards

---

<div align="center">

**Built with ❤️ as a learning project**

[View Documentation](docs/README.md) • [Architecture](docs/architecture-design.md) • [Diagrams](docs/diagrams/README.md) • [Roadmap](docs/project-roadmap.md)

---

⭐ **Star this repository** if you find the documentation useful!

</div>
