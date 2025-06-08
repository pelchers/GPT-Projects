# Part 2 — Frontend Web App Setup (n8n AI Agent Project)

---

## Overview

This guide covers how to build the frontend for your n8n-powered AI agent platform, with a focus on:

* ChatGPT-style interface (using OpenUI or a custom React component)
* Action buttons for triggering documentation/agent tasks (e.g., send to Notion)
* API communication with your n8n backend (using standard n8n Docker build)
* Best practices for environment variables and frontend security

---

## A. ChatGPT-style Chat Integration

### Option 1: Custom React Components

#### 1. Choosing a UI Toolkit

* **OpenUI**: An emerging standard for customizable UI primitives. Use it if available, or pick a mature React chat component (e.g., [react-chat-ui](https://github.com/brandonmowat/react-chat-ui), [react-chatbot-kit](https://fredrikoseberg.github.io/react-chatbot-kit-docs/)).
* **Custom implementation**: Build your own chat interface in React, with components for chat history, message input, user/agent avatars, etc.

#### 2. Basic Component Structure

* `<ChatWindow />` — displays chat history, auto-scrolls to latest
* `<MessageInput />` — lets the user type/submit requests
* `<ActionButtons />` — quick actions like “Send to Notion”, “Document in Airtable”, etc.
* State management via React Context, Redux, or useState/useReducer

---

### Option 2: Using Open WebUI for the Main Chat Feature

* **Import ********[Open WebUI](https://github.com/open-webui/open-webui)******** as a standalone or embedded chat interface** for the main chat area of your app.

  * Open WebUI provides a ChatGPT-like, full-featured chat experience out-of-the-box and can be themed or embedded as an iframe or imported React component, depending on your build process.
  * Use Open WebUI for the home/main chat feature or AI assistant experience.
* **For specialized, task-specific chats elsewhere in the app:**

  * Use your custom `<ChatWindow />`, `<MessageInput />`, and `<ActionButtons />` components for use cases such as onboarding, workflow-specific sidebars, or pages where chat is a secondary feature or needs custom logic/integration.
* **Benefits of this approach:**

  * Quick setup of a rich, production-grade chat interface for your main AI feature.
  * Retain full customizability for other areas—easier to optimize specialized workflows or limit scope/context.
* **Deployment:**

  * Integrate Open WebUI via npm (if available) or as an embedded iframe, then use React for custom components elsewhere.
  * Route API requests from Open WebUI to your n8n backend just as you do for custom React chat components.

#### Importing Open WebUI as a React Component/Module

* **If Open WebUI is published as an npm/yarn package:**

  1. Install:

     ```bash
     npm install open-webui
     # or
     yarn add open-webui
     ```
  2. Import and use as a component:

     ```jsx
     import { OpenWebUIChat } from 'open-webui';

     function MainChat() {
       return (
         <OpenWebUIChat
           apiEndpoint={import.meta.env.VITE_N8N_API_URL}
           theme="light"
           // ...other props/config
         />
       );
     }
     ```
  3. **Configuration:**

     * Set the `apiEndpoint` to your n8n backend.
     * Customize theme, prompt, etc., as props if supported.
     * Refer to [Open WebUI documentation](https://github.com/open-webui/open-webui) for all available props and advanced options.

* **If no official module exists:**

  * Use the iframe embedding method or follow the repo’s instructions for custom integrations.

* **Why use the React component/module approach?**

  * Directly integrates Open WebUI into your React app’s component tree.
  * Enables better control of props, style, state, and events.
  * More maintainable and future-proof as your frontend evolves.

---

#### Notes: Open WebUI vs Custom React Chat UI

* **Open WebUI:**

  * **Best for:** Rapidly deploying a robust, ChatGPT-style interface with minimal coding.
  * **Customizability:**

    * As an *embedded iframe*, you get limited visual/theming control; deep customizations or integrating with app state is harder.
    * If available as a React (or other framework) module, you can import and configure more, but still have less internal flexibility than building your own.
    * Built-in features (chat history, user management, plugins) may be more than needed for simple/targeted agent tasks.
  * **Embedding:**

    * Easiest via iframe—add to any page, fast and simple, but limited integration with your app’s UI/state.
    * If npm/library install is available, you can import it as a React component for better integration and some customization.
    * For a dedicated, “main” AI chat, Open WebUI is ideal. For smaller, specialized or tightly integrated agent features, custom React components are better.
  * **Ease of use:** Fastest to launch, minimal code, but “deeper” custom logic may be more complex than writing your own.
* **Custom React Chat UI:**

  * **Best for:** Complete control, fine-tuned design, embedding chat anywhere, and adapting to unique agent workflows.
  * **Customizability:** Unlimited—design the exact chat and UI logic you need.
  * **Integration:** Works seamlessly with any React/Vite/Next.js app state or data. Enables tight coupling with other UI features or workflows.
  * **Ease of use:** More upfront work, but scales much better as your feature set or business requirements grow.
* **Summary Recommendation:**

  * Use **Open WebUI** if your main goal is quick deployment of a full-featured AI chat and you’re fine with “black box” integration.
  * Use **custom React chat UI** if you need total control, frequent updates, or specialized chat experiences.
  * **Hybrid is great:** Open WebUI for the “main” AI chat, custom components for specialized use on other pages/flows.

---

## B. Web App Structure & File Layout Example

```
my-app/
├── client/                        # Frontend React app (Open WebUI + custom chat)
│   ├── Dockerfile                 # Docker build for frontend
│   ├── package.json
│   ├── src/
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   ├── chat/
│   │   │   │   ├── ChatWindow.tsx         # Custom chat window for specialized flows
│   │   │   │   ├── MessageInput.tsx       # Custom input component for chat
│   │   │   │   ├── ActionButtons.tsx      # Buttons for task triggers
│   │   │   │   └── index.ts
│   │   ├── hooks/
│   │   │   ├── useChatApi.ts              # Hook for sending messages to n8n
│   │   │   ├── usePageContent.ts          # Hook for custom backend API
│   │   ├── lib/
│   │   │   ├── queryClient.ts             # React Query config for APIs
│   │   │   ├── utils.ts                   # Utility functions
│   │   │   └── api.ts                     # Central API helpers
│   │   ├── pages/
│   │   │   ├── chat.tsx                   # Main chat page (Open WebUI)
│   │   │   ├── feature-support.tsx        # Specialized chat (custom React)
│   │   │   └── ...                        # Other pages
│   │   ├── App.tsx                        # Main app & routing
│   │   └── main.tsx
│   └── public/
├── backend-n8n/                  # RECOMMENDED: n8n as a dedicated service/container for workflows and agent logic (production, cloud, Docker setups)
│   ├── Dockerfile                # Customizes n8n, or just use official image
│   └── ...
├── server/                       # Core backend (Express for API, proxy, DB, or advanced logic)
│   ├── Dockerfile                # For containerized custom backend
│   ├── index.ts                  # Server entry point
│   ├── routes.ts                 # API routes (REST endpoints)
│   ├── storage.ts                # Data access layer (ORM/db)
│   ├── db.ts                     # DB connection/config
│   ├── middlewares.ts            # Express middlewares
│   ├── n8n-proxy.ts              # (Optional) Proxy for n8n if tight coupling is needed (not default)
│   └── vite.ts                   # Vite integration (SSR/dev)
├── infra/                        # Infrastructure and orchestration config
│   ├── docker-compose.yml        # Main Compose file for local multi-service dev
│   ├── .env                      # Shared env variables (never committed)
│   └── README.md
├── shared/                       # Shared types, schemas
│   └── schema.ts
├── docs/
│   ├── file-tree-structure.md
│   └── ...
├── .gitignore
├── .env                          # Never commit secrets!
├── package.json
└── ...

```

* **Key points:**

  * Keep all chat and action UI logic in `/components/`
  * Use `/hooks/` for any reusable logic (e.g., `useChatApi`)
  * Pages can be routed with React Router or similar
  * Dockerfile allows containerized builds for local or cloud deployment

---

## C. Action Buttons and API Integration

### 1. Action Buttons

* Render below or beside the chat input
* Examples: “Document in Notion”, “Save to Google Docs”, “Send to Airtable”
* On click, send a custom payload or flag in the next message to n8n (e.g., `{ action: 'notion', content: '...' }`)

### 2. Connecting to n8n API

* Use `fetch` or `axios` to send chat/actions to your n8n HTTP endpoint
* Endpoint URL should be set via env variable (e.g., `VITE_N8N_API_URL`)
* Example:

```ts
const response = await fetch(`${import.meta.env.VITE_N8N_API_URL}/webhook/ai-agent`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ message, action })
});
```

* Handle loading state, errors, and render agent’s response in the chat window

---

## D. Deployment & Environment Variables

* Store API URLs and any public config in `.env` or via Render dashboard for cloud
* Example `.env` for frontend:

```
VITE_N8N_API_URL=http://localhost:5678
```

* For local dev, use Vite or Docker Compose to run alongside backend
* For Render.com: deploy as a separate service; set `VITE_N8N_API_URL` in Render dashboard to point to your backend service’s public URL

---

## E. Security & Best Practices

* **Never expose secrets** (API keys, private URLs, DB credentials) in frontend code or `.env`
* Only use public/configuration variables (e.g., n8n endpoint)
* Always validate/escape user input before sending to backend
* Use HTTPS for all network communication

---

## Next Steps

* Scaffold your chat UI and components
* Connect to your running n8n instance (with standard n8n Docker build)
* Test chat flow, action buttons, and API round-trips
* Move to Part 3: n8n Agent Workflow & Backend Logic





# Frontend Chat Components – React UI & Open WebUI Integration

This guide demonstrates how to structure your chat features using both a custom React component library for specialized/chat-enabled pages and Open WebUI for the main ChatGPT-like experience in your app.

---

## 1. Main Chat Page: Open WebUI Integration

### **A. Install and Import Open WebUI**

* If Open WebUI is published as an npm/yarn package:

  ```bash
  npm install open-webui
  # or
  yarn add open-webui
  ```

### **B. Usage on Main Chat Page**

```jsx
import { OpenWebUIChat } from 'open-webui';

export default function MainChatPage() {
  return (
    <div style={{ height: '100vh' }}>
      <OpenWebUIChat
        apiEndpoint={import.meta.env.VITE_N8N_API_URL}
        theme="light"
        // other props as needed
      />
    </div>
  );
}
```

* Place this component on your `/chat`, `/ai`, or homepage as your central conversational interface.
* Pass `apiEndpoint` and other props as needed to configure Open WebUI’s behavior and backend connectivity.
* Style with your own CSS, wrapper div, or use provided theming options.

---

## 2. Specialized Pages: Custom React Chat Components

* Use your own React chat UI library or components (e.g., `<ChatWindow />`, `<MessageInput />`, `<ActionButtons />`) on onboarding, workflow, or feature-specific pages.
* Example page structure for a context-aware support bot or feature chat:

```jsx
import ChatWindow from '../components/ChatWindow';
import MessageInput from '../components/MessageInput';
import ActionButtons from '../components/ActionButtons';

export default function FeatureSupportPage() {
  const [messages, setMessages] = useState([]);

  // Example: handle message submission
  const handleSend = (message, action) => {
    fetch(`${import.meta.env.VITE_N8N_API_URL}/webhook/ai-agent`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message, action }),
    })
      .then((res) => res.json())
      .then((reply) => setMessages((msgs) => [...msgs, { role: 'ai', content: reply.content }]))
      .catch(/* handle error */);
  };

  return (
    <div>
      <ChatWindow messages={messages} />
      <MessageInput onSend={handleSend} />
      <ActionButtons onAction={handleSend} />
    </div>
  );
}
```

* Compose your chat UI flexibly for sidebars, popups, task-specific flows, etc.
* Add logic for handling context, user roles, or special triggers as needed.

---

## 3. When to Use Each Approach

* **Open WebUI:** Use for the main, high-engagement AI chat experience with maximum features and easy theming.
* **Custom React Components:** Use for embedding chat into specialized flows, onboarding, project dashboards, or anywhere fine-grained UX/control is needed.
* **Hybrid Approach:** Main chat page (Open WebUI) + custom chat widgets/components on other pages for the best mix of speed and flexibility.

---

## 4. Best Practices

* Keep your custom chat components modular and reusable for rapid adaptation as requirements evolve.
* Maintain a clear interface/API between chat UI and n8n endpoints for future backend upgrades.
* Only pass public configuration (never secrets) to Open WebUI or your custom chat.
* Use environment variables for backend URLs and configure per environment (local/dev/prod).
* Style and theme chat UI to match your overall brand/app for a unified user experience.




# Part 2 Continued Expansion – File Tree & API Flow (n8n Hybrid)

This document provides a full reference for the project file tree and the communication flow between frontend (React), traditional backend (Express/DB), and n8n agent workflows. Use this as your central reference for any full-stack n8n hybrid implementation.

---

Here’s the **updated file tree and notes** for your **Part 2 Continued Expansion – File Tree & API Flow (n8n Hybrid)** doc, per your new specification:

---

## Project File Tree Structure

```
my-app/
├── client/                        # Frontend React app (Open WebUI + custom chat)
│   ├── Dockerfile                 # Docker build for frontend
│   ├── package.json
│   ├── src/
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   ├── chat/
│   │   │   │   ├── ChatWindow.tsx         # Custom chat window for specialized flows
│   │   │   │   ├── MessageInput.tsx       # Custom input component for chat
│   │   │   │   ├── ActionButtons.tsx      # Buttons for task triggers
│   │   │   │   └── index.ts
│   │   ├── hooks/
│   │   │   ├── useChatApi.ts              # Hook for sending messages to n8n
│   │   │   ├── usePageContent.ts          # Hook for custom backend API
│   │   ├── lib/
│   │   │   ├── queryClient.ts             # React Query config for APIs
│   │   │   ├── utils.ts                   # Utility functions
│   │   │   └── api.ts                     # Central API helpers
│   │   ├── pages/
│   │   │   ├── chat.tsx                   # Main chat page (Open WebUI)
│   │   │   ├── feature-support.tsx        # Specialized chat (custom React)
│   │   │   └── ...                        # Other pages
│   │   ├── App.tsx                        # Main app & routing
│   │   └── main.tsx
│   └── public/
├── backend-n8n/                  # RECOMMENDED: n8n as a dedicated service/container for workflows and agent logic (production, cloud, Docker setups)
│   ├── Dockerfile                # Customizes n8n, or just use official image
│   └── ...
├── server/                       # Core backend (Express for API, proxy, DB, or advanced logic)
│   ├── Dockerfile                # For containerized custom backend
│   ├── index.ts                  # Server entry point
│   ├── routes.ts                 # API routes (REST endpoints)
│   ├── storage.ts                # Data access layer (ORM/db)
│   ├── db.ts                     # DB connection/config
│   ├── middlewares.ts            # Express middlewares
│   ├── n8n-proxy.ts              # (Optional) Proxy for n8n if tight coupling is needed (not default)
│   └── vite.ts                   # Vite integration (SSR/dev)
├── infra/                        # Infrastructure and orchestration config
│   ├── docker-compose.yml        # Main Compose file for local multi-service dev
│   ├── .env                      # Shared env variables (never committed)
│   └── README.md
├── shared/                       # Shared types, schemas
│   └── schema.ts
├── docs/
│   ├── file-tree-structure.md
│   └── ...
├── .gitignore
├── .env                          # Never commit secrets!
├── package.json
└── ...

```

---

### Note: Should n8n Be Separate from the Server? (Best Practice)

**Keeping ******\`\`****** as a Separate Service/Folder**

* **Pros:**

  * n8n is designed to run as its own process/container.
  * Updates, scaling, and debugging are easier; restart n8n without touching your main server.
  * Keeps workflow logic and business logic clearly separated.
  * Works perfectly with Docker Compose, Render.com, Railway, and other cloud PaaS.
  * No coupling headaches—each service has its own lifecycle.
* **Cons:**

  * If you want ultra-tight integration (running custom Node code in n8n, or calling Express code from inside n8n), you’ll need to use HTTP APIs or set up a simple proxy route (see `server/n8n-proxy.ts`).
  * Monolith deployment (n8n + server in one process) is not supported and leads to maintenance pain.

**Combining n8n with Server Code in One Folder/Process:**

* **Possible, but not recommended.**
* Not officially supported, makes upgrades harder, can cause permission/port conflicts, and doesn’t scale or fit cloud best practices.
* Only use if you have a *very* special reason.

**Best Practice / Recommendation:**

* **Always keep ******\`\`****** separate from your server.**
* Use HTTP API/webhooks for integration, or an optional minimal proxy route (`server/n8n-proxy.ts`) if truly necessary.
* This matches n8n’s architecture, cloud deployment, and modern scalable project standards.

> **Summary:** For maintainability, scalability, and compatibility, run n8n as a dedicated service/folder/container (`backend-n8n`). Use the `server/` backend for core API, DB, or advanced logic. Only tightly couple with n8n via proxy when necessary, not as the default.

---

**Your file is now accurate and aligned with your architecture and best practices!**

## API & Communication Flow

### **Standard API Data (Page Content, Admin, etc.)**

```
client/src/hooks/usePageContent.ts
  → fetches /api/page-content/:pageId
    → server/routes.ts (Express API route)
      → server/storage.ts (DB access via ORM)
        → Database (e.g., Postgres)
```

* **Usage in Component:**

```typescript
// client/src/pages/about.tsx
import { usePageContent } from '../hooks/usePageContent';

export default function About() {
  const { data: aboutContent, isLoading } = usePageContent('about');
  return <div>{isLoading ? 'Loading...' : aboutContent?.title || 'Default Title'}</div>;
}
```

---

### **AI Agent / Chat Actions via n8n**

```
client/src/hooks/useChatApi.ts
  → fetches [n8n]/webhook/ai-agent
    → n8n workflow (no Express in path)
      → Performs documentation, task, or logging via n8n nodes
      → Optionally calls Notion, Airtable, Google Docs, etc.
```

* **Usage in Component:**

```typescript
// client/src/pages/feature-support.tsx
import useChatApi from '../hooks/useChatApi';

const { sendMessage } = useChatApi();

const handleSend = (message, action) => {
  sendMessage({ message, action }).then((reply) =>
    setMessages((msgs) => [...msgs, { role: 'ai', content: reply.content }])
  );
};
```

---

## React Query Setup & Central API Helpers

**File:** `client/src/lib/queryClient.ts`

```typescript
import { QueryClient } from 'react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: ({ queryKey }) => fetch(queryKey[0]).then(res => res.json()),
    },
  },
});
```

**File:** `client/src/hooks/usePageContent.ts`

```typescript
import { useQuery } from 'react-query';

