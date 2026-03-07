# M1-K1 Interaction Shell

> Layout and UI patterns for the AI Native bar. Use this as the reference when designing in Pencil (wireframes/mockups).
>
> Last updated: 2026-03-06

## Purpose

Single-document spec for the **spatial layout** and **three UI patterns** of the bar. Visual style (2D pixel / 3D / Liquid Glass) is theme-switchable and not defined here.

---

## Layout (Zones)

```
┌─────────────────────────────────────────────────────────┐
│  [ Island: auth status ]     ← top, center               │
├──────────┬──────────────────────────────────┬──────────┤
│          │                                    │          │
│  Board   │   Bar scene                        │  (cards  │
│  (left   │   · Bartender figure (mid–upper)   │   can    │
│   wall)  │   · Conversation bubbles           │   dock   │
│          │   · Optional foreground (bar top)  │   here)  │
│          │                                    │          │
├──────────┴──────────────────────────────────┴──────────┤
│  [ Input bar: text + voice, suggest chips above ]       │  ← bottom
└─────────────────────────────────────────────────────────┘
```

- **Top:** Island (auth status). Expands downward as a drawer when opened.
- **Center:** Main view — bar scene, bartender, conversation. Primary focus.
- **Left:** Message board — one tilted “wall” panel (scale: peripheral ↔ focus).
- **Edges (all four):** Dockable cards (menu, tab) can be docked to any edge by user; collapsed = small tab/card peeking from border.
- **Bottom:** Input bar (text field, send, mic). Suggest-reply chips sit just above this bar when present.

---

## Three UI Patterns

### 1. Dockable cards (Menu, Tab)

- **Collapsed:** Small card/tab tucked into a border (user chooses which edge per card). Click or drag out to open.
- **Open:** Floating panel; can be dragged, then dragged back to an edge to collapse.
- **Trigger:** Conversation (agent sends `open_menu` / `open_tab`) or user click/drag.
- **V1 set:** Menu card, Tab (bill) card. Each can be docked to a different edge.

### 2. Message board (left wall)

- **Single panel** on the left, tilted (perspective like Stage Manager), not multiple blocks.
- **Two states:** Peripheral (small, “corner of eye”) ↔ Focus (larger, “looking at the wall”).
- **Trigger:** Conversation (`focus_board` / `unfocus_board`) or user click on the board.
- **Position:** Fixed left; no docking to other edges.

### 3. Auth (island + drawer)

- **Default:** Top-center “island” showing status (e.g. “路人 · 未登记” or “昵称 · 今晚第 N 杯”). Compact, always visible.
- **Expanded:** Clicking the island opens a **drawer from the top** (downward). Content: login / register / profile (single view per `intent`).
- **Trigger:** Conversation (`open_auth_drawer` with intent) or user click on island.
- **Not** a dockable card; only island + drawer.

---

## Checklist for Pencil

Use this when drawing wireframes:

- [ ] **Island** at top center; drawer opens downward.
- [ ] **Bar scene** in the middle: bartender figure (mid–upper), space for conversation bubbles.
- [ ] **Input bar** at bottom (text + send + mic); optional row of suggest chips above it.
- [ ] **Left:** One message board panel, tilted; show both states (small vs focused) if needed.
- [ ] **Edges:** Placeholder for 1–2 dockable cards (menu, tab) — collapsed (tab peeking) and expanded (panel) states.
- [ ] No top tab bar or sidebar nav; only this shell + agent-driven UI.

---

## References

- Agent actions that drive this shell: [`docs/actions-v1.md`](./actions-v1.md)
- Tech stack (frontend, AG-UI): [`docs/tech-stack.md`](../tech-stack.md)
