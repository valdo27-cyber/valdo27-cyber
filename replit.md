# TradeAI Pro

A professional day trading analysis platform with AI-powered signals, real-time market data, and Bloomberg-style dark UI for stocks, crypto, futures, and forex.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080, proxied at `/api`)
- `pnpm --filter @workspace/trading-platform run dev` — run the React frontend (proxied at `/`)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string
- Required env: `AI_INTEGRATIONS_OPENAI_BASE_URL`, `AI_INTEGRATIONS_OPENAI_API_KEY` — via Replit OpenAI integration

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 (`artifacts/api-server`)
- Frontend: React + Vite + Tailwind CSS + Recharts + Framer Motion + Wouter (`artifacts/trading-platform`)
- DB: PostgreSQL + Drizzle ORM (`lib/db`)
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval from OpenAPI spec (`lib/api-spec`)
- AI: OpenAI `gpt-4o-mini` via Replit AI integration (`@workspace/integrations-openai-ai-server`)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — source of truth for all API contracts
- `lib/db/src/schema/` — DB schema files (watchlist, journal, alerts, conversations, messages)
- `artifacts/api-server/src/routes/` — Express route handlers (market, analysis, news, watchlist, journal, alerts, dashboard)
- `artifacts/api-server/src/lib/market-data.ts` — simulated market data + CoinGecko crypto + technical indicators
- `artifacts/trading-platform/src/pages/` — React pages (dashboard, chart, watchlist, news, journal, alerts, performance)
- `lib/api-client-react/src/generated/` — generated React Query hooks (do not edit manually)
- `lib/api-zod/src/generated/` — generated Zod schemas (do not edit manually)

## Architecture decisions

- Contract-first API: OpenAPI spec is the source of truth; all hooks and validation schemas are generated from it via Orval
- Market data is simulated with seeded pseudo-random (deterministic per symbol+bar) + CoinGecko for live crypto prices
- All DB date fields are serialized to ISO strings in route handlers (Drizzle returns JS Date objects, OpenAPI expects strings)
- AI signals use `gpt-4o-mini` with `response_format: { type: "json_object" }` for reliable structured output; graceful fallback to heuristic signals on error
- Shared proxy routes `/api` to the Express server and `/` to the Vite dev server — no custom proxy config needed in Vite

## Product

- **Dashboard**: KPI cards (PnL, win rate, open trades), NVDA price area chart, top AI signal panel, scrolling ticker tape
- **Analysis (Chart)**: Symbol search, price+volume chart, AI BUY/SELL/HOLD signal with buy/sell probabilities and technical analysis
- **Watchlist**: Live-updating prices for tracked assets; add/remove symbols
- **News**: AI-generated + curated financial news with sentiment badges; AI market sentiment gauge
- **Journal**: Trade log with entry/exit tracking, close-trade workflow, PnL calculation
- **Alerts**: Price/percent-change alert creation and management
- **Performance**: Win rate, profit factor, PnL chart, win/loss pie chart, drawdown stats

## User preferences

- Dark Bloomberg/TradingView-style UI (navy + cyan accent)
- Bilingual market data: US stocks, Brazilian stocks/futures (B3), crypto, forex, global indices

## Gotchas

- After editing any API route, restart the `artifacts/api-server: API Server` workflow to rebuild
- Dates from Drizzle are JS `Date` objects — always call `.toISOString()` before returning JSON
- `pnpm --filter @workspace/api-spec run codegen` must be re-run whenever `openapi.yaml` changes
- Do not run `pnpm dev` at workspace root — individual workflows handle this
- AI calls use `max_tokens` (not `max_completion_tokens`) with `response_format: { type: "json_object" }`

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
