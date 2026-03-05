# M1-K1 Technical Specification

> V1 Architecture & Technology Selection
>
> Last updated: 2026-03-06

## Table of Contents

- [V1 Scope](#v1-scope)
- [Architecture Overview](#architecture-overview)
- [Monorepo Structure](#monorepo-structure)
- [Frontend](#frontend)
- [Backend](#backend)
- [AI Agent](#ai-agent)
- [Database](#database)
- [Authentication](#authentication)
- [Real-time Communication](#real-time-communication)
- [Developer Toolchain](#developer-toolchain)
- [Deployment](#deployment)
- [Mobile Strategy](#mobile-strategy)
- [Analytics](#analytics)
- [End-to-End Type Flow](#end-to-end-type-flow)

---

## V1 Scope

Core experience: User enters the bar, talks to the AI bartender, browses the menu,
places orders, reads/posts on the message board, with ambient music playing.

| Priority | Module | V1 Scope | Deferred |
|----------|--------|----------|----------|
| P0 | Customer System | Register / Login / Guest / Profile | — |
| P0 | AI Bartender | Conversational agent with personality, memory, and drink recommendations | Multi-agent collaboration |
| P0 | Message Board | Public real-time messages | — |
| P1 | Ordering System | Menu browsing + simulated ordering | Payment integration |
| P1 | Background Music | Playlist playback with ambient atmosphere | User-uploaded tracks |
| P2 | Direct Message | — | V2 |
| P2 | Sales Statistics | — | V2 |
| P3 | Blend System / Easter Eggs | — | V2+ |

---

## Architecture Overview

Full TypeScript monorepo. Single language across frontend, backend, and AI layer.
Progressive evolution path designed for future scaling.

```
V1:   Full TS (Deno backend, Node frontend, shared packages)
V1.5: + Capacitor mobile shell
V2:   + Python AI microservice (if TS AI SDK hits capability ceiling)
V2+:  + Go real-time layer (if concurrent connections exceed Node/Deno limits)
```

Each evolution step is demand-driven, not pre-architected.

---

## Monorepo Structure

**Tooling:** Turborepo + pnpm

```
M1-K1/
├── apps/
│   ├── web/              # Vite 8 + React 19 — Node.js runtime
│   ├── server/           # Hono — Deno runtime
│   └── mobile/           # Capacitor shell (V1.5)
│
├── packages/
│   ├── shared/           # Zod schemas, TypeScript types, constants
│   ├── ai/               # Mastra agent definitions, memory config
│   └── db/               # Drizzle schema, migrations
│
├── turbo.json
├── pnpm-workspace.yaml
└── package.json
```

**Runtime split:**

| Target | Runtime | Rationale |
|--------|---------|-----------|
| `apps/web` | Node.js | Vite/React ecosystem fully Node-native; zero compatibility risk |
| `apps/server` | Deno | Native TS, built-in fmt/lint/test, permission sandbox, `npm:` compat for all deps |
| `packages/*` | Both | Pure TS + Zod; consumed by both runtimes without issue |

Turborepo orchestrates builds across both runtimes. Deno tasks configured via
`deno task` in `apps/server/deno.json`, invoked by Turborepo pipeline.

---

## Frontend

### Core

| Layer | Choice | Key Reason |
|-------|--------|------------|
| Build tool | **Vite 8** (Rolldown) | VoidZero ecosystem alignment; ESM dev server for instant startup; Rolldown Rust bundler for production (10-30x faster than Rollup) |
| UI framework | **React 19** | Largest ecosystem, team familiarity, Mastra/AI SDK streaming integration |
| Styling | **TailwindCSS v4** | CSS-first config (`@theme`), native Vite plugin, P3 color gamut, container queries, `@starting-style` transitions |
| Components | **shadcn/ui** (Radix primitives) | Code ownership (copied, not installed), zero runtime overhead, Tailwind-native, 50+ accessible components |
| Routing | **TanStack Router** | Automatic route param/search param type inference; Zod schema validation on search params; built-in loader with caching |
| Server state | **TanStack Query** | Automatic caching, background refetch, stale-while-revalidate; type-safe integration with Hono RPC via `hono-rpc-query` |
| Client state | **Zustand** | Minimal store-as-hook pattern; no providers; slice-based subscriptions for precise re-renders |
| Forms | **React Hook Form + Zod** | Shared Zod schemas from `packages/shared` for validation; `@hookform/resolvers/zod` |
| Animation | **Motion** (Framer Motion) | Gesture support, layout animations, `AnimatePresence` for enter/exit; combined with TailwindCSS v4 native CSS transitions for simple cases |
| Audio | **Howler.js** | Web Audio API with HTML5 fallback; playlist loop, crossfade, volume control; cross-browser; smooth Capacitor migration path |

### Styling Strategy

TailwindCSS handles structural/standard styles. Custom CSS handles atmospheric effects.

**Tailwind:** Layout, spacing, typography, colors, responsive, dark mode, states, shadcn components.

**Custom CSS:** `@keyframes` animations (ambient glow, pulsing lights), multi-layer `text-shadow` (neon text), complex `backdrop-filter` compositions, pseudo-element decorations, dynamic CSS custom properties driven by JS.

Both coexist in the same CSS entry file via `@import "tailwindcss"` + custom rules.

### API Communication

Hono RPC (`hc` client) provides end-to-end type inference from server route definitions
to frontend API calls. Combined with `hono-rpc-query`, TanStack Query hooks are
auto-generated with full type safety — no codegen, no OpenAPI spec, just TypeScript
inference.

---

## Backend

### Core

| Layer | Choice | Key Reason |
|-------|--------|------------|
| Runtime | **Deno** | Native TS (zero config), built-in tooling (fmt/lint/test), permission sandbox, `npm:` full compat, LTS releases |
| Framework | **Hono** | TypeScript-first, ~13KB, 70K req/s, native WebSocket, Deno first-class support, `hc` RPC client for frontend type sharing |
| Validation | **Zod** (via `@hono/zod-validator`) | Shared schemas across entire stack |

### Why Hono over alternatives

- **vs Fastify:** Hono is multi-runtime (Deno/Node/Edge), TypeScript-first with built-in Zod integration, and provides `hc` RPC client — Fastify is Node-only with JSON Schema validation.
- **vs Express:** Express is architecturally outdated (callback-based, no built-in validation, no type inference).
- **vs ElysiaJS:** Elysia is Bun-first; loses optimizations on other runtimes. Deno is our target runtime, not Bun.

### Mastra Integration

Mastra 1.0+ provides server adapters including Hono. The AI agent layer is embedded
within the Hono server via Mastra's Hono adapter, sharing the same process and database
connection.

---

## AI Agent

### Framework: Mastra

| Capability | Mastra Feature | M1-K1 Application |
|------------|---------------|-------------------|
| Agent | Stateful entity with instructions, model, tools | AI Bartender "Mik7ra" with personality, wit, drink knowledge |
| Working Memory | Persistent scratchpad (Markdown), resource-scoped | Remember each customer's name, preferences, visit history across all conversations |
| Semantic Recall | Vector similarity search on past messages | "What did I order last time?" — retrieve relevant past interactions |
| Observational Memory | Auto-compress long conversations into long-term memory | Gradual personality adaptation; long-term customer relationship |
| Message History | Thread-based conversation tracking | Individual conversation continuity |
| Workflow | Graph-based orchestration (`.then()` / `.branch()` / `.parallel()`) | Order flow: intent classification → recommendation → confirmation → processing |
| RAG | Built-in ingest → chunk → retrieve pipeline | Cocktail knowledge base: recipes, flavor profiles, pairing suggestions |
| Evals | Integrated evaluation framework | Test bartender response quality, personality consistency |

### Memory Architecture

Mastra Memory requires three components working together:

| Component | Role | Implementation |
|-----------|------|----------------|
| **Storage** | Relational data (messages, threads, Working Memory, Observational Memory) | `PostgresStore` (`@mastra/pg`) → self-hosted PostgreSQL |
| **Vector** | Embedding index for Semantic Recall | `PgVector` (`@mastra/pg`) → same PostgreSQL instance via pgvector extension |
| **Embedder** | Text → vector conversion | OpenAI `text-embedding-3-small` (1536 dims) |

Storage and Vector share the same PostgreSQL connection. Mastra auto-creates its own
tables (messages, threads, resources, traces, evals, workflow snapshots) alongside
the application's Drizzle-managed business tables.

Embedding model choice: `text-embedding-3-small` over local `fastembed` because
fastembed's `bge-small-en` model (384 dims) has weak multilingual support and
uncertain Deno runtime compatibility via `onnxruntime-node`.

### Model Strategy

Mastra is model-agnostic via Vercel AI SDK providers. Initial selection:

- **Primary:** OpenAI GPT-4o (tool calling reliability, streaming)
- **Fallback/cost optimization:** Configurable per-request via Mastra's `prepareCall`

### Streaming

AI responses stream to the frontend via SSE (Server-Sent Events), native to
Mastra/Vercel AI SDK. The frontend consumes streams via `useChat` or custom SSE
handling.

---

## Database

### PostgreSQL + pgvector (self-hosted)

| Aspect | Choice | Rationale |
|--------|--------|-----------|
| Database | **PostgreSQL 17** | Relational + vector in one engine; mature, reliable |
| Hosting | **Self-hosted Docker** (`pgvector/pgvector:pg17`) | Full data ownership; co-located with backend for minimal latency; no vendor dependency |
| Vector extension | **pgvector** | Semantic recall for AI memory; no separate vector DB needed |
| ORM | **Drizzle** | Pure TS schema definition; ~57KB bundle; <500ms cold start; SQL-first (critical for pgvector queries); native TS type inference without codegen |

Single PostgreSQL instance serves four consumers via separate tables:

| Consumer | Tables Managed By | Data |
|----------|-------------------|------|
| Business logic | **Drizzle ORM** | Users, orders, menu items, message board |
| AI memory | **Mastra PostgresStore** | Messages, threads, Working Memory, Observational Memory (auto-created) |
| Vector index | **Mastra PgVector** | Embedding vectors for Semantic Recall (auto-created, pgvector extension) |
| Auth | **Better Auth** | Sessions, accounts, verification tokens (auto-created) |

### Why Drizzle over Prisma

- Schema in pure TypeScript (not a DSL) — fits monorepo `packages/db` sharing model
- 35x smaller bundle (57KB vs 2MB+), 2-6x faster cold start
- SQL-first API gives direct control over pgvector similarity queries (`<->` operator)
- No code generation step; types inferred at compile time

---

## Authentication

### Better Auth

| Aspect | Detail |
|--------|--------|
| Approach | Self-hosted, framework-agnostic |
| Storage | User data in own PostgreSQL (via Drizzle adapter) |
| Features | Email/password, OAuth (GitHub, Google), Guest → registered user conversion, 2FA, Passkeys, multi-session |
| Integration | Official Hono integration; session middleware for route protection |
| Type safety | Native `$Infer` for user/session types |
| Cost | Free, open source (26.8K GitHub stars) |

### Why Better Auth over alternatives

- **vs Supabase Auth:** No external service dependency; data in own DB; framework-agnostic (not tied to Supabase ecosystem).
- **vs Clerk:** No per-MAU cost; full data ownership; self-hosted.
- **vs Auth.js:** Framework-agnostic (Auth.js is Next.js-centric); better TypeScript inference.

Guest mode — critical for M1-K1's "walk into the bar" experience — implemented via
Better Auth's anonymous session plugin with conversion to registered account.

---

## Real-time Communication

| Use Case | Transport | Implementation |
|----------|-----------|----------------|
| AI Bartender streaming | **SSE** | Mastra/Vercel AI SDK native output; unidirectional server→client |
| Message Board | **WebSocket** | Hono native WebSocket on Deno; bidirectional |

No Socket.io — Hono's built-in WebSocket support is sufficient for V1 scale and avoids
an additional dependency.

---

## Developer Toolchain

### Lint & Format

| Target | Lint | Format | Rationale |
|--------|------|--------|-----------|
| `apps/web`, `packages/*` | **oxlint** | **oxfmt** | VoidZero ecosystem (shared OXC parser with Vite/Rolldown); 690+ rules; 2.7x faster than Biome lint; oxfmt 3x faster than Biome format; built-in Tailwind class sorting |
| `apps/server` | **`deno lint`** | **`deno fmt`** | Deno built-in; zero config; no external tools needed |

### Why oxlint + oxfmt over Biome

- 690 vs 250 lint rules; full React hooks rule coverage out of the box
- Built-in Tailwind class sorting in oxfmt (Biome lacks this)
- Same OXC parser as Vite 8 (Rolldown) — unified toolchain under VoidZero
- 100% Prettier conformance in oxfmt (Biome is close but not 100%)
- Production-adopted by Shopify, Airbnb, Mercedes-Benz, Vue.js

### Testing

| Target | Tool |
|--------|------|
| `apps/web` | **Vitest** (Vite-native, same config, same transforms) |
| `apps/server` | **`deno test`** (built-in, zero config) |
| `packages/*` | **Vitest** |

### Git Workflow

| Tool | Purpose |
|------|---------|
| **lefthook** | Git hooks (pre-commit lint/format check); faster than husky + lint-staged |
| **Conventional Commits** | `feat:` / `fix:` / `chore:` / `docs:` message format |

---

## Deployment

Self-hosted on a single server, Docker Compose orchestrated.

```
Server
├── Docker Compose
│   ├── postgres     pgvector/pgvector:pg17 (business data + AI memory + vectors)
│   ├── server       Deno + Hono + Mastra (backend API + AI agent)
│   └── umami        Umami v3 (analytics, shares postgres instance)
│
├── Nginx            Reverse proxy + SSL termination + static file serving
│   ├── /            → apps/web build output (SPA)
│   ├── /api/*       → server container
│   └── /media/*     → Docker volume (music, avatars, menu images)
│
└── Certbot          Let's Encrypt auto-renewal
```

| Layer | Service | Rationale |
|-------|---------|-----------|
| Frontend | **Nginx** static serving (optionally fronted by **Cloudflare CDN**) | Self-hosted; Cloudflare free tier adds global edge caching if needed |
| Backend | **Docker** (Deno + Hono) | Co-located with database for minimal latency |
| Database | **Docker** (PostgreSQL 17 + pgvector) | Full data ownership; single instance for all consumers |
| File Storage | **Docker volume + Nginx** static serving | V1 files are operator-curated (music, menu images), not mass UGC; Nginx serves static files at maximum efficiency with zero extra components |
| Analytics | **Docker** (Umami v3) | Shares PostgreSQL instance (separate database); ~200MB RAM |
| SSL | **Certbot** + Let's Encrypt | Auto-renewal; Nginx integration |
| Monitoring | **Uptime Kuma** (optional) | Self-hosted uptime/health monitoring |

### File Storage Evolution

```
V1:   Docker volume + Nginx static serving
      → Music, avatars, menu images written by Hono API, served by Nginx
      → Zero extra components

V1.5: + Garage (self-hosted S3) or Cloudflare R2
      → When user-generated uploads grow or CDN distribution becomes necessary
      → S3 API enables presigned URL direct upload from frontend/mobile
```

### Operations

| Concern | Approach |
|---------|----------|
| Backups | `pg_dump` cron → offsite storage (remote server or cloud S3) |
| Migrations | `drizzle-kit push` / `drizzle-kit migrate` |
| Logs | Docker log driver → file rotation; centralized via Loki if needed |
| Process management | Docker `restart: unless-stopped`; Nginx via systemd |
| Security | Deno permission sandbox + Docker network isolation + Nginx rate limiting + fail2ban |
| Updates | Git pull → Docker build → `docker compose up -d` |

---

## Mobile Strategy

Progressive approach — no upfront native investment.

```
V1.0:  PWA (Service Worker + Web App Manifest)
       → Installable on mobile home screen; zero extra code
       → Limitation: no background audio, iOS push notifications limited

V1.5:  Capacitor shell wrapping the existing SPA
       → Native push notifications (APNs/FCM)
       → Background audio playback via native plugin
       → App Store distribution
       → apps/mobile/ contains only Capacitor config + native project shells
       → UI code = apps/web/ build output (100% reuse)

V2+:   Evaluate Expo/React Native only if native rendering becomes necessary
       → Unlikely for content/interaction-focused app
```

Capacitor adds **zero changes** to the existing web codebase. It wraps the production
build in a native WebView container.

---

## Analytics

### Umami v3

| Aspect | Detail |
|--------|--------|
| Purpose | Visitor analytics (UV/PV, page dwell time, peak hours, referrers, popular menu items) |
| Hosting | Self-hosted Docker, shares the PostgreSQL instance (separate `umami` database) |
| Resources | ~200MB RAM; single container |
| Privacy | Cookie-free; GDPR-compliant without consent banners |
| Tracking | <1KB script; configurable filename to bypass ad blockers |
| Features | Real-time dashboard, custom events, segments/cohorts, multi-site, team management |
| License | MIT |

Distinct from "Sales Statistics" (P2 feature in V1 scope). Sales Statistics aggregates
business data (orders, revenue) from Drizzle tables; Umami tracks user behavior
(visits, navigation patterns, engagement) independently.

Custom events track M1-K1-specific interactions:

- `bartender_conversation_start` — user initiates AI chat
- `drink_order` — order placed (with drink name as property)
- `menu_browse` — menu category viewed
- `music_toggle` — background music play/pause
- `message_post` — message board submission

---

## End-to-End Type Flow

The core value of full-TS monorepo: a single Zod schema propagates type safety from
database to UI with zero manual synchronization.

```
packages/shared/          Zod schema (single source of truth)
    │
    ├─→ packages/db/      Drizzle table definition (infers column types from Zod)
    │       │
    │       └─→ apps/server/   Hono route (validates request via @hono/zod-validator)
    │               │
    │               └─→ Hono RPC type export (typeof route)
    │                       │
    │                       └─→ apps/web/   hc<AppType> client (auto-inferred request/response types)
    │                               │
    │                               ├─→ TanStack Query (typed cache keys + data)
    │                               ├─→ TanStack Router (typed search params via Zod)
    │                               └─→ React Hook Form (typed form validation via Zod)
    │
    └─→ packages/ai/      Mastra tool input schemas (same Zod definitions)
```

Any field change in the Zod schema triggers compile-time errors across every layer
that consumes it. No runtime type drift. No API contract documentation to maintain.

---

## Technology Summary

| Layer | Technology |
|-------|-----------|
| Monorepo | Turborepo + pnpm |
| Frontend Runtime | Node.js |
| Frontend Build | Vite 8 (Rolldown) |
| UI Framework | React 19 |
| Styling | TailwindCSS v4 + custom CSS |
| UI Components | shadcn/ui (Radix) |
| Routing | TanStack Router |
| Server State | TanStack Query |
| Client State | Zustand |
| Forms | React Hook Form + Zod |
| Animation | Motion (Framer Motion) |
| Audio | Howler.js |
| Backend Runtime | Deno |
| Backend Framework | Hono |
| AI Agent Framework | Mastra (on Vercel AI SDK) |
| ORM | Drizzle |
| Database | PostgreSQL 17 + pgvector (self-hosted Docker) |
| Auth | Better Auth |
| Real-time | Hono WebSocket + SSE |
| Validation | Zod (shared) |
| Lint | oxlint (frontend) / deno lint (backend) |
| Format | oxfmt (frontend) / deno fmt (backend) |
| Test | Vitest (frontend) / deno test (backend) |
| Git Hooks | lefthook |
| Deploy | Self-hosted Docker Compose + Nginx |
| File Storage | Docker volume + Nginx (→ Garage / R2 when needed) |
| Analytics | Umami v3 (self-hosted) |
| Mobile (V1.5) | Capacitor |
