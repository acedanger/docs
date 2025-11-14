# Trading Analysis Dashboard - API Documentation

This document provides comprehensive documentation for all API endpoints available in the Trading Analysis Dashboard application.

## Base URL
All API endpoints are relative to the base URL of your application deployment.
- Development: `http://localhost:5000`
- Production: `https://your-production-domain.com`. Will be set in the `.env` file.

## Health Check Endpoints

These endpoints do not require authentication and are used for monitoring and orchestration.

### 1. Basic Health Check

**Endpoint:** `GET /health`

**Description:** Basic health check endpoint for load balancers and monitoring.

**Authentication:** Not required

**Sample Response:**
```json
{
  "status": "healthy",
  "timestamp": "2025-11-14T10:30:00.000000",
  "service": "stocks-trading-analysis"
}
```
HTTP Status: `200 OK`

---

### 2. Detailed Health Check

**Endpoint:** `GET /health/detailed`

**Description:** Comprehensive health check including database connectivity, environment, and disk space.

**Authentication:** Not required

**Sample Response:**
```json
{
  "status": "healthy",
  "timestamp": "2025-11-14T10:30:00.000000",
  "service": "stocks-trading-analysis",
  "checks": {
    "database": {
      "status": "healthy",
      "message": "Database connection successful"
    },
    "environment": {
      "status": "healthy",
      "message": "All required environment variables are set"
    },
    "disk_space": {
      "status": "healthy",
      "message": "Disk space OK: 45.23GB free"
    }
  }
}
```
HTTP Status: `200 OK` (healthy) or `503 Service Unavailable` (degraded)

---

### 3. Readiness Check

**Endpoint:** `GET /health/ready`

**Description:** Kubernetes readiness probe to determine if the app is ready to serve traffic.

**Authentication:** Not required

**Sample Response:**
```json
{
  "status": "ready",
  "timestamp": "2025-11-14T10:30:00.000000"
}
```
HTTP Status: `200 OK` (ready) or `503 Service Unavailable` (not ready)

---

### 4. Liveness Check

**Endpoint:** `GET /health/live`

**Description:** Kubernetes liveness probe to verify the application is running.

**Authentication:** Not required

**Sample Response:**
```json
{
  "status": "alive",
  "timestamp": "2025-11-14T10:30:00.000000"
}
```
HTTP Status: `200 OK`

---

## Authentication
Most API endpoints require OAuth 2.0 authentication. Users must be logged in and authorized to access protected endpoints. Health check endpoints do not require authentication.

### Authentication Headers
No special headers are required as authentication is handled via session cookies.

### Error Response for Unauthorized Access
```json
{
  "success": false,
  "error": "Authentication required",
  "redirect_to_login": true
}
```
HTTP Status: `401 Unauthorized`

---

## API Endpoints

### 1. Get Available Months

**Endpoint:** `GET /api/months`

**Description:** Retrieves a list of all months that have trading data available.

**Parameters:** None

**Sample Request:**
```http
GET /api/months
```

**Sample Response:**
```json
{
  "success": true,
  "months": [
    {
      "month": "2024-08",
      "total_return_with_dividends": 1250.75
    },
    {
      "month": "2024-07",
      "total_return_with_dividends": -320.50
    },
    {
      "month": "2024-06",
      "total_return_with_dividends": 890.25
    }
  ],
  "data_source": "postgresql"
}
```

**Response Fields:**
- `success` (boolean): Indicates if the request was successful
- `months` (array): List of month objects
  - `month` (string): Month in YYYY-MM format
  - `total_return_with_dividends` (number): Total return including dividends for that month
- `data_source` (string): Database source (always "postgresql")

**Error Response:**
```json
{
  "success": false,
  "error": "Database connection failed"
}
```
HTTP Status: `500 Internal Server Error`

---

### 2. Get Month Data

**Endpoint:** `GET /api/month/<month>`

**Description:** Retrieves detailed trading data for a specific month including summary, trades, and dividends.

**Parameters:**
- `month` (path parameter): Month in YYYY-MM format (e.g., "2024-08")

**Sample Request:**
```http
GET /api/month/2024-08
```

