# M1-K1 V1 Agent Actions

> Agent → Frontend UI actions for the AI Native bar experience.
>
> All UI changes are driven by conversation; the bartender emits these actions so the app updates panels, board focus, and ephemeral UI.
>
> Last updated: 2026-03-06

## Overview

- **Transport:** Emitted over AG-UI (e.g. as tool calls or custom events). Schema is transport-agnostic.
- **Scope:** V1 only. Covers menu, tab (bill), auth drawer, message board focus, suggest replies, music, toast.
- **Direction:** Agent → Frontend only. User intents are always expressed as messages (text or suggested reply click).

---

## Action List

| Action | Purpose | Payload |
|--------|---------|---------|
| `open_menu` | Open the menu dockable card (from edge). Optionally show a subset. | `MenuPayload` |
| `close_menu` | Collapse menu back to edge. | — |
| `open_tab` | Open the bill/tab dockable card with current order. | `TabPayload` |
| `close_tab` | Collapse tab back to edge. | — |
| `open_auth_drawer` | Open top island drawer (login / register / profile). | `AuthDrawerPayload` |
| `close_auth_drawer` | Close the auth drawer. | — |
| `focus_board` | Scale message board from peripheral to focus (user “looking at the wall”). | — |
| `unfocus_board` | Scale message board back to peripheral. | — |
| `suggest_replies` | Show reply chips below the last agent message. | `SuggestRepliesPayload` |
| `set_music` | Control background music (play/pause/mood). | `MusicPayload` |
| `show_toast` | Ephemeral notification (e.g. “已记在账上”). | `ToastPayload` |

---

## Payload Schemas

### Menu

- **open_menu**
  - `items?: MenuItem[]` — If present, show only these items (e.g. 2–3 recommendations). If absent, frontend fetches/shows full menu.
  - `title?: string` — Optional heading (e.g. “今晚推荐”).

```ts
interface MenuItem {
  id: string;
  name: string;
  category?: string;
  description?: string;
  price?: number;
  imagePath?: string;
}
```

### Tab (Bill)

- **open_tab** / **update_tab**
  - `orders: TabOrderItem[]`
  - `total?: number`
  - `status?: 'pending' | 'preparing' | 'ready' | 'completed'`

```ts
interface TabOrderItem {
  orderId: string;
  items: { menuItemId: string; name: string; quantity: number; price?: number }[];
  status?: string;
}
```

### Auth drawer

- **open_auth_drawer**
  - `intent: 'login' | 'register' | 'profile'` — Which view to show in the drawer.

### Suggest replies

- **suggest_replies**
  - `replies: string[]` — 1–5 short labels; clicking one sends that text as the user message.

### Music

- **set_music**
  - `action: 'play' | 'pause' | 'next' | 'volume'`
  - `value?: number` — For `volume`, 0–1.
  - `mood?: string` — Optional hint for track selection (e.g. “chill”, “jazz”).

### Toast

- **show_toast**
  - `message: string`
  - `duration?: number` — ms; default e.g. 3000.

---

## Zod Schema (single source of truth)

Below is the canonical schema for backend (Mastra) and frontend. Place in `packages/shared` when the monorepo exists.

```ts
import { z } from "zod";

// --- Payloads ---
export const menuItemSchema = z.object({
  id: z.string(),
  name: z.string(),
  category: z.string().optional(),
  description: z.string().optional(),
  price: z.number().optional(),
  imagePath: z.string().optional(),
});

export const menuPayloadSchema = z.object({
  items: z.array(menuItemSchema).optional(),
  title: z.string().optional(),
});

export const tabOrderItemSchema = z.object({
  orderId: z.string(),
  items: z.array(
    z.object({
      menuItemId: z.string(),
      name: z.string(),
      quantity: z.number(),
      price: z.number().optional(),
    })
  ),
  status: z.string().optional(),
});

export const tabPayloadSchema = z.object({
  orders: z.array(tabOrderItemSchema),
  total: z.number().optional(),
  status: z.enum(["pending", "preparing", "ready", "completed"]).optional(),
});

export const authDrawerPayloadSchema = z.object({
  intent: z.enum(["login", "register", "profile"]),
});

export const suggestRepliesPayloadSchema = z.object({
  replies: z.array(z.string()).min(1).max(5),
});

export const musicPayloadSchema = z.object({
  action: z.enum(["play", "pause", "next", "volume"]),
  value: z.number().min(0).max(1).optional(),
  mood: z.string().optional(),
});

export const toastPayloadSchema = z.object({
  message: z.string(),
  duration: z.number().positive().optional(),
});

// --- Action envelope (AG-UI tool call or custom event) ---
export const actionSchema = z.discriminatedUnion("name", [
  z.object({ name: z.literal("open_menu"), payload: menuPayloadSchema.optional() }),
  z.object({ name: z.literal("close_menu") }),
  z.object({ name: z.literal("open_tab"), payload: tabPayloadSchema }),
  z.object({ name: z.literal("close_tab") }),
  z.object({ name: z.literal("open_auth_drawer"), payload: authDrawerPayloadSchema }),
  z.object({ name: z.literal("close_auth_drawer") }),
  z.object({ name: z.literal("focus_board") }),
  z.object({ name: z.literal("unfocus_board") }),
  z.object({ name: z.literal("suggest_replies"), payload: suggestRepliesPayloadSchema }),
  z.object({ name: z.literal("set_music"), payload: musicPayloadSchema }),
  z.object({ name: z.literal("show_toast"), payload: toastPayloadSchema }),
]);

export type AgentAction = z.infer<typeof actionSchema>;
export type MenuPayload = z.infer<typeof menuPayloadSchema>;
export type TabPayload = z.infer<typeof tabPayloadSchema>;
export type AuthDrawerPayload = z.infer<typeof authDrawerPayloadSchema>;
export type SuggestRepliesPayload = z.infer<typeof suggestRepliesPayloadSchema>;
export type MusicPayload = z.infer<typeof musicPayloadSchema>;
export type ToastPayload = z.infer<typeof toastPayloadSchema>;
```

---

## When to use (agent guidelines)

| Action | Example trigger |
|--------|------------------|
| `open_menu` | User asks for recommendations or “看看酒单”; after recommending 2–3 drinks, open menu with `items` set to those. |
| `close_menu` | User says “先收起来” or conversation moves away from ordering. |
| `open_tab` | User places an order (via conversation); show tab with updated orders. User says “看看今晚的账”. |
| `close_tab` | User says “收起来” re bill. |
| `open_auth_drawer` | User says “登记一下” / “登录” / “看看我的资料” → intent register / login / profile. |
| `close_auth_drawer` | After successful login/register or user says “先不登记了”. |
| `focus_board` | User says “看看墙上写了什么” or “留言板”. |
| `unfocus_board` | User says “先不看了” or returns to main conversation. |
| `suggest_replies` | After welcome, or after asking a yes/no choice; 1–5 short options. |
| `set_music` | User “换首安静点的” → mood; “暂停一下” → pause. |
| `show_toast` | After placing order (“已记在账上”), or short confirmation. |

---

## Implementation notes

- **AG-UI:** Emit each action as a structured tool call (e.g. `open_menu`) or as a custom AG-UI event with `actionSchema`-valid payload. Frontend subscribes and updates shell state (which panel is open, board scale, toast queue).
- **Batching:** Multiple actions in one turn are allowed (e.g. `open_tab` + `show_toast` after order). Order of execution: menu/tab/drawer/board first, then suggest_replies, then toast.
- **Idempotency:** `open_*` with same payload can be treated as no-op or refresh; `close_*` is always safe to repeat.
