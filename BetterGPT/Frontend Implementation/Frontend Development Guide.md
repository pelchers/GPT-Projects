# 💻 Frontend Development Guide

This guide provides comprehensive best practices and structural guidelines for building robust, performant, and maintainable frontend components in our React + TypeScript + TailwindCSS application.

## 🔄 Component Architecture Overview

```mermaid
graph TD
    subgraph "Component Architecture"
        A[Page Component] -->|Composes| B[Feature Components]
        B -->|Uses| C[UI Components]
        B -->|Uses| D[Custom Hooks]
        D -->|Manages| E[State]
        D -->|Handles| F[API Calls]
        D -->|Manages| G[WebSocket]
    end
```

---

## 📌 Component Architecture

### ✅ Core Component Structure

```mermaid
graph TD
    subgraph "Chat Component Structure"
        A[ChatWindow] -->|Contains| B[MessageList]
        A -->|Contains| C[MessageInput]
        A -->|Contains| D[ActionButtons]
        B -->|Renders| E[MessageItem]
        E -->|Shows| F[Avatar]
        E -->|Shows| G[MessageContent]
    end
```

```typescript
// client/src/components/chat/ChatWindow.tsx
import { useChat } from '@/hooks/useChat';
import { MessageList } from './MessageList';
import { MessageInput } from './MessageInput';
import { ActionButtons } from './ActionButtons';

export function ChatWindow() {
  const { messages, sendMessage, isLoading } = useChat();
  
  return (
    <div className="flex flex-col h-full">
      <MessageList messages={messages} />
      <div className="border-t p-4">
        <MessageInput onSend={sendMessage} />
        <ActionButtons />
      </div>
    </div>
  );
}

// client/src/components/chat/MessageList.tsx
export function MessageList({ messages }: MessageListProps) {
  return (
    <div className="flex-1 overflow-y-auto p-4 space-y-4">
      {messages.map((message) => (
        <MessageItem key={message.id} message={message} />
      ))}
    </div>
  );
}

// client/src/components/chat/MessageItem.tsx
export function MessageItem({ message }: MessageItemProps) {
  return (
    <div className="flex items-start space-x-3">
      <Avatar user={message.user} />
      <div className="flex-1">
        <div className="flex items-center space-x-2">
          <span className="font-medium">{message.user.name}</span>
          <span className="text-sm text-gray-500">
            {formatDate(message.createdAt)}
          </span>
        </div>
        <div className="mt-1 prose">
          <MessageContent content={message.content} />
        </div>
      </div>
    </div>
  );
}
```

---

## 📌 State Management with React Query

### ✅ Query Client Configuration

```mermaid
graph TD
    subgraph "State Management Flow"
        A[React Component] -->|Uses| B[useQuery Hook]
        B -->|Fetches| C[API]
        B -->|Caches| D[React Query Cache]
        E[useMutation Hook] -->|Updates| C
        E -->|Invalidates| D
        D -->|Updates| A
    end
```

```typescript
// client/src/lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      cacheTime: 1000 * 60 * 30, // 30 minutes
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
  },
});

// client/src/hooks/useChat.ts
export function useChat(chatId: string) {
  const { data: messages, isLoading } = useQuery({
    queryKey: ['messages', chatId],
    queryFn: () => api.getMessages(chatId),
  });

  const { mutate: sendMessage } = useMutation({
    mutationFn: (content: string) => api.sendMessage(chatId, content),
    onSuccess: () => {
      queryClient.invalidateQueries(['messages', chatId]);
    },
  });

  return { messages, isLoading, sendMessage };
}
```

---

## 📌 Real-time Updates with WebSocket

### ✅ WebSocket Integration

```mermaid
sequenceDiagram
    participant C as Component
    participant H as useWebSocket Hook
    participant W as WebSocket
    participant S as Server
    participant Q as Query Cache
    
    C->>H: Initialize
    H->>W: Connect
    W->>S: Authenticate
    S-->>W: Connected
    W-->>H: Connection Established
    H-->>C: Socket Ready
    
    Note over C,Q: Real-time Updates
    S->>W: Broadcast Event
    W-->>H: Receive Event
    H->>Q: Update Cache
    Q-->>C: Re-render
```

```typescript
// client/src/hooks/useWebSocket.ts
export function useWebSocket() {
  const [socket, setSocket] = useState<WebSocket | null>(null);

  useEffect(() => {
    const ws = new WebSocket(WS_URL);
    
    ws.onmessage = (event) => {
      const { type, data } = JSON.parse(event.data);
      switch (type) {
        case 'message:created':
          queryClient.setQueryData(
            ['messages', data.chatId],
            (old: Message[]) => [...old, data]
          );
          break;
        case 'chat:updated':
          queryClient.invalidateQueries(['chats']);
          break;
      }
    };

    setSocket(ws);
    return () => ws.close();
  }, []);

  return socket;
}

// client/src/components/chat/ChatWindow.tsx
export function ChatWindow() {
  const socket = useWebSocket();
  // ... rest of the component
}
```

---

## 📌 File Upload Handling

### ✅ File Upload Component

```mermaid
graph TD
    subgraph "File Upload Flow"
        A[FileUpload Component] -->|Handles| B[Drop Event]
        A -->|Handles| C[Select Event]
        B -->|Processes| D[File List]
        C -->|Processes| D
        D -->|Uploads| E[API]
        E -->|Updates| F[Query Cache]
        F -->|Notifies| G[UI]
    end
```