**Sample Response:**
```json
{
  "success": true,
  "summary": {
    "month": "2024-08",
    "total_trades": 15,
    "winning_trades": 9,
    "losing_trades": 6,
    "win_rate": 60.0,
    "trading_profit_loss": 850.75,
    "total_dividends": 125.50,
    "dividend_income": 125.50,
    "total_return_with_dividends": 976.25,
    "trading_pl": 850.75,
    "total_pl": 976.25
  },
  "trades": [
    {
      "symbol": "AAPL",
      "buy_date": "2024-08-01",
      "sell_date": "2024-08-15",
      "buy_price": 195.50,
      "sell_price": 198.75,
      "volume": 100,
      "total_profit_loss": 325.00,
      "return_percentage": 1.66,
      "trade_result": "Win"
    },
    {
      "symbol": "TSLA",
      "buy_date": "2024-08-10",
      "sell_date": "2024-08-20",
      "buy_price": 245.00,
      "sell_price": 240.50,
      "volume": 50,
      "total_profit_loss": -225.00,
      "return_percentage": -1.84,
      "trade_result": "Loss"
    }
  ],
  "dividends": [
    {
      "transaction_date": "2024-08-15",
      "symbol": "MSFT",
      "action": "Cash Dividend",
      "amount": 75.50
    },
    {
      "transaction_date": "2024-08-28",
      "symbol": "AAPL",
      "action": "Cash Dividend",
      "amount": 50.00
    }
  ],
  "data_source": "postgresql"
}
```

**Response Fields:**
- `success` (boolean): Request success status
- `summary` (object): Monthly summary statistics
  - `month` (string): Month in YYYY-MM format
  - `total_trades` (number): Total number of completed trades
  - `winning_trades` (number): Number of profitable trades
  - `losing_trades` (number): Number of losing trades
  - `win_rate` (number): Win rate percentage
  - `trading_profit_loss` (number): Total profit/loss from trades
  - `total_dividends` (number): Total dividend income
  - `total_return_with_dividends` (number): Combined trading P&L and dividends
- `trades` (array): List of completed trades
  - `symbol` (string): Stock symbol
  - `buy_date` (string): Purchase date (YYYY-MM-DD)
  - `sell_date` (string): Sale date (YYYY-MM-DD)
  - `buy_price` (number): Purchase price per share
  - `sell_price` (number): Sale price per share
  - `volume` (number): Number of shares traded
  - `total_profit_loss` (number): Total profit or loss for the trade
  - `return_percentage` (number): Return percentage
  - `trade_result` (string): "Win" or "Loss"
- `dividends` (array): List of dividend payments
  - `transaction_date` (string): Dividend payment date
  - `symbol` (string): Stock symbol
  - `action` (string): Type of dividend (e.g., "Cash Dividend")
  - `amount` (number): Dividend amount

**Error Responses:**
```json
{
  "success": false,
  "error": "Month not found"
}
```
HTTP Status: `404 Not Found`

```json
{
  "success": false,
  "error": "Database query failed"
}
```
HTTP Status: `500 Internal Server Error`

---

### 3. Get Trade Details

**Endpoint:** `GET /api/trade-details/<month>`

**Description:** Retrieves detailed trade information for a specific month (similar to trades in month data but with additional details).

**Parameters:**
- `month` (path parameter): Month in YYYY-MM format

**Sample Request:**
```http
GET /api/trade-details/2024-08
```

**Sample Response:**
```json
{
  "success": true,
  "trades": [
    {
      "symbol": "AAPL",
      "buy_date": "2024-08-01",
      "sell_date": "2024-08-15",
      "buy_price": 195.50,
      "sell_price": 198.75,
      "volume": 100,
      "profit_per_share": 3.25,
      "total_profit_loss": 325.00,
      "return_percentage": 1.66,
      "trade_result": "Win"
    }
  ],
  "data_source": "postgresql"
}
```

---

### 4. Get Timeframe Data

**Endpoint:** `GET /api/timeframe-data`

**Description:** Retrieves trading analysis data for a custom timeframe or all-time data with optional symbol filtering.

**Query Parameters:**
- `start` (string, optional): Start date in YYYY-MM-DD format
- `end` (string, optional): End date in YYYY-MM-DD format
- `all` (string, optional): Set to "true" for all-time data (ignores start/end dates)
- `symbols` (string, optional): Comma-separated list of stock symbols to filter by

**Sample Requests:**
```http
GET /api/timeframe-data?start=2024-01-01&end=2024-08-31
GET /api/timeframe-data?all=true
GET /api/timeframe-data?start=2024-06-01&end=2024-08-31&symbols=AAPL,TSLA,MSFT
```

