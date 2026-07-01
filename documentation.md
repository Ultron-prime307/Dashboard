# Ledger — Task Management Dashboard
### Complete Product & Technical Documentation

---

## 1. Overview

Ledger is a Kanban-style task management dashboard. Tasks move across four lanes — **Backlog → In Progress → Review → Done** — with drag-and-drop, priority tagging, due dates, assignees, search, and filtering. A working prototype is included (`task-dashboard.html`); this document also specifies the technology stack to take it to a production, multi-user product.

**Core features (in the prototype)**
- Create, edit, delete tasks
- Drag-and-drop between status lanes
- Priority levels (High / Medium / Low) with visual stamps
- Due dates with automatic overdue detection
- Assignee and tag fields
- Search by title/description
- Filter by priority and tag
- Live stats panel (total, completed, overdue, high priority)
- Data persists automatically between sessions

---

## 2. Prototype Tech Stack (what's included)

| Layer | Technology | Why |
|---|---|---|
| Structure | HTML5 | Single-file, zero build step |
| Styling | Hand-written CSS (custom properties) | Full control, no framework overhead for a self-contained demo |
| Behavior | Vanilla JavaScript (ES6+) | No dependencies, loads instantly |
| Fonts | Fraunces (display), Inter (body), IBM Plex Mono (data/labels) — via Google Fonts | Distinct visual identity |
| Persistence | Key-value storage API (`window.storage`) | Tasks survive reloads without a backend |

This is intentionally dependency-free so it runs anywhere a browser does. It's a strong prototype and internal tool, but for a real multi-user product you'll want the stack below.

---

## 3. Recommended Production Tech Stack

### Frontend
| Choice | Notes |
|---|---|
| **React 18+ (with TypeScript)** | Component model fits a card/lane board well; TypeScript catches data-shape bugs (task fields, statuses) early |
| **Vite** | Fast dev server and build tooling |
| **Tailwind CSS** | Utility-first styling, easy to keep the design system consistent |
| **@dnd-kit/core** | Accessible, well-maintained drag-and-drop for React (better than raw HTML5 DnD for keyboard/mobile support) |
| **TanStack Query (React Query)** | Server-state caching, background refetching, optimistic updates when moving cards |
| **Zustand** or React Context | Lightweight local/UI state (open modals, filters) |
| **React Hook Form + Zod** | Form handling and schema validation for the task modal |

### Backend
| Choice | Notes |
|---|---|
| **Node.js + Express** or **Fastify** | REST (or GraphQL via Apollo/Yoga) API layer |
| **PostgreSQL** | Relational data (tasks, users, projects, comments) with strong query support |
| **Prisma ORM** | Type-safe DB access, migrations |
| **Redis** | Caching + pub/sub for real-time board updates |
| **WebSockets (Socket.IO) or Server-Sent Events** | Live updates when teammates move/edit cards |

### Auth & Access Control
- **Auth.js (NextAuth)** or **Clerk/Auth0** for login (email, SSO, OAuth)
- Role-based access control: Owner / Member / Viewer per project/board

### Infrastructure
| Concern | Choice |
|---|---|
| Hosting (frontend) | Vercel or Netlify |
| Hosting (backend/API) | Render, Railway, or AWS (ECS/Fargate) |
| Database hosting | Supabase, Neon, or AWS RDS (Postgres) |
| File uploads (attachments) | AWS S3 / Cloudflare R2 |
| CI/CD | GitHub Actions |
| Monitoring | Sentry (errors), PostHog or Plausible (analytics) |

### Suggested full production stack summary
> **React + TypeScript + Tailwind (frontend)** → **Node/Express + PostgreSQL + Prisma (backend)** → **WebSockets for real-time sync** → **Vercel + Railway/Supabase (hosting)**

This is the same stack pattern used by tools like Linear, Trello, and Height, scaled down appropriately.

---

## 4. Data Model

```
Task {
  id: string (uuid)
  title: string
  description: string
  status: 'backlog' | 'progress' | 'review' | 'done'
  priority: 'low' | 'medium' | 'high'
  dueDate: date | null
  assigneeId: string | null   // FK -> User
  tag: string
  boardId: string             // FK -> Board
  createdAt: timestamp
  updatedAt: timestamp
}

User {
  id: string (uuid)
  name: string
  email: string
  avatarUrl: string
}

Board {
  id: string (uuid)
  name: string
  ownerId: string   // FK -> User
}
```

For production, add a `Comment` table (task discussion) and an `Activity` table (audit log of status/assignee changes) — both are common asks once teams start using a board.

---

## 5. Architecture Diagram (production)

```
┌─────────────┐      HTTPS/REST       ┌──────────────┐
│   React SPA │ ───────────────────▶ │  Express API  │
│  (Vercel)   │ ◀─────────────────── │   (Railway)   │
└─────────────┘      WebSocket        └──────┬───────┘
                                              │
                                     ┌────────▼────────┐
                                     │   PostgreSQL     │
                                     │   (Supabase)     │
                                     └──────────────────┘
```

---

## 6. Running the Prototype

The included `task-dashboard.html` is self-contained:

1. Open the file directly in any modern browser, **or**
2. Serve it locally for a URL-based workflow:
   ```bash
   npx serve .
   ```
3. Data is saved automatically as you work — no setup required.

---

## 7. Roadmap / Next Steps

1. **Multi-user boards** — invite teammates, assign roles
2. **Real-time sync** — WebSocket updates when others move cards
3. **Comments & activity log** per task
4. **Subtasks / checklists**
5. **Notifications** — due-date reminders via email
6. **Mobile app** — React Native reusing the API layer
7. **Analytics view** — burndown chart, cycle time per lane

---

*Prepared as accompanying documentation for the Ledger task dashboard prototype.*
