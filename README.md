# KaraQueue

A locally-hosted karaoke queue web app where multiple users on the same network can add YouTube karaoke links to a shared queue that plays in real time.

## What it does

- Anyone on the local network opens `localhost:3000/add`, pastes a YouTube karaoke link, and enters their name
- The main screen at `localhost:3000` plays songs one by one from the queue
- When a song ends, the next one plays automatically
- All connected clients see the queue update in real time

## Stack

- **Next.js** (App Router, TypeScript) with a custom server
- **PostgreSQL** + Prisma (persistence)
- **Socket.io** (real-time WebSocket sync)
- **YouTube IFrame Player API** (embedded player + auto-advance on song end)

## Views

| Route | Purpose |
|---|---|
| `localhost:3000` | Main screen — player, now playing, queue, history |
| `localhost:3000/add` | Mobile-friendly form to add a song |

## Song lifecycle

`queued` → `playing` → `played`