**Sample Response:**
```json
{
  "success": true,
  "data": {
    "summary": {
      "trading_profit_loss": 2450.75,
      "total_dividends": 380.50,
      "total_trades": 45,
      "winning_trades": 28,
      "win_rate_percentage": 62.22
    },
    "weekly_summary": [
      {
        "week_start": "2024-08-26",
        "period": "2024-08-26",
        "trading_profit_loss": 150.25,
        "total_dividends": 25.00,
        "total_trades": 3,
        "winning_trades": 2,
        "win_rate_percentage": 66.67
      },
      {
        "week_start": "2024-08-19",
        "period": "2024-08-19",
        "trading_profit_loss": -75.50,
        "total_dividends": 50.00,
        "total_trades": 2,
        "winning_trades": 0,
        "win_rate_percentage": 0.0
      }
    ],
    "monthly_summary": [
      {
        "month_start": "2024-08-01",
        "period": "2024-08",
        "trading_profit_loss": 850.75,
        "total_dividends": 125.50,
        "total_trades": 15,
        "winning_trades": 9,
        "win_rate_percentage": 60.0
      },
      {
        "month_start": "2024-07-01",
        "period": "2024-07",
        "trading_profit_loss": -320.50,
        "total_dividends": 95.25,
        "total_trades": 12,
        "winning_trades": 4,
        "win_rate_percentage": 33.33
      }
    ],
    "open_positions": [
      {
        "symbol": "NVDA",
        "shares": 150
      },
      {
        "symbol": "AMD",
        "shares": 200
      }
    ]
  },
  "summary": {
    "trading_profit_loss": 2450.75,
    "total_dividends": 380.50,
    "total_trades": 45,
    "winning_trades": 28,
    "win_rate_percentage": 62.22
  },
  "weekly_summary": "...",
  "monthly_summary": "...",
  "open_positions": "...",
  "data_source": "postgresql"
}
```

**Response Fields:**
- `success` (boolean): Request success status
- `data` (object): Main data container
  - `summary` (object): Overall summary for the timeframe
    - `trading_profit_loss` (number): Total trading profit/loss
    - `total_dividends` (number): Total dividend income
    - `total_trades` (number): Number of completed trades
    - `winning_trades` (number): Number of profitable trades
    - `win_rate_percentage` (number): Win rate percentage
  - `weekly_summary` (array): Weekly breakdown data
    - `week_start` (string): Week start date
    - `period` (string): Display period
    - `trading_profit_loss` (number): Weekly trading P&L
    - `total_dividends` (number): Weekly dividend income
    - `total_trades` (number): Weekly trades count
    - `winning_trades` (number): Weekly winning trades
    - `win_rate_percentage` (number): Weekly win rate
  - `monthly_summary` (array): Monthly breakdown data (same structure as weekly)
  - `open_positions` (array): Current open positions
    - `symbol` (string): Stock symbol
    - `shares` (number): Number of shares held
- Additional top-level fields mirror the data object for backwards compatibility

**Error Responses:**
```json
{
  "success": false,
  "error": "Start date and end date are required"
}
```
HTTP Status: `400 Bad Request`

```json
{
  "success": false,
  "error": "Invalid date format. Use YYYY-MM-DD"
}
```
HTTP Status: `400 Bad Request`

---

### 5. Get Symbols

**Endpoint:** `GET /api/symbols`

**Description:** Retrieves a list of all distinct stock symbols that have been traded, along with company information.

**Parameters:** None

**Sample Request:**
```http
GET /api/symbols
```

**Sample Response:**
```json
{
  "success": true,
  "symbols": [
    {
      "symbol": "AAPL",
      "company_name": "Apple Inc."
    },
    {
      "symbol": "TSLA",
      "company_name": "Tesla, Inc."
    },
    {
      "symbol": "MSFT",
      "company_name": "Microsoft Corporation"
    },
    {
      "symbol": "NVDA",
      "company_name": "NVIDIA Corporation"
    }
  ],
  "data_source": "postgresql"
}
```

**Response Fields:**
- `success` (boolean): Request success status
- `symbols` (array): List of symbol objects
  - `symbol` (string): Stock ticker symbol
  - `company_name` (string): Company name or cleaned description
- `data_source` (string): Database source

### 6. Get Symbols Summary

**Endpoint:** `GET /api/symbols/summary`

**Description:** Retrieves comprehensive summary data for all traded symbols including trading stats, dividends, and open positions.

**Parameters:** None

**Sample Request:**
```http
GET /api/symbols/summary
```

**Sample Response:**
```json
{
  "success": true,
  "symbols": [
    {
      "symbol": "AAPL",
      "total_trades": 25,
      "trading_profit_loss": 3450.50,
      "winning_trades": 18,
      "win_rate_percentage": 72.0,
      "avg_profit_loss": 138.02,
      "avg_return_percentage": 2.15,
      "total_volume": 2500,
      "total_dividend_payments": 8,
      "total_dividends": 320.00,
      "total_return": 3770.50,
      "first_trade_date": "2024-01-15",
      "last_trade_date": "2024-08-30",
      "first_dividend_date": "2024-02-15",
      "last_dividend_date": "2024-08-15",
      "current_shares_held": 100,
      "has_open_position": true
    }
  ],
  "data_source": "postgresql"
}
```

