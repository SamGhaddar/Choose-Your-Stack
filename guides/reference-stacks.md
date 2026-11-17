# Reference Stacks

Last updated: 2026-11-17

Start from the closest row below, then open the guide for each layer and adjust. Most projects end up as REST plus one or two add-ons.

## How to use this doc

Attach it with the guides you need, or reference it from a project's `CLAUDE.md`. Use the prompt in the [README](../README.md).

Default rule: pick the boring, widely used option for each layer, and add a second tool only when a named symptom in a guide's refactoring section shows up.

## Stack by project type

| Project type | Front end | Back end and database | API style and add-ons |
| --- | --- | --- | --- |
| Marketplace or booking app | Mobile app plus a web app | One back end, PostgreSQL | REST; webhooks for payments; push notifications on mobile and SSE on web for status updates; WebSockets only if in-app chat |
| E-commerce store | The platform's storefront, or Next.js if custom | Platform handles it, or PostgreSQL if custom | The platform's API (Shopify uses GraphQL) or REST; webhooks for orders and payments |
| Client website or landing page | Static site (Astro or Next.js static) | None; a few serverless endpoints for forms | Webhooks from form or CRM tools |
| AI chat or agent app | Web app (React or Next.js) | Python or Node, PostgreSQL | REST; SSE for streaming model output; queue for long jobs |
| Trading bot or live dashboard | Web dashboard | Python, PostgreSQL | REST for orders and history; WebSockets for market data; one always-on process, safe order placement, paper trading first |
| Chat or collaboration tool | Web and mobile app | Node, PostgreSQL (a JSONB column for varied message shapes) | REST for history and auth; WebSockets for live messages |
| Video calling feature | Web and mobile app | Existing back end | REST; WebRTC via LiveKit or Daily; WebSockets for signaling |
| IoT or device telemetry | Web dashboard | PostgreSQL or a time-series database | MQTT from devices; REST for the dashboard; queue for processing |
| Microservices backend | Any | One database per service | REST or GraphQL at the edge; gRPC between services; Kafka or RabbitMQ for events |
| Integration with a bank or government system | Any | Your own wrapper service | Whatever they offer (often SOAP), wrapped behind your own REST service |

## Layer guides

- [Frontend & mobile](frontend-and-mobile.md)
- [Backend](backend.md)
- [Databases](databases.md)
- [API styles](api-styles.md)
- [Auth & payments](auth-and-payments.md)
- [Hosting & deploy](hosting-and-deploy.md)
- [Glossary](glossary.md)

## Regrets to flag before building

- Money or reservation data in a database without strict rules (use PostgreSQL).
- Building your own sign-in instead of using a managed provider.
- Starting with microservices; begin with one back end and split later.
- Storing files inside the database.
- Trusting the browser to confirm a payment, or skipping webhook signature checks and duplicate-event handling.
- Picking a payment or hosting provider before checking it works in your target country.
