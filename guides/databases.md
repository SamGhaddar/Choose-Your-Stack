# Database Decision Guide

Last updated: 2026-11-17

Use PostgreSQL by default. For items with different shapes, start with a JSONB column inside PostgreSQL, and add MongoDB only when that stops working at scale; one database is simpler for a solo developer. SQL databases store data in tables with fixed columns; MongoDB stores flexible documents.

## How to use this doc

Attach it at the start of a Claude session, or reference it from a project's `CLAUDE.md`. Use the prompt in the [README](../README.md).

## Quick picker

| Database | Choose when | Avoid when |
| --- | --- | --- |
| PostgreSQL | Connected data with relationships, money, complex queries; the default | You only store loose, constantly changing documents |
| MySQL | Your host or team already uses it, or a WordPress or PHP stack | You are starting fresh with no constraint (pick PostgreSQL) |
| MongoDB | Each item can have different fields (chat messages, logs, varied catalogs), and a PostgreSQL JSONB column no longer fits | Data has many relationships or strict money rules |
| SQLite | Small apps, prototypes, local or single-user tools | Many users write at the same time |
| Redis | Caching, sessions, rate limits, short-lived data | It would be the only place the data lives |

Rule of thumb: keep data that must stay exact (money, reservations) in PostgreSQL with database constraints, and let records whose fields differ per item start in a JSONB column, moving to MongoDB only if volume demands it.

## Before launch

Set up automated backups and test one restore, and use a migration tool or ORM (Prisma, Drizzle, Alembic) so schema changes are tracked.

## Refactoring: when to switch

| Symptom | Likely cause | Move |
| --- | --- | --- |
| Tables full of empty columns, or constant schema changes for varied records | Item shapes differ | Move that data to MongoDB or a JSON column in PostgreSQL; keep the rest relational |
| Pages re-read the same data constantly | No caching | Add Redis or HTTP caching before scaling the database |
