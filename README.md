
# TaskFlow Socket Service

# Real-time WebSocket Server

<div align="center">

<img src="https://api.dicebear.com/7.x/shapes/svg?seed=socket-io&backgroundColor=010101&size=120" alt="Socket Service Logo" width="120" height="120">

### A standalone Socket.io server for real-time task management

### Handles live updates, notifications, and collaborative features

---

## Deployed Service

### [https://taskflow-socket-production.up.railway.app](https://taskflow-socket-production.up.railway.app)

### Socket endpoint for TaskFlow application

---

**Main Application:** [https://taskflow-fawn-psi.vercel.app](https://taskflow-fawn-psi.vercel.app)

**Source Code:** [https://github.com/anshsharmacse/taskflow-socket](https://github.com/anshsharmacse/taskflow-socket)

---

![Socket.io](https://img.shields.io/badge/Socket.io-4-010101?style=for-the-badge&logo=socket.io)
![Node.js](https://img.shields.io/badge/Node.js-18-339933?style=for-the-badge&logo=node.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?style=for-the-badge&logo=typescript)
![Railway](https://img.shields.io/badge/Railway-Deployed-8B5CF6?style=for-the-badge&logo=railway)

</div>

---

## Overview

This microservice provides real-time WebSocket communication for the TaskFlow collaborative task management application. Built with Socket.io, it enables instant updates across all connected clients when tasks are created, updated, or deleted.

### Why a Separate Socket Service?

```mermaid
flowchart LR
    subgraph Problem
        P1[Vercel Serverless]
        P2[No Persistent Connections]
        P3[No WebSocket Support]
    end
    
    subgraph Solution
        S1[Separate Socket Service]
        S2[Persistent Connections]
        S3[Full WebSocket Support]
    end
    
    P1 --> P2
    P2 --> P3
    P3 --> S1
    S1 --> S2
    S2 --> S3
```

**Problem Explained:** Vercel's serverless architecture doesn't support persistent WebSocket connections. Each serverless function is ephemeral and terminates after request completion, making real-time communication impossible.

**Solution Explained:** A dedicated Socket.io server deployed on Railway maintains persistent connections with clients, enabling true real-time bidirectional communication.

---

## Architecture

### System Integration

```mermaid
graph TB
    subgraph Client Layer
        C1[Web Browser]
        C2[Mobile Browser]
    end
    
    subgraph Frontend - Vercel
        F1[Next.js App]
        F2[REST API]
    end
    
    subgraph Socket Service - Railway
        S1[Socket.io Server]
        S2[Connection Manager]
        S3[Event Handlers]
    end
    
    subgraph Data Layer
        D1[(PostgreSQL)]
    end
    
    C1 --> F1
    C2 --> F1
    F1 --> F2
    F2 --> D1
    C1 -.-> S1
    C2 -.-> S1
    S1 --> S2
    S2 --> S3
```

**Integration Explained:** The socket service runs independently from the main Next.js application. Clients connect to both the REST API for CRUD operations and the Socket server for real-time updates simultaneously.

### Connection Flow

```mermaid
sequenceDiagram
    participant Client
    participant NextJS as Next.js App
    participant Socket as Socket Server
    participant OtherClients
    
    Client->>NextJS: HTTP Request
    NextJS-->>Client: Response
    Client->>Socket: WebSocket Connect
    Socket-->>Client: Connection Ack
    Client->>Socket: Authenticate
    Socket->>Socket: Join Rooms
    Socket-->>Client: Auth Confirmed
    
    Note over Client,OtherClients: Real-time Event
    Client->>NextJS: Create Task
    NextJS-->>Client: Task Created
    Client->>Socket: Emit Event
    Socket->>OtherClients: Broadcast Update
```

**Flow Explained:** Clients establish dual connections - HTTP for API calls and WebSocket for real-time updates. After authentication, users join specific rooms to receive targeted notifications.

---

## Event System

### Event Types

| Event | Direction | Description |
|-------|-----------|-------------|
| `connection` | Client to Server | Initial WebSocket connection |
| `authenticate` | Client to Server | Send user credentials |
| `task:created` | Client to Server | New task created notification |
| `task:updated` | Client to Server | Task modified notification |
| `task:deleted` | Client to Server | Task removed notification |
| `task:assigned` | Server to Client | Task assignment notification |

### Event Flow Diagram

```mermaid
flowchart TD
    subgraph Client Events
        C1[task:created]
        C2[task:updated]
        C3[task:deleted]
    end
    
    subgraph Server Processing
        S1[Validate Event]
        S2[Find Target Rooms]
        S3[Broadcast Event]
    end
    
    subgraph Client Notifications
        N1[task:assigned]
        N2[UI Update]
    end
    
    C1 --> S1
    C2 --> S1
    C3 --> S1
    S1 --> S2
    S2 --> S3
    S3 --> N1
    N1 --> N2
```

**Event Processing Explained:** Each event goes through validation, room identification, and broadcasting. The server determines which clients should receive the notification based on task ownership and assignment.

---

## Room System

### Room Types

```mermaid
flowchart LR
    subgraph User Rooms
        R1[Room: user/userId]
        R2[Room: email/userEmail]
    end
    
    subgraph Purpose
        P1[Personal Notifications]
        P2[Task Assignment by Email]
    end
    
    R1 --> P1
    R2 --> P2
```

**Room System Explained:** Each user joins two rooms upon authentication. The user ID room handles personal notifications, while the email room enables task assignment before user registration.

### Room Joining Logic

```mermaid
sequenceDiagram
    participant Client
    participant Server
    
    Client->>Server: authenticate userId email
    Server->>Server: Validate credentials
    Server->>Server: Join room user/userId
    Server->>Server: Join room email/userEmail
    Server-->>Client: authenticated success
```

**Join Sequence Explained:** After successful authentication, the server automatically joins the client to both their user ID room and email room. This ensures they receive all relevant notifications.

---

## Project Structure

```
taskflow-socket/
├── src/
│   └── index.ts          # Main server entry point
├── package.json          # Dependencies and scripts
├── tsconfig.json         # TypeScript configuration
└── README.md             # Documentation
```

### Key Files

| File | Purpose |
|------|---------|
| `src/index.ts` | Socket.io server initialization and event handlers |
| `package.json` | Project dependencies and run scripts |
| `tsconfig.json` | TypeScript compiler settings |

---

## API Reference

### Server Configuration

```typescript
const io = new Server(httpServer, {
  cors: {
    origin: process.env.CORS_ORIGIN,
    methods: ["GET", "POST"],
    credentials: true,
  },
});
```

### Authentication Event

```typescript
// Client emits
socket.emit("authenticate", {
  userId: "user_id",
  email: "user@example.com"
});

// Server responds
socket.emit("authenticated", { success: true });
```

### Task Events

```typescript
// Task created
socket.emit("task:created", {
  task: { id, title, assigneeEmail }
});

// Task updated
socket.emit("task:updated", {
  taskId: "task_id",
  updates: { status, priority }
});

// Task deleted
socket.emit("task:deleted", {
  taskId: "task_id"
});
```

---

## Deployment

### Railway Deployment Steps

```mermaid
flowchart TD
    A[Create GitHub Repo] --> B[Connect to Railway]
    B --> C[Configure Environment]
    C --> D[Set PORT variable]
    D --> E[Set CORS_ORIGIN variable]
    E --> F[Deploy Service]
    F --> G[Get Railway URL]
    G --> H[Update Main App Config]
```

**Deployment Process Explained:** The service is deployed to Railway which provides persistent WebSocket support. After deployment, update the main application with the socket URL.

### Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `PORT` | Yes | Server listening port | `3003` |
| `CORS_ORIGIN` | Yes | Allowed frontend origin | `https://your-app.vercel.app` |

---

## Local Development

### Prerequisites

| Requirement | Version | Purpose |
|-------------|---------|---------|
| Node.js or Bun | 18+ / Latest | JavaScript runtime |
| TypeScript | 5.x | Type checking |

### Quick Start

```bash
# Clone the repository
git clone https://github.com/anshsharmacse/taskflow-socket.git
cd taskflow-socket

# Install dependencies
bun install

# Create environment file
echo "PORT=3003" > .env
echo "CORS_ORIGIN=http://localhost:3000" >> .env

# Start development server
bun run dev
```

### Development Commands

| Command | Description |
|---------|-------------|
| `bun run dev` | Start development server with hot reload |
| `bun run build` | Build TypeScript to JavaScript |
| `bun run start` | Start production server |

---

## Monitoring

### Health Check

```mermaid
flowchart LR
    A[Health Request] --> B[GET /health]
    B --> C{Server Running?}
    C -->|Yes| D[Return 200 OK]
    C -->|No| E[Connection Failed]
```

**Health Monitoring Explained:** Railway provides automatic health checks. A simple endpoint confirms the server is running and accepting connections.

### Logging

The server logs important events:
- New client connections
- Authentication attempts
- Room joins
- Event broadcasts
- Disconnections

---

## Security

### CORS Configuration

```mermaid
flowchart TD
    A[Client Request] --> B{Origin Allowed?}
    B -->|Yes| C[Accept Connection]
    B -->|No| D[Reject with CORS Error]
    C --> E[Establish WebSocket]
```

**Security Explained:** CORS ensures only the configured frontend origin can establish WebSocket connections, preventing unauthorized access.

### Authentication Flow

1. Client connects via WebSocket
2. Client emits `authenticate` event with credentials
3. Server validates and joins rooms
4. Unauthenticated clients receive no broadcasts

---

## Performance

### Connection Pooling Benefits

```mermaid
flowchart TD
    subgraph HTTP Polling
        H1[Multiple Requests]
        H2[High Overhead]
        H3[Increased Latency]
    end
    
    subgraph WebSocket
        W1[Single Connection]
        W2[Low Overhead]
        W3[Minimal Latency]
    end
    
    H1 --> H2 --> H3
    W1 --> W2 --> W3
```

**Performance Benefits Explained:** A single WebSocket connection multiplexes all events, reducing connection overhead compared to multiple HTTP requests. This results in lower latency and more efficient bandwidth usage.

---

## Troubleshooting

### Common Issues

```mermaid
flowchart TD
    A[Connection Issue] --> B{Error Type?}
    B -->|CORS| C[Check CORS_ORIGIN env var]
    B -->|Timeout| D[Check server health]
    B -->|Auth Failed| E[Verify credentials format]
    B -->|No Events| F[Check room membership]
    
    C --> G[Update env and restart]
    D --> H[Check Railway logs]
    E --> I[Verify userId and email]
    F --> J[Debug room joining logic]
```

**Troubleshooting Guide Explained:** Most issues fall into four categories. CORS errors require environment variable fixes. Timeouts indicate server problems. Auth failures need credential verification. Missing events suggest room membership issues.

---

## Developer Information

<div align="center">

### **Ansh Sharma**

**National Institute of Technology Calicut**

---

[![GitHub](https://img.shields.io/badge/GitHub-anshsharmacse-black?style=for-the-badge&logo=github)](https://github.com/anshsharmacse)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Ansh_Sharma-blue?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/anshsharmacse/)

</div>

---

## License

This project is released under the MIT License.

---

<div align="center">

**Built with Socket.io and deployed on Railway**

</div>
