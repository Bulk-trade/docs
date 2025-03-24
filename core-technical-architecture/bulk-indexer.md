# Bulk-indexer

## Bulk DeFi Indexer Technical Documentation

### Overview

The Bulk DeFi Indexer is a high-performance data indexing and API service designed to track and analyze vault activities on the Solana blockchain. It processes vault transactions, calculates metrics, and provides real-time data access for the frontend application.

### System Architecture

#### Tech Stack

1. **Backend Framework**
   * Rust (Core language)
   * Axum (Web framework)
   * Tower-HTTP (Middleware)
2. **Database**
   * TimescaleDB (Time-series PostgreSQL extension)
   * Features used:
     * Hypertables for time-series data
     * Continuous aggregates
     * Retention policies
3. **Blockchain Integration**
   * Solana SDK
   * Anchor Client
   * Custom RPC client implementation

#### Core Components

1.  **Transaction Processor**

    ```rust
    ```

    * Receives transaction notifications from frontend
    * Verifies transactions on-chain
    * Updates database with transaction data
2.  **Metrics Calculator**

    ```rust
    ```

    * Calculates TVL, APY, PnL
    * Processes share price updates
    * Handles performance fee calculations
3.  **API Server**

    ```rust
    ```

    * Provides REST endpoints
    * Handles real-time data requests
    * Manages user authentication

### Database Schema

#### 1. Vault Metrics Table

```sql
CREATE TABLE vault_metrics (
    time TIMESTAMPTZ NOT NULL,
    vault_address TEXT NOT NULL,
    tvl NUMERIC NOT NULL,
    volume_24h NUMERIC NOT NULL,
    pnl NUMERIC NOT NULL,
    apy NUMERIC NOT NULL,
    share_price NUMERIC NOT NULL
);
```

#### 2. User Transactions Table

```sql
CREATE TABLE user_transactions (
    time TIMESTAMPTZ NOT NULL,
    user_address TEXT NOT NULL,
    vault_address TEXT NOT NULL,
    transaction_type TEXT NOT NULL,
    amount NUMERIC NOT NULL,
    shares NUMERIC NOT NULL,
    signature TEXT NOT NULL UNIQUE,
    status TEXT NOT NULL,
    market_index INTEGER NOT NULL
);
```

#### 3. Pending Withdrawals Table

```sql
CREATE TABLE pending_withdrawals (
    id SERIAL PRIMARY KEY,
    user_address TEXT NOT NULL,
    vault_address TEXT NOT NULL,
    amount NUMERIC NOT NULL,
    requested_at TIMESTAMPTZ NOT NULL,
    status TEXT NOT NULL,
    market_index INTEGER NOT NULL,
    signature TEXT NOT NULL UNIQUE
);
```

### Data Flow

1.  **Transaction Indexing**

    ```mermaid
    graph LR
        A[Frontend] --> B[API Server]
        B --> C[Transaction Processor]
        C --> D[Solana RPC]
        C --> E[Database]
    ```
2.  **Metrics Calculation**

    ```mermaid
    graph LR
        A[Scheduled Job] --> B[Metrics Calculator]
        B --> C[Database Query]
        C --> D[Calculation Logic]
        D --> E[Database Update]
    ```

### API Endpoints

1.  **Transaction Notification**

    ```http
    POST /api/transaction
    Content-Type: application/json

    {
      "signature": "string",
      "vault_address": "string",
      "user_address": "string",
      "amount": "string",
      "market_index": number,
      "transaction_type": "DEPOSIT" | "WITHDRAW" | "PENDING_WITHDRAW"
    }
    ```
2.  **User Positions**

    ```http
    GET /api/user/:address/positions
    ```
3.  **Vault Metrics**

    ```http
    GET /api/vault/:address/metrics
    ```

### Performance Optimizations

1. **Database Optimizations**
   * TimescaleDB Hypertables for efficient time-series queries
   * Proper indexing on frequently queried columns
   * Partitioning by time for faster historical data access
2. **Caching Strategy**
   * In-memory caching for frequently accessed metrics
   * Cache invalidation based on transaction updates
   * Periodic cache refresh for time-based metrics
3. **Query Optimizations**
   * Materialized views for complex calculations
   * Continuous aggregates for historical data
   * Efficient pagination for large datasets

### Security Measures

1. **Transaction Verification**
   * On-chain signature verification
   * Double-spend protection
   * Invalid transaction detection
2. **Data Validation**
   * Input sanitization
   * Type checking
   * Amount validation
3. **API Security**
   * Rate limiting
   * CORS configuration
   * Error handling

### Monitoring and Maintenance

1. **Health Checks**
   * Database connection monitoring
   * RPC node health checks
   * API endpoint monitoring
2. **Data Maintenance**
   * Regular database cleanup
   * Old data archival
   * Index optimization
3. **Performance Monitoring**
   * Query performance tracking
   * Transaction processing time
   * API response times

### Error Handling

1.  **Error Types**

    ```rust
    pub enum IndexerError {
        DatabaseError(sqlx::Error),
        RpcError(solana_client::client_error::ClientError),
        TransactionNotFound(String),
        InvalidTransactionData(String),
        DeserializationError(String),
        CalculationError(String),
    }
    ```
2. **Recovery Strategies**
   * Automatic retry for transient failures
   * Graceful degradation
   * Error reporting and logging

###
