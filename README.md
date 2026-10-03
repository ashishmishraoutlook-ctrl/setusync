# SetuSync MVP

SetuSync is a persistent, search-aware offline memory layer for SerpApi-powered applications.

## What this MVP proves

1. While online, a search is sent through the backend to SerpApi.
2. The normalized result is persisted in the phone/browser using IndexedDB.
3. The same search can be served locally while the device has no network connection.
4. A search that has never been acquired is explicitly reported as unavailable offline.
5. Cached results carry retrieval/freshness metadata so the UI does not pretend stale data is live.

The MVP intentionally does **not** include a hub, P2P, CRDT synchronization, AWS, distributed workers, or AI. Those are later extensions after the local-first core is proven.

## Architecture

```text
                 MOBILE / BROWSER
┌───────────────────────────────────────────────────────────────┐
│ React + TypeScript PWA                                       │
│                                                               │
│ Search UI → SetuSync Core → IndexedDB + local search         │
│                    │                                          │
│                    └──── online only ────┐                   │
└───────────────────────────────────────────┼───────────────────┘
                                            │
                                      HTTPS / JSON
                                            │
                              ┌─────────────▼────────────┐
                              │ Spring Boot 4.1 backend  │
                              │ /api/search              │
                              └─────────────┬────────────┘
                                            │
                                    SerpApi Search API
```

## Requirements

- Node.js 22+
- Java 21+
- Maven 3.6.3+ (or use your IDE's Maven support)
- A SerpApi API key for live searches

Spring Boot 4.1.1 is the current stable release used by this project and requires at least Java 17; Java 21 is used here. See the official Spring Boot system requirements.

## Run backend

```bash
cd backend
export SERPAPI_API_KEY="your-key"
mvn spring-boot:run
```

The backend listens on `http://localhost:8080`.

For a no-key UI demo, you can use the mock provider:

```bash
export SETUSYNC_PROVIDER=mock
mvn spring-boot:run
```

## Run frontend

```bash
cd frontend
npm install
npm run dev
```

Open the URL printed by Vite.

For a phone demo on the same Wi-Fi network, run Vite with `--host` and point the browser to your computer's LAN IP. The backend CORS configuration is intentionally permissive for development only.

## Demo sequence

1. Start backend and frontend.
2. Make sure the phone/browser is online.
3. Search for a query such as `PM Kisan eligibility`.
4. Confirm the result shows `LIVE → SAVED LOCALLY`.
5. Turn on airplane mode or disable network.
6. Search the same query again. It should return `LOCAL CACHE`.
7. Search a never-seen query. It should show `NOT AVAILABLE OFFLINE`.
8. Restore the network and search the uncached query. It is fetched and saved.
9. Disconnect again and repeat the query to prove persistence.

## Core design rules

- A cache key includes every parameter that can change the meaning of a search, not only `q`.
- Offline reads come from the local database/index.
- Offline misses never fabricate an answer.
- Stale results are visibly labelled with their retrieval time.
- The SerpApi private API key lives only on the backend.

## Next milestones

1. Add deterministic benchmarks for local-search latency and cache hit rate.
2. Add an offline-preparation queue for user-selected topics.
3. Add richer local ranking and related-query matching.
4. Add observability and deployment.
5. Only then consider optional SDK/cloud/relay extensions.
