# Prosperity - Diagrams Index

**Version:** 1.0  
**Date:** October 15, 2025  
**Purpose:** Central index for all system diagrams and visual documentation

---

## Overview

This directory contains comprehensive visual documentation for the Prosperity portfolio management application, including architecture diagrams, use case flows, data flow patterns, and state management visualizations.

All diagrams are created using [Mermaid](https://mermaid.js.org/), which allows them to be rendered directly in Markdown files on GitHub and most modern documentation platforms.

---

## Diagram Categories

### 1. System Architecture
**File:** [system-architecture.md](system-architecture.md)

Visual representations of the system's technical architecture, deployment structure, and component organization.

**Diagrams included:**
- **High-Level System Architecture** - Complete hybrid AWS/Azure cloud architecture showing all major components and their interactions
- **Component Architecture** - Detailed breakdown of frontend (Blazor WASM) and backend (Azure Functions) layers with services and handlers
- **Database Schema Relationships** - Entity relationship diagram showing Cosmos DB collections and their relationships
- **Deployment Architecture** - CI/CD pipeline flow with GitHub Actions deploying to AWS and Azure
- **Technology Stack Overview** - High-level view of all technologies used across the stack

**Best for:**
- Understanding the overall system design
- Seeing how AWS and Azure components interact
- Reviewing deployment and infrastructure setup
- Technology stack at a glance

---

### 2. Use Case Flows
**File:** [use-case-flows.md](use-case-flows.md)

Step-by-step visual representations of key user workflows and feature interactions.

**Diagrams included:**
- **User Authentication Flow** - Complete login sequence with JWT token generation
- **Dashboard Load Flow** - How portfolio data is fetched, calculated, and displayed
- **Add New Holding Flow** - Process of adding a stock purchase, including price fetching and average cost calculation
- **Panel Expansion & Background Refresh Flow** - Hierarchical panel system with smart background refresh
- **Record Deposit Flow** - Depositing money into an account and updating balances
- **Record Transfer Flow** - Transferring money between accounts with balance validation
- **Edit Holding (Multiple Purchases)** - How average cost basis is calculated for additional stock purchases
- **Data Import/Export Flow** - CSV import/export process with validation and error handling
- **User Journey - First Time Setup** - Complete onboarding flow for new users
- **Error Handling Flow** - Comprehensive error handling logic across all operations

**Best for:**
- Understanding user workflows
- Documenting feature behavior
- Planning user experience improvements
- Troubleshooting specific use cases

---

### 3. Data Flow & State Management
**File:** [data-flow.md](data-flow.md)

Detailed visualizations of how data flows through the system and how application state is managed.

**Diagrams included:**
- **Data Flow Overview** - Complete data flow from user action through frontend, backend, database, and external APIs
- **State Management (Fluxor)** - Redux-pattern state management in Blazor using Fluxor
- **Example State Structure** - Application state tree showing authentication, portfolio, and UI state
- **Portfolio Calculation Flow** - Step-by-step calculation of holdings, account summaries, and portfolio totals
- **Cash Balance Management Flow** - How deposits, transfers, and purchases affect account cash balances
- **Cash Balance Formula** - Mathematical representation of cash balance calculation
- **Multi-Currency Conversion Flow** - Currency conversion process using ExchangeRate-API with real-time fetching
- **Currency Conversion Example** - Concrete example of multi-currency portfolio valuation
- **Average Cost Basis Calculation** - Detailed breakdown of averaging multiple stock purchases
- **API Request/Response Cycle** - Complete sequence of an API call with Fluxor state updates

**Best for:**
- Understanding data transformations
- Debugging calculation logic
- Optimizing data fetching strategies
- Learning state management patterns

---

## Quick Reference

### By Diagram Type

#### Architecture Diagrams
- System Architecture → [system-architecture.md](system-architecture.md#high-level-system-architecture)
- Component Architecture → [system-architecture.md](system-architecture.md#component-architecture)
- Database Schema → [system-architecture.md](system-architecture.md#database-schema-relationships)
- Deployment Pipeline → [system-architecture.md](system-architecture.md#deployment-architecture)

#### Sequence Diagrams
- User Authentication → [use-case-flows.md](use-case-flows.md#user-authentication-flow)
- Dashboard Load → [use-case-flows.md](use-case-flows.md#dashboard-load-flow)
- Add Holding → [use-case-flows.md](use-case-flows.md#add-new-holding-flow)
- Panel Expansion & Background Refresh → [use-case-flows.md](use-case-flows.md#panel-expansion--background-refresh-flow)
- API Request/Response → [data-flow.md](data-flow.md#api-requestresponse-cycle)

#### Flowcharts
- Data Flow → [data-flow.md](data-flow.md#data-flow-overview)
- Portfolio Calculations → [data-flow.md](data-flow.md#portfolio-calculation-flow)
- Cash Balance → [data-flow.md](data-flow.md#cash-balance-management-flow)
- Currency Conversion → [data-flow.md](data-flow.md#multi-currency-conversion-flow)
- Error Handling → [use-case-flows.md](use-case-flows.md#error-handling-flow)
- Import/Export → [use-case-flows.md](use-case-flows.md#data-importexport-flow)

#### Entity Relationship Diagrams
- Database Schema → [system-architecture.md](system-architecture.md#database-schema-relationships)

---

## Common Use Cases

### For Development
**Starting a new feature?**
1. Review relevant use case flow to understand existing patterns
2. Check data flow diagrams to understand state management
3. Review component architecture to identify where code belongs

**Debugging an issue?**
1. Start with the use case flow to identify the expected behavior
2. Follow the data flow diagram to trace the problem
3. Check error handling flow for proper error management

### For Documentation
**Onboarding new developers?**
1. Start with system architecture overview
2. Review technology stack diagram
3. Walk through key use case flows
4. Deep dive into data flow and state management

**Creating user documentation?**
1. Use use case flows as reference
2. Simplify sequence diagrams for user guides
3. Extract key steps from flowcharts

### For Planning
**Planning a new feature?**
1. Create new use case flow diagram
2. Identify impacted components in architecture
3. Plan data flow changes
4. Consider error scenarios

**Refactoring?**
1. Review current architecture diagrams
2. Document proposed changes in new diagrams
3. Compare before/after flows
4. Validate against use cases

---

## Diagram Conventions

### Colors
- **Blue** (`#e1f5ff`) - User-facing components, inputs, start points
- **Orange** (`#ff9900`) - AWS services
- **Azure Blue** (`#0078d4`) - Azure services
- **Green** (`#d4edda`) - Data stores, successful outcomes, end points
- **Yellow** (`#fff3cd`) - Processing, business logic, decisions
- **Red** (`#f8d7da`) - Errors, negative outcomes, destructive actions
- **Gray** (`#e7e8ea`, `#f8f9fa`) - External services, neutral elements

### Shapes
- **Rectangles** - Processes, services, components
- **Rounded Rectangles** - UI elements, user interactions
- **Cylinders** - Databases, data stores
- **Diamonds** - Decision points, conditionals
- **Circles** - Start/end points in flowcharts
- **Actors** - Users or external systems in sequence diagrams

### Line Styles
- **Solid Lines** (`-->`) - Primary flow, synchronous calls
- **Dotted Lines** (`-.->`) - Secondary flow, optional paths
- **Bold Lines** (`==>`) - Critical paths, important flows

---

## Viewing Diagrams

### In GitHub
All Mermaid diagrams render automatically when viewing Markdown files on GitHub.

### In VS Code
Install the **Mermaid Preview** extension:
1. Open Extensions (Ctrl+Shift+X)
2. Search for "Mermaid Preview"
3. Install and reload
4. Open any `.md` file and use the preview pane

### Standalone
Use [Mermaid Live Editor](https://mermaid.live/) to view and edit diagrams online.

---

## Maintenance

### Updating Diagrams
When updating diagrams:
1. Ensure all node IDs are unique within the diagram
2. Test rendering in GitHub or Mermaid Live Editor
3. Update this index if adding new diagram categories
4. Keep colors consistent with conventions above
5. Add comments in complex diagrams for clarity

### Adding New Diagrams
When creating new diagrams:
1. Choose appropriate diagram type (flowchart, sequence, ER, etc.)
2. Follow existing naming conventions
3. Add to relevant documentation file
4. Update this index with new diagram reference
5. Include description of what the diagram represents

---

## Related Documentation

- **Product Requirements Document** - [../product-requirements-document.md](../product-requirements-document.md)
- **Architecture Design** - [../architecture-design.md](../architecture-design.md)
- **Stakeholder Requirements** - [../stakeholder-requirements.md](../stakeholder-requirements.md)
- **Project Roadmap** - [../project-roadmap.md](../project-roadmap.md)
- **Competitive Analysis** - [../competitive-analysis.md](../competitive-analysis.md)

---

## Diagram Statistics

| File | Diagrams | Type Distribution |
|------|----------|-------------------|
| system-architecture.md | 5 | Flowcharts (4), Tech Stack (1) |
| use-case-flows.md | 10 | Sequence (6), Flowcharts (4) |
| data-flow.md | 10 | Flowcharts (8), Sequence (2) |
| **Total** | **25** | **Sequence (8), Flowcharts (16), ER (1)** |

---

**Last Updated:** October 15, 2025  
**Maintained By:** Marcel Rienks  
**Mermaid Version:** 11.1.0+

---

*For questions or suggestions about these diagrams, please refer to the main project documentation or create an issue in the repository.*