**Response Fields:**
- `symbol` (string): Stock ticker symbol
- `total_trades` (number): Total number of completed trades
- `trading_profit_loss` (number): Total P&L from trades
- `winning_trades` (number): Number of profitable trades
- `win_rate_percentage` (number): Win rate percentage
- `avg_profit_loss` (number): Average profit/loss per trade
- `avg_return_percentage` (number): Average return percentage
- `total_volume` (number): Total shares traded
- `total_dividend_payments` (number): Number of dividend payments
- `total_dividends` (number): Total dividend income
- `total_return` (number): Combined trading P&L and dividends
- `first_trade_date` (string): Date of first trade
- `last_trade_date` (string): Date of most recent trade
- `first_dividend_date` (string): Date of first dividend
- `last_dividend_date` (string): Date of most recent dividend
- `current_shares_held` (number): Number of shares currently held
- `has_open_position` (boolean): Whether user has open position

---

### 7. Get Symbol Details

**Endpoint:** `GET /api/symbols/<symbol>`

**Description:** Retrieves detailed information for a specific symbol including all trades, dividends, and statistics.

**Parameters:**
- `symbol` (path parameter): Stock ticker symbol

**Sample Request:**
```http
GET /api/symbols/AAPL
```

**Sample Response:**
```json
{
  "success": true,
  "data": {
    "symbol": "AAPL",
    "summary": {
      "total_trades": 25,
      "trading_profit_loss": 3450.50,
      "winning_trades": 18,
      "win_rate_percentage": 72.0,
      "avg_profit_loss": 138.02,
      "min_profit_loss": -250.00,
      "max_profit_loss": 550.00,
      "avg_return_pct": 2.15,
      "min_return_pct": -3.5,
      "max_return_pct": 5.8,
      "avg_holding_days": 12,
      "total_volume": 2500,
      "total_cost_basis": 487500.00,
      "first_trade_date": "2024-01-15",
      "last_trade_date": "2024-08-30",
      "total_dividend_payments": 8,
      "total_dividends": 320.00,
      "avg_dividend_amount": 40.00,
      "first_dividend_date": "2024-02-15",
      "last_dividend_date": "2024-08-15",
      "total_return": 3770.50,
      "current_shares_held": 100
    },
    "trades": [
      {
        "buy_date": "2024-08-01",
        "sell_date": "2024-08-15",
        "buy_price": 195.50,
        "sell_price": 198.75,
        "volume": 100,
        "total_profit_loss": 325.00,
        "return_percentage": 1.66,
        "trade_result": "Win",
        "holding_days": 14
      }
    ],
    "dividends": [
      {
        "transaction_date": "2024-08-15",
        "action": "Cash Dividend",
        "amount": 40.00
      }
    ]
  },
  "data_source": "postgresql"
}
```

---

### 8. Get Calendar Data

**Endpoint:** `GET /api/calendar-data`

**Description:** Retrieves daily trading activity data for calendar view including weekly aggregates.

**Query Parameters:**
- `year` (number, optional): Year to display (defaults to current year)
- `month` (number, optional): Month to display 1-12 (defaults to current month)

**Sample Request:**
```http
GET /api/calendar-data?year=2024&month=8
```

**Sample Response:**
```json
{
  "success": true,
  "year": 2024,
  "month": 8,
  "days": [
    {
      "date": "2024-08-01",
      "day": 1,
      "day_of_week": 4,
      "trade_count": 3,
      "trading_pnl": 150.25,
      "dividend_count": 0,
      "dividends": 0,
      "total_pnl": 150.25,
      "has_activity": true,
      "week_trade_count": 12,
      "week_winning_trades": 8,
      "week_total_pnl": 450.75,
      "week_win_rate": 66.67
    }
  ],
  "data_source": "postgresql"
}
```

---

### 9. Get Day Details

**Endpoint:** `GET /api/day-details`

**Description:** Retrieves detailed trades and dividends for a specific day.

**Query Parameters:**
- `date` (string, required): Date in YYYY-MM-DD format

**Sample Request:**
```http
GET /api/day-details?date=2024-08-15
```

