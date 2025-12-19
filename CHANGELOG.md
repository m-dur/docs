# Changelog

All notable changes to the Financial Data Fetcher project are documented in this file, based on actual git history from the main project repository.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## Git History Commands

```bash
# View full commit history
git log --oneline

# View commits for specific month
git log --format="%h %ad %s" --date=short --since="2025-07-01" --until="2025-08-01"

# View files changed in a commit
git show <commit-hash> --stat --name-only

# View commits per month
git log --format="%ad" --date=format:"%Y-%m" | sort | uniq -c
```

---

## December 2025

**Commits**: 5 | **Key Focus**: Data lineage visualization, expense charts

### Added
- `ModelExplorerPage.jsx` - ReactFlow-based data lineage visualization showing API routes → database tables → frontend pages (33dbed9)
- `SchemaPage.jsx` - Database schema visualization page (33dbed9)

### Changed
- Expense chart improvements (28e2c8b)
- Query performance optimizations for downstream data (eecdac3, e8b4734)
- Sidebar navigation updates for new pages (33dbed9)

### Key Commits
| Hash | Date | Description |
|------|------|-------------|
| 28e2c8b | 2025-12-17 | expense chart changes |
| 79408f9 | 2025-12-07 | misc lineage chart |
| 33dbed9 | 2025-12-01 | linage chart - Added ModelExplorerPage |
| eecdac3 | 2025-11-30 | query performance - down stream |

---

## November 2025

**Commits**: 8 | **Key Focus**: Performance optimizations, misc improvements

### Changed
- Query performance improvements (eecdac3, e8b4734)
- Various bug fixes and improvements

### Key Commits
| Hash | Date | Description |
|------|------|-------------|
| 9f9b837 | 2025-11-30 | misc |
| 2125723 | 2025-11-29 | misc |
| fb23072 | 2025-11-24 | misc |
| f90b05a | 2025-11-18 | misc |
| 77ebdc3 | 2025-11-14 | misc |

---

## October 2025

**Commits**: 3 | **Key Focus**: Transaction filtering, balance tracking

### Added
- Top N transactions feature for analytics (eca2b33, 990bc53)

### Changed
- Balance tracking improvements (340307e)

### Key Commits
| Hash | Date | Description |
|------|------|-------------|
| eca2b33 | 2025-10-26 | top n transactions |
| 340307e | 2025-10-18 | balance changes |

---

## September 2025

**Commits**: 2 | **Key Focus**: Chart improvements

### Changed
- Balance tracking updates (f8f0574)
- Graph scaling improvements (51e4353)

### Key Commits
| Hash | Date | Description |
|------|------|-------------|
| f8f0574 | 2025-09-19 | balance changes |
| 51e4353 | 2025-09-04 | graph scaling |

---

## August 2025

**Commits**: 6 | **Key Focus**: Stock price reliability, investment charts

### Added
- `daily_stock_price_updater_dag.py` - New DAG for 7-day stock price backfill (7a3fa11)
- Investment chart crosshair feature (d637311)
- Multiple data validation and backfill scripts (283871f, 7a3fa11)

### Fixed
- Investment chart price history gaps (d637311)
- Stock price data accuracy issues (7a3fa11)
- Investment chart filter bugs (283871f)

### Changed
- Payment history filtering improvements (4cedb9f)
- Stock table display enhancements (630e1ae)

### Key Commits
| Hash | Date | Description |
|------|------|-------------|
| 4cedb9f | 2025-08-30 | payment history filtering |
| d637311 | 2025-08-23 | Fix investment chart price history gaps and add crosshair |
| 7a3fa11 | 2025-08-03 | fixing stock prices - Added daily_stock_price_updater_dag |
| 283871f | 2025-08-01 | investment chart filter fix |

---

## July 2025

**Commits**: 30 | **Key Focus**: Major feature development (Transfer Detection, Payment Mappings, Net Worth improvements)

### Added - Transfer Detection System (4f464aa)
- `investment_transfers.py` - Backend route for bank-to-investment transfer detection
- `TransferDetectionBanner.jsx` - UI component for transfer alerts
- `FrontendTransferBanner.jsx` - Frontend-based detection alternative
- `useTransferDetection.js` - Custom React hook for transfer pattern matching
- `categories.py` - Category management routes
- Database tables: `custom_transfer_patterns`, `detected_investment_transfers`

### Added - Payment Mapping System (5db8a8b)
- `mapped_payments.py` - Payment pattern mapping routes
- `payment_mappings.py` - Payment mappings management
- `transaction_mappings.py` - Transaction categorization API
- `PaymentMappingsPage.jsx` - UI for managing payment patterns

### Added - Net Worth & Analytics (d614f94, 1bc51ad)
- `bank_balance_history.py` - Account balance history tracking
- `schema_routes.py` - Database schema introspection
- `route_discovery.py` - Auto-generated API documentation (629e1d6)
- `DailyChangeBreakdown.jsx` - Net worth daily change attribution
- `NetWorthFormula.jsx` - Visual net worth formula display
- `BankBalanceChangeView.jsx` - Bank balance trend visualization
- `CalendarSpendingView.jsx` - Calendar-based spending heat map
- `DatabaseLineageChart.jsx` - Database relationship visualization
- `SubscriptionMappingsManager.jsx` - Subscription payment management

