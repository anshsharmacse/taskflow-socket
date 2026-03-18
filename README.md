# TaskFlow Socket Service

Real-time WebSocket service for TaskFlow task management application.

## Deployment Options

### Option 1: Deploy on Railway

1. **Create a Railway account** at [railway.app](https://railway.app)

2. **Create a new project** and select "Deploy from GitHub repo"

3. **Connect your repository** containing this socket service

4. **Set environment variables:**
   ```
   CORS_ORIGIN=https://your-taskflow-app.vercel.app
   ```

5. **Railway will automatically:**
   - Detect the package.json
   - Install dependencies
   - Run `npm run build` and `npm start`
   - Assign a public URL

6. **Note the deployed URL** (e.g., `https://taskflow-socket.up.railway.app`)

---

### Option 2: Deploy on Render

1. **Create a Render account** at [render.com](https://render.com)

2. **Create a new Web Service**

3. **Connect your repository**

4. **Configure the service:**
   - **Name:** taskflow-socket
   - **Environment:** Node
   - **Build Command:** `npm install && npm run build`
   - **Start Command:** `npm start`

5. **Add environment variable:**
   - Key: `CORS_ORIGIN`
   - Value: `https://your-taskflow-app.vercel.app`

6. **Deploy** and note the URL (e.g., `https://taskflow-socket.onrender.com`)

---

### Option 3: Deploy on Fly.io

1. **Install flyctl:** `curl -L https://fly.io/install.sh | sh`

2. **Login:** `fly auth login`

3. **Create a `Dockerfile`:**
   ```dockerfile
   FROM oven/bun:1
   
   WORKDIR /app
   
   COPY package.json ./
   RUN bun install
   
   COPY . .
   RUN bun build index.ts --outdir dist --target node
   
   ENV PORT=8080
   EXPOSE 8080
   
   CMD ["node", "dist/index.js"]
   ```

4. **Deploy:**
   ```bash
   fly launch
   fly deploy
   ```

---

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `PORT` | Server port (auto-set by most platforms) | `3003` |
| `CORS_ORIGIN` | Comma-separated allowed origins | `https://taskflow.vercel.app` |

---

## Update TaskFlow Frontend

After deploying the socket service, add this environment variable to your Vercel project:

```
NEXT_PUBLIC_SOCKET_URL=https://your-socket-service-url
```

### In Vercel Dashboard:
1. Go to your project → Settings → Environment Variables
2. Add `NEXT_PUBLIC_SOCKET_URL` with your deployed socket URL
3. Redeploy the application

---

## Local Development

```bash
# Install dependencies
bun install

# Run in development mode
bun run dev

# Build for production
bun run build

# Start production server
npm start
```

---

## API Events

### Client → Server
- `authenticate` - Authenticate user with `{ userId, email, name }`
- `task:created` - Notify about task creation `{ task, assigneeEmail }`
- `task:updated` - Notify about task update `{ task, previousAssigneeEmail }`
- `task:deleted` - Notify about task deletion `{ taskId, taskTitle, assigneeEmail, creatorEmail }`

### Server → Client
- `authenticated` - Confirmation of authentication
- `task:assigned` - New task assigned to user
- `task:updated` - Task was updated
- `task:deleted` - Task was deleted

---

## Developer

**ANSH SHARMA**  
National Institute of Technology Calicut