**Sample Response:**
```json
{
  "success": true,
  "date": "2024-08-15",
  "trades": [
    {
      "symbol": "AAPL",
      "buy_date": "2024-08-01",
      "sell_date": "2024-08-15",
      "buy_price": 195.50,
      "sell_price": 198.75,
      "volume": 100,
      "total_profit_loss": 325.00,
      "return_percentage": 1.66,
      "trade_result": "Win"
    }
  ],
  "dividends": [
    {
      "symbol": "MSFT",
      "transaction_date": "2024-08-15",
      "action": "Cash Dividend",
      "amount": 75.50
    }
  ],
  "summary": {
    "date": "2024-08-15",
    "total_trading_pnl": 325.00,
    "total_dividends": 75.50,
    "total_pnl": 400.50,
    "trade_count": 1,
    "dividend_count": 1,
    "winning_trades": 1,
    "win_rate": 100.0
  },
  "data_source": "postgresql"
}
```

---

### 10. Export Calendar Day Data

**Endpoint:** `GET /api/calendar/export/<format>`

**Description:** Export trades for a specific day in CSV, Excel, or PDF format.

**Parameters:**
- `format` (path parameter): Export format - `csv`, `excel`, or `pdf`

**Query Parameters:**
- `date` (string, required): Date in YYYY-MM-DD format

**Sample Request:**
```http
GET /api/calendar/export/csv?date=2024-08-15
```

**Response:** File download in requested format

**Error Responses:**
```json
{
  "success": false,
  "error": "No trades found for this date"
}
```
HTTP Status: `404 Not Found`

---

### 11. Get Trading Calendar Metrics

**Endpoint:** `GET /api/trading-calendar/metrics`

**Description:** Retrieves trading calendar metrics including market open/close times and trading days. Public endpoint (no authentication required).

**Authentication:** Not required

**Query Parameters:** None

**Sample Response:**
```json
{
  "success": true,
  "metrics": {
    "current_time": "2024-08-15T14:30:00",
    "market_status": "open",
    "market_open_time": "09:30:00",
    "market_close_time": "16:00:00",
    "is_trading_day": true,
    "next_trading_day": "2024-08-16",
    "trading_days_this_month": 22,
    "trading_days_this_year": 252
  }
}
```

---

### 12. Export Monthly Data

**Endpoint:** `GET /api/monthly/export/<format>`

**Description:** Export monthly trading data in CSV, Excel, or PDF format.

**Parameters:**
- `format` (path parameter): Export format - `csv`, `excel`, or `pdf`

**Query Parameters:**
- `month` (string, required): Month in YYYY-MM format

**Sample Request:**
```http
GET /api/monthly/export/pdf?month=2024-08
```

**Response:** File download in requested format with monthly summary, trades, and dividends

---

### 13. Upload Transaction History CSV

**Endpoint:** `POST /api/upload-csv`

**Description:** Upload and process transaction history and realized gains CSV files from brokerage.

**Content-Type:** `multipart/form-data`

**Parameters:**
- `transaction_history_file` (file, required): Transaction history CSV file
- `realized_gains_file` (file, required): Realized gains CSV file

**Sample Response (streaming JSON):**
```json
{"type": "progress", "percentage": 25, "message": "Starting data processing..."}
{"type": "progress", "percentage": 40, "message": "Processing transaction history and realized gains..."}
{"type": "progress", "percentage": 60, "message": "Synchronizing with database..."}
{"type": "progress", "percentage": 80, "message": "Finalizing data import..."}
{"type": "success", "message": "Successfully processed files", "stats": {"transactions_inserted": 279, "realized_gains_lots": 713, "broker_verified_trades": 156}}
```

**Error Response:**
```json
{"type": "error", "message": "Both files must be CSV files"}
```
HTTP Status: `400 Bad Request`

**Note:** Requires user to have brokerage account number set in profile.

---

### 14. Get Upload History

**Endpoint:** `GET /api/upload-history`

**Description:** Retrieves history of CSV file uploads with processing statistics.

**Sample Response:**
```json
{
  "success": true,
  "history": [
    {
      "transaction_filename": "transactions.csv",
      "gains_filename": "realized_gains.csv",
      "timestamp": "2024-08-15T14:30:00",
      "transaction_file_size": 524288,
      "gains_file_size": 262144,
      "status": "success",
      "user_email": "user@example.com",
      "account_id": 1,
      "brokerage_account": "12345678",
      "transactions_processed": 279,
      "months_updated": 3
    }
  ]
}
```

---

## Portfolio Management Endpoints

### 1. Get Portfolio Holdings

**Endpoint:** `GET /api/portfolio/holdings`

**Description:** Retrieves all portfolio holdings for the current user with cached prices.

