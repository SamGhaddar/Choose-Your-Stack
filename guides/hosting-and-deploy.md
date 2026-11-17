# Hosting and Deploy Guide

Last updated: 2026-11-17

Buy these layers instead of building them; a managed service is almost always safer and faster than a home-made one. Start on a managed platform and move to AWS or GCP only at real scale or a compliance need.

## How to use this doc

Attach it at the start of a Claude session, or reference it from a project's `CLAUDE.md`. Use the prompt in the [README](../README.md).

## Quick picker

| Concern | Default | Rule |
| --- | --- | --- |
| Hosting | Static or Next.js sites: Vercel, Netlify or Cloudflare Pages. Custom back end: Railway, Render or Fly.io | Serverless hosts cannot hold WebSockets and cut off long streams and jobs, so apps with WebSockets, always-running bots or background workers need an always-on host (Railway, Render, Fly.io). Move to AWS or GCP only at real scale or a compliance need |
| Files and images | Object storage (S3, Cloudflare R2, Supabase Storage) | Keep the file in storage and only its link in the database |
| Email | A transactional email service (Resend, Postmark) | Verify your sending domain before launch |
| Errors and uptime | Error tracking (Sentry) plus an uptime check | Add before launch, not after the first outage |
| Secrets | Environment variables on the host | Never commit keys to git; anything in front-end or mobile code (such as NEXT_PUBLIC_ variables) is public |

Picking a hosting provider before checking it works in your target country is a common regret; confirm availability first.
