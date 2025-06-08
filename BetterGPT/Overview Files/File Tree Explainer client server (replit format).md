# 🌟 BetterGPT - Project File Tree Structure

This document provides a comprehensive overview of the project's file organization and directory structure, optimized for direct service implementations.

## 🔄 System Architecture Overview

```mermaid
graph TD
    subgraph "Frontend"
        A[React App] -->|HTTP/WS| B[API Client]
        B -->|State| C[React Query]
        C -->|Cache| D[Local Storage]
    end
    
    subgraph "Backend"
        E[Express Server] -->|Routes| F[API Endpoints]
        F -->|Process| G[Services]
        G -->|Store| H[Database]
        G -->|Cache| I[Redis]
    end
    
    subgraph "Development"
        J[n8n Server] -->|Prototype| K[New Features]
    end
```

## 📁 Root Directory Structure

```
better-gpt/
├── client/                   # Frontend React application
├── server/                   # Backend Express application
├── dev-playground/          # n8n development playground (non-production)
├── shared/                   # Shared types and schemas
├── docs/                     # Project documentation
│   ├── architecture/         # Technical architecture docs
│   ├── api/                  # API documentation
│   └── guides/              # Development guides
├── .gitignore               # Git ignore configuration
├── .env                     # Environment configuration
├── components.json          # Shadcn/ui components configuration
├── drizzle.config.ts        # Drizzle ORM configuration
├── package-lock.json        # NPM dependency lock file
├── package.json             # NPM package configuration
├── postcss.config.js        # PostCSS configuration
├── tailwind.config.ts       # Tailwind CSS configuration
├── tsconfig.json            # TypeScript configuration
└── vite.config.ts           # Vite build tool configuration
```

## 🎨 Client Directory Structure

```mermaid
graph TD
    subgraph "Client Structure"
        A[src] -->|Components| B[UI Layer]
        A -->|Hooks| C[Logic Layer]
        A -->|Lib| D[Utils Layer]
        A -->|Pages| E[Route Layer]
        
        B -->|Layout| F[Layout Components]
        B -->|Chat| G[Chat Components]
        B -->|Files| H[File Components]
        B -->|UI| I[Base Components]
    end
```

```
client/
├── src/
│   ├── components/          # Reusable React components
│   │   ├── layout/          # Layout components (header, footer)
│   │   ├── chat/            # Chat-related components
│   │   │   ├── ChatWindow.tsx        # Thread pane w/ file previews
│   │   │   ├── MessageInput.tsx      # Slash-commands, prompt helpers
│   │   │   ├── ActionButtons.tsx     # "/new-doc", "/summarise" …
│   │   │   └── index.ts
│   │   ├── files/           # File management components
│   │   │   ├── FilePreview.tsx       # File preview component
│   │   │   ├── FileUpload.tsx        # File upload component
│   │   │   └── index.ts
│   │   └── ui/              # Shadcn/ui component library
│   ├── hooks/               # Custom React hooks
│   │   ├── useChatApi.ts    # Chat API integration
│   │   ├── useFilesApi.ts   # File management API
│   │   ├── useAuth.ts       # Authentication hook
│   │   └── useWebSocket.ts  # WebSocket connection hook
│   ├── lib/                 # Utility libraries
│   │   ├── api.ts           # API client setup
│   │   ├── auth.ts          # Authentication utilities
│   │   ├── queryClient.ts   # React Query configuration
│   │   └── utils.ts         # General utilities
│   ├── pages/               # Page components
│   │   ├── chat/            # Chat interface
│   │   ├── files/           # File management
│   │   ├── projects/        # Project management
│   │   └── settings/        # User settings
│   ├── App.tsx              # Main application component
│   └── main.tsx             # Application entry point
└── public/                  # Static assets
```

## 🖥️ Server Directory Structure

```mermaid
graph TD
    subgraph "Server Structure"
        A[src] -->|Services| B[Core Services]
        A -->|Routes| C[API Routes]
        A -->|Middleware| D[Middleware]
        A -->|DB| E[Database]
        A -->|WebSocket| F[Real-time]
        
        B -->|AI| G[AI Service]
        B -->|Files| H[File Service]
        B -->|Auth| I[Auth Service]
        B -->|Notify| J[Notification Service]
    end
```

```
server/
├── src/
│   ├── services/            # Core service implementations
│   │   ├── ai.ts           # OpenAI integration
│   │   ├── chat.ts         # Chat management
│   │   ├── files.ts        # File processing & storage
│   │   ├── notifications.ts # Slack/Email integration
│   │   └── scheduler.ts    # Cron jobs & scheduled tasks
│   ├── routes/             # API route handlers
│   │   ├── chat.ts         # Chat endpoints
│   │   ├── files.ts        # File endpoints
│   │   ├── projects.ts     # Project endpoints
│   │   └── auth.ts         # Authentication endpoints
│   ├── middleware/         # Express middleware
│   │   ├── auth.ts         # Authentication middleware
│   │   ├── error.ts        # Error handling
│   │   └── validation.ts   # Request validation
│   ├── db/                 # Database configuration
│   │   ├── index.ts        # Database connection
│   │   └── migrations/     # Database migrations
│   ├── websocket/          # WebSocket implementation
│   │   ├── server.ts       # WebSocket server
│   │   └── handlers.ts     # Event handlers
│   ├── utils/              # Utility functions
│   │   ├── logger.ts       # Logging utility
│   │   └── validators.ts   # Validation utilities
│   └── config/             # Configuration files
│       ├── index.ts        # Main configuration
│       └── env.ts          # Environment variables
├── Dockerfile              # Server container definition
└── package.json           # Server dependencies
```

## 🎮 Development Playground