**Sample Response:**
```json
{
  "success": true,
  "holdings": [
    {
      "id": 1,
      "symbol": "AAPL",
      "security_type": "STOCK",
      "holding_type": "stock",
      "shares": 100,
      "cost_basis": 150.50,
      "average_cost": 150.50,
      "purchase_date": "2024-01-15",
      "notes": "Long-term hold",
      "current_price": 195.50,
      "change": 2.50,
      "percent_change": 1.3,
      "market_value": 19550.00,
      "total_cost": 15050.00,
      "total_gain_loss": 4500.00,
      "gain_loss_percent": 29.9,
      "price_updated_at": "2024-08-15T14:30:00"
    }
  ]
}
```

---

### 2. Add Portfolio Holding

**Endpoint:** `POST /api/portfolio/holdings`

**Description:** Add a new holding to the portfolio.

**Request Body:**
```json
{
  "symbol": "AAPL",
  "shares": 100,
  "cost_basis": 150.50,
  "security_type": "STOCK",
  "purchase_date": "2024-01-15",
  "notes": "Long-term investment"
}
```

**Response:**
```json
{
  "success": true,
  "holding": {
    "id": 1,
    "symbol": "AAPL",
    "security_type": "STOCK",
    "shares": 100,
    "cost_basis": 150.50,
    "purchase_date": "2024-01-15",
    "notes": "Long-term investment",
    "created_at": "2024-08-15T14:30:00"
  },
  "message": "Added AAPL to portfolio"
}
```

---

### 3. Update Portfolio Holding

**Endpoint:** `PUT /api/portfolio/holdings/<holding_id>`

**Description:** Update an existing portfolio holding.

**Parameters:**
- `holding_id` (path parameter): ID of the holding to update

**Request Body:**
```json
{
  "shares": 150,
  "cost_basis": 155.00,
  "notes": "Increased position"
}
```

**Response:**
```json
{
  "success": true,
  "holding": {
    "id": 1,
    "symbol": "AAPL",
    "security_type": "STOCK",
    "shares": 150,
    "cost_basis": 155.00,
    "notes": "Increased position",
    "updated_at": "2024-08-15T15:00:00"
  },
  "message": "Holding updated successfully"
}
```

---

### 4. Delete Portfolio Holding

**Endpoint:** `DELETE /api/portfolio/holdings/<holding_id>`

**Description:** Remove a holding from the portfolio.

**Parameters:**
- `holding_id` (path parameter): ID of the holding to delete

**Response:**
```json
{
  "success": true,
  "message": "Removed AAPL from portfolio"
}
```

---

### 5. Validate Symbol

**Endpoint:** `GET /api/portfolio/validate-symbol/<symbol>`

**Description:** Validate a stock symbol using Finnhub API.

**Parameters:**
- `symbol` (path parameter): Stock symbol to validate

**Response:**
```json
{
  "valid": true,
  "symbol": "AAPL",
  "current_price": 195.50,
  "message": "AAPL is a valid symbol"
}
```

**Error Response:**
```json
{
  "valid": false,
  "symbol": "INVALID",
  "error": "Symbol \"INVALID\" not found or invalid"
}
```

---

### 6. Get Symbol Price

**Endpoint:** `GET /api/portfolio/price/<symbol>`

**Description:** Get current price for a symbol and update cache.

**Parameters:**
- `symbol` (path parameter): Stock symbol

**Response:**
```json
{
  "success": true,
  "symbol": "AAPL",
  "price_data": {
    "current_price": 195.50,
    "change": 2.50,
    "percent_change": 1.3,
    "high": 196.75,
    "low": 193.25,
    "open": 194.00,
    "previous_close": 193.00
  }
}
```

---

### 7. Refresh Portfolio Prices

**Endpoint:** `POST /api/portfolio/refresh-prices`

**Description:** Force refresh all portfolio prices from Finnhub API.

**Response:**
```json
{
  "success": true,
  "prices": {
    "AAPL": {
      "c": 195.50,
      "d": 2.50,
      "dp": 1.3,
      "h": 196.75,
      "l": 193.25,
      "o": 194.00,
      "pc": 193.00
    }
  },
  "message": "Refreshed prices for 5 symbols"
}
```

---

### 8. Capture Portfolio Snapshot

**Endpoint:** `POST /api/portfolio/snapshot`

**Description:** Capture a snapshot of current portfolio value for historical tracking.

**Request Body:**
```json
{
  "snapshot_type": "MANUAL",
  "notes": "Monthly review"
}
```

