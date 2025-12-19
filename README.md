# Financial Data Fetcher

A comprehensive full-stack financial management application integrating with Plaid API for real-time financial data aggregation, investment portfolio tracking, and automated data pipeline orchestration.

> **[View Changelog](changelog.html)** - See the complete development history with monthly summaries and key commits.

## Architecture Overview

This application is built as a modern full-stack solution with enterprise-grade data management capabilities:

- **Frontend**: React 18 with Vite build system, TanStack Query for state management
- **Backend**: Flask (Python) with modular blueprint architecture
- **Database**: PostgreSQL with sophisticated financial data modeling
- **Orchestration**: Apache Airflow for automated ETL pipelines
- **API Integration**: Plaid API for multi-institution financial data

### System Architecture Diagram

```plantuml
@startuml
!theme plain
skinparam backgroundColor #FEFEFE
skinparam boxPadding 10
skinparam defaultFontSize 11

title Financial Data Fetcher - System Architecture

package "Frontend Layer (20 Pages)" #E8F5E9 {
  [React 18 Application] as React
  [Vite Dev Server] as Vite
  [TanStack Query] as Query
  [Chart.js + Plotly] as Charts
  React --> Query : State Management
  React --> Vite : Build & HMR
  React --> Charts : Visualizations
}

package "API Gateway" #E3F2FD {
  [HTTPS Proxy] as Proxy
  note right of Proxy : Port 3000 → 5001
}

package "Backend Layer (17 Routes)" #FFF3E0 {
  [Flask Application] as Flask
  [Blueprint Routes] as Routes
  [Plaid Service] as PlaidSvc
  Flask --> Routes : /api/*
  Routes --> PlaidSvc : Financial Data
}

package "Data Processing" #F3E5F5 {
  [FinancialDataHandler] as FDH
  [SingleInstitutionHandler] as SIH
  [Processors] as Processors
  FDH --> SIH : Per-Institution
  SIH --> Processors : Transform
}

package "External APIs" #FFEBEE {
  cloud "Plaid API" as PlaidAPI
  cloud "Yahoo Finance" as YahooAPI
  PlaidSvc ..> PlaidAPI : HTTPS
  Processors ..> YahooAPI : Market Data
}

package "Data Layer (27 Tables + 3 Mat. Views + 19 Views)" #E0F2F1 {
  database "PostgreSQL" as DB {
    collections "Plaid: institutions, items, access_tokens"
    collections "Transactions: transactions, mappings"
    collections "Investments: portfolio_holdings, stock_sales"
    collections "Cash: cash_transactions"
    collections "Net Worth: net_worth_snapshots"
  }
  [Connection Pool] as Pool
  [Materialized Views] as MatViews
  note bottom of MatViews : stg_transactions\ncash_inflow_summary\ncash_outflow_summary
  Pool --> DB
  Pool --> MatViews
}

package "Orchestration (5 DAGs)" #FCE4EC {
  [Apache Airflow] as Airflow
  [financial_data_dag] as DAG1
  [stock_price_tracker_dag] as DAG2
  [net_worth_snapshot_dag] as DAG3
  Airflow --> DAG1 : Plaid Sync
  Airflow --> DAG2 : Price Updates
  Airflow --> DAG3 : Snapshots
  DAG1 --> Flask : Trigger Updates
}

package "Infrastructure" #F1F8E9 {
  node "Docker Compose" as Docker {
    [Flask Container]
    [Airflow Container]
    [Postgres Container]
  }
}

' Connections
React --> Proxy : HTTPS
Proxy --> Flask : HTTP
Routes --> FDH : Process
FDH --> Pool : Query/Update
DAG1 --> Pool : Batch Processing
DAG1 --> MatViews : REFRESH

' Styling
skinparam component {
  BackgroundColor #FFFFFF
  BorderColor #90A4AE
  FontColor #37474F
  ArrowColor #607D8B
}

skinparam database {
  BackgroundColor #ECEFF1
  BorderColor #607D8B
}

skinparam cloud {
  BackgroundColor #FFF9C4
  BorderColor #F57C00
}

@enduml
```

