# API Style Decision Guide

Last updated: 2026-11-13

Most projects need one main API style for request/response, plus one or two add-ons for real-time or events. Use this guide to pick them when starting a new project or refactoring an old one.

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

Most projects end up with REST plus one or two add-ons. See Reference stacks for common combinations. The flowchart version is in the [README](../README.md).

## Side-by-side comparison

| Style | Who starts the talk | Data format | Runs over | Works in browser | Best at | Weak at |
| --- | --- | --- | --- | --- | --- | --- |
| REST | Client asks | JSON (usually) | HTTP | Yes | Simple CRUD, public APIs, caching | Over/under-fetching, real-time |
| GraphQL | Client asks | JSON | HTTP (subscriptions over WebSocket) | Yes | Many screens needing different slices of data | HTTP caching, simple APIs, complexity |
| gRPC | Client asks, or streams both ways | Protobuf (binary) | HTTP/2 | Only via gRPC-Web proxy | Fast service-to-service calls, strict contracts | Browsers, human debugging |
| SOAP | Client asks | XML | HTTP, SMTP, others | Awkward | Integrating with legacy bank/gov/health systems | Everything new |
| Webhooks | Server calls you | JSON | HTTP POST | n/a (server to server) | Event notifications between services | Needs a public URL, retries, signature checks |
| SSE | Server pushes, one way | Text/JSON | HTTP | Yes | Live feeds, AI token streaming, notifications | Client-to-server messages |
| WebSockets | Both, anytime | Text, JSON, binary | Upgraded HTTP, persistent | Yes | Chat, live dashboards, multiplayer, trading feeds | Scaling many open connections, no built-in caching |
| WebRTC | Peer to peer | Audio, video, data channels | UDP mostly | Yes | Video/voice calls, screen share, P2P file transfer | Needs signaling + TURN servers, hard to debug |
| MQTT | Pub/sub via a broker | Any bytes | TCP (or WebSocket) | Via WebSocket | IoT devices, sensors, cars, flaky networks | General web APIs |
| Message queue (RabbitMQ, Kafka, SQS) | Pub/sub via a broker | Any | TCP | No | Background jobs, decoupling microservices, event logs | Direct user-facing calls |

## Each style in short

### REST

The waiter model: one URL per resource, HTTP verbs (GET, POST, PUT/PATCH, DELETE), stateless requests.

- **Use when:** you're building a typical web or mobile backend, a public API, or you're unsure. It's the default.
- **Avoid when:** clients need very different data shapes per screen (consider GraphQL), or you need live updates (add SSE/WebSockets).
- **Tooling:** OpenAPI spec, FastAPI, Express/NestJS, Next.js route handlers, Supabase auto-REST.

### GraphQL

One endpoint. The client writes a query asking for exactly the fields it needs.

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
- **Must do:** verify signatures, respond fast (queue the work), make handlers idempotent (events can arrive twice or out of order), log every event.
- **Examples:** Stripe, GitHub, Shopify, Twilio.

### SSE (Server-Sent Events)

A normal HTTP response that stays open and streams events one way, server to client. The browser reconnects automatically.

- **Use when:** you only need server → client push: notifications, live status, progress bars, streaming LLM output.
- **Avoid when:** the client also sends frequent messages (use WebSockets).
- **Note:** this is how most AI chat apps stream replies. It is simpler than WebSockets and works through most proxies.

### WebSockets

A persistent two-way channel opened by an HTTP upgrade handshake.

- **Use when:** both sides talk often and fast: chat, collaborative editing, multiplayer games, live trading/market data.
- **Avoid when:** one-way push is enough (SSE), or requests are occasional (REST).
- **Watch out for:** sticky sessions or a pub/sub layer (Redis) when scaling past one server, plus reconnect logic and heartbeats.
- **Tooling:** Socket.IO, ws, Supabase Realtime, Pusher, Ably.

### WebRTC

Peer-to-peer audio, video and data between browsers or apps.

