# 💻 Frontend Responsibilities Guide

This guide details frontend responsibilities, outlining primary files, interactions, state management, and integration points with our backend services.

## 🔄 Frontend Architecture Overview

```mermaid
graph TD
    subgraph "Frontend Architecture"
        A[React Application] -->|Uses| B[Component Library]
        A -->|Uses| C[State Management]
        A -->|Uses| D[API Integration]
        A -->|Uses| E[WebSocket]
        B -->|Provides| F[UI Components]
        C -->|Manages| G[Application State]
        D -->|Handles| H[Server Communication]
        E -->|Handles| I[Real-time Updates]
    end
```

## ✅ Frontend Responsibilities Overview

Our frontend (built with **Vite + React + TypeScript + TailwindCSS**) manages the following critical tasks:

* **User Interface & Interaction** (chat UI, slash commands, navigation)
* **State & Data Management** (React-Query caching, global state)
* **API Integration** (via REST API endpoints & hooks)
* **Authentication Management** (JWT & OAuth via hooks)
* **Real-time Updates Handling** (WebSockets integration)
* **File Upload Interface & Management** (frontend upload logic)
* **Error Handling & User Feedback** (toast notifications, validations)

## 📂 Core Frontend Structure

```mermaid
graph TD
    subgraph "Directory Structure"
        A[client/src] -->|Contains| B[components]
        A -->|Contains| C[hooks]
        A -->|Contains| D[lib]
        A -->|Contains| E[contexts]
        A -->|Contains| F[pages]
        A -->|Contains| G[types]
        
        B -->|Contains| H[layout]
        B -->|Contains| I[chat]
        B -->|Contains| J[files]
        B -->|Contains| K[common]
        
        C -->|Contains| L[useChat]
        C -->|Contains| M[useAuth]
        C -->|Contains| N[useFiles]
        C -->|Contains| O[useWebSocket]
    end
```

```
client/
├── src/
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── MainLayout.tsx
│   │   ├── chat/
│   │   │   ├── ChatWindow.tsx
│   │   │   ├── MessageList.tsx
│   │   │   ├── MessageItem.tsx
│   │   │   ├── MessageInput.tsx
│   │   │   └── ActionButtons.tsx
│   │   ├── files/
│   │   │   ├── FileUpload.tsx
│   │   │   ├── FileList.tsx
│   │   │   └── FilePreview.tsx
│   │   └── common/
│   │       ├── Button.tsx
│   │       ├── Input.tsx
│   │       └── Toast.tsx
│   ├── hooks/
│   │   ├── useChat.ts
│   │   ├── useAuth.ts
│   │   ├── useFiles.ts
│   │   └── useWebSocket.ts
│   ├── lib/
│   │   ├── api.ts
│   │   ├── queryClient.ts
│   │   └── websocket.ts
│   ├── contexts/
│   │   ├── AuthContext.tsx
│   │   └── ToastContext.tsx
│   ├── pages/
│   │   ├── chat/
│   │   │   ├── index.tsx
│   │   │   └── [id].tsx
│   │   ├── files/
│   │   │   └── index.tsx
│   │   └── auth/
│   │       ├── login.tsx
│   │       └── register.tsx
│   └── types/
│       ├── chat.ts
│       ├── file.ts
│       └── user.ts
```

## ⚙️ Detailed Frontend Responsibilities

```mermaid
graph TD
    subgraph "Responsibility Flow"
        A[User Interface] -->|Handles| B[User Interactions]
        C[State Management] -->|Manages| D[Application State]
        E[API Integration] -->|Communicates| F[Backend Services]
        G[Authentication] -->|Secures| H[User Sessions]
        I[Real-time Updates] -->|Synchronizes| J[Live Data]
        K[File Management] -->|Handles| L[File Operations]
        M[Error Handling] -->|Manages| N[Error States]
    end
```

| Responsibility                         | Implementation Details                                    | Key Components                                    |
| -------------------------------------- | --------------------------------------------------------- | ------------------------------------------------- |
| **User Interface & Interaction**       | Chat UI, navigation, slash-commands                       | `ChatWindow.tsx`, `MessageInput.tsx`              |
| **State & Data Management**            | React-Query caching, global state                         | `queryClient.ts`, custom hooks                    |
| **API Integration**                    | REST API communication                                    | `api.ts`, `useChat.ts`                           |
| **Authentication**                     | JWT & OAuth management                                    | `AuthContext.tsx`, `useAuth.ts`                   |
| **Real-time Updates**                  | WebSocket integration                                     | `useWebSocket.ts`, `websocket.ts`                 |
| **File Management**                    | Upload, preview, organization                             | `FileUpload.tsx`, `FileList.tsx`                  |
| **Error Handling**                     | Toast notifications, error boundaries                     | `Toast.tsx`, `ErrorBoundary.tsx`                  |

## 🔄 Data Flow Architecture

### ✅ Component to Service Flow

```mermaid
sequenceDiagram
    participant C as Component
    participant H as Hook
    participant A as API Client
    participant S as Server
    
    C->>H: Request Data
    H->>A: API Call
    A->>S: HTTP Request
    S-->>A: Response
    A-->>H: Process Data
    H-->>C: Update UI
```

### ✅ Real-time Update Flow

```mermaid
sequenceDiagram
    participant S as Server
    participant W as WebSocket
    participant H as Hook
    participant C as Component
    participant Q as Query Cache
    
    S->>W: Broadcast Event
    W->>H: Receive Event
    H->>Q: Update Cache
    Q->>C: Trigger Re-render
    C->>C: Update UI
```

## 📌 Key Implementation Patterns