### Data Flow Architecture

```plantuml
@startuml
!theme plain
skinparam backgroundColor #FEFEFE

title Financial Data Flow - Complete ETL Pipeline

actor User as U
participant "React UI" as UI
participant "Flask API" as API
participant "Plaid Service" as PS
participant "Airflow DAG" as DAG
database "PostgreSQL" as DB
participant "External APIs" as EXT

== User Initiated Flow ==
U -> UI : Request Data
UI -> API : GET /api/transactions
API -> DB : Query materialized views
note right: stg_transactions,\ncash_inflow_summary,\ncash_outflow_summary
DB -> API : Return cached results
API -> UI : JSON response
UI -> U : Display data

== Plaid Institution Sync (Sequential) ==
U -> UI : Click "Sync All"
UI -> API : POST /fetch_financial_data
loop For each institution (rate limited)
  API -> PS : Get access token
  PS -> EXT : Plaid transactions/sync
  EXT -> PS : Financial data
  PS -> API : Process & validate
  API -> DB : Upsert transactions
  API -> DB : Update account_history
end
API -> DB : REFRESH MATERIALIZED VIEWS
note right: stg_transactions,\ncash_inflow_summary,\ncash_outflow_summary
API -> UI : Success with counts

== Plaid Link Flow ==
U -> UI : Add Institution
UI -> API : GET /create_link_token
API -> EXT : Plaid Link Token
EXT -> UI : Open Plaid Link
U -> UI : Select bank & login
UI -> API : POST /exchange_public_token
API -> EXT : Exchange for access_token
API -> DB : Store access_token

== Automated Airflow Pipeline ==
DAG -> DAG : Scheduled trigger (daily)
DAG -> API : Sync all institutions
API -> DB : Process transactions
DAG -> DB : Refresh materialized views
DAG -> DB : Log telemetry in plaid_api_calls

== Investment Price Updates ==
DAG -> DAG : Check if trading day
alt Is Trading Day
  DAG -> EXT : Yahoo Finance API
  EXT -> DAG : Market prices
  DAG -> DB : Update stock_price_history
  DAG -> DB : Update portfolio_history
else Weekend/Holiday
  DAG -> DAG : Skip (AirflowSkipException)
end

== CSV Import Flow ==
U -> UI : Upload Fidelity CSV
UI -> UI : Parse & detect duplicates
UI -> API : POST /api/investments/import-csv
API -> DB : Check existing holdings
API -> DB : Create portfolio_holdings
API -> DB : Record cash_transactions
API -> DB : FIFO stock_sales processing
API -> UI : Import summary

@enduml
```

## Key Features

### Financial Data Management
- **Multi-Institution Support**: Connect and sync data from multiple banks and financial institutions
- **Real-time Transaction Syncing**: Automated transaction categorization and tracking
- **Investment Portfolio Tracking**: Stock holdings, performance metrics, and capital gains calculations
- **Net Worth Monitoring**: Historical net worth tracking with asset allocation analysis
- **Cash Flow Analysis**: Income and expense tracking with transfer filtering

### Investment Management
- **Portfolio Dashboard**: Real-time portfolio valuation with performance metrics
- **Stock Trading**: Buy/sell functionality with FIFO tax lot tracking
- **CSV Import**: Bulk import investment transactions from Fidelity and other brokers
- **Duplicate Detection**: Intelligent handling of duplicate transactions across imports
- **Market Data**: Cached pricing with scheduled updates via Airflow DAGs

### Data Pipeline & ETL
- **Automated Syncing**: Scheduled data pulls via Airflow DAGs
- **Data Quality Checks**: Validation and consistency monitoring
- **Transaction Processing**: Complex SQL transformations for financial analytics
- **API Telemetry**: Comprehensive tracking of all Plaid API calls

