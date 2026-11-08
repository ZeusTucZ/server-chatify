# Chatify Server — Real-Time Chat Backend

Express + Socket.io + PostgreSQL backend for
[chatify-app](https://github.com/ZeusTucZ/chatify-app), a real-time chat web
app and class project. This repository holds the server only.

## What it does

- Accepts Socket.io connections from the frontend.
- Stores every chat message in a PostgreSQL `messages` table:

  ```sql
  CREATE TABLE IF NOT EXISTS messages (
      id SERIAL PRIMARY KEY,
      client_offset TEXT UNIQUE,
      content TEXT
  );
  ```

  The table is created automatically on startup. The `client_offset` column
  exists in the schema but the current code does not use it when inserting
  messages.
- When a client connects without a recovered session, replays the messages it
  missed: it selects rows with `id` greater than the `serverOffset` sent by
  the client and emits each one as a `chat message` event together with its
  id.
- When a client sends a `chat message`, inserts it into the database and
  broadcasts it to all connected clients with its new id.
- `GET /` returns a small placeholder page.

## Stack

- Express 4
- Socket.io 4
- `pg` (PostgreSQL client)

## Architecture

```text
Browser — chatify-app (React + Vite + socket.io-client)
  ↔ Socket.io
Express + Socket.io server (this repo)
  ↔ PostgreSQL (messages table)
```

## Run locally

Requirements: Node.js and a PostgreSQL database.

```bash
npm install
```

Set the environment variables (see below), then:

```bash
npm start   # runs node index.js
```

| Variable       | Default                 | Description                              |
| -------------- | ----------------------- | ---------------------------------------- |
| `DATABASE_URL` | — (required)            | PostgreSQL connection string             |
| `PORT`         | `3000`                  | Port the server listens on (all interfaces) |
| `CLIENT_URL`   | `http://localhost:5173` | Allowed CORS origin for the frontend     |

The frontend connects to `http://localhost:3000` by default (or to the URL in
its `VITE_SOCKET_URL` variable), so run this server before opening
[chatify-app](https://github.com/ZeusTucZ/chatify-app).

## Current state and limitations

- Class project, work in progress.
- No tests — the `test` script in `package.json` is the npm placeholder and
  exits with an error. No CI and no live demo URL.
- No authentication: anyone who can reach the server can connect and post.
- No channel-scoped delivery yet: messages are broadcast to every connected
  client, even though the frontend has a channel-switching UI.
