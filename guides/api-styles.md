# API Style Decision Guide

Last updated: 2026-11-17

Most projects need one main API style for request/response, plus one or two add-ons for real-time or events. Use this guide to pick them when starting a new project or refactoring an old one. An API is the waiter between the front end and the back end.

## How to use this doc

Attach it at the start of a Claude session, or reference it from a project's `CLAUDE.md` in Claude Code.

Prompt to paste with it:

> Use the attached API Style Decision Guide. Here is my project: [what it does, who uses it, web/mobile/IoT, real-time needs, team size, existing stack]. Recommend the main API style and any add-ons, explain why using the guide's criteria, and flag anything I'm likely to regret later.

## Quick picker

1. **Pick the main API (the first yes wins):**
    - A partner system only speaks SOAP? → SOAP, wrapped behind your own REST service
    - Several frontends need different data slices? → GraphQL
    - Anything else (most projects) → **REST (default)**
2. **Add one style for every yes:**
    - Server pushes live updates, one way only? → SSE
    - Both sides send messages often, in real time? → WebSockets
    - Need to react to another service's events? → Webhooks
    - Audio, video or screen sharing? → WebRTC (via LiveKit/Daily)
    - Devices or sensors sending telemetry? → MQTT
    - Slow work that can finish later? → Message queue
    - Internal services need fast, typed calls? → gRPC

Most projects end up with REST plus one or two add-ons. See [Reference stacks](reference-stacks.md) for common combinations. The flowchart version is in the [README](../README.md).

## Side-by-side comparison

| Style | Who starts the talk | Data format | Runs over | Works in browser | Best at | Weak at |
| --- | --- | --- | --- | --- | --- | --- |
| REST | Client asks | JSON (usually) | HTTP | Yes | Simple CRUD, public APIs, caching | Over/under-fetching, real-time |
| GraphQL | Client asks | JSON | HTTP (subscriptions over WebSocket) | Yes | Many screens needing different slices of data | HTTP caching, simple APIs, complexity |
| gRPC | Client asks, or streams both ways | Protobuf (binary) | HTTP/2 | Only via gRPC-Web proxy | Fast service-to-service calls, strict contracts | Browsers, human debugging |
| SOAP | Client asks | XML | HTTP, SMTP, others | Awkward | Integrating with legacy bank/gov/health systems | Everything new |
| Webhooks | Server calls you | JSON | HTTP POST | n/a (server to server) | Event notifications between services | Needs a public URL, retries, signature checks |
| SSE | Server pushes, one way | Text/JSON | HTTP | Yes | Live feeds, AI token streaming, notifications | Client-to-server messages; browser EventSource is GET-only (AI streaming often uses fetch over POST); about 6 connections per domain on HTTP/1.1 |
| WebSockets | Both, anytime | Text, JSON, binary | Upgraded HTTP, persistent | Yes | Chat, live dashboards, multiplayer, trading feeds | Scaling many open connections, no built-in caching |
| WebRTC | Peer to peer | Audio, video, data channels | UDP mostly | Yes | Video/voice calls, screen share, P2P file transfer | Needs signaling + TURN servers, hard to debug |
| MQTT | Pub/sub via a broker | Any bytes | TCP (or WebSocket) | Via WebSocket | IoT devices, sensors, vehicles, flaky networks | General web APIs |
| Message queue (RabbitMQ, Kafka, SQS) | Producer to broker to worker | Any | TCP | No | Background jobs (a Redis- or Postgres-backed queue is enough for small apps), decoupling microservices, event logs (Kafka) | Direct user-facing calls |

## Each style in short

### REST

The waiter model: one URL per resource, HTTP verbs (GET, POST, PUT/PATCH, DELETE), stateless requests.

- **Use when:** you're building a typical web or mobile backend, a public API, or you're unsure. It's the default.
- **Avoid when:** clients need very different data shapes per screen (consider GraphQL), or you need live updates (add SSE/WebSockets).
- **Tooling:** OpenAPI spec, FastAPI, Express/NestJS, Next.js route handlers, Supabase auto-REST.

### GraphQL

One endpoint. The client writes a query asking for exactly the fields it needs. With REST the server decides what each address returns; with GraphQL the client lists the fields it wants in one request.

- **Use when:** several frontends (web, iOS, Android) need different slices of related data, or REST is forcing many round trips per screen.
- **Avoid when:** the API is small or CRUD-shaped, or you rely on HTTP caching/CDNs. It adds schema, resolver and N+1 query work.
- **Tooling:** Apollo, GraphQL Yoga, Hasura, Supabase pg_graphql.

### gRPC

You call a function on another machine. Contracts are defined in `.proto` files, data is binary Protobuf over HTTP/2, and it supports streaming in both directions.

- **Use when:** you have internal service-to-service calls (microservices), need high throughput or low latency, or work in polyglot teams that need strict typed contracts.
- **Avoid when:** the client is a browser (needs a gRPC-Web proxy) or the API is public-facing.
- **Tooling:** protoc/buf, grpc-go, grpcio (Python), Connect RPC (browser-friendly).

### SOAP

Strict XML envelopes with a WSDL contract and built-in WS-Security/transactions.

- **Use when:** you must integrate with an existing bank, insurer, government or hospital system that only speaks SOAP.
- **Avoid when:** building anything new. Use REST or gRPC instead.

### Webhooks