## Project Structure

```
├── app/                          # Flask backend application
│   ├── app.py                   # Main Flask application (port 5001)
│   ├── config.py                # Environment configuration
│   ├── plaid_service.py         # Plaid API integration
│   ├── database_info.sql        # Consolidated database schema
│   ├── routes/                  # API endpoints (17 route files)
│   │   ├── analytics.py         # Financial analytics & date range presets
│   │   ├── api_routes.py        # Core API endpoints
│   │   ├── bank_balance_history.py  # Bank balance tracking
│   │   ├── categories.py        # Transaction categorization
│   │   ├── investment_transfers.py  # Transfer pattern detection
│   │   ├── investments.py       # Portfolio operations, realized gains, CSV import
│   │   ├── mapped_payments.py   # Payment mapping functionality
│   │   ├── misc.py             # Miscellaneous utilities
│   │   ├── misc_mappings_stats.py  # Mapping statistics
│   │   ├── net_worth_routes.py  # Net worth calculations
│   │   ├── payment_mappings.py  # Payment mapping routes
│   │   ├── refresh_routes.py    # Data refresh endpoints
│   │   ├── route_discovery.py   # API route introspection
│   │   ├── schema_routes.py     # Database schema access
│   │   ├── sql.py              # SQL query execution
│   │   ├── transactions.py      # Transaction management
│   │   └── transaction_mappings.py  # Transaction mappings
│   └── financial_data/          # Clean architecture data layer
│       ├── db_operations/       # Database access layer
│       ├── handlers/            # Business logic coordination
│       ├── processors/          # Data transformation
│       ├── services/            # Domain-specific services
│       └── utils/               # Shared utilities
│
├── frontend/                    # React frontend application
│   ├── src/
│   │   ├── main.jsx            # Application entry point
│   │   ├── App.jsx             # Main app component with routing
│   │   ├── components/         # 32 reusable UI components
│   │   │   ├── common/         # Header, Sidebar, CircularProgressRing
│   │   │   ├── dashboard/      # SummaryCard
│   │   │   ├── investments/    # PortfolioChart, StockHoldings, RealizedGains, TransactionImporter, etc.
│   │   │   ├── networth/       # NetWorthChart, AssetAllocation, LiabilityBreakdown, MonthlyCashFlowChart, etc.
│   │   │   ├── widgets/        # CreditCardWidget
│   │   │   └── notifications/  # TransferDetectionBanner, FrontendTransferBanner
│   │   ├── pages/              # 20 route-level pages
│   │   │   ├── HomePage.jsx, ExpensesPage.jsx, TransactionsPage.jsx
│   │   │   ├── InvestmentsPage.jsx, NetWorthPage.jsx, AllBalancesPage.jsx
│   │   │   ├── PaymentHistoryPage.jsx, SubscriptionsPage.jsx, InstitutionsPage.jsx
│   │   │   ├── AnalysisPage.jsx, PaymentMappingsPage.jsx, PlaidApiCatalogPage.jsx
│   │   │   ├── SqlEditorPage.jsx, DatabaseSchemaPage.jsx, SchemaPage.jsx
│   │   │   └── ModelExplorerPage.jsx, RouteMapPage.jsx
│   │   └── context/            # React context providers
│   ├── vite.config.js          # Vite config with API proxy
│   └── package.json            # Frontend dependencies
│
├── airflow/                    # Apache Airflow orchestration
│   └── dags/                   # Data pipeline definitions (5 DAGs)
│       ├── financial_data_dag.py      # Main sync + materialized view refresh
│       ├── stock_price_tracker_dag.py # Stock price updates from Yahoo Finance
│       ├── daily_stock_price_updater_dag.py # Daily price batch updates
│       ├── net_worth_snapshot_dag.py  # Net worth snapshot calculations
│       └── test_import_dag.py         # Import testing utilities
│
├── docker-compose.yml          # Multi-service orchestration
├── Dockerfile                  # Custom Airflow image
├── Makefile                    # Build automation
└── requirements.txt            # Python dependencies
```

