```markdown
# TaskFlow Socket Service

# Real-time WebSocket Server

<div align="center">

<img src="<img width="1397" height="645" alt="image" src="https://github.com/user-attachments/assets/4da48703-152e-4be3-9640-2cc13218e465" />
" width="120" height="120">

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
    
    P1 --> P2 --> P3
    P3 --> S1
    S1 --> S2 --> S3
```

**Problem Explained:** Vercel's serverless architecture doesn't support persistent WebSocket connections. Each serverless function is ephemeral and terminates after request completion, making real-time communication impossible.

**Solution Explained:** A dedicated Socket.io server deployed on Railway maintains persistent connections with clients, enabling true real-time bidirectional communication.

---

## Architecture

### System Integration

```mermaid
graph TB
    subgraph "Client Layer"
        C1[Web Browser]
        C2[Mobile Browser]
    end
    
    subgraph "Frontend - Vercel"
        F1[Next.js App]
        F2[REST API]
    end
    
    subgraph "Socket Service - Railway"
        S1[Socket.io Server]
        S2[Connection Manager]
        S3[Event Handlers]
    end
    
    subgraph "Data Layer"
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

**Integration Explained:** The socket service runs independently from the main Next.js application. Clients connect to both the REST API (for CRUD operations) and the Socket server (for real-time updates) simultaneously.

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

**Flow Explained:** Clients establish dual connections - HTTP for API calls and WebSocket for real-time updates. After authentication, clients join specific rooms based on their user ID and email for targeted notifications.

---

## Features

| Feature | Description |
|---------|-------------|
| **Real-time Broadcasting** | Instant task updates to all relevant users |
| **Room-based Routing** | Targeted notifications using user rooms |
| **Auto-reconnection** | Automatic reconnection on connection drop |
| **CORS Protection** | Secure cross-origin requests |
| **Graceful Fallback** | HTTP long-polling fallback if WebSocket fails |

### Event System

```mermaid
flowchart TD
    subgraph "Client to Server Events"
        C1[task:created]
        C2[task:updated]
        C3[task:deleted]
        C4[authenticate]
    end
    
    subgraph "Server Processing"
        S1[Validate Event]
        S2[Find Target Rooms]
        S3[Broadcast]
    end
    
    subgraph "Server to Client Events"
        R1[task:assigned]
        R2[task:updated:broadcast]
        R3[task:deleted:broadcast]
    end
    
    C1 --> S1
    C2 --> S1
    C3 --> S1
    C4 --> S1
    S1 --> S2 --> S3
    S3 --> R1
    S3 --> R2
    S3 --> R3
```

**Event Flow Explained:** Clients emit events for actions they perform. The server validates each event, determines which rooms should receive the notification, and broadcasts to all relevant clients.

---

## Room Strategy

### Room Types

```mermaid
flowchart LR
    subgraph "User Rooms"
        R1[Room: user:userId]
        R2[Room: email:userEmail]
    end
    
    subgraph "Purpose"
        P1[Direct user notifications]
        P2[Email-based task assignment]
    end
    
    R1 --> P1
    R2 --> P2
```

**Room Strategy Explained:** Each user joins two rooms upon connection. The user ID room enables direct notifications. The email room ensures users receive tasks assigned to their email before they registered.

### Broadcasting Logic

```mermaid
flowchart TD
    A[Task Event Received] --> B{Has Assignee?}
    B -->|Yes| C[Get Assignee Email]
    B -->|No| D[Skip Notification]
    C --> E{Assignee Exists?}
    E -->|Yes| F[Get User ID]
    E -->|No| G[Use Email Room]
    F --> H[Join User Room]
    G --> H
    H --> I[Broadcast to Room]
```

**Broadcasting Explained:** When a task is assigned, the server determines the target room. If the assignee exists, it uses their user ID room. If not, it uses the email room so the task is waiting when they sign up.

---

## API Reference

### Connection

```javascript
// Connect to socket server
const socket = io('https://taskflow-socket-production.up.railway.app', {
  transports: ['websocket', 'polling'],
  withCredentials: true
});
```

### Events

#### Client to Server

| Event | Payload | Description |
|-------|---------|-------------|
| `authenticate` | `{ userId, email }` | Authenticate and join rooms |
| `task:created` | `{ task }` | Notify new task creation |
| `task:updated` | `{ taskId, updates }` | Notify task update |
| `task:deleted` | `{ taskId, assigneeEmail }` | Notify task deletion |

#### Server to Client

| Event | Payload | Description |
|-------|---------|-------------|
| `authenticated` | `{ success, rooms }` | Authentication confirmation |
| `task:assigned` | `{ task }` | New task assigned to user |
| `task:updated:broadcast` | `{ taskId, updates }` | Task update notification |
| `task:deleted:broadcast` | `{ taskId }` | Task deletion notification |

### Usage Example

```javascript
// Client-side implementation
import { io } from 'socket.io-client';

const socket = io(SOCKET_URL, {
  transports: ['websocket', 'polling']
});