The "reverse API": you register a URL, and the other service POSTs to it when an event happens.

- **Use when:** reacting to third-party events (payment succeeded, code pushed, order placed) or notifying your own customers' systems.
- **Must do:** verify signatures, respond fast (queue the work), tolerate duplicate or out-of-order events (deduplicate by the provider's event ID), log every event.
- **Examples:** Stripe, GitHub, Shopify, Twilio.

### SSE (Server-Sent Events)

A normal HTTP response that stays open and streams events one way, server to client. The browser reconnects automatically.

- **Use when:** you only need server → client push: notifications, live status, progress bars, streaming LLM output.
- **Avoid when:** the client also sends frequent messages (use WebSockets), or the client is a mobile app that may be in the background (use push notifications there).
- **Note:** this is how most AI chat apps stream replies. It is simpler than WebSockets and works through most proxies. Browser `EventSource` is GET-only with no custom headers, so AI streaming often uses fetch streaming over POST instead.

### WebSockets

A persistent two-way channel opened by an HTTP upgrade handshake.

- **Use when:** both sides talk often and fast: chat, collaborative editing, multiplayer games, live trading/market data.
- **Avoid when:** one-way push is enough (SSE), or requests are occasional (REST).
- **Watch out for:** sticky sessions or a pub/sub layer (Redis) when scaling past one server, plus reconnect logic and heartbeats. Serverless hosts cannot hold WebSockets; use an always-on host.
- **Tooling:** Socket.IO, ws, Supabase Realtime, Pusher, Ably.

### WebRTC

Peer-to-peer audio, video and data between browsers or apps.

- **Use when:** you need video/voice calls, screen sharing or low-latency P2P data.
- **Reality:** it still needs a signaling server (often WebSockets) and STUN/TURN servers. Group calls usually go through an SFU media server.
- **Tooling:** LiveKit, Daily, Agora, Twilio Video. Rarely worth building from scratch.

### MQTT and message queues

Publish/subscribe through a broker. Senders and receivers never talk directly.

- **MQTT:** a lightweight protocol for devices, sensors and vehicles on unreliable networks (Mosquitto, EMQX, AWS IoT Core).
- **Queues/streams:** for background jobs and decoupling services (RabbitMQ, Kafka, SQS, BullMQ, Celery). Kafka is an event log rather than a job queue; for small apps a Redis- or Postgres-backed job queue is enough.
- **Use when:** work can happen later, many services react to one event, or devices report telemetry.

## Mixing styles in one app

Most apps use one main API style plus add-ons, each for a different job, and that is normal, not a mistake.

| Job | Style | Why |
| --- | --- | --- |
| Core records and profiles | REST (main) | Simple reads and writes on clear resources |
| Payment confirmations | Webhooks from the payment provider | The provider tells you when money moves |
| Live chat | WebSockets | Two-way and instant |
| Status updates | Push notifications on mobile, SSE on web | The server tells the user without a refresh |
| Long tasks such as PDFs or emails | Queue plus worker | The app stays fast while work runs in the background |

Chat normally uses WebSockets, not GraphQL, because plain REST or GraphQL cannot push messages instantly both ways. GraphQL subscriptions do exist, but they run over WebSocket anyway. REST and GraphQL can run side by side in one app, as GitHub does. Only add GraphQL when some screens need flexible data slices that REST handles badly. For most projects, REST plus the right add-ons is enough.

## Reference stacks

See [reference-stacks.md](reference-stacks.md) for the full stack by project type.

## Refactoring: when to switch

Only migrate when you see a symptom below. Add the new style alongside the old one rather than rewriting everything.

| Symptom | Likely cause | Move |
| --- | --- | --- |
| Clients poll an endpoint every few seconds | No push channel | Add SSE (one-way) or WebSockets (two-way) |
| A screen makes 5+ REST calls, or payloads are mostly unused fields | Over/under-fetching | Add a GraphQL layer or a few purpose-built REST endpoints (BFF) |
| Internal service calls are slow or contracts keep breaking | Loose JSON between services | gRPC between services, REST stays public |
| Request times out doing heavy work (emails, PDFs, AI calls) | Doing slow work inline | Queue + worker; notify via webhook/SSE when done |
| Missed or duplicated payment events | Fragile webhook handling | Signature checks, deduplicate by the provider's event ID, store first then process |
| WebSocket server falls over with more users | Single-node connection state | Redis pub/sub or a managed service (Ably, Pusher, Supabase Realtime) |
| GraphQL server is slow or complex for a small API | Overkill | Simplify back to REST |

Safe migration pattern: version the API (`/v2`) or run old and new side by side, move one client at a time, then remove the old path.

## Myths to ignore

- **"REST isn't serious enough for banks."** Most modern banking and open-banking APIs are REST. SOAP survives mainly in legacy systems.
- **"gRPC is 7–10× faster."** The gain depends on payload size and call volume. For a typical app backend, the database is the bottleneck, not JSON.
- **"GitHub's API is entirely GraphQL."** GitHub runs both a REST API and a GraphQL API.
- **"WebRTC needs no server."** It needs signaling and usually TURN relays. Group calls run through media servers.
- **"GraphQL is always better than REST."** It trades fetch flexibility for more server complexity and harder caching.
- **"Pick one API style."** Most real products combine a main style with push or event add-ons (see Mixing styles in one app).
