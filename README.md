# M1-K1: A Unique Bar on the Web

> In a net bar, Tender is the Night.
>
> **Project Stage:** V1 - Planning Phase

## Introduction

`M1-K1` is an online virtual bar which is running by the boss, *Mik7ra*, who is
also called *Kiracoon*.

## Service

> Services the final version of `M1-K1` will provide

- [ ] Message Board - public messages

- [ ] Direct Message - private messages, contacting a specific customer; direct, controlled and private

- [ ] Background Music - provide a dreaming atmosphere

- [ ] Ordering System - classic and signature cocktails

- [ ] Customer System - signing up & logging in, profile, supporting guest yet

- [ ] Sales Statistics - for sales checking

- [ ] Easter Eggs

- [ ] Blend System - special cocktails

- [ ] AI Bartender - an AI agent

## Tech Stack

> Full specification: [`docs/tech-stack.md`](docs/tech-stack.md)

**Architecture:** Full TypeScript monorepo (Turborepo + pnpm), progressive evolution path.

| Layer | Technology |
|-------|-----------|
| Frontend | Vite 8 (Rolldown) · React 19 · TailwindCSS v4 · shadcn/ui · TanStack Router & Query · Zustand |
| Backend | Deno · Hono · Zod · Better Auth |
| AI | Mastra (Vercel AI SDK) · 4-tier memory · RAG · Workflow |
| Data | PostgreSQL 17 + pgvector · Drizzle ORM |
| Tooling | oxlint + oxfmt · deno lint/fmt · Vitest · lefthook |
| Deploy | Self-hosted Docker Compose · Nginx |
| Analytics | Umami v3 (self-hosted, cookie-free) |
| Mobile | PWA (V1) → Capacitor (V1.5) |

