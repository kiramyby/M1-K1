# M1-K1 Development Roadmap

> V1 Build Plan
>
> Last updated: 2026-03-06

## Overview

Six phases, dependency-ordered. Each phase produces a runnable increment.

```
Phase 0  Scaffold        ██░░░░░░░░░░░░░░░░░░  Monorepo + infra + dev tooling
Phase 1  Foundation      ████░░░░░░░░░░░░░░░░  DB schema + auth + UI shell
Phase 2  Core — AI       ██████░░░░░░░░░░░░░░  AI Bartender (agent + memory + RAG)
Phase 3  Core — Social   ████████░░░░░░░░░░░░  Message Board + Ordering System
Phase 4  Atmosphere      ██████████░░░░░░░░░░  Background Music + visual polish
Phase 5  Ship            ████████████░░░░░░░░  Production deploy + analytics + PWA
```

Phases 2-4 have parallel work streams (frontend + backend). Phase 3 can partially
overlap with Phase 2 once the backend foundation is stable.

---

## Phase 0 — Scaffold

> Goal: Empty monorepo that builds, lints, formats, and runs dev servers.

### 0.1 Monorepo Init

- [ ] Init pnpm workspace + Turborepo
- [ ] `pnpm-workspace.yaml` defining `apps/*` and `packages/*`
- [ ] `turbo.json` with `build`, `dev`, `lint`, `format`, `test` pipelines
- [ ] Root `package.json` with workspace scripts

### 0.2 Apps

- [ ] `apps/web`: Vite 8 + React 19 + TailwindCSS v4 scaffold
  - TanStack Router (file-based routes, empty pages)
  - TanStack Query provider
  - Zustand root store (empty)
  - TailwindCSS v4 with `@theme` bar color tokens
  - shadcn/ui init (Button, Input, Dialog, Toast as starter set)
- [ ] `apps/server`: Deno + Hono scaffold
  - `deno.json` with tasks (`dev`, `test`, `lint`, `fmt`)
  - Hono app with health check route (`GET /api/health`)
  - CORS middleware for local dev
  - Environment variable loading

### 0.3 Packages

- [ ] `packages/shared`: Zod schemas (empty barrel export), shared types, constants
- [ ] `packages/db`: Drizzle config + empty schema + migration setup
- [ ] `packages/ai`: Mastra config placeholder

### 0.4 Dev Tooling

- [ ] oxlint config (`.oxlintrc.json`) for `apps/web` + `packages/*`
- [ ] oxfmt config with Tailwind class sorting enabled
- [ ] lefthook config (pre-commit: lint + format check)
- [ ] `.gitignore`, `.env.example`

### 0.5 Local Infrastructure

- [ ] `docker-compose.yml`: PostgreSQL 17 + pgvector
- [ ] Verify Deno connects to local PG
- [ ] Verify Drizzle can push schema to local PG

**Exit criteria:** `pnpm dev` starts both web (Vite) and server (Deno + Hono).
Frontend fetches `/api/health` and displays response. Lint/format passes CI.

---

## Phase 1 — Foundation

> Goal: Users can register, log in, browse as guest, and see a basic UI shell.

### 1.1 Database Schema (packages/db)

- [ ] `users` table (extended by Better Auth)
- [ ] `sessions` table (Better Auth)
- [ ] `menu_items` table (id, name, category, description, price, image_path, available)
- [ ] `orders` table (id, user_id, status, created_at)
- [ ] `order_items` table (id, order_id, menu_item_id, quantity)
- [ ] `messages` table (id, user_id, content, created_at) — for Message Board
- [ ] Seed script: populate menu with initial cocktail list
- [ ] Run initial migration via `drizzle-kit push`

### 1.2 Authentication (apps/server)

- [ ] Better Auth setup with Hono integration
- [ ] Email/password registration + login
- [ ] Guest (anonymous) session creation
- [ ] Guest → registered account conversion flow
- [ ] OAuth provider (GitHub — one provider is enough for V1)
- [ ] Session middleware on Hono routes
- [ ] Hono RPC type export for auth routes

### 1.3 UI Shell (apps/web)

- [ ] Layout: dark-themed bar atmosphere shell
  - Top bar (logo, user avatar/login)
  - Main content area
  - Bottom navigation or sidebar (Bartender / Menu / Board / Music)
- [ ] Auth pages: Login, Register (shadcn form components + React Hook Form + Zod)
- [ ] Guest entry: "Walk in" button → anonymous session
- [ ] Profile page: basic info display, avatar placeholder
- [ ] Connect Hono `hc` client, configure TanStack Query
- [ ] Auth state management (Zustand store for session)
- [ ] Protected route wrapper (redirect to login or guest prompt)

### 1.4 Type Flow Validation

- [ ] Verify end-to-end: Zod schema → Drizzle table → Hono route → hc client → TanStack Query → UI render
- [ ] Modify one Zod field, confirm compile errors propagate to all consumers

