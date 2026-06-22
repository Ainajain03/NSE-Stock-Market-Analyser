# StockSage AI

A full-stack Indian NSE stock market intelligence platform. Bloomberg-meets-Blade-Runner dark aesthetic with 5 analysis pages, 10 legendary investor AI personas, and real-time technical/fundamental/sentiment analysis.

## Architecture

**Monorepo (pnpm workspaces)**

```
artifacts/
  api-server/      — Express 5 backend (port 8080, proxied via /api)
  stocksage/       — React+Vite frontend (port 21773, proxied via /)
lib/
  api-spec/        — OpenAPI 3.0 spec (source of truth)
  api-client-react/ — Generated React Query hooks (from openapi spec)
  api-zod/         — Generated Zod schemas
  db/              — Drizzle ORM + PostgreSQL schema
  integrations-anthropic-ai/ — Anthropic Claude AI client
```

## Key Tech

- **Frontend**: React 18, Vite, Tailwind v4, shadcn/ui, Framer Motion, lightweight-charts (TradingView), Zustand, Wouter routing
- **Backend**: Express 5, Drizzle ORM, PostgreSQL
- **Market Data**: Yahoo Finance API (unofficial, NSE stocks via `.NS` suffix, 15-min delayed, in-memory TTL cache)
- **AI**: Claude claude-sonnet-4-6 via Replit AI Integrations (no direct API key needed)
- **Code Generation**: Orval generates React Query hooks from OpenAPI spec

## Pages

1. `/dashboard` — Command Center: stock search, 4 score cards (Technical/Fundamental/Sentiment/Overall), Final AI Verdict button with entry zone + stop-loss + targets
2. `/technical` — Technical Analysis: TradingView candlestick chart (with volume), 10+ indicators (RSI, MACD, Bollinger Bands, ADX, Supertrend, etc.), AI summary
3. `/fundamental` — Fundamental Analysis: 5-year financials, ratios, quarterly results, valuation (DCF, Graham Number), peers
4. `/sentiment` — Sentiment Analysis: Fear & Greed gauge, FII/DII flows, news cards, AI synthesis
5. `/experts` — 10 legendary investors (Jhunjhunwala, Damani, Kedia, Buffett, Graham, Lynch, Soros, Livermore, Munger, Minervini)
6. `/experts/:id` — Expert Chat: full-page AI chat with each expert's persona, stock context awareness

## Global Features

- Floating AI chatbot (bottom-right FAB) for global StockSage AI queries
- Stock search autocomplete (285 NSE stocks across Nifty50, Nifty500, midcap, smallcap, sector-based search)
- Live mini-ticker: Nifty50, Bank Nifty, Sensex (real-time from Yahoo Finance)
- Active strategy badge when expert selected (changes UI accent color)
- Zustand stores: `stockStore` (selected stock) + `strategyStore` (active expert strategy)

## Data Layer

- **Real-time quotes**: `artifacts/api-server/src/lib/yahooFinance.ts` — Yahoo Finance wrapper with TTL cache (60s quotes, 5min daily OHLCV, 60s intraday)
- **Stock universe**: `artifacts/api-server/src/lib/nseStocks.ts` — 285 NSE stocks covering Nifty50, Next50, Midcap100, Smallcap, all sectors
- **Symbol mapping**: NSE `M_M` → Yahoo `M&M.NS`, indices `^NSEI` `^NSEBANK` `^BSESN`; fallback to mock if Yahoo fails
- **Mock math**: `artifacts/api-server/src/lib/mockData.ts` — technical indicator math (SMA, EMA, RSI, MACD, Bollinger, ATR), expert profiles, OHLCV fallback generator

## Backend Routes

All under `/api`:
- `GET /stocks/search?q=` — search by symbol, name, or sector across 285 stocks
- `GET /stocks/market-overview` — Nifty50, BankNifty, Sensex (live from Yahoo)
- `GET /stocks/:symbol/quote` — live quote (Yahoo Finance → mock fallback)
- `GET /stocks/:symbol/historical?timeframe=` — OHLCV candles (Yahoo → mock fallback)
- `GET /technical/:symbol?timeframe=` — 10+ technical indicators + signals (real OHLCV)
- `POST /technical/:symbol/ai-summary` — Claude AI technical analysis
- `GET /fundamental/:symbol` — 5yr financials, ratios, DCF valuation, peers
- `POST /fundamental/:symbol/ai-analysis` — Claude AI fundamental analysis
- `GET /sentiment/market` — Fear & Greed, FII/DII flows (MUST be before /:symbol)
- `GET /sentiment/:symbol` — stock-specific sentiment
- `POST /sentiment/:symbol/ai-summary` — Claude AI sentiment summary
- `GET /experts` — all 10 expert profiles
- `POST /experts/:expertId/chat` — AI chat with expert persona
- `GET /experts/:expertId/analyze/:symbol` — expert analyzes a stock (live price)
- `GET /dashboard/:symbol` — aggregated scores + benchmark comparison
- `POST /dashboard/:symbol/final-verdict` — comprehensive Claude AI analysis
- `GET /watchlist` / `POST /watchlist` / `DELETE /watchlist/:id` — PostgreSQL watchlist (live prices)
- `POST /chat/global` — global StockSage AI chatbot

## Database

PostgreSQL via Replit. Single `watchlist` table (id, symbol, name, added_at).

## Environment Variables

- `DATABASE_URL` — PostgreSQL connection
- `AI_INTEGRATIONS_ANTHROPIC_BASE_URL` — Anthropic proxy URL
- `AI_INTEGRATIONS_ANTHROPIC_API_KEY` — Anthropic API key (via Replit AI Integrations)
- `SESSION_SECRET` — session secret

## Codegen

To regenerate API client after OpenAPI spec changes:
```
pnpm --filter @workspace/api-spec run codegen
```

## Important Notes

- `/sentiment/market` MUST be registered before `/sentiment/:symbol` in Express (static before dynamic)
- All frontend types import from `@workspace/api-client-react` (not deep `/src/generated/api.schemas`)
- `lib/api-zod/src/index.ts` must only export from `./generated/api` (not `./generated/types`)
- The API server bundles everything with esbuild — restart the workflow after any backend changes
- Yahoo Finance blocks some server-side requests; always wrap in try/catch with mock fallback
- `lightweight-charts` uses vanilla JS API — mount in `useEffect`, use `ResizeObserver` for width, sort+dedup data before `setData`
