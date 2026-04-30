# 10,000 Checkboxes

A real-time collaborative checkbox grid with 10,000 checkboxes, built with **WebSocket** and **Redis**. Features an authentication layer for access control.

**Live Demo:** [https://check-box-redis.onrender.com](https://check-box-redis.onrender.com)

---

## Features

- 10,000 interactive checkboxes synced in real-time across all connected clients
- WebSocket-based live state synchronization
- Redis for state persistence and pub/sub
- JWT-based authentication (access + refresh tokens)
- PostgreSQL for user data

---

## Tech Stack

| Layer | Tech |
|-------|------|
| **Runtime** | Node.js |
| **Framework** | Express |
| **ORM** | Drizzle ORM |
| **Database** | PostgreSQL (NeonDB / Docker) |
| **Cache/State** | Redis (Docker / Cloud) |
| **Real-time** | Socket.io |
| **Auth** | JWT (jsonwebtoken) + bcryptjs |
| **Validation** | Joi |
| **Env Config** | dotenv |
| **Extras** | cookie-parser, cors, pg |

---

## Environment Variables

Create a `.env` file:

```env
JWT_ACCESS_SECRET=your_access_secret
JWT_ACCESS_EXPIRES_IN=1d
JWT_REFRESH_SECRET=your_refresh_secret
JWT_REFRESH_EXPIRES_IN=7d

DATABASE_URL=postgresql://user:pass@host:port/db
REDIS_URL=redis://host:port
```

---

## Local Setup

### Prerequisites
- Node.js
- Docker (for local Redis & PostgreSQL)

### 1. Clone & Install

```bash
git clone https://github.com/tilak9606/check-box-redis.git
cd check-box-redis
npm install
```

### 2. Start Services (Docker)

```bash
docker run -d -p 6379:6379 --name redis redis:alpine
docker run -d -p 5432:5432 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=checkboxes --name postgres postgres:alpine

You can use online DB service also in .env DATABASE_URL
```

Create `.env`:
```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/checkboxes
REDIS_URL=redis://localhost:6379
```

### 3. Run

```bash
npm run start
```

App runs at `http://localhost:8000`

---

## Entry Point

`server.js`

---

## Demo User-Pass

```
abc@gmail.com
Abc@123456
```