```mermaid
graph TD
    subgraph "Development Playground"
        A[n8n Server] -->|Prototype| B[New Features]
        B -->|Test| C[Workflows]
        B -->|Test| D[Integrations]
        B -->|Document| E[Results]
    end
```

```
dev-playground/            # n8n development environment
├── Dockerfile             # n8n container definition
├── README.md             # Playground documentation
└── workflows/            # Example n8n workflows
    └── README.md         # Workflow documentation
```

## 📚 Documentation Structure

```mermaid
graph TD
    subgraph "Documentation"
        A[docs] -->|Technical| B[Architecture]
        A -->|API| C[Endpoints]
        A -->|Guides| D[Development]
        
        B -->|Overview| E[System Design]
        B -->|Services| F[Service Design]
        B -->|Data| G[Data Flow]
    end
```

```
docs/
├── architecture/          # Technical architecture
│   ├── overview.md       # System overview
│   ├── services.md       # Service architecture
│   └── data-flow.md      # Data flow diagrams
├── api/                  # API documentation
│   ├── endpoints.md      # API endpoints
│   └── websocket.md      # WebSocket events
└── guides/              # Development guides
    ├── setup.md         # Development setup
    ├── deployment.md    # Deployment guide
    └── playground.md    # Development playground guide
```

## 🔑 Key File Purposes

### Configuration Files
- `package.json`: Defines project dependencies and scripts
- `tailwind.config.ts`: Configures Tailwind CSS utility classes
- `vite.config.ts`: Configures Vite build tool
- `drizzle.config.ts`: Configures Drizzle ORM
- `tsconfig.json`: TypeScript compiler configuration

### Core Application Files
- `client/src/App.tsx`: Main React application
- `client/src/main.tsx`: Application bootstrap
- `server/src/index.ts`: Express server setup
- `server/src/routes/`: API endpoint definitions
- `shared/schema.ts`: Database schema definitions

### Service Implementations
- `server/src/services/ai.ts`: OpenAI integration
- `server/src/services/files.ts`: File processing
- `server/src/services/notifications.ts`: External notifications
- `server/src/services/scheduler.ts`: Scheduled tasks

### Real-time Features
- `server/src/websocket/server.ts`: WebSocket server
- `server/src/websocket/handlers.ts`: Event handlers
- `client/src/hooks/useWebSocket.ts`: WebSocket hook

## 🚀 Application Architecture & Communication Flow

### Frontend-Backend Communication

```mermaid
sequenceDiagram
    participant C as Client
    participant A as API Client
    participant S as Server
    participant D as Database
    
    C->>A: Request
    A->>S: HTTP/WS
    S->>D: Query
    D-->>S: Response
    S-->>A: Data
    A-->>C: Update UI
```

```typescript
// client/src/hooks/useChatApi.ts
export function useChatApi() {
  const queryClient = useQueryClient();
  
  return {
    sendMessage: async (message: string) => {
      // Direct API call
      const response = await api.post('/chat/message', { message });
      
      // Update cache
      queryClient.invalidateQueries(['chat', 'messages']);
      
      return response.data;
    }
  };
}

// server/src/routes/chat.ts
router.post('/message', async (req, res) => {
  const { message } = req.body;
  
  // Process message
  const result = await chatService.processMessage(message);
  
  // Broadcast to WebSocket
  websocketServer.broadcast('chat.message', result);
  
  res.json(result);
});
```

### WebSocket Implementation

```mermaid
sequenceDiagram
    participant C as Client
    participant W as WebSocket
    participant S as Server
    participant R as Redis
    
    C->>W: Connect
    W->>S: Authenticate
    S->>R: Subscribe
    R-->>S: Events
    S-->>W: Broadcast
    W-->>C: Update
```

```typescript
// server/src/websocket/server.ts
export class WebSocketServer {
  private wss: WebSocket.Server;
  
  constructor() {
    this.wss = new WebSocket.Server({ port: 8080 });
    this.setupEventHandlers();
  }
  
  private setupEventHandlers() {
    this.wss.on('connection', (ws) => {
      ws.on('message', this.handleMessage.bind(this));
    });
  }
  
  broadcast(event: string, data: any) {
    this.wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify({ event, data }));
      }
    });
  }
}
```

### Service Integration

```mermaid
graph TD
    subgraph "Service Layer"
        A[Express Server] -->|Routes| B[Services]
        B -->|AI| C[OpenAI]
        B -->|Storage| D[S3]
        B -->|Cache| E[Redis]
        B -->|DB| F[PostgreSQL]
    end
```

```typescript
// server/src/services/ai.ts
export class AIService {
  private openai: OpenAI;
  
  constructor() {
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateResponse(prompt: string) {
    const completion = await this.openai.chat.completions.create({
      model: "gpt-4",
      messages: [{ role: "user", content: prompt }]
    });
    
    return completion.choices[0].message.content;
  }
}

// server/src/services/files.ts
export class FileService {
  private s3: S3Client;
  
  constructor() {
    this.s3 = new S3Client({
      region: process.env.AWS_REGION
    });
  }
  
  async uploadFile(file: Buffer, key: string) {
    await this.s3.send(new PutObjectCommand({
      Bucket: process.env.AWS_BUCKET,
      Key: key,
      Body: file
    }));
  }
}
```

## 🎯 Key Architectural Principles

1. **Direct Service Integration**
   - All external services integrated directly
   - Clear service boundaries
   - Type-safe implementations

2. **Real-time Features**
   - WebSocket for live updates
   - Redis for pub/sub
   - Optimistic UI updates

3. **Type Safety**
   - End-to-end TypeScript
   - Shared schema definitions
   - Runtime validation

4. **Development Playground**
   - Separate n8n environment
   - No production dependencies
   - Clear documentation

Would you like me to elaborate on any part of this structure or show more detailed implementation examples? 