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
- **Brokerage Integration**: Schwab and SnapTrade APIs for multi-broker portfolio aggregation

### System Architecture Diagram

```mermaid
flowchart LR
    subgraph ext["External"]
        direction TB
        PL["Plaid"]
        SC["Schwab"]
        ST["SnapTrade<br/>test + prod"]
        YF["Yahoo Finance<br/>prices"]
    end

    subgraph orch["Orchestration"]
        direction TB
        DAG1["financial_data_pipeline<br/>every 3h · 7 institutions"]
        DAG2["stock_price_tracker<br/>Mon-Fri 1:01 PM PT"]
    end

    subgraph app["Application"]
        direction TB
        API["Flask API<br/>23 blueprints"]
        SVC["Services<br/>plaid · schwab · snaptrade<br/>ETL handlers"]
        API --- SVC
    end

    subgraph data["Data"]
        direction TB
        DB[("PostgreSQL<br/>public · schwab · snaptrade<br/>16 materialized views")]
    end

    subgraph clients["Clients"]
        direction TB
        WEB["Web<br/>React · TanStack Query"]
        MOB["Mobile PWA<br/>10 tabs · /api/mobile/*"]
    end

    subgraph notify["Notifications"]
        direction TB
        TG["Telegram"]
        DC["Discord"]
    end

    subgraph sec["Auth"]
        AUTH["Authentik OIDC<br/>Bearer tokens"]
    end

    ext -. raw data .-> orch
    orch -->|upsert| data
    orch -.-> notify
    app --> data
    app --> ext
    clients --> AUTH --> app
    data -. read .-> TG
    data -. read .-> DC
    orch -. refresh MVs .-> data
```

### Data Flow Architecture

```mermaid
flowchart LR
    subgraph fd["financial_data_pipeline · every 3h"]
        direction TB
        CHK1["check connectivity"]
        GT["get_institution_tokens<br/>decrypt access tokens"]
        subgraph par["7 institutions in parallel"]
            direction TB
            CITI["Citi"]
            BECU["BECU"]
            AMZN["Amazon"]
            TJX["TJX"]
            BOFA["BoA"]
            AMEX["Amex"]
            CAP1["Capital One"]
        end
        NW1["create net_worth_v2 snapshot"]
        N1["Telegram · Discord summary"]
        CHK1 --> GT --> par --> NW1 --> N1
    end

    subgraph sp["stock_price_tracker · Mon-Fri 1:01 PM PT"]
        direction TB
        CHK2["check connectivity"]
        SYM["get symbols from holdings"]
        PRICES["fetch yfinance prices"]
        SNAP["SnapTrade prod Fidelity sync<br/>accounts · balances · holdings · activities"]
        SCH["Schwab sync"]
        NW2["update NW v2 with investments"]
        TOKEN["Schwab token expiry check"]
        N2["Telegram net worth update"]
        CHK2 --> SYM --> PRICES --> SCH
        CHK2 --> SNAP
        SCH --> NW2
        SNAP --> NW2
        NW2 --> TOKEN --> N2
    end

    subgraph host["Host-level"]
        direction TB
        CRON["check_dag_liveness.sh<br/>every 30 min"]
        CRON -. alert if stale .-> N2
    end
```



## Component Catalog

Grounded in code, not aspiration. Names link to real modules.

### Backend (Flask)

| Area | Blueprint | Prefix | Primary tables |
|---|---|---|---|
| Brokerage | `schwab.py` | `/api/schwab` | `schwab.accounts/holdings/transactions/balances` |
| Brokerage | `snaptrade.py` | `/api/snaptrade` | `snaptrade.accounts/holdings/activities/portfolio_equity` |
| Banking | `transactions.py` | `/transactions` | `transactions`, `accounts`, `institutions` |
| Investments | `investments.py` | `/investments` | `portfolio_holdings`, `stock_price_history` |
| Analytics | `analytics.py` | `/analytics` | `transactions`, `categories`, `mapping_rules` |
| Net Worth | `net_worth_v2_routes.py` | `/api/net-worth-v2` | `net_worth_v2` |
| Bank history | `bank_balance_history.py` | `/api/bank_balance_history` | `account_history` |
| Reimbursements | `reimbursements.py` | `/reimbursements` | `reimbursements`, `transactions` |
| Plaid Link | `misc.py` | `/` (root) | `access_tokens`, `items`, `institutions` |
| Airflow ops | `airflow_routes.py` | `/api/airflow` | Airflow REST |

