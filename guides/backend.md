# Backend Decision Guide

Last updated: 2026-11-17

Pick the language you already know: for most app back ends, database design matters more than language speed. The back end is the kitchen, holding the logic and the rules.

## How to use this doc

Attach it at the start of a Claude session, or reference it from a project's `CLAUDE.md`. Use the prompt in the [README](../README.md).

## Quick picker

| Language | Choose when | Common framework |
| --- | --- | --- |
| Python | AI, data, trading, scripting | FastAPI, Django |
| Node.js (TypeScript) | Same language as the front end, real-time features, small team | Express, NestJS, Next.js route handlers |
| PHP | WordPress sites, cheap shared hosting | Laravel |
| Ruby | Fast prototypes with batteries included | Rails |
| Java | Large enterprise systems with strict structure | Spring |

## Skipping a custom back end

Skip a custom back end when the app is mostly sign-in plus simple data (CRUD): a backend-as-a-service such as Supabase gives you the database, auth and an automatic REST API. If you use it, turn on Row Level Security for every table, otherwise the public key can expose your data. Move to your own back end once you have custom business rules, heavy background jobs or complex payments.