- **Use when:** you need video/voice calls, screen sharing or low-latency P2P data.
- **Reality:** it still needs a signaling server (often WebSockets) and STUN/TURN servers. Group calls usually go through an SFU media server.
- **Tooling:** LiveKit, Daily, Agora, Twilio Video. Rarely worth building from scratch.

### MQTT and message queues

Publish/subscribe through a broker. Senders and receivers never talk directly.

- **MQTT:** a lightweight protocol for devices, sensors and vehicles on unreliable networks (Mosquitto, EMQX, AWS IoT Core).
- **Queues/streams:** for background jobs and decoupling services (RabbitMQ, Kafka, SQS, BullMQ, Celery).
- **Use when:** work can happen later, many services react to one event, or devices report telemetry.

## Reference stacks

Real projects mix styles. Start from the closest row and adjust.

| Project type | Main API | Add-ons | Why |
| --- | --- | --- | --- |
| Marketplace / booking app (e.g. Revv) | REST | Webhooks for payments; SSE or push notifications for booking status; WebSockets only if in-app chat | CRUD-heavy, mobile clients, third-party payments |
| E-commerce store (e.g. INER-G) | REST, or the platform's API (Shopify GraphQL) | Webhooks for orders and payments | Platform does most of the work |
| Client websites / landing pages | None, or a few REST/serverless endpoints | Webhooks from form/CRM tools | Mostly static |
| AI chat or agent app | REST | SSE for streaming model output; queue for long jobs | Token streaming is one-way |
| Trading bot / live dashboard | REST for orders and history | WebSockets for market data; queue for signals | Prices change every millisecond |
| Chat / collaboration tool | REST for history and auth | WebSockets for live messages | Two-way, high frequency |
| Video calling feature | REST | WebRTC via LiveKit/Daily, WebSockets for signaling | Media needs P2P/SFU |
| IoT / vehicle telemetry (e.g. EV diagnostics) | MQTT from devices | REST for the dashboard; queue for processing | Low power, unreliable networks |
| Microservices backend | REST or GraphQL at the edge | gRPC between services; Kafka/RabbitMQ for events | Public stays simple, internals stay fast |
| Integration with a bank or government system | Whatever they offer (often SOAP) | Wrap it behind your own REST service | Keep legacy at the edge |

## Refactoring: when to switch

Only migrate when you see a symptom below. Add the new style alongside the old one rather than rewriting everything.

| Symptom | Likely cause | Move |
| --- | --- | --- |
| Clients poll an endpoint every few seconds | No push channel | Add SSE (one-way) or WebSockets (two-way) |
| A screen makes 5+ REST calls, or payloads are mostly unused fields | Over/under-fetching | Add a GraphQL layer or a few purpose-built REST endpoints (BFF) |
| Internal service calls are slow or contracts keep breaking | Loose JSON between services | gRPC between services, REST stays public |
| Request times out doing heavy work (emails, PDFs, AI calls) | Doing slow work inline | Queue + worker; notify via webhook/SSE when done |
| Missed or duplicated payment events | Fragile webhook handling | Signature checks, idempotency keys, store-then-process |
| WebSocket server falls over with more users | Single-node connection state | Redis pub/sub or a managed service (Ably, Pusher, Supabase Realtime) |
| GraphQL server is slow or complex for a small API | Overkill | Simplify back to REST |

Safe migration pattern: version the API (`/v2`) or run old and new side by side, move one client at a time, then remove the old path.

## Myths to ignore

- **"REST isn't serious enough for banks."** Most modern banking and open-banking APIs are REST. SOAP survives mainly in legacy systems.
- **"gRPC is 7–10× faster."** The gain depends on payload size and call volume. For a typical app backend, the database is the bottleneck, not JSON.
- **"GitHub's API is entirely GraphQL."** GitHub runs both a REST API and a GraphQL API.
- **"WebRTC needs no server."** It needs signaling and usually TURN relays. Group calls run through media servers.
- **"GraphQL is always better than REST."** It trades fetch flexibility for more server complexity and harder caching.
- **"Pick one API style."** Most real products combine a main style with push or event add-ons (see Reference stacks).