Plus 13 more blueprints for categories, payment mappings, SQL editor, schema discovery, and dev utilities.

### Services

| Service | Wraps | Key operations |
|---|---|---|
| `plaid_service` | Plaid API | OAuth, `transactions/sync`, investments |
| `schwab_service` | Schwab Trader API | OAuth refresh, accounts, holdings, transactions, balances |
| `snaptrade_service` | SnapTrade (test client) | connections, activities, equity |
| `snaptrade_prod_service` | SnapTrade (prod client) | Fidelity: accounts, balances, holdings, activities, cost-basis derivation |
| `InvestmentsService` | yfinance | price history, earnings |

ETL layer: `FinancialDataHandler` → `SingleInstitutionHandler` → processors (`AccountsProcessor`, `TransactionsProcessor`, `InstitutionsProcessor`) → DB ops (idempotent upserts).

### Airflow DAGs

| DAG | Schedule | Purpose | Key writes |
|---|---|---|---|
| `financial_data_pipeline` | 7 AM / 12 PM / 5 PM / 10 PM PST | Sync 7 institutions via Plaid, refresh materialized views, push summary | `transactions`, `account_history`, `net_worth_v2` |
| `stock_price_tracker` | Mon-Fri 1:01 PM PT | Fetch prices, sync Schwab + SnapTrade prod Fidelity, update NW investment section | `stock_price_history`, `schwab.*`, `snaptrade.*`, `net_worth_v2` |

### Data Layer

Three Postgres schemas on a single instance:

- **`public`** · 56 tables — core Plaid data (`transactions`, `account_history`, `institutions`, `access_tokens`), net worth (`net_worth_v2`, `net_worth_snapshots`), investments (`portfolio_holdings`, `stock_price_history`), reimbursements, categorization rules
- **`schwab`** · 7 tables — accounts, balances, holdings, transactions, orders, `oauth_tokens`, `sync_state`
- **`snaptrade`** · 13 tables — accounts, holdings, activities, portfolio_equity, orders, connections, option_holdings, `prod_users`, sync_state

16 materialized views accelerate common queries (monthly cash flow, expense summaries, daily net worth history). Refreshed at the end of each sync run.

### Frontend

- **Auth**: Authentik OIDC via `oidc-client-ts`; bearer tokens auto-injected by Axios interceptor
- **State**: TanStack Query (no Redux) with Context providers for sidebar + mobile nav
- **Desktop**: 18 pages (Transactions, Expenses, Investments, Net Worth v2, Brokerage, Reimbursements, etc.)
- **Mobile PWA**: 10-tab bottom navigation with dedicated `/api/mobile/*` pre-aggregated endpoints for instant loads

### Infrastructure

- **Containers** (plaid-app compose): `plaid_backend` (Flask), `plaid_frontend` (Vite), `airflow_scheduler`, `airflow_webserver`, `airflow_postgres`, `docker-monitor`
- **Remote access**: Cloudflare Tunnel to `miguelduran.me` + Tailscale overlay network
- **Host-level cron**: `check_dag_liveness.sh` every 30 min, alerts via Telegram if Airflow itself is down
- **Public static**: `portfoliopulse.pages.dev` (Cloudflare Pages) for brokerage-partner-facing landing

---

## Key Features