## Technical Implementation

### Backend Architecture

**Flask Application**:
- Modular blueprint architecture with `/api/` prefix
- RESTful API design with comprehensive error handling
- Database connection pooling for performance
- Plaid webhook support for real-time updates

**Data Layer** (Clean Architecture):
- **db_operations**: Low-level database access with prepared statements
- **handlers**: `FinancialDataHandler` (main orchestrator), `SingleInstitutionHandler` (per-institution refresh)
- **processors**: Data transformation and validation
- **services**: Domain-specific business rules (investments, net worth)
- **utils**: `get_db_connection()`, shared helpers

**Key Backend Patterns**:
- Sequential institution processing with rate limiting (2s delay between institutions)
- Materialized view refresh after every sync and institution removal
- Plaid Link update mode for re-authentication without losing data
- FIFO stock sale processing for capital gains tracking

### Frontend Architecture

**React + Vite**:
- Modern React 18 with functional components
- TanStack Query for server state management
- Vite dev server with HTTPS and API proxy
- CSS Modules for component styling
- Chart.js and Plotly.js for data visualization

**20 Pages** organized by domain:

| Category | Pages |
|----------|-------|
| **Financial** | ExpensesPage, TransactionsPage, PaymentHistoryPage, SubscriptionsPage, AnalysisPage |
| **Investment** | InvestmentsPage, NetWorthPage, AllBalancesPage |
| **Admin/Dev** | SqlEditorPage, DatabaseSchemaPage, SchemaPage, ModelExplorerPage, RouteMapPage, PlaidApiCatalogPage, InstitutionsPage |
| **Home** | HomePage |
| **Mappings** | PaymentMappingsPage |

**Key Components**:
- `PortfolioChart`: Drag-to-select with gain/loss calculation, crosshair plugin
- `TransactionImporter`: CSV parsing with duplicate detection (1% price tolerance)
- `NetWorthChart`: Historical trends with asset allocation breakdown
- `TransferDetectionBanner`: Pattern-based transfer identification
- `RealizedGains`: Three-tab view (summary/detailed/by-symbol) with FIFO tracking

### Database Design

**Transaction-Based Accounting**:
- All cash balances calculated from transaction sums
- No stored balances to prevent discrepancies
- FIFO stock sale tracking for accurate tax reporting

**27 Base Tables** organized by domain:

| Domain | Tables |
|--------|--------|
| **Plaid Integration** | `institutions`, `items`, `access_tokens`, `institution_cursors`, `plaid_api_calls` |
| **Transactions** | `transactions`, `transaction_mappings`, `transaction_specific_mappings`, `category_mappings`, `group_mappings` |
| **Investments** | `portfolio_holdings`, `portfolio_history`, `stock_sales`, `stock_price_history`, `stock_market_data`, `stock_earnings_dates`, `stock_earnings_results` |
| **Cash Management** | `cash_transactions`, `investment_cash_holdings`, `investment_account_balances` |
| **Transfers** | `custom_transfer_patterns`, `detected_investment_transfers` |
| **Net Worth** | `net_worth_snapshots`, `account_history`, `earnings_history` |

**3 Materialized Views** (for performance, refreshed via Airflow):
- `stg_transactions`: Pre-aggregated transaction data
- `cash_inflow_summary`: Income aggregation with transfer filtering
- `cash_outflow_summary`: Expense aggregation with transfer filtering

**19 Calculated Views**:

