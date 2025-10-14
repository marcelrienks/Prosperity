# Personal Requirements Document

**Date:** October 14, 2025
**Author:** Marcel Rienks

---

## 1. Asset Types to Support
- Stocks (JSE-listed, international)
- ETFs (local and global)
- Property shares (REITs, fractional property platforms)
- Unit trusts
- Cash accounts

## 2. Currency Support and Conversion
- ZAR (primary)
- USD, EUR, GBP (for international holdings)
- Automatic FX conversion for reporting
- Manual override for custom rates

## 3. Reporting Requirements
- Portfolio value over time (line graph)
- Asset allocation (pie chart)
- Performance tracking (annualized, absolute)
- Tax report (capital gains, dividends)
- Dividend income tracking
- Downloadable CSV/PDF reports

## 4. Account Customization
- Custom account/portfolio names
- Custom icons or color tags
- Grouping by account type or goal

## 5. Data Import/Export
- CSV import for transactions and holdings
- Export to CSV and PDF
- API integration (future)

## 6. Notification Preferences
- Price alerts (per asset)
- Dividend alerts
- Portfolio value threshold alerts
- Email and in-app notifications

## 7. Authentication & Security
- Email/password login
- Optional 2FA (TOTP)
- Data encryption at rest and in transit

## 8. Data Refresh & Historical Data
- Daily refresh (minimum)
- On-demand manual refresh
- 5+ years of historical data

## 9. Mobile/Responsive Design
- Fully responsive web app

---

## South African Investment Account Types
- Stocks (JSE, EasyEquities, etc.)
- TFSA (Tax-Free Savings Account)
- Property share platforms (EasyProperties, Wealth Migrate)

---

## Feature Prioritization Matrix
| Feature                        | MVP | Future |
|--------------------------------|-----|--------|
| Stocks, ETFs, tracking         |  X  |        |
| Property share tracking        |  X  |        |
| Multi-currency support         |  X  |        |
| Reporting (basic graphs)       |  X  |        |
| Tax & dividend reports         |     |   X    |
| Custom account names/icons     |  X  |        |
| CSV import/export              |  X  |        |
| API integrations               |     |   X    |
| Price/dividend alerts          |  X  |        |
| 2FA, encryption                |     |   X    |

---

## MVP Scope Definition
- Track stocks, ETFs, crypto, property shares
- ZAR and major foreign currency support
- Basic reporting (value, allocation, performance)
- CSV import/export
- Price/dividend alerts
- Secure authentication (email/password, 2FA)
- Responsive

---

## User Stories (Core Features)
- As a user, I want to add and track multiple investment accounts so I can see my total net worth.
- As a user, I want to import my holdings from CSV so I can onboard quickly.
- As a user, I want to receive price and dividend alerts so I can act on market changes.
- As a user, I want to view my portfolio value and allocation over time so I can monitor performance.
- As a user, I want to customize account names and icons so I can organize my investments.
- As a user, I want secure login and 2FA so my data is protected.
- As a user, I want to use the app on my phone and desktop so I can access it anywhere.

---

## Technical Preferences
- Manual data entry and CSV import for MVP
- API integrations for automation in future

## Privacy & Security Requirements
- All data encrypted at rest and in transit
- No third-party data sharing
- User data export and deletion on request

---

*End of document*