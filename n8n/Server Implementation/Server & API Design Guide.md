# 🌐 Server & API Design Guide

This document clearly outlines REST API best practices, structured endpoint design, and strategies for optimizing Express middleware to achieve performance, security, and maintainability.

---

## 📌 REST API Best Practices

Follow these critical principles to ensure your API is robust, intuitive, and maintainable:

### ✅ Consistent Endpoint Naming

* Use clear, resource-based naming conventions.
* Prefer nouns over verbs, e.g. `/chats`, `/users`, `/files`.
* Use plural nouns for resources.

### Example:

```
GET    /users         → List all users
POST   /users         → Create a new user
GET    /users/:id     → Get a specific user
PATCH  /users/:id     → Update specific user
DELETE /users/:id     → Delete user
```

---

### ✅ Clear HTTP Methods & Status Codes

* **GET:** Retrieve data (200 OK, 404 Not Found).
* **POST:** Create data (201 Created).
* **PUT/PATCH:** Update data (200 OK, 204 No Content).
* **DELETE:** Remove data (200 OK, 204 No Content).
* **Error Codes:** Use clear, standard HTTP error codes (400, 401, 403, 404, 500).

---

### ✅ Pagination & Filtering Standards

* Use query parameters consistently (`?page=2&limit=10`).
* Support filtering clearly: (`?status=active&type=user`).

---

### ✅ Security & Authentication

* Enforce HTTPS for all endpoints.
* Clearly defined authentication via JWT and OAuth.
* Never expose sensitive data unnecessarily.

---

## 🌐 Endpoint Structure Document (Example)

### **Chats Endpoints**

```
GET    /chats                     → List all chats
POST   /chats                     → Create new chat
GET    /chats/:chatId             → Retrieve specific chat
PATCH  /chats/:chatId             → Update chat details
DELETE /chats/:chatId             → Delete chat
```

### **Chat Messages Endpoints**

```
GET    /chats/:chatId/messages         → List all messages for a chat
POST   /chats/:chatId/messages         → Create new message in chat
GET    /chats/:chatId/messages/:msgId  → Retrieve specific message
PATCH  /chats/:chatId/messages/:msgId  → Update specific message
DELETE /chats/:chatId/messages/:msgId  → Delete message
```

### **File Management Endpoints**

```
GET    /files                 → List files
POST   /files/upload          → Upload new file
GET    /files/:fileId         → Retrieve file metadata
DELETE /files/:fileId         → Delete file
```

---

## 📌 Detailed Endpoint Usage: Server vs. n8n

### **Chats and Chat Messages:**

* **Server:** Handles CRUD operations, secure authentication, and initial request validation.
* **n8n:** Automates AI-driven tasks triggered by chat interactions, such as slash-command parsing and message summarization.

### **File Management:**

* **Server:** Responsible for securely handling file uploads, direct database storage, and retrieval operations.
* **n8n:** Performs post-processing tasks like thumbnail generation, media optimization, and automated notifications on file updates.

This separation ensures robust security and efficient workflow automation, leveraging server-side control for security and n8n for automation flexibility.

---

## ⚙️ Express Middleware Optimization Guide

Middleware can significantly affect your API’s performance, security, and maintainability:

### ✅ Recommended Middleware Stack

```typescript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import compression from 'compression';
import rateLimit from 'express-rate-limit';

const app = express();

// Security Middlewares
app.use(helmet());
app.use(cors({
  origin: ['https://trusted-domain.com'],
}));
app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
}));

// Parsing Middlewares
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: true }));

// Compression Middleware (Improves performance)
app.use(compression());
```

---

### ✅ Middleware Best Practices

* **Security First:** Always prioritize security middleware (Helmet, CORS, Rate Limiter).
* **Minimal Parsing Overhead:** Use body-parsing middlewares judiciously, limit payload sizes.
* **Use Compression:** Reduce response payload sizes to optimize network usage.
* **Error Handling:** Consistently structured and informative error-handling middleware.

---

### ✅ Optimized Error Handling Example

```typescript
// Error handling middleware
app.use((err, req, res, next) => {
  const status = err.status || 500;
  res.status(status).json({
    status: 'error',
    message: err.message || 'Internal Server Error',
    details: process.env.NODE_ENV === 'development' ? err.stack : undefined,
  });
});
```

---

## 📗 Summary of API & Middleware Best Practices

* **Consistency:** Clear, standardized endpoint design.
* **Performance:** Use middleware optimally for speed and efficiency.
* **Security:** Robust middleware configuration and authentication.
* **Scalability & Maintainability:** Clean, clear, modular middleware and API design.

---

## 📌 Recommended Next Steps:

* Implement outlined API endpoint structure explicitly.
* Clearly document API contracts (OpenAPI or Swagger documentation).
* Test middleware configurations thoroughly (security, compression effectiveness).
* Create visual flow diagrams of middleware execution and endpoint handling.