### Mobile Experience
- **Apple Stocks-Inspired Design**: Dark theme mobile app with green/red accent colors
- **9-Tab Navigation**: Home, Activity, Cash Flow, Invest, Accounts, Brokerage, Income, Payments, Subscriptions with bottom navigation
- **Home Dashboard**: Net Worth, Cash Available, Credit Used stat cards with daily change arrows
- **Activity Tab**: Transactions and subscriptions with inline expand, swipe-to-categorize, long-press actions
- **Cash Flow Tab**: Trend line chart with pill time selectors (1W, 1M, 3M, 1Y, YTD)
- **Invest Tab**: Simplified portfolio view with total value and holdings list
- **Accounts Tab**: Expandable accordion groups by account type with totals
- **Brokerage Tab**: Multi-broker portfolio view with performance charts and holdings across Schwab and SnapTrade
- **Income Tab**: Income analytics with category breakdowns, stacked bar charts, and monthly trends
- **Payments Tab**: Credit card payment tracking with account color-coding and payment history
- **Subscriptions Tab**: Recurring subscription overview with cost breakdowns and icon support
- **Touch Gestures**: Pull-to-refresh, swipe actions, long-press context menus
- **Mobile-Optimized APIs**: Pre-aggregated endpoints for instant page loads
- **Progressive Web App (PWA)**: Installable app with service worker for offline capability and native-like experience

### Financial Data Management
- **Multi-Institution Support**: Connect and sync data from multiple banks and financial institutions
- **Real-time Transaction Syncing**: Automated transaction categorization and tracking
- **Transaction Location Mapping**: Geocoded transaction locations with interactive map visualization
- **Investment Portfolio Tracking**: Stock holdings, performance metrics, and capital gains calculations
- **Net Worth Monitoring**: Historical net worth tracking with asset allocation analysis and transfer in-transit handling
- **Cash Flow Analysis**: Income and expense tracking with transfer filtering and flexible date range presets
- **Monthly Cash Flow Impact**: Comprehensive monthly analysis of income, expenses, and investment gains
- **Historical Balance Tracking**: Bridged data system combining historical records with real-time API data for complete balance history
- **Statement Import**: Import and reconcile historical financial statements for extended data coverage
- **Reimbursement Tracking**: Link incoming Venmo, PayPal, and Zelle payments to original expenses with split support

### Investment Management
- **Portfolio Dashboard**: Real-time portfolio valuation with performance metrics
- **Stock Trading**: Buy/sell functionality with FIFO tax lot tracking
- **Stock Split Handling**: Automatic detection and adjustment of stock splits via Airflow DAG
- **CSV Import**: Bulk import investment transactions from brokers
- **Duplicate Detection**: Intelligent handling of duplicate transactions across imports
- **Market Data**: Cached pricing with scheduled updates via Airflow DAGs
- **Holdings Comparison**: Compare current holdings vs sold positions with "what if held" analysis
- **Transfer Smoothing**: Intelligent smoothing of transfers in portfolio charts to prevent sudden jumps
- **Multi-Broker Aggregation**: Unified portfolio view combining Plaid, Schwab, and SnapTrade brokerage data
- **Plaid Investment Sync**: Direct investment holdings and transaction syncing via Plaid API

### Data Pipeline & ETL
- **Automated Syncing**: Scheduled data pulls via Airflow DAGs
- **Stock Split Detection**: Automatic detection of recent splits with holdings/history adjustment
- **Data Quality Checks**: Validation and consistency monitoring
- **Transaction Processing**: Complex SQL transformations for financial analytics
- **API Telemetry**: Comprehensive tracking of all API calls
- **Raw Data Capture**: Serialization and storage of complete API payloads for debugging and analysis
- **Materialized Views**: Pre-computed aggregations for significantly faster API responses (100x speedup)
- **Brokerage Sync Pipeline**: Automated syncing of positions and transactions from connected brokerages

## Project Structure