| Category | Views |
|----------|-------|
| **Accounts** | `accounts`, `depository_accounts`, `credit_accounts`, `items_calc` |
| **Net Worth** | `current_net_worth`, `current_net_worth_enhanced`, `net_worth_history`, `monthly_net_worth_summary`, `asset_allocation`, `liability_breakdown` |
| **Investments** | `current_investment_cash`, `current_portfolio_value_by_lots`, `portfolio_performance`, `portfolio_performance_enhanced`, `tax_year_capital_gains`, `cash_transaction_history` |
| **Transactions** | `transactions_v2`, `subscription_name_resolver`, `view_credit_card_monthly_summary`, `account_history_with_calcs` |

### Data Pipeline Architecture

**5 Apache Airflow DAGs**:
- `financial_data_dag`: Plaid sync + materialized view refresh (`stg_transactions`, `cash_inflow_summary`, `cash_outflow_summary`)
- `stock_price_tracker_dag`: Yahoo Finance price updates with market hours detection
- `daily_stock_price_updater_dag`: Batch price updates for all holdings
- `net_worth_snapshot_dag`: Daily net worth calculations and snapshots
- `test_import_dag`: Import testing and validation utilities

**Performance Optimizations**:
- Materialized views reduced query time from 400ms to <1ms
- Cached market data (no API calls on page load)
- Batch processing for efficiency
- Incremental data loading
- Connection pooling

## Getting Started

### Prerequisites
- Python 3.9+
- Node.js 18+
- PostgreSQL 12+
- Docker & Docker Compose

### Quick Start

1. **Clone the repository**:
```bash
git clone <repository-url>
cd frontend_change
```

2. **Set up environment**:
```bash
cp .env.example .env
# Configure Plaid credentials and database settings
```

3. **Start the backend**:
```bash
# Install Python dependencies
pip install -r requirements.txt

# Run Flask application
python app/app.py
```

4. **Start the frontend**:
```bash
cd frontend
npm install
npm run dev
```

5. **Start Airflow** (optional for automated pipelines):
```bash
make build  # Build custom Airflow image
make up     # Start all services
```

### Development URLs
- Frontend: https://localhost:3000
- Backend API: http://localhost:5001/api/
- Airflow UI: http://localhost:8080

## Key Features in Detail

### Investment CSV Import System

The application supports sophisticated CSV import for investment transactions:

**Supported Formats**:
- Fidelity brokerage statements
- 401k/retirement account exports
- Custom CSV formats (configurable)

**Intelligent Processing**:
- Duplicate detection with price tolerance (1% variance allowed)
- Automatic cash flow calculation
- FIFO stock sale processing
- Capital gains tracking
- Transaction categorization

### Cash Flow Analysis

**Accurate Tracking**:
- Transfer transactions excluded to prevent double-counting
- Income vs expense categorization
- Monthly and yearly aggregations
- Custom date range analysis

### Net Worth Monitoring

**Comprehensive Coverage**:
- Bank accounts (checking, savings)
- Investment portfolios
- Credit card balances
- Historical trending
- Asset allocation breakdown

## Configuration

### Environment Variables (.env)
```bash
# Plaid Configuration
PLAID_CLIENT_ID=your_client_id
PLAID_SECRET=your_secret
PLAID_ENV=development

# Database Configuration
DATABASE_URL=postgresql://user:password@localhost/dbname

# Application Settings
FLASK_ENV=development
FLASK_DEBUG=1
```

### Database Schema Management

All database changes must be made in `app/database_info.sql`:
- Single source of truth for schema
- Organized into clearly marked sections
- Includes tables, views, and indexes
- Never create separate migration files

## Monitoring & Observability

### API Telemetry
- All Plaid API calls tracked in `plaid_api_calls` table
- Response times, error rates, and rate limits monitored
- Request/response correlation for debugging

### Data Quality
- Automated validation checks
- Consistency monitoring
- Error detection and logging
- Historical quality trends

## Testing

```bash
# Backend tests
pytest app/tests/

# Frontend tests
cd frontend && npm test

# Integration tests
python app/tests/integration/
```