export function usePageContent(pageId: string) {
  return useQuery({
    queryKey: [`/api/page-content/${pageId}`],
  });
}
```

---

## Server API Route Example

**File:** `server/routes.ts`

```typescript
app.get('/api/page-content/:pageId', async (req, res) => {
  const content = await storage.getPageContent(req.params.pageId);
  res.json(content);
});
```

**File:** `server/storage.ts`

```typescript
export async function getPageContent(pageId: string) {
  return await db
    .select()
    .from(pageContent)
    .where(eq(pageContent.pageId, pageId));
}
```

---

## n8n Integration Example

**File:** `client/src/hooks/useChatApi.ts`

```typescript
export default function useChatApi() {
  const sendMessage = async ({ message, action }) => {
    const response = await fetch(`${import.meta.env.VITE_N8N_API_URL}/webhook/ai-agent`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message, action }),
    });
    return await response.json();
  };
  return { sendMessage };
}
```

---

## Notes

* All n8n workflow endpoints and backend API URLs should be managed with environment variables (e.g., `VITE_N8N_API_URL`).
* Use React Query and API hooks to keep all API access centralized, cache-aware, and modular.
* The backend (Express) can provide additional REST endpoints and proxy to n8n as needed for legacy or advanced workflows.
* Keep this file tree and API flow doc up to date across all project documentation!