**Response:**
```json
{
  "success": true,
  "snapshot": {
    "id": 1,
    "snapshot_time": "2024-08-15T14:30:00",
    "total_value": 195500.00,
    "total_cost": 150500.00,
    "total_gain_loss": 45000.00,
    "total_return_pct": 29.9,
    "holdings_count": 10
  }
}
```

---

### 9. Get Portfolio History

**Endpoint:** `GET /api/portfolio/history`

**Description:** Get historical portfolio value snapshots for charting.

**Query Parameters:**
- `days` (number, optional): Number of days to retrieve (default: 30, 0 for all)
- `type` (string, optional): Filter by snapshot type (e.g., "MANUAL", "AUTOMATIC")

**Sample Request:**
```http
GET /api/portfolio/history?days=90&type=AUTOMATIC
```

**Response:**
```json
{
  "history": [
    {
      "id": 1,
      "snapshot_time": "2024-08-15T14:30:00",
      "total_value": 195500.00,
      "total_cost": 150500.00,
      "total_gain_loss": 45000.00,
      "total_return_pct": 29.9,
      "holdings_count": 10,
      "snapshot_type": "AUTOMATIC",
      "notes": ""
    }
  ]
}
```

---

### 10. Delete Portfolio Snapshot

**Endpoint:** `DELETE /api/portfolio/snapshot/<snapshot_id>`

**Description:** Delete a portfolio snapshot.

**Parameters:**
- `snapshot_id` (path parameter): ID of the snapshot to delete

**Response:**
```json
{
  "success": true,
  "message": "Deleted MANUAL snapshot from 2024-08-15T14:30:00"
}
```

---

### 11. Upload Portfolio CSV

**Endpoint:** `POST /api/portfolio/upload-csv` or `POST /api/portfolio/upload`

**Description:** Upload CSV file with portfolio holdings (bulk import).

**Content-Type:** `multipart/form-data`

**Parameters:**
- `csv_file` (file, required): CSV file with holdings

**CSV Format:**
Required columns: `symbol`, `shares`
Optional columns: `cost_basis`, `average_cost`, `security_type`, `holding_type`, `type`, `purchase_date`, `notes`

**Sample Response:**
```json
{
  "success": true,
  "imported": 15,
  "updated": 3,
  "errors": [
    "Row 5: Invalid shares value for TSLA"
  ],
  "message": "Successfully imported 15 holdings and updated 3 existing holdings"
}
```

---

## Authentication Endpoints

### 1. Login Page

**Endpoint:** `GET /auth/login`

**Description:** Displays the login page for unauthenticated users.

**Sample Response:** HTML login page

---

### 2. Initiate OAuth Login

**Endpoint:** `GET /login`

**Description:** Redirects user to Google OAuth authorization page.

**Sample Response:** HTTP 302 redirect to Google OAuth

---

### 3. OAuth Callback

**Endpoint:** `GET /auth/callback`

**Description:** Handles the OAuth callback from Google and creates user session.

**Query Parameters:**
- `code` (string): Authorization code from OAuth provider
- `state` (string): State parameter for security

**Sample Response:** HTTP 302 redirect to dashboard or login page with flash message

---

### 4. Logout

**Endpoint:** `GET /logout`

**Description:** Clears user session and logs out the user.

**Sample Response:** HTTP 302 redirect to login page with success message

---

### 5. User Profile

**Endpoint:** `GET /auth/profile`

**Description:** Displays user profile information (requires authentication).

**Sample Response:** HTML profile page with user information

---

## Application Routes

### 1. Dashboard Home

**Endpoint:** `GET /`

**Description:** Main dashboard page - redirects to portfolio page (requires authentication).

**Sample Response:** Redirect to `/portfolio`

---

### 2. Portfolio Page

**Endpoint:** `GET /portfolio`

**Description:** Portfolio management page for tracking holdings (requires authentication).

**Sample Response:** HTML portfolio page

---

### 3. Monthly Dashboard

**Endpoint:** `GET /dashboard`

**Description:** Monthly trading dashboard page (requires authentication).

**Sample Response:** HTML dashboard page

---

### 4. Summary Analysis

**Endpoint:** `GET /summary`

**Description:** Summary analysis page (requires authentication).

**Sample Response:** HTML summary page

---

### 5. Timeframe Analysis

**Endpoint:** `GET /summary/timeframe`

**Description:** Timeframe analysis page (requires authentication).

**Sample Response:** HTML timeframe analysis page

---

### 6. Calendar View

**Endpoint:** `GET /summary/calendar`

**Description:** Trading calendar view page (requires authentication).

**Sample Response:** HTML calendar page

---

### 7. Summary by Symbol

