# 10,000 Checkboxes

A real-time collaborative checkbox grid with 10,000 checkboxes, built with **WebSocket** and **Redis**. Features an authentication layer for access control.

**Live Demo:** [https://check-box-redis.onrender.com](https://check-box-redis.onrender.com)

---

## Project Overview

10,000 Checkboxes is a real-time collaborative application where multiple users can simultaneously interact with a grid of 10,000 checkboxes. Changes made by any user are instantly synchronized across all connected clients using WebSocket connections. The application uses Redis for state persistence and pub/sub messaging, ensuring scalability across multiple server instances. A JWT-based authentication layer controls access to the grid.

**Key Design Decisions:**
- **Redis Pub/Sub**: Enables horizontal scaling — multiple server instances can synchronize state across clients
- **Bit-level Storage**: Checkbox states stored efficiently in Redis bitmaps (10,000 bits ≈ 1.25 KB)
- **JWT Auth**: Stateless authentication with httpOnly cookie-based refresh tokens
- **Rate Limiting**: Prevents abuse of real-time endpoints and authentication APIs

---

## Tech Stack

| Layer | Tech | Purpose |
|-------|------|---------|
| **Runtime** | Node.js | Server runtime |
| **Framework** | Express | HTTP server & middleware |
| **ORM** | Drizzle ORM | Type-safe SQL queries |
| **Database** | PostgreSQL (NeonDB / Docker) | User data persistence |
| **Cache/State** | Redis (Docker / Cloud) | Checkbox state + Pub/Sub |
| **Real-time** | Socket.io | Bidirectional WebSocket events |
| **Auth** | JWT (jsonwebtoken) + bcryptjs | Access/refresh token flow |
| **Validation** | Joi | Request body validation |
| **Rate Limit** | express-rate-limit | API protection |
| **Env Config** | dotenv | Environment management |
| **Extras** | cookie-parser, cors, pg | Middleware utilities |

---

## Features Implemented

### Core Features
- **10,000 Interactive Checkboxes** — Grid rendered client-side with instant toggle feedback
- **Real-time Sync** — WebSocket broadcasts checkbox changes to all connected clients
- **State Persistence** — Checkbox states survive server restarts via Redis bitmap storage
- **Cross-Instance Sync** — Redis Pub/Sub synchronizes state across multiple server nodes

### Authentication Features
- **JWT Access Tokens** — Short-lived tokens (15 min - 1 day) for API authorization
- **Refresh Token Rotation** — Long-lived refresh tokens (7 days) stored in httpOnly cookies
- **Password Hashing** — bcryptjs with salt rounds for secure password storage
- **Protected Routes** — Middleware guards WebSocket connections and checkbox APIs

### Operational Features
- **Rate Limiting** — Tiered limits on auth endpoints and checkbox toggle events
- **Input Validation** — Joi schemas validate all incoming requests
- **CORS Configuration** — Secure cross-origin setup for production
- **Health Check Endpoint** — `GET /health` for monitoring

---

## Environment Variables

Create a `.env` file in the project root (see `.env.example`):

```env
# JWT Authentication
JWT_ACCESS_SECRET=your_super_secret_access_key_min_32_chars
JWT_ACCESS_EXPIRES_IN=1d
JWT_REFRESH_SECRET=your_super_secret_refresh_key_min_32_chars
JWT_REFRESH_EXPIRES_IN=7d

# Database
DATABASE_URL=postgresql://user:password@host:port/database

# Redis
REDIS_URL=redis://localhost:6379
# For Redis Cloud: redis://default:password@host:port

```

---

## Local Setup

### Prerequisites
- Node.js (v18+ recommended)
- Docker (for local Redis & PostgreSQL) — or use cloud services

### 1. Clone & Install

```bash
git clone https://github.com/tilak9606/check-box-redis.git
cd check-box-redis
npm install
```

### 2. Start Infrastructure (Docker)

```bash
# Start Redis
docker run -d -p 6379:6379 --name redis redis:alpine

# Start PostgreSQL
docker run -d -p 5432:5432 \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=checkboxes \
  --name postgres postgres:alpine
```

> **Alternative**: Use cloud services like [NeonDB](https://neon.tech) for PostgreSQL and [Upstash](https://upstash.com) for Redis — just update `DATABASE_URL` and `REDIS_URL` in `.env`.

### 3. Configure Environment

```bash
cp .env.example .env
# Edit .env with your actual values
```

### 4. Initialize Database

```bash
# Drizzle ORM will handle migrations on first run
# Or run explicit migration if available:
npm run db:migrate
```

### 5. Run Application

```bash
npm run start
```

App runs at `http://localhost:8000`

### 6. Verify Setup

```bash
curl http://localhost:8000/health
# Expected: {"health": "true"}
```

---

### Implementation Details

| Component | Behavior |
|-----------|----------|
| **Access Token** | Short-lived JWT in `Authorization` header |
| **Refresh Token** | Long-lived JWT in httpOnly, Secure, SameSite=strict cookie |
| **Token Rotation** | New refresh token issued on every refresh; old one invalidated |
| **Logout** | Clear cookie + blacklist refresh token in Redis |
| **Password Security** | bcryptjs with 10+ salt rounds |

### Auth Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Create new account |
| POST | `/api/auth/login` | Authenticate, receive tokens |
| POST | `/api/auth/refresh-token` | Rotate access token (cookie-based) |
| POST | `/api/auth/logout` | Clear session |

---

### Event Flow

1. **Client connects** → Server authenticates via JWT in handshake headers
2. **Client toggles checkbox** → Emits `checkbox:toggle` with `{index, checked}`
3. **Server validates** → Rate limit check + index bounds (0-9999)
4. **Server updates Redis** → `SETBIT checkboxes:index index value`
5. **Server publishes** → `PUBLISH checkbox:updates {index, checked, timestamp}`
6. **All clients receive** → `checkbox:updated` event updates UI

### Socket.io Events

| Event | Direction | Payload | Description |
|-------|-----------|---------|-------------|
| `connection` | C→S | `{token}` | Auth via JWT in handshake |
| `checkbox:toggle` | C→S | `{index: number, checked: boolean}` | User toggles checkbox |
| `checkbox:updated` | S→C | `{index, checked, userId?, timestamp}` | Broadcast to all clients |
| `checkbox:batch` | S→C | `Array<{index, checked}>` | Initial state sync on connect |
| `error` | S→C | `{message}` | Validation/auth errors |

### Redis Data Structure

```
Key: checkboxes:index
Type: Bitmap (10,000 bits)
Operations: SETBIT / GETBIT / BITFIELD
```

---

## Redis Setup Instructions

### Option 1: Local Docker (Recommended for Development)

```bash
# Pull and run Redis
docker pull redis:alpine
docker run -d -p 6379:6379 --name redis redis:alpine

# Verify
docker exec -it redis redis-cli ping
# Expected: PONG

# Monitor in real-time (optional)
docker exec -it redis redis-cli monitor
```

### Option 2: Redis Cloud (Production)

Services like [Upstash](https://upstash.com) or [Redis Cloud](https://redis.com/cloud/):

```env
REDIS_URL=redis://default:password@your-host.upstash.io:6379
```

### Option 3: Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'
services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
  postgres:
    image: postgres:alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: checkboxes
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  redis_data:
  postgres_data:
```

Run: `docker-compose up -d`

---

## Demo Credentials

```
Email:    abc@gmail.com
Password: Abc@123456
```

---

##Demo

**Live Demo:** https://youtu.be/9JqrwbVntK4