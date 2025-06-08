## 🔗 n8n + Server Integration Overview for Team‑Docs

This file explains **where the server (Express + Drizzle)** takes charge, **where n8n** picks up automation, and **how both coordinate** for the team‑project document flow (Tiptap / Editor.js). It’s structured as **three quick tables** followed by **three deep‑dive sections** with code.

---

### 📊 Table 1 – Responsibility Matrix

| Layer              | Primary Duties                                     | Key Endpoints / Nodes                                          |
| ------------------ | -------------------------------------------------- | -------------------------------------------------------------- |
| **React Frontend** | UI, “Ask AI” button, autocomplete dropdown         | `/api/ai/*` (fetch)                                            |
| **Express Server** | Auth, validation, DB writes, proxy to n8n          | `/api/ai/tiptap`, `/api/ai/editorjs`, `/api/projects/:id/docs` |
| **n8n**            | AI calls, notifications, file sync, version writes | `*/webhook/*-ai-request`, Postgres node, Slack node            |

### 📊 Table 2 – Data‑Flow Steps (Happy Path)

| # | Action                       | Origin → Destination            |
| - | ---------------------------- | ------------------------------- |
| 1 | User clicks **Ask AI**       | React → Express                 |
| 2 | Server proxies JSON          | Express → n8n Webhook           |
| 3 | n8n processes (OpenAI, save) | n8n internal                    |
| 4 | n8n returns JSON             | n8n → Express                   |
| 5 | Server relays to UI          | Express → React                 |
| 6 | UI updates doc; save version | React → Express (optional save) |

### 📊 Table 3 – Storage & Versioning Owners

| Data            | Written By                                   | Location                    |
| --------------- | -------------------------------------------- | --------------------------- |
| **Current doc** | React (save) → Express → Drizzle             | `project_docs` table        |
| **AI version**  | n8n (Postgres node) *or* Express after relay | `document_versions` table   |
| **File backup** | n8n (File node)                              | `/files/{userId}/ai/..json` |

---

## Section A – Frontend Trigger ➜ Server Proxy

### 1 ️⃣ React Fetch Helper

```ts
// triggerAI.ts (shared for Tiptap / Editor.js)
export async function triggerAI(path: 'tiptap'|'editorjs', payload) {
  const res = await fetch(`/api/ai/${path}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  })
  return res.json()
}
```

### 2 ️⃣ Express Proxy Route

```ts
// routes/ai.ts
router.post('/:format', async (req, res) => {
  const { format } = req.params // 'tiptap' or 'editorjs'
  const webhook = `https://n8n.myhost.com/webhook/${format}-ai-request`
  const aiResp = await fetch(webhook, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(req.body)
  })
  const json = await aiResp.json()
  res.json(json) // step 5 of Table 2
})
```

**Explainer:** The server *only* validates auth & size limits, logs, then forwards to n8n. No AI keys live in server env.

---

## Section B – n8n Workflow Details

### 1 ️⃣ Webhook Trigger Node

* **URL**: `/webhook/tiptap-ai-request` (or `editorjs-ai-request`)
* Expects: `prompt`, `mode`, `content`, `userId`.

### 2 ️⃣ Set / Function Node (Prompt Builder)

```js
let sys = '';
if ($json.mode === 'summarize') sys = 'Summarize:';
if ($json.mode === 'rewrite')  sys = 'Rewrite clearly:';
return { prompt: `${sys}\n${JSON.stringify($json.content)}` };
```

### 3 ️⃣ OpenAI Node

* Model: **gpt‑4o**
* Input: `{{$node["Set"].json["prompt"]}}`

### 4 ️⃣ Function → Editor‑Specific Formatter

```js
// For Tiptap
return {
  content: { type:'doc', content:[{ type:'paragraph', content:[{type:'text',text:$json.response}]}] }
}
```

```js
// For Editor.js
return {
  content: { time:Date.now(), blocks:[{ type:'paragraph', data:{ text:$json.response } }], version:'2.28.0' }
}
```

### 5 ️⃣ Postgres Node (Optional)

```sql
INSERT INTO document_versions (document_id, content, author_type, tag)
VALUES ({{ $json.docId }}, {{ $json.content }}, 'ai', 'ai_generated');
```

### 6 ️⃣ Slack Node (Notify Team)

```
“✍️ AI updated *{{ $json.docTitle }}* in project {{ $json.projectName }}”
```

### 7 ️⃣ HTTP Response Node

Returns `{ content, status:'success' }` to Express.

---

## Section C – Sync & Dashboard Surfacing

### 1 ️⃣ Persist Version (Server)

```ts
await db.insert(documentVersions).values({
  documentId: docId,
  content: aiJson,
  authorType: 'ai',
  tag: 'ai_generated'
})
```

### 2 ️⃣ Send to Personal Dashboards

* **Server** queries `project_members` for the project.
* Loops through each `userId` and writes a record to `user_dashboard_docs` linking the new version.

### 3 ️⃣ Render in Team Tab

```tsx
<ProjectDocsGrid docs={projectDocs} />
```

* Conditionally render editor based on `doc.format`.
* Provide **history dropdown** → fetches `document_versions`.

**Explainer:** Because both editors output JSON, the dashboard just needs to know which renderer to mount.

---

### 🚀 Key Takeaways

1. **Server** = gatekeeper (auth, logging) & DB API.
2. **n8n** = heavy‑lift automation (AI, notifications, backups).
3. Keep **format** field (`'tiptap'|'editorjs'`) + **version table** for effortless cross‑editor collaboration.

> Start small (no Postgres node) → add persistence when ready. n8n flows remain stateless to the UI; the server mediates all security.