// Authenticate on connection
socket.on('connect', () => {
  socket.emit('authenticate', {
    userId: user.id,
    email: user.email
  });
});

// Listen for task assignments
socket.on('task:assigned', (data) => {
  // Add new task to UI
  addTaskToStore(data.task);
});

// Notify on task creation
const createTask = async (task) => {
  const response = await fetch('/api/tasks', {
    method: 'POST',
    body: JSON.stringify(task)
  });
  
  const newTask = await response.json();
  socket.emit('task:created', { task: newTask });
};
```

---

## Deployment

### Railway Deployment

```mermaid
flowchart TD
    subgraph "Prerequisites"
        P1[GitHub Repository]
        P2[Railway Account]
        P3[Environment Variables]
    end
    
    subgraph "Deployment Steps"
        D1[Create New Project]
        D2[Connect Repository]
        D3[Configure Variables]
        D4[Deploy]
    end
    
    subgraph "Post-Deployment"
        PD1[Get Service URL]
        PD2[Update Frontend Env]
        PD3[Test Connection]
    end
    
    P1 --> D1
    P2 --> D1
    P3 --> D3
    D1 --> D2 --> D3 --> D4
    D4 --> PD1 --> PD2 --> PD3
```

**Deployment Process Explained:** Deploy to Railway by connecting your GitHub repository, setting environment variables, and letting Railway handle the build and deployment automatically.

### Environment Variables

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `PORT` | Yes | Server port | `3003` |
| `CORS_ORIGIN` | Yes | Allowed frontend origin | `https://taskflow-fawn-psi.vercel.app` |

### Railway Configuration

```toml
# railway.toml
[build]
builder = "nixpacks"

[deploy]
startCommand = "bun run start"
healthcheckPath = "/health"
healthcheckTimeout = 100
restartPolicyType = "on_failure"
restartPolicyMaxRetries = 3
```

---

## Local Development

### Prerequisites

| Requirement | Version |
|-------------|---------|
| Node.js | 18.x+ |
| Bun | Latest |

### Quick Start

```bash
# Clone repository
git clone https://github.com/anshsharmacse/taskflow-socket.git
cd taskflow-socket

# Install dependencies
bun install

# Create environment file
cp .env.example .env

# Configure variables
# PORT=3003
# CORS_ORIGIN=http://localhost:3000

# Start development server
bun run dev
```

### Project Structure

```
taskflow-socket/
├── src/
│   └── index.ts          # Main server entry
├── package.json
├── tsconfig.json
├── .env.example
└── README.md
```

---

## Monitoring

### Health Check Endpoint

```javascript
// GET /health
{
  "status": "ok",
  "uptime": 3600,
  "connections": 42
}
```

### Logging

```mermaid
flowchart LR
    subgraph "Log Levels"
        L1[ERROR - Critical issues]
        L2[WARN - Potential problems]
        L3[INFO - Connection events]
        L4[DEBUG - Detailed traces]
    end
    
    subgraph "Log Events"
        E1[Connection established]
        E2[Authentication success]
        E3[Event broadcast]
        E4[Connection dropped]
    end
    
    L3 --> E1
    L3 --> E2
    L4 --> E3
    L2 --> E4
```

**Logging Strategy Explained:** The service logs key events at appropriate levels. Connections and authentications are INFO level. Event broadcasts are DEBUG for troubleshooting. Disconnections are WARN to track potential issues.

---

## Security

### CORS Configuration

```javascript
const io = new Server(httpServer, {
  cors: {
    origin: process.env.CORS_ORIGIN,
    methods: ['GET', 'POST'],
    credentials: true
  }
});
```

### Authentication Flow

```mermaid
flowchart TD
    A[Client Connects] --> B[Socket Connection]
    B --> C[Client Emits Authenticate]
    C --> D{Valid Credentials?}
    D -->|Yes| E[Join User Rooms]
    D -->|No| F[Disconnect]
    E --> G[Confirm Authenticated]
    F --> H[Log Security Event]
```

**Security Flow Explained:** Every socket connection must authenticate before joining rooms. Unauthenticated connections are disconnected, and security events are logged for monitoring.

---

## Performance

### Connection Pooling

```mermaid
graph TB
    subgraph "Single Socket Connection"
        S1[Client]
        S2[Socket Connection]
        S3[Multiple Event Channels]
    end
    
    subgraph "Benefits"
        B1[Reduced Overhead]
        B2[Lower Latency]
        B3[Efficient Bandwidth]
    end
    
    S1 --> S2 --> S3
    S3 --> B1
    S3 --> B2
    S3 --> B3
```

**Pooling Benefits Explained:** A single WebSocket connection multiplexes all events, reducing connection overhead compared to multiple HTTP requests. This results in lower latency and more efficient bandwidth usage.

### Scalability Considerations

| Factor | Current | Future |
|--------|---------|--------|
| Connections | Single Instance | Socket.io Cluster |
| Rooms | In-Memory | Redis Adapter |
| Persistence | None | Redis for Pub/Sub |

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

### Debug Mode

```bash
# Enable debug logging
DEBUG=socket.io* bun run dev
```

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
```
