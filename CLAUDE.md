# CLAUDE.md — KaraQueue

## Project purpose

KaraQueue is a locally-hosted karaoke queue web app. Users on the same LAN add YouTube karaoke links to a shared queue. The main screen plays them one by one automatically.

## Architecture

### Custom Next.js server (`server.ts`)
Next.js App Router does not support persistent WebSocket connections in API routes, so we use a custom HTTP server that mounts both Next.js and Socket.io on the same port (3000).

### Real-time (Socket.io)
All queue state changes are broadcast to every connected client via Socket.io. The main screen is the source of truth for playback — it fires the `song:ended` event when the YouTube player triggers `ENDED`, which the server handles to advance the queue.

### Persistence (PostgreSQL + Prisma)
Queue entries are stored in Postgres so the queue survives server restarts.

## Database schema

```
Song {
  id          String   @id @default(cuid())
  youtubeUrl  String
  addedBy     String
  position    Int
  status      SongStatus  // QUEUED | PLAYING | PLAYED
  createdAt   DateTime @default(now())
}
```

## Socket.io events

| Event | Direction | Payload | Description |
|---|---|---|---|
| `song:add` | client → server | `{ youtubeUrl, addedBy }` | User submits a song |
| `queue:update` | server → all clients | full queue array | Broadcast after any queue change |
| `song:ended` | client (main screen) → server | `{ songId }` | Player signals song finished |
| `song:skip` | client → server | `{ songId }` | Skip current song |

## Routes

| Route | Description |
|---|---|
| `/` | Main screen: embedded YouTube player, now playing, queue, played history |
| `/add` | Mobile-friendly form: YouTube URL + name input |

## Key behaviours

- On page load, main screen fetches current queue and renders state
- When `song:ended` fires, server marks current song `PLAYED`, promotes next `QUEUED` song to `PLAYING`, broadcasts `queue:update`
- If queue is empty when a song ends, player shows an idle/waiting state
- `/add` page is intentionally minimal — optimized for phones on the same LAN

## Environment variables

```
DATABASE_URL=postgresql://user:password@localhost:5432/karaqueue
```

## Dev commands

```bash
npm run dev     # starts custom server with Next.js + Socket.io
npx prisma migrate dev   # run migrations
npx prisma studio        # inspect DB
```

## Constraints

- Localhost only — no auth, no remote access intended
- No user accounts — identity is just a name string entered on /add
- YouTube IFrame API handles playback; no audio processing on the server