### Added - Investment Features
- `RealizedGains.jsx` - Capital gains tracking component (9242a64)
- Earnings dates tracking with graph visualization (cf9bb2a, 1d9e44d, 2a35b8f)
- Database tables: `stock_earnings_dates`, `stock_earnings_results`

### Added - Infrastructure
- Airflow email notifications for DAG failures (8703f08)
- DAG internet connectivity checks (5bca65c)
- DAG security improvements (5e00801)

### Changed
- Investment chart filters and stock sales handling (9242a64)
- Credit card chart visualizations (5e5ca0f, 1c13c78)
- Net worth chart height adjustments (553866c)
- DAG end-of-day net worth fixes (91bcc14)
- Intraday net worth change calculations (229cab7)
- Airflow timing synchronization (367b123)
- Data label and Y-axis scaling fixes (c5ff2c6)

### Fixed
- Double subscription counting (7a38398)
- DAG error handling and payment mapping formatting (8a76b7d)

### Key Commits
| Hash | Date | Description |
|------|------|-------------|
| 9242a64 | 2025-07-29 | investment chart filter, stock sales, etc |
| 4f464aa | 2025-07-26 | **massive changes** - Transfer detection system |
| 5db8a8b | 2025-07-25 | db org - Payment mappings system |
| 629e1d6 | 2025-07-21 | naming fix - Route discovery, Subscription manager |
| 1bc51ad | 2025-07-18 | new nw views - DailyChangeBreakdown, NetWorthFormula |
| cf9bb2a | 2025-07-14 | earnings dates - Stock earnings tracking |
| d614f94 | 2025-07-09 | misc changes - Schema routes, balance history, multiple components |

---

## June 2025

**Commits**: 3 | **Key Focus**: Project initialization

### Added - Initial Release (e472d64)

**Backend (Flask)**:
- `app.py` - Main Flask application (port 5001)
- `config.py` - Environment configuration
- `plaid_service.py` - Plaid API integration
- `net_worth_service.py` - Net worth calculation service
- `database_info.sql` - Centralized database schema
- `routes/analytics.py` - Financial analytics endpoints
- `routes/investments.py` - Investment portfolio operations
- `routes/transactions.py` - Transaction management
- `routes/net_worth_routes.py` - Net worth calculations
- `financial_data/` - Clean architecture data layer (db_operations, handlers, processors, services, utils)

**Frontend (React 18 + Vite)**:
- Core pages: HomePage, TransactionsPage, ExpensesPage, InvestmentsPage, NetWorthPage
- Investment components: PortfolioSummary, PortfolioChart, StockHoldings, AssetAllocation, TopPerformers, CashSummary, AddStockForm, SellStockForm, TransactionImporter
- Net worth components: NetWorthSummary, NetWorthChart, AssetAllocation, LiabilityBreakdown
- Common components: Header, Sidebar
- State management: TanStack Query, SidebarContext
- Utilities: formatting.js, queryClient.js

**Infrastructure (Airflow)**:
- `financial_data_dag.py` - Main Plaid sync pipeline (4x daily)
- `stock_price_tracker_dag.py` - Yahoo Finance price updates (weekdays at market close)
- `net_worth_snapshot_dag.py` - Net worth calculations
- `test_import_dag.py` - Diagnostic DAG
- Docker Compose configuration with PostgreSQL
- Makefile automation

**Documentation**:
- `CLAUDE.md` - AI assistant instructions
- `README.MD` - Project documentation
- `.env.example` - Environment template

### Key Commits
| Hash | Date | Description |
|------|------|-------------|
| e472d64 | 2025-06-19 | **Initial commit with .gitignore** |
| 709a963 | 2025-06-25 | misc changes |
| 7493d24 | 2025-06-26 | misc changes |

---

## Summary Statistics

### Commits by Month
| Month | Commits | Key Features |
|-------|---------|--------------|
| Jun 2025 | 3 | Initial release |
| Jul 2025 | 30 | Transfer detection, payment mappings, net worth views |
| Aug 2025 | 6 | Stock price reliability, chart improvements |
| Sep 2025 | 2 | Graph scaling, balance changes |
| Oct 2025 | 3 | Top N transactions |
| Nov 2025 | 8 | Performance optimizations |
| Dec 2025 | 5 | Data lineage visualization |
| **Total** | **57** | |

### Current Application Stats (as of Dec 2025)
- **Backend Routes**: 12+ blueprint files, 163+ endpoints
- **Frontend Pages**: 18 routes
- **Database Tables**: 27+
- **Airflow DAGs**: 4 active (1 disabled)

---

## Version Reference

For semantic versioning reference based on feature milestones:

| Version | Date | Milestone |
|---------|------|-----------|
| v1.0.0 | 2025-06-19 | Initial release (e472d64) |
| v1.1.0 | 2025-07-09 | Schema routes, balance history (d614f94) |
| v1.2.0 | 2025-07-18 | Net worth views, earnings dates (1bc51ad, cf9bb2a) |
| v2.0.0 | 2025-07-26 | Transfer detection, payment mappings (4f464aa, 5db8a8b) |
| v2.1.0 | 2025-08-03 | Stock price reliability (7a3fa11) |
| v2.2.0 | 2025-12-01 | Data lineage visualization (33dbed9) |
