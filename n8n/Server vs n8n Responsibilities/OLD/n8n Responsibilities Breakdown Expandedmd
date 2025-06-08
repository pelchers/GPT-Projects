## 📁 Project File Tree (Reference Layout)

```
/monorepo-root
├── attached_assets/                # User-uploaded assets
├── backup/                         # Backup configs
├── client/                         # Vite + React app
│   └── src/
│       ├── components/
│       ├── hooks/
│       ├── lib/
│       ├── pages/
│       └── main.tsx
├── docs/                           # Versioned guides & API docs
├── fixes/                          # Bug‑fix notes
├── server/                         # Express + Drizzle backend
│   ├── routes/
│   ├── db/                         # Schemas + migrations
│   ├── middleware/
│   ├── sockets/                    # WS gateway
│   └── index.ts
├── shared/                         # Zod types & shared enums
├── n8n/                            # Low‑code automations
├── tailwind.config.ts
└── ...
```

*Everything below refers to these directories.*

---

## 🅰️ Deep‑Dive – **Server‑Only Layer**

*(Reference master doc ID 68424b0961d481919a327eefac300105)*
Each numbered item now contains:

1. **Purpose** – why it exists
2. **Type Definitions** – shared types from `/shared/`
3. **DB Schema** – `drizzle` schema snippet
4. **Flow Diagram** – UI ➜ Srv ➜ DB ➜ WS (arrows indicate direction)
5. **Route / Code** – minimal example
6. **Gotchas & Tips**

---

### 1️⃣ Auth / JWT Validation

**Purpose**: Guarantee every request carries verified user & role claims

**Types (shared/jwt.ts)**

```ts
export interface JwtUser { id: UUID; role: 'user'|'admin'; email: string }
```

**Schema**

```ts
export const users = pgTable('users', {
  id: uuid('id').primaryKey(),
  email: text('email').unique().notNull(),
  passwordHash: text('password_hash').notNull(),
  role: text('role').default('user')
})
```

**Flow**

```
UI ➜ Authorization header ——▶ Srv verifyToken middleware ——▶ next handler
                             ▲                            │
                             │ (throws 401/403)            ▼
                           DB not hit               route logic continues
```

**Route / Code**

```ts
// server/middleware/auth.ts
import { RequestHandler } from 'express';
import jwt from 'jsonwebtoken';
import { JwtUser } from '../../shared/jwt';

export const verifyToken: RequestHandler = (req,res,next)=>{
  const token = req.headers.authorization?.split(' ')[1]
  if(!token) return res.status(401).json({error:'No token'})
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET) as JwtUser
    return next()
  } catch(err){
    return res.status(403).json({error:'Invalid or expired token'})
  }
}
```

**Gotchas & Tips**

* 🕐 **Clock‑skew**: ensure server & n8n timers in sync or `exp` may fail.
* 🪪 **Role caching**: embed `role` so no extra DB read is required.
* 🔁 **Refresh flow**: client hits `/auth/refresh` to rotate token before expiry.

---

### 2️⃣ Fuzzy User & File Search

**Purpose**: Power autocomplete dropdown & quick‑file search in modals.

**Types**

```ts
export interface SearchUser { id: UUID; name: string; avatar: string|null }
export interface SearchFile { id: UUID; title: string }
```

**Schema / Index** (migration snippet)

```sql
CREATE INDEX users_name_trgm ON users USING gin (name gin_trgm_ops);
CREATE INDEX files_title_trgm ON files USING gin (title gin_trgm_ops);
```

**Flow**

```
UI ➜ /search?q=ann ─▶ Srv controller ─▶ DB ILIKE + LIMIT ─▶ Srv json ─▶ UI list
```

**Route**

```ts
router.get('/search', verifyToken, async (req,res)=>{
  const q = (req.query.q || '').toString()
  const users = await db.execute(sql`
    SELECT id,name,avatar FROM users
    WHERE name ILIKE ${'%' + q + '%'} LIMIT 8`)
  const files = await db.execute(sql`
    SELECT id,title FROM files
    WHERE project_id IN (
      SELECT project_id FROM project_members WHERE user_id=${req.user.id})
      AND title ILIKE ${'%' + q + '%'} LIMIT 8`)
  res.json({users,files} satisfies {users:SearchUser[];files:SearchFile[]})
})
```