```typescript
// client/src/components/chat/FileUpload.tsx
export function FileUpload() {
  const { mutate: uploadFile, isLoading } = useMutation({
    mutationFn: (file: File) => {
      const formData = new FormData();
      formData.append('file', file);
      return api.uploadFile(formData);
    },
    onSuccess: (data) => {
      queryClient.invalidateQueries(['files']);
      toast.success('File uploaded successfully');
    },
  });

  const handleDrop = useCallback((files: File[]) => {
    files.forEach(uploadFile);
  }, [uploadFile]);

  return (
    <div
      className="border-2 border-dashed rounded-lg p-4"
      onDrop={(e) => {
        e.preventDefault();
        handleDrop(Array.from(e.dataTransfer.files));
      }}
    >
      <input
        type="file"
        onChange={(e) => e.target.files && handleDrop(Array.from(e.target.files))}
        className="hidden"
      />
      <div className="text-center">
        <p>Drop files here or click to upload</p>
      </div>
    </div>
  );
}
```

---

## 📌 Authentication Flow

### ✅ Auth Context and Hooks

```mermaid
sequenceDiagram
    participant U as User
    participant A as AuthProvider
    participant L as Login Form
    participant S as Server
    participant C as Context
    
    U->>L: Enter Credentials
    L->>S: Login Request
    S-->>L: Token + User
    L->>A: Update Context
    A->>C: Store User
    C-->>A: Context Updated
    A-->>U: Redirect to App
```

```typescript
// client/src/contexts/AuthContext.tsx
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const user = await api.getCurrentUser();
      setUser(user);
    } catch {
      setUser(null);
    } finally {
      setIsLoading(false);
    }
  };

  const login = async (credentials: Credentials) => {
    const { user, token } = await api.login(credentials);
    localStorage.setItem('token', token);
    setUser(user);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, isLoading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// client/src/hooks/useAuth.ts
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

---

## 📌 Error Handling

### ✅ Error Boundaries and Toast Notifications

```mermaid
graph TD
    subgraph "Error Handling Flow"
        A[Error Boundary] -->|Catches| B[Component Error]
        A -->|Shows| C[Fallback UI]
        D[Toast System] -->|Shows| E[Success Toast]
        D -->|Shows| F[Error Toast]
        D -->|Shows| G[Info Toast]
        H[API Error] -->|Handles| I[Error Handler]
        I -->|Shows| D
    end
```

```typescript
// client/src/components/ErrorBoundary.tsx
export class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean }
> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="p-4 bg-red-50 text-red-700 rounded-lg">
          <h2 className="text-lg font-semibold">Something went wrong</h2>
          <button
            onClick={() => this.setState({ hasError: false })}
            className="mt-2 text-sm underline"
          >
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// client/src/components/ToastNotifications.tsx
export function ToastNotifications() {
  const { toasts, removeToast } = useToast();

  return (
    <div className="fixed bottom-4 right-4 space-y-2">
      {toasts.map((toast) => (
        <div
          key={toast.id}
          className={`p-4 rounded-lg shadow-lg ${
            toast.type === 'error' ? 'bg-red-500' : 'bg-green-500'
          } text-white`}
        >
          {toast.message}
          <button
            onClick={() => removeToast(toast.id)}
            className="ml-2 text-sm underline"
          >
            Dismiss
          </button>
        </div>
      ))}
    </div>
  );
}
```

---

## 📌 API Integration

### ✅ API Client Setup

```mermaid
graph TD
    subgraph "API Integration"
        A[API Client] -->|Configures| B[Axios Instance]
        B -->|Adds| C[Auth Interceptor]
        B -->|Adds| D[Error Interceptor]
        E[API Methods] -->|Uses| B
        E -->|Returns| F[Typed Responses]
        G[Custom Hooks] -->|Uses| E
    end
```

```typescript
// client/src/lib/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorized
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export const chatApi = {
  getMessages: (chatId: string) => api.get(`/chats/${chatId}/messages`),
  sendMessage: (chatId: string, content: string) =>
    api.post(`/chats/${chatId}/messages`, { content }),
  uploadFile: (formData: FormData) =>
    api.post('/files/upload', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    }),
};
```

---

## 📌 Development Playground Integration

```mermaid
graph TD
    subgraph "Development Flow"
        A[Feature Branch] -->|Creates| B[Prototype]
        B -->|Tests| C[New Feature]
        C -->|Documents| D[Results]
        D -->|If Successful| E[Migration Plan]
        E -->|Implements| F[Production]
    end
```

```typescript
// dev-playground/components/NewFeature.tsx
export function NewFeature() {
  // Test implementation
  return (
    <div className="p-4 border rounded-lg">
      <h2>New Feature Prototype</h2>
      {/* Implementation */}
    </div>
  );
}
```

---

## 🚀 Next Steps

1. **Implementation**
   - Set up component structure
   - Configure React Query
   - Implement WebSocket integration

2. **Testing**
   - Unit tests for components
   - Integration tests for hooks
   - E2E testing with Cypress

3. **Documentation**
   - Component documentation
   - Hook usage examples
   - Development playground guides

Would you like me to elaborate on any part of this structure or show more detailed implementation examples? 