```
├── app/                          # Flask backend application
│   ├── app.py                   # Main Flask application
│   ├── config.py                # Environment configuration
│   ├── plaid_service.py         # Plaid API integration
│   ├── database_info.sql        # Consolidated database schema
│   ├── routes/                  # API endpoints
│   │   ├── analytics.py         # Financial analytics & date range presets
│   │   ├── api_routes.py        # Core API endpoints
│   │   ├── investments.py       # Portfolio operations, realized gains, CSV import
│   │   ├── net_worth_routes.py  # Net worth calculations
│   │   ├── transactions.py      # Transaction management
│   │   ├── reimbursements.py   # Reimbursement tracking & linking
│   │   ├── schwab.py           # Schwab brokerage integration
│   │   ├── snaptrade.py        # SnapTrade brokerage integration
│   │   ├── plaid_investments.py # Plaid investment data
│   │   └── mapping_rules.py    # Unified transaction mapping rules
│   └── financial_data/          # Clean architecture data layer
│       ├── db_operations/       # Database access layer
│       ├── handlers/            # Business logic coordination
│       ├── processors/          # Data transformation
│       ├── services/            # Domain-specific services
│       │   ├── schwab_service.py      # Schwab OAuth & API
│       │   ├── snaptrade_service.py   # SnapTrade SDK integration
│       │   └── plaid_investments_service.py  # Plaid investment sync
│       └── utils/               # Shared utilities
│
├── frontend/                    # React frontend application
│   ├── src/
│   │   ├── main.jsx            # Application entry point
│   │   ├── App.jsx             # Main app component with routing
│   │   ├── components/         # Reusable UI components
│   │   │   ├── common/         # Header, Sidebar, shared components
│   │   │   ├── dashboard/      # Dashboard widgets
│   │   │   ├── investments/    # Investment components
│   │   │   ├── networth/       # Net worth visualizations
│   │   │   └── mobile/         # Mobile-specific components
│   │   │       ├── MobileApp.jsx          # Mobile app wrapper
│   │   │       ├── MobileBottomNav.jsx    # 9-tab bottom navigation
│   │   │       └── tabs/                  # Tab-specific views
│   │   │           ├── HomeTab/           # Net worth, cash, credit cards
│   │   │           ├── ActivityTab/       # Transactions, subscriptions
│   │   │           ├── CashFlowTab/       # Income/expense charts
│   │   │           ├── InvestTab/         # Portfolio summary
│   │   │           ├── AccountsTab/       # Account balances
│   │   │           ├── BrokerageTab/       # Multi-broker portfolio
│   │   │           ├── IncomeTab/          # Income analytics
│   │   │           ├── PaymentsTab/        # Payment tracking
│   │   │           └── SubsTab/            # Subscription overview
│   │   ├── hooks/              # Custom React hooks
│   │   │   └── useIsMobile.js  # Mobile detection hook
│   │   ├── pages/              # Route-level pages
│   │   │   ├── IncomePageV2.jsx          # Income analytics dashboard
│   │   │   ├── PaymentHistoryPageV2.jsx  # Payment history with charts
│   │   │   ├── SubscriptionsPageV2.jsx   # Subscription management
│   │   │   ├── SnapTradePage.jsx         # Brokerage portfolio view
│   │   │   └── ReimbursementsPage.jsx    # Reimbursement tracking
│   │   └── context/            # React context providers
│   │       └── MobileNavContext.jsx  # Mobile tab state
│   ├── vite.config.js          # Vite config with API proxy
│   └── package.json            # Frontend dependencies
│
├── airflow/                    # Apache Airflow orchestration
│   └── dags/                   # Data pipeline definitions
│       ├── sync_pipeline.py          # Main sync + materialized view refresh
│       ├── stock_price_tracker_dag.py # Stock price updates + split detection
│       └── snapshot_pipeline.py      # Net worth snapshot calculations
│
├── scripts/                    # Utility scripts
│   ├── refetch_split_adjusted_prices.py   # Refetch prices with split adjustment
│   └── update_portfolio_history_prices.py # Update portfolio history from price data
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
- **handlers**: Main orchestrator and per-institution refresh handlers
- **processors**: Data transformation and validation
- **services**: Domain-specific business rules (investments, net worth)
- **utils**: Database connections and shared helpers

**Key Backend Patterns**:
- Sequential institution processing with rate limiting
- Materialized view refresh after every sync
- Plaid Link update mode for re-authentication without data loss
- FIFO stock sale processing for capital gains tracking

### Frontend Architecture

**React + Vite**:
- Modern React 18 with functional components
- TanStack Query for server state management
- Vite dev server with HTTPS and API proxy
- CSS Modules for component styling
- Chart.js and Plotly.js for data visualization
- Responsive design with dedicated mobile experience

**Mobile Architecture**:
- Viewport-based detection (768px breakpoint) automatically switches between desktop and mobile
- Complete separation of mobile/desktop UX - desktop view unchanged
- Mobile-specific context for tab state management
- Touch gesture support (swipe, long-press, pull-to-refresh)
- Mobile-optimized API endpoints for instant page loads

**Pages** organized by domain:

| Category | Pages |
|----------|-------|
| **Financial** | Expenses, Transactions, Payment History, Subscriptions, Analysis, Reimbursements |
| **Investment** | Investments, Net Worth, All Balances, SnapTrade/Brokerage |
| **Admin/Dev** | SQL Editor, Database Schema, Model Explorer, Route Map, Institutions |

**Key Components**:
- `PortfolioChart`: Drag-to-select with gain/loss calculation, crosshair plugin
- `TransactionImporter`: CSV parsing with duplicate detection (1% price tolerance)
- `NetWorthChart`: Historical trends with asset allocation breakdown
- `TransferDetectionBanner`: Pattern-based transfer identification
- `RealizedGains`: Three-tab view (summary/detailed/by-symbol) with FIFO tracking
- `InstitutionToggle`: Multi-broker institution selector for filtering portfolio views
- `SplitReimbursementModal`: Split and link reimbursement payments to original expenses

### Database Design

**Transaction-Based Accounting**:
- All cash balances calculated from transaction sums
- No stored balances to prevent discrepancies
- FIFO stock sale tracking for accurate tax reporting

**Data Stores** organized by domain:

| Domain | Description |
|--------|-------------|
| **Plaid Integration** | Institution connections and encrypted credentials |
| **Transactions** | Transaction records with categorization and mappings |
| **Investments** | Portfolio holdings, sales history, price data, stock splits |
| **Cash Management** | Cash transactions and account balances |
| **Transfers** | Custom transfer patterns and detected transfers |
| **Analytics** | Net worth snapshots (with in-transit tracking), account history, earnings |

**Key Tables**:
- `portfolio_holdings`: Individual stock lots with cost basis
- `cash_transactions`: All cash movements with categorization
- `stock_sales`: Capital gains tracking with tax implications
- `stock_splits`: Stock split events for price/quantity normalization
- `net_worth_snapshots`: Daily snapshots with in_transit_amount for transfers between accounts

**Materialized Views** (for performance, refreshed via Airflow):
- Pre-aggregated transaction data
- Income aggregation with transfer filtering
- Expense aggregation with transfer filtering

**Calculated Views**:

| Category | Purpose |
|----------|---------|
| **Accounts** | Account listings by type |
| **Net Worth** | Current and historical net worth with breakdowns |
| **Investments** | Portfolio value, performance, capital gains |
| **Transactions** | Enhanced transaction views with mappings |
| **Stock Splits** | Cumulative split factors for price adjustment |

### Data Pipeline Architecture

**Apache Airflow DAGs**:
- Plaid sync + materialized view refresh
- Stock split detection with automatic holdings/history adjustment
- Market data price updates with market hours detection
- Daily price batch updates for all holdings
- Net worth snapshot calculations

**Stock Split Detection**:
- Checks yfinance for splits in last 60 days for all portfolio symbols
- Records new splits in `stock_splits` table
- Automatically adjusts `portfolio_holdings` (quantity × ratio, price ÷ ratio)
- Updates `portfolio_history` for dates before split
- Pre-populated with known major splits (NFLX, GOOGL, AMZN, TSLA, AAPL, NVDA)

**Performance Optimizations**:
- Materialized views reduced query time from 400ms to <1ms
- Database statistics materialized view provides 100x speedup (~720ms to ~7ms)
- Cached market data (no API calls on page load)
- Batch processing for efficiency
- Incremental data loading
- Connection pooling

## Key Features in Detail

### Investment CSV Import System

The application supports sophisticated CSV import for investment transactions:

**Supported Formats**:
- Major brokerage statements
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
- Custom date range analysis with presets (YTD, Prior YTD, Past 3/6/12 Months, All Time)
- 12-month rolling averages for expense comparisons

### Net Worth Monitoring

**Comprehensive Coverage**:
- Bank accounts (checking, savings)
- Investment portfolios
- Credit card balances
- Historical trending
- Asset allocation breakdown
- Transfer in-transit handling (money between accounts)

## Monitoring & Observability

### API Telemetry
- All external API calls tracked with response times
- Error rates and rate limits monitored
- Request/response correlation for debugging

### Data Quality
- Automated validation checks
- Consistency monitoring
- Error detection and logging
- Historical quality trends

## Intelligent Assistants & Automation


### Discord Finance Bot
- **Natural Language Queries**: Ask financial questions in plain English
- **Claude CLI Integration**: Uses Claude Code CLI for accurate SQL generation
- **Chart Generation**: Matplotlib-powered charts for spending trends and portfolio visualization
- **Secure**: Only responds to authorized Discord user ID

### Telegram LLM Bot
- **Hybrid Processing**: Claude CLI for SQL generation with Ollama fallback for local processing
- **Financial Query Support**: Natural language queries for transactions, balances, investments
- **Direct Formatting**: Results formatted directly without hallucination risk

### Docker Error Monitor
- **Real-time Log Monitoring**: Watches container logs for ERROR, CRITICAL, FATAL, EXCEPTION patterns
- **Smart Error Parsing**: Extracts DAG name, task ID, and error message from Airflow errors
- **Deduplication**: Groups repeated errors with cooldown to reduce notification spam
- **Interactive Telegram Alerts**: Approve or Ignore buttons for proposed fixes
- **Claude-Based Auto-Fixing**: Analyzes codebase and proposes code fixes for errors
- **ShellFish Handoff**: Mobile debugging support with prompt handoff to Claude Code


## Security

### Docker Containerization
- **Isolated Containers**: Flask backend and React frontend run in separate Docker containers
- **Port Binding Security**: Service ports bound to Tailscale IP only; database ports are no longer externally exposed (not accessible from WiFi network)
- **Hot-Reload Development**: Volume mounts enable code changes without container rebuilds
- **Health Checks**: Container health monitoring via `/api/health` endpoint
- **Makefile Commands**: Simplified operations (`make plaid`, `make build-plaid`, `make logs-plaid`)

### API Key Authentication
- **All Routes Protected**: Every endpoint requires valid `X-API-Key` header except explicit whitelist
- **Whitelist System**: Only public pages, health checks, webhooks, and static files bypass authentication
- **Constant-Time Comparison**: Timing-attack resistant key validation using HMAC
- **Centralized Middleware**: Flask `before_request` hook validates all incoming requests
- **Frontend Integration**: Axios interceptor automatically adds API key to all requests
- **Keychain Storage**: API keys stored securely in macOS Keychain (not in code or config files)

### Token Encryption
- **AES-256-GCM Encryption**: All sensitive tokens encrypted at rest
- **macOS Keychain Integration**: Encryption keys stored securely in system Keychain
- **Migration Support**: Scripts to encrypt existing plaintext tokens
- **Verification Tools**: Encryption verification without exposing sensitive data

### Access Control
- **VPN-Based Access**: Remote access via Tailscale for secure mobile connectivity
- **No Public Endpoints**: Application only accessible on local network or VPN
- **Credential Protection**: Schema explorer hides sensitive database fields
- **Network Isolation**: WiFi users cannot access the application (Tailscale-only remote access)

