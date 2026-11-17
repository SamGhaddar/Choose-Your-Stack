# Auth and Payments Guide

Last updated: 2026-11-17

Use a managed sign-in provider and a payment provider that supports your target country; never build either yourself.

## How to use this doc

Attach it at the start of a Claude session, or reference it from a project's `CLAUDE.md`. Use the prompt in the [README](../README.md).

## Quick picker

| Concern | Default | Rule |
| --- | --- | --- |
| Sign-in | A managed auth provider (Supabase Auth, Clerk, Auth0, Firebase Auth) | Never store passwords or invent your own scheme |
| Payments | A payment provider that supports your target country | Confirm the provider operates in your target country before designing around it; confirm payments by webhook, not by trusting the browser |

Authorization is separate from sign-in: every endpoint and database rule must check that the signed-in user owns the row they ask for. Rate-limit public and AI endpoints to cap abuse and cost.

Webhook handlers must verify signatures, respond fast (queue the work), and deduplicate by the provider's event ID; see [API styles](api-styles.md).