**Endpoint:** `GET /summary/symbol`

**Description:** Symbol summary list page (requires authentication).

**Sample Response:** HTML symbol summary page

---

### 8. Symbol Detail Page

**Endpoint:** `GET /summary/symbol/<symbol>`

**Description:** Individual symbol detail page (requires authentication).

**Parameters:**
- `symbol` (path parameter): Stock ticker symbol

**Sample Response:** HTML symbol detail page

---

### 9. Trading Calendar (Public)

**Endpoint:** `GET /trading-calendar`

**Description:** Trading calendar metrics page - public access, no authentication required.

**Sample Response:** HTML trading calendar page

---

### 10. Upload Page

**Endpoint:** `GET /upload`

**Description:** CSV file upload page for importing brokerage data (requires authentication).

**Sample Response:** HTML upload page

---

## Error Handling

### Global Error Pages

**404 Not Found:**
```http
GET /nonexistent-page
```
Returns custom 404.html template
HTTP Status: `404 Not Found`

**500 Internal Server Error:**
Returns custom 500.html template for server errors
HTTP Status: `500 Internal Server Error`

### API Error Format
All API endpoints return errors in this format:
```json
{
  "success": false,
  "error": "Error message describing what went wrong"
}
```

### Common HTTP Status Codes
- `200 OK` - Request successful
- `400 Bad Request` - Invalid request parameters
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - Access denied (user not authorized)
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error
- `503 Service Unavailable` - Service temporarily unavailable

## Rate Limiting
Currently, no rate limiting is implemented on API endpoints.

## Data Types and Formats

### Date Formats
- **Month:** YYYY-MM (e.g., "2024-08")
- **Date:** YYYY-MM-DD (e.g., "2024-08-15")
- **Datetime/Timestamp:** ISO 8601 format (e.g., "2024-08-15T14:30:00" or "2024-08-15T14:30:00.000000")

### Currency
All monetary values are returned as numbers representing USD amounts (e.g., 1250.75 = $1,250.75).

### Decimal Precision
- Monetary values: Up to 2 decimal places
- Percentages: Up to 2 decimal places
- Share quantities: Up to 2 decimal places (supports fractional shares)
- Prices: Up to 2 decimal places

### Trade Types
- **Long:** Standard buy and sell trades
- **Short:** Short selling trades (Sell Short → Buy to Cover)

### Security Types
- **STOCK:** Common stock
- **ETF:** Exchange-traded fund
- **OPTION:** Stock option
- **BOND:** Bond security
- **CRYPTO:** Cryptocurrency

### Snapshot Types
- **MANUAL:** User-initiated snapshot
- **AUTOMATIC:** System-generated snapshot (e.g., daily, weekly)
- **EOD:** End of day snapshot

## Database Schema Context

The API operates on the following main database views/tables:
- `trading_analysis.monthly_combined_summary` - Monthly aggregated data
- `trading_analysis.matched_trades` - Individual completed trades
- `trading_analysis.dividend_transactions` - Dividend payments
- `trading_analysis.raw_transactions` - Raw transaction data

## Sample Integration Code

### JavaScript Fetch Example
```javascript
// Get available months
async function fetchMonths() {
  try {
    const response = await fetch('/api/months');
    const data = await response.json();

    if (data.success) {
      console.log('Available months:', data.months);
    } else {
      console.error('Error:', data.error);
    }
  } catch (error) {
    console.error('Network error:', error);
  }
}

// Get month data with error handling
async function fetchMonthData(month) {
  try {
    const response = await fetch(`/api/month/${month}`);

    if (response.status === 401) {
      // Handle authentication required
      window.location.href = '/auth/login';
      return;
    }

    const data = await response.json();

    if (data.success) {
      return data;
    } else {
      throw new Error(data.error);
    }
  } catch (error) {
    console.error('Failed to fetch month data:', error);
    throw error;
  }
}
```

### Python Requests Example
```python
import requests

# Assuming you have a session with authentication cookies
session = requests.Session()

def get_timeframe_data(start_date, end_date, symbols=None):
    params = {
        'start': start_date,
        'end': end_date
    }

    if symbols:
        params['symbols'] = ','.join(symbols)

    response = session.get(
        'http://localhost:5000/api/timeframe-data',
        params=params
    )

    if response.status_code == 401:
        raise Exception("Authentication required")

    data = response.json()

    if not data['success']:
        raise Exception(f"API Error: {data['error']}")

    return data['data']
```

This documentation covers all available API endpoints with comprehensive examples and error handling scenarios. The API is designed to be RESTful and returns consistent JSON responses with proper HTTP status codes.
