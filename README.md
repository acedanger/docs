# Trading Analysis Dashboard Documentation

This repository contains the documentation for the Trading Analysis Dashboard application - a Flask-based web platform for tracking and analyzing stock trading performance with real-time portfolio management.

## Documentation Structure

- **Overview** - Application architecture, data models, and core concepts
- **Features** - Detailed documentation of portfolio tracking, trading analysis, and data import capabilities
- **API Reference** - RESTful API endpoint documentation with examples

## Local Development

Install the [Mintlify CLI](https://www.npmjs.com/package/mint) to preview documentation changes locally:

```bash
npm i -g mint
```

Run the following command at the root of the documentation directory:

```bash
mint dev
```

View your local preview at `http://localhost:3000`.

## Publishing Changes

Changes are automatically deployed to production after pushing to the default branch via GitHub integration.

## About the Application

The Trading Analysis Dashboard provides:

- **Portfolio Management** - Track stocks, ETFs, and mutual funds with real-time pricing
- **Trading Analysis** - Monthly P&L reports with trade-by-trade detail
- **Hybrid Matching** - Broker-level accuracy combining realized gains and transaction history
- **CSV Import** - Bulk data import for holdings and transactions
- **Multi-User Support** - OAuth authentication with isolated user data

Built with Flask, PostgreSQL, and integrating with Finnhub for real-time market data.