### ✅ Component Pattern

```mermaid
graph TD
    subgraph "Component Pattern"
        A[Feature Component] -->|Uses| B[Custom Hooks]
        A -->|Uses| C[UI Components]
        B -->|Manages| D[State]
        B -->|Handles| E[API Calls]
        B -->|Manages| F[WebSocket]
        C -->|Renders| G[UI Elements]
    end
```

```typescript
// Example of a feature component
export function ChatFeature() {
  // Data fetching
  const { data, isLoading } = useQuery(...);
  
  // Real-time updates
  const socket = useWebSocket();
  
  // User interactions
  const { mutate } = useMutation(...);
  
  // Error handling
  const { toast } = useToast();
  
  return (
    <ErrorBoundary>
      {isLoading ? (
        <LoadingSpinner />
      ) : (
        <div className="space-y-4">
          <ChatList data={data} />
          <ChatInput onSubmit={mutate} />
        </div>
      )}
    </ErrorBoundary>
  );
}
```

### ✅ Hook Pattern

```mermaid
graph TD
    subgraph "Hook Pattern"
        A[Custom Hook] -->|Uses| B[React Query]
        A -->|Uses| C[Context]
        B -->|Manages| D[Server State]
        C -->|Provides| E[Global State]
        A -->|Returns| F[State & Methods]
    end
```

```typescript
// Example of a custom hook
export function useChatFeature() {
  const queryClient = useQueryClient();
  const { toast } = useToast();
  
  const query = useQuery({
    queryKey: ['chat'],
    queryFn: () => api.getChat(),
  });
  
  const mutation = useMutation({
    mutationFn: (data) => api.updateChat(data),
    onSuccess: () => {
      queryClient.invalidateQueries(['chat']);
      toast.success('Chat updated');
    },
  });
  
  return {
    data: query.data,
    isLoading: query.isLoading,
    update: mutation.mutate,
  };
}
```

## 🎨 UI Component Library

### ✅ Base Components

```mermaid
graph TD
    subgraph "Component Library"
        A[Base Components] -->|Extends| B[HTML Elements]
        A -->|Uses| C[TailwindCSS]
        A -->|Provides| D[Styled Components]
        D -->|Used By| E[Feature Components]
        D -->|Used By| F[Page Components]
    end
```

```typescript
// Example of a reusable button component
export function Button({
  variant = 'primary',
  size = 'md',
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      className={cn(
        'rounded-lg font-medium transition-colors',
        {
          'bg-blue-500 text-white hover:bg-blue-600': variant === 'primary',
          'bg-gray-100 text-gray-900 hover:bg-gray-200': variant === 'secondary',
        },
        {
          'px-4 py-2 text-sm': size === 'sm',
          'px-6 py-3 text-base': size === 'md',
          'px-8 py-4 text-lg': size === 'lg',
        }
      )}
      {...props}
    >
      {children}
    </button>
  );
}
```

## 🔒 Security Considerations

### ✅ Authentication Flow

```mermaid
sequenceDiagram
    participant U as User
    participant C as Component
    participant A as Auth Hook
    participant S as Server
    
    U->>C: Login Request
    C->>A: Submit Credentials
    A->>S: Authenticate
    S-->>A: JWT Token
    A->>A: Store Token
    A-->>C: Update Auth State
    C-->>U: Redirect to App
```

```typescript
// Protected route wrapper
export function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, isLoading } = useAuth();
  const router = useRouter();

  useEffect(() => {
    if (!isLoading && !user) {
      router.push('/login');
    }
  }, [user, isLoading, router]);

  if (isLoading) {
    return <LoadingSpinner />;
  }

  return user ? <>{children}</> : null;
}
```

## 📱 Responsive Design

### ✅ TailwindCSS Breakpoints

```mermaid
graph TD
    subgraph "Responsive Design"
        A[Component] -->|Uses| B[Tailwind Classes]
        B -->|Applies| C[Base Styles]
        B -->|Applies| D[Responsive Styles]
        D -->|Mobile| E[sm:]
        D -->|Tablet| F[md:]
        D -->|Desktop| G[lg:]
        D -->|Large| H[xl:]
    end
```

```typescript
// Example of responsive component
export function ResponsiveLayout() {
  return (
    <div className="
      grid
      grid-cols-1
      md:grid-cols-2
      lg:grid-cols-3
      gap-4
      p-4
    ">
      <Sidebar className="hidden md:block" />
      <MainContent />
      <ChatWindow className="hidden lg:block" />
    </div>
  );
}
```

## 🎮 Development Playground

```mermaid
graph TD
    subgraph "Development Process"
        A[Feature Branch] -->|Creates| B[Prototype]
        B -->|Tests| C[New Feature]
        C -->|Documents| D[Results]
        D -->|If Successful| E[Migration Plan]
        E -->|Implements| F[Production]
    end
```

```typescript
// dev-playground/README.md
# Frontend Development Playground

This directory contains experimental frontend features and prototypes.

## Structure
- `/components` - New UI components
- `/hooks` - Custom hook experiments
- `/pages` - Page prototypes
- `/styles` - Style experiments

## Usage
1. Create feature branch
2. Implement prototype
3. Document results
4. Create migration plan if successful
```

## 🚀 Next Steps

1. **Implementation**
   - Set up component library
   - Configure state management
   - Implement real-time features

2. **Testing**
   - Unit tests for components
   - Integration tests for hooks
   - E2E testing with Cypress

3. **Documentation**
   - Component documentation
   - Hook usage examples
   - Development playground guides

Would you like me to elaborate on any part of this structure or show more detailed implementation examples? 