**Gotchas**

* Scope files by membership or sharing flag to prevent data‑leak.
* Add `ILIKE $1 ESCAPE '\'` to safely handle `% _` chars.

---

### 3️⃣ Project & ACL CRUD

**Purpose**: Manage projects, enforce member roles.

**Types**

```ts
export type MemberRole = 'owner'|'editor'|'viewer'
export interface Project { id:UUID; ownerId:UUID; title:string; slug:string }
export interface ProjectMember { projectId:UUID; userId:UUID; role:MemberRole }
```

**Schema**

```ts
export const projects = pgTable('projects', {
  id: uuid('id').primaryKey().defaultRandom(),
  ownerId: uuid('owner_id').notNull(),
  title: text('title').notNull(),
  slug: text('slug').unique().notNull()
})
export const projectMembers = pgTable('project_members', {
  projectId: uuid('project_id').notNull(),
  userId: uuid('user_id').notNull(),
  role: text('role').$type<MemberRole>()
})
```

**Flow (create)**

```
UI ➜ POST /projects ─▶ Srv tx (insert project + member) ─▶ DB ⇆ ─▶ Srv JSON ─▶ UI
```

**Gotchas**

* Wrap slug generation in retry loop (unique violation) for high concurrency.

---

### 4️⃣ Favorites / Stars

**Purpose**: Personal quick‑access list.

**Type & Schema**

```ts
export const userFavorites = pgTable('user_favorites', {
  userId: uuid('user_id'),
  docId: uuid('doc_id'),
  createdAt: timestamp('created_at').defaultNow()
})
```

**Flow**

```
UI ★ ➜ POST /docs/:id/star ─▶ Srv UPSERT ─▶ DB ⇆ ─▶ WS ⇢ favorites.updated
```

**Tip**: use compound PK (userId, docId) for idempotent toggle.

---

### 5️⃣ Friends / Follows

**Purpose**: Build social feed & @mentions.

**Schema**

```ts
export const userFollow = pgTable('user_follow', {
  followerId: uuid('follower_id'),
  followeeId: uuid('followee_id'),
  createdAt: timestamp('created_at').defaultNow()
})
```

**Flow**

```
UI ➜ POST /users/:id/follow ─▶ Srv insert ─▶ DB ⇆ ─▶ WS ⇢ follow.feed
```

**Gotcha**: Add unique index `(followerId, followeeId)` to prevent dup.

---

### 6️⃣ File Move / Copy

**Schema impact**: Only modify `projectId` FK or duplicate row with new UUID.

**Flow**

```
UI ➜ /files/:id/move ─▶ Srv ACL ✔ ─▶ DB update ─▶ WS file.moved
                                 └───▶ n8n webhook (async Slack msg)
```

---

### 7️⃣ Chat Creation & Chat‑File Linking

**Tables**

```ts
projectChats(id,projectId,title,slug)
chatFiles(chatId,fileId)
```

**Flow**

```
UI ➜ POST /projects/:id/chat ─▶ Srv insert  ─▶ WS chat.created
AI file (n8n) ─▶ POST /api/files ─▶ Srv insert
                  └── link chatFiles ─▶ WS chat.message(file)
```

---

### 8️⃣ Image Presign & Quota

**Quota Calculation**

```
newSize + user.storage_used <= plan.max_bytes ? allow : 403
```

**Flow**

```
UI ➜ /uploads/presign ─▶ Srv quota ✔ ─▶ S3 signedURL ─▶ UI PUT
UI PUT success webhook ─▶ Srv update bytes_used
```

---

### 9️⃣ Search Index Sync

Run in same txn after doc save to avoid stale vector.

---

### 🚀 Full Request Lifeline (favorite example)

```
UI click ★
  ↓ fetch (jwt)
Srv verifyToken
  ↓ upsert user_favorites
DB write
  ↓ Ws broadcast favorites.updated
UI optimistically updates list
```

This concludes the hyper‑detailed Server‑Only spec.
