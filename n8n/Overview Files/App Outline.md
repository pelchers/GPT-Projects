Below is a **copy-ready “App Overview & PRD”** that assumes the project is, first and foremost, an **AI-centric chat workspace**—with deep document storage, navigation, and collaboration layers wrapped around that chat experience.

> **Paste it into** `docs/app-overview-prd.md` (or similar).
> The file-tree block is positioned up top so devs can instantly map concepts to folders.

---

```text
my-app/
├── client/                # Vite + React (AI-chat-first UI)
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│       ├── components/
│       │   ├── layout/
│       │   ├── chat/
│       │   │   ├── ChatWindow.tsx        # Thread pane w/ file previews
│       │   │   ├── MessageInput.tsx      # Slash-commands, prompt helpers
│       │   │   ├── ActionButtons.tsx     # “/new-doc”, “/summarise” …
│       │   │   └── index.ts
│       ├── hooks/
│       │   ├── useChatApi.ts             # Talks to Express → n8n
│       │   ├── usePageContent.ts
│       │   ├── useAuth.ts
│       ├── lib/
│       │   ├── queryClient.ts            # React-Query
│       │   ├── utils.ts
│       │   ├── api.ts                    # REST helpers
│       │   ├── auth.ts
│       ├── pages/
│       │   ├── chat.tsx                  # Primary screen
│       │   ├── feature-support.tsx       # Docs on slash-commands / tips
│       │   ├── login.tsx
│       │   ├── register.tsx
│       │   └── …
│       ├── App.tsx
│       └── main.tsx
│   └── public/
├── server/                # Express + WS + Drizzle
│   ├── Dockerfile
│   ├── index.ts
│   ├── routes.ts           # /chat, /files, /projects …
│   ├── storage.ts          # S3 helpers, quota calc
│   ├── db.ts               # Postgres + pgvector
│   ├── auth/               # JWT, OAuth
│   ├── middlewares.ts
│   ├── n8n-proxy.ts        # Safe tunnel to n8n webhooks
│   └── vite.ts
├── backend-n8n/           # Low-code automations (AI, embeds, alerts)
│   └── Dockerfile
├── infra/                 # docker-compose, IaC stubs
│   ├── docker-compose.yml
│   └── .env
├── shared/                # Zod/TypeScript shared schemas
│   └── schema.ts
├── docs/
│   ├── app-overview-prd.md
│   └── layer-specs/        # A_Server, B_n8n, C_Hybrid
└── …
```

---

## 1. Vision & Positioning

| Topic                      | Statement                                                                                                                                                                                                                                                  |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Core Idea**              | *Chat-first* AI workspace where every prompt, reply, and generated asset becomes a **first-class, navigable document**—not an ephemeral chat scroll.                                                                                                       |
| **Primary Users**          | • **Developers & dev-learners** doing research, RFCs, snippets  • **Technical writers / researchers** compiling large knowledge bases  • **Power users** who hate losing AI context.                                                                       |
| **Key Pain Points Solved** | 1. Traditional chat (ChatGPT) makes file output painful to retrieve. 2. Context history is hard to curate. 3. Collaboration on prompts / results requires copy-paste.                                                                                      |
| **Differentiators**        | • “/” **slash-commands** create full docs, code snippets, tasks  • **File sidebar** & **favorites** directly inside chat thread  • Built-in **project switcher** + file move  • n8n automations (AI embeddings, alerts, backups) configurable per project. |

---

## 2. Functional Pillars

### 2.1 Conversational AI Engine

* GPT-4o (default) via n8n **AI Transform** workflow.
* Chat history cached in Redis per-thread (20-message sliding window).
* Slash-command parser (`/new-doc`, `/outline`, `/summarise`, `/thumb`) handled in **ActionButtons.tsx** + server route `/chat/command`.

### 2.2 File & Document Layer

* Every AI-generated markdown / Tiptap JSON stored in `project_docs`.
* **Move / copy** across projects via drag-drop; Slack notify via n8n.
* Stars, follows, version history (90-day retention).
* Image uploads create auto-thumbnails through **n8n Image Optimize**.

### 2.3 Navigation & Search

* Global fuzzy search (users, files, chat threads) under `Ctrl+K`.
* pgvector + embeddings (workflow 8) for semantic “Ask the Project” queries.
* **Favorites view**, **Recent AI**, **Digest email** Monday summary.

### 2.4 Collaboration & Permissions

* Roles: **owner / editor / viewer** at project level.
* Real-time WS events: `chat.message`, `doc.updated`, `file.moved`.
* Follow a doc to get DM on moves or new AI versions.

### 2.5 Automation (n8n)

Workflows 1-11 (see *B\_n8n layer*) covering:
`Ask AI` → backups, Slack/email, embeddings, nightly cleanup, digests.

---

## 3. Product Requirements (MVP)

| #  | Requirement                               | Acceptance Test                                                         |
| -- | ----------------------------------------- | ----------------------------------------------------------------------- |
| P1 | Chat window with slash-commands           | Typing `/new-doc Project Brief` creates a Tiptap doc linked in sidebar. |
| P2 | AI responses attach as doc or inline text | `mode=generate` returns doc link; editor opens in split pane.           |
| P3 | File move across projects                 | Drag file tile → new project; Slack messages in both channels.          |
| P4 | Search bar surfaces files + chat          | `invoice` returns ≤ 500 ms results list sorted by project.              |
| P5 | Weekly digest email                       | Monday 08:00 UTC email lists top 5 starred docs; open-rate tracked.     |
| P6 | Thumbnails for images                     | Upload PNG → 400-px JPEG thumb stored; appears in file card.            |

chats can contain files, files cant contain chats... project can contain both files, chats and chats with files and projects 
also have imported reference files for project context and also a project instructions user written message for project purpose 
and context to persist through all files and chats. 

---

## 4. Non-Functional Targets

| Dimension                 | Target                                                |
| ------------------------- | ----------------------------------------------------- |
| Latency (chat round-trip) | ≤ 1 s P95 (server→n8n→server)                         |
| AI cost ceiling           | <\$0.002 per request average                          |
| File durability           | Primary S3 + daily Drive backup                       |
| Availability              | 99.9 % (excluding planned n8n upgrades)               |
| Privacy                   | GDPR export/delete APIs; token logging off by default |

---

## 5. Release Timeline

| Phase     | Milestones                                         | ETA     |
| --------- | -------------------------------------------------- | ------- |
| **Alpha** | Core chat + `/summarise` + doc sidebar             | Month 1 |
| **Beta**  | Project roles, file move, Slack alerts, thumbnails | Month 2 |
| **GA**    | Embeddings search, weekly digests, quota & cleanup | Month 3 |

---

## 6. Open Questions

1. Will we support real-time multi-cursor for Editor.js out of the box?
2. How granular should file permissions be (per-file vs project-wide)?
3. Preferred AI model fallbacks if GPT-4o budget exceeded?

---

**Next Actions**

* Review scope with engineering & design.
* Finalize DB schema (`shared/schema.ts`).
* Spin up `infra/docker-compose.yml` with Postgres, Redis, n8n.

*(End of overview)*
