# Financial Data Fetcher

A comprehensive full-stack financial management application integrating with Plaid API for real-time financial data aggregation, investment portfolio tracking, and automated data pipeline orchestration.

## 🏗️ Architecture Overview

This application is built as a modern full-stack solution with enterprise-grade data management capabilities:

- **Frontend**: React 18 with Vite build system, TanStack Query for state management
- **Backend**: Flask (Python) with modular blueprint architecture
- **Database**: PostgreSQL with sophisticated financial data modeling
- **Orchestration**: Apache Airflow for automated ETL pipelines
- **API Integration**: Plaid API for multi-institution financial data

## 🚀 Key Features

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

## 📁 Project Structure

```
├── app/                          # Flask backend application
│   ├── app.py                   # Main Flask application (port 5001)
│   ├── config.py                # Environment configuration
│   ├── plaid_service.py         # Plaid API integration
│   ├── database_info.sql        # Centralized database schema (single source of truth)
│   ├── routes/                  # API endpoints (all use /api/ prefix)
│   │   ├── analytics.py         # Financial analytics endpoints
│   │   ├── investments.py       # Investment portfolio operations
│   │   ├── transactions.py      # Transaction management
│   │   ├── net_worth_routes.py  # Net worth calculations
│   │   └── plaid_routes.py      # Plaid API integrations
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
│   │   ├── components/         # Reusable UI components
│   │   │   ├── common/         # Shared components
│   │   │   ├── dashboard/      # Dashboard widgets
│   │   │   ├── investments/    # Investment components
│   │   │   └── networth/       # Net worth visualizations
│   │   ├── pages/              # Route-level components
│   │   └── context/            # React context providers
│   ├── vite.config.js          # Vite config with API proxy
│   └── package.json            # Frontend dependencies
│
├── airflow/                    # Apache Airflow orchestration
│   └── dags/                   # Data pipeline definitions
│       ├── financial_data_dag.py      # Main sync pipeline
│       ├── stock_price_tracker_dag.py # Stock price updates
│       └── net_worth_snapshot_dag.py  # Net worth calculations
│
├── docker-compose.yml          # Multi-service orchestration
├── Dockerfile                  # Custom Airflow image
├── Makefile                    # Build automation
└── requirements.txt            # Python dependencies
```

## 🛠️ Technical Implementation

### Backend Architecture

**Flask Application**:
- Modular blueprint architecture with `/api/` prefix
- RESTful API design with comprehensive error handling
- Database connection pooling for performance
- Plaid webhook support for real-time updates

**Data Layer** (Clean Architecture):
- **db_operations**: Low-level database access with prepared statements
- **handlers**: Business logic orchestration
- **processors**: Data transformation and validation
- **services**: Domain-specific business rules
- **utils**: Shared database connections and helpers

### Frontend Architecture

**React + Vite**:
- Modern React 18 with functional components
- TanStack Query for server state management
- Vite dev server with HTTPS and API proxy
- CSS Modules for component styling
- Chart.js and Plotly.js for data visualization

**Key Components**:
- Portfolio charts with real-time valuations
- Transaction importer with CSV parsing
- Net worth tracking with historical trends
- Cash flow analysis with customizable date ranges

### Database Design

**Transaction-Based Accounting**:
- All cash balances calculated from transaction sums
- No stored balances to prevent discrepancies
- FIFO stock sale tracking for accurate tax reporting

**Key Tables**:
- `portfolio_holdings`: Individual stock lots with cost basis
- `cash_transactions`: All cash movements with categorization
- `investment_cash_holdings`: Account metadata (balance always 0)
- `stock_sales`: Capital gains tracking with tax implications
- `plaid_api_calls`: Comprehensive API telemetry

**Optimized Views**:
- `current_investment_cash`: Real-time cash balances
- `net_worth_history`: Historical net worth calculations
- `cash_inflow/outflow_summary`: Cash flow analysis with transfer filtering

### Data Pipeline Architecture

**Apache Airflow DAGs**:
- Scheduled financial data synchronization
- Stock price updates from Yahoo Finance
- Net worth snapshot calculations
- Error handling and retry logic

**Performance Optimizations**:
- Cached market data (no API calls on page load)
- Batch processing for efficiency
- Incremental data loading
- Connection pooling

## 🚦 Getting Started

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

## 📊 Key Features in Detail

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

## 🔧 Configuration

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

## 📈 Monitoring & Observability

### API Telemetry
- All Plaid API calls tracked in `plaid_api_calls` table
- Response times, error rates, and rate limits monitored
- Request/response correlation for debugging

### Data Quality
- Automated validation checks
- Consistency monitoring
- Error detection and logging
- Historical quality trends

## 🧪 Testing

```bash
# Backend tests
pytest app/tests/

# Frontend tests
cd frontend && npm test

# Integration tests
python app/tests/integration/
```

## 📝 Development Guidelines

### Code Style
- Python: Follow PEP 8, use type hints
- JavaScript: ESLint configuration provided
- SQL: Use CTEs for complex queries
- Git: Conventional commits

### Best Practices
- Never store calculated balances
- Always validate CSV imports
- Use database transactions for data integrity
- Cache expensive calculations
- Log all external API calls

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes following coding standards
4. Add tests for new functionality
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- Plaid API for financial data access
- Apache Airflow for workflow orchestration
- React and Vite communities
- Chart.js and Plotly.js for visualizations