**Exit criteria:** User can register, log in, enter as guest, see the shell with
navigation. Auth state persists across page reloads.

---

## Phase 2 — Core: AI Bartender

> Goal: User has a streaming conversation with the AI bartender who remembers them.

### 2.1 Mastra Setup (packages/ai)

- [ ] Mastra init with Hono server adapter
- [ ] PostgresStore + PgVector pointing to local PG
- [ ] OpenAI `text-embedding-3-small` as embedder
- [ ] Verify Mastra auto-creates its tables in PG

### 2.2 Bartender Agent Definition

- [ ] System prompt: personality (Mik7ra), tone, knowledge boundaries
- [ ] Working Memory config: resource-scoped (per-user across threads)
  - Template: name, drink preferences, visit count, notes
- [ ] Semantic Recall config: topK=3, messageRange=2
- [ ] Observational Memory: enabled, default compression threshold
- [ ] Message History: lastMessages=20

### 2.3 Agent Tools

- [ ] `lookupMenu` — query menu_items table, return matching drinks
- [ ] `getCustomerPreferences` — read Working Memory for current user
- [ ] `placeOrder` — insert into orders + order_items tables
- [ ] `getOrderHistory` — query past orders for current user

### 2.4 RAG: Cocktail Knowledge Base

- [ ] Prepare cocktail knowledge documents (recipes, flavor profiles, pairing tips)
- [ ] Ingest via Mastra RAG pipeline → PgVector
- [ ] Agent can answer "What's in a Negroni?" or "Something citrusy?" from knowledge base

### 2.5 API Routes (apps/server)

- [ ] `POST /api/chat` — send message, stream response (SSE)
- [ ] `GET /api/chat/threads` — list user's conversation threads
- [ ] `GET /api/chat/threads/:id` — get thread messages
- [ ] `POST /api/chat/threads` — create new thread
- [ ] Wire Mastra agent to routes with user context from Better Auth session

### 2.6 Chat UI (apps/web)

- [ ] Chat interface: message list + input box
- [ ] Streaming response rendering (token-by-token display)
- [ ] Typing indicator while agent is responding
- [ ] Thread management: new conversation, switch threads
- [ ] Display agent tool actions (e.g. "Looking up the menu..." indicator)
- [ ] Mobile-responsive chat layout

### 2.7 Memory Validation

- [ ] Conversation 1: tell bartender your name and a drink preference
- [ ] Conversation 2 (new thread): bartender recalls name and preference via Working Memory
- [ ] Long conversation: verify Observational Memory compresses older messages
- [ ] Semantic Recall: "What did I order last time?" retrieves correct context

**Exit criteria:** Streaming AI conversation works. Bartender has personality,
remembers the user across conversations, can look up and recommend drinks from
the menu, and can place orders.

---

## Phase 3 — Core: Social + Ordering

> Goal: Message Board is live. Orders are tracked.

Can begin once Phase 1 backend is stable; runs in parallel with Phase 2 frontend work.

### 3.1 Message Board — Backend

- [ ] `GET /api/messages` — paginated message list
- [ ] `POST /api/messages` — post a message (auth required, guest OK)
- [ ] WebSocket endpoint `/api/ws/messages` — real-time broadcast
  - Connection auth (session token)
  - Join/leave tracking
  - Broadcast new messages to all connected clients

### 3.2 Message Board — Frontend

- [ ] Message list component (virtual scroll for performance)
- [ ] Message input with character limit
- [ ] Real-time updates via WebSocket (new messages appear instantly)
- [ ] User avatar + name display per message
- [ ] Connection status indicator (connected / reconnecting)
- [ ] Timestamp display (relative: "2 minutes ago")

### 3.3 Ordering System — Backend

- [ ] `GET /api/menu` — full menu with categories
- [ ] `GET /api/menu/:id` — single item detail
- [ ] `POST /api/orders` — place order (also callable by AI Bartender tool)
- [ ] `GET /api/orders` — user's order history
- [ ] `PATCH /api/orders/:id` — update order status (admin)
- [ ] Order status enum: pending → preparing → ready → completed

### 3.4 Ordering System — Frontend

- [ ] Menu browsing page: categories, item cards with images
- [ ] Item detail modal/sheet (description, price, order button)
- [ ] Order placement flow (confirm → success toast)
- [ ] Order history page
- [ ] Order status display (real-time via polling or WebSocket)

### 3.5 Integration: AI + Ordering

- [ ] Bartender can reference items currently on the menu
- [ ] Bartender's `placeOrder` tool creates real orders visible in order history
- [ ] Bartender can check order status via tool

**Exit criteria:** Users can browse menu, place orders (directly and through
bartender), see order history. Message Board shows real-time public messages.

---

## Phase 4 — Atmosphere

> Goal: The bar feels alive. Visual and audio polish.

### 4.1 Background Music — Backend

- [ ] `GET /api/music/playlist` — return playlist metadata
- [ ] Music files served via Nginx static `/media/music/`
- [ ] Admin: upload/manage tracks (or seed from prepared files)

### 4.2 Background Music — Frontend

- [ ] Howler.js audio player integration
- [ ] Playlist with shuffle / sequential playback
- [ ] Play/pause toggle, volume slider (Zustand state)
- [ ] Track info display (now playing)
- [ ] Crossfade between tracks
- [ ] Persist playback state across page navigation

### 4.3 Visual Atmosphere

- [ ] Custom CSS: ambient glow effects, neon text, dynamic gradients
- [ ] Dark theme refinement: bar-appropriate color palette
- [ ] Motion animations: page transitions, card interactions, chat bubbles
- [ ] Glassmorphism / frosted glass elements for panels
- [ ] Loading states and skeleton screens
- [ ] Responsive design pass: mobile, tablet, desktop

### 4.4 UX Polish

- [ ] Toast notifications (order placed, new message, etc.)
- [ ] Error states and fallback UI
- [ ] Empty states (no orders yet, no messages, first visit)
- [ ] Keyboard shortcuts (focus chat input, toggle music)
- [ ] Favicon and meta tags

**Exit criteria:** The bar has a distinct atmosphere. Music plays. Animations are
smooth. UI feels cohesive and polished across screen sizes.

---

## Phase 5 — Ship

> Goal: Production-ready deployment. Monitoring. PWA.

### 5.1 Production Infrastructure

- [ ] Production `docker-compose.yml` (environment-specific configs)
- [ ] Nginx config: reverse proxy, SSL termination, static caching headers, gzip/brotli
- [ ] Certbot SSL setup + auto-renewal cron
- [ ] PostgreSQL: production credentials, connection pool tuning
- [ ] Environment variable management (`.env.production`)
- [ ] `pg_dump` backup cron → offsite storage
- [ ] Firewall rules (only 80/443 exposed)

### 5.2 Analytics

- [ ] Umami v3 Docker container (shares PG instance, separate `umami` database)
- [ ] Tracking script in `apps/web` index.html
- [ ] Custom events: `bartender_conversation_start`, `drink_order`, `menu_browse`,
  `music_toggle`, `message_post`
- [ ] Verify dashboard shows real-time data

### 5.3 PWA

- [ ] `vite-plugin-pwa` integration
- [ ] Web App Manifest (name, icons, theme color, display: standalone)
- [ ] Service Worker: cache shell + static assets for offline
- [ ] Install prompt on mobile

### 5.4 Performance & Security

- [ ] Lighthouse audit: target 90+ on Performance, Accessibility, Best Practices
- [ ] Nginx rate limiting on API routes
- [ ] fail2ban for SSH and Nginx
- [ ] Deno permission flags locked down (`--allow-net`, `--allow-read`, `--allow-env`)
- [ ] Content Security Policy headers
- [ ] CORS production whitelist

### 5.5 Testing

- [ ] Frontend: Vitest unit tests for critical utilities and hooks
- [ ] Backend: `deno test` for API route handlers and auth flows
- [ ] AI: Mastra Evals for bartender response quality
- [ ] End-to-end: manual test checklist (register, chat, order, message, music)

**Exit criteria:** App is live on production server. SSL works. Backups run.
Analytics collecting data. PWA installable on mobile.

---

## Dependency Graph

```
Phase 0 (Scaffold)
    │
    ▼
Phase 1 (Foundation: DB + Auth + Shell)
    │
    ├────────────────┐
    ▼                ▼
Phase 2 (AI)     Phase 3 (Social + Orders)  ← can run in parallel
    │                │
    └───────┬────────┘
            ▼
      Phase 4 (Atmosphere)
            │
            ▼
      Phase 5 (Ship)
```

Phase 2 and 3 can overlap: the backend foundation from Phase 1 supports both.
Frontend work in Phase 2 (chat UI) and Phase 3 (message board, menu) are independent.

---

## Post-V1

Features first, infrastructure optimizations only when bottlenecks appear.

| Priority | Addition | Trigger |
|----------|----------|---------|
| **V1.5** | **Capacitor mobile shell** | V1 validated, mobile users want native app |
| **V1.5** | **Direct Message** | User demand for private conversations |
| **V2** | **Blend System** | Core menu stable, ready for custom cocktail mechanics |
| **V2** | **Sales Statistics** | Enough order data to justify a dashboard |
| **V2** | **Easter Eggs** | Core experience polished, room for creative touches |
| V2+ | Garage / Cloudflare R2 | User-generated uploads grow beyond filesystem |
| V2+ | Python AI microservice | TS AI SDK hits capability ceiling |
| V2+ | Go real-time layer | Concurrent connections exceed Deno limits |
