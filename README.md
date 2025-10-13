# Prosperity: Personal Stock Portfolio Manager

**Prosperity** is a web-based application designed to help users track and manage their personal investment portfolios across multiple accounts and asset types. The application combines a modern .NET front end with scalable AWS Lambda (.NET Core) services and a cloud-native backend, all orchestrated via AWS CloudFormation and secured under a custom subdomain using AWS Route 53. This document describes the high-level functionality, technology stack, and a phased plan to deliver a robust portfolio management solution.

---

## Core Features

- **Multi-Account Support**: Organize your investments by region, stock exchange, account type (e.g., TFSA), or even unique categories like property shares.
- **Account Pages**: Each account has a dedicated page showing detailed information for all holdings.
- **Portfolio Overview**: Aggregate view of all accounts, displaying total deposited, invested, current value, and profits.
- **Stock Tracking**: 
  - Ticker symbol
  - Short company description
  - Current stock price (fetched live via Google Finance API)
  - Purchase details (amount, price, date) stored in a local database
  - Real-time calculation of current value and profit/loss
- **Authentication**: Secure login required for all site access.
- **Cloud Native Deployment**: Infrastructure as code (CloudFormation), public subdomain under prosperity, and DNS management with AWS Route 53.

---

## Project Plan

### 1. Competitive Analysis

**Objective:** Understand the landscape and gather inspiration from popular stock portfolio managers.  
**Action Items:**
- Research and document core features, UX/UI approaches, and pricing models of leading solutions such as:
  - Personal Capital
  - Yahoo Finance Portfolio
  - Sharesight
  - Morningstar Portfolio Manager
  - Delta Investment Tracker
- Identify strengths and weaknesses, focusing on:
  - Account types supported
  - Real-time pricing accuracy
  - Reporting and analytics
  - Security and privacy measures
  - User onboarding and authentication flows

### 2. Stakeholder Questionnaire

**Objective:** Gather detailed requirements to inform the Product Requirements Document (PRD).

**Sample Questions:**
- What types of assets should be supported (stocks, ETFs, crypto, property, etc.)?
- Should the app support currencies and currency conversion?
- What level of reporting is needed (graphs, tax reports, performance over time)?
- What customizations should users have per account (naming, icons, grouping)?
- Should users be able to import/export portfolio data?
- What notification features are desired (price alerts, dividend alerts, etc.)?
- What are the authentication requirements (OAuth, 2FA, SSO)?
- How often should pricing data be refreshed? Is historical data required?
- Should there be mobile support or a responsive web design?
- Are there any compliance or regulatory considerations?
- What are the preferred third-party integrations (e.g., Google Finance, broker APIs)?

### 3. Architecture Brainstorm

**Objective:** Design a scalable, maintainable, and secure system architecture.

**Components:**
- **Frontend:** ASP.NET web application (Razor Pages or Blazor), responsive design.
- **Backend Service Layer:** AWS Lambda (.NET Core) for business logic and integration (pricing, account management).
- **Database:** AWS RDS (SQL or NoSQL depending on requirements) for portfolio, account, and transaction data.
- **Authentication:** AWS Cognito or custom .NET Identity (integrated with Lambda and frontend).
- **Pricing API Integration:** Google Finance (scraping or third-party API), with caching for rate limiting.
- **Infrastructure:** AWS CloudFormation templates for automated resource provisioning.
- **Domain & DNS:** AWS Route 53 for subdomain management under prosperity.
- **Security:** HTTPS everywhere, IAM roles for Lambda, encrypted data at rest and in transit.

### 4. Full Project Roadmap

**Phase 1: Discovery & Requirements**
- Conduct detailed competitor analysis.
- Complete stakeholder questionnaire and finalize PRD.
- Define MVP scope.

**Phase 2: Architecture & Design**
- Finalize architecture diagrams and technology selections.
- Design database schema and API contracts.
- Create wireframes and UI prototypes.

**Phase 3: Core Development**
- Set up CloudFormation scripts for base infrastructure.
- Implement authentication and account management flows.
- Develop the portfolio and account pages.
- Integrate live stock pricing.
- Create backend Lambda functions for business logic.
- Set up DB schema and data access layers.

**Phase 4: Testing & QA**
- Unit and integration tests for all modules.
- Manual QA of UI/UX flows.
- Security and performance testing.

**Phase 5: Deployment**
- Configure Route 53 and deploy to public subdomain.
- Enable HTTPS and security best practices.
- Set up monitoring and logging.

**Phase 6: Iteration & Feature Expansion**
- Add advanced features (alerts, reporting, imports/exports, mobile/responsive enhancements).
- Gather user feedback for continuous improvement.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

This project is licensed under the